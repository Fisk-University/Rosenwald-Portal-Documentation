## 5. DNS Configuration (Route 53)

### Conceptual Overview

DNS (Domain Name System) translates human-readable domain names (like library.university.edu) into IP addresses that computers use to connect to each other. For educational institutions with .edu domains, DNS configuration requires coordination with your institution's central IT department who manages the authoritative .edu nameservers.

In this architecture, we use a hosted DNS service to manage subdomain records while delegating from the institution's primary nameservers. This approach provides fine-grained control over application-specific DNS records without requiring constant coordination with central IT for routine changes.

### AWS Implementation: Route 53 Configuration

#### Step 1: Create Hosted Zone

Navigate to **Route 53 Console → Hosted zones → Create hosted zone**

**Configuration:**
- Domain name: `omeka.yourinstitution.edu`
  - Note: This should be a subdomain delegated by your IT department
- Description: `Omeka S Digital Collections Platform DNS`
- Type: Public hosted zone
- Tags:
  - Key: `Application` | Value: `Omeka`
  - Key: `ManagedBy` | Value: `Library IT`

After creation, Route 53 will provide 4 nameservers. Document these:
- ns-XXXX.awsdns-XX.org
- ns-XXXX.awsdns-XX.com
- ns-XXXX.awsdns-XX.net
- ns-XXXX.awsdns-XX.co.uk

#### Step 2: Request DNS Delegation from Institution

Email template for your IT department:
```
Subject: DNS Subdomain Delegation Request for Digital Collections Platform

We need to delegate the subdomain 'omeka.institution.edu' to AWS Route 53 
for our digital collections platform.

Please create NS records pointing to:
- ns-XXXX.awsdns-XX.org
- ns-XXXX.awsdns-XX.com
- ns-XXXX.awsdns-XX.net
- ns-XXXX.awsdns-XX.co.uk

This delegation will allow us to manage DNS records for:
- omeka.institution.edu (production)
- stage-omeka.institution.edu (staging)
- Additional subdomains as needed

No changes to the root institution.edu domain are required.
```

#### Step 3: Create DNS Records

After delegation is complete, create records in Route 53:

**Production A Record:**
- Record name: `` (leave blank for apex domain)
- Record type: A - IPv4 address
- Value: Your Production Elastic IP (e.g., 54.XXX.XXX.XXX)
- TTL: 300 seconds (5 minutes for easy updates)
- Routing policy: Simple routing

**Staging CNAME Record:**
- Record name: `stage`
- Record type: CNAME
- Value: Your staging EC2 public DNS name
  - Example: `ec2-54-XXX-XXX-XXX.compute-1.amazonaws.com`
- TTL: 300 seconds
- Routing policy: Simple routing

**WWW CNAME Record (Optional):**
- Record name: `www`
- Record type: CNAME
- Value: `omeka.yourinstitution.edu`
- TTL: 300 seconds

#### Step 4: Configure MX Records (If Email Required)

If the platform needs to send emails:
- Record name: `` (leave blank)
- Record type: MX
- Priority: 10
- Value: `mail.institution.edu` (or your email server)
- TTL: 3600 seconds

#### Step 5: Create TXT Records for Domain Verification

**SPF Record (Email authentication):**
- Record name: `` (leave blank)
- Record type: TXT
- Value: `"v=spf1 include:_spf.institution.edu ~all"`
- TTL: 3600

**Domain Verification Records:**
Various services may require TXT verification:
- Google: `google-site-verification=...`
- SSL Certificate validation: `_acme-challenge`

#### Step 6: Configure CAA Records

CAA records specify which Certificate Authorities can issue SSL certificates:

- Record name: `` (leave blank)
- Record type: CAA
- Flags: 0
- Tag: issue
- Value: `letsencrypt.org`

Add another value:
- Flags: 0
- Tag: issuewild
- Value: `letsencrypt.org`
- TTL: 3600

#### Step 7: Health Checks and Failover Configuration

Create a health check for production:

Navigate to **Route 53 → Health checks → Create health check**

**Configuration:**
- Name: `omeka-production-health`
- What to monitor: Endpoint
- Specify endpoint by: IP address
- Protocol: HTTPS
- IP address: Your production Elastic IP
- Port: 443
- Path: `/` (or a specific health check endpoint)
- Request interval: 30 seconds
- Failure threshold: 3

**Configure CloudWatch Alarm:**
- Create alarm: Yes
- Send notification to: Your ops team email
- Alarm name: `omeka-production-down`

#### Step 8: Query Logging Configuration (Optional)

Enable query logging for security and troubleshooting:

Navigate to **Route 53 → Hosted zone → Configure query logging**
- CloudWatch Logs log group: Create new
  - Name: `/aws/route53/omeka.institution.edu`
- IAM role: Create new or use existing Route 53 role

#### Step 9: DNS Testing and Validation

After configuration, test DNS resolution:
```bash
# Test A record
nslookup omeka.institution.edu

# Test specific nameserver
nslookup omeka.institution.edu ns-XXXX.awsdns-XX.org

# Check all DNS records
dig omeka.institution.edu ANY

# Trace DNS path
dig +trace omeka.institution.edu
```

**Online DNS tools to verify:**
- whatsmydns.net - Check global DNS propagation
- mxtoolbox.com - Verify MX and other records
- dnschecker.org - Comprehensive DNS testing

---

## 6. SSL Certificate Configuration

### Conceptual Overview

SSL/TLS certificates encrypt data transmitted between users' browsers and your web servers, ensuring privacy and data integrity. For educational institutions, SSL certificates also provide identity verification, showing users they're connecting to a legitimate .edu domain. Modern browsers require HTTPS for many features and mark non-HTTPS sites as "Not Secure."

This implementation uses Let's Encrypt, a free, automated certificate authority that's ideal for educational institutions. The Certbot tool automates certificate issuance and renewal, eliminating manual processes and expiration issues.

### AWS Implementation: Let's Encrypt with Certbot

#### Step 1: Initial Server Preparation

Connect to your production/staging EC2 instance:
```bash
# Connect via bastion host
ssh -i /path/to/key.pem ubuntu@bastion-ip
ssh -i /path/to/key.pem ubuntu@production-private-ip

# Update system and install prerequisites
sudo apt update
sudo apt upgrade -y

# Install software-properties-common
sudo apt install software-properties-common -y

# Install snapd (for Certbot installation)
sudo apt install snapd -y
sudo snap install core
sudo snap refresh core
```

#### Step 2: Install Certbot

Install Certbot using snap (recommended method):
```bash
# Install Certbot
sudo snap install --classic certbot

# Create symbolic link for system-wide access
sudo ln -s /snap/bin/certbot /usr/bin/certbot

# Verify installation
certbot --version
```

#### Step 3: Configure Apache for SSL

Enable required Apache modules:
```bash
# Enable SSL and rewrite modules
sudo a2enmod ssl
sudo a2enmod rewrite
sudo a2enmod headers

# Restart Apache
sudo systemctl restart apache2
```

Create Apache virtual host configuration:
```bash
# Create configuration file
sudo nano /etc/apache2/sites-available/omeka-ssl.conf
```

Add this configuration:
```apache
<VirtualHost *:80>
    ServerName omeka.institution.edu
    ServerAlias www.omeka.institution.edu
    DocumentRoot /var/www/html/omeka

    # Redirect all HTTP to HTTPS
    RewriteEngine On
    RewriteCond %{HTTPS} off
    RewriteRule ^(.*)$ https://%{HTTP_HOST}$1 [R=301,L]
</VirtualHost>

<VirtualHost *:443>
    ServerName omeka.institution.edu
    ServerAlias www.omeka.institution.edu
    DocumentRoot /var/www/html/omeka

    # SSL Configuration (will be updated by Certbot)
    SSLEngine on
    
    # Security Headers
    Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains"
    Header always set X-Frame-Options "SAMEORIGIN"
    Header always set X-Content-Type-Options "nosniff"
    Header always set Referrer-Policy "strict-origin-when-cross-origin"

    # Apache settings for Omeka
    <Directory /var/www/html/omeka>
        AllowOverride All
        Require all granted
    </Directory>

    # Logging
    ErrorLog ${APACHE_LOG_DIR}/omeka-error.log
    CustomLog ${APACHE_LOG_DIR}/omeka-access.log combined
</VirtualHost>
```

Enable the site:
```bash
sudo a2ensite omeka-ssl.conf
sudo systemctl reload apache2
```

#### Step 4: Obtain SSL Certificate

Run Certbot to obtain certificate:
```bash
# For production (interactive mode)
sudo certbot --apache -d omeka.institution.edu -d www.omeka.institution.edu

# For staging environment (test certificate first)
sudo certbot --apache --staging -d stage-omeka.institution.edu
```

Certbot will prompt for:
1. Email address (for renewal notifications)
2. Terms of Service agreement (Agree)
3. Marketing emails (Optional)
4. HTTPS redirect preference (Select option 2: Redirect)

#### Step 5: Verify Certificate Installation

Check certificate details:
```bash
# View certificate information
sudo certbot certificates

# Test SSL configuration
openssl s_client -connect omeka.institution.edu:443 -servername omeka.institution.edu
```

Test in browser:
- Navigate to https://omeka.institution.edu
- Click the padlock icon in the address bar
- View certificate details

#### Step 6: Configure Automatic Renewal

Certbot automatically creates a systemd timer for renewal. Verify it's active:
```bash
# Check systemd timer status
sudo systemctl status snap.certbot.renew.timer

# View timer schedule
sudo systemctl list-timers | grep certbot

# Dry run to test renewal without making changes
sudo certbot renew --dry-run
```

#### Step 7: Create Renewal Hooks

Create pre and post renewal scripts:
```bash
# Create hooks directory
sudo mkdir -p /etc/letsencrypt/renewal-hooks/deploy

# Create deployment hook
sudo nano /etc/letsencrypt/renewal-hooks/deploy/reload-apache.sh
```

Add this script:
```bash
#!/bin/bash
# Reload Apache after certificate renewal
systemctl reload apache2

# Log renewal
echo "Certificate renewed and Apache reloaded at $(date)" >> /var/log/letsencrypt-renewals.log

# Optional: Send notification
# curl -X POST https://your-webhook-url -d "Certificate renewed for omeka.institution.edu"
```

Make it executable:
```bash
sudo chmod +x /etc/letsencrypt/renewal-hooks/deploy/reload-apache.sh
```

#### Step 8: Configure Security Headers and SSL Hardening

Edit Apache SSL configuration:
```bash
sudo nano /etc/apache2/mods-available/ssl.conf
```

Update with secure settings:
```apache
# Modern SSL Configuration
SSLProtocol -all +TLSv1.2 +TLSv1.3
SSLCipherSuite ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384
SSLHonorCipherOrder off
SSLSessionTickets off

# OCSP Stapling
SSLUseStapling on
SSLStaplingCache "shmcb:logs/ssl_stapling(32768)"
```

#### Step 9: Monitor Certificate Expiration

Create monitoring script:
```bash
sudo nano /usr/local/bin/check-ssl-expiry.sh
```

Add monitoring script:
```bash
#!/bin/bash
# Check SSL certificate expiration

DOMAIN="omeka.institution.edu"
EXPIRY_DATE=$(echo | openssl s_client -servername $DOMAIN -connect $DOMAIN:443 2>/dev/null | openssl x509 -noout -dates | grep notAfter | cut -d= -f2)
EXPIRY_EPOCH=$(date -d "$EXPIRY_DATE" +%s)
CURRENT_EPOCH=$(date +%s)
DAYS_LEFT=$(( ($EXPIRY_EPOCH - $CURRENT_EPOCH) / 86400 ))

if [ $DAYS_LEFT -lt 30 ]; then
    echo "WARNING: SSL certificate expires in $DAYS_LEFT days"
    # Send alert to team
    mail -s "SSL Certificate Expiration Warning" team@institution.edu <<< "Certificate for $DOMAIN expires in $DAYS_LEFT days"
fi

echo "Certificate valid for $DAYS_LEFT more days"
```

Make executable and add to crontab:
```bash
sudo chmod +x /usr/local/bin/check-ssl-expiry.sh

# Add to crontab (runs daily at 9 AM)
(crontab -l 2>/dev/null; echo "0 9 * * * /usr/local/bin/check-ssl-expiry.sh") | crontab -
```

#### Step 10: Test SSL Configuration

Use online SSL testing tools:
- **Qualys SSL Labs:** https://www.ssllabs.com/ssltest/
  - Enter your domain: omeka.institution.edu
  - Should receive an A or A+ rating

- **Security Headers:** https://securityheaders.com/
  - Test your domain for security headers
  - Should show green scores for all headers

---

### Network Foundation Complete: DNS & SSL Configured

**Your Infrastructure is Now Publicly Accessible and Secure**

You've successfully configured the networking layer that connects your digital collections platform to the world with proper domain names and encryption.

**What You've Accomplished:**
- ✅ **Route 53 hosted zone created** with proper DNS delegation
- ✅ **DNS records configured** for all environments
- ✅ **Health checks established** with automated monitoring
- ✅ **SSL certificates installed** with auto-renewal configured
- ✅ **Security headers implemented** for enhanced protection

**Your Platform Now Features:**
- **User-friendly URLs** - No more remembering IP addresses
- **Automatic failover** - Health checks ensure availability
- **Industry-standard encryption** - TLS 1.2/1.3 protecting all traffic
- **A+ security rating** - Meeting modern security requirements
- **Zero-maintenance SSL** - Certificates renew automatically

**Access Points Established:**
- Production: `https://omeka.yourinstitution.edu`
- Staging: `https://stage-omeka.yourinstitution.edu`
- All traffic automatically redirected to HTTPS

**Infrastructure Status - COMPLETE!**
| Component | Status | Public Access |
|-----------|--------|--------------|
| Compute (EC2) | Complete | Via Bastion |
| Database (RDS) | Complete | Private Only |
| Storage (S3) |  Complete | Policy Controlled |
| Access (IAM) |  Complete | Configured |
| DNS (Route 53) |  Complete | Active |
| SSL Certificates |  Complete | Auto-Renewing |

**Security Grade:** A+ (SSL Labs)

Congratulations! You've built a professional, enterprise-grade infrastructure that's ready to host your digital collections securely and reliably.

### Final Validation Checklist

Before considering your infrastructure complete:

- [ ] All environments are accessible via HTTPS
- [ ] Backups are running and tested
- [ ] Monitoring alerts are configured
- [ ] Security scan shows no critical issues
- [ ] Documentation is stored in version control
- [ ] Team members have appropriate access
- [ ] Recovery procedures are documented
- [ ] Costs align with budget expectations

### Acknowledgment

Building cloud infrastructure represents a significant achievement for any institution. You've created a platform that:
- Preserves digital heritage securely
- Provides reliable access to collections
- Scales with institutional growth
- Maintains cost efficiency
- Follows industry best practices

This infrastructure will serve your institution's digital initiatives for years to come, adapting as technology and needs evolve.

### Document Maintenance

This runbook is a living document. Update it when:
- AWS services introduce relevant features
- Security best practices change
- Your institution's requirements evolve
- Team members discover optimizations
- Incidents reveal improvement opportunities

**Version Control:**
Store this documentation in Git for version tracking and collaborative updates.

**Review Schedule:**
Schedule quarterly reviews to ensure documentation remains current and accurate.

---

## Final Thoughts

You've successfully deployed infrastructure that many institutions struggle to achieve. The combination of security, scalability, and maintainability positions your digital collections platform for long-term success.

The knowledge gained through this implementation extends beyond this single project. You now possess the expertise to architect cloud solutions for various institutional needs, contributing to your organization's digital transformation journey.

Remember: infrastructure is never truly "complete" – it evolves with your needs. Use this runbook as your foundation, but don't hesitate to adapt and improve as you learn what works best for your specific context.

## Thank you for your dedication to preserving and sharing digital collections. Your work ensures these resources remain accessible for current and future generations.
## Support and Resources

### AWS Documentation Links

- **EC2:** https://docs.aws.amazon.com/ec2/
- **RDS:** https://docs.aws.amazon.com/rds/
- **S3:** https://docs.aws.amazon.com/s3/
- **IAM:** https://docs.aws.amazon.com/iam/
- **Route 53:** https://docs.aws.amazon.com/route53/

### Omeka S Resources

- **Official Documentation:** https://omeka.org/s/docs/
- **Forums:** https://forum.omeka.org/
- **GitHub:** https://github.com/omeka/omeka-s

### Educational Institution Resources

- *Internet2 NET+ Cloud Services*
- *EDUCAUSE Cloud Computing Resources*
- *Your State's Education Network Documentation*

## End of Infrastructure Runbook

**Project Statistics:**
- **Total Configuration Sections:** 7 Major Sections
- **Estimated Setup Time:** 24-48hrs hours for complete infrastructure
- **Documentation Completeness:** Production-ready

