# Auto Pulse — Global Automotive Intelligence

[Live dashboard](https://german-auto-pulse.vercel.app/) · [Monthly automation](.github/workflows/monthly-refresh.yml) · [CI tests](.github/workflows/ci.yml) · [Methodology](docs/methodology.md)

**Auto Pulse is an independent, publicly reproducible automotive data-analysis portfolio project.** It compares electric-vehicle adoption and sales in 32 national markets, including Germany, China, the United States, Japan, Norway and major European countries. The Germany deep dive adds published passenger-car registrations, brands, BEV/PHEV figures, manufacturing and factory exports.

### Explore the dashboard

- **Global overview:** choose a country and reporting year, scan four headline EV metrics, view contextual insights, search and filter country rankings.
- **Country comparison:** compare two countries using a consistent source, year and metric.
- **Germany:** explore national registration trends, car brands, powertrain shares, domestic production and factory exports.
- **Source and data quality:** read definitions, data periods, limitations, attribution and test results.

**Definitions matter:** EV adoption share = new battery-electric plus plug-in hybrid sales as a percentage of all new cars. EV sales are vehicles, not manufacturing. BEV share of EV sales is a different denominator than total new-car EV share. Production/factory exports in Germany are not the same as foreign-trade values or the domestic EV market.

### Data and reporting periods

| Series | Reporting period | Publisher |
| --- | --- | --- |
| International EV adoption, sales, fully electric mix, fleet penetration | 2020–2025, full annual observations | [Our World in Data, based on IEA Global EV Outlook 2026](https://ourworldindata.org/electric-car-sales) |
| Germany new passenger-car registrations and selected brands | January–September 2026 initial snapshot | [KBA](https://www.kba.de/DE/Statistik/Fahrzeuge/Neuzulassungen/MonatlicheNeuzulassungen/monatl_neuzulassungen_node.html) |
| Germany production, factory exports, latest-month BEV/PHEV | September 2026 initial snapshot | [VDA](https://www.vda.de/de/aktuelles/zahlen-und-daten/monatszahlen) |

The dashboard **does not compare Germany's partial 2026 year with other countries' full 2025 years**. Source years are visibly separated. Other countries' overall vehicle production, imports, exports and registrations have not yet been harmonized.

### Architecture

```text
Official source tables (IEA/OWID; KBA/VDA)
       |
       v
Python ingestion + parsing + validation
       |
       v
Validated JSON snapshots + SQL analytics (DuckDB / SQLite)
       |
       +--> 11+ pytest validations --> GitHub monthly Actions
       |
       v
Static dashboard (HTML, CSS, vanilla JS, SVG)
       |
       v
Vercel public site loads latest validated JSON from this repo
```

This is an intentionally lightweight static architecture: no API subscription, managed database, or always-on backend.

### Run locally

```bash
git clone https://github.com/hjsarker/auto-pulse.git
cd auto-pulse
python3 -m http.server 8000 --directory web
# visit http://localhost:8000
```

To test the data pipeline:

```bash
python3 -m pip install -r requirements.txt
python3 -m pytest -q
python3 -m pipeline.global_ev
python3 -m pipeline.main --offline
python3 build_standalone.py
```

Refresh public international datasets manually with `python3 -m pipeline.global_ev --refresh`. To check the German source, use `python3 -m pipeline.main --check-only`. Data is only replaced after validation. The dashboard source files live under `web/`. The self-contained distributable is `deploy-index.html`.

### Monthly refresh — no paid GitHub plan required

The scheduled [GitHub Actions workflow](.github/workflows/monthly-refresh.yml) runs **on the first day of each month at 08:17 Europe/Berlin**. It checks sources, validates new data, regenerates analytics and the snapshot, and commits updated verified files. It also records source-check outcomes. The dashboard reads the current JSON from the independent repository when visitors open the site and falls back to the last embedded validated snapshot if the remote files are unavailable. Source publication delays mean a run on the first may legitimately find no new month.

**Known limitation as of October 2026:** automated requests to VDA returned HTTP 403, so the Germany update step cannot yet be guaranteed to succeed. Country-level IEA/OWID refresh was tested successfully. **German brand data is not yet refreshed automatically**. A blocked source leaves the last verified period in place; the workflow flags the failure. This limitation must be resolved before claiming the German monthly feed is fully automated.

GitHub Actions schedules may be delayed and can be disabled in inactive public repositories; check the [Actions tab](https://github.com/hjsarker/auto-pulse/actions) periodically. Code changes still require a new web deployment; **validated data changes** are read directly from GitHub.

### Quality and reproducibility

The test suite checks source shape, percentage bounds, required country coverage, reconciled monthly German totals, date ordering, electric-powertrain totals and year transitions. SQL outputs include year-over-year change, ranking, rolling registration averages and lag functions. Source HTML changes can break ingestion, so failed downloads and failed validation must not be silently treated as successful updates.

See [methodology](docs/methodology.md) for precise scope and provenance. The original publishers retain their dataset licences and attribution requirements. Auto Pulse is an independent project and not affiliated with those publishers.

### Licence

Application source code: MIT (see [LICENSE](LICENSE)). External source data: subject to each publisher's terms.
