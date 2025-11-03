# MODULE-001: UnitedSearch Configuration and Search Logic Guide

## Purpose

UnitedSearch extends Omeka-S's basic search capabilities by providing two specialized search blocks that understand relationships between metadata properties. The module was developed specifically for the Rosenwald project's need to search hierarchical geographic data (State → County), but it can handle any paired metadata relationships. Unlike Omeka's default search which treats all fields independently, UnitedSearch understands that certain properties are related and filters them accordingly.

## Applies To
- Omeka-S version 4.0.0 or higher
- Sites with hierarchical or related metadata properties
- Collections requiring filtered search interfaces
- Projects needing dynamic form interactions

## Maintainer
RWCF Development Team

## Last Updated
Nov 3 2025

---

## 1. Prerequisites

UnitedSearch has minimal technical requirements but understanding your metadata structure is crucial. The module needs to analyze your existing items to build relationship maps between properties.

| Requirement | Description | Why It's Needed |
|------------|-------------|-----------------|
| **Omeka-S Version** | 4.0.0 or higher | Uses modern block API features |
| **PHP Version** | PHP 7.4+ with JSON extension | Handles relationship data structures |
| **JavaScript** | Modern browser with JS enabled | Powers dynamic filtering |
| **Existing Items** | At least 10-20 items with metadata | Needs data to build relationships |
| **Metadata Structure** | Consistent property usage | Relationships require consistent data |

---

## 2. Understanding UnitedSearch Components

### 2.1 Two Search Block Types

UnitedSearch provides two distinct search blocks, each serving different use cases. Understanding when to use each block is essential for creating effective search interfaces.

**Item Set | Property Search Block:**
This block searches within a specific collection (item set) using a single property. It's perfect when users know which collection to search but need to filter by a specific attribute. For example, searching for all schools in the "Alabama Collection" built in a specific year. The block presents a dropdown of all unique values for the selected property within that item set.

**Dual Property Search Block:**
This is the flagship feature - it understands relationships between two properties and filters the second based on the first selection. When a user selects "Alabama" in the State dropdown, the County dropdown instantly updates to show only Alabama counties, not all counties in the database. This creates an intuitive search experience that matches how users think about hierarchical data.

### 2.2 How Property Relationships Work

The module's intelligence comes from analyzing your existing data to discover relationships. When you configure a dual property search, UnitedSearch scans all items to find which values of Property B appear with each value of Property A. This creates a relationship map that powers the dynamic filtering.

For example, if your items have State and County metadata, the module builds a map like:
- Alabama → [Madison, Jefferson, Mobile, Montgomery...]
- Georgia → [Fulton, DeKalb, Gwinnett, Cobb...]
- Tennessee → [Davidson, Shelby, Knox, Hamilton...]

This mapping happens automatically - you don't need to define relationships manually. However, this means your data must be consistent. If you sometimes use "AL" and sometimes "Alabama", the module treats them as different values.

---

## 3. Installation Process

### 3.1 Module Installation

UnitedSearch installation is straightforward as it has no external dependencies. The module is self-contained and doesn't require composer or npm packages.

```bash
# Navigate to modules directory
cd /var/www/html/omeka-s/modules

# Download from GitHub
sudo wget https://github.com/Fisk-University/UnitedSearch/archive/refs/heads/CI/CD-Testing.zip

# Extract and rename
sudo unzip CI-CD-Testing.zip
sudo mv UnitedSearch-CI-CD-Testing UnitedSearch

# Set permissions
sudo chown -R www-data:www-data UnitedSearch
sudo chmod -R 755 UnitedSearch
```

After installation, activate the module in Omeka's admin panel. Navigate to Modules, find UnitedSearch in the list, and click Install. No configuration is needed at the module level - all setup happens when adding blocks to pages.

### 3.2 Verifying Installation

To confirm UnitedSearch is working, check that new block types appear when editing pages. Edit any site page and click "Add new block" - you should see "Item Set | Property Search" and "Dual Property Search" in the block list. If these don't appear, check that the module is activated and that you're using Omeka-S 4.0 or higher.

---

## 4. Configuring Search Blocks

### 4.1 Adding Item Set | Property Search

This simpler block is good for learning how UnitedSearch works before tackling dual property search. It creates a search interface for one property within one collection.

**Configuration steps:**

1. **Edit your target page**: Usually the homepage or a dedicated search page
2. **Add new block**: Select "Item Set | Property Search"
3. **Choose Item Set**: Select which collection to search within
4. **Select Property**: Choose the metadata field users will search by
5. **Customize labels**: Optionally change the dropdown label and placeholder text

The block automatically populates the dropdown with all unique values of the selected property found in that item set. If you choose "Year Built" as the property, the dropdown shows every year that appears in your data: 1920, 1921, 1925, etc.

### 4.2 Configuring Dual Property Search

The dual property search requires more thought about your metadata relationships. You're defining how two properties relate to each other across your entire site.

**Key configuration options:**

**First Property (Primary):**
This is what users select first. Choose a property with relatively few unique values (10-50 is ideal). Common choices include State, Category, Department, or Collection Type. This property should logically determine what values make sense for the second property.

**Second Property (Dependent):**
This property's values are filtered based on the first selection. Choose something with many values that logically group under the first property. Common choices include County (under State), Subcategory (under Category), or Building (under Campus).

**Join Type (AND vs OR):**
- **AND (Restrictive)**: Shows only values that actually appear together in your data. If no items have "Alabama" + "Fulton County" together, Fulton won't appear when Alabama is selected.
- **OR (Permissive)**: Shows all values regardless of relationships. Useful when your data is incomplete but you still want to show all possibilities.

**Labels and Placeholders:**
Customize the text users see. Instead of technical property names like "dcterms:spatial", use friendly labels like "Select a State" and "Choose a County".

---

## 5. Search Indexing Logic

### 5.1 How UnitedSearch Builds Its Index

Unlike traditional search engines that build persistent indexes, UnitedSearch uses dynamic indexing. When a page with a search block loads, the module queries the database to build fresh relationship data. This approach ensures searches always reflect current data but requires efficient queries for good performance.

The indexing process follows these steps:
1. Query all items in the site (or item set if specified)
2. Extract unique values for the configured properties
3. Build a relationship matrix showing which values appear together
4. Cache relationships in memory for the page session
5. Send simplified data to the browser for JavaScript filtering

This happens every page load, which means new items are immediately searchable. However, it also means very large collections (10,000+ items) might experience slower page loads.

### 5.2 Database Query Optimization

UnitedSearch uses several strategies to maintain performance with large datasets. Understanding these helps you structure your data for optimal search performance.

**Efficient SQL Queries:**
The module uses indexed columns and minimizes JOIN operations. It queries only the specific properties needed rather than loading full item records. For dual property search, it uses a single optimized query to find relationships rather than multiple queries.

**Property Selection Matters:**
Properties stored as single values search faster than multi-value properties. If an item has multiple counties, the relationship building becomes more complex. When possible, structure data to have single values for search properties.

**Item Set Filtering:**
When using Item Set | Property Search, the module only queries items in that set, dramatically reducing the dataset size. This is why item set searches are faster than site-wide dual property searches.

### 5.3 Client-Side Filtering Logic

After the server builds relationship data, JavaScript handles the interactive filtering. Understanding this helps when customizing or troubleshooting the interface.

The browser receives a JSON structure mapping first property values to arrays of related second property values. When users select from the first dropdown, JavaScript instantly filters the second dropdown without server communication. This makes the interface responsive even on slow connections.

```javascript
// Simplified view of the data structure
relationshipMap = {
  "Alabama": ["Madison", "Jefferson", "Mobile"],
  "Georgia": ["Fulton", "DeKalb", "Gwinnett"],
  "Tennessee": ["Davidson", "Shelby", "Knox"]
}

// When user selects "Alabama", JavaScript shows only its counties
```

---

## 6. Advanced Configuration

### 6.1 Handling Special Data Scenarios

Real-world data is messy. UnitedSearch includes strategies for common data quality issues that would otherwise break search functionality.

**Inconsistent Property Values:**
If your data has variations like "AL", "Alabama", and "alabama", the module treats these as different values. Before implementing UnitedSearch, standardize your data using Omeka's batch edit features. Alternatively, use the module for fields with controlled vocabularies where consistency is guaranteed.

**Multi-Value Properties:**
When items have multiple values for a property (e.g., an item spanning multiple counties), UnitedSearch includes that item in searches for any of its values. The relationship builder understands multi-value properties and creates appropriate mappings.

**Missing or Null Values:**
Items lacking values for search properties are excluded from dropdowns but still appear in results if other criteria match. The module gracefully handles missing data without breaking the search interface.

### 6.2 Performance Tuning

For sites with thousands of items, search block performance becomes important. These optimizations help maintain responsive searches.

**Limit Relationship Depth:**
Instead of mapping every possible combination, consider breaking complex searches into steps. Rather than State → County → City in one block, use two blocks: State → County on one page, then County → City on the results page.

**Use Item Set Constraints:**
Whenever possible, use Item Set | Property Search instead of site-wide dual property search. Smaller datasets mean faster indexing. Consider organizing items into regional item sets to reduce query scope.

**Cache Strategy:**
While UnitedSearch doesn't include built-in caching, you can improve performance with page-level caching in Omeka or server-side caching with Redis/Memcached. Cache search pages for 5-10 minutes to reduce database queries while keeping results relatively fresh.

---

## 7. Customization Options

### 7.1 Styling Search Blocks

UnitedSearch provides semantic HTML with predictable CSS classes for styling. The module doesn't impose visual design, letting your theme control appearance.

**CSS classes for targeting:**
```css
/* Container for entire search block */
.united-search-block {
  /* Your styles */
}

/* Individual property dropdowns */
.property-select {
  /* Dropdown styling */
}

/* Search button */
.search-submit {
  /* Button styling */
}

/* Loading states */
.united-search-loading {
  /* Show spinner or loading message */
}
```

### 7.2 JavaScript Events and Hooks

The module triggers JavaScript events you can listen to for custom functionality. This allows adding analytics, loading indicators, or custom validation without modifying module code.

```javascript
// Listen for search events
document.addEventListener('unitedSearchSubmit', function(e) {
    // Track search in analytics
    console.log('Search performed:', e.detail);
});

// Listen for property changes
document.addEventListener('unitedSearchPropertyChange', function(e) {
    // Show loading spinner
    document.querySelector('.search-container').classList.add('loading');
});
```

---

## 8. Troubleshooting

### 8.1 Common Issues and Solutions

| Issue | Cause | Solution |
|-------|-------|----------|
| **Dropdowns are empty** | No items have the selected properties | Verify items have metadata for chosen properties |
| **Second dropdown doesn't filter** | JavaScript error or missing relationships | Check browser console for errors, ensure items have both properties |
| **Slow page loads** | Too many items being indexed | Use item set filtering or reduce number of items |
| **Duplicate values in dropdowns** | Inconsistent data entry | Standardize metadata values using batch edit |
| **Search returns no results** | Mismatched property configuration | Verify search form properties match item metadata structure |

### 8.2 Debugging Search Indexing

When searches don't work as expected, understanding what data the module sees helps diagnose issues. Enable Omeka's development mode to see detailed logs of search indexing.

In `config/local.config.php`:
```php
'logger' => [
    'log' => true,
    'priority' => \Laminas\Log\Logger::DEBUG,
],
```

Check logs at `/logs/application.log` to see:
- How many items are being indexed
- Which property values are found
- Relationship mappings being created
- Query performance timing

### 8.3 Data Quality Checks

Before implementing UnitedSearch, audit your metadata quality. The module works best with clean, consistent data.

**Run these checks:**
1. List all unique values for search properties - look for variations
2. Count items missing search properties - decide if this is acceptable
3. Verify relationship logic - manually check that mapped relationships make sense
4. Test with sample searches - ensure results match expectations

---

## 9. Integration with Other Modules

### 9.1 Working with Advanced Search

UnitedSearch complements rather than replaces Omeka's Advanced Search. Use UnitedSearch for guided searches where users select from known values, and Advanced Search for open-ended queries where users type search terms.

A typical search strategy might include:
- UnitedSearch on homepage for casual users
- Advanced Search link for researchers
- Both options on a dedicated search page

### 9.2 CSV Import Considerations

When bulk importing items via CSV Import or ZipImport, ensure your CSV columns match the properties configured in UnitedSearch. Consistency is crucial - if UnitedSearch expects "dcterms:spatial" for geographic data, your CSV must use exactly that column header.

---

## 10. Best Practices

### 10.1 Planning Search Properties

Before configuring UnitedSearch, map out your search strategy:
1. Identify the most common search patterns from user research
2. Choose properties with controlled vocabularies when possible
3. Ensure selected properties have good data coverage
4. Test relationships with a subset of data first

### 10.2 User Experience Guidelines

- Place search blocks prominently on homepage or dedicated search page
- Provide clear labels that match user terminology
- Include help text explaining what each dropdown contains
- Show result counts after searching
- Provide alternative search methods for edge cases

---

## Change History

| Version | Date | Author | Description |
|---------|------|--------|-------------|
| 1.0 | Nov 3 2025 | Sai Kiran Boppana| Initial UnitedSearch documentation |

---

*This document is part of the Rosenwald-Fund Collection documentation suite.*
