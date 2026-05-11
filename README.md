# JMeter Performance Testing Framework

> **Apache JMeter 5.6.3** test suite covering four progressive assignments — from basic load testing to full CI/CD integration. All tests are wired to a **single GitHub Actions workflow** that runs everything with one click.

---

## Table of Contents

- [Repository Structure](#repository-structure)
- [Prerequisites](#prerequisites)
- [Assignment 1 – First Demo Load Test](#assignment-1--first-demo-load-test)
- [Assignment 2 – Data-Driven Search](#assignment-2--data-driven-search)
- [Assignment 3 – Performance Test Types](#assignment-3--performance-test-types)
- [Assignment 4 – Petstore API Load Test](#assignment-4--petstore-api-load-test)
- [CI/CD – Run All Tests with One Click](#cicd--run-all-tests-with-one-click)
- [Running Tests Locally](#running-tests-locally)
- [Viewing Reports](#viewing-reports)

---

## Repository Structure

```
├── .github/
│   └── workflows/
│       └── run-all-jmeter-tests.yml    ← Single-click CI/CD workflow
│
├── Assignment # 01/
│   ├── Assignment-1-First_DEMO_LoadTest.jmx
│   ├── results.jtl
│   └── report/                         ← Pre-generated HTML report
│
├── Assignment # 02/
│   ├── Assignment-2-The-Data-Driven-Search.jmx
│   ├── data.csv                        ← Search keyword test data
│   ├── results2.jtl
│   └── report2/                        ← Pre-generated HTML report
│
├── Assignment # 03/
│   ├── Assignment-3-LoadText.jmx
│   ├── Assignment-3-SoakTest.jmx
│   ├── Assignment-3-SpikeTest.jmx
│   └── Assignment-3-StressTest.jmx
│
└── Assignment # 04/
    ├── petstore-api-test.jmx
    └── CI-CD-SETUP.md
```

---

## Prerequisites

| Tool | Version | Purpose |
|---|---|---|
| Apache JMeter | 5.6.3 | Test execution engine |
| Java (JDK/JRE) | 17+ | Required by JMeter |
| jp@gc Plugins | latest | Required by Assignment 3 thread groups |

**Install jp@gc Plugins** (for Assignment 3 only):
1. Open JMeter GUI
2. Go to **Options → Plugins Manager**
3. Install: `jpgc-casutg` (Ultimate Thread Group) and `jpgc-tst` (Stepping Thread Group)

---

## Assignment 1 – First Demo Load Test

**File:** `Assignment # 01/Assignment-1-First_DEMO_LoadTest.jmx`  
**Target:** `https://demowebshop.tricentis.com`

### What It Tests
Simulates a realistic e-commerce user journey on the Tricentis Demo Web Shop under sustained load.

### User Flow (Transaction: "Test")
1. **Dashboard** — `GET /` — loads the homepage
2. **Get Product** — `GET /141-inch-laptop` — views a product page
3. **Add to Cart** — `POST /addproducttocart/details/31/1` — adds the item (qty: 1)
4. **Get Cart** — `GET /cart` — validates the cart page

### Thread Configuration
| Parameter | Value |
|---|---|
| Virtual Users (Threads) | 10 |
| Ramp-up Period | 10 seconds |
| Test Duration | 300 seconds (5 minutes) |
| Loop | Infinite (scheduler-controlled) |
| On Sample Error | Continue |

### Config Elements
- **HTTP Header Manager** — Chrome 146 User-Agent, `sec-ch-ua`, Sec-Fetch headers
- **HTTP Cookie Manager** — clears cookies each iteration
- **HTTP Cache Manager** — clears cache each iteration
- **DNS Cache Manager** — clears DNS each iteration
- **HTTP Authorization Manager** — present for auth-ready extension
- **Constant Timer** — 6.9 s think time on the product page (disabled by default)

---

## Assignment 2 – Data-Driven Search

**File:** `Assignment # 02/Assignment-2-The-Data-Driven-Search.jmx`  
**Data File:** `Assignment # 02/data.csv`  
**Target:** `https://demowebshop.tricentis.com`

### What It Tests
Parameterised search flows where each virtual user searches with a different keyword read from a CSV file, verifying the keyword appears in the response.

### CSV Data (`data.csv`)
```
keywords
computer
book
laptop
electronics
gift card
```

### User Flow (Transaction: "Test")
1. **Dashboard** — `GET /` — loads homepage
2. **Search Item** — `GET /search?q=${keywords}` — searches with the CSV keyword
3. **Response Assertion** — verifies the keyword string appears in the response body

### Thread Configuration
| Parameter | Value |
|---|---|
| Virtual Users (Threads) | 5 |
| Ramp-up Period | 1 second |
| Loops per Thread | 2 |
| CSV Share Mode | All threads |
| CSV Recycle | Yes (loops back when EOF reached) |
| Ignore CSV Header | Yes |
| On Sample Error | Continue |

### Key JMeter Elements
- **CSV Data Set Config** — reads `data.csv`, maps column to `${keywords}` variable
- **Response Assertion** — `Contains` check: asserts `${keywords}` is present in the response data

---

## Assignment 3 – Performance Test Types

All four tests target `https://test.k6.io` and use the jp@gc plugin thread groups.

---

### 3a – Load Test

**File:** `Assignment # 03/Assignment-3-LoadText.jmx`

Measures system performance under an expected, sustained load profile.

| Parameter | Value |
|---|---|
| Thread Group | jp@gc Ultimate Thread Group |
| Threads | 30 |
| Ramp-up | 120 seconds |
| Hold Duration | 300 seconds |
| Ramp-down | 120 seconds |
| Loop | Infinite (duration-controlled) |

---

### 3b – Soak Test

**File:** `Assignment # 03/Assignment-3-SoakTest.jmx`

Runs a low user count for an extended period to detect memory leaks, resource degradation, and slow performance decay over time.

| Parameter | Value |
|---|---|
| Thread Group | Standard Thread Group |
| Virtual Users | 5 |
| Ramp-up | 43,200 seconds (12 hours) |
| Loop | Infinite |
| Throughput Control | Constant Throughput Timer — 10 req/min |

---

### 3c – Spike Test

**File:** `Assignment # 03/Assignment-3-SpikeTest.jmx`

Hits the system with a sudden, instant burst of concurrent users (all at once) to test behaviour under sudden traffic surges.

| Parameter | Value |
|---|---|
| Thread Group | Standard Thread Group |
| Virtual Users | 40 |
| Ramp-up | 5 seconds |
| Loops | 1 |
| Synchronizing Timer | Releases all 40 threads simultaneously |

> The **Synchronizing Timer** (`groupSize=40`) holds all threads at a barrier until all 40 are ready, then releases them at exactly the same instant — creating a true spike.

---

### 3d – Stress Test

**File:** `Assignment # 03/Assignment-3-StressTest.jmx`

Gradually steps up load beyond normal capacity to find the breaking point of the system.

| Parameter | Value |
|---|---|
| Thread Group | jp@gc Stepping Thread Group |
| Max Threads | 50 |
| Start Users Count | 10 (initial batch) |
| Add Users Every | 60 seconds |
| Users Added Per Step | 10 |
| Hold Time at Peak | 180 seconds |
| Ramp-up per step | 1 second |
| Stop Users | 10 every 30 seconds |

---

## Assignment 4 – Petstore API Load Test

**File:** `Assignment # 04/petstore-api-test.jmx`  
**Target:** `https://petstore.swagger.io/v2`

### What It Tests
REST API load test against the Swagger Petstore, exercising a create-then-read pattern with HTTP response assertions.

### API Flow
1. **POST /v2/pet** — Creates a new pet with JSON body:
   ```json
   {
     "id": 123,
     "category": { "id": 1, "name": "Dogs" },
     "name": "JMeter Test Dog",
     "photoUrls": ["https://example.com/photo.jpg"],
     "tags": [{ "id": 1, "name": "test" }],
     "status": "available"
   }
   ```
   → **Assertion:** HTTP 200

2. **GET /v2/pet/123** — Retrieves the pet by ID  
   → **Assertion:** HTTP 200

### Thread Configuration
| Parameter | Value |
|---|---|
| Virtual Users | 50 |
| Ramp-up | 120 seconds |
| Loops per Thread | 20 |
| On Sample Error | Continue |
| Content-Type Header | `application/json` |
| Variable `${petId}` | `123` |

---

## CI/CD – Run All Tests with One Click

**Workflow file:** `.github/workflows/run-all-jmeter-tests.yml`

### Architecture

```
workflow_dispatch (one click)
        │
        ├── Job: assignment-1-load-test          (parallel)
        ├── Job: assignment-2-data-driven         (parallel)
        ├── Job: assignment-3-performance-tests   (parallel, matrix × 4)
        │       ├── LoadTest
        │       ├── SoakTest
        │       ├── SpikeTest
        │       └── StressTest
        ├── Job: assignment-4-petstore-api        (parallel)
        │
        └── Job: summary  ← waits for all, prints pass/fail table
```

### Triggers
| Trigger | When |
|---|---|
| `workflow_dispatch` | Manual — "Run workflow" button |
| `push` | Any push to `main` or `master` |
| `pull_request` | PR targeting `main` or `master` |

### What Each Job Does
1. Checks out the repository
2. Installs Java 17 (Temurin)
3. Downloads and installs JMeter 5.6.3
4. (Assignment 3 only) Downloads jp@gc PluginsManager + installs `jpgc-casutg`
5. Runs JMeter in non-GUI mode (`jmeter -n`)
6. Generates JTL results + HTML dashboard report
7. Uploads artifacts (30-day retention)

### Artifacts Produced
| Artifact Name | Contents |
|---|---|
| `assignment-1-results` | `results.jtl` + HTML report |
| `assignment-2-results` | `results.jtl` + HTML report |
| `assignment-3-LoadTest-results` | `results.jtl` + HTML report |
| `assignment-3-SoakTest-results` | `results.jtl` + HTML report |
| `assignment-3-SpikeTest-results` | `results.jtl` + HTML report |
| `assignment-3-StressTest-results` | `results.jtl` + HTML report |
| `assignment-4-results` | `results.jtl` + HTML report |

### How to Trigger

#### Option A — Manual (One Click)
1. Go to your repo on GitHub
2. Click **Actions** tab
3. Select **Run All JMeter Assignments** in the left sidebar
4. Click **Run workflow** → **Run workflow**

#### Option B — On Push
Any push to `main`/`master` automatically triggers all assignments.

> **First-time setup on a forked repo:** GitHub disables Actions on forks by default.  
> Go to **Actions** tab → click **"I understand my workflows, go ahead and enable them"**

---

## Running Tests Locally

```bash
# Assignment 1
jmeter -n \
  -t "Assignment # 01/Assignment-1-First_DEMO_LoadTest.jmx" \
  -l results-a1.jtl \
  -e -o report-a1/

# Assignment 2
jmeter -n \
  -t "Assignment # 02/Assignment-2-The-Data-Driven-Search.jmx" \
  -l results-a2.jtl \
  -e -o report-a2/

# Assignment 3 – Load Test
jmeter -n \
  -t "Assignment # 03/Assignment-3-LoadText.jmx" \
  -l results-a3-load.jtl \
  -e -o report-a3-load/

# Assignment 3 – Soak Test
jmeter -n \
  -t "Assignment # 03/Assignment-3-SoakTest.jmx" \
  -l results-a3-soak.jtl \
  -e -o report-a3-soak/

# Assignment 3 – Spike Test
jmeter -n \
  -t "Assignment # 03/Assignment-3-SpikeTest.jmx" \
  -l results-a3-spike.jtl \
  -e -o report-a3-spike/

# Assignment 3 – Stress Test
jmeter -n \
  -t "Assignment # 03/Assignment-3-StressTest.jmx" \
  -l results-a3-stress.jtl \
  -e -o report-a3-stress/

# Assignment 4
jmeter -n \
  -t "Assignment # 04/petstore-api-test.jmx" \
  -l results-a4.jtl \
  -e -o report-a4/
```

> Ensure `JMETER_HOME/bin` is on your `PATH` and jp@gc plugins are installed before running Assignment 3.

---

## Viewing Reports

### Pre-generated Reports (checked into repo)
| Assignment | Report Path |
|---|---|
| Assignment 1 | `Assignment # 01/report/index.html` |
| Assignment 2 | `Assignment # 02/report2/index.html` |

Open `index.html` in any browser — no server needed.

### CI-generated Reports
Download from **GitHub Actions → your run → Artifacts** section.
Extract the zip and open `index.html`.

### JTL Results
Raw `.jtl` files can be loaded into:
- JMeter GUI → **Add → Listener → View Results Tree** (load existing results)
- [JMeter Dashboard Generator](https://jmeter.apache.org/usermanual/generating-dashboard.html): `jmeter -g results.jtl -o report/`

---

## Author

**iqraimtiaz-tech** · iqra.imtiaz@azm.dev
