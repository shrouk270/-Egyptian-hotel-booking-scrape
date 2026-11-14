Booking.com Hotel & Guest Review Scraper

A Python-based web scraper that collects hotel information and guest reviews from Booking.com for one or multiple destinations.

The scraper uses Playwright to automate a real Chrome or Chromium browser, search for hotels, load search results, visit individual hotel pages, and extract guest reviews.

Collected data can be exported to JSON, CSV, or Excel, with one row per guest review.

Features
Multiple destinations: Scrape one or more towns across different countries.
Automated browser: Uses Playwright to control Chrome or Chromium.
Hotel information: Collects hotel names, addresses, prices, and ratings.
Guest reviews: Extracts review titles, positive and negative comments, dates, countries, and scores.
Multiple review sources: Attempts to collect reviews through the review API, embedded page data, and the full reviews panel.
Duplicate removal: Removes duplicate reviews collected from different sources.
Multiple export formats: Save results as JSON, CSV, or Excel.
Customizable scraping: Control the number of hotels and reviews per hotel.
Independent destination processing: If one destination fails, the scraper continues with the remaining destinations.
Automatic saving: Saves collected data when scraping is interrupted with Ctrl+C.
Offline tests: Includes unit tests that run without opening a browser or accessing the internet.
Requirements
Python 3.8 or newer
Google Chrome (optional)
Playwright Chromium (installed automatically during setup)
Installation
1. Clone the repository
bash
git clone https://github.com/YOUR_USERNAME/booking-review-scraper.git
cd booking-review-scraper

2. Create a virtual environment

Windows:

bash
python -m venv .venv
.venv\Scripts\activate


macOS / Linux:

bash
python -m venv .venv
source .venv/bin/activate

3. Install dependencies
bash
pip install -e .

4. Install Playwright Chromium
bash
python -m playwright install chromium


After installation, the booking-scraper command will be available.

If Google Chrome is installed, the scraper uses it. Otherwise, it uses the Chromium browser downloaded by Playwright.

Usage

You can run the scraper using either:

bash
python scraping.py


or:

bash
booking-scraper

1. Scrape one destination

Collect hotels and reviews from Cairo, Egypt:

bash
python scraping.py -c Egypt -t Cairo

2. Scrape multiple towns in one country
bash
python scraping.py -c Egypt -t "Cairo, Luxor, Alexandria" -o csv

3. Scrape destinations in different countries
bash
python scraping.py \
    -d "Cairo, Egypt" \
    -d "Marrakech, Morocco"

4. Read destinations from a file

Create a places.txt file:

text
Cairo, Egypt
Luxor, Egypt
Marrakech, Morocco
Istanbul, Turkey


Run:

bash
python scraping.py --places places.txt -o excel


Lines beginning with # are treated as comments.

5. Specify an output directory
bash
booking-scraper -c Egypt -t Aswan --output-dir results


The output directory is created automatically if it doesn't exist.

Command-Line Options
Option	Default	Description
-c, --country	—	Country for subsequent towns
-t, --town	—	Town to scrape; supports comma-separated towns
-d, --destination	—	Destination in Town, Country format
--places	—	Text file containing destinations
-o, --out_format	json	Output format: json, csv, or excel
--output-dir	Script directory	Directory for output files
-s, --startdate	Today	Check-in date (YYYY-MM-DD)
-e, --enddate	Tomorrow	Check-out date (YYYY-MM-DD)
-r, --rooms	1	Number of rooms
-p, --persons	1	Number of adults
--max-hotels	20	Maximum hotels per destination
--max-reviews	8	Maximum reviews per hotel
--headless	Off	Run browser without displaying its window

Note: Check-in dates, check-out dates, room count, and guest count affect the prices displayed by Booking.com. They do not determine which guest reviews are collected.

Output

Each destination is saved in a separate file.

Example filenames:

text
hotels-in-Cairo-Egypt.json
hotels-in-Luxor-Egypt.csv
hotels-in-Marrakech-Morocco.xlsx

Available fields

Each row represents one guest review.

Column	Example
hotel_name	TUT Hotel-Downtown
destination	Cairo, Egypt
address	31 Talaat Harb, 4280160 Cairo, Egypt
price_text	EGP 31,112
numeric_rating	9.5
review_title	Exceptional
review_positive	What the guest liked
review_negative	What the guest didn't like
review_date	2026-10-21
review_country	Germany
review_rate_scale	10/10

Hotel information is repeated for every review belonging to that hotel.

If a hotel has no readable reviews, it is still included with one row containing empty review fields.

CSV compatibility

CSV files are exported using UTF-8 with a BOM (utf-8-sig) to ensure that Arabic, accented characters, and other international text display correctly in Microsoft Excel.

How Review Collection Works

Booking.com may display only a limited number of reviews directly on a hotel page. To collect additional reviews, the scraper attempts multiple sources in the following order:

1. Review API
Detects the review-list request made by the hotel page.
Reuses the captured request from within the browser page.
Requests additional review batches using pagination offsets.
Retrieves up to 25 reviews per request.
2. Hotel Page Data

If additional reviews are needed, the scraper checks:

JSON data embedded in the hotel page.
Rendered review cards when embedded JSON is unavailable.
3. Read All Reviews Panel

As a final fallback, the scraper:

Opens the full reviews panel.
Navigates through available review pages.
Extracts the displayed review information.
Duplicate Removal

Reviews collected from different sources are checked and deduplicated to avoid storing the same review multiple times.

The scraper stops collecting reviews once it reaches the configured --max-reviews limit for a hotel.

Project Structure
text
booking-review-scraper/
│
├── scraping.py
│   └── Command-line interface, browser setup,
│       destination search, and hotel discovery.
│
├── hotel_page.py
│   └── Hotel page parsing, review extraction,
│       API handling, and review processing.
│
├── file_writer.py
│   └── JSON, CSV, and Excel export functionality.
│
├── tests/
│   └── Offline unit tests using sample HTML and JSON.
│
├── places.example.txt
│   └── Example destination list.
│
├── pyproject.toml
│   └── Project configuration and dependencies.
│
└── README.md
    └── Project documentation.

Using the Scraper from Python

The scraper can also be imported and used directly in Python without running the command-line interface.

python
from scraping import get_data

paths = get_data(
    destinations=[
        ("Egypt", "Aswan"),
        ("Egypt", "Luxor"),
    ],
    out_format="csv",
    max_hotels=5,
    output_dir="results",
)

print(paths)

Parameters
Parameter	Description
destinations	List of (country, town) tuples
out_format	Export format: JSON, CSV, or Excel
max_hotels	Maximum hotels per destination
output_dir	Output directory
Running Tests

The project includes offline unit tests that use sample HTML and JSON files.

Run all tests with:

bash
python -m unittest discover tests


The tests do not require:

An internet connection.
A running browser.
Access to Booking.com.
Troubleshooting
Could not identify a Booking search page

Booking.com may have displayed a CAPTCHA or robot verification page.

Solution:

Run the scraper without --headless.
Complete the verification in the browser window.
Allow the scraper to continue.

The scraper waits up to 90 seconds for manual verification.

Missing prices or fewer reviews in headless mode

Booking.com may serve reduced page content to headless browsers.

Solution:

Run the scraper in normal browser mode without --headless.

No results or empty output

Booking.com periodically changes its website structure and HTML selectors.

Solution:

Check whether the page structure has changed.
Review the selectors near the beginning of the relevant functions in scraping.py and hotel_page.py.
Update the selectors if necessary.
Review API stops working

The internal review endpoint may change or become unavailable.

Solution:

The scraper attempts to fall back to embedded page data and rendered review cards, followed by the full reviews panel.

However, these alternatives may be slower and return fewer reviews.

Responsible Use

Please use this project responsibly.

Review Booking.com's Terms and Conditions before scraping.
Keep request volumes low and avoid excessive traffic.
Respect website restrictions and access controls.
Do not republish guest reviews or personal information without appropriate authorization.
Use collected information for legitimate learning and research purposes.

Important: This scraper relies partly on an internal Booking.com GraphQL endpoint (ReviewList). This is not a public or officially documented API. Booking.com may change, restrict, or remove it at any time.

The scraper includes alternative extraction methods, but their availability and completeness depend on Booking.com's current website structure.

Disclaimer

This project is intended for educational and research purposes.

It is not affiliated with, endorsed by, or officially associated with Booking.com.

Users are responsible for ensuring that their use of this software complies with applicable laws, website terms, and data protection requirements.

License

Add your chosen open-source license to the repository (for example, MIT License) and include the corresponding LICENSE file.
