# Technical Specification & Operational Manual: `Scrapy`

> **Document Class:** Engineering Reference Standard  
> **Domain:** Asynchronous Web Scraping, Twisted/Asyncio Reactor, Data Extraction Pipelines  
> **Target Release:** `Scrapy >= 2.11`

---

## 1. Architectural Anatomy & Reactive Data Flow

Scrapy is not an HTTP client wrapper; it is an asynchronous, event-driven web scraping framework operating atop the **Twisted Reactor** (or `asyncio` event loop). It processes concurrent I/O operations non-blockingly via a centralized finite-state engine.

```text
               +------------------------------------------------------+
               |                    Scrapy Engine                     |
               +------------------------------------------------------+
                 |  ▲              |  ▲                     ▲     |
       Requests  |  | Requests     |  | Responses           |     | Extracted
       (Initial) |  | (Yielded)    |  |                     |     | Items
                 ▼  |              ▼  |                     |     ▼
               +---------+       +-------------------+      |   +---------------+
               | Spiders |       | Spider Middleware |      |   | Item Pipeline |
               +---------+       +-------------------+      |   +---------------+
                                   |  ▲                     |
                                   |  | Response/Request    |
                                   ▼  |                     |
               +-----------+     +-----------------------+  |
               | Scheduler |<--->| Downloader Middleware |  |
               +-----------+     +-----------------------+  |
                                   |  ▲                     |
                                   |  | HTTP Traffic        |
                                   ▼  |                     |
                                 +------------+             |
                                 | Downloader |-------------+
                                 +------------+
```

### Discrete Subsystem Responsibilities

1. **Scrapy Engine:** Central orchestrator; mediates data flow signals between all components and triggers events upon state transitions.
2. **Scheduler:** Priority queue maintaining pending `scrapy.Request` objects; handles deduplication (`RFPDupeFilter` via SHA1 fingerprinting).
3. **Downloader:** Asynchronous HTTP/HTTPS fetcher built on Twisted's HTTP client; handles connection pooling, DNS caching, and SSL handshakes.
4. **Spiders:** User-defined stateful parsers; consume `Response` payloads and yield structured `Item` dictionaries or subsequent `Request` instances.
5. **Item Pipeline:** Sequential sink filters; validate, clean, deduplicate, and persist yielded items into databases (PostgreSQL, MongoDB, S3).
6. **Downloader Middleware:** Hook interface sitting between Engine and Downloader (`process_request`, `process_response`, `process_exception`). Used for proxy rotation, user-agent spoofing, and HTTP retries.
7. **Spider Middleware:** Hook interface sitting between Engine and Spider (`process_spider_input`, `process_spider_output`).

---

## 2. Command Grammar & Control Matrix

Scrapy commands operate in two scopes: **Global** (standalone execution) and **Project** (inside an active workspace containing `scrapy.cfg`).

| Command | Scope | Syntax | Operational Semantics |
| :--- | :--- | :--- | :--- |
| **`crawl`** | Project | `scrapy crawl <spider_name> -O out.jsonl` | Boots Twisted reactor, spawns spider instance, pipes items to feed export. |
| **`shell`** | Global | `scrapy shell "<url>" --nolog` | Interactive IPython/bpython REPL bound to pre-fetched `response` and `spider` objects. |
| **`fetch`** | Global | `scrapy fetch --headers "<url>"` | Fetches raw HTTP document via Scrapy downloader without invoking spider logic. |
| **`view`** | Global | `scrapy view "<url>"` | Fetches page via Scrapy and renders it in local default browser (auditing JS dependency). |
| **`parse`** | Project | `scrapy parse --spider=<name> -c <method> <url>` | Deterministic unit-testing of spider callback routines with explicit contracts. |
| **`genspider`**| Project | `scrapy genspider -t crawl <name> <domain>` | Scaffolds spider boilerplate from templates (`basic`, `crawl`, `xmlfeed`, `csvfeed`). |
| **`runspider`**| Global | `scrapy runspider <script.py>` | Executes self-contained standalone spider script without formal project hierarchy. |
| **`settings`** | Both | `scrapy settings --get CONCURRENT_REQUESTS` | Queries resolved runtime engine configuration parameters. |

---

## 3. Selector Engine & Parsing Pipeline

Scrapy utilizes the `parsel` library, combining **W3C XPath 1.0** and **CSS3 Selectors** compiled into C-optimized `lxml` trees.

### 3.1 Selector Syntax Algebra

| Target Pattern | CSS3 Selector | XPath 1.0 Equivalent | Speed / Nuance |
| :--- | :--- | :--- | :--- |
| **Attribute Value** | `a::attr(href)` | `//a/@href` | XPath supports substring queries (`contains()`, `starts-with()`). |
| **Text Extraction** | `div.title::text` | `//div[@class='title']/text()` | XPath handles recursive string collation: `string(//div)`. |
| **Parent Traversal** | *Unsupported* | `//input[@id='sku']/ancestor::form` | XPath permits arbitrary directional graph traversal. |
| **Regex Extraction**| `response.css('h1::text').re(r'ID: (\d+)')` | `response.xpath('//h1/text()').re_first(r'\d+')` | Extracts compiled regex match groups directly from nodes. |

### 3.2 Production Item Loader Pipeline

Decouple extraction logic from spider parsers using `ItemLoader` with composable input/output processors:

```python
from itemloaders import ItemLoader
from itemloaders.processors import TakeFirst, MapCompose, Identity
from w3lib.html import remove_tags, replace_escape_chars

def clean_currency(value: str) -> float:
    return float(value.replace("$", "").replace(",", "").strip())

class ProductLoader(ItemLoader):
    default_output_processor = TakeFirst()
    
    # Input pipeline: Strip tags -> normalize whitespace -> cast float
    price_in = MapCompose(remove_tags, replace_escape_chars, clean_currency)
    description_in = MapCompose(remove_tags, str.strip)
    tags_out = Identity() # Preserve list structure
```

---

## 4. Hardened Production Configuration (`settings.py`)

Deploy this configuration to eliminate bottlenecks, prevent memory leaks, bypass aggressive anti-bot heuristics, and maintain deterministic concurrency.

```python
# ==============================================================================
# PhTex High-Performance Production Scrapy settings.py
# ==============================================================================
BOT_NAME = "phtex_crawler"
SPIDER_MODULES = ["phtex_crawler.spiders"]
NEWSPIDER_MODULE = "phtex_crawler.spiders"

# 1. Concurrency & I/O Optimization
TWISTED_REACTOR = "twisted.internet.asyncioreactor.AsyncioSelectorReactor"
CONCURRENT_REQUESTS = 32
CONCURRENT_REQUESTS_PER_DOMAIN = 16
CONCURRENT_REQUESTS_PER_IP = 0

# 2. Adaptive AutoThrottle Algorithm (Dynamic Backoff)
AUTOTHROTTLE_ENABLED = True
AUTOTHROTTLE_START_DELAY = 1.0
AUTOTHROTTLE_MAX_DELAY = 10.0
AUTOTHROTTLE_TARGET_CONCURRENCY = 2.0
AUTOTHROTTLE_DEBUG = False

# 3. Timeout & Download Resilience
DOWNLOAD_TIMEOUT = 15
RETRY_ENABLED = True
RETRY_TIMES = 3
RETRY_HTTP_CODES = [500, 502, 503, 504, 522, 524, 408, 429]

# 4. Memory Safety & OS Resource Bounds
MEMUSAGE_ENABLED = True
MEMUSAGE_LIMIT_MB = 2048
MEMUSAGE_NOTIFY_MAIL = ["ops@example.com"]
MEMUSAGE_WARNING_MB = 1536

# 5. DNS Caching & Connection Persistence
DNSCACHE_ENABLED = True
DNSCACHE_SIZE = 10000
DNS_TIMEOUT = 5

# 6. HTTP Cache (Dev & Re-crawl Optimization)
HTTPCACHE_ENABLED = False
HTTPCACHE_EXPIRATION_SECS = 86400
HTTPCACHE_DIR = "httpcache"
HTTPCACHE_IGNORE_HTTP_CODES = [301, 302, 403, 404, 500]
HTTPCACHE_STORAGE = "scrapy.extensions.httpcache.FilesystemCacheStorage"

# 7. Request Fingerprinting & Deduplication Standard
REQUEST_FINGERPRINTER_IMPLEMENTATION = "2.7"
DUPEFILTER_CLASS = "scrapy.dupefilters.RFPDupeFilter"

# 8. Modern Default Headers (Heuristic Spoofing)
DEFAULT_REQUEST_HEADERS = {
    "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8",
    "Accept-Language": "en-US,en;q=0.9",
    "Accept-Encoding": "gzip, deflate, br",
    "Sec-Fetch-Dest": "document",
    "Sec-Fetch-Mode": "navigate",
    "Sec-Fetch-Site": "none",
    "Sec-Fetch-User": "?1",
    "Upgrade-Insecure-Requests": "1",
}

# 9. Feed Export Optimization (Streamed JSON Lines)
FEEDS = {
    "output/data_%(time)s.jsonl": {
        "format": "jsonlines",
        "encoding": "utf8",
        "store_empty": False,
        "overwrite": False,
    }
}
```

---

## 5. Programmatic Orchestration Routine

In production architectures, Scrapy should run decoupled from the `scrapy` CLI wrapper, integrated into Celery tasks, FastAPI microservices, or headless Cron workers via `CrawlerRunner`.

### Programmatic Multi-Spider Execution (`crawler_worker.py`)

```python
import asyncio
from twisted.internet import asyncioreactor
asyncioreactor.install() # Must occur prior to importing reactor

from twisted.internet import reactor
from scrapy.crawler import CrawlerRunner
from scrapy.utils.project import get_project_settings
from scrapy.utils.log import configure_logging
from phtex_crawler.spiders.catalog import CatalogSpider
from phtex_crawler.spiders.inventory import InventorySpider

async def run_pipeline():
    configure_logging()
    settings = get_project_settings()
    runner = CrawlerRunner(settings)

    # Chain concurrent spiders sequentially or in parallel
    print("[*] Spawning CatalogSpider...")
    await runner.crawl(CatalogSpider, category="electronics")
    
    print("[*] Spawning InventorySpider...")
    await runner.crawl(InventorySpider)
    
    print("[✓] Scraping Pipeline Completed Successfully.")

if __name__ == "__main__":
    task = run_pipeline()
    asyncio.ensure_future(task)
    reactor.run() # Starts event loop
```

---

## 6. Diagnostic Matrix & Failure Modes

| Symptom / Error | Root Cause | Definitive Engineering Remediation |
| :--- | :--- | :--- |
| **`HTTP 403 Forbidden` / Cloudflare Challenge** | Missing TLS fingerprinting (JA3), modern Sec-CH headers, or static IP blacklisting. | Implement residential proxy rotation, pass `DOWNLOADER_CLIENTCONTEXTFACTORY`, or deploy Playwright/Puppeteer sidecar. |
| **`ReactorAlreadyInstalledError`** | Twisted reactor initialized after asyncio event loop or imported twice. | Execute `asyncioreactor.install()` before importing `reactor` or any Scrapy modules. |
| **Memory Ballooning (OOM Crash)** | Unbounded request depth queue, unclosed response bodies, or cycle references in spider callbacks. | Enforce `DEPTH_PRIORITY = 1`, `SCHEDULER_DISK_QUEUE`, and enable `MEMUSAGE_LIMIT_MB`. |
| **`ResponseNeverReceived` / DNS Timeouts** | Exhaustion of OS ephemeral socket ports or DNS resolution latency under high concurrency. | Enable `DNSCACHE_ENABLED = True`, reduce `DOWNLOAD_TIMEOUT = 10`, and tune host `net.ipv4.tcp_tw_reuse = 1`. |
| **Divergent Content vs Browser View** | Target site utilizes Client-Side Rendering (CSR) via React/Vue/Angular. | Audit network XHR/GraphQL endpoints via DevTools, or hook `scrapy-playwright` headless driver into Downloader. |
