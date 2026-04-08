
# Zip Importer Errors

This document captures known issues and failure patterns related to the ZipImport module and provides guidance for diagnosis and resolution. The full troubleshooting will be in the module's repo; below are the most common errors.

---

## Overview

The ZipImport module relies on structured ZIP archives containing:
- Metadata (CSV)
- Associated media files

It inherits behavior from the Omeka S CSV Import module and is sensitive to both file structure and server configuration.

---

## Common Failure Modes

### 1. Images Not Attached to Items

**Cause**
Mismatch between CSV file references and actual file names.

**Resolution**
- Verify exact filename matches
- Confirm case sensitivity
- Ensure files exist at expected relative paths

---

### 2. Images Skipped Without Errors

**Cause**
Silent failures due to:
- PHP memory limits
- Execution timeouts
- File size limits

**Resolution**
- Increase PHP limits:

```ini
memory_limit = 512M
max_execution_time = 300
upload_max_filesize = 128M
post_max_size = 128M
```

- Re-run import and monitor logs

---

### 3. Partial Imports

**Cause**
Process timeout or interrupted execution.

**Resolution**

-   Break large ZIP files into smaller batches
-   Run imports during low server load periods
-   Monitor CPU and memory usage

---

### 4. CSV Not Recognized

**Cause**
Incorrect encoding or delimiter issues.

**Resolution**

-   Ensure CSV is UTF-8 encoded
-   Verify delimiter matches expected format (comma-separated)

---

### 5. Import Completes but Items Missing Media

**Cause**
Misnaming or non visible char mismatch.

**Resolution**

-   Ensure consistant naming
-   Use copy paste to ensure accurate naming

---

Logging and Validation
----------------------

### Recommended Enhancements

To improve traceability:

-   Log all skipped files during import
-   Generate checksum manifest for uploaded assets
-   Compare expected vs actual imported file counts

---

File Structure Requirements
---------------------------

Example ZIP structure:

import.zip\
│\
├── metadata.csv\
├── image1.jpg\
├── image2.jpg

- Or for nesting (adding multpile files to one item):

import.zip\
│\
├── metadata.csv\
├── image1.jpg\
├── image2/\
    ├── image.jpg\
    ├── document.pdf


---

Best Practices
--------------

-   Validate CSV before import
-   Test with small dataset first
-   Maintain consistent naming conventions
-   Avoid spaces or special characters in filenames

---

Summary
-------

ZipImport failures are typically caused by:

-   File naming mismatches
-   PHP configuration limits
-   Improper ZIP structure
-   Encoding inconsistencies

Systematic validation prior to import significantly reduces failure rates.
