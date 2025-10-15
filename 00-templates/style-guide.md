# RWCF Documentation Style Guide

## Purpose
This style guide ensures consistency, clarity, and professionalism across all RWCF documentation. All contributors should reference this guide when creating or updating documentation.

---

## 1. Writing Tone & Voice

### Tone
- **Technical but accessible** - Write for engineers with varying experience levels
- **Direct and concise** - Get to the point without unnecessary words
- **Professional yet approachable** - Avoid overly formal language
- **Action-oriented** - Use imperative mood for instructions

### Examples
 **Good**: "Configure the Apache server by editing the config file"  
 **Avoid**: "You might want to consider possibly configuring the Apache server"

 **Good**: "Run the following command to restart the service"  
 **Avoid**: "The command that should be run is as follows"

---

## 2. File Naming Conventions

### General Rules
- Use **lowercase** only
- Use **hyphens** to separate words (kebab-case)
- Be **descriptive** but concise
- Include document type when relevant (e.g., `-sop`, `-guide`, `-blueprint`)
- Use **numeric prefixes** for ordered content (e.g., `01-`, `02-`)

### Examples
```
 Good:
- aws-ec2-setup-sop.md
- ssl-configuration-guide.md
- 01-initial-server-setup.md
- troubleshooting-apache-errors.md

 Avoid:
- AWS_EC2_Setup.md (wrong case, underscores)
- ssl.md (too vague)
- NewServerSetupInstructions.md (camelCase)
- apache-troubleshooting-guide-for-common-errors-final-v2.md (too long)
```

---

## 3. Document Structure

### Standard Headers
Every document should include:
```markdown
# [Document Title]

## Purpose
[One paragraph explaining why this document exists]

## Scope
[What this covers and what it doesn't]

## Last Updated
[YYYY-MM-DD]
```

### Section Hierarchy
- Use `#` for main title (one per document)
- Use `##` for major sections
- Use `###` for subsections
- Avoid going deeper than `####`

---

## 4. Code Formatting

### Inline Code
Use backticks for commands, file names, and short code references:
- `sudo systemctl restart apache2`
- Edit the `/etc/apache2/apache2.conf` file
- Set the variable `ENVIRONMENT=production`

### Code Blocks
Always specify the language for syntax highlighting:

```bash
# Shell/Bash commands
sudo apt update
sudo apt install apache2
```

```yaml
# YAML configuration
version: '3.8'
services:
  web:
    image: nginx:latest
```

```json
# JSON data
{
  "name": "rwcf-project",
  "version": "1.0.0"
}
```

### Command Output
When showing expected output, use a comment or separate block:
```bash
$ systemctl status apache2
# Output:
# ● apache2.service - The Apache HTTP Server
#     Loaded: loaded (/lib/systemd/system/apache2.service; enabled)
#     Active: active (running)
```

---

## 5. Formatting Standards

### Lists
- Use `-` for unordered lists (not `*` or `+`)
- Use `1.` for ordered lists (let Markdown auto-number)
- Maintain consistent indentation (2 spaces)

### Tables
Use tables for structured data:
```markdown
| Column 1 | Column 2 | Column 3 |
|----------|----------|----------|
| Data 1   | Data 2   | Data 3   |
```

### Emphasis
- **Bold** for important terms or warnings: `**Important**`
- *Italics* for emphasis or new terms: `*Note*`
- Avoid underlining or ALL CAPS

### Warnings and Notes
Use blockquotes with labels:
```markdown
> **Warning**: This action cannot be undone

> **Note**: Requires sudo privileges

> **Tip**: Use tab completion to avoid typos
```

---

## 6. Language & Terminology

### Technical Terms
- Define acronyms on first use: "Amazon Web Services (AWS)"
- Use consistent terminology throughout all docs
- Maintain a glossary for project-specific terms

### Common Terms (Use Consistently)
- "repository" not "repo" (in formal docs)
- "configure" not "config"
- "administrator" not "admin" (in formal docs)
- "database" not "DB" (spell out in documentation)

### Active Voice
 **Use**: "The script creates a backup"  
 **Avoid**: "A backup is created by the script"

---

## 7. Screenshots & Diagrams

### When to Include
- Complex UI procedures
- Architecture diagrams
- Network topology
- Visual confirmation of successful steps

### Naming Convention
```
images/
├── 01-aws-console-ec2-dashboard.png
├── 02-security-group-configuration.png
└── network-architecture-diagram.svg
```

### Format Requirements
- Use PNG for screenshots
- Use SVG for diagrams when possible
- Keep file sizes reasonable (<500KB for screenshots)
- Add alt text for accessibility

---

## 8. Version Control

### Commit Messages for Docs
```
docs: Add SSL configuration SOP
docs: Update Apache troubleshooting guide
docs: Fix typos in installation guide
fix: Correct command syntax in backup SOP
```

### Branch Naming for Documentation
```
docs/add-ssl-guide
docs/update-troubleshooting
docs/fix-installation-steps
```

---

## 9. Review Checklist

Before submitting documentation:

- [ ] Follows file naming conventions
- [ ] Includes required headers (Purpose, Scope, Last Updated)
- [ ] Uses correct markdown formatting
- [ ] Code blocks have language specified
- [ ] Commands are tested and working
- [ ] Links are valid and use relative paths where appropriate
- [ ] Spell-checked and grammar-checked
- [ ] Technical accuracy verified
- [ ] Follows tone and voice guidelines
- [ ] Images are optimized and named correctly

---

## 10. Common Mistakes to Avoid

1. **Don't assume knowledge** - Link to prerequisites or explain terms
2. **Don't skip validation** - Always include how to verify success
3. **Don't use absolute paths** - Use relative paths or variables
4. **Don't forget rollback** - Include recovery steps for risky operations
5. **Don't mix concerns** - Keep SOPs focused on single procedures
6. **Don't use outdated commands** - Verify compatibility with current versions
7. **Don't ignore errors** - Document common errors and solutions

---

## Quick Reference

### Document Types & Their Purpose
- **SOP** (Standard Operating Procedure): Step-by-step instructions
- **Guide**: Conceptual explanation with some procedures  
- **Blueprint**: High-level design or architecture document
- **Runbook**: Emergency response procedures
- **README**: Overview and entry point for a section

### Markdown Quick Reference
```markdown
# H1 Header
## H2 Header
### H3 Header

**bold text**
*italic text*
`inline code`

[Link text](url)
![Alt text](image.png)

- Unordered list
1. Ordered list

> Blockquote

| Table | Header |
|-------|--------|
| Data  | Data   |
```

---

*This style guide is a living document. Submit suggestions for improvements via pull request.*