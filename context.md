# AI Agent Handoff Summary: Bookmark Dashboard

This document provides all the context needed to resume work on the **Bookmark Dashboard** project.

## Project Overview
- **Type**: Single-Page Application (SPA)
- **Architecture**: A single `index.html` file containing HTML, vanilla CSS, and vanilla ES6 JavaScript. **No build step, no npm, no bundler.**
- **Deployment**: Hosted on Vercel at `https://bookmarks.mrtalukdar.dev` (auto-deploying from GitHub `main` branch).
- **Goal**: A personal bookmark manager that syncs across devices for authenticated users, with a local-storage fallback for guests.

## Tech Stack & Dependencies
- **UI/UX**: Vanilla HTML5, CSS3, and JavaScript. Uses Material Design Icons via CDN.
- **Backend & Auth**: Firebase JS SDK (v11.4.0) via CDN ES Module imports.
  - **Firebase Auth**: Google Sign-in provider, using `signInWithRedirect` + `getRedirectResult`.
  - **Firebase Firestore**: Stores user bookmarks and custom tag icons at `/users/{uid}/data/bookmarks` and `/users/{uid}/data/tagIcons`.

## Firebase Config
```javascript
const firebaseConfig = {
  apiKey: "AIzaSyBRocQivNxXpXUJ9PA1DjgUbU_MijLCcFI",
  authDomain: "bookmarks.mrtalukdar.dev",   // custom domain — intentional, see Auth Fix below
  projectId: "bookmark-dashboard-71eac",
  storageBucket: "bookmark-dashboard-71eac.firebasestorage.app",
  messagingSenderId: "767430128890",
  appId: "1:767430128890:web:b597ada83ec777bf98c543",
  measurementId: "G-T9XHMH185X"
};
```

## Core Features (All Implemented & Working)
1. **Bookmark Management**: Add, edit, delete, tag, and categorize bookmarks.
2. **Cloud Sync**: Signed-in users read/write their bookmarks and tag icons directly to their protected Firestore document.
3. **Guest Mode & Migration**: Unauthenticated users save to `localStorage`. On first sign-in, local bookmarks are auto-migrated to Firestore. New accounts are pre-populated with 12 default developer-focused bookmarks.
4. **Dark Mode**: Toggleable and persisted via `localStorage` per-device.
5. **Google Sign-In**: Fully working in production. Auth state displays user avatar + name in header with a sign-out button.
6. **Custom Delete Confirmation Modal**: Styled in-app modal (no `window.confirm()`).
7. **Meatball Menu on Cards**: Each bookmark card has a `⋮` button (top-right, inline with title) that reveals a styled dropdown with Edit (pencil icon) and Delete (trash icon, red) actions.

## Auth Architecture — Critical Details

### The Problem That Was Solved
`signInWithRedirect` redirected successfully but `onAuthStateChanged` always fired with `user = null`.

**Root cause**: `<script type="module">` is implicitly deferred — it runs **after** the full HTML parse and after the inline `<script>` at the end of `<body>`. When `init()` called `window.firebaseGetRedirectResult(...)`, the module hadn't executed yet, so `window.firebaseGetRedirectResult` was `undefined`. The TypeError silently aborted execution, meaning `onAuthStateChanged` was never registered.

**Fix**: Moved `getRedirectResult` and `onAuthStateChanged` calls **into the module script itself** (inside `<head>`), not inside `init()`. The module now handles all auth initialization directly.

### The Custom `authDomain` + Vercel Proxy Pattern
- **Problem**: `authDomain: "bookmark-dashboard-71eac.firebaseapp.com"` caused modern browsers (Chrome Privacy Sandbox, Safari ITP) to block the cross-site cookie/state transfer from `.firebaseapp.com` back to `.mrtalukdar.dev`.
- **Fix**:
  1. `authDomain` changed to `"bookmarks.mrtalukdar.dev"` (the custom domain).
  2. `vercel.json` created to proxy Firebase auth handler routes under the custom domain:
     ```json
     {
       "rewrites": [
         { "source": "/__/auth/:path*", "destination": "https://bookmark-dashboard-71eac.firebaseapp.com/__/auth/:path*" },
         { "source": "/__/firebase/:path*", "destination": "https://bookmark-dashboard-71eac.firebaseapp.com/__/firebase/:path*" }
       ]
     }
     ```
  3. `https://bookmarks.mrtalukdar.dev/__/auth/handler` added as an **Authorized redirect URI** in GCP OAuth 2.0 client credentials.
  4. `bookmarks.mrtalukdar.dev` added to Firebase Console > Authentication > Authorized Domains.

## Script Execution Order (Important)
```
<head>
  <script type="module">   ← Firebase SDK, config, globals, getRedirectResult, onAuthStateChanged
  </script>
</head>
<body>
  ...HTML...
  <script>                 ← App logic (init, renderCards, etc.) — runs FIRST during parse
  </script>
</body>
```
The module script runs **after** the inline body script. Any Firebase calls must live in the module, not in `init()`.

## Card Meatball Menu — Implementation

### How it works
- Each card has a `.card-menu` wrapper (relative positioned, `margin-left: auto`) in `.card-header`, inline with the title.
- A `⋮` button (`.card-menu-btn`) toggles a hidden dropdown (`.card-menu-dropdown`).
- Clicking the button calls `toggleCardMenu(event, id)` which calls `e.stopPropagation()` to prevent immediate closure, closes any open menus, then opens the target dropdown by adding class `open`.
- A `document.addEventListener('click', closeCardMenus)` registered in `init()` closes all dropdowns on any outside click.

### Dropdown items
- **Edit**: `mdi-pencil` icon + "Edit" label → calls `closeCardMenus(); openModal(id)`
- **Delete**: `mdi-delete` icon + "Delete" label (red) → calls `closeCardMenus(); deleteBookmark(id)`

## Custom Delete Confirmation Modal
- HTML element: `<div class="overlay" id="confirm-overlay">` before the add/edit modal.
- JS: `confirmDelete(message, onConfirm)` shows modal and stores callback; `confirmProceed()` executes it; `confirmCancel()` dismisses.
- `deleteBookmark(id)` calls `confirmDelete(...)` instead of `window.confirm()`.

## Key Files
- `index.html` — entire app (HTML, CSS, JS)
- `vercel.json` — Vercel rewrites for Firebase Auth handler proxy
- `context.md` — this file

## Current Status
**Everything is working.** The app is live at `https://bookmarks.mrtalukdar.dev`. All features described above are implemented and pushed to `main`. No known bugs or pending tasks.
