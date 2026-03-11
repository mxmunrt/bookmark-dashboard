# Bookmark Dashboard

A sleek, self-hosted bookmarking single-page application (SPA) focused on clean design, categorization, and cross-device synchronization.

**Live Demo / Hosted Version**: This application is already deployed and ready to use at [bookmarks.mrtalukdar.dev](https://bookmarks.mrtalukdar.dev). If you don't want to self-host, you can simply use this hosted version!

## Features

- **Google Sign-In**: Secure authentication using Firebase so your data is tied to your account.
- **Cloud Sync**: Bookmarks and custom tag icons are instantly saved to Firestore.
- **Offline Fallback**: If an unauthenticated user uses the app, data is saved locally via `localStorage`. 
- **Smart Migration**: Automatically turns local bookmarks into cloud-synced bookmarks when a returning user signs in for the first time.
- **Dark Mode**: Toggleable dark mode preference saved per-device.
- **Zero Build Step**: Native HTML, CSS, and vanilla JavaScript using CDN imports — no `npm`, Webpack, or Vite needed to run.

## Setup & Deployment

1. **Fork or copy** this repository.
2. **Deploy to Vercel**: 
   - Go to [Vercel](https://vercel.com/) and click "Add New Project"
   - Import this GitHub repository
   - Leave build settings empty (it's just a static HTML file) and click **Deploy**.

## Firebase Configuration

This app requires a Firebase project for authentication and database synchronization.

1. Create a project in the [Firebase Console](https://console.firebase.google.com/).
2. Enable **Authentication** and turn on the **Google** sign-in provider.
3. **IMPORTANT**: Go to Authentication -> Settings -> **Authorized domains** and add your live Vercel URL (e.g., `yourapp.vercel.app`).
4. Create a **Firestore Database** and add the following Security Rules:
   ```javascript
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       match /users/{userId}/{document=**} {
         allow read, write: if request.auth != null && request.auth.uid == userId;
       }
     }
   }
   ```
5. Grab your Firebase Web Config object and replace the `firebaseConfig` section inside [`index.html`](index.html).

## Local Development

Since this project uses ES Module imports (`<script type="module">`), you cannot open the `index.html` file directly in your browser using the `file://` protocol. 

To run it locally, serve it over HTTP:

```bash
# Using Python
python3 -m http.server 8080

# Using Node.js
npx serve
```
Then visit `http://localhost:8080`.

---
**Built with ❤️ by [mrtalukdar.dev](https://mrtalukdar.dev)**
