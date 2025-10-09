# SOP Template

> **Note**: This template serves as a reference standard. Not all sections are required for every SOP. Use the sections that are relevant to your specific procedure.

---

## [SOP-ID]: [Title of Standard Operating Procedure]

### Purpose
Explain why this procedure exists — e.g., to standardize server builds, reduce risk, or provide a consistent approach to X.

### Applies To
List the environments, systems, or roles this SOP applies to (e.g., "All AWS EC2 instances in dev, stage, prod").

### Maintainer
[Name]

### Last Updated
YYYY-MM-DD

---

## 1. Prerequisites

| Requirement | Description |
|------------|-------------|
| **Access Level** | (e.g., SSH + sudo privileges, AWS Console access) |
| **Dependencies** | (e.g., RDS instance configured, domain name provisioned) |
| **Tools** | (e.g., AWS CLI v2, Git, nano, Certbot) |
| **Data Needed** | (e.g., DB credentials, IP addresses, SSL certs) |

---

## 2. Procedure

Step-by-step instructions written in imperative style.  
Each step should be verifiable, repeatable, and safe to execute.

### Example Structure

1. **Preparation** – confirm backups, validate environment
2. **Execution** – commands, configuration edits, and restarts
3. **Verification** – tests to ensure success
4. **Cleanup** – remove temp files, close sessions

### Example formatting:

```bash
sudo systemctl restart apache2
```

---

## 3. Validation / Verification

Describe how to confirm success and what the expected outputs should be.

| Validation Step | Command / Action | Expected Result |
|----------------|------------------|-----------------|
| Verify Apache status | `sudo systemctl status apache2` | "active (running)" |
| Confirm DNS propagation | `dig ns rosenwald.fisk.edu @8.8.8.8` | Returns AWS nameservers |
| Check SSL | Browser check | Padlock + valid certificate chain |

---

## 4. Rollback Procedure

Define clear steps for reverting safely if the procedure fails.

1. Revert to latest EC2 or RDS snapshot (record snapshot ID)
2. Undo configuration edits (.bak files recommended)
3. Restart affected services and validate recovery

---

## 5. Change History

| Version | Date | Author | Description |
|---------|------|--------|-------------|
| 1.0 | YYYY-MM-DD | [Your Name] | Initial SOP creation |
| 1.1 | YYYY-MM-DD | [Your Name] | Revised after field validation |

---

## 6. Related Documents

* [Runbook Reference]
* [Troubleshooting Guide]
* [Replication Blueprint]

---

## Template Usage Notes

### When to use each section:

- **Prerequisites**: Essential for complex procedures requiring specific access or tools
- **Procedure**: Required for all SOPs - the core content
- **Validation**: Highly recommended for any procedure with testable outcomes
- **Rollback**: Critical for any procedure that modifies production systems
- **Change History**: Important for tracking evolution of critical procedures
- **Related Documents**: Use when procedure is part of larger workflow

### Formatting Guidelines:

- Use code blocks for commands: ` ```bash ... ``` `
- Use tables for structured data
- Bold important warnings or notes
- Keep steps numbered and concise
- Include example outputs where helpful