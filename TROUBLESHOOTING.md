# Troubleshooting: Menu Not Showing

## Problem: Menu/Navigation Not Visible on Live Site

If you're seeing a basic page without the menu, navigation tabs, or proper styling, follow these steps:

---

## Step 1: Verify GitHub Pages Source Setting

1. Go to your repository: https://github.com/cyberfranko/cyberfranko.github.io
2. Click **Settings** tab
3. Click **Pages** in the left sidebar
4. Under "Build and deployment":
   - **Source** MUST be set to `GitHub Actions` (NOT "Deploy from a branch")

### If it says "Deploy from a branch":
This is the problem! GitHub is serving old static files instead of your MkDocs site.

**Fix:**
- Change **Source** to `GitHub Actions`
- Click Save
- Wait 2-3 minutes
- Refresh your site

---

## Step 2: Check if Workflow Exists

1. Go to the **Actions** tab in your repository
2. You should see a workflow called "Deploy MkDocs to GitHub Pages"
3. Check if it ran after your last push

### If you don't see any workflows:
The workflow file might not have been pushed. Let's verify:

```powershell
# Check if workflow file exists
ls .github/workflows/deploy.yml

# If it doesn't exist, you need to push it
git add .github/workflows/deploy.yml
git commit -m "Add GitHub Actions workflow"
git push origin main
```

---

## Step 3: Manually Trigger Deployment

If the workflow exists but didn't run:

1. Go to **Actions** tab
2. Click on "Deploy MkDocs to GitHub Pages" on the left
3. Click the **Run workflow** button (dropdown on the right)
4. Select `main` branch
5. Click green **Run workflow** button
6. Wait 1-2 minutes for it to complete
7. Refresh your site: https://cyberfranko.github.io

---

## Step 4: Check Workflow Status

If the workflow ran but failed:

1. Go to **Actions** tab
2. Click on the failed workflow run (red X icon)
3. Click on the job that failed
4. Check the error message
5. Common issues:
   - Missing permissions (already set in our workflow)
   - Syntax error in mkdocs.yml
   - Missing dependencies

---

## Step 5: Clear Browser Cache

Sometimes your browser shows an old cached version:

- **Windows**: Press `Ctrl + Shift + R` (hard refresh)
- **Mac**: Press `Cmd + Shift + R`
- Or try opening in **Incognito/Private window**

---

## Step 6: Verify Deployment

After workflow completes successfully:

1. Visit: https://cyberfranko.github.io
2. You should see:
   - Top navigation tabs: Home | Cybersecurity | Programming | Projects
   - Search icon in top right
   - Dark/Light mode toggle
   - Professional Material Design theme

---

## Quick Checklist

- [ ] GitHub Pages Source is set to "GitHub Actions"
- [ ] Workflow file exists at `.github/workflows/deploy.yml`
- [ ] Workflow ran successfully (green checkmark in Actions tab)
- [ ] Cleared browser cache / tried incognito mode
- [ ] Waited at least 2-3 minutes after workflow completion

---

## Still Not Working?

### Test Locally First

```powershell
# Build the site locally
python -m mkdocs build

# Serve it locally
python -m mkdocs serve
```

Open http://127.0.0.1:8000 - if the menu shows here, the configuration is correct and it's a deployment issue.

---

## Most Common Issue

**90% of the time**, the issue is that GitHub Pages Source is set to "Deploy from a branch" instead of "GitHub Actions".

**Fix:** Settings → Pages → Source → Select "GitHub Actions"

---

## Need More Help?

Check the workflow logs:
1. Go to **Actions** tab
2. Click latest workflow run
3. Click on the "build" or "deploy" job
4. Read the logs for error messages

---

## What Should Work

Once properly configured:
- Every push to `main` branch automatically triggers deployment
- Takes 1-2 minutes to complete
- Site updates automatically
- Full navigation, search, and theme features work

