# Semantic Link — Model Health & Security Suite

> One notebook. 13 tools. Zero guesswork.

A comprehensive diagnostic, remediation, and security auditing toolkit for Microsoft Fabric semantic models, built as a single self-contained Fabric notebook with an interactive ipywidgets console.

**Submission for the 2026 Microsoft Fabric Semantic Link Developer Experience Challenge.**

---

![Dashboard overview](Screenshots/1%20-%20Model%20Health%20Suite.png)

---

## 🎯 What It Does

Scans any Microsoft Fabric semantic model for:

- **Copilot Readiness** — descriptions, synonyms, hidden keys, format strings, AI Prep configuration
- **Lineage** — measure → source columns, blast radius, unused columns, dead code detection
- **Test Framework** — every DAX measure evaluated, relationships validated
- **Model Diff** — compare against another model (dev vs prod drift detection)
- **Report Visuals** — broken references, unused measures (filtered to reports using this model)
- **Security Audit** — role compliance, RLS/OLS coverage, member assignments
- **Security X-Ray** — the headline feature. Effective access map with AAD group expansion and workspace privilege bypass detection
- **Data Quality** — referential integrity, null rate analysis
- **DAX Dependencies** — fan-in/fan-out analysis, circular references
- **sempy_labs integrations** — BPA, Vertipaq, Direct Lake, Capacity (opt-in)

Then **auto-fixes** 9 categories with one click: descriptions (AI or template), synonyms, hidden keys, format strings, summarize-by, model description, AI instructions, schema reduction (via LinguisticMetadata), and role descriptions.

---

## 💡 Problems We Solve

![Problems 1](Screenshots/2%20-%20What%20Problems%20Does%20This%20Solve.png)

![Problems 2](Screenshots/3%20-%20What%20Problems%20Does%20This%20Solve%20cont.png)

---

## 🔒 The Killer Feature — Security X-Ray

**The problem nobody else solves:** In enterprise Power BI, security is managed through Azure AD groups, not individual users. An AD group like `pbi-sales-emea` gets assigned to an RLS role, and that group might contain 200 users, nested sub-groups, and service principals. Today there is **no tool in Power BI, no REST API, no sempy function, and no third-party solution** that answers:

> *"Given all the RLS filters, OLS restrictions, AD group memberships, and workspace roles — what does each real human actually see when they open this model?"*

The Security X-Ray combines:
- **TOM** → RLS filters + OLS restrictions
- **DAX `INFO.ROLES()`, `INFO.ROLEMEMBERSHIPS()`** → role members (scale-out safe)
- **Microsoft Graph `/groups/{id}/transitiveMembers`** → AAD group expansion to real users (including nested groups)
- **Fabric REST `v1/workspaces/{id}/roleAssignments`** → workspace role priority

...into a single **Effective Access Map** that flags the critical blind spot: **users who bypass RLS via workspace Admin/Member/Contributor privileges**. This is a silent security risk that standard Power BI UI never surfaces.

---

## 🏗️ Architecture & Quick Start

![Architecture](Screenshots/4%20-%20Architecture%20%26%20Quick%20Start.png)

```
┌────────────────────────────────────────────────────────────┐
│               Cell 3: Interactive Console                  │
│  (ipywidgets: workspace/model picker, SPN, AI context)    │
└────────────────┬───────────────────────────────────────────┘
                 │ Run Scan button
                 ▼
┌────────────────────────────────────────────────────────────┐
│      Cell 2: Scan Engine (13 tools)                        │
│                                                             │
│  ├─ TOM (sempy_labs.tom)         RLS/OLS/Roles/Measures   │
│  ├─ DAX (sempy.fabric)           INFO functions, metrics  │
│  ├─ Fabric REST (v1/*)           Workspace roles, models  │
│  ├─ Graph API (/v1.0/groups)     AAD group expansion      │
│  ├─ Azure OpenAI (Fabric auth)   AI descriptions/synonyms │
│  └─ sempy_labs (BPA, Vertipaq)   Optional rule-based scan │
│                                                             │
└────────────────┬───────────────────────────────────────────┘
                 │ _LAST_SCAN dict + _LAST_SCAN_LOG buffer
                 ▼
┌────────────────────────────────────────────────────────────┐
│ Cell 4: Scan Log                                           │
│  (prints _LAST_SCAN_LOG — stdout captured via             │
│   contextlib.redirect_stdout so Cell 3 form stays visible)│
└────────────────┬───────────────────────────────────────────┘
                 │
                 ▼
┌────────────────────────────────────────────────────────────┐
│ Cell 5: View Report                                        │
│  (chunked widgets.HTML in VBox — bypasses 100KB limit)    │
└────────────────┬───────────────────────────────────────────┘
                 │
                 ▼
┌────────────────────────────────────────────────────────────┐
│ Cell 6: Apply & Verify                                     │
│  (TOM write via sempy_labs.tom.connect_semantic_model)    │
└────────────────────────────────────────────────────────────┘
```

---

## 📦 What's in the Repo

```
├── 2026_SemanticLink_NatarajanManivasagan_ModelHealthAndSecuritySuite.ipynb  (the submission)
├── sample_report.html                                                          (a one-shot rendered dashboard)
├── README.md
├── LICENSE                                                                     (MIT)
└── Screenshots/                                                                (37 images)
```

---

## 🚀 Quick Start

### Prerequisites

![Prerequisites](Screenshots/5%20-%20Required%20Permission%20%26%20Requirements.png)

| Requirement | Detail |
|---|---|
| **Runtime** | Microsoft Fabric notebook, Runtime 1.3+ (Spark 3.5) |
| **Capacity** | F2+ or P1+ (AI features need Copilot tenant switch ON) |
| **Semantic model** | Any Import / Direct Lake / DirectQuery model |
| **Service Principal** (recommended) | workspace Viewer+ role, `Directory.Read.All` in Azure AD, Power BI API access |

### Import into Fabric

1. Clone this repo or download the `.ipynb` file
2. Open your Fabric workspace → **Import** → **Notebook** → From computer
3. Upload `2026_SemanticLink_NatarajanManivasagan_ModelHealthAndSecuritySuite.ipynb`
4. Attach to any lakehouse in the workspace (optional — HTML report saves there)

### Don't have a semantic model to test?

Use Microsoft's sample **Contoso** or **Wide World Importers** model:
- Power BI Service → **Learn** → Samples → Pick any sample
- OR deploy via [Microsoft's sample semantic model templates](https://learn.microsoft.com/en-us/power-bi/create-reports/sample-datasets)
- The notebook works on ANY semantic model — no customization needed

### Graceful degradation

The notebook handles missing capabilities gracefully:

| Missing | What still works |
|---------|------------------|
| No Service Principal | Scan runs, but Security X-Ray skips AAD group expansion (shows raw role members only) |
| No Fabric Copilot (F2+) | AI descriptions fall back to rule-based template descriptions |
| No `Directory.Read.All` on SPN | Security X-Ray shows `obj:GUID` instead of resolved AD group names |
| No workspace Admin | Scan is fully read-only, just can't apply fixes via Cell 6 |

### Run

```
Cell 1 → Cell 2 → Cell 3 → [Click Run Scan] → Cell 4 → Cell 5 → Cell 6
```

| Cell | What it does |
|---|---|
| **1** | `%pip install` dependencies (kernel restarts once) |
| **2** | Loads the 13-tool engine + AI client (~5s) |
| **3** | Interactive ipywidgets console — select model, configure SPN, click Run Scan. Form stays visible; scan log is captured for Cell 4 |
| **4** | **Scan Log** — prints the full verbose scan output (tool-by-tool progress, AI calls, Graph expansion) |
| **5** | Renders the full HTML dashboard (scores, findings, fix plan, Security X-Ray tabs) |
| **6** | Review fix plan, edit `SELECTED_FIXES`, apply fixes to the model |

### Demo Mode — safe screenshots for public sharing

Enable the **"Mask PII (Demo Mode)"** checkbox in Cell 3 before running a scan. All user emails, display names, and AD group names are replaced with sequential pseudonyms:

| Original | Masked |
|----------|--------|
| `natarajan.manivasagan@company.com` | `user1@example.com` |
| `Natarajan Manivasagan` | `User 1` |
| `pbi-sales-emea` | `Group A` |

Same person/group always maps to the same pseudonym — so relationships stay visible in screenshots without exposing real identities.

---

## 🔧 Troubleshooting

![Troubleshooting](Screenshots/5%20-%20Troubleshooting.png)

**Common issues:**

- **Graph API 403** — SPN missing `Directory.Read.All` in Azure AD → App Registrations → API permissions
- **AI unavailable** — needs Fabric F2+ capacity with Copilot tenant switch ON; falls back to template descriptions automatically
- **Workspace not found** — SPN needs Viewer/Contributor on the target workspace
- **Security Audit score = 0%** — usually means no roles defined or TOM read failed
- **Cell 5 shows nothing** — run Cell 3 with Run Scan first

Full troubleshooting table lives in Cell 0 of the notebook.

---

## ⚡ Performance

![Performance](Screenshots/6%20-%20Performance.png)

| Model Size | Tables | Measures | Scan Time |
|-----------|--------|----------|-----------|
| Small | 6 | 30 | ~30s |
| Medium | 20 | 100 | ~60s |
| Large | 50+ | 200+ | ~90-120s |

**Performance optimizations applied:**
- Data Quality: batched DAX UNION query (1 call for N columns vs N calls)
- Security X-Ray: SPN-based Graph token via MSAL (0.3s vs 60s+ `notebookutils` timeout)
- Report Visuals: filters to only reports connected to this model
- AI descriptions: parallelized via Azure OpenAI Fabric Copilot auth
- HTML rendering: chunked into 50KB blocks for Fabric's widget pipeline

---

## 📸 Cell-by-Cell Walkthrough

### Cell 1 — Setup (install dependencies)

![Cell 1 Setup](Screenshots/7%20-%20Setup%201%20Cell.png)

### Cell 2 — Scan Engine (13 tools load)

![Cell 2 Engine](Screenshots/8%20-%20Scan%20Engine%202%20Cell.png)

![Cell 2 Output](Screenshots/8%20-%20Scan%20Engine%20OP%202%20Cell.png)

### Cell 3 — Interactive Console

![Cell 3 Config](Screenshots/9%20-%20Config%203%20Cell.png)

The live form after launch — workspace / model / SPN / tool selection / AI context:

![Cell 3 Live](Screenshots/10%20-%20Config%20OP%203%20Cell.png)

After clicking Run Scan, the form stays visible; scan progress is captured to the log buffer:

![Cell 3 Progress](Screenshots/10%20-%20Config%20OP%20Log%203%20Cell.png)

### Cell 4 — Scan Log

![Cell 4 Log](Screenshots/11%20-%20Scan%20Log%204%20Cell.png)

Full verbose output (tool-by-tool progress, Graph API calls, AI generation, SPN auth):

![Cell 4 Output](Screenshots/11%20-%20Scan%20Log%20OP%204%20Cell.png)

### Cell 5 — View Report

![Cell 5 Intro](Screenshots/12%20-%20View%20Report%205%20Cell.png)

The interactive HTML dashboard. Scroll through to see Score Cards, Detailed Findings, Fix Plan, Security Audit, Security X-Ray, and Export tabs:

<details>
<summary><b>Full dashboard walkthrough — 18 screenshots</b> (click to expand)</summary>

![Dashboard 1](Screenshots/13%20-%20View%20Report%20OP%201-%205%20Cell.png)
![Dashboard 2](Screenshots/14%20-%20View%20Report%20OP%202-%205%20Cell.png)
![Dashboard 3](Screenshots/15%20-%20View%20Report%20OP%203-%205%20Cell.png)
![Dashboard 4](Screenshots/16%20-%20View%20Report%20OP%204-%205%20Cell.png)
![Dashboard 5](Screenshots/17%20-%20View%20Report%20OP%205-%205%20Cell.png)
![Dashboard 6](Screenshots/18%20-%20View%20Report%20OP%206-%205%20Cell.png)
![Dashboard 7](Screenshots/19%20-%20View%20Report%20OP%207-%205%20Cell.png)
![Dashboard 8](Screenshots/20%20-%20View%20Report%20OP%208-%205%20Cell.png)
![Dashboard 9](Screenshots/21%20-%20View%20Report%20OP%209-%205%20Cell.png)
![Dashboard 10](Screenshots/22%20-%20View%20Report%20OP%2010-%205%20Cell.png)
![Dashboard 11](Screenshots/23%20-%20View%20Report%20OP%2011-%205%20Cell.png)
![Dashboard 12](Screenshots/24%20-%20View%20Report%20OP%2012-%205%20Cell.png)
![Dashboard 13](Screenshots/25%20-%20View%20Report%20OP%2013-%205%20Cell.png)
![Dashboard 14](Screenshots/26%20-%20View%20Report%20OP%2014-%205%20Cell.png)
![Dashboard 15](Screenshots/27%20-%20View%20Report%20OP%2015-%205%20Cell.png)
![Dashboard 16](Screenshots/28%20-%20View%20Report%20OP%2016-%205%20Cell.png)
![Dashboard 17](Screenshots/29%20-%20View%20Report%20OP%2017-%205%20Cell.png)
![Dashboard 18](Screenshots/30%20-%20View%20Report%20OP%2018-%205%20Cell.png)

</details>

### Cell 6 — Apply & Verify

![Cell 6 Intro](Screenshots/31%20-%20Apply%20%26%20Verify%206%20Cell.png)

Preview all auto-generated fixes by category:

![Cell 6 Fix Plan](Screenshots/32%20-%20Apply%20%26%20Verify%20OP%201-%206%20Cell.png)

Edit `SELECTED_FIXES`, re-run to apply — score deltas render as a colored HTML table showing before/after:

![Cell 6 Results](Screenshots/33%20-%20Apply%20%26%20Verify%20OP%202-%206%20Cell.png)

---

## 📄 Sample Output

A full rendered HTML dashboard from a real run is included: [`sample_report.html`](sample_report.html)

Download and open in your browser to see the complete output without running the notebook.

---

## 🛠️ Technical Highlights

- **Chunked HTML rendering** — overcomes Fabric's ~100KB `displayHTML` limit by splitting the dashboard into `widgets.HTML` chunks inside a VBox
- **Scan log capture** — `contextlib.redirect_stdout` sends scan output to a buffer so the Cell 3 form stays visible; Cell 4 replays the log on demand
- **SPN-based Graph token** — via MSAL, bypasses the unreliable `notebookutils.credentials.getToken("https://graph.microsoft.com")` that can hang 60+ seconds
- **AAD group expansion** — `/groups/{id}/transitiveMembers` with pagination, works with nested groups
- **Admin bypass detection** — cross-references workspace role priority (Admin > Member > Contributor > Viewer) against RLS role assignments
- **Batched DAX queries** — null-rate checks run as one UNION query (10x faster than per-column)
- **Azure OpenAI via Fabric auth** — `synapse.ml.fabric.credentials.get_openai_httpx_sync_client()` + `openai.AzureOpenAI()`, no API key needed
- **TOM write for LinguisticMetadata** — enables Copilot `CustomInstructions` and schema simplification (`isAvailableInMDX`)
- **PII masking (Demo Mode)** — sequential pseudonyms with consistent mapping across tables so relationships stay visible in screenshots

---

## 📝 License

MIT — see [LICENSE](LICENSE)

---

## 👤 Author

**Natarajan Manivasagan** — Fabric / Power BI developer at Tiger Analytics
Built for the **Microsoft Fabric Community Contest 2026 — Semantic Link Developer Experience Challenge**.

- LinkedIn: [www.linkedin.com/in/natarajan-manivasagan](https://www.linkedin.com/in/natarajan-manivasagan)
- Power BI / Fabric Community: [community.fabric.microsoft.com — user 1345926](https://community.fabric.microsoft.com/t5/user/viewprofilepage/user-id/1345926)
