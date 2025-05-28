# ail-feeder-atom-rss

**ail-feeder-atom-rss** is an external Atom and RSS feed ingestion tool for the [AIL framework](https://github.com/ail-project/AIL-framework). It allows you to automatically fetch, parse, and push Atom/RSS feed entries and extracted URLs into AIL for further analysis.

## Features

- **Batch or Single Feed Processing:**  
  Ingests multiple RSS/Atom feeds defined in a `links.txt` file, or process a single feed directly via the `--link` command-line option.
- **Automatic Content Extraction:**  
  Utilizes powerful Python libraries (`feedparser`, `newspaper3k`, `trafilatura`, etc.) to extract entries, metadata, and discover embedded URLs from feed summaries and content.
- **URL Extraction & Enrichment:**  
  Optionally extracts and processes URLs found within feed entries. For each URL, detailed article content and NLP metadata (authors, keywords, etc.) are gathered.
- **Caching with Redis:**  
  Prevents duplicate processing of previously seen feeds and URLs by leveraging Redis as a fast in-memory cache.
- **Integration with AIL:**  
  Pushes all processed data directly to an AIL instance via the [PyAIL](https://github.com/ail-project/PyAIL) library.
- **Configurable & Verbose:**  
  Supports a flexible configuration file for credentials and infrastructure, and provides detailed logging with the `--verbose` flag.

## Installation & Requirements

Install Python dependencies:
```bash
pip3 install -U -r requirements.txt
```

**Key dependencies:**
- [pyail](https://github.com/ail-project/PyAIL) — AIL integration.
- [trafilatura](https://github.com/adbar/trafilatura) — Feed parsing and content extraction.
- [feedparser](https://github.com/kurtmckee/feedparser) — Atom/RSS parsing.
- [newspaper3k](https://github.com/codelucas/newspaper) — Article text extraction and NLP.
- [redis](https://github.com/andymccurdy/redis-py) — Caching.
- [validators](https://github.com/kvesteri/validators) — URL validation.
- [urlextract](https://github.com/lipoja/URLExtract) — Extracts URLs from text.

## Configuration

1. **Create configuration file:**  
   Copy and edit the sample config file `etc/ail-feeder-atom-rss.cfg` to `/etc/ail-feeder-atom-rss.cfg`.  
   Fill in your Redis, AIL credentials, and other parameters as needed.

2. **Prepare feeds list:**  
   Add each RSS or Atom feed URL you want to process into `links.txt`, one per line.

## Usage

1. **Start AIL:**  
   Ensure your AIL instance is running and accessible.

2. **Run the feeder:**  
   Use the following command to see all available options:
   ```bash
   python3 bin/feeder.py --help
   ```

### Command-Line Options (Detailed)

- `-h`, `--help`  
  Display the help message with descriptions of all available options and exit.

- `--verbose`  
  Enable verbose output. When this flag is set, the script outputs detailed logs to help with debugging and to give insight into the processing steps, including which feeds and entries are processed, skipped, or uploaded.

- `--nocache`  
  Disable Redis-based caching.  
  By default, the script uses Redis to remember which feeds and entries have already been processed, avoiding duplicate processing.  
  If you use `--nocache`, the script will ignore the cache and process every feed and entry as if they were new, regardless of previous runs.

- `--urlextract`  
  Enable URL extraction from feed entries.  
  When this flag is used, the script analyzes the summary and content fields of each feed entry to find any embedded URLs. For each discovered URL, it performs additional extraction: downloading the target page, parsing the article content, and extracting NLP features (such as authors, keywords, and publication date). This data is also uploaded to AIL.

- `--link LINK`  
  Specify a single feed URL to process.  
  If this option is set, the given link is **added** to the list of feeds obtained from `links.txt`. This allows you to process one-off feeds on demand without editing your primary links file.  
  Example:  
  ```bash
  python3 bin/feeder.py --link "https://example.com/feed.xml"
  ```
  If you only want to process a specific feed, you can temporarily clear `links.txt` and use this option.

#### Example Usage

- Process all feeds in `links.txt` with verbose output:
  ```bash
  python3 bin/feeder.py --verbose
  ```

- Process a single feed and extract URLs from entries:
  ```bash
  python3 bin/feeder.py --link "https://example.com/feed.xml" --urlextract
  ```

- Process all feeds and ignore the cache, forcing re-processing of all entries:
  ```bash
  python3 bin/feeder.py --nocache
  ```

## How it Works

- The script fetches and parses each feed, extracting metadata and entries.
- For each entry, it can extract embedded URLs (when `--urlextract` is specified) and fetch full article content and NLP metadata.
- All processed items are sent to AIL for storage and analysis.
- Redis is used to cache processed feeds and URLs, enabling efficient re-runs and incremental updates.

## Troubleshooting

- Ensure your AIL configuration and credentials are correct in `/etc/ail-feeder-atom-rss.cfg`.
- Redis must be running and accessible as configured.
- For debugging, use the `--verbose` flag to see detailed logs.

## License

See [LICENSE](LICENSE) for details.
