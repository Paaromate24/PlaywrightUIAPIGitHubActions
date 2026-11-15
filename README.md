# PlaywrightUIAPIGitHubActions
To Automate UI &amp; API using Playwright and TypeScript and run in Git Hub Actions

### * This Project is for personal skills purpose ONLY and should NOT be used by ANYONE

Acknowledgments
* This project was inspired by and built with guidance from **[Naveen Kuntla](https://naveenautomationlabs.com/)** as part of **[Playwright Javascript/Typescript Training]**.

## Application Overview

A combined **UI + API test automation framework** built with Playwright and TypeScript.

* **UI (Web) under test:** the [OpenCart demo store](https://naveenautomationlabs.com/opencart/) – login, home/account page, product search, product details and common page elements (logo, search box, cart, footers).
* **API under test:** [GoRest](https://gorest.co.in) user CRUD APIs, [Restful-Booker](https://restful-booker.herokuapp.com) booking APIs and the Spotify OAuth2 (client credentials) + Albums API.
* **Network interception:** logging and mocking (fake JSON / fake HTML) of browser requests with `page.route`.
* Runs on multiple environments (`dev`, `qa`, `stage`) through `config/.env.<ENV>` files and executes in **GitHub Actions** (`.github/workflows`) and **Jenkins** (`Jenkinsfile`).
* Produces Playwright HTML, Allure and Reporting Labs reports.

## Tech Stack

| Category | Technology |
|----------|------------|
| Language | TypeScript (ES modules) |
| Test Framework | [Playwright Test](https://playwright.dev) (`@playwright/test`) |
| Runtime | Node.js |
| Design Pattern | Page Object Model + custom Playwright fixtures (`pagefixtures`, `apifixtures`) + `ApiHelper` |
| Data-Driven Testing | CSV (`csv-parse`), Excel (`xlsx`), JSON |
| API Schema Validation | `ajv` (JSON Schema) |
| Environment Config | `dotenv` (`config/.env.<ENV>`) |
| Reporting | Playwright HTML, Allure (`allure-playwright`, `allure-commandline`), `reporting-labs` |
| CI/CD | GitHub Actions (smoke / regression / sharded runs), Jenkins |

## Project Structure

```text
PlaywrightUIAPIGitHubActions
├── .github/workflows/        # GitHub Actions: e2e.yml, master.yml, playwright.yml, sharded-tests.yml
├── config/                   # Environment files: .env.dev, .env.qa, .env.stage, .env.example
├── src/
│   ├── api/ApiHelper.ts      # Generic GET/POST/PUT/DELETE wrapper over APIRequestContext
│   ├── fixtures/             # pagefixtures.ts (page objects + CSV data), apifixtures.ts (apiHelper)
│   ├── pages/                # BasePage, LoginPage, HomePage, SearchResultsPage, ProductInfoPage, RegisterPage, DemoPage
│   ├── schema/userschema.json# JSON schema for user API responses
│   ├── testdata/             # logindata.csv/.json, opencarttestdata.xlsx, product.csv
│   └── utils/                # CsvHelper, ExcelHelper, JsonHelper
├── tests/
│   ├── homepage.spec.ts, loginpage.spec.ts          # Legacy UI specs (skipped)
│   ├── homepagefix.spec.ts, loginpagefix.spec.ts    # UI specs using fixtures
│   ├── productpage.spec.ts, searchpage.spec.ts      # UI specs using fixtures
│   └── api/                  # API + network interception specs
├── Jenkinsfile               # Jenkins pipeline
├── playwright.config.ts      # Playwright config (env loading, reporters, projects)
├── reporting-labs.config.ts  # Reporting Labs config
└── package.json
```

## Running the Tests

**Prerequisites:** Node.js and npm.

```bash
# 1. Install dependencies and browsers
npm ci
npx playwright install chromium

# 2. Configure the environment (copy config/.env.example and fill in the values)
#    BASE_URL, USERNAME, PASSWORD, API_BASE_URL, API_TOKEN,
#    OAUTH_CLIENT_ID, OAUTH_CLIENT_SECRET, GRANT_TYPE

# 3. Run the tests
npm test                                       # all tests (default ENV=qa)
ENV=stage npx playwright test                  # choose an environment (dev | qa | stage)
npm run test:headed                            # headed mode
npm run test:chrome                            # chromium project only
npx playwright test --grep @smoke              # smoke tests
npx playwright test --grep @regression         # regression tests
npx playwright test tests/api                  # API tests only
npx playwright test tests/loginpagefix.spec.ts # a single spec file

# 4. Reports
npx playwright show-report reports/html-report # Playwright HTML report
npm run allure:report                          # generate and open the Allure report
npm run allure:clean                           # clean Allure results
```

> Locally tests run headed; when `CI` is set they run headless with 2 workers and 2 retries.

## Testing Scenarios

Status: **Active** = enabled, **Skipped** = marked `test.skip` in code. Results are not recorded here – run the suite for the latest pass/fail status.

### UI Scenarios

| S.No | Scenario | Test Data/Input | Expected Result | Tags | spec File | Status |
|------|----------|-----------------|-----------------|------|-----------|--------|
| 1 | Login page title | Open login page | Title is `Account Login` | @smoke | [loginpagefix.spec.ts](tests/loginpagefix.spec.ts) | Active |
| 2 | Forgotten password link exists | Open login page | Link is displayed | @regression | [loginpagefix.spec.ts](tests/loginpagefix.spec.ts) | Active |
| 3 | Login with valid credentials | `USERNAME` / `PASSWORD` from `.env` | Logout link visible; page title is `My Account` | @regression | [loginpagefix.spec.ts](tests/loginpagefix.spec.ts) | Active |
| 4 | Login with invalid credentials (fixture data) | Rows from `logindata.csv` via `testData` fixture | Invalid login error is displayed for every row | – | [loginpagefix.spec.ts](tests/loginpagefix.spec.ts) | Active |
| 5 | Login with invalid credentials (CSV) | `logindata.csv`: `invalid@open.com`/`wrong123`, empty user/`pw1234`, `pwapril@pw.com`/empty password | Invalid login error is displayed (one test per row) | @regression | [loginpagefix.spec.ts](tests/loginpagefix.spec.ts) | Active |
| 6 | Login with invalid credentials (Excel) | `opencarttestdata.xlsx`, sheet `login` | Invalid login error is displayed (one test per row) | – | [loginpagefix.spec.ts](tests/loginpagefix.spec.ts) | Active |
| 7 | Login with invalid credentials (JSON) | `logindata.json`: `wrong@open.com`/`wrong@123`, `invalid@open.com`/empty, `pwapril@pw.com`/`wrong@123` | Invalid login error is displayed (one test per row) | – | [loginpagefix.spec.ts](tests/loginpagefix.spec.ts) | Active |
| 8 | Logo, search box, cart and footers on login page | Open login page | Logo, search box and cart visible; 16 footer links | @smoke | [loginpagefix.spec.ts](tests/loginpagefix.spec.ts) | Active |
| 9 | Home page title | Logged in with `USERNAME` / `PASSWORD` | Title is `My Account` | – | [homepagefix.spec.ts](tests/homepagefix.spec.ts) | Active |
| 10 | Logout link exists | Logged in user | Logout link is displayed | – | [homepagefix.spec.ts](tests/homepagefix.spec.ts) | Active |
| 11 | Home page headers | Logged in user | Headers: `My Account`, `My Orders`, `My Affiliate Account`, `Newsletter` | – | [homepagefix.spec.ts](tests/homepagefix.spec.ts) | Active |
| 12 | Logo, search box, cart and footers (post login) | Logged in user | Logo, search box and cart visible; 16 footer links | – | [homepagefix.spec.ts](tests/homepagefix.spec.ts), [productpage.spec.ts](tests/productpage.spec.ts), [searchpage.spec.ts](tests/searchpage.spec.ts) | Active |
| 13 | Product header | Search `macbook`, open `MacBook Pro` | Header is `MacBook Pro` | – | [productpage.spec.ts](tests/productpage.spec.ts) | Active |
| 14 | Product images count | Search `macbook`, open `MacBook Pro` | 4 images | – | [productpage.spec.ts](tests/productpage.spec.ts) | Active |
| 15 | Product information | Search `macbook`, open `MacBook Pro` | Brand `Apple`, Product Code `Product 18`, Reward Points `800`, Availability `Out Of Stock`, price and ex-tax price `$2,000.00` | – | [productpage.spec.ts](tests/productpage.spec.ts) | Active |
| 16 | Search results count | `product.csv`: macbook (3), imac (1), samsung (2), canon (1) | Result count equals `resultcount` (one test per row) | – | [searchpage.spec.ts](tests/searchpage.spec.ts) | Active |
| 17 | Land on product page from search | `product.csv` search key + product name | Page title equals the product name (one test per row) | – | [searchpage.spec.ts](tests/searchpage.spec.ts) | Active |
| 18 | Legacy login page tests (title, forgot password link, valid login) | `pwapril@pw.com` / `pw123` | Title `Account Login`, link visible, login succeeds | – | [loginpage.spec.ts](tests/loginpage.spec.ts) | Skipped |
| 19 | Legacy home page tests (title, logout link, headers) | `pwapril@pw.com` / `pw123` | Title `My Account`, logout link, 4 headers | – | [homepage.spec.ts](tests/homepage.spec.ts) | Skipped |

### API & Network Interception Scenarios

| S.No | Scenario | Test Data/Input | Expected Result | Tags | spec File | Status |
|------|----------|-----------------|-----------------|------|-----------|--------|
| 20 | GoRest GET all users | `GET /public/v2/users` with token | 200; at least 1 user | @smoke @regression | [user.api.spec.ts](tests/api/user.api.spec.ts) | Active |
| 21 | GoRest create user | `POST /public/v2/users` with name, unique email, gender `male`, status `active` | 201; user id returned | @regression | [user.api.spec.ts](tests/api/user.api.spec.ts) | Active |
| 22 | GoRest update user | `PUT /public/v2/users/{id}` with new name, status `inactive` | 200; name and status updated | @regression | [user.api.spec.ts](tests/api/user.api.spec.ts) | Active |
| 23 | GoRest delete user | `DELETE /public/v2/users/{id}` | 204 | @regression | [user.api.spec.ts](tests/api/user.api.spec.ts) | Active |
| 24 | Create user and verify with GET | POST user, then `GET /users/{id}` | 201, then 200 with name `apiautomation` | @regression | [user.api.individual.spec.ts](tests/api/user.api.individual.spec.ts) | Active |
| 25 | Update user and verify with GET | POST, GET, PUT (`apiautomation-update`, `inactive`), GET | 200s; updated name and status persisted | @regression | [user.api.individual.spec.ts](tests/api/user.api.individual.spec.ts) | Active |
| 26 | Delete user and verify | POST, GET, DELETE, GET | DELETE 204; subsequent GET 404 with `Resource not found` | @regression | [user.api.individual.spec.ts](tests/api/user.api.individual.spec.ts) | Active |
| 27 | Single user response schema | Create user, `GET /users/{id}`, validate with `userschema.json` (Ajv) | 200; response matches schema | @smoke | [user.api.schema.spec.ts](tests/api/user.api.schema.spec.ts) | Active |
| 28 | All users response schema | `GET /public/v2/users`, validate array of `userschema.json` | 200; every item matches schema | @smoke | [user.api.schema.spec.ts](tests/api/user.api.schema.spec.ts) | Active |
| 29 | Booking CRUD with token | `POST /auth` (`admin`/`password123`), create, update (price 121, `Lunch`), delete booking | Auth 200; create 200; update 200 with new values; delete 201 | @regression | [booking.api.spec.ts](tests/api/booking.api.spec.ts) | Active |
| 30 | Spotify OAuth2 token and album | Client credentials from `.env`; `GET /v1/albums/4aawyAB9vmqN3uQ7FjRGTy` | Token 200; album 200 with 3 images | @regression | [spotify.oauth2.spec.ts](tests/api/spotify.oauth2.spec.ts) | Active |
| 31 | Intercept and log requests | Navigate to OpenCart home page with `**/*` route | All requests logged and continued | – | [intercept.spec.ts](tests/api/intercept.spec.ts) | Active |
| 32 | Mock search with fake JSON | Route `route=product/search&search=macbook` fulfilled with fake products | Mocked JSON response served | @smoke | [intercept.spec.ts](tests/api/intercept.spec.ts) | Active |
| 33 | Mock search page with fake HTML | Route fulfilled with fake HTML (2 products) | Heading `Search Results`; products `Fake MacBook Pro`, `Fake iPhone 20`; prices `$599`, `$999` | @smoke | [intercept.spec.ts](tests/api/intercept.spec.ts) | Active |
| 34 | GoRest raw GET / POST / PUT / DELETE (practice) | Direct `request` calls to `gorest.co.in` | 200 / 201 / 200 / 204 | – | [users.api.practice.spec.ts](tests/api/users.api.practice.spec.ts) | Skipped |