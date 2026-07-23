# AEL_Webapp
Vibe code webapp be like BRuhhhhhhhhhhh🗣️🗣️🔥🔥💯

### 📄 `README.md` (Copy & Paste to Repository Root)

```markdown
# 🛠️ AEL Maintenance SPA (Offline-First PWA)

A lightweight, single-page Progressive Web App (PWA) designed for field technicians
to conductmaintenance inspections, scan QR codes, take photos,
and log records—**even in environments with zero internet connectivity**. 

The app stores data locally using **IndexedDB** and automatically
syncs with **Google Sheets** and **Google Drive** (generating Word `.docx` documents)
via **Google Apps Script (GAS)** when a connection is restored.

---

## 📁 Repository File Structure

To ensure the Progressive Web App (PWA) functions properly,
all files must reside in the **root directory** (`/`) of your repository:

```text
/ (Repository Root)
├── index.html        # Main SPA UI, styles, and core JavaScript logic
├── sw.js            # Service Worker for offline caching & CDN preloading
└── README.md        # Project documentation

```

---

## 🏗️ System Architecture & Workflow

```text
[ Offline Field Operation ]
       │
       ▼
┌──────────────┐
│  sw.js (SW)  │ ──> Caches HTML/CSS/JS & QR Code CDN for offline rendering
└──────────────┘
       │
       ▼
┌──────────────┐
│ IndexedDB    │ ──> Saves records & Base64 photos locally (synced: false)
└──────────────┘
       │
 [ Connection Restored / Manual Sync ]
       │
       ▼
┌──────────────┐
│ GAS Web App  │ ──> Receives POST requests (batch / single sync)
└──────────────┘
       │
       ├───> Writes metadata to Google Sheets
       ├───> Saves photos to Google Drive
       └───> Populates Google Docs template & exports as Word (.docx)

```

---

## 📖 Component Breakdown

### 1. `index.html` (The Web Application)

* **Screen Navigation**: Simple single-page layout switching (`#home`, `#service`, `#camera`, `#library`, `#data`, `#settings`).
* **QR Code Scanner**: Powered by `html5-qrcode`. Automatically populates `Site` and `Machine` dropdowns upon scanning a QR payload formatted as `ProjectSite, MachineName`.
* **IndexedDB Local Database (`AEL_MaintenanceDB`)**:
* `projectSites`: Stores site names locally.
* `machines`: Stores machine information.
* `photos`: Stores maintenance records, Base64 compressed images, timestamps, and `synced` status (`true`/`false`).


* **`/clear-all` Protection**: The settings page requires users to explicitly type `/clear-all` in a text prompt to wipe local IndexedDB stores, preventing accidental data loss.

### 2. `sw.js` (Service Worker)

* **Offline Interceptor**: Acts as a client-side proxy server.
* **Pre-caching**: Caches `index.html` and the external QR Code library CDN (`unpkg.com`) during the `install` phase so the web app can boot without internet.

---

## ⚠️ Important Things to Know When Modifying

### 1. Modifying JavaScript or Dependencies

* **Service Worker Cache Invalidation (`sw.js`)**:
Whenever you update code in `index.html` or change CSS/JS dependencies, you **MUST increment the cache version** in `sw.js`:
```javascript
// Change 'v1' to 'v2' when making code updates
const CACHE_NAME = 'ael-maintenance-v2'; 

```


*Why?* If you don't update the cache name, returning users will continue loading the old version stored in their browser cache.
* **CDN Dependencies & Offline Mode**:
If you add new external CSS/JS libraries via `<script src="...">` or `<link href="...">`, you **must add their URL to `ASSETS_TO_CACHE` in `sw.js**`:
```javascript
const ASSETS_TO_CACHE = [
  './',
  './index.html',
  '[https://unpkg.com/html5-qrcode@2.3.8/html5-qrcode.min.js](https://unpkg.com/html5-qrcode@2.3.8/html5-qrcode.min.js)',
  '[https://cdn.example.com/new-library.min.js](https://cdn.example.com/new-library.min.js)' // Add new CDNs here!
];

```



### 2. Camera & Photo Optimization

* **Canvas Image Compression**:
High-resolution camera photos can crash browser memory if stored directly in IndexedDB. Before saving, images are drawn to a `<canvas>` element and compressed to `JPEG 0.7` quality:
```javascript
const dataUrl = canvas.toDataURL('image/jpeg', 0.7); // Keeps base64 payload under ~300KB

```


*Recommendation*: Do not exceed `0.85` quality or remove compression unless strictly necessary for image fidelity.

### 3. Google Apps Script (GAS) Endpoint Integration

* Update your published Google Apps Script Web App URL inside `index.html`:
```javascript
const GAS_WEB_APP_URL = '[https://script.google.com/macros/s/YOUR_ACTUAL_DEPLOYMENT_ID/exec](https://script.google.com/macros/s/YOUR_ACTUAL_DEPLOYMENT_ID/exec)';

```


* **CORS & HTTP Requests**:
GAS Web Apps automatically handle CORS when receiving `POST` requests formatted as `text/plain` or JSON stringified payloads. Do **not** set headers to `application/json`, or the browser may trigger CORS pre-flight preflight check errors.

### 4. GitHub Pages Deployment Rules

1. Navigate to **Settings** ➔ **Pages** in your GitHub repository.
2. Under **Build and deployment**, set **Source** to `Deploy from a branch`.
3. Select branch: `main` and folder: `/(root)`.
4. Ensure `sw.js` is kept in the **root folder**. Placing `sw.js` in a subfolder (e.g., `/js/sw.js`) will restrict its service worker scope and break offline functionality for root pages.

---

## 🧪 How to Test Offline Functionality

1. Open the app in **Google Chrome** or **Safari**.
2. Open Developer Tools (`F12` or `Cmd+Option+I`) ➔ Go to the **Application** tab ➔ **Service Workers**.
3. Check the **Offline** checkbox.
4. Refresh the page or try capturing photos and scanning QR codes. All records should seamlessly save to IndexedDB.
5. Uncheck **Offline**. The app will detect connection restoration and automatically trigger `syncAllToCloud()`.
