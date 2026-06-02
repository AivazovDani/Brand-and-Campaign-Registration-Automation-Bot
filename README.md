Brand and Campaign Registration Automation Bot:

A Flask web application that automatically fills Google Forms with brand registration data pulled from Airtable for diffrent providers. Built for SMS campaign registration workflows — it reads brand and registration data from Airtable, opens a real Chrome browser, and fills the corresponding Google Form fields automatically.

What It Does

Different SMS carriers and registration platforms each have their own Google Forms for brand and campaign registration. This tool automates filling those forms by:

1. Pulling brand data from an Airtable Brands table -> example: Brand Name column: Paywise
2. Pulling campaign copies (texts) from an Airtable Registration table -> example: Zarkon | 10DLC | Paywise
3. Opening the target Google Form in a real Chrome browser
4. Filling all fields, selecting dropdowns/checkboxes/radio buttons, and waiting for manual review before submission

Project Structure

├── app.py
│   Flask web server — routes and entry points
│
├── google_form_selenium.py
│   Form filler for Campaigner (provider)
│
├── google_form_selenium_tcrpy.py
│   Form filler for TCR (provider)
│
├── google_form_selenium_bulktext.py
│   Form filler for BulkText (provider) campaign registration
│
├── zarkon_brand_registration.py
│   Form filler for Zarkon (provider) brand registration
│
├── bulktext_brand_registration.py
│   Form filler for BulkText (provider) brand registration
│
├── xeebi_new_campaign_stage_one.py
│   Xeebi (provider) campaign registration — Stage 1
│
├── xeebi_new_campaign_stage_two.py
│   Xeebi (provider) campaign registration — Stage 2
│
├── OLD_xeebi.py
│   Legacy Xeebi filler (kept for reference)
│
├── Dockerfile
│   Container setup with Chrome + virtual display
│
├── supervisord.conf
│   Process manager — starts Xvfb, VNC, Flask
│
├── requirements.txt
│   Python dependencies
│
└── .gitignore

Prerequisites

- Python 3.11+
- Google Chrome installed (handled automatically in Docker)
- An Airtable account with the required base structure (see below)

Setup

1. Clone the repository

bash
git clone <repo-url>
cd <repo-folder>


2. Create a .env file

env
AIRTABLE_API_KEY=your_airtable_api_key
AIRTABLE_BASE_ID=your_airtable_base_id

Get your API key from https://airtable.com/account. The Base ID is in your Airtable URL: https://airtable.com/app12345/...

3. Install dependencies

bash
pip install -r requirements.txt


4. Run locally

bash
python app.py


The app runs on http://localhost:5015.


Running with Docker:

The Docker setup includes a virtual display (Xvfb) so Chrome can run headlessly on a server, plus a noVNC interface so you can watch the browser fill forms in real time from your browser.

Build and run

bash
docker build -t form-filler .
docker run -p 5015:5015 -p 6080:6080 --env-file .env form-filler



How the container works

Supervisord manages four processes in order:

1. Xvfb — creates a virtual display at :99 (Chrome needs a screen even on a server)
2. x11vnc — exposes that virtual display as a VNC stream
3. noVNC — makes the VNC stream viewable in a browser at port 6080
4. Flask — starts the web app

Airtable Structure

The app expects two tables in your Airtable base:

Brands table design in Airtable:

| Field | Description |
| Name | Brand name (used to look up records) |
| Name (from Entity) | Legal company name |
| EIN/VAT (from Entity) | Tax ID / EIN |
| Owner name (from Entity) | Full name of the owner |
| Phone (from Entity) | Phone number |
| Address (from Entity) | Full address (street on line 1, City, State ZIP on line 2) |
| Domain | Website URL |
| Use-Case | SMS campaign use case description |
| Safe Page URL | Opt-in page URL |
| Message Flow | Call to action / message flow description |
| Opt-In Screenshot | URL to opt-in workflow image |
| Entity Name | Entity/LLC name |
| Entity name | (variant field used in some forms) |

Registration table design in Airtable:

| Field | Description |
| Name | Identifier (matched against brand name) |
| Registered copies | Sample SMS messages |

Flask Routes:

| Route | Method | Script | Description |
| / | GET | — | Home page |
| /run | POST | `google_form_selenium.py | Main campaigner form |
| /run-tcr | POST | google_form_selenium_tcrpy.py | TCR registration form |
| /run-bulk-text | POST | google_form_selenium_bulktext.py | BulkText campaign form |
| /run-zbr | POST | zarkon_brand_registration.py | Zarkon brand registration |
| /run-xeebi-stage-one | POST | xeebi_new_campaign_stage_one.py | Xeebi Stage 1 |
| /run-xeebi-stage-two | POST | xeebi_new_campaign_stage_two.py | Xeebi Stage 2 |
| /run-bl | POST | bulktext_brand_registration.py | BulkText brand registration |
| /run-xeebi-tfn | POST | n8n webhook | Xeebi TFN via n8n |
| /run-bind-10dlc | POST | n8n webhook | Bind 10DLC via n8n |
| /run-d7 | POST | n8n webhook | D7 via n8n |
| /run-reve-tfn | POST | n8n webhook | Reve TFN via n8n |
| /run-deliveryhub | POST | n8n webhook | DeliveryHub via n8n |
| /run-bindtfn | POST | n8n webhook | Bind TFN via n8n |


How the Form Filling Works:

Each script follows the this pattern:

1. Fetch record — looks up the brand by name in Airtable
2. Extract form data — maps Airtable fields to form question keywords, builds hardcoded values (opt-in/opt-out messages, help messages, etc.)
3. Launch Chrome — opens a real browser with automation-detection flags disabled
4. Fill fields — finds each form question by partial text match on the M7eMe CSS class (Google Forms' question label class), then fills the associated input or textarea
5. Handle special inputs — dropdowns, checkboxes, and radio buttons are handled separately since they can't be typed into
6. Wait for review — holds the browser open for 3 minutes so you can review and manually submit



Environment Variables:

| Variable | Required | Description |
| AIRTABLE_API_KEY | Yes | Your Airtable personal access token |
| AIRTABLE_BASE_ID | Yes | The ID of your Airtable base |



Notes:

- Forms are not auto-submitted — the browser stays open for 3 minutes after filling for manual review and submission
- Each script is independent and can be run directly from the command line for testing: python google_form_selenium.py
- The zarkon_brand_registration.py script uses undetected_chromedriver instead of standard Selenium to bypass bot detection on that specific form
- Some routes forward to n8n webhooks instead of running Selenium locally — those are for workflows that have been migrated to n8n automation
