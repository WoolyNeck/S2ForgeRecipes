# S2ForgeRecipes

Forge Recipes for Season2 of The Cycle: Frontier game

## Goal

Just a quick UI to help forging

## Development

```bash
npm install
npm run dev
```

## Build

```bash
npm run build
```

## GitHub Pages Deployment

This project is automatically deployed to GitHub Pages when you push to the main branch.

### Setup Steps:

1. **Push to GitHub:**
   ```bash
   git add .
   git commit -m "Initial commit"
   git remote add origin https://github.com/YOUR_USERNAME/S2ForgeRecipes.git
   git push -u origin main
   ```

2. **Enable GitHub Pages:**
   - Go to your repository settings
   - Navigate to "Pages" in the left sidebar
   - Under "Build and deployment", set Source to "GitHub Actions"

3. **Done!** The GitHub Action will automatically build and deploy your app. It will be available at:
   `https://YOUR_USERNAME.github.io/S2ForgeRecipes/`

### Note:
If your repository name is different from `S2ForgeRecipes`, update the `base` path in [vite.config.js](vite.config.js#L6)