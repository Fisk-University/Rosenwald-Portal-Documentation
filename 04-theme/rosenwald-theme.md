# THEME-001: Rosenwald Theme Installation and Configuration Guide

## Purpose
This comprehensive guide covers the installation, configuration, and customization of the Rosenwald Fund Collection theme for Omeka-S. This theme was specifically designed for displaying archival materials related to Julius Rosenwald's philanthropic work in African-American education. The theme emphasizes metadata-first display principles, supports dual-property search functionality through integration with custom modules, and provides a modern, accessible interface suitable for academic and cultural heritage institutions.

## Applies To
- Omeka-S installations version 4.0 and above
- Institutions implementing the Rosenwald digital archive framework
- Sites requiring advanced search capabilities with geographic metadata filtering
- Academic libraries and HBCUs digitizing historical collections

## Maintainer
RWCF Development Team

## Last Updated
Nov 03 2025

---

## 1. Prerequisites

Before installing the Rosenwald theme, ensure your environment meets these requirements. Each component plays a crucial role in the theme's functionality and customization capabilities.

| Requirement | Description | Why It's Needed |
|------------|-------------|-----------------|
| **Omeka-S Installation** | Version 4.0 or higher, fully configured | The theme requires modern Omeka-S API features introduced in v4.0 |
| **PHP Version** | PHP 7.4+ with JSON, XML, and GD extensions | Theme uses modern PHP features and image processing capabilities |
| **Node.js** | Version 14+ with npm package manager | Required for SCSS compilation and build tools if customizing styles |
| **Git** | Latest version for version control | Enables cloning from repository and tracking customizations |
| **File Permissions** | Write access (755) to `/themes` directory | Apache needs to read theme files; admin needs write for updates |
| **Recommended Modules** | UnitedSearch, ZipImport, AnyCloud | Theme has specific styling and integration for these modules |

---

## 2. Understanding the Theme Structure

The Rosenwald theme follows Omeka-S's standard theme architecture but includes additional build tools for SCSS compilation. Understanding this structure is essential for customization and troubleshooting.

### 2.1 Directory Layout and Purpose

Each directory serves a specific function in the theme's operation. The separation of concerns makes maintenance and customization more manageable.

```
Rosenwald-Fund-Collection/
│
├── asset/                          # All static resources
│   ├── css/                        # Compiled stylesheets (DO NOT EDIT DIRECTLY)
│   │   ├── style.css              # Main compiled CSS from SCSS
│   │   └── style.css.map          # Source map for debugging
│   │
│   ├── sass/                       # SCSS source files (EDIT THESE)
│   │   ├── _base.scss             # Reset and foundational styles
│   │   ├── _variables.scss        # Color, spacing, typography variables
│   │   ├── _mixins.scss           # Reusable SCSS functions
│   │   ├── _layout.scss           # Grid and structural styles
│   │   ├── _components.scss       # Button, card, form styles
│   │   ├── _modules.scss          # Module-specific styling
│   │   └── style.scss             # Main file that imports all partials
│   │
│   ├── js/                         # JavaScript files
│   │   ├── theme.js               # Main theme JavaScript
│   │   ├── search-enhancements.js # Dual-property search functionality
│   │   └── lazy-load.js           # Image lazy loading implementation
│   │
│   ├── fonts/                      # Custom web fonts
│   │   └── [font-files]           # WOFF2/WOFF files for performance
│   │
│   └── img/                        # Theme images
│       ├── default-header.jpg     # Fallback header image
│       └── icons/                  # UI icons and sprites
│
├── view/                           # PHP template files
│   ├── common/                     # Shared template components
│   │   ├── header.phtml           # Site header with navigation
│   │   ├── footer.phtml           # Site footer with credits
│   │   └── pagination.phtml       # Reusable pagination component
│   │
│   ├── layout/                     # Layout wrapper templates
│   │   └── layout.phtml           # Main HTML structure
│   │
│   └── omeka/
│       └── site/
│           ├── index/             # Homepage templates
│           │   └── index.phtml    # Homepage layout
│           │
│           ├── item/              # Item display templates
│           │   ├── browse.phtml   # Grid/list view of items
│           │   └── show.phtml     # Single item detailed view
│           │
│           ├── item-set/          # Collection templates
│           │   ├── browse.phtml   # Collections listing
│           │   └── show.phtml     # Single collection view
│           │
│           ├── page/              # Static page templates
│           │   └── show.phtml     # Custom page display
│           │
│           └── search/             # Search result templates
│               ├── index.phtml    # Search form
│               └── results.phtml  # Search results display
│
├── config/                         # Configuration files
│   └── theme.ini                  # Theme metadata and settings
│
├── gulpfile.js                     # Gulp task automation
├── package.json                    # Node.js dependencies
├── package-lock.json              # Locked dependency versions
└── README.md                      # Basic documentation
```

### 2.2 Key Files Explained

Understanding these critical files helps you know where to make changes for different customization needs.

#### theme.ini Configuration File
This file controls theme metadata and configurable options that appear in the admin panel. It defines what settings administrators can change without touching code.

```ini
[info]
name = "Rosenwald Fund Collection"
version = "2.0.0"
author = "LaTaevia Berry"
description = "A theme designed for archival collections"
theme_link = "https://github.com/Fisk-University/Rosenwald-Fund-Collection"
omeka_version_constraint = "^3.0.0 || ^4.0.0"

[config]
# These create form fields in the admin interface
elements.logo.type = "Omeka\Form\Element\Asset"
elements.logo.options.label = "Logo"

elements.footer.type = "Textarea"
elements.footer.options.label = "Footer Content"
elements.footer.attributes.rows = 5
```

#### gulpfile.js Build Configuration
This file defines automated tasks for compiling SCSS, optimizing images, and preparing production builds. Understanding gulp tasks helps you maintain the theme efficiently.

```javascript
// Key tasks you'll use:
// gulp sass - Compiles SCSS to CSS once
// gulp sass:watch - Watches for changes and auto-compiles
// gulp build - Production build with minification
// gulp images - Optimizes all images
```

#### Main SCSS File (asset/sass/style.scss)
This is the entry point for all styles. It imports partials in a specific order to ensure proper cascade and variable availability.

```scss
// Import order matters!
@import 'variables';    // Define first
@import 'mixins';       // Functions second
@import 'base';         // Reset third
@import 'layout';       // Structure fourth
@import 'components';   // Components fifth
@import 'modules';      // Module overrides last
```

---

## 3. Installation Process

### 3.1 Downloading and Installing the Theme

The installation process involves cloning the repository, setting proper permissions, and activating through the admin interface. Each step is crucial for proper functionality.

```bash
# Step 1: Navigate to your Omeka themes directory
# This is where all themes must be located for Omeka to recognize them
cd /var/www/html/omeka-s/themes

# Step 2: Clone the repository with the specific branch
# The feature/scss-gulp-setup branch contains the latest SCSS implementation
sudo git clone -b feature/scss-gulp-setup \
    https://github.com/Fisk-University/Rosenwald-Fund-Collection.git

# Step 3: Verify the clone was successful
# You should see all theme files listed
ls -la Rosenwald-Fund-Collection/

# Step 4: Set ownership to web server user
# This ensures Apache can read all theme files
sudo chown -R www-data:www-data Rosenwald-Fund-Collection

# Step 5: Set proper permissions
# 755 allows read/execute for all, write for owner only
sudo chmod -R 755 Rosenwald-Fund-Collection
```

### 3.2 Installing Build Tools (For Customization)

If you plan to customize the SCSS or modify the build process, you'll need Node.js tools. This step is optional for sites using the theme as-is.

```bash
# Navigate to the theme directory
cd /var/www/html/omeka-s/themes/Rosenwald-Fund-Collection

# Install Node.js dependencies defined in package.json
# This installs gulp, sass compiler, and other build tools
npm install

# You may see warnings about vulnerabilities - these are typically
# in development dependencies and don't affect the production theme
# To fix them if needed:
npm audit fix

# Install Gulp CLI globally if not already installed
# This allows you to run gulp commands from command line
npm install -g gulp-cli

# Verify gulp is working by listing available tasks
gulp --tasks

# Test the SCSS compilation to ensure everything works
gulp sass
# You should see: "Finished 'sass' after XXms"
```

### 3.3 Activating the Theme in Omeka-S

After installation, you must activate the theme through Omeka's admin interface. This process registers the theme with your site and applies its templates.

1. **Access Admin Panel**: Navigate to `https://your-site.edu/admin` and log in with administrator credentials

2. **Navigate to Themes Section**: 
   - Click on your site name in the left sidebar
   - Select "Theme" from the submenu
   - You'll see a grid of available themes

3. **Activate Rosenwald Theme**:
   - Find "Rosenwald Fund Collection" in the grid
   - Click the "Use this theme" button
   - The page will reload with the theme activated

4. **Configure Initial Settings**:
   - Click "Configure theme" to access settings
   - Upload a logo (recommended: 300x100px PNG)
   - Set footer text with institution information
   - Save your changes

5. **Verify Installation**:
   - Visit your public site
   - Confirm the new theme is displaying
   - Check that navigation and basic features work

---

## 4. Theme Configuration

### 4.1 Admin Panel Settings

The theme provides various configurable options through the admin interface. These settings allow customization without touching code, making the theme accessible to non-technical administrators.

#### Logo and Branding
The logo appears in the site header and should represent your institution. The theme automatically handles sizing and positioning.

- **Recommended Format**: PNG with transparent background
- **Recommended Size**: 300px width × 100px height
- **File Size**: Keep under 200KB for performance
- **Fallback**: If no logo is set, site title displays as text

#### Footer Configuration
The footer can contain copyright information, funding acknowledgments, and contact details. HTML is supported for formatting.

```html
<!-- Example footer content -->
<p>© 2024 Fisk University. All rights reserved.</p>
<p>This project was made possible by a grant from the Mellon Foundation.</p>
<p>Contact: <a href="mailto:archives@fisk.edu">archives@fisk.edu</a></p>
```

#### Color Customization
While the theme includes default colors matching the Rosenwald project branding, you can override them through custom CSS in the admin panel.

```css
/* Add to Settings → Edit CSS */
:root {
    --primary-color: #8B0000;      /* Rosenwald red */
    --secondary-color: #F5F5DC;    /* Cream accent */
    --text-color: #333333;         /* Dark gray text */
    --link-color: #0066CC;         /* Blue links */
    --link-hover: #0052A3;         /* Darker blue on hover */
}
```

### 4.2 Homepage Block Configuration

The homepage uses Omeka's block system for flexible layouts. Each block serves a specific purpose in presenting your collection to visitors.

#### Creating an Effective Homepage

1. **Hero Section** (First Impression)
   - Use a high-quality historical image (1920×600px recommended)
   - Add overlaid text introducing the collection
   - Include a clear call-to-action button

2. **Search Block** (Primary Functionality)
   - Position the dual-property search prominently
   - Configure for State/County or other hierarchical data
   - Ensure it connects to the UnitedSearch module

3. **Featured Items Carousel** (Highlight Important Materials)
   - Select 5-7 visually compelling items
   - Write engaging captions
   - Link to full item records

4. **Browse by Collection** (Navigation Aid)
   - Display item sets as cards with thumbnails
   - Show item counts for each collection
   - Order by importance or alphabetically

5. **Recent Additions** (Fresh Content)
   - Display the last 6-12 items added
   - Helps returning visitors find new content
   - Auto-updates as you add materials

6. **About Section** (Context)
   - Brief project description
   - Link to full about page
   - Include key partners or funders

### 4.3 Navigation Menu Setup

The navigation structure helps users explore your collection efficiently. The theme supports multi-level menus with dropdowns for complex site structures.

```php
// The theme automatically generates navigation from your site's
// navigation settings in Omeka-S admin → Sites → Navigation

// Example structure:
- Home
- Browse
  - All Items
  - Collections
  - Map View
- About
  - The Project
  - Julius Rosenwald
  - Partner Institutions
- Search
- Contact
```

---

## 5. SCSS Customization Guide

### 5.1 Understanding the SCSS Structure

SCSS (Sassy CSS) provides variables, nesting, and functions that make styling more maintainable. The theme uses a modular approach where each aspect of styling is separated into its own file.

#### Variables File (_variables.scss)
This file centralizes all design tokens. Changing a variable here updates it throughout the entire theme. This is where you define your design system.

```scss
// Color Palette - These establish the visual identity
$primary-color: #8B0000;        // Used for headers, buttons
$secondary-color: #F5F5DC;      // Used for backgrounds, accents
$success-color: #28a745;        // Positive actions, confirmations
$warning-color: #ffc107;        // Warnings, important notices
$error-color: #dc3545;          // Errors, destructive actions

// Typography Scale - Ensures consistent text sizing
$base-font-size: 16px;          // Body text size
$h1-size: 2.5rem;               // Page titles
$h2-size: 2rem;                 // Section headers
$h3-size: 1.75rem;              // Subsection headers
$h4-size: 1.5rem;               // Card titles
$h5-size: 1.25rem;              // Metadata labels
$h6-size: 1rem;                 // Small headers

// Spacing System - Creates visual rhythm
$spacing-unit: 8px;             // Base spacing unit
$spacing-xs: $spacing-unit;     // 8px
$spacing-sm: $spacing-unit * 2; // 16px
$spacing-md: $spacing-unit * 3; // 24px
$spacing-lg: $spacing-unit * 4; // 32px
$spacing-xl: $spacing-unit * 6; // 48px

// Layout Dimensions
$container-width: 1200px;        // Maximum content width
$sidebar-width: 300px;           // Fixed sidebar width
$header-height: 80px;            // Fixed header height

// Responsive Breakpoints - Mobile-first approach
$mobile: 480px;                  // Phones
$tablet: 768px;                  // Tablets
$desktop: 1024px;                // Laptops
$wide: 1440px;                   // Large screens
```

#### Mixins File (_mixins.scss)
Mixins are reusable chunks of CSS that can accept parameters. They reduce repetition and ensure consistency across components.

```scss
// Button mixin - Creates consistent button styles
@mixin button($bg-color: $primary-color, $text-color: white) {
    display: inline-block;
    padding: 12px 24px;
    background-color: $bg-color;
    color: $text-color;
    border: none;
    border-radius: 4px;
    font-weight: 600;
    text-decoration: none;
    transition: all 0.3s ease;
    cursor: pointer;
    
    &:hover {
        background-color: darken($bg-color, 10%);
        transform: translateY(-2px);
        box-shadow: 0 4px 8px rgba(0,0,0,0.15);
    }
    
    &:active {
        transform: translateY(0);
    }
    
    &:disabled {
        opacity: 0.6;
        cursor: not-allowed;
    }
}

// Card mixin - Creates consistent card layouts
@mixin card($padding: $spacing-md) {
    background: white;
    border-radius: 8px;
    box-shadow: 0 2px 4px rgba(0,0,0,0.1);
    padding: $padding;
    transition: box-shadow 0.3s ease;
    
    &:hover {
        box-shadow: 0 4px 12px rgba(0,0,0,0.15);
    }
}

// Responsive mixin - Simplifies media queries
@mixin respond-to($breakpoint) {
    @if $breakpoint == 'mobile' {
        @media (max-width: $mobile) { @content; }
    } @else if $breakpoint == 'tablet' {
        @media (min-width: #{$mobile + 1px}) and (max-width: $tablet) { @content; }
    } @else if $breakpoint == 'desktop' {
        @media (min-width: #{$tablet + 1px}) { @content; }
    }
}
```

### 5.2 Compiling SCSS Changes

After making changes to SCSS files, you must compile them to CSS. The theme uses Gulp to automate this process and ensure consistency.

```bash
# One-time compilation after making changes
gulp sass

# This command:
# 1. Reads all SCSS files starting with style.scss
# 2. Processes imports and variables
# 3. Compiles to standard CSS
# 4. Generates source maps for debugging
# 5. Outputs to asset/css/style.css

# Watch mode for development
gulp sass:watch

# This command:
# 1. Runs initial compilation
# 2. Watches all SCSS files for changes
# 3. Automatically recompiles when you save
# 4. Shows compilation errors in terminal
# 5. Continues running until you stop it (Ctrl+C)

# Production build with optimization
gulp build --production

# This command:
# 1. Compiles SCSS with compression
# 2. Removes source maps
# 3. Autoprefixes for browser compatibility
# 4. Minifies the output
# 5. Optimizes for production use
```

### 5.3 Common Customization Examples

These examples show how to customize frequent design requirements without breaking the theme's structure.

#### Changing the Color Scheme
```scss
// In _variables.scss, update the color palette
$primary-color: #003366;        // Change from red to navy
$secondary-color: #F0E68C;      // Change from cream to khaki

// The entire theme will update automatically
// Buttons, links, headers all use these variables
```

#### Customizing Item Grid Layout
```scss
// In _components.scss
.items-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
    gap: $spacing-lg;
    
    // Responsive adjustments
    @include respond-to('mobile') {
        grid-template-columns: 1fr;  // Single column on mobile
    }
    
    @include respond-to('tablet') {
        grid-template-columns: repeat(2, 1fr);  // Two columns on tablet
    }
    
    .item-card {
        @include card();  // Use the card mixin
        
        img {
            width: 100%;
            height: 200px;
            object-fit: cover;
            border-radius: 4px 4px 0 0;
        }
        
        .item-title {
            font-size: $h5-size;
            margin: $spacing-sm 0;
            color: $primary-color;
        }
    }
}
```

---

## 6. Template Customization

### 6.1 Understanding Template Hierarchy

Omeka-S uses a specific template hierarchy to determine which PHP file renders each page type. Understanding this hierarchy helps you customize the right template for your needs.

```
Template Loading Order (first found wins):
1. Theme template (your customization)
2. Module template (if module provides one)
3. Omeka core template (default fallback)

This means you only need to create templates for
pages you want to customize - others use defaults
```

## 6.2 Customizing Item Display (REVISED)

The item display template controls how individual items appear to visitors. Understanding what metadata to show and how to organize it is crucial for your archive's usability. The template receives an `$item` object from Omeka that contains all the item's data, and your job is to extract and display the relevant fields in a user-friendly way.

For the Rosenwald project, you'll want to display specific metadata fields that are relevant to historical school records. These might include school name, location (state/county), year built, and funding information. Rather than showing ALL metadata fields (which can overwhelm users), select the most important ones for your audience.

**Example of displaying specific metadata:**
```php
// Get only the fields relevant to your collection
$schoolName = $item->value('rosenwald:schoolName');
$yearBuilt = $item->value('dcterms:date');

// Display them in a structured way
echo "<h2>$schoolName</h2>";
echo "<p>Built: $yearBuilt</p>";
```

The key is identifying which metadata fields your users need most and presenting them clearly. Test with actual users to see if they can find the information they're looking for quickly.

## 6.3 Creating Custom Page Blocks (REVISED)

Custom blocks allow site administrators to build rich pages without coding. Think of blocks as reusable components like "hero banner," "featured items grid," or "search box" that can be arranged on any page. Each block has its own template file and can accept configuration options through the admin interface.

Blocks are powerful because they separate content from presentation. An administrator can add a hero banner to multiple pages, each with different images and text, using the same underlying template. This consistency improves maintenance and ensures design cohesion across your site.

To create a custom block, you need three things: a block template file in the correct directory, registration in your module/theme, and proper handling of the block's configuration data. The block receives its settings through the `$block` object.

**Basic block structure example:**
```php
// Get configuration from the block
$heading = $block->dataValue('heading');
$bgColor = $block->dataValue('backgroundColor', '#ffffff');

// Output the HTML with those values
<div style="background: <?= $bgColor ?>">
    <h2><?= $heading ?></h2>
</div>
```
### 7 Module Integration
## 7.1 UnitedSearch Integration

The UnitedSearch module provides two types of search blocks that the theme must style appropriately. The dual-property search is particularly important for the Rosenwald project as it enables searching by State and County in a hierarchical manner. When a user selects a state, the county dropdown dynamically updates to show only counties in that state.

The theme needs specific CSS classes to style these search forms consistently with your design. The module outputs standard HTML forms with predictable class names, so your theme can target these elements without modifying the module itself. This separation ensures the module remains updatable while your styling persists.

JavaScript enhancement is also important here. While the module provides basic functionality, the theme can add loading states, animations, and improved user feedback. For example, when the county dropdown is updating after a state selection, you might show a spinner or disable the field temporarily to indicate something is happening.

**Key styling targets:**
```scss
.dual-property-search {
    // Style the overall search container
}
.property-select {
    // Style each dropdown container
}
.search-submit {
    // Style the search button
}
```

## 7.2 ZipImport Module Support

The ZipImport module requires special consideration because it handles bulk uploads that can create hundreds of items at once. The theme must be prepared to display these items consistently, regardless of whether they were added individually or through bulk import. This means your item templates should handle potentially missing metadata gracefully.

An important consideration is that bulk-imported items might have inconsistent metadata. For example, some items might have thumbnails while others don't, or some might have complete location data while others have only partial information. Your templates need fallback displays for these scenarios to maintain a professional appearance.

The theme should also provide clear visual feedback in the admin area during imports. Since imports can take several minutes for large batches, administrators need to see progress indicators and status messages. This might include progress bars, status badges (processing, complete, error), and summary statistics.

**Handling missing data example:**
```php
// Always check if data exists before displaying
$thumbnail = $item->primaryMedia();
if ($thumbnail) {
    // Show thumbnail
} else {
    // Show placeholder image
}
```

## 7.3 AnyCloud S3 Integration

When using AnyCloud with AWS S3, media files are stored in the cloud rather than on your server. This has significant implications for how the theme handles images. S3 URLs are different from local URLs, often longer and from external domains. The theme must handle both local and S3 media seamlessly.

Performance optimization becomes crucial with S3 media. Since images are loading from external servers, you want to implement lazy loading to prevent too many simultaneous requests. The theme should load visible images first, then load others as users scroll. This dramatically improves initial page load times, especially on browse pages with many items.

Another consideration is responsive images. S3 can store multiple sizes of each image, and the theme should request the appropriate size based on the display context. Loading a 4000px image for a 200px thumbnail wastes bandwidth and slows your site. The theme should use Omeka's built-in thumbnail sizing with S3's different size URLs.

**Detecting and handling S3 media:**
```php
$mediaUrl = $media->originalUrl();
$isS3 = strpos($mediaUrl, 's3.amazonaws.com') !== false;

if ($isS3) {
    // Use lazy loading and CDN optimization
    echo '<img data-src="' . $mediaUrl . '" class="lazy">';
}
```

The key is that S3 media requires different optimization strategies than local files, and your theme should detect and handle these cases automatically.

---

## 8. Troubleshooting Guide

### 8.1 Common Installation Issues

These are the most frequent problems encountered during theme installation and their solutions.

| Issue | Symptoms | Cause | Solution |
|-------|----------|-------|----------|
| **Theme not appearing** | Theme doesn't show in admin panel | Incorrect folder name or permissions | Rename folder to exactly "Rosenwald-Fund-Collection", set permissions to 755 |
| **Styles not loading** | Site appears unstyled or broken | CSS file missing or path issues | Run `gulp sass` to compile SCSS, check for compilation errors |
| **"Cannot read property" errors** | JavaScript console errors | Missing Node modules | Run `npm install` in theme directory |
| **Gulp command not found** | Terminal error when running gulp | Gulp CLI not installed globally | Run `npm install -g gulp-cli` |
| **SCSS compilation fails** | Error messages during gulp sass | Syntax errors in SCSS files | Check error message for file and line number, fix syntax |
| **Images not displaying** | Broken image icons | Wrong paths or permissions | Check asset/img/ permissions, verify image paths in templates |
| **Module styles missing** | UnitedSearch not styled properly | Module not activated | Ensure module is installed and activated in Omeka |

### 8.2 SCSS Compilation Errors

Understanding compilation errors helps you fix styling issues quickly. Here are common errors and their solutions.

#### Error: `File to import not found or unreadable`
```scss
// Error example:
Error: File to import not found or unreadable: variables
       on line 5 of sass/style.scss
>> @import 'variables';

// Solution: Check that _variables.scss exists and path is correct
// The underscore prefix is required for partial files
```

#### Error: `Undefined variable`
```scss
// Error example:
Error: Undefined variable: "$primary-color"
       on line 23 of sass/_components.scss

// Solution: Ensure variables are imported before use
// Check import order in style.scss
```

#### Error: `Invalid CSS after...`
```scss
// Error example:
Error: Invalid CSS after "...ckground-color:": expected expression

// Solution: Check for missing semicolons, brackets, or values
// Look for typos in property names
```

### 8.3 Performance Issues

If the site is loading slowly, these optimizations can help improve performance.

#### Image Optimization
```bash
# Install image optimization tools
npm install --save-dev gulp-imagemin

# Add to gulpfile.js:
const imagemin = require('gulp-imagemin');

gulp.task('images', () => {
    return gulp.src('asset/img/**/*')
        .pipe(imagemin([
            imagemin.mozjpeg({quality: 75, progressive: true}),
            imagemin.optipng({optimizationLevel: 5}),
            imagemin.svgo({
                plugins: [{removeViewBox: false}]
            })
        ]))
        .pipe(gulp.dest('asset/img'));
});

# Run optimization
gulp images
```

#### CSS Optimization
```bash
# Production build with minification
gulp build --production

# This creates minified CSS without sourcemaps
# Reduces file size by 30-40% typically
```

#### Enable Browser Caching
Add to `.htaccess` in theme root:
```apache
# Enable browser caching for theme assets
<IfModule mod_expires.c>
    ExpiresActive On
    ExpiresByType text/css "access plus 1 month"
    ExpiresByType application/javascript "access plus 1 month"
    ExpiresByType image/jpeg "access plus 1 year"
    ExpiresByType image/png "access plus 1 year"
    ExpiresByType image/svg+xml "access plus 1 year"
    ExpiresByType font/woff2 "access plus 1 year"
</IfModule>

# Enable compression
<IfModule mod_deflate.c>
    AddOutputFilterByType DEFLATE text/html text/css application/javascript
</IfModule>
```

### 8.4 Debug Mode

Enable debug mode when troubleshooting to see detailed error messages.

```php
// In config/theme.ini, add:
[info]
debug = true

// In config/local.config.php (Omeka root), add:
return [
    'logger' => [
        'log' => true,
        'priority' => \Laminas\Log\Logger::DEBUG,
    ],
    'translator' => [
        'locale' => 'en_US',
    ],
    'view_manager' => [
        'display_exceptions' => true,
    ],
];
```

Check logs for detailed error information:
```bash
# Theme-specific errors
tail -f /var/www/html/omeka-s/logs/application.log

# PHP errors
tail -f /var/log/apache2/error.log

# Watch both simultaneously
tail -f /var/www/html/omeka-s/logs/application.log /var/log/apache2/error.log
```

---

## 9. Best Practices

### 9.1 Development Workflow

Following a structured development process ensures maintainable customizations and prevents issues.

1. **Version Control Everything**
   ```bash
   # Initialize git in your theme
   cd themes/Rosenwald-Fund-Collection
   git init
   git add .
   git commit -m "Initial theme customization"
   ```

2. **Use Feature Branches**
   ```bash
   # Create branch for new feature
   git checkout -b feature/custom-homepage-blocks
   # Make changes, test, then merge
   ```

3. **Document Customizations**
   - Keep a CUSTOMIZATIONS.md file
   - List all modified files
   - Explain why changes were made
   - Include examples of usage

4. **Test on Staging First**
   - Never edit production directly
   - Clone production to staging
   - Test thoroughly before deploying

5. **Keep Dependencies Updated**
   ```bash
   # Regularly update Node packages
   npm update
   npm audit fix
   ```

### 9.2 Accessibility Guidelines

Ensuring your site is accessible to all users is both ethical and often legally required.

- **Color Contrast**: Maintain WCAG AA standards (4.5:1 for normal text, 3:1 for large text)
- **Semantic HTML**: Use proper heading hierarchy (h1 → h2 → h3)
- **Alt Text**: Every image must have descriptive alt text
- **Keyboard Navigation**: All interactive elements must be keyboard accessible
- **ARIA Labels**: Use ARIA labels for screen reader users where needed
- **Focus Indicators**: Ensure visible focus states for keyboard users

### 9.3 Performance Optimization

Optimize for fast page loads, especially important for users with slower connections.

- **Image Guidelines**:
  - Maximum width: 1920px for hero images
  - Use WebP format with JPG fallback
  - Implement lazy loading for below-fold images
  - Compress all images before upload

- **CSS/JS Optimization**:
  - Minify production CSS and JavaScript
  - Combine files where possible
  - Use CDN for common libraries
  - Remove unused styles

- **Caching Strategy**:
  - Enable browser caching via .htaccess
  - Use Omeka's built-in page caching
  - Consider CDN for static assets
  - Implement service worker for offline access

---

## 10. Related Documents

- `unitedsearch.md` - Dual property search configuration
- `zipimport.md` - Bulk import module setup and usage
- `dualpropertysearch.md` - Advanced search implementation
- `anycloud-config.md` - S3 storage configuration for media files
- `omeka-install.md` - Base Omeka-S installation and configuration

---

## Change History

| Version | Date | Author | Description |
|---------|------|--------|-------------|
| 1.0 | Nov 3 2025 | Sai Kiran Boppana | Initial comprehensive theme documentation |
| 1.1 | Nov 3 2025 | Sai Kiran Boppana | Added SCSS compilation troubleshooting |

---

*This document is part of the Rosenwald-Fund Collection documentation suite.*
