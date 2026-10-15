# Beyond-HD qBittorrent Search Plugin

A qBittorrent search plugin for [Beyond-HD](https://beyond-hd.me/) that uses BHD's official JSON API directly, supporting text, IMDB, and TMDB searches with automatic pagination.

## Requirements

- qBittorrent 4.x or 5.x with the search plugin feature enabled
- Python 3.8+ (Python 3.12+ recommended for qBittorrent 5.x)
- A Beyond-HD account with API access
- `requests` Python library (`pip install requests`)

## Getting your API keys

1. Log in to Beyond-HD
2. Click your profile picture → **My Security**
3. Copy both the **API Key** and **RSS Key** (each is 32 characters)

Both keys are required — the API key authenticates requests, while the RSS key is needed for BHD to include `download_url` in results.

## Installation

### 1. Place the plugin file

Copy `beyondhd.py` into your qBittorrent engines directory:

| Platform | Path |
|---|---|
| **Linux (native)** | `~/.local/share/qBittorrent/nova3/engines/` |
| **Docker (linuxserver/qbittorrent)** | `/config/qBittorrent/data/nova3/engines/` |
| **Windows** | `%AppData%\qBittorrent\nova3\engines\` |
| **macOS** | `~/Library/Application Support/qBittorrent/nova3/engines/` |

### 2. Create the configuration file

Create a `bhd.json` file in the **same directory** as `beyondhd.py`:

```json
{
    "api_key": "your_32_char_api_key",
    "rss_key": "your_32_char_rss_key",
    "url": "[https://beyond-hd.me](https://beyond-hd.me)",
    "sort_by_seeders": true,
    "min_seeders": 0
}
```

### 3. Add the plugin in qBittorrent

1. Open qBittorrent
2. Go to **View → Search Engine** (enable it if not already visible)
3. Click the **Search plugins…** button (bottom right of the Search tab)
4. Click **Install a new one**
5. Select **Local file**
6. Browse to and select `beyondhd.py`
7. Click **Yes** when prompted to install
8. Restart qBittorrent if the plugin doesn't appear immediately

Alternatively, you can install via **Web link** by pasting a raw URL to the `beyondhd.py` file.

**Important:** The filename must be `beyondhd.py` to match the plugin's registered name — qBittorrent will reject it as "not supported" if the filename and plugin name don't align.

## Configuration options

| Option | Type | Default | Description |
|---|---|---|---|
| `api_key` | string | — | Your 32-char BHD API key (required) |
| `rss_key` | string | — | Your 32-char BHD RSS key (required for download links) |
| `url` | string | `https://beyond-hd.me` | BHD base URL (rarely needs changing) |
| `sort_by_seeders` | bool | `true` | Sort results by seeders descending before display |
| `min_seeders` | int | `0` | Filter out results with fewer seeders (0 = no filter) |

### Why sort by seeders?

qBittorrent's search UI displays a maximum of **500 results** per search. Since BHD can return 1000+ results for broad searches, sorting by seeders ensures the 500 displayed are the most relevant/healthy torrents rather than an arbitrary subset.

## Usage

### From qBittorrent

1. Go to the **Search** tab
2. Select **Beyond-HD** from the plugins dropdown
3. Enter your search term and click **Search**

### Supported search formats

| Input | Search type | Example |
|---|---|---|
| Plain text | Name search | `star trek` |
| `tt` prefix | IMDB ID | `tt1375666` |
| `imdb:` prefix | IMDB ID | `imdb:1375666` |
| Full IMDB URL | IMDB ID (auto-extracted) | `https://www.imdb.com/title/tt1375666/` |
| `tmdb:` prefix | TMDB ID (movie) | `tmdb:27208` |
| Full TMDB URL | TMDB ID (auto-detects tv/movie) | `https://www.themoviedb.org/tv/1396` |
| `tvdb:` prefix | Falls back to text search | `tvdb:253138` |

### IMDB link parsing

The plugin automatically extracts the IMDB ID when you paste a full IMDB URL into the search box:

```text
[https://www.imdb.com/title/tt1375666/?ref_=nv_sr_1](https://www.imdb.com/title/tt1375666/?ref_=nv_sr_1)
```

The plugin parses this and extracts `tt1375666`, strips the `tt` prefix (BHD's API expects the numeric ID), and searches BHD by IMDB ID. This works with or without trailing slashes, query parameters, or referrer tags in the URL.

### TMDB link parsing

Similarly, the plugin parses TMDB URLs and auto-detects whether it's a TV show or movie:

```text
[https://www.themoviedb.org/movie/27208-inception](https://www.themoviedb.org/movie/27208-inception)
[https://www.themoviedb.org/tv/1396-breaking-bad](https://www.themoviedb.org/tv/1396-breaking-bad)
```

- For **movie** URLs, the plugin extracts the ID and searches BHD with the `movie/` prefix
- For **TV** URLs, the plugin extracts the ID and searches BHD with the `tv/` prefix

This means you can paste a TMDB link directly from your browser and the plugin will correctly route it as a TV or movie search without you needing to specify the type manually.

**Note:** BHD's API does not support TVDB ID searches directly — `tvdb:` input falls back to a text search using the ID as a keyword.

### From the command line

The plugin follows qBittorrent's `nova2.py` argument convention:

```bash
python3 beyondhd.py <category> "<search term>"
```

Examples:

```bash
python3 beyondhd.py all "star trek"
python3 beyondhd.py movies "inception"
python3 beyondhd.py tv "breaking bad"
python3 beyondhd.py all "tt1375666"
python3 beyondhd.py all "[https://www.imdb.com/title/tt1375666/](https://www.imdb.com/title/tt1375666/)"
python3 beyondhd.py all "[https://www.themoviedb.org/tv/1396](https://www.themoviedb.org/tv/1396)"
```

## Features

- **Official BHD API** — uses `POST /api/torrents/<api_key>` with JSON payloads, matching BHD's documented API
- **Automatic pagination** — fetches all result pages via BHD's `page` parameter until `total_pages` is exhausted, so you get the complete result set (not capped at 100 per page)
- **IMDB/TMDB URL parsing** — paste a full IMDB or TMDB link and the plugin auto-extracts the ID and routes it correctly
- **HDR/DV feature detection** — appends `DV`, `HDR10`, `HDR10+`, `HLG` to titles when multiple features are present
- **Mojibake repair** — automatically fixes double-encoded unicode in titles
- **Seeders sorting/filtering** — configurable via `bhd.json`

## Troubleshooting

### "Plugin bhd is not supported"

The filename and plugin registration don't match. Ensure the file is named `beyondhd.py`.

### Zero results despite valid search

1. **Check your `rss_key`** — BHD omits `download_url` from results without a valid RSS key, and the plugin skips results missing download URLs.
2. **Verify key lengths** — both keys must be exactly 32 characters.
3. **Test the API directly:**

```bash
curl -s -X POST "[https://beyond-hd.me/api/torrents/YOUR_API_KEY](https://beyond-hd.me/api/torrents/YOUR_API_KEY)" \
  -H "Accept: application/json" \
  -H "Content-Type: application/json" \
  -d '{"action":"search","rsskey":"YOUR_RSS_KEY","search":"ubuntu","page":"1"}' | python3 -m json.tool | head -50
```

If `"status_code": 0`, a key is invalid.

### Same result returned regardless of search term

You're running the plugin directly without arguments — the `__main__` block defaults to a hardcoded `ubuntu` test search. Use the full argument form: `python3 beyondhd.py all "your search"`.

## License

Provided as-is for personal use with a valid Beyond-HD account. Respect BHD's terms of service and API rate limits.