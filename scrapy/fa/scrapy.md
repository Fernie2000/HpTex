# مشخصات فنی و راهنمای جامع عملیاتی: `Scrapy`

> **رده سند:** مرجع استاندارد مهندسی سیستم  
> **حوزه تخصصی:** وب اسکرپینگ ناهمگام، راکتور Twisted/Asyncio، خط‌لوله‌های استخراج و پردازش داده  
> **نسخه هدف:** `Scrapy >= 2.11`

---

## ۱. آناتومی معماری و چرخه جریان داده (Data Flow)

فریم‌ورک `Scrapy` صرفاً یک کتابخانه ساده ارسال درخواست‌های HTTP (مانند `requests`) نیست؛ بلکه یک فریم‌ورک مهندسی مبتنی بر رخداد (Event-driven) و ناهمگام است که بر بستر **Twisted Reactor** (یا حلقه رخداد `asyncio`) پیاده‌سازی شده است. این سیستم با مدیریت غیرمسدودکننده (Non-blocking I/O)، هزاران درخواست هم‌زمان وب را از طریق یک موتور حالت متناهی مرکزی پردازش می‌کند.

```text
               +------------------------------------------------------+
               |             موتور مرکزی (Scrapy Engine)              |
               +------------------------------------------------------+
                 |  ▲              |  ▲                     ▲     |
       درخواست‌ها |  | درخواست‌های   |  | پاسخ‌های دریافتی     |     | اقلام استخراج‌شده
        (اولیه)  |  | جدید (تولیدی) |  | (Responses)        |     | (Yielded Items)
                 ▼  |              ▼  |                     |     ▼
               +---------+       +-------------------+      |   +---------------+
               | اسپایدر |       | میان‌افزار اسپایدر |      |   | خط‌لوله اقلام  |
               | Spiders |       | Spider Middleware |      |   | Item Pipeline |
               +---------+       +-------------------+      |   +---------------+
                                   |  ▲                     |
                                   |  | درخواست / پاسخ      |
                                   ▼  |                     |
               +-----------+     +-----------------------+  |
               | زمان‌بند صف |<--->| میان‌افزار دانلودر    |  |
               | Scheduler |     | Downloader Middleware |  |
               +-----------+     +-----------------------+  |
                                   |  ▲                     |
                                   |  | ترافیک شبکه (HTTP)  |
                                   ▼  |                     |
                                 +------------+             |
                                 |  دانلودر   |-------------+
                                 | Downloader |
                                 +------------+
```

### وظایف اجزای اصلی معماری

۱. **موتور مرکزی (Engine):** مغز متفکر سیستم؛ کنترل جریان داده میان تمام بخش‌ها را بر عهده داشته و با تغییر وضعیت، رویدادها را فراخوانی می‌کند.  
۲. **زمان‌بند صف (Scheduler):** صف اولویت‌بندی‌شده درخواست‌ها؛ مسئولیت نگهداری و همچنین حذف درخواست‌های تکراری (از طریق الگوریتم اثرانگشت SHA1 در `RFPDupeFilter`) را به دوش می‌کشد.  
۳. **دانلودر (Downloader):** واکش‌کننده ناهمگام HTTP/HTTPS؛ ارتباطات هم‌زمان شبکه، کش DNS، مدیریت کانکشن‌پولینگ و هندشیک‌های TLS را کنترل می‌کند.  
۴. **اسپایدرها (Spiders):** کلاس‌های اختصاصی کاربر جهت تجزیه و تحلیل ساختار HTML/JSON پاسخ‌ها و تولید خروجی‌های ساختاریافته (`Item`) یا درخواست‌های جدید (`Request`).  
۵. **خط‌لوله اقلام (Item Pipeline):** فیلترهای سریالی متوالی جهت اعتبارسنجی داده‌ها، پاک‌سازی، حذف اقلام تکراری و در نهایت ذخیره‌سازی در پایگاه داده (PostgreSQL، MongoDB یا S3).  
۶. **میان‌افزار دانلودر (Downloader Middleware):** لایه میانی میان Engine و Downloader جهت تزریق رفتارهای شبکه‌ای نظیر چرخش پروکسی، تغییر User-Agent و تلاش مجدد (Retry).  
۷. **میان‌افزار اسپایدر (Spider Middleware):** قلاب‌های پردازشی میان Engine و Spiders برای رهگیری ورودی‌ها و خروجی‌های توابع پارسر.

---

## ۲. گرامر دستورات و ماتریس کنترل CLI

دستورات `scrapy` به دو دسته **سراسری (Global)** (قابل اجرا در هر مسیر بدون پروژه) و **محیط پروژه (Project)** (نیازمند وجود فایل پیکربندی `scrapy.cfg`) تفکیک می‌شوند:

| دستور | قلمرو | ساختار نحوی | عملکرد سیستمی و منطق اجرا |
| :--- | :--- | :--- | :--- |
| **`crawl`** | پروژه | `scrapy crawl <spider_name> -O out.jsonl` | بالا آوردن راکتور Twisted، اجرای پردازش عنکبوت و استریم اقلام به فایل خروجی. |
| **`shell`** | سراسری | `scrapy shell "<url>" --nolog` | محیط تعاملی IPython متصل به آبجکت‌های `response` و `spider` برای تست سلکتورها. |
| **`fetch`** | سراسری | `scrapy fetch --headers "<url>"` | دریافت مستقیم سند HTTP خام از طریق دانلودر Scrapy بدون دخالت کدهای اسپایدر. |
| **`view`** | سراسری | `scrapy view "<url>"` | دریافت صفحه و باز کردن آن در مرورگر سیستم جهت ارزیابی وابستگی صفحه به کدهای جاوااسکریپت. |
| **`parse`** | پروژه | `scrapy parse --spider=<name> -c <method> <url>` | آزمون تعیین‌کننده (Unit Test) توابع callback اسپایدر با متدهای مشخص. |
| **`genspider`**| پروژه | `scrapy genspider -t crawl <name> <domain>` | تولید خودکار قالب اولیه اسپایدر از روی الگوها (`basic`، `crawl`، `xmlfeed`، `csvfeed`). |
| **`runspider`**| سراسری | `scrapy runspider <script.py>` | اجرای یک اسپایدر مستقل پایتون بدون نیاز به ساختار پوشه‌بندی و پروژه فرمال. |
| **`settings`** | هر دو | `scrapy settings --get CONCURRENT_REQUESTS` | بررسی مقدار نهایی اعمال‌شده در متغیرهای تنظیمات انجین. |

---

## ۳. موتور انتخاب‌گرها (Selectors) و خط‌لوله استخراج

ابزار Scrapy از کتابخانه داخلی `parsel` بهره می‌برد که ساختار **W3C XPath 1.0** و **CSS3 Selectors** را در قالب درخت‌های فوق سریع زبان C در `lxml` اجرا می‌کند.

### ۳.۱ جدول تطبیقی سلکتورهای CSS و XPath

| هدف استخراج | سلکتور CSS3 | معادل استاندارد XPath 1.0 | نکات مهندسی و سرعت |
| :--- | :--- | :--- | :--- |
| **مقدار ویژگی (Attribute)** | `a::attr(href)` | `//a/@href` | ساختار XPath امکان بررسی زیررشته‌ها (`contains()`) را فراهم می‌کند. |
| **استخراج متن گره** | `div.title::text` | `//div[@class='title']/text()` | امکان تجمیع بازگشتی تمام متون فرزندان با تابع `string(//div)`. |
| **پیمایش به سمت والد** | *پشتیبانی نمی‌شود* | `//input[@id='sku']/ancestor::form` | قابلیت پیمایش دوبعدی در کل گراف DOM با محورهای XPath. |
| **استخراج با عبارات باقاعده**| `response.css('h1::text').re(r'ID: (\d+)')` | `response.xpath('//h1/text()').re_first(r'\d+')` | اجرای موتور Regex مستقیماً روی محتوای متنی نودها بدون نیاز به پایتون خام. |

### ۳.۲ خط‌لوله پردازش تمیز داده‌ها (`ItemLoader`)

جهت جلوگیری از درهم‌تنیدگی کدهای استخراج و پاک‌سازی داده، از کلاس `ItemLoader` به همراه پردازنده‌های ورودی/خروجی استفاده کنید:

```python
from itemloaders import ItemLoader
from itemloaders.processors import TakeFirst, MapCompose, Identity
from w3lib.html import remove_tags, replace_escape_chars

def clean_currency(value: str) -> float:
    return float(value.replace("$", "").replace(",", "").strip())

class ProductLoader(ItemLoader):
    default_output_processor = TakeFirst()
    
    # خط‌لوله ورودی: حذف تگ‌ها -> نرمال‌سازی فواصل -> تبدیل به عدد اعشاری
    price_in = MapCompose(remove_tags, replace_escape_chars, clean_currency)
    description_in = MapCompose(remove_tags, str.strip)
    tags_out = Identity() # حفظ ساختار لیست برای برچسب‌ها
```

---

## ۴. فایل تنظیمات بهینه‌شده برای محیط پروداکشن (`settings.py`)

این پیکربندی گلوگاه‌های I/O را حذف کرده، مصرف رم را محدود ساخته، مکانیزم تطبیقی ارسال ریکوئست را فعال و خطاهای متداول شبکه را کنترل می‌کند:

```python
# ==============================================================================
# PhTex High-Performance Production Scrapy settings.py
# ==============================================================================
BOT_NAME = "phtex_crawler"
SPIDER_MODULES = ["phtex_crawler.spiders"]
NEWSPIDER_MODULE = "phtex_crawler.spiders"

# ۱. هم‌روندی شبکه و استفاده از راکتور مدرن Asyncio
TWISTED_REACTOR = "twisted.internet.asyncioreactor.AsyncioSelectorReactor"
CONCURRENT_REQUESTS = 32
CONCURRENT_REQUESTS_PER_DOMAIN = 16
CONCURRENT_REQUESTS_PER_IP = 0

# ۲. الگوریتم کنترل تطبیقی نرخ درخواست‌ها (AutoThrottle)
AUTOTHROTTLE_ENABLED = True
AUTOTHROTTLE_START_DELAY = 1.0
AUTOTHROTTLE_MAX_DELAY = 10.0
AUTOTHROTTLE_TARGET_CONCURRENCY = 2.0
AUTOTHROTTLE_DEBUG = False

# ۳. تنظیمات تایم‌اوت و تاب‌آوری در برابر خطاهای سرور
DOWNLOAD_TIMEOUT = 15
RETRY_ENABLED = True
RETRY_TIMES = 3
RETRY_HTTP_CODES = [500, 502, 503, 504, 522, 524, 408, 429]

# ۴. حفاظت از رم و جلوگیری از خطای OOM سرور
MEMUSAGE_ENABLED = True
MEMUSAGE_LIMIT_MB = 2048
MEMUSAGE_WARNING_MB = 1536

# ۵. بهینه‌سازی و کش درخواست‌های DNS
DNSCACHE_ENABLED = True
DNSCACHE_SIZE = 10000
DNS_TIMEOUT = 5

# ۶. استاندارد هشینگ درخواست‌ها جهت جلوگیری از تکرار
REQUEST_FINGERPRINTER_IMPLEMENTATION = "2.7"
DUPEFILTER_CLASS = "scrapy.dupefilters.RFPDupeFilter"

# ۷. هدرهای شبیه‌سازی مرورگرهای مدرن
DEFAULT_REQUEST_HEADERS = {
    "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8",
    "Accept-Language": "fa-IR,fa;q=0.9,en-US;q=0.8,en;q=0.7",
    "Accept-Encoding": "gzip, deflate, br",
    "Sec-Fetch-Dest": "document",
    "Sec-Fetch-Mode": "navigate",
    "Sec-Fetch-Site": "none",
    "Sec-Fetch-User": "?1",
    "Upgrade-Insecure-Requests": "1",
}

# ۸. پایپ‌لاین اکسپورت داده‌ها به فرمت خطی و استریم JSON Lines
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

## ۵. اجرای برنامه‌نویسی‌شده و سرویس‌محور (Programmatic Execution)

در زیرساخت‌های بزرگ، کراولرها نباید از طریق CLI اجرا شوند؛ بلکه باید با کلاس `CrawlerRunner` درون میکروسرویس‌های FastAPI یا ورکر‌های توزیع‌شده Celery ادغام گردند.

### اسکریپت اجرای موازی چند اسپایدر (`crawler_worker.py`)

```python
import asyncio
from twisted.internet import asyncioreactor
asyncioreactor.install() # باید حتماً پیش از هرگونه ایمپورت از reactor اجرا گردد

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

    print("[*] در حال اجرای اسپایدر کاتالوگ...")
    await runner.crawl(CatalogSpider, category="electronics")
    
    print("[*] در حال اجرای اسپایدر موجودی انبار...")
    await runner.crawl(InventorySpider)
    
    print("[✓] چرخه کراول با موفقیت به پایان رسید.")

if __name__ == "__main__":
    task = run_pipeline()
    asyncio.ensure_future(task)
    reactor.run() # آغاز حلقه پردازش رخداد
```

---

## ۶. جدول عیب‌یابی و سناریوهای بحرانی

| علامت و خطا | علت ریشه‌ای مشکل | راه‌حل مهندسی و قطعی |
| :--- | :--- | :--- |
| **خطای `HTTP 403 Forbidden` / چالش کلودفلر** | عدم تطابق اثرانگشت TLS (JA3) یا مسدودسازی رنج آی‌پی‌های دیتاسنتری. | استفاده از شبکه پروکسی‌های مسکونی (Residential)، تنظیم `DOWNLOADER_CLIENTCONTEXTFACTORY` یا درایور Playwright. |
| **خطای `ReactorAlreadyInstalledError`** | راکتور Twisted پس از حلقه asyncio یا به صورت تکراری ایمپورت شده است. | اجرای دستور `asyncioreactor.install()` در نخستین خط اسکریپت و پیش از هرگونه ماژول شبکه. |
| **انفجار مصرف حافظه رم (OOM Crash)** | رشد نامحدود صف درخواست‌ها یا نگهداری ارجاعات چرخشی (Circular References) در کالبک‌ها. | تنظیم `DEPTH_PRIORITY = 1`، استفاده از صف‌های دیسکی `SCHEDULER_DISK_QUEUE` و فعال‌سازی `MEMUSAGE_LIMIT_MB`. |
| **خطای `ResponseNeverReceived` / تایم‌اوت DNS** | اشباع پورت‌های موقت سوکت سیستم عامل در تعداد هم‌روندی بسیار بالا. | فعال‌سازی `DNSCACHE_ENABLED = True`، کاهش تایم‌اوت به ۱۰ ثانیه و بهینه‌سازی `tcp_tw_reuse` در لینوکس. |
| **تفاوت محتوای دریافتی با مرورگر واقعی** | وب‌سایت از نوع تک‌صفحه‌ای (SPA) بوده و داده‌ها را با فرانت‌اند React/Vue رندر می‌کند. | پایش درخواست‌های مستقیم API در تب Network مرورگر یا اتصال افزونه `scrapy-playwright` به دانلودر. |
