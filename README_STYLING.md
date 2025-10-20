# 🎨 Leafcloud Documentation - Styling Guide

> **Updated:** October 17, 2025  
> **Status:** ✅ Production Ready

---

## 🚀 Quick Start

### View Documentation Locally

```bash
cd documentation-rtd
mkdocs serve
```

Visit: http://localhost:8000

### Build for Production

```bash
mkdocs build
```

---

## 📦 What's New

The documentation now features:

✅ **Leafcloud Brand Colors**
- Green: `#00E08F` (primary accent)
- Blue: `#0077FF` (secondary accent)
- Black: `#121926` (header/footer)

✅ **Season Font** - Matching main website  
✅ **Dark Header & Footer** - Professional appearance  
✅ **Navigation Tabs** - Easy section access  
✅ **Modern Design** - Clean, branded look  
✅ **Enhanced Features** - Search, code copy, dark mode  
✅ **Fully Responsive** - Works on all devices  

---

## 📁 Key Files

### Styling
- **`/docs/stylesheets/extra.css`** - All custom styles (900+ lines)
- **`/mkdocs.yml`** - Configuration and features

### Assets
- **`/docs/assets/fonts/SeasonVFTRIALVF.woff2`** - Brand font
- **`/docs/assets/image/logo.svg`** - Leafcloud logo
- **`/docs/assets/image/favicon.svg`** - Favicon

---

## 🎨 Brand Colors Reference

```css
/* Primary Colors */
--leafcloud-green: #00E08F;   /* Links, accents, buttons */
--leafcloud-blue: #0077FF;    /* Hover, info */
--leafcloud-black: #121926;   /* Header, footer, headings */
--leafcloud-white: #ffffff;   /* Backgrounds */
--leafcloud-gray: #444;       /* Body text */
```

---

## 🔧 Common Customizations

### Change a Color

Edit `/docs/stylesheets/extra.css`:
```css
:root {
  --leafcloud-green: #YourColor;  /* Line 9 */
}
```

### Disable Navigation Tabs

Edit `/mkdocs.yml`:
```yaml
features:
  # - navigation.tabs           # Comment out
  # - navigation.tabs.sticky
```

### Change Font

Update `/docs/stylesheets/extra.css`:
```css
--md-text-font: "YourFont", sans-serif;  /* Line 23 */
```

---

## 📖 Documentation

- **Full Summary:** [`STYLING_UPDATE_SUMMARY.md`](./STYLING_UPDATE_SUMMARY.md)
- **Maintenance Guide:** [`STYLING_MAINTENANCE_GUIDE.md`](./STYLING_MAINTENANCE_GUIDE.md)

---

## 🎯 Features Enabled

### Navigation
- ✅ Instant loading
- ✅ Progress indicators
- ✅ Navigation tabs (sticky)
- ✅ Section expansion
- ✅ Back to top button

### Search
- ✅ Search suggestions
- ✅ Highlight results
- ✅ Share search

### Content
- ✅ Code copy buttons
- ✅ Code annotations
- ✅ Edit/view page links
- ✅ Collapsible admonitions

### Theme
- ✅ Light/dark mode toggle
- ✅ Social links in footer
- ✅ Custom colors
- ✅ Responsive design

---

## 🔄 Deployment

### ReadTheDocs (Automatic)
Deploys automatically on push to main branch.

### Manual Build
```bash
mkdocs build
# Upload /site directory to your server
```

### GitHub Pages
```bash
mkdocs gh-deploy
```

---

## 🐛 Troubleshooting

### Styles not updating?
```bash
# Hard refresh browser: Ctrl+Shift+R
# Or restart server:
mkdocs serve
```

### Build errors?
```bash
# Reinstall dependencies:
pip install -r docs/requirements.txt

# Check YAML syntax:
python -c "import yaml; yaml.safe_load(open('mkdocs.yml'))"
```

---

## 📊 Before & After

### Before
- Generic blue/gray theme
- Euclid font
- No brand identity
- Basic features

### After
- ✅ Leafcloud green branding
- ✅ Season font
- ✅ Professional appearance
- ✅ Enhanced features
- ✅ Navigation tabs
- ✅ Dark mode support

---

## 🔗 Resources

- **MkDocs Material:** https://squidfunk.github.io/mkdocs-material/
- **Main Website:** https://leaf.cloud/
- **Documentation:** https://docs.leaf.cloud/

---

## 👥 Support

Questions about styling? Check:
1. [`STYLING_MAINTENANCE_GUIDE.md`](./STYLING_MAINTENANCE_GUIDE.md)
2. MkDocs Material docs
3. Browser console (F12) for errors

---

## ✅ Checklist for Updates

When updating styles:

- [ ] Test locally with `mkdocs serve`
- [ ] Check mobile responsive design
- [ ] Verify color contrast (accessibility)
- [ ] Test all interactive elements
- [ ] Review on multiple browsers
- [ ] Check build output for errors
- [ ] Update version in documentation

---

## 📝 Version History

### Version 2.0 (October 2025)
- Complete rebrand with Leafcloud colors
- Season font integration
- Navigation tabs
- Enhanced features
- Dark mode support
- Social links
- Modern design

### Version 1.0 (Previous)
- Basic Material theme
- Default colors
- Minimal customization

---

*For detailed information, see [`STYLING_UPDATE_SUMMARY.md`](./STYLING_UPDATE_SUMMARY.md)*

