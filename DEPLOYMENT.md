# Deployment Guide for TalentFlow

This guide covers deploying TalentFlow to various hosting platforms.

## 🚀 Quick Deploy

### Vercel (Recommended)

[![Deploy with Vercel](https://vercel.com/button)](https://vercel.com/new/clone)

1. **Push to GitHub**
   ```bash
   git add .
   git commit -m "Initial commit"
   git push origin main
   ```

2. **Deploy to Vercel**
   - Go to [vercel.com](https://vercel.com)
   - Click "New Project"
   - Import your GitHub repository
   - Vercel will auto-detect Vite configuration
   - Click "Deploy"

3. **Configuration**
   - Build Command: `npm run build`
   - Output Directory: `dist`
   - Install Command: `npm install`

### Netlify

1. **Push to GitHub** (same as above)

2. **Deploy to Netlify**
   - Go to [netlify.com](https://netlify.com)
   - Click "Add new site" → "Import an existing project"
   - Choose your GitHub repository
   - Configure:
     - Build command: `npm run build`
     - Publish directory: `dist`
   - Click "Deploy site"

3. **Add Redirects** (create `public/_redirects` file)
   ```
   /*    /index.html   200
   ```

### GitHub Pages

1. **Install gh-pages**
   ```bash
   npm install --save-dev gh-pages
   ```

2. **Update package.json**
   ```json
   {
     "homepage": "https://yourusername.github.io/hr_management",
     "scripts": {
       "predeploy": "npm run build",
       "deploy": "gh-pages -d dist"
     }
   }
   ```

3. **Update vite.config.js**
   ```javascript
   export default defineConfig({
     plugins: [react()],
     base: '/hr_management/' // Your repo name
   })
   ```

4. **Deploy**
   ```bash
   npm run deploy
   ```

## 🔧 Build Optimization

### Production Build

```bash
npm run build
```

This creates an optimized production build in the `dist` folder.

### Preview Production Build Locally

```bash
npm run preview
```

### Bundle Analysis

To analyze bundle size:

```bash
npm run build -- --mode analyze
```

## 🌐 Environment Variables

For production deployment, you may want to configure:

1. **Disable Mock Server in Production**
   
   Update `src/main.jsx`:
   ```javascript
   // Only use mock server in development
   if (import.meta.env.DEV) {
     makeServer();
   }
   ```

2. **API Configuration**
   
   If you want to switch to a real backend in the future:
   ```javascript
   const API_BASE_URL = import.meta.env.VITE_API_URL || '/api';
   ```

## 📊 Performance Optimization

### Code Splitting

The app uses React Router which supports automatic code splitting. To enhance:

```javascript
// Use React.lazy for route components
const JobsBoard = lazy(() => import('./features/jobs/JobsBoard'));
```

### Image Optimization

If you add images:
- Use WebP format
- Implement lazy loading
- Use appropriate sizes

### Caching Strategy

Vite automatically generates hashed filenames for caching. Configure in `vite.config.js`:

```javascript
export default defineConfig({
  build: {
    rollupOptions: {
      output: {
        manualChunks: {
          vendor: ['react', 'react-dom', 'react-router-dom'],
          redux: ['@reduxjs/toolkit', 'react-redux'],
          dnd: ['@dnd-kit/core', '@dnd-kit/sortable', '@dnd-kit/utilities'],
        },
      },
    },
  },
});
```

## 🔐 Security Considerations

### Content Security Policy (CSP)

Add to your hosting platform headers:

```
Content-Security-Policy: default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline';
```

### HTTPS

All modern hosting platforms (Vercel, Netlify) provide automatic HTTPS.

## 📱 Progressive Web App (PWA)

To make the app installable:

1. **Install Vite PWA plugin**
   ```bash
   npm install vite-plugin-pwa -D
   ```

2. **Configure in vite.config.js**
   ```javascript
   import { VitePWA } from 'vite-plugin-pwa';
   
   export default defineConfig({
     plugins: [
       react(),
       VitePWA({
         registerType: 'autoUpdate',
         manifest: {
           name: 'TalentFlow',
           short_name: 'TalentFlow',
           description: 'HR Management Platform',
           theme_color: '#3b82f6',
         },
       }),
     ],
   });
   ```

## 🐛 Troubleshooting

### Blank Page After Deployment

**Issue**: App shows blank page in production

**Solutions**:
1. Check browser console for errors
2. Verify `base` path in `vite.config.js` matches your deployment URL
3. Ensure routing redirects are configured (see platform-specific guides above)

### IndexedDB Not Working

**Issue**: Data not persisting

**Solutions**:
1. Check browser compatibility (IndexedDB is supported in all modern browsers)
2. Verify the site is served over HTTPS (some browsers restrict IndexedDB on HTTP)
3. Check browser storage quota

### Build Errors

**Issue**: Build fails with errors

**Solutions**:
1. Delete `node_modules` and reinstall: `rm -rf node_modules && npm install`
2. Clear build cache: `rm -rf dist`
3. Check Node.js version (should be 18+)

## 📈 Monitoring

### Add Analytics

**Google Analytics:**
```javascript
// In index.html
<script async src="https://www.googletagmanager.com/gtag/js?id=GA_MEASUREMENT_ID"></script>
```

**Vercel Analytics:**
```bash
npm install @vercel/analytics
```

```javascript
// In App.jsx
import { Analytics } from '@vercel/analytics/react';

function App() {
  return (
    <>
      <YourApp />
      <Analytics />
    </>
  );
}
```

## ✅ Pre-Deployment Checklist

- [ ] All features tested and working
- [ ] No console errors in production build
- [ ] Responsive design verified on mobile/tablet
- [ ] Performance tested (Lighthouse score)
- [ ] README.md updated with live demo link
- [ ] Meta tags added for social sharing
- [ ] Favicon added
- [ ] Error boundaries implemented
- [ ] Loading states for all async operations
- [ ] 404 page created (optional)

## 🎯 Post-Deployment

1. **Test the deployed app thoroughly**
   - All routes work
   - Data persists after refresh
   - Drag-and-drop works
   - Forms submit correctly

2. **Performance audit**
   - Run Lighthouse in Chrome DevTools
   - Aim for 90+ scores

3. **Update README**
   - Add live demo link
   - Add deployment badge

4. **Share**
   - Update your portfolio
   - Share on LinkedIn/Twitter
   - Include in job applications

---

## 📞 Support

If you encounter issues:
1. Check the [Known Issues](README.md#known-issues) section
2. Review browser console for errors
3. Verify all dependencies are installed correctly
4. Ensure you're using Node.js 18+

Happy deploying! 🚀

