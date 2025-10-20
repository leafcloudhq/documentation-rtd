# Leafcloud Documentation - Styling Maintenance Guide

This guide helps you maintain and customize the Leafcloud documentation styling.

---

## 📁 Key Files

### 1. `/docs/stylesheets/extra.css`
The main custom stylesheet containing all brand styling.

**Structure:**
```css
/* Fonts - Line 1-12 */
/* Root Variables - Line 14-67 */
/* Global Settings - Line 69-75 */
/* Header/Navigation - Line 77-142 */
/* Sidebar Navigation - Line 144-200 */
/* Table of Contents - Line 202-230 */
/* Main Content - Line 232-350 */
/* Code Blocks - Line 352-410 */
/* Admonitions - Line 412-480 */
/* Tables - Line 482-520 */
/* Buttons - Line 522-580 */
/* Footer - Line 582-640 */
/* Images - Line 642-670 */
/* Search - Line 672-720 */
/* Scrollbar - Line 722-740 */
/* Responsive - Line 742-780 */
/* Utilities - Line 782-830 */
```

### 2. `/mkdocs.yml`
MkDocs configuration with theme settings, features, and extensions.

### 3. Brand Assets
- `/docs/assets/fonts/SeasonVFTRIALVF.woff2` - Main font
- `/docs/assets/image/logo.svg` - Logo
- `/docs/assets/image/favicon.svg` - Favicon

---

## 🎨 Common Customization Tasks

### Change Brand Colors

**Location:** `/docs/stylesheets/extra.css` (Lines 8-13)

```css
:root {
  /* Brand Colors */
  --leafcloud-green: #00E08F;    /* Primary accent */
  --leafcloud-blue: #0077FF;     /* Secondary accent */
  --leafcloud-black: #121926;    /* Header/footer/text */
  --leafcloud-white: #ffffff;    /* Backgrounds */
  --leafcloud-gray: #444;        /* Body text */
}
```

**To change:**
1. Update the color values
2. Save the file
3. Refresh the documentation (auto-reloads if using `mkdocs serve`)

### Change Font

**Option 1: Use a different custom font**

1. Add font file to `/docs/assets/fonts/`
2. Update `@font-face` in `extra.css` (Line 1-12):
```css
@font-face {
  font-family: "YourFont";
  src: url("../assets/fonts/YourFont.woff2") format("woff2");
  font-weight: 400-700;
  font-style: normal;
  font-display: swap;
}
```
3. Update the font variable (Line 23):
```css
--md-text-font: "YourFont", -apple-system, sans-serif;
```

**Option 2: Use a system font**

Update the font variable directly:
```css
--md-text-font: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
```

### Change Header/Footer Style

**Header Background** (Line 91):
```css
.md-header {
  background-color: var(--leafcloud-black); /* Change this */
  color: var(--leafcloud-white);
}
```

**Footer Background** (Line 582):
```css
.md-footer {
  background-color: var(--leafcloud-black); /* Change this */
  color: var(--leafcloud-white);
}
```

### Adjust Link Colors

**Primary Links** (Line 304):
```css
.md-typeset a {
  color: var(--leafcloud-green);    /* Normal state */
}

.md-typeset a:hover {
  color: var(--leafcloud-blue);     /* Hover state */
}
```

### Customize Code Blocks

**Background Color** (Line 366):
```css
.md-typeset code {
  background-color: #f8f9fa;  /* Change this */
  border: 1px solid var(--leafcloud-gray-light);
}
```

**Header Accent** (Line 387):
```css
.md-typeset .highlight::before {
  background-color: var(--leafcloud-green);  /* Change this */
}
```

### Modify Button Styles

**Primary Button** (Line 522):
```css
.md-typeset .md-button {
  background-color: var(--leafcloud-green);  /* Background */
  color: var(--leafcloud-black);             /* Text */
  border-radius: 6px;                        /* Corners */
  padding: 0.625rem 1.5rem;                  /* Padding */
}
```

### Adjust Spacing

**Content Padding** (Line 242):
```css
.md-content__inner {
  padding: 2rem 2.5rem;  /* Vertical, Horizontal */
}
```

**Heading Margins** (Lines 260-310):
```css
.md-typeset h1 {
  margin-top: 0;
  margin-bottom: 1.5rem;  /* Adjust as needed */
}
```

---

## 🔧 MkDocs Configuration Changes

### Add/Remove Navigation Tabs

**Location:** `/mkdocs.yml` (Lines 104-115)

To disable tabs:
```yaml
features:
  # - navigation.tabs           # Comment out or remove
  # - navigation.tabs.sticky
```

### Change Theme Colors

**Location:** `/mkdocs.yml` (Lines 86-101)

Note: We use `primary: custom` and `accent: custom` because colors are defined in CSS. To use MkDocs Material's built-in colors instead:

```yaml
palette:
  - scheme: default
    primary: teal        # Or: indigo, blue, green, etc.
    accent: teal
```

### Enable/Disable Features

**Location:** `/mkdocs.yml` (Lines 104-124)

Comment out features you don't want:
```yaml
features:
  - navigation.instant
  # - navigation.tabs        # Disabled
  - navigation.sections
```

### Add Social Links

**Location:** `/mkdocs.yml` (Lines 148-160)

Add new social links:
```yaml
social:
  - icon: fontawesome/brands/github
    link: https://github.com/yourorg
  - icon: fontawesome/brands/mastodon
    link: https://mastodon.social/@yourhandle
```

Available icons: https://squidfunk.github.io/mkdocs-material/setup/changing-the-logo-and-icons/#logo-icon-bundled

---

## 🔍 Troubleshooting

### Styles Not Updating

1. **Hard refresh:** Ctrl+Shift+R (or Cmd+Shift+R on Mac)
2. **Clear browser cache**
3. **Restart mkdocs server:**
   ```bash
   # Stop the server (Ctrl+C)
   mkdocs serve
   ```

### Font Not Loading

1. **Check file path** in `@font-face` declaration
2. **Verify file exists** in `/docs/assets/fonts/`
3. **Check browser console** for 404 errors
4. **Try different format:**
   ```css
   src: url("../assets/fonts/font.woff2") format("woff2"),
        url("../assets/fonts/font.woff") format("woff");
   ```

### Colors Not Applying

1. **Check CSS specificity** - May need `!important`
2. **Verify CSS variable** is defined in `:root`
3. **Check for typos** in variable names
4. **Inspect element** in browser dev tools

### Build Errors

**Common issues:**

1. **Invalid YAML:**
   ```bash
   # Test YAML syntax
   python -c "import yaml; yaml.safe_load(open('mkdocs.yml'))"
   ```

2. **Missing dependencies:**
   ```bash
   pip install -r docs/requirements.txt
   ```

3. **Broken links:**
   - Check `mkdocs build` output for warnings
   - Fix relative links in markdown files

---

## 📦 Deployment

### Build for Production

```bash
cd documentation-rtd
mkdocs build
```

Output will be in `/site` directory.

### Deploy to ReadTheDocs

Automatically deploys on push to main branch (configured in `.readthedocs.yml`).

### Deploy to GitHub Pages

```bash
mkdocs gh-deploy
```

### Deploy to Custom Server

1. Build the site: `mkdocs build`
2. Upload `/site` directory contents to your web server
3. Configure server to serve static files

---

## 🎯 Best Practices

### 1. **Keep Backups**
Before making major changes:
```bash
cp docs/stylesheets/extra.css docs/stylesheets/extra.css.backup
```

### 2. **Test Locally**
Always test changes with `mkdocs serve` before deploying.

### 3. **Use Variables**
Define colors and values as CSS variables for easy updates:
```css
:root {
  --my-custom-color: #hexcode;
}

.element {
  color: var(--my-custom-color);
}
```

### 4. **Comment Your Changes**
Add comments to explain custom styling:
```css
/* Custom: Larger heading for mobile devices */
@media screen and (max-width: 600px) {
  .md-typeset h1 {
    font-size: 2rem;
  }
}
```

### 5. **Check Accessibility**
- Test color contrast: https://webaim.org/resources/contrastchecker/
- Test keyboard navigation
- Test with screen readers (if possible)

### 6. **Keep It DRY**
Use CSS variables instead of repeating values:
```css
/* Good */
:root {
  --card-border-radius: 8px;
}
.card { border-radius: var(--card-border-radius); }
.button { border-radius: var(--card-border-radius); }

/* Avoid */
.card { border-radius: 8px; }
.button { border-radius: 8px; }
```

---

## 📚 Resources

### Official Documentation
- **MkDocs:** https://www.mkdocs.org/
- **Material for MkDocs:** https://squidfunk.github.io/mkdocs-material/
- **PyMdown Extensions:** https://facelessuser.github.io/pymdown-extensions/

### CSS Reference
- **MDN CSS:** https://developer.mozilla.org/en-US/docs/Web/CSS
- **CSS Variables:** https://developer.mozilla.org/en-US/docs/Web/CSS/Using_CSS_custom_properties

### Color Tools
- **Contrast Checker:** https://webaim.org/resources/contrastchecker/
- **Color Picker:** https://htmlcolorcodes.com/
- **Palette Generator:** https://coolors.co/

### Font Tools
- **Google Fonts:** https://fonts.google.com/
- **Font Squirrel:** https://www.fontsquirrel.com/
- **Transfonter (Convert fonts):** https://transfonter.org/

---

## 🆘 Getting Help

### Check These First
1. Browser console (F12) for errors
2. MkDocs build output for warnings
3. This maintenance guide
4. MkDocs Material documentation

### Common Solutions
- **Styling not applying:** Clear cache, hard refresh
- **Build fails:** Check YAML syntax, dependencies
- **Font not loading:** Check file path and format
- **Colors wrong:** Verify CSS variables

---

## 📝 Quick Reference

### CSS Classes You Can Use in Markdown

```markdown
<!-- Add a highlight box -->
<div class="highlight-box">
  Important information here
</div>

<!-- Add green text -->
<span class="text-green">Green text</span>

<!-- Add blue text -->
<span class="text-blue">Blue text</span>
```

### Useful mkdocs.yml Settings

```yaml
# Disable feature
features:
  # - navigation.tabs     # Commented = disabled

# Site metadata
site_name: Your Site Name
site_url: https://your-domain.com
site_description: Your description

# Copyright
copyright: Copyright &copy; 2025 Your Company
```

---

## 🔄 Version Control

When committing changes:

```bash
# Check what changed
git diff docs/stylesheets/extra.css
git diff mkdocs.yml

# Commit with descriptive message
git add docs/stylesheets/extra.css mkdocs.yml
git commit -m "docs: update documentation styling - [describe changes]"
git push
```

---

*Last Updated: October 17, 2025*  
*Maintainer: Leafcloud Team*

