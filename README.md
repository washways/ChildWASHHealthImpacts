# Child Health Burden – WASH Focus (GBD 2023 Explorer)

This repository contains a lightweight, browser-based tool to explore how children are affected by different causes of morbidity and mortality across age bands, with a specific focus on **Water, Sanitation and Hygiene (WASH)**–related causes.

It is designed as an ** analytical prototype**, built in **pure HTML, JavaScript and CSS**, using a single-page app pattern. No backend is required – everything runs in the browser.

---

## 1. Purpose

The tool helps to answer questions like:

* *At what ages do WASH-related diseases account for the largest share of child DALYs or deaths?*
* *Which specific causes account for most of the WASH-attributable burden in a given country?*
* *How does the WASH-related share compare to other causes across childhood and adolescence?*

It is primarily intended for:

* **UNICEF WASH and Health teams** preparing briefs, slides, and talking points.
* **Policy and advocacy discussions** with Ministries of Health, Water, and Finance.
* **Internal brainstorming** on where WASH investments might yield the biggest health gains by age.

> This is **not** a polished public-facing product; it’s an exploratory tool to support analysis and dialogue.

---

## 2. What the App Does

The app:

* Loads a **Global Burden of Disease (GBD 2023)** extract from a CSV file.
* Filters records to:

  * **Latest available year** in the data.
  * **Sex = “Both”**.
  * **Child age bands**:
    `<28 days`, `1–5 months`, `6–11 months`, `12–23 months`, `2–4 years`, `5–9 years`, `10–14 years`, `15–19 years`.
* Classifies each cause into one of three **WASH buckets**:

  * **WASH Core** – strongly and directly WASH-driven.
  * **WASH Extended** – partial, but important, WASH/hygiene/IPC linkage.
  * **Non-WASH** – everything else.
* Provides interactive filters:

  * Country, measure (e.g. DALYs / deaths), metric (Number / Percent).
  * Child age bands (with a “Select all” shortcut).
  * Top N causes (slider).
  * Include/exclude WASH buckets.
  * Free-text cause filter.
* Displays three main charts:

1. **Chart 1 – Causes vs age (stacked by age)**
   Top causes, with each bar split by age band.

2. **Chart 3 – Causes vs WASH bucket**
   Top causes, with each bar split into WASH Core / Extended / Non-WASH.

3. **Chart 2 – Age vs WASH bucket**
   For each age band, shows the split between WASH Core / Extended / Non-WASH, either as:

   * Absolute burden, or
   * Shares (% of total burden in that age band).

* Shows a **summary sentence** of overall WASH Core and WASH Core+Extended share for the selected country and filters.
* Includes an **introductory splash screen** explaining methodology, which can be dismissed and optionally hidden on subsequent visits (using `localStorage`).

---

## 3. Tech Stack & How It Works

Everything lives in a single `index.html` file:

* **HTML** – Layout, controls, splash screen, chart containers.
* **CSS** – UNICEF-inspired styling, responsive layout, splash overlay.
* **JavaScript**:

  * [PapaParse](https://www.papaparse.com/) (CDN) to load and parse the CSV (`IHMEGBD2023.csv`).
  * [Chart.js](https://www.chartjs.org/) (CDN) for stacked bar charts.
  * Custom logic for:

    * Data filtering and aggregation.
    * WASH attribution (`WASH Core`, `WASH Extended`, `Non-WASH`).
    * UI interactions (filters, reset, age selection, splash).

### 3.1 Data Flow (High Level)

1. **On load**

   * The splash screen appears.
   * The app downloads `IHMEGBD2023.csv` via `Papa.parse(...)`.
   * It parses rows, coerces numeric fields (`year`, `val`), and identifies the **latest year**.
   * It filters to `sex_name === "Both"` and that latest year.
   * For each row, it adds a `wash_bucket` field using the mapping function.

2. **Controls are populated**

   * `countrySelect` from unique `location_name`.
   * `measureSelect` from unique `measure_name` (e.g. `Deaths`, `DALYs (Disability-Adjusted Life Years)`).
   * `metricSelect` from `metric_name`, **excluding** `"Rate"` (to avoid confusion).
   * Age multi-select from the defined child age bands present in the data.

3. **When filters change**

   * The app builds a **filtered subset** based on:

     * Country, measure, metric.
     * Selected age bands.
     * Selected WASH buckets.
     * Optional cause text filter.
   * It recomputes aggregations and rebuilds all three charts:

     * Chart 1: `cause × age`.
     * Chart 3: `cause × wash_bucket`.
     * Chart 2: `age × wash_bucket`.
   * It updates the summary text with overall WASH shares.

4. **Charts**

   * Implemented with Chart.js stacked horizontal bars.
   * Legends for age bands (Chart 1) or WASH buckets (Charts 2 & 3).
   * Tooltips showing raw values and percentages (per cause or per age).
   * Titles adjust automatically when only one age band is selected (e.g. `– age: 5–9 years`).

---

## 4. WASH Attribution Methodology

The mapping is deliberately **simple and transparent** and is implemented in JavaScript in a single function `getWashBucket(causeName)`.

### 4.1 WASH Core

Causes that are **strongly and directly** WASH-driven – primarily faecal–oral or unsafe excreta/water exposure:

* Diarrheal diseases
* Typhoid fever
* Paratyphoid fever
* Typhoid and paratyphoid fevers (combined category in some GBD extracts)
* Ascariasis
* Trichuriasis
* Hookworm disease
* Schistosomiasis
* Intestinal nematode infections
* Invasive Non-typhoidal Salmonella (iNTS)
* Other intestinal infectious diseases

**Interpretation**
These are the classic **“WASH wins”** – improving water quality, sanitation, hygiene, and safe excreta management should substantially reduce these burdens.

### 4.2 WASH Extended

Causes where WASH / hygiene / infection prevention and control (IPC) play an important but **partial** role:

* Lower respiratory infections
* Upper respiratory infections
* Otitis media
* Neonatal sepsis and other neonatal infections
* Maternal sepsis and other maternal infections
* Cellulitis
* Other skin and subcutaneous diseases
* Tetanus
* Urinary tract infections and interstitial nephritis

**Interpretation**
These conditions are influenced by WASH and hygiene (e.g. overcrowding, sanitation, handwashing, IPC in health facilities) but also by crowding, vaccination, clinical care, and other broader health-system factors.

### 4.3 Non-WASH

Everything else falls into **Non-WASH** – not because WASH is irrelevant, but because it is **not the primary or dominant lever** (e.g., injuries, congenital anomalies, nutritional disorders, cancers).

---

## 5. Data Requirements

The app expects a CSV with a GBD-style schema, including (at minimum):

* `measure_name` – e.g. `"Deaths"`, `"DALYs (Disability-Adjusted Life Years)"`
* `metric_name` – `"Number"`, `"Rate"`, `"Percent"` (the UI will only expose Number and Percent)
* `location_name` – country name
* `sex_name` – `"Both"`, `"Male"`, `"Female"`
* `age_name` – standard GBD age bands (must include child bands)
* `cause_name` – cause labels that match those in the WASH mapping
* `year` – integer year
* `val` – numeric metric value

If you start from an Excel file such as `IHMEGBD2023.xlsx`:

1. Export to CSV: **`IHMEGBD2023.csv`**.
2. Place it in the **repo root**, next to `index.html`.

---

## 6. Getting Started

### 6.1 Clone the Repository

```bash
git clone https://github.com/your-org/child-wash-gbd-explorer.git
cd child-wash-gbd-explorer
```

### 6.2 Add the Data

* Export your GBD extract from Excel as `IHMEGBD2023.csv`.
* Copy it into the **repository root**, next to `index.html`.

### 6.3 Run a Local Web Server

You can use Python’s built-in HTTP server:

```bash
# If "python" refers to Python 3 on your system:
python -m http.server 8000

# Otherwise:
python3 -m http.server 8000
```

Then open:

```text
http://localhost:8000
```

in your browser.

> Opening `index.html` via `file://` will **not** work for CSV loading because of browser security; you must use an HTTP server.

---

## 7. Using the App

### 7.1 Splash Screen

On first load, you’ll see a **methodology splash screen**:

* Explains the data source, WASH attribution, and interpretation caveats.
* You can tick **“Don’t show this again on this device”** and click **“Start exploring”** to hide it permanently (for that browser/device).

### 7.2 Main Controls (Left Panel)

* **Country**
  Choose a country from the GBD extract.

* **Measure**
  Select `"DALYs (Disability-Adjusted Life Years)"` or `"Deaths"` (depending on what’s in your data).

* **Metric**
  Choose between:

  * `"Number"` – absolute counts.
  * `"Percent"` – proportions of the relevant total.
    (Rate is intentionally hidden.)

* **Child age bands**
  Multi-select; you can:

  * Use Ctrl/Cmd + click to select multiple.
  * Use the **“All”** button to quickly select all child bands.

* **Display mode (Chart 2)**

  * *Absolute burden* – uses raw `val` values.
  * *WASH share (%)* – each age band’s bar sums to 100%.

* **Top N causes**
  Slider (5–25) controlling how many causes appear in Charts 1 & 3.

* **Include WASH buckets**
  Checkboxes to include/exclude:

  * WASH Core
  * WASH Extended
  * Non-WASH

* **Filter causes (text)**
  Free-text filter applied to `cause_name` (e.g. “diarrh”, “sepsis”).

* **Reset all filters**
  Restores defaults:

  * Country: `Malawi` (if present), otherwise the first country.
  * Measure: `DALYs (Disability-Adjusted Life Years)` (if present), otherwise first measure.
  * Metric: `Number` (if present), otherwise first metric.
  * All child ages selected.
  * All WASH buckets included.
  * Top N = 10.
  * Empty cause filter.
  * Display mode = Absolute burden.

### 7.3 Charts

1. **Chart 1 – Top causes by age band (stacked by age)**

   * Horizontal stacked bars.
   * Each bar = a cause.
   * Segments = age bands (ordered by child age).
   * Title includes country, measure, metric, year, and age selection (e.g. `– age: 5–9 years` or `– ages: <28 days to 19 years`).

2. **Chart 3 – Top causes by WASH bucket**

   * Horizontal stacked bars.
   * Each bar = a cause.
   * Segments = WASH Core / WASH Extended / Non-WASH.
   * Helps you see which of the top causes are WASH vs non-WASH.

3. **Chart 2 – WASH share by age**

   * Horizontal stacked bars.
   * Each bar = an age band.
   * Segments = WASH Core / WASH Extended / Non-WASH.
   * In *Absolute* mode, x-axis is the chosen measure/metric value.
   * In *Share (%)* mode, each age band sums to 100%, making it easy to compare **relative** WASH importance across ages.

At the bottom, a summary line reports:

> In **{country}**, WASH Core causes account for **X%** of the selected child {measure}, and WASH Core + Extended account for **Y%**.

---

## 8. Suggestions for Improvement / Roadmap

Some concrete improvements to consider:

### 8.1 WASH Attribution Refinement

* **Country-specific mapping**
  Allow users to upload a JSON or edit an on-screen mapping table to adjust which causes are considered WASH Core/Extended for their context.

* **Probabilistic attribution**
  Instead of a hard bucket, assign each cause a fractional WASH share (e.g. 0.7, 0.3) and compute “WASH-attributable” burden as a continuous measure.

* **Facility vs community attribution**
  For neonatal and maternal sepsis, separate WASH/IPC in health facilities vs community-level WASH views.

### 8.2 Data & UI Enhancements

* Support **multiple years** with a year selector (and possibly trends).
* Add **multi-country comparison** (e.g. faceted charts, small multiples).
* Allow **export** of the filtered dataset and chart images (PNG/SVG) for easy use in slide decks.
* Add a **notes panel** for country-specific caveats.

### 8.3 Technical Improvements

* Split the single file into:

  * `index.html`
  * `styles.css`
  * `app.js`

* Add a minimal **build/serve script** (e.g. npm + `lite-server` or Vite) while still keeping it framework-free.

* Introduce basic **unit tests** for:

  * WASH mapping logic.
  * Aggregation functions (cause × age, age × WASH).

* Replace the `<select multiple>` age control with a more **mobile-friendly** pill-based toggle UI.

### 8.4 Accessibility & Localization

* Improve **keyboard navigation** and ARIA attributes.
* Add **language support** (e.g. English/French) via a simple translation dictionary and a language toggle.
* Offer a **colourblind-safe palette** option while keeping UNICEF-style branding.

---

## 9. Limitations & Caveats

* This tool **does not replace** formal burden-of-disease modelling or national surveillance.
* WASH attribution is **simplified** and intentionally coarse – it is meant as a **conversation starter**, not a definitive attribution model.
* GBD cause definitions and categories can change; mappings may need updating if cause names or hierarchies change.
* Only **child age bands** are included; adult ages are filtered out in this prototype.

---

## 10. Acknowledgements

* **Data**: Global Burden of Disease Study 2023 (GBD 2023) Results, Institute for Health Metrics and Evaluation (IHME).
* **Concept & implementation**: UNICEF WASH team prototype for internal analytical use.
