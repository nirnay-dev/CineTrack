# CineTrack 🎬 — Personal Movie & Series Watchlist Tracker

A sleek, serverless, single-file web application to track movies and TV series with real-time metadata, cast headshots, franchise batch exploration, and 2-way Google Sheets database synchronization.

Live Demo: [https://nirnay-dev.github.io/CineTrack/](https://nirnay-dev.github.io/CineTrack/)

---

## ✨ Features

- **Minimalist Aesthetic Grid**: Card fronts display clean poster art, region badges, and rewatch indicators.
- **IMDb & Netflix Style Cast Carousel**: Clicking any title opens a detailed modal with top-billed circular cast photos, character roles, runtime, and storylines.
- **Click-to-Filter by Actor**: Click any actor's portrait to view all movies or shows in your watchlist featuring that performer.
- **Industry & Regional Cinema Filtering**:
  - 🎬 Hollywood (English / Global)
  - 🇮🇳 Bollywood (Hindi)
  - 🌴 South Indian Cinema (Telugu, Tamil, Malayalam, Kannada)
  - 🇯🇵 Anime / Japanese
  - 🇰🇷 K-Drama / Korean
- **Dynamic In-App Sorting**:
  - Default (Google Sheet row order)
  - Title (A &rarr; Z and Z &rarr; A)
  - Release Year (Newest & Oldest)
  - Most Rewatched
- **Live Search & Confirmation Preview**: Type any movie or TV series name with instant poster previews, rating score (★), and synopsis confirmation before adding.
- **Franchise & Universe Batch Explorer**:
  - One-click presets for **Marvel (MCU)**, **Batman / DC**, **Star Wars**, **Harry Potter**, **Lord of the Rings**, and **Fast & Furious**.
  - Search any collection or director (e.g. *Spider-Man*, *Christopher Nolan*, *Pixar*).
  - Automatically identifies titles already in your sheet with a green **✓ Watched** badge.
  - Multi-select unwatched movies and batch add them directly to your sheet in one click.
- **Dual TMDB Mirror Resolution**: Automatically switches between `api.tmdb.org` and `api.themoviedb.org` to bypass regional ISP/DNS restrictions.
- **Rewatch Counter & Time Tracker**: Keep track of rewatches and automatically calculate your total hours watched.
- **Deduplication Engine**: Prevents adding duplicates by incrementing rewatch counts instead.

---

## 🚀 Quick Setup & Deployment

Because CineTrack is a **100% client-side single file (`index.html`)**, you can deploy it on GitHub Pages in under a minute.

### 1. Host on GitHub Pages
1. Fork or push `index.html` to your repository.
2. Go to **Settings** &rarr; **Pages**.
3. Under **Build and deployment**, set Source to **Deploy from a branch** and select `main` / `root`.
4. Your site will be live at `https://<username>.github.io/<repo>/`!

---

## 📊 Google Sheets Backend Integration

CineTrack uses Google Sheets as its free cloud database via Google Apps Script.

### Step 1: Create Your Google Sheet
1. Create a new Google Sheet.
2. In Row 1, set the following headers:
   - Column A: `title`
   - Column B: `tmdb_id`
   - Column C: `type`
   - Column D: `rewatches`

### Step 2: Deploy Google Apps Script
1. Inside your sheet, open **Extensions** &rarr; **Apps Script**.
2. Replace all script code with the following:

```javascript
function doGet() {
  const sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
  const rows = sheet.getDataRange().getValues();
  const headers = rows.shift(); // Remove header row
  const data = rows.map(r => ({
    title: r[0],
    tmdb_id: r[1],
    type: r[2] || 'movie',
    rewatches: Number(r[3] || 1)
  }));
  return ContentService.createTextOutput(JSON.stringify({ status: 'success', data }))
    .setMimeType(ContentService.MimeType.JSON);
}

function doPost(e) {
  const sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
  const body = JSON.parse(e.postData.contents);
  
  if (body.action === 'delete') {
    const rows = sheet.getDataRange().getValues();
    for (let i = 1; i < rows.length; i++) {
      if (Number(rows[i][1]) === Number(body.tmdb_id) || rows[i][0] === body.title) {
        sheet.deleteRow(i + 1);
        break;
      }
    }
  } else {
    sheet.appendRow([body.title, body.tmdb_id, body.type, body.rewatches || 1]);
  }
  
  return ContentService.createTextOutput(JSON.stringify({ status: 'success' }))
    .setMimeType(ContentService.MimeType.JSON);
}
```

3. Click **Deploy** &rarr; **New deployment**.
4. Set type to **Web app**:
   - **Execute as**: `Me`
   - **Who has access**: `Anyone`
5. Copy the generated Web App URL and set it as `DEFAULT_SHEET_URL` inside `index.html`.

---

## 🛠️ Tech Stack

- **Frontend**: Vanilla JavaScript (ES6+), HTML5, Tailwind CSS
- **APIs**: The Movie Database (TMDB v3), Google Apps Script Web App API
- **Fonts**: Inter & Outfit (Google Fonts)

---

## 📄 License & Attribution

This product uses the TMDB API but is not endorsed or certified by TMDB.
Released under the MIT License.