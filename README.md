# 🛡️ ShieldView

**Client-side vulnerability management dashboard ingesting Tenable/Nessus CSV exports, cross-referencing CISA KEV, with SLA tracking and executive reporting.**

[![Live Demo](https://img.shields.io/badge/demo-live-blue)](https://mrdchiang.github.io/amp-shield/)
[![Version](https://img.shields.io/badge/version-6.2.0-green)](#changelog)
[![Lines](https://img.shields.io/badge/code-6,349%20lines-brightgreen)](#architecture)
[![License](https://img.shields.io/badge/license-MIT-lightgrey)](#license)

---

## About

ShieldView is a single-file HTML vulnerability management dashboard that transforms raw Tenable/Nessus CSV exports into an actionable, filterable, cross-referenced view of your security posture. Built as the first stage in a 4-tool remediation pipeline, it ingests scanner data, enriches findings with CISA's Known Exploited Vulnerabilities catalog, tracks remediation SLAs, detects end-of-life software, manages risk acceptances, and produces executive-grade reports — all in the browser with zero server dependencies.

**Live demo:** [https://mrdchiang.github.io/amp-shield/](https://mrdchiang.github.io/amp-shield/)

---

## Key Features

- **CSV Import** — Drag-and-drop Tenable/Nessus vulnerability exports plus SCCM/PDQ device inventory CSVs. Automatic column mapping, duplicate handling, and per-asset enrichment.
- **CISA KEV Live Feed** — Auto-fetches the CISA Known Exploited Vulnerabilities JSON catalog, cross-references against imported findings, and displays KEV badges with BOD 22-01 due dates, ransomware flags, and required actions. Graceful fallback to cached/hardcoded data when offline.
- **CVE / Plugin Toggle** — Switch between CVE-centric and scanner-plugin-centric views. Group findings by CVE to see per-asset status tracking, or by plugin to see the raw scanner output grouped by check name.
- **EOL Detection Engine** — Built-in database of 99 operating system and software entries (Windows, macOS, Linux distros, SQL Server, Exchange, Kubernetes, VMware, PAN-OS, Java, PostgreSQL, and more). Fuzzy-matches against asset OS strings and server roles. Dashboard summary card highlights EOL and approaching-EOL assets.
- **Executive Dashboard** — Standalone executive view with trend sparklines from snapshot history, team breakdown cards with severity distribution bars, top-5 CVE table, auto-generated risk summary paragraph, letter-grade scoring (A–F) factoring KEV response, resolution rate, and SLA compliance.
- **SLA / Due-Date Engine** — Configurable severity-based remediation deadlines (Critical: 15 days, High: 30 days, Medium: 90 days, Low: 180 days). CISA KEV findings use federal BOD 22-01 deadlines. Overdue tracking with dashboard stat card, "Days to Due" column with red/amber/green color coding, overdue filter, and SLA breach penalties rolled into the executive grade.
- **Risk Acceptances with Auto-Expiry** — Structured exception records with justification, accepted-by, expiration date, and compensating controls. Modal form with inline validation (minimum 20-character justification). Auto-expiry engine reverts expired exceptions to Active with full audit trail entries. Dedicated Exceptions page sorted by expiry urgency with color coding (<30 days red, <60 days amber).
- **Import Diffing with Snapshots** — Every CSV re-import captures a snapshot of the previous findings set and computes a diff (new, resolved, unchanged). Snapshot history powers the dashboard trend sparkline showing finding counts over time.
- **Snipe-IT Integration** — REST API configuration card for connecting to a Snipe-IT asset management instance. Connection test, paginated hardware fetch, asset enrichment (assigned user, purchase date, department, warranty status), and per-asset Snipe-IT detail cards. Demo data loader for offline evaluation — no Snipe-IT instance required.
- **Pipeline Page** — Visual overview of the full 4-tool security pipeline: ShieldView → RemFlow → TheValidator → Launchpad. Shows data flow, record counts, timestamps, and cross-tool handoff status.
- **4-Team Assignment Engine** — Rule-based team assignment from hostname prefixes, OU paths, OS type, and server roles. Routes findings to Endpoint, Factory, Networking, or Servers teams. Each team has its own detail page with stats, top CVEs, EOL status, and member roster.

### Additional Features

- **Dashboard** — Clickable stat cards that apply filters (severity, state, KEV, overdue, team), live search, severity dropdown, and group-by-check toggle.
- **CVE Detail View** — Per-CVE breakdown with description, CVSS, solution guidance, affected assets table with hostname/OS/state/last-seen, and per-asset actions.
- **Solutions View** — Solution/check name grouping with asset counts and severity ranges for planning patch campaigns.
- **Vendor / Family View** — Classification rules that map plugins to vendor families (Microsoft, Linux, Cisco, VMware, etc.) for vendor-level reporting.
- **Recently Fixed** — Timeline of resolved findings driven by `fixedAt` timestamps.
- **Assets Page** — Full asset inventory with OS, OU, assigned user, server roles, warranty status, EOL status, and team assignment. Filters by OS, team, warranty, and EOL status.
- **Teams Page** — 7-team structure (Infrastructure, Platform Engineering, Security Operations, Application Services, Database & Storage, End-User Computing, Endpoint Engineering) with per-team stats, period-over-period deltas, asset lists, and top findings.
- **Admin Page** — SLA policy configuration, KEV feed management, and data controls.
- **Data Sources Page** — CSV upload cards for Tenable findings and SCCM/PDQ assets, Snipe-IT API configuration, and CISA KEV feed status.
- **Dark/Light Theme** — Full light-mode support via CSS custom properties data attribute toggle.
- **Collapsible Sidebar** — Space-efficient navigation with section labels and icons.
- **Mobile Responsive** — Layout adapts down to 320px viewports.
- **Audit Trail** — Every state transition logs timestamp, from-state, to-state, reason, and source (manual/RemFlow/TheValidator).

---

## Architecture

ShieldView is a **single-file HTML application** (~6,400 lines) with no build step, no framework, and no server-side dependencies. Everything runs in the browser.

| Layer | Implementation |
|-------|---------------|
| **UI** | HTML5 with CSS custom properties for theming. All components rendered client-side via vanilla DOM manipulation. |
| **Data** | `localStorage` key-value store with a typed accessor layer. All state is persisted in the browser and survives reloads. |
| **Charts** | Hand-rolled inline SVG — trend sparklines, severity distribution bars, and team breakdown charts. No charting library. |
| **Import** | Client-side CSV parser with configurable column mapping. Tenable and SCCM/PDQ formats supported. |
| **Feeds** | `fetch()` to `cisa.gov` for the KEV JSON catalog, cached in localStorage with auto-refresh. |
| **Contracts** | Shared `js/shared/contract.js` ES module defining the data contract shared across ShieldView, RemFlow, TheValidator, AskClippy, and Launchpad — all subpaths of the same GitHub Pages origin. |

### Same-Origin Shared Contract

All five tools deploy under `mrdchiang.github.io` and share a single `localStorage` namespace. The shared contract (`contract.js`) defines every key, record schema, validator, and accessor — ensuring type safety and consistent state across the toolchain without a backend.

```
ShieldView (find vulns) → RemFlow (remediate) → TheValidator (verify) → Launchpad (dashboard)
```

---

## Getting Started

1. **Open in browser** — Navigate to [https://mrdchiang.github.io/amp-shield/](https://mrdchiang.github.io/amp-shield/) or open `index.html` locally:
   ```bash
   open index.html      # macOS
   start index.html     # Windows
   xdg-open index.html  # Linux
   ```

2. **Go to Data Sources** — Click **⚙️ Data Sources** in the sidebar.

3. **Drop a CSV or load demo data**:
   - Upload a Tenable/Nessus `.csv` export under "Vulnerability Findings"
   - Upload an SCCM/PDQ device inventory `.csv` under "Device Assets"
   - Or click **🖥️ Load Demo Data** on the Snipe-IT card to explore with sample data

4. **Explore** — Navigate through Dashboard, Executive, Assets, Teams, and the Pipeline to see your vulnerability posture.

---

## Technologies

| Technology | Usage |
|-----------|-------|
| **HTML5** | Semantic markup, file API for CSV imports, responsive layout |
| **CSS3** | Custom properties for theming, CSS Grid/Flexbox, dark/light mode |
| **Vanilla JavaScript** | DOM manipulation, CSV parsing, state management, SVG rendering |
| **SVG** | Hand-rolled trend sparklines, severity bars, pie charts |
| **localStorage API** | Client-side persistence for findings, assets, config, snapshots, KEV cache |
| **CISA KEV JSON Feed** | Live `fetch()` to CISA's Known Exploited Vulnerabilities catalog |
| **Snipe-IT REST API** | Optional integration for asset enrichment and warranty tracking |

---

## Related Tools

ShieldView is the first step in a 4-tool security remediation pipeline. All tools share the same data contract via `localStorage` on `mrdchiang.github.io`.

| Tool | Description | URL |
|------|-------------|-----|
| 🛡️ **ShieldView** | Vulnerability management — find and triage | [mrdchiang.github.io/amp-shield](https://mrdchiang.github.io/amp-shield/) |
| 🔧 **RemFlow** | Remediation orchestration — deploy fixes | [mrdchiang.github.io/remflow](https://mrdchiang.github.io/remflow/) |
| ✅ **TheValidator** | Post-remediation verification | [mrdchiang.github.io/thevalidator](https://mrdchiang.github.io/thevalidator/) |
| 🤖 **AskClippy** | AI security assistant | [mrdchiang.github.io/askclippy](https://mrdchiang.github.io/askclippy/) |
| 🚀 **Launchpad** | Unified security tools hub | [mrdchiang.github.io/security-tools](https://mrdchiang.github.io/security-tools/) |

---

## Data Model

ShieldView tracks three core entity types linked through a many-to-many relationship:

| Entity | Key Fields | Description |
|--------|-----------|-------------|
| **Finding** | CVE, check/plugin name, asset hostname, port, severity, state, KEV flag, firstSeen, dueDate, solution | One vulnerability instance on one asset |
| **Asset** | hostname, OS, OU, device user, server roles, team assignment, warranty status | Enriched device inventory |
| **Exception** | justification, acceptedBy, acceptedAt, expiresAt, compensatingControl | Risk acceptance record on a finding |

State transitions flow: **Active → Actioned → Fixed → Verified**, with side paths for **Risk Accepted**, **False Positive**, and **Deferred**. Every transition writes an audit trail entry.

---

## Changelog

| Version | Date | Highlights |
|---------|------|------------|
| **v6.2.0** | 2026-07-24 | Expiring risk acceptances with auto-expiry engine, exception form modal, Exceptions page |
| **v6.1.0** | 2026-07-24 | SLA/due-date engine, severity-based deadlines, KEV BOD 22-01 deadlines, executive grade scoring |
| **v6.0.0** | 2026-07-24 | Live CISA KEV JSON feed integration with caching and fallback |
| **v5.0.0** | 2026-07-24 | Snipe-IT REST API integration, warranty detection, asset enrichment |
| **v4.0.0** | 2026-07-24 | Executive page rebuild, 4-team assignment engine, SVG trend chart, report export |
| **v3.0.0** | 2026-07-24 | End of Life detection engine with 99-entry OS/software database |
| **v2.4.0** | 2026-06-25 | Vendor/family classification view, per-asset plugin name storage |
| **v2.3.1** | 2026-06-18 | Access control audit, team period-over-period deltas |
| **v2.3.0** | 2026-06-10 | Executive dashboard, trend snapshots, recently-fixed view |
| **v2.2.0** | 2026-06-02 | Teams config, precomputed stats tables, background cache warm |
| **v2.1.0** | 2026-05-15 | Streaming CSV importer, asset tags, solutions view, change-request drafts |
| **v2.0.0** | 2026-05-01 | Initial release: scanner importer, KEV feed, dashboard, CVE detail |

---

## License

MIT
