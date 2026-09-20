Price Comparison Scraper

A configuration-driven web scraper that collects product listings from multiple online retailers and combines them into a single, comparable dataset.

Built in 2023 as a self-directed project while studying BA (Hons) Business Management with Information Systems.

What it does

Given a list of retailer configurations, the scraper visits each site, follows every product link on the listing pages, and extracts a consistent set of fields into one output file:

Field	Description
web_name	Which retailer the record came from
product_names	Product title as listed
prices	Price as displayed on the product page
product_types	Category, taken from the site's breadcrumb trail
urls	Direct link to the product page

The current configuration targets Nintendo Switch games across three Vietnamese retailers — nshop.com.vn, herogame.vn and muagame.vn. The dataset in main_project/output_folder/ contains roughly 870 product records collected across those sites.

The design idea

The point of the project was not to scrape three particular shops. It was to avoid writing a separate scraper for each one.

There is one spider. Every site-specific detail — the start URL, the CSS selectors for the product list, the price, the product name, the XPath for the category — lives as an entry in main_project/list_data.json. The spider reads that file at runtime and works through each entry in turn.

Adding a fourth retailer means adding a block of configuration. It does not mean writing any code:

json
{
    "web_name": "example_shop",
    "dynamic_website": false,
    "allowed_domain": "www.example.com",
    "start_url": "https://www.example.com/switch-games?page=1",
    "products": ".product-card",
    "current_product_url": "a::attr(href)",
    "products_url": "https://www.example.com",
    "product_names": "h1.product-title::text",
    "prices": ".price span::text",
    "product_types": "//nav[@class='breadcrumb']/a[2]/text()",
    "next_page": "a.next::attr(href)",
    "next_page_path": null
}

This keeps the scraping logic in one place and the site-specific brittleness in another, so when a retailer redesigns its pages only the configuration needs updating.

Handling JavaScript pagination

Not every site paginates in a way a standard crawler can follow. Some load the next page through JavaScript, so there is no link in the HTML to follow.

main_project/automation/selenium/click_next_page/ handles those cases. It drives a real Chrome browser, clicks the next-page control repeatedly until the control no longer exists, and writes the collected page URLs to next_page_url.json for the spider to consume.

This runs as a separate, optional step rather than as part of every crawl. Driving a browser is slow, and the set of page URLs for a site rarely changes between runs, so re-collecting them on every scrape wastes time for no benefit.

Technologies
Python 3.10
Scrapy — crawling framework and item pipeline
Selenium — browser automation for JavaScript-driven pagination
Project structure
main_project/
├── spiders/webspider.py     # the single generic spider
├── items.py                 # field definitions
├── pipelines.py             # item pipeline
├── settings.py              # Scrapy settings
├── list_data.json           # per-site configuration
├── automation/selenium/     # JavaScript pagination handling
└── output_folder/           # CSV and JSON output
Running it
bash
git clone https://github.com/binamite/price_comparison.git
cd price_comparison
pip install scrapy selenium
scrapy crawl webspider -o output_folder/products_datas.csv

Requires a Chrome driver on the system path if the Selenium pagination step is used.

Sample output
csv
web_name,product_names,prices,product_types,urls
muagame,The Legend of Zelda: Tears of the Kingdom - Nintendo Switch,1.280.000₫,GAME NINTENDO SWICH,https://muagame.vn/game/...
nshop,SW305 - DRAGON BALL THE BREAKERS cho Nintendo Switch,"900,000 ₫",Game Cho Nintendo Switch & Switch Lite,https://www.nshop.com.vn/products/...
Known limitations

Recorded honestly, as a list of what a second version would fix:

Prices are not normalised. The three sites format prices differently — 1.280.000₫ against "900,000 ₫" — so the raw output is not directly comparable. A cleaning step in the pipeline is the obvious next piece of work, and it is the main thing standing between this and a usable comparison.
A hardcoded absolute file path remains in the Selenium module, pointing at the original development machine. It would need replacing with a relative path before the project runs anywhere else.
In-spider pagination is currently disabled. The spider processes the listing pages it is given; multi-page collection depends on the separate Selenium step.
No dependency file. Dependencies are installed manually rather than from requirements.txt.
Compiled artefacts are committed (.pyc, .DS_Store) and should be excluded via .gitignore.
A note on scope

This is a command-line tool that produces a dataset. It is not a hosted website and has no user interface — an earlier version of this README described it as one, which was inaccurate.

Anyone reusing the approach should check the terms of service and robots.txt of the sites they point it at, and keep request rates reasonable.

Contact

Ky Quang Pham — via GitHub @binamite
