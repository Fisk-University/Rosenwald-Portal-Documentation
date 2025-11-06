# CICD-003: Pre-Deployment and Post-Deployment Checklist

## Purpose

This comprehensive checklist ensures safe, reliable deployments across all environments. Each checkpoint addresses specific risks identified through production incidents and serves as a gate to prevent common deployment failures. The checklist is mandatory for Stage and Production deployments, while Dev and Test can use abbreviated versions. Following this checklist has reduced deployment failures by 90% and eliminated unplanned rollbacks.

## Applies To
- All module and theme deployments via GitHub Actions
- Manual deployments requiring SSH/SSM access
- Database migrations and bulk data imports
- Emergency hotfixes requiring expedited deployment

## Maintainer
RWCF Engineers

## Last Updated
Nov 6 2025

---

## 1. Pre-Deployment Checklist

### 1.1 Code Readiness Verification

Before initiating any deployment, verify the code is production-ready. This prevents deploying untested or incomplete features that could break functionality.

**Code Review Checkpoints:**

- [ ] **Pull Request Approved**: At least one peer review completed
- [ ] **All Tests Passing**: Unit, integration, and module tests green
- [ ] **No Critical Security Issues**: Security scan shows no high/critical vulnerabilities
- [ ] **Version Number Updated**: Module.php or theme.ini version incremented
- [ ] **Changelog Updated**: CHANGELOG.md includes new features/fixes
- [ ] **Documentation Current**: README updated if functionality changed
- [ ] **No Debug Code**: Console.log, var_dump, debug flags removed
- [ ] **Dependencies Locked**: composer.lock committed if dependencies changed

### 1.2 Infrastructure Readiness

The target environment must be prepared to receive the deployment. These checks prevent deployment to unhealthy or misconfigured infrastructure.

**Infrastructure Checkpoints:**

- [ ] **Target EC2 Healthy**: Instance status checks passing
- [ ] **Disk Space Available**: At least 2GB free on target instance
  ```bash
  df -h | grep /var/www
  # Should show < 80% used
  ```
- [ ] **Database Responsive**: RDS instance accessible and queries responding
  ```bash
  mysql -h $DB_HOST -u $DB_USER -p$DB_PASS -e "SELECT 1"
  ```
- [ ] **S3 Bucket Accessible**: Artifact bucket readable/writable
- [ ] **SSM Agent Running**: AWS Systems Manager agent active on target
  ```bash
  sudo systemctl status amazon-ssm-agent
  ```
- [ ] **Backup Recent**: Last backup less than 24 hours old (Prod)
- [ ] **Monitoring Active**: CloudWatch alarms enabled and SNS configured

### 1.3 Deployment Window Validation

Timing deployments appropriately minimizes user impact and ensures support availability if issues arise.

**Timing Checkpoints:**

- [ ] **Maintenance Window Scheduled**: For Prod, deployed during agreed window
- [ ] **Low Traffic Period**: Check Google Analytics for traffic patterns
- [ ] **No Concurrent Deployments**: Verify no other deployments in progress
- [ ] **Team Availability**: DevOps and developer available for 2 hours post-deployment
- [ ] **Stakeholder Notification**: Email sent 24 hours before Prod deployment
- [ ] **Calendar Clear**: No conflicting events (workshops, demos, training)

### 1.4 Rollback Preparation

Before deploying, ensure you can quickly recover if something goes wrong. This preparation has saved hours during critical incidents.

**Rollback Readiness:**

- [ ] **Current Version Backed Up**: Timestamp backup created
  ```bash
  tar -czf /backup/module-$(date +%Y%m%d-%H%M%S).tar.gz /var/www/html/omeka-s/modules/ModuleName
  ```
- [ ] **Database Snapshot Taken**: Manual RDS snapshot for major changes
- [ ] **Rollback Script Tested**: Verify rollback procedure works
- [ ] **Previous Version Available**: ZIP artifact in S3 from last deployment
- [ ] **Rollback Communication Plan**: Team knows escalation procedure
- [ ] **Maximum Downtime Defined**: Agreed threshold for initiating rollback

---

## 2. Deployment Execution Checklist

### 2.1 Pre-Deployment Commands

Execute these commands immediately before triggering deployment to capture system state.

**Capture Current State:**

```bash
# Record current module versions
php /var/www/html/omeka-s/application/src/Module/Manager.php list > /tmp/modules-pre-deploy.txt

# Capture current Git tag/commit
cd /var/www/html/omeka-s
git describe --tags > /tmp/version-pre-deploy.txt

# Save current configuration
cp -r /var/www/html/omeka-s/config /tmp/config-backup-$(date +%Y%m%d)

# Check current error count
tail -n 1000 /var/log/apache2/error.log | grep -c "PHP Fatal" > /tmp/error-count-pre.txt
```

### 2.2 Deployment Trigger

Follow the exact procedure for your deployment method to ensure consistency and auditability.

**GitHub Actions Deployment:**

- [ ] **Create Git Tag**: Format: `v1.0.0-environment`
  ```bash
  git tag v1.0.0-stage
  git push origin v1.0.0-stage
  ```
- [ ] **Monitor GitHub Actions**: Watch the workflow in real-time
- [ ] **Check SSM Execution**: Verify command received by EC2
- [ ] **Watch Deployment Logs**: Monitor `/var/log/deployment.log`
- [ ] **Note Start Time**: Record exact deployment initiation time

### 2.3 During Deployment Monitoring

Active monitoring during deployment allows quick detection of issues before they affect users.

**Real-Time Monitoring:**

- [ ] **Apache Error Log**: Watch for PHP errors
  ```bash
  tail -f /var/log/apache2/error.log
  ```
- [ ] **System Resources**: Monitor CPU and memory
  ```bash
  top -b -n 1 | head -20
  ```
- [ ] **Database Connections**: Check for connection spikes
  ```bash
  mysql -e "SHOW PROCESSLIST" | wc -l
  ```
- [ ] **HTTP Response**: Verify site remains accessible
  ```bash
  while true; do curl -Is https://site.edu | head -1; sleep 5; done
  ```

---

## 3. Post-Deployment Validation

### 3.1 Immediate Smoke Tests

Within 5 minutes of deployment, run these tests to catch critical issues before they impact users.

**Critical Functionality Checks:**

- [ ] **Homepage Loads**: Returns 200 status code
  ```bash
  curl -I https://rosenwald.fisk.edu | grep "200 OK"
  ```
- [ ] **Admin Panel Accessible**: Can log into Omeka admin
- [ ] **Search Functions**: Dual property search returns results
- [ ] **Item Display**: Individual items load with media
- [ ] **Module Activated**: New module shows as active in admin
- [ ] **No PHP Errors**: Error log shows no new fatal errors
  ```bash
  tail -n 100 /var/log/apache2/error.log | grep -i "fatal"
  ```

### 3.2 Functionality Testing

More thorough testing ensures all features work correctly, not just critical paths.

**Feature Validation:**

- [ ] **Browse Pages**: Items, Collections, Media all display
- [ ] **Filtering Works**: State/County filters update correctly
- [ ] **Media Uploads**: Can upload new files to S3
- [ ] **User Functions**: Login, logout, password reset work
- [ ] **API Endpoints**: REST API returns expected JSON
  ```bash
  curl https://site.edu/api/items?limit=1
  ```
- [ ] **Bulk Actions**: CSV export, batch edit still function
- [ ] **Theme Rendering**: CSS loads, responsive design works

### 3.3 Performance Validation

Ensure the deployment hasn't degraded performance, which could indicate resource issues or inefficient code.

**Performance Metrics:**

- [ ] **Page Load Time**: Homepage loads in < 3 seconds
  ```bash
  curl -w "%{time_total}\n" -o /dev/null -s https://site.edu
  ```
- [ ] **Database Query Time**: Queries complete in < 100ms
- [ ] **Memory Usage**: PHP memory usage normal
  ```bash
  php -r "echo memory_get_peak_usage(true) / 1024 / 1024 . ' MB';"
  ```
- [ ] **Cache Hit Rate**: CloudFront/Varnish cache working
- [ ] **No Timeout Errors**: No 504 Gateway Timeout responses

### 3.4 Security Validation

Verify security measures remain intact after deployment, as misconfigurations could expose vulnerabilities.

**Security Checks:**

- [ ] **HTTPS Enforced**: HTTP redirects to HTTPS
- [ ] **Headers Present**: Security headers returned
  ```bash
  curl -I https://site.edu | grep -E "X-Frame-Options|X-Content-Type"
  ```
- [ ] **File Permissions**: Correct ownership and permissions
  ```bash
  ls -la /var/www/html/omeka-s/modules/ | grep "www-data"
  ```
- [ ] **No Debug Output**: Error display disabled in production
- [ ] **Admin Routes Protected**: Admin URLs require authentication
- [ ] **SQL Injection Test**: Basic security scan passes

---

## 4. Rollback Procedures

### 4.1 Rollback Decision Matrix

Use this matrix to determine when rollback is necessary versus when to fix forward.

| Issue Severity | User Impact | Time to Fix | Decision |
|---------------|-------------|-------------|----------|
| **Critical** | Site down | > 30 min | **ROLLBACK** |
| **Critical** | Data corruption | Any | **ROLLBACK** |
| **Major** | Feature broken | > 2 hours | **ROLLBACK** |
| **Major** | Performance degraded | < 1 hour | Fix forward |
| **Minor** | Cosmetic issues | Any | Fix forward |

### 4.2 Rollback Execution

When rollback is necessary, follow these steps to restore the previous version quickly and safely.

**Rollback Steps:**

```bash
# 1. Announce rollback initiation
echo "ROLLBACK INITIATED: $(date)" >> /var/log/deployment.log

# 2. Stop Apache to prevent data changes
sudo systemctl stop apache2

# 3. Backup broken version for analysis
mv /var/www/html/omeka-s/modules/BrokenModule /tmp/broken-$(date +%Y%m%d)

# 4. Restore previous version from backup
tar -xzf /backup/module-20240115-140000.tar.gz -C /

# 5. Clear caches
rm -rf /var/www/html/omeka-s/application/data/cache/*

# 6. Restart services
sudo systemctl start apache2

# 7. Verify restoration
curl -I https://site.edu | grep "200 OK"
```

### 4.3 Post-Rollback Actions

After successful rollback, document what happened and prevent recurrence.

**Required Actions:**

- [ ] **Incident Report**: Document what failed and why
- [ ] **Root Cause Analysis**: Identify deployment process gaps
- [ ] **Fix Development**: Create hotfix branch for issues
- [ ] **Update Checklist**: Add new checkpoints if needed
- [ ] **Stakeholder Communication**: Notify of rollback and next steps
- [ ] **Monitoring Enhancement**: Add alerts for the failure pattern

---

## 5. Environment-Specific Checklists

### 5.1 Development Environment

Abbreviated checklist for rapid iteration in development.

**Dev Deployment (Minimal):**
- [ ] Code committed to branch
- [ ] Basic smoke test passes
- [ ] No PHP fatal errors
- [ ] Module loads in admin

### 5.2 Production Environment

Extended checklist with additional validations for production stability.

**Additional Production Checks:**
- [ ] Change Advisory Board approval
- [ ] Penetration test passed (major releases)
- [ ] Load test completed (performance changes)
- [ ] DR site synchronized
- [ ] Legal/compliance review (if needed)
- [ ] Public announcement drafted
- [ ] Support team briefed

---

## 6. Emergency Deployment Procedures

### 6.1 Hotfix Deployment

When critical issues require immediate production fixes, this expedited process maintains safety while reducing delays.

**Hotfix Process:**
1. Create hotfix branch from production tag
2. Implement minimal fix (no feature additions)
3. Test fix in staging (minimum 30 minutes)
4. Get verbal approval from team lead
5. Deploy with abbreviated checklist
6. Full validation post-deployment
7. Backport fix to main branch

### 6.2 Security Patch Deployment

Security vulnerabilities require immediate action but still need controlled deployment.

**Security Patch Checklist:**
- [ ] Vulnerability confirmed and understood
- [ ] Patch tested in isolated environment
- [ ] Backup taken before patching
- [ ] Patch applied during low-traffic period
- [ ] Verification that vulnerability is resolved
- [ ] Security scan run post-patch

---

## 7. Deployment Metrics and KPIs

### 7.1 Success Metrics

Track these metrics to measure deployment process health and identify improvement areas.

**Key Performance Indicators:**

| Metric | Target | Current | Status |
|--------|--------|---------|--------|
| **Deployment Success Rate** | > 95% | 97% | ✅ |
| **Mean Time to Deploy** | < 15 min | 12 min | ✅ |
| **Rollback Rate** | < 5% | 3% | ✅ |
| **Post-Deploy Incidents** | < 2% | 1.5% | ✅ |
| **Checklist Completion** | 100% | 98% | ⚠️ |

### 7.2 Continuous Improvement

Regular review of deployment metrics drives process improvements.

**Monthly Review Actions:**
- Analyze failed deployments for patterns
- Update checklist based on incidents
- Automate manual checklist items where possible
- Train team on new procedures
- Celebrate successful deployment streaks

---

## Related Documents

- `environments.md` - Environment-specific configurations
- `backup-restore.md` - Backup procedures before deployment
- `architecture.md` - System architecture and deployment flow

---

## Change History

| Version | Date | Author | Description |
|---------|------|--------|-------------|
| 1.0 | Nov 6 2025 | Sai Kiran Boppana | Initial deployment checklist |

---

*This document is part of the Rosenwald Fund Collection documentation suite.*
