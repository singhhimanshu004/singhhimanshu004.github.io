# Deploy to GitHub Pages

This site is configured as a GitHub user site at:

https://singhhimanshu004.github.io

The repository must be named exactly `singhhimanshu004.github.io`, and the site is served from the root path (`/`).

## Go-live steps

1. Create a new GitHub repository named exactly:

   ```text
   singhhimanshu004.github.io
   ```

2. Set the local remote to that repository and push this project to the `main` branch:

   ```bash
   git remote remove origin 2>/dev/null || true
   git remote add origin git@github.com:singhhimanshu004/singhhimanshu004.github.io.git
   git branch -M main
   git push -u origin main
   ```

   If you prefer HTTPS instead of SSH, use:

   ```bash
   git remote remove origin 2>/dev/null || true
   git remote add origin https://github.com/singhhimanshu004/singhhimanshu004.github.io.git
   git branch -M main
   git push -u origin main
   ```

3. In the GitHub repository, open **Settings → Pages**.

4. Under **Build and deployment**, set **Source** to **GitHub Actions**.

5. Pushes to `main` will automatically run `.github/workflows/deploy.yml`, build the Astro site, upload `dist/`, and deploy with GitHub Pages.

6. After the workflow completes, the site will be live at:

   https://singhhimanshu004.github.io

## Custom domain later

To add a custom domain later:

1. Add the domain in **Settings → Pages → Custom domain**.
2. Create the required DNS records with your DNS provider as shown by GitHub Pages.
3. Add a `public/CNAME` file containing only the domain name, for example:

   ```text
   example.com
   ```

4. Update `astro.config.mjs` so `site` matches the custom domain, for example:

   ```js
   site: 'https://example.com',
   base: '/',
   ```
