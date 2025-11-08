# GitHub Pages Deployment Guide

## Step-by-Step Instructions

### ✅ Step 1: Test Locally (IMPORTANT!)

Before pushing to GitHub, test your site locally:

```bash
# Install dependencies (if not already installed)
pip install -r requirements.txt

# Start local development server
mkdocs serve
```

Open your browser to [http://127.0.0.1:8000](http://127.0.0.1:8000)

**Verify:**
- [ ] Site loads correctly
- [ ] All pages are accessible
- [ ] Navigation works
- [ ] Search function works
- [ ] Dark/Light mode toggle works
- [ ] Code syntax highlighting works

Press `Ctrl+C` to stop the server.

---

### ✅ Step 2: Push to GitHub

```bash
# Add all files
git add .

# Commit changes
git commit -m "Initial MkDocs setup with content"

# Push to GitHub
git push origin main
```

---

### ✅ Step 3: Configure GitHub Pages

1. **Go to your GitHub repository**: `https://github.com/cyberfranko/cyberfranko.github.io`

2. **Navigate to Settings**:
   - Click on **Settings** tab
   - Click on **Pages** in the left sidebar

3. **Configure Source**:
   - Under "Build and deployment"
   - **Source**: Select `GitHub Actions`
   - (This should be the default option now)

4. **Wait for deployment**:
   - Go to the **Actions** tab
   - Watch the "Deploy MkDocs to GitHub Pages" workflow run
   - It should complete in 1-2 minutes

5. **Visit your site**:
   - Once the action completes, visit: `https://cyberfranko.github.io`
   - It may take a few minutes for DNS to propagate

---

### ✅ Step 4: Verify Deployment

Check that your site is live:

1. Visit `https://cyberfranko.github.io`
2. Test all navigation links
3. Try the search function
4. Verify all content pages load correctly

---

## 📋 Troubleshooting

### Workflow Fails

If the GitHub Actions workflow fails:

1. Check the **Actions** tab for error messages
2. Common issues:
   - Missing permissions (should be set in workflow)
   - Requirements.txt issues (check package versions)
   - Build errors (check mkdocs.yml syntax)

### Site Not Updating

If changes don't appear:

1. Wait 2-3 minutes for deployment to complete
2. Clear browser cache (`Ctrl+Shift+R` or `Cmd+Shift+R`)
3. Check Actions tab to ensure workflow completed

### Page Not Found (404)

1. Verify the file exists in `docs/` folder
2. Check that it's listed in `mkdocs.yml` navigation
3. Ensure file paths match exactly (case-sensitive)

---

## 🔄 Daily Workflow

After initial setup, making updates is simple:

```bash
# 1. Edit files in docs/ folder
# 2. Test locally
mkdocs serve

# 3. Push to GitHub
git add .
git commit -m "Add new content"
git push origin main

# 4. GitHub Actions automatically deploys!
```

---

## 🎨 Customization Tips

### Change Theme Colors

Edit `mkdocs.yml`:

```yaml
theme:
  palette:
    - scheme: default
      primary: teal  # Change this
      accent: amber  # Change this
```

Available colors: red, pink, purple, deep purple, indigo, blue, light blue, cyan, teal, green, light green, lime, yellow, amber, orange, deep orange

### Add New Pages

1. Create markdown file in `docs/`
2. Add to navigation in `mkdocs.yml`:

```yaml
nav:
  - Home: index.md
  - New Section:
      - New Page: path/to/page.md
```

### Add Custom Logo

1. Create `docs/img/` folder
2. Add logo image
3. Update `mkdocs.yml`:

```yaml
theme:
  logo: img/logo.png
  favicon: img/favicon.ico
```

---

## 📚 Useful Resources

- [MkDocs Documentation](https://www.mkdocs.org/)
- [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/)
- [Markdown Guide](https://www.markdownguide.org/)
- [GitHub Pages Docs](https://docs.github.com/en/pages)

---

## ✨ What's Already Set Up

- ✅ Dark/Light theme toggle
- ✅ Search functionality
- ✅ Navigation tabs
- ✅ Code syntax highlighting with copy button
- ✅ Responsive design (mobile-friendly)
- ✅ GitHub repository link
- ✅ Content sections: Cybersecurity, Programming, Projects
- ✅ Automatic deployment via GitHub Actions

---

## 🎉 You're All Set!

Your cybersecurity wiki is ready to go. Just test locally, push to GitHub, and your site will be live!

Need help? Check the troubleshooting section or review the GitHub Actions logs.

