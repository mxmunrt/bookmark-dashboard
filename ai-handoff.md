# AI Agent Handoff Summary: Bookmark Dashboard

This document provides all the context needed to resume work on the **Bookmark Dashboard** project.

## 📌 Project Overview
- **Type**: Single-Page Application (SPA)
- **Architecture**: A single `index.html` file containing HTML, vanilla CSS, and vanilla ES6 JavaScript. **No build step, no npm, no bundler.**
- **Deployment**: Hosted on Vercel at `https://bookmarks.mrtalukdar.dev` (auto-deploying from GitHub `main` branch).
- **Goal**: A personal bookmark manager that syncs across devices for authenticated users, with a local-storage fallback for guests.

## 🛠️ Tech Stack & Dependencies
- **UI/UX**: Vanilla HTML5, CSS3, and JavaScript. Uses Material Design Icons via CDN.
- **Backend & Auth**: Firebase JS SDK (v11.4.0) via CDN ES Module imports.
  - **Firebase Auth**: Configured for Google Sign-in provider.
  - **Firebase Firestore**: Stores user bookmarks and custom tag icons.

## ✨ Core Features Implemented
1. **Bookmark Management**: Add, edit, delete, tag, and categorize bookmarks.
2. **Cloud Sync**: 
   - Signed-in users read/write their bookmarks and tag icons directly to their protected Firestore document (`/users/{uid}`).
3. **Guest Mode & Migration**:
   - Unauthenticated users save data to `localStorage`.
   - On their *first* sign-in, any local bookmarks are automatically migrated to Firestore.
   - Brand new accounts are pre-populated with 12 default developer-focused bookmarks.
4. **Dark Mode**: Toggleable and persisted via `localStorage` per-device.

## 🐞 Current Blocker / Status
**The Google Sign-in flow is failing in production (Vercel)**. The user is redirected to Google, selects their account, and is redirected back to the dashboard, but the authentication state is not retained (`window.firebaseOnAuthStateChanged` triggers with `user = null`).

### What we've tried / Debugging context:
1. **Firebase Config & GCP**:
   - The original implementation used `signInWithPopup`. This failed in production due to a `Cross-Origin-Opener-Policy` (COOP) error blocking the popup window from communicating back to the main Vercel `same-origin` window.
   - We switched to `signInWithRedirect` and `getRedirectResult()` to bypass the popup blocker.
   - **GCP Configuration**: `mrtalukdar.dev` and the Vercel URL are added to the Authorized Domains in both Firebase Console and the Google Cloud API Credentials (HTTP Referrer restrictions are in place).

### Your First Task
**Debug the `signInWithRedirect` flow inside `index.html`.** 
Why is the user returning to the app without a valid session?
- Check if third-party cookie restrictions in modern browsers are breaking the redirect flow on the custom domain.
- Consider if Firebase Auth needs to be initialized differently, or if we need to switch from authDomain `*.firebaseapp.com` to the custom domain to prevent cross-origin cookie drops.
- Review the `init()` function and `getRedirectResult()` placement in the `<script type="module">` at the bottom of `index.html`.
