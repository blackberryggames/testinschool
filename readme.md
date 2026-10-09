# AniWatch

A self-hosted anime streaming site. Search with AniList, stream via AllAnime (megacloud/anikoto), all proxied through a Cloudflare Worker so HLS segments work from the browser.

```
┌─────────────────────┐     ┌────────────────────────────────┐
│  GitHub Pages       │────▶│  Cloudflare Worker             │
│  index.html         │     │  worker.js                     │
│  (your frontend)    │◀────│                                │
└─────────────────────┘     │  /anilist/search  → AniList   │
                             │  /anilist/details → AniList   │
                             │  /allanime/search → AllAnime  │
                             │  /allanime/episodes           │
                             │  /allanime/sources            │
                             │  /proxy/m3u8  ──▶ rewrite    │
                             │  /proxy/seg   ──▶ pass-thru  │
                             └────────────────────────────────┘
```

---

## Step 1 — Deploy the Cloudflare Worker

### Option A: Wrangler CLI (recommended)

1. Install Wrangler:
   ```bash
   npm install -g wrangler
   wrangler login
   ```

2. Create a new worker project:
   ```bash
   mkdir aniwatch-worker && cd aniwatch-worker
   ```

3. Copy `worker.js` into the folder. Create `wrangler.toml`:
   ```toml
   name = "aniwatch-worker"
   main = "worker.js"
   compatibility_date = "2024-01-01"
   ```

4. Deploy:
   ```bash
   wrangler deploy
   ```

5. Copy the Worker URL — it looks like:
   `https://aniwatch-worker.YOUR-SUBDOMAIN.workers.dev`

### Option B: Cloudflare Dashboard (no CLI)

1. Go to [dash.cloudflare.com](https://dash.cloudflare.com) → **Workers & Pages** → **Create**.
2. Click **Create Worker** → give it a name → **Deploy**.
3. Click **Edit code**, paste the contents of `worker.js`, then **Save and deploy**.
4. Copy the URL shown at the top.

---

## Step 2 — Configure the frontend

Open `index.html` and find this line near the top of the `<script>` block:

```js
const WORKER = 'https://YOUR_WORKER.YOUR_SUBDOMAIN.workers.dev';
```

Replace it with your real Worker URL from Step 1.

---

## Step 3 — Host on GitHub Pages

1. Create a new GitHub repository (can be private or public).
2. Drop `index.html` (and optionally `README.md`) into the root of the repo.
3. Go to **Settings → Pages → Source** → select **main branch / root**.
4. GitHub will give you a URL like `https://yourusername.github.io/your-repo/`.

That's it — open the URL and start watching.

---

## Features

| Feature | Details |
|---------|---------|
| Search | AniList GraphQL (no key required) |
| Metadata | Titles, cover art, synopsis, genres, score, studio |
| Episode list | Via AllAnime API (sub + dub) |
| Streams | AllAnime → megacloud/anikoto HLS sources |
| HLS proxy | Cloudflare Worker rewrites all segment/key URLs |
| Quality selector | Shows available resolutions from hls.js levels |
| Language | Sub / Dub toggle per show |
| Captions | Toggle on/off (when the stream includes subtitle tracks) |
| Watch history | Saved to localStorage (watched episodes dimmed) |
| Player | Native `<video>` + hls.js for browsers without native HLS |

---

## Troubleshooting

**"Could not load trending" / CORS errors**
- Your `WORKER` URL in `index.html` is wrong or not deployed yet.

**Episodes not found**
- AllAnime's persisted query hashes rotate occasionally. If episodes stop loading, check the latest hashes from projects like [yorumi-cli](https://github.com/davenarchives/yorumi-cli) or [ani-cli-rs](https://github.com/vorlie/ani-cli-rs) and update the `*_HASH` constants at the top of `worker.js`.

**Video plays audio but no picture / black screen**
- Try a different source chip. Some AllAnime sources are embeds, not direct HLS.

**Worker rate-limited**
- Cloudflare's free Workers tier handles 100,000 requests/day. For personal use this is plenty; add KV caching if you need more.

---

## Updating query hashes

The AllAnime API uses persisted GraphQL queries with SHA-256 hashes. These occasionally change. In `worker.js`:

```js
const SEARCH_HASH   = 'c4305f3918591071...';   // ← update if search breaks
const EPISODES_HASH = 'c8f3ac51f598e630...';   // ← update if episodes break
const SOURCES_HASH  = '5f1a64b73793cc22...';   // ← update if sources break
```

To find current hashes, look at the network tab on allanime.to, or check the source of ani-cli-rs / yorumi-cli.

---

## License

Personal/educational use. AniList data is under their [API Terms](https://anilist.gitbook.io/anilist-apiv2-docs/). AllAnime content belongs to its respective rights holders.
