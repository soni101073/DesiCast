# DesiCast — Indian Podcast Player Blueprint

> **Goal**: Build a standalone, single-file HTML/CSS/JS podcast player focused on Indian podcasts in Hindi and English. Premium dark UI, ad-free playback, folder-based library. Deploy to GitHub Pages.

---

## Architecture

| Layer | Technology | Details |
|---|---|---|
| **UI** | Vanilla HTML + CSS | Single file, no frameworks |
| **Logic** | Vanilla JavaScript | Inline `<script>` block |
| **Search** | iTunes Search API | Free, no API key required |
| **Podcast Feed** | RSS/XML parsing | Fetch podcast RSS feed → parse episodes |
| **Playback** | HTML5 `<audio>` element | Direct MP3 playback — no iframes, no restrictions |
| **Storage** | `localStorage` | Folders, saved episodes, preferences |
| **Hosting** | GitHub Pages | Static `.html` file |

---

## API Integration

### 1. Search Podcasts (iTunes Search API)

**Endpoint:**
```
https://itunes.apple.com/search?term={query}&country=IN&media=podcast&limit=25
```

**Response shape (key fields):**
```json
{
  "results": [
    {
      "collectionId": 123456,
      "collectionName": "The Ranveer Show",
      "artistName": "BeerBiceps",
      "artworkUrl600": "https://...600x600bb.jpg",
      "feedUrl": "https://anchor.fm/s/.../podcast/rss",
      "genres": ["Comedy", "Society & Culture"],
      "primaryGenreName": "Comedy",
      "trackCount": 250,
      "country": "IND"
    }
  ]
}
```

**No API key needed. No rate limits for reasonable usage. CORS-friendly.**

### 2. Get Episodes (RSS Feed Parsing)

Each podcast has a `feedUrl` (RSS/XML). Fetch and parse it to get episodes.

**CORS workaround** (RSS feeds may not have CORS headers):
```
https://api.allorigins.win/raw?url={encodeURIComponent(feedUrl)}
```

**Parse the XML** to extract episodes:
```js
const xml = new DOMParser().parseFromString(text, 'text/xml');
const items = xml.querySelectorAll('item');
items.forEach(item => {
  const title = item.querySelector('title')?.textContent;
  const enclosure = item.querySelector('enclosure');
  const audioUrl = enclosure?.getAttribute('url'); // Direct MP3 URL
  const duration = item.querySelector('itunes\\:duration, duration')?.textContent;
  const pubDate = item.querySelector('pubDate')?.textContent;
  const description = item.querySelector('description')?.textContent;
  const image = item.querySelector('itunes\\:image')?.getAttribute('href');
});
```

### 3. Playback (HTML5 Audio)

```js
const AUD = new Audio();
AUD.src = episode.audioUrl; // Direct MP3 — just works
AUD.play();
```

**No YouTube, no iframes, no embedding restrictions. Podcast MP3 URLs are public.**

---

## Features

### Tab 1: Home
- **Header**: "DesiCast" branding with 🇮🇳 accent
- **Language Toggle**: Hindi / English switch (filters all content)
- **Quick Play Grid**: 6 curated podcast tiles:
  | Podcast | Search Query | Gradient |
  |---|---|---|
  | The Ranveer Show | "The Ranveer Show" | Purple |
  | Aaj Tak Radio | "Aaj Tak podcast" | Red |
  | Sadhguru | "Sadhguru podcast" | Blue |
  | Tanmay Bhat | "Honestly by Tanmay Bhat" | Green |
  | The Diary of a CEO | "Diary of a CEO" | Amber |
  | Crime Kahaniyan | "Crime Kahaniyan Hindi" | Violet |
- **Featured Banner**: "Discover Indian Podcasts" → links to Search tab

### Tab 2: Search
- Search bar with debounce (300ms)
- Results show: podcast artwork, name, author, episode count, genre
- Tap a podcast → opens **Podcast Detail View**:
  - Podcast artwork (large), name, author, description
  - Episode list (newest first): title, date, duration
  - Tap episode → plays it
  - Save button → add episode to a folder

### Tab 3: Library (Folder System)
- **Create Folder** button (prompt for name)
- Folder list: folder icon, name, episode count, delete button
- Tap folder → shows saved episodes
  - Play All, Back button, delete individual episodes
- All data persisted in `localStorage`

### Mini Player (bottom bar)
- Shows: episode artwork, title, podcast name
- Play/pause button
- Tap to expand **Full Player**

### Full Player (slide-up overlay)
- Large artwork
- Episode title + podcast name
- Seek bar with current time / duration
- Controls: previous, play/pause, next (within folder or search results)
- Speed control: 0.5x, 1x, 1.25x, 1.5x, 2x
- Save to folder button (♡ / folder icon)
- Swipe down to close

---

## Design System

### Colors (Dark Theme)
```css
:root {
  --bg: #000;
  --c1: #111;      /* card background */
  --c2: #1a1a1a;   /* subtle bg */
  --c3: #2a2a2a;   /* borders */
  --ac: #ff6b35;   /* accent — saffron/orange (Indian flag inspired) */
  --ac2: #138808;  /* secondary accent — green */
  --tx1: #fff;     /* primary text */
  --tx2: #ccc;     /* secondary text */
  --tx3: #777;     /* muted text */
}
```

### Typography
```css
font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
```

### Layout
- **Mobile-first** flex column: views → mini player → tab bar
- No `position: fixed` for mini/tabs (use flex layout like SONORA)
- Views scroll independently with `overflow-y: auto; flex: 1`
- Tab bar: Home | Search | Library (3 tabs)

### Animations
- Mini player: slide-up on first play
- Full player: CSS transform slide-up/down
- Tab switches: opacity fade
- Buttons: scale on active (0.95)
- Loading: ring spinner

---

## File Structure

**Single file**: `DesiCast.html` (also copy as `index.html` for GitHub Pages)

```
DesiCast.html
├── <head>: meta, title, PWA manifest (inline), favicon (inline base64)
├── <style>: All CSS (~200 lines)
├── <body>: 
│   ├── Welcome screen (optional skip)
│   ├── Shell (flex column)
│   │   ├── Views container
│   │   │   ├── Home view (tiles, featured)
│   │   │   ├── Search view (search bar, results, podcast detail)
│   │   │   └── Library view (folders, folder detail)
│   │   ├── Mini player
│   │   ├── Full player (overlay)
│   │   ├── Tab bar
│   │   └── Folder picker modal
│   └── Toast notification
├── <script>: All JavaScript (~400 lines)
│   ├── Constants (icons SVGs)
│   ├── State object
│   ├── iTunes API search function
│   ├── RSS feed parser
│   ├── CORS proxy handler
│   ├── Audio player (HTML5 Audio)
│   ├── Quick play tiles
│   ├── Search + podcast detail
│   ├── Folder CRUD (localStorage)
│   ├── Folder picker modal
│   ├── UI update functions
│   ├── Tab switching
│   ├── Toast
│   └── Service worker (inline, for PWA)
```

---

## Key Implementation Notes

### CORS for RSS Feeds
Podcast RSS feeds often don't include CORS headers. Use a free CORS proxy:
```js
const CORS_PROXIES = [
  'https://api.allorigins.win/raw?url=',
  'https://corsproxy.io/?',
];
```
Try the first proxy; if it fails, try the next.

### Audio Playback Events
```js
const AUD = new Audio();
AUD.addEventListener('timeupdate', updateSeekBar);
AUD.addEventListener('loadedmetadata', updateDuration);
AUD.addEventListener('ended', playNext);
AUD.addEventListener('play', () => { S.playing = true; updPlayUI(); });
AUD.addEventListener('pause', () => { S.playing = false; updPlayUI(); });
AUD.addEventListener('error', handleError);
```

### Speed Control
```js
function setSpeed(rate) { AUD.playbackRate = rate; }
// Options: [0.5, 0.75, 1, 1.25, 1.5, 2]
```

### Podcast Detail View
When user taps a podcast from search results:
1. Show loading spinner
2. Fetch the `feedUrl` via CORS proxy
3. Parse XML → extract episodes (title, audioUrl, duration, date, image)
4. Render episode list
5. Tap episode → `AUD.src = episode.audioUrl; AUD.play();`

### Episode Data Model
```js
{
  id: 'ep_' + index,
  title: 'Episode Title',
  podcast: 'Podcast Name',
  audioUrl: 'https://...mp3',
  duration: 3600, // seconds
  date: '2026-01-15',
  thumb: 'https://...artwork.jpg',
  description: 'Episode description...'
}
```

---

## Reference: SONORA (sister project)
DesiCast follows the same architecture as [SONORA](https://github.com/soni101073/sonora) — a YouTube-based music player. Reuse the same:
- Flex shell layout (views → mini → tabs)
- Folder system (create, save, delete)
- Folder picker modal
- Full player slide-up/down
- Toast notifications
- PWA service worker pattern
- Dark theme design tokens

**Key difference**: SONORA uses YouTube IFrame API for audio. DesiCast uses HTML5 `<audio>` with direct MP3 URLs — **much simpler and more reliable**.

---

## Deployment
1. Save as `index.html`
2. Git init, commit, push to GitHub
3. Enable GitHub Pages (Settings → Pages → main branch → root)
4. Live at: `https://github.com/soni101073/DesiCast/`
