# Grok Imagine – Bulk Favorites Downloader

A Tampermonkey userscript that bulk-downloads all of your saved/favorited images and videos from [grok.com/imagine](https://grok.com/imagine). It remembers what it has already downloaded so future runs only grab new ones.

---

## Features

- **Bulk downloads all favorites** — no limit, grabs everything
- **Chunked downloading** — downloads in batches of 200, pausing between each so your browser doesn't get overwhelmed
- **Download history** — remembers every downloaded file by ID so repeat runs only fetch new favorites
- **Save IDs only mode** — mark your current favorites as "already have" without downloading any files (useful for setting a baseline)
- **Re-download everything mode** — ignore history and grab everything fresh
- **No page interference** — does not patch `window.fetch` so Grok's own page loads and favorites grid work normally
- **Works with images and videos** — downloads `.jpg`, `.png`, `.webp`, and `.mp4` files
- **Smart filenames** — files are named `YYYY-MM-DD_HHMM_ID_model_prompt.ext` so they're easy to sort and search
- **Persistent history** — download history survives page reloads and browser restarts via Tampermonkey storage

---

## Requirements

- [Tampermonkey](https://www.tampermonkey.net/) browser extension (Chrome, Firefox, Edge, Safari)
- A [grok.com](https://grok.com) account with saved favorites

---

## Installation

1. Install [Tampermonkey](https://www.tampermonkey.net/) for your browser if you haven't already
2. Click the downloaded `grok-imagine-bulk-download.user.js` file — Tampermonkey should automatically prompt you to install it
3. Alternatively, open the Tampermonkey dashboard → click **Create new script** → paste the entire script contents → save

---

## How to Use

1. Go to **[grok.com/imagine/favorites](https://grok.com/imagine/favorites)**
2. Make sure the page has fully loaded
3. You'll see a **"⬇ Download New Favorites"** button in the bottom-right corner of the page
4. Click it — a modal will appear asking what you'd like to do

### Modal Options

| Option | What it does |
|---|---|
| **⬇ Download new favorites** | Fetches all favorites, skips ones already downloaded, downloads the rest in chunks of 200 |
| **🔖 Save IDs only (no download)** | Marks all current favorites as "already have" without downloading any files |
| **⬇ Download everything (ignore history)** | Re-downloads all favorites regardless of history |

5. Files are saved to your browser's default **Downloads** folder under a `grok-favorites/` subfolder

> **Tip:** Disable "Ask where to save each file" in your browser's download settings, otherwise you'll get a save dialog for every single file.

---

## How It Works

### API Approach
Rather than trying to scroll the page and scrape visible images (which doesn't work because Grok uses a virtualized grid that only renders ~10 cards at a time), the script directly calls Grok's internal REST API:

```
POST https://grok.com/rest/media/post/list
Body: { limit: 40, filter: { source: "MEDIA_POST_SOURCE_LIKED" }, cursor: "..." }
```

This is the same endpoint Grok's own website uses to load your favorites. Because the script runs inside your logged-in browser session, your session cookies are sent automatically — no API key or login needed.

### Pagination
The API returns 40 items per page along with a `nextCursor` token. The script keeps calling the API with the cursor from each response until no cursor is returned, meaning all pages have been fetched. There is no limit on how many pages it will fetch.

### Chunked Downloading
Once all favorites are collected, they are split into chunks of 200 and downloaded sequentially. After each chunk finishes being queued, the script pauses for 5 seconds to give the browser time to process the downloads before starting the next chunk. This prevents the browser from being overwhelmed with hundreds of simultaneous download requests.

### Download History
Every successfully queued file's ID is saved to Tampermonkey's persistent storage (`GM_setValue`). On future runs the script fetches the full favorites list from the API but filters out any IDs already in history before downloading. History is saved incrementally during downloading so if the browser crashes mid-run, already-queued files won't be re-downloaded next time.

### No Page Interference
Earlier versions of the script patched `window.fetch` to intercept auth tokens, which accidentally broke Grok's page loading. The current version uses `GM_xmlhttpRequest` instead — this runs in Tampermonkey's own isolated context and cannot interfere with the page's own network requests.

---

## File Naming

Downloaded files are named using this format:

```
grok-favorites/YYYY-MM-DD_HHMM_XXXXXXXX_modelname_your-prompt-here.jpg
```

For example:
```
grok-favorites/2025-03-01_1430_a1b2c3d4_grok-2_a-cat-wearing-a-spacesuit.jpg
```

This makes files easy to sort by date, search by prompt, and identify by model.

---

## Resetting Download History

If you want to re-download everything from scratch (e.g. you got a new computer, files were lost, or you just want a fresh start):

- Click the **"🗑 Reset download history"** link underneath the download button
- Confirm the prompt
- The next run will treat all favorites as new and download everything again

Alternatively, you can select **"⬇ Download everything (ignore history)"** from the modal for a one-time full re-download without permanently clearing the history.

---

## Troubleshooting

**Button doesn't appear**
Make sure you are on `grok.com/imagine` or `grok.com/imagine/favorites`. The script only runs on those pages.

**"API request failed" error**
Make sure you are logged into grok.com. Try refreshing the page and clicking the button again.

**Only getting 0 items**
Refresh the page, wait for your favorites grid to fully load, then click the button.

**Downloads going to wrong folder**
The script saves to a `grok-favorites/` subfolder inside your browser's default downloads location. To change the base downloads location, update it in your browser settings.

**Want to check what's happening under the hood**
Open DevTools (F12) → Console tab. The script logs everything prefixed with `[GrokDL]` including API responses, page counts, and any errors.

---

## Privacy

This script runs entirely inside your browser. It only communicates with `grok.com`, `assets.grok.com`, and `x.ai` to fetch your own media. No data is sent to any third-party server.

---

## Disclaimer

This is an unofficial tool not affiliated with Grok or X. It uses Grok's internal API which may change at any time. Use at your own risk.
