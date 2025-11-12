# MODULE-01A: DualPropertySearch Setup and Browse Page Integration Guide

## Purpose

DualPropertySearch is the advanced implementation component of UnitedSearch that enables hierarchical property filtering directly on browse pages, not just as search blocks. While UnitedSearch provides the blocks you add to pages, DualPropertySearch integrates this functionality into Omeka's core browsing experience. This means users can filter items while browsing without leaving the results page, creating a seamless research experience especially important for large archival collections.

## Applies To
- Sites requiring persistent filter options on browse pages
- Collections with complex hierarchical metadata relationships
- Projects needing both block-based and integrated search options
- High-traffic sites where browse page optimization is critical

## Maintainer
RWCF Development Team

## Last Updated
Nov 3 2025

---

## 1. Understanding DualPropertySearch vs UnitedSearch

### 1.1 Relationship Between Modules

Many users confuse these modules, so understanding their relationship is crucial. UnitedSearch is the base module providing search block functionality - it's what you install and configure. DualPropertySearch is an extension or enhancement that takes UnitedSearch's capabilities and embeds them into browse pages. Think of UnitedSearch as the engine and DualPropertySearch as the integration that puts that engine into different parts of your site.

The practical difference is where searches happen. UnitedSearch blocks live on specific pages you designate (homepage, search page, etc.) and redirect to results. DualPropertySearch lives on the browse pages themselves, filtering results in real-time without navigation. This creates a more fluid research experience where users can refine their browsing without losing their place.

### 1.2 When to Use Each Approach

**Use UnitedSearch blocks when:**
- You want a prominent search interface on the homepage
- Users should be guided to search before browsing
- You need search as an entry point to collections
- Different pages need different search configurations

**Use DualPropertySearch integration when:**
- Users browse first, then need to filter results
- Research workflows involve iterative refinement
- You want persistent filters across pagination
- Browse pages are the primary interface for finding items

Most sites benefit from both approaches working together. Users can start with a guided search from the homepage (UnitedSearch block) then refine results using browse page filters (DualPropertySearch integration).

---

## 2. Installation and Setup

### 2.1 Prerequisites and Dependencies

DualPropertySearch requires UnitedSearch to be installed first since it extends that module's functionality. The installation process involves both modules working together, so proper setup order matters.

| Requirement | Description | Why It's Essential |
|------------|-------------|-------------------|
| **UnitedSearch Module** | Must be installed and activated first | Provides core search functionality |
| **Omeka-S 4.0+** | Modern version with hook support | Browse page integration needs recent APIs |
| **Theme Compatibility** | Theme must not override browse templates | Integration requires standard browse structure |
| **JavaScript Support** | Modern browser with JS enabled | Dynamic filtering requires client-side code |

### 2.2 Installation Process

Since DualPropertySearch is typically bundled with UnitedSearch or available as an extension, installation involves enabling additional functionality rather than installing a separate module. The exact process depends on your version of UnitedSearch.

```bash
# If DualPropertySearch is a separate module
cd /var/www/html/omeka-s/modules
wget [DualPropertySearch URL]
unzip DualPropertySearch.zip

# If it's part of UnitedSearch, enable in configuration
cd UnitedSearch
# Edit config/module.config.php to enable browse integration
```

After installation, clear Omeka's cache to ensure the new functionality loads properly. The integration happens automatically - there's no separate activation step in the admin panel.

### 2.3 Verifying Installation

To confirm DualPropertySearch is working, navigate to any browse page (`/items/browse` or `/item-sets/browse`). You should see filter dropdowns above the item grid. If filters don't appear, check that both modules are active and that your theme hasn't overridden browse templates in a way that breaks integration.

---

## 3. Browse Page Integration

### 3.1 How Browse Page Injection Works

DualPropertySearch uses Omeka's event system to inject filter controls into browse pages without modifying core files. This approach ensures compatibility with Omeka updates while maintaining flexibility. When a browse page loads, the module listens for the render event, analyzes the current context (what's being browsed), and injects appropriate filter controls before the item grid.

The injection process is intelligent - it only adds filters where they make sense. If browsing a specific collection, filters reflect that collection's metadata. If browsing site-wide, filters cover all available properties. This context awareness prevents irrelevant filter options from appearing.

The module also maintains filter state across pagination. When users move to page 2 of results, their filter selections persist. This requires careful URL parameter management and session handling to create a smooth experience.

### 3.2 Configuring Browse Page Filters

Browse page configuration happens at the site level rather than per-page like search blocks. This ensures consistency across all browsing experiences. Configuration involves selecting which properties appear as filters and in what order.

**Key configuration decisions:**

**Primary Filter Property:**
Choose the main classification for your collection. This appears as the first dropdown and determines what appears in the second. For the Rosenwald project, this is typically "State" since geographical browsing is primary. The property should have limited unique values (under 50) to prevent overwhelming dropdown menus.

**Secondary Filter Property:**
Select the dependent property that gets filtered based on the primary selection. This typically has many more values that logically group under the primary property. For Rosenwald, this is "County" with hundreds of values that organize under states. The relationship between properties must be logical to users.

**Filter Positioning:**
Decide where filters appear relative to other browse page elements. Options typically include above the results grid (most common), in a sidebar (good for multiple filters), or as a collapsible panel (saves space on mobile). The position affects both usability and theme compatibility.

### 3.3 Advanced Browse Integration Features

DualPropertySearch includes several advanced features that enhance the browse experience beyond simple filtering.

**Results Counter:**
The module dynamically updates result counts as filters are applied. Users see "Showing 27 of 1,432 items" which updates to "Showing 27 of 43 items matching your filters" when filters are active. This immediate feedback helps users understand the impact of their selections.

**Clear Filters Option:**
A prominent "Clear all filters" button lets users reset their browsing without navigating away. This is crucial when users want to broaden their search after drilling down too specifically. The button only appears when filters are active, keeping the interface clean.

**Filter Breadcrumbs:**
Active filters display as removable tags above results: "State: Alabama ×" and "County: Madison ×". Users can remove individual filters by clicking the × without clearing everything. This granular control improves the research experience.

---

## 4. Configuration Examples

### 4.1 Basic State/County Configuration

This is the most common configuration for geographical collections like the Rosenwald project. It creates an intuitive browsing experience where users narrow from state to specific counties.

**Site settings configuration:**
```php
// In site settings or configuration file
'dualPropertySearch' => [
    'enabled' => true,
    'primaryProperty' => 'dcterms:spatial.state',
    'secondaryProperty' => 'dcterms:spatial.county',
    'position' => 'above-results',
    'showCounts' => true,
    'allowClear' => true
]
```

**How it works in practice:**
1. User visits `/items/browse` and sees all items
2. First dropdown shows all states in the collection
3. User selects "Alabama"
4. Page refreshes showing only Alabama items
5. Second dropdown now shows only Alabama counties
6. User selects "Madison County"
7. Results narrow to items from Madison County, Alabama

### 4.2 Category/Subcategory Configuration

For collections organized by topic rather than geography, configure hierarchical subject browsing. This works well for museum collections, document archives, or photo collections with thematic organization.

```php
'dualPropertySearch' => [
    'primaryProperty' => 'dcterms:type',
    'secondaryProperty' => 'dcterms:subject',
    'labels' => [
        'primary' => 'Document Type',
        'secondary' => 'Specific Topic'
    ]
]
```

This creates browsing where users might select "Photographs" then see subcategories like "Buildings", "People", "Events" relevant to photographs only.

### 4.3 Multi-Level Configuration

Some collections need more than two levels of filtering. While DualPropertySearch handles two properties, you can chain multiple instances for deeper hierarchies.

```php
'dualPropertySearch' => [
    'filters' => [
        [
            'primary' => 'dcterms:spatial.state',
            'secondary' => 'dcterms:spatial.county'
        ],
        [
            'primary' => 'dcterms:date.decade',
            'secondary' => 'dcterms:date.year'
        ]
    ]
]
```

This creates two filter sets: geographical (State/County) and temporal (Decade/Year). Users can apply both simultaneously for precise browsing.

---

## 5. Performance Optimization

### 5.1 Caching Strategies

Browse pages with DualPropertySearch can be resource-intensive since they query relationships on every page load. Implementing proper caching dramatically improves performance for high-traffic sites.

**Query Result Caching:**
The module caches relationship data between properties for a configurable duration. If the relationship between States and Counties doesn't change often, cache it for hours or days. This reduces database queries from hundreds to one per cache period.

**Page-Level Caching:**
Consider caching entire browse pages with common filter combinations. If many users browse "Alabama > Madison County", cache that specific view. Use URL parameters as cache keys to maintain different cached versions for different filter combinations.

**Client-Side Caching:**
The JavaScript that powers dynamic filtering can cache relationship data in browser localStorage. This means returning visitors get instant filter updates without server communication. Balance cache duration with data freshness requirements.

### 5.2 Database Indexing

Proper database indexing is crucial for DualPropertySearch performance. The module performs complex queries joining items with multiple property values. Without appropriate indexes, these queries can take seconds on large collections.

**Essential indexes to create:**
```sql
-- Index for property value lookups
CREATE INDEX idx_property_value ON value(property_id, value_text);

-- Index for item property relationships  
CREATE INDEX idx_item_property ON value(resource_id, property_id);

-- Composite index for filtered queries
CREATE INDEX idx_filtered_browse ON value(property_id, value_text, resource_id);
```

These indexes dramatically speed up the relationship building queries that DualPropertySearch performs. Monitor slow query logs to identify additional indexing opportunities specific to your data.

### 5.3 Progressive Enhancement

Implement progressive enhancement so browse pages remain functional even if JavaScript fails or loads slowly. The basic approach ensures core functionality works without JavaScript, then enhances the experience when JavaScript is available.

**Fallback behavior without JavaScript:**
- Filters submit as regular forms with page refresh
- Both dropdowns show all values (no dynamic filtering)
- Users can still filter, just less conveniently

**Enhanced behavior with JavaScript:**
- Second dropdown updates instantly when first changes
- No page refresh until user clicks "Apply Filters"
- Loading indicators during updates
- Keyboard navigation support

## 5.4 JSON Cache Implementation (Critical Performance Fix)

This optimization reduced page load times from 15+ seconds to under 3 seconds by eliminating repetitive database queries. Instead of building relationship maps on every page load, the module caches the computed relationships as JSON with a time-to-live (TTL) setting.

**The Problem We Solved:**
Before this optimization, every browse page load would query the entire database to build State→County relationships. With thousands of items, this meant analyzing every item's metadata on every page view. If 100 users visited the browse page, the same expensive query ran 100 times, creating the 15-second load times that made the site unusable.

**The JSON Cache Solution:**
The relationship data gets computed once, then stored as a JSON file with a 1-hour TTL. Subsequent page loads read this pre-computed JSON instead of querying the database. When the cache expires after an hour, it rebuilds automatically on the next request. This balanced data freshness with performance.

**Implementation approach:**
```php
// Check if cache exists and is fresh
$cacheFile = '/cache/property-relationships.json';
$cacheAge = time() - filemtime($cacheFile);
$ttl = 3600; // 1 hour in seconds

if (file_exists($cacheFile) && $cacheAge < $ttl) {
    // Use cached data - takes milliseconds
    $relationships = json_decode(file_get_contents($cacheFile), true);
} else {
    // Rebuild cache - happens once per hour
    $relationships = $this->buildRelationships(); // Expensive query
    file_put_contents($cacheFile, json_encode($relationships));
}
```

**Cache Invalidation Strategy:**
- Automatic expiry after 1 hour ensures new items appear
- Manual cache clear button in admin for immediate updates
- Cache clears automatically when items are bulk imported
- Different cache files for different browse contexts (collections vs site-wide)

**Why 1 Hour TTL:**
We chose 1 hour as the spot between performance and data freshness. Most archival collections don't change minute-by-minute, so hour-old data is acceptable. For sites with more frequent updates, you could reduce to 30 minutes. For static collections, increase to 24 hours.

This single optimization made the browse pages usable again and should be implemented on any site with 500+ items using DualPropertySearch.

---

## 6. Troubleshooting Browse Integration

### 6.1 Filters Not Appearing

When filters don't show on browse pages, systematic troubleshooting helps identify the issue quickly.

**Check these in order:**
1. **Module activation**: Confirm both UnitedSearch and DualPropertySearch are active
2. **Theme compatibility**: Temporarily switch to default theme to test
3. **Browse template**: Ensure theme doesn't override `/items/browse` template
4. **JavaScript errors**: Check browser console for error messages
5. **Configuration**: Verify property names match your metadata exactly

### 6.2 Performance Issues

Slow browse pages usually indicate inefficient queries or missing indexes. Use these techniques to diagnose performance problems.

**Enable query logging:**
```php
// In local.config.php
'db' => [
    'profiler' => true,
    'log' => true
]
```

**Check query performance:**
Look for queries taking over 100ms. These are usually relationship-building queries that need optimization through indexing or caching.

**Monitor resource usage:**
Use server monitoring to check if PHP memory or CPU spike during browse page loads. High resource usage suggests too much data is being processed per request.

### 6.3 Filter Logic Problems

When filters don't behave as expected, the issue is usually data quality rather than code problems.

**Common data issues:**
- **Inconsistent values**: "Alabama" vs "AL" vs "alabama" are treated as different
- **Extra spaces**: "Alabama " with trailing space won't match "Alabama"
- **Missing relationships**: Items with State but no County break the filter chain
- **Multi-value confusion**: Items with multiple counties appear in unexpected places

**Data quality SQL queries:**
```sql
-- Find inconsistent state values
SELECT DISTINCT(value_text), COUNT(*) 
FROM value 
WHERE property_id = [state_property_id]
GROUP BY LOWER(TRIM(value_text));

-- Find items missing county when state exists
SELECT r.id FROM resource r
WHERE EXISTS (SELECT 1 FROM value WHERE resource_id = r.id AND property_id = [state_id])
AND NOT EXISTS (SELECT 1 FROM value WHERE resource_id = r.id AND property_id = [county_id]);
```

---

## 7. Customization and Theming

### 7.1 Styling Browse Filters

DualPropertySearch provides semantic HTML with CSS classes for complete style control. The module doesn't enforce any visual design, letting your theme determine appearance.

**CSS structure for targeting:**
```css
/* Container for all browse filters */
.browse-filters {
    /* Position above or beside results */
}

/* Individual filter groups */
.filter-group {
    /* Spacing between filter sets */
}

/* Filter dropdowns */
.filter-select {
    /* Dropdown appearance */
}

/* Active filter tags */
.active-filters .filter-tag {
    /* Removable filter badges */
}

/* Clear filters button */
.clear-filters-btn {
    /* Reset button styling */
}
```

### 7.2 Mobile Responsive Considerations

Browse pages with filters need special attention on mobile devices where screen space is limited. DualPropertySearch includes responsive behaviors, but themes should enhance these.

**Mobile optimization strategies:**
- Collapse filters into an expandable panel on small screens
- Stack dropdowns vertically instead of horizontally
- Use full-width dropdowns for better touch targets
- Show active filters as a count ("3 filters active") with tap to expand
- Consider moving filters to a slide-out drawer

### 7.3 Accessibility Requirements

Ensure browse page filters are accessible to all users, including those using screen readers or keyboard navigation.

**Key accessibility features:**
- Proper label association with form controls
- Keyboard navigation between filters
- ARIA labels for dynamic content updates
- Focus management when filters are applied
- Screen reader announcements for result changes

---

## 8. Integration Best Practices

### 8.1 Planning Filter Strategy

Before implementing DualPropertySearch, plan your filtering strategy based on user research and collection characteristics.

**Questions to answer:**
1. What are the most common browsing patterns?
2. Which metadata properties do users understand?
3. How many filter levels are actually useful?
4. Should different collections have different filters?
5. How do filters interact with search?

### 8.2 Progressive Disclosure

Don't overwhelm users with too many filter options immediately. Start with essential filters and reveal additional options as needed.

**Implementation approach:**
- Show primary filters by default
- Add "Advanced filters" expansion for additional properties
- Remember user preferences for filter visibility
- Provide filter presets for common research patterns

### 8.3 Analytics and Monitoring

Track how users interact with browse page filters to improve the experience over time.

**Metrics to monitor:**
- Most used filter combinations
- Filters that return zero results
- Average number of filter refinements per session
- Filter abandonment rate
- Performance metrics for different filter combinations

---

## Related Documents

- `unitedsearch.md` - Base search module configuration

---

## Change History

| Version | Date | Author | Description |
|---------|------|--------|-------------|
| 1.0 | Nov 3 2025 | Sai Kiran Boppana | Initial DualPropertySearch documentation |

---

*This document is part of the Rosenwald-Fund Collection documentation suite.*
