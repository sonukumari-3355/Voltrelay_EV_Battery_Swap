# ⚡ VoltRelay Energy: EV Battery-Swapping Network Operations & Unit Economics Analysis

![Python](https://img.shields.io/badge/Python-3.10+-blue.svg)
![Pandas](https://img.shields.io/badge/Pandas-Data%20Analysis-150458.svg)
![Matplotlib](https://img.shields.io/badge/Matplotlib%20%26%20Seaborn-Visualization-orange.svg)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen.svg)

## 📌 Project Overview
**VoltRelay Energy** operates an unmanned Battery-as-a-Service (BaaS) swapping network for electric two-wheelers (2W) and three-wheelers (3W) across **six Indian cities** (Bengaluru, Delhi NCR, Hyderabad, Pune, Mumbai, and Jaipur). 

Over an **18-month operational period (January 2024 – June 2025)** encompassing **3.88 million swap attempts**, completed swaps nearly tripled. However, rapid top-line growth coincided with rising service failures, declining new-rider retention, and eroding per-swap profitability. This project performs an end-to-end relational data audit, unit-economics modeling, and cohort root-cause investigation to guide VoltRelay's next operating budget allocation.

---

## 🗂️ Dataset Architecture (8 Relational Tables)
| Table | Records | Description |
| :--- | :---: | :--- |
| `swap_events.csv.gz` | 3,877,013 | Granular attempt-level logs (completed, stockout failures, abandoned queues, pricing, energy drawn) |
| `station_hourly_status.csv.gz` | 1,487,712 | Hourly station telemetry (charged inventory, charger availability, cabinet temperatures, grid outages) |
| `support_tickets.csv` | 44,000 | Customer complaints across app chat, call center, WhatsApp, and partner escalations (English & Hinglish) |
| `riders.csv` | 20,000| Rider demographics, vehicle class (`2W`/`3W`), plan type, signup cohorts, and fleet partner links |
| `batteries.csv` | 6,500 | Battery asset ledger: supplier (`Cellora`, `Amptek`, `Kyron`), manufacturing lot, SOH%, and retirement dates |
| `city_daily_context.csv` | 3,282 | Daily temperature, rainfall, grid outage hours, holidays, and competitor promotion flags |
| `stations.csv` | 152 | Station metadata: city, zone, host type, charger generation (`Gen1`/`Gen2`/`Gen3`), rent, and grid tariffs |
| `fleet_partners.csv` | 12 | B2B enterprise contract terms, volume discounts, amendment dates, and peak surcharge rules |

---

## 🧹 Data Quality Audit & Cleaning Pipeline
Before running analytical queries, eight planted data quality issues were identified and resolved:
1. **Internal Test Stations Excluded:** Removed 2 inactive test stations (`STN-TST-01`, `STN-TST-02`) and their `62,031` zero-value test rows.
2. **Firmware Timezone Bug Fixed:** Corrected **`139,490` timestamps** logged 5 hours 30 minutes early on stations running firmware `v3.2.0` between March 10 and April 14, 2025.
3. **Offline Sync Deduplication:** Removed **`987` duplicate retry records** generated within ≤180 seconds at poor-connectivity stations under `sync_mode = offline_batch`.
4. **Geographic Standardization:** Normalized inconsistent city spellings in `riders.csv` (`BLR`, `Bangalore`, `Gurgaon`, `New Delhi`, `Bombay`, etc.) into the 6 canonical cities.
5. **Sensor & Odometer Outlier Handling:** Clipped battery SOC/SOH readings at `100%` and filtered negative or >250 km odometer reset outliers into `clean_km`.
6. **Funnel vs. Revenue Separation:** Evaluated failure rates across all `3,813,995` clean attempts while computing unit economics strictly on `swap_completed` events.
7. **Missing Telemetry Handling:** Excluded `missing` telemetry rows in hourly station logs rather than zero-imputing.
8. **Support Ticket Text Classification:** Applied English/Hinglish keyword mapping on `rider_comment` to uncover battery and stockout complaints misfiled under `"other"`.

---

## 🔍 Key Analytical Findings

### 1. Volume Growth vs. Seasonal Failure Spikes
* Completed swaps grew from **103,488/month** (Jan 2024) to **303,650/month** (May 2025).
* Baseline failure rates hold near **4.3%**, but surge sharply during summer heat months—peaking at **12.39% (May 2024)** and **9.97% (May 2025)**.
* While the July 2024 base price hike lifted margins, average revenue per swap eroded by **₹4.90** (`₹63.36 → ₹58.46`) between October 2024 and May 2025 due to rising discounted B2B volume
### 2. Service Failures Concentrate in 3W Fleets, Northern Cities & Gen1 Hardware
* **Vehicle Class:** 3W riders fail at nearly **2x the rate of 2W riders** (**10.39%** vs. **5.45%**), driven by stockouts (`6.42%`) and queue abandonment (`3.23%`).
* **Geography:** Failures are highest in **Jaipur (7.85%)**, **Delhi NCR (7.35%)**, and **Hyderabad (6.69%)**. Meanwhile, **Mumbai (4.45% failure)** suffers the lowest gross margin (**₹22.94/swap**) due to high real estate rent and electricity tariffs.
* **Charger Hardware:** Legacy **Gen1 stations (51 sites)** suffer a **7.23% failure rate**, whereas **Gen2 (4.85%)** and **Gen3 (4.84%)** cut failures by **33%**.

### 3. Premature Battery Failure (`Kyron` 2W Defect)
* **100% of Kyron 2W packs (1,385 / 1,385)** retired prematurely at **60.53% SOH** after delivering only **401.55 swaps** on average—compared to **643.0 swaps** for Cellora and Amptek (`0` retired).
* This increased battery depreciation from **₹49.82/swap** to **₹79.85/swap (+60%)**.
* When 2W SOH drops below **70%**, average delivered range plunges **33.5% (`65.01 km → 43.21 km`)**, forcing riders to swap 50% more often.

### 4. The Volume–Margin Paradox (`ZipDrop` Contract Renegotiation)
* Largest partner **ZipDrop (`FP-03`)** drives **549,161 swaps** (#1 in volume) but delivers the **lowest gross margin on the network (₹20.08/swap before battery wear)**—over 50% lower than **FeastFly (`FP-01`, ₹30.37/swap)**.
* Following ZipDrop's **November 1, 2024 contract amendment** (`12% → 28%` discount + peak surcharge waiver), average billed revenue fell from **₹57.48 to ₹48.64**, leaving gross contribution margin at **₹19.26/swap** (cash-negative after battery wear).

### 5. Root Causes of New-Rider Churn
* **Primary Driver:** New riders who experience **1+ failure in their first 5 attempts** have a 30-day retention rate of **88.34%** vs. **91.40%** for failure-free riders (**35.6% higher early churn**). City retention mirrors reliability, lowest in **Jaipur (88.88%)** and **Delhi NCR (89.87%)**.
* **Secondary Contributors:** Churned riders received lower-SOH packs in their first 5 visits (**90.77% vs. 92.55%**), and **5,175 of 44,000 support tickets** remain unresolved.

---

## 🎯 Strategic Budget Recommendations

| Priority | Action Item | Expected Business Impact |
| :---: | :--- | :--- |
| **1. Approve** | **Retrofit 51 `Gen1` stations to `Gen2/Gen3` + expand 3W inventory** in Jaipur, Delhi NCR, and Hyderabad | Cuts network failure rate by ~33% at existing sites without adding new rent overhead |
| **2. Approve** | **Phase out Kyron 2W battery lots** (`KY-2407–2409`), shift procurement to Cellora/Amptek, and enforce **75% SOH retirement** | Recovers **₹1.5+ Crore/year** in excess asset wear and restores 2W rider range |
| **3. Approve** | **Roll out Peak/Off-Peak pricing network-wide** (`₹53.40/swap` peak margin) and **renegotiate ZipDrop's discount cap to 16–18%** | Lifts net per-swap contribution margin by **+₹10–₹14/swap** |
| **4. Reject** | **Reject blanket new-station expansion and reject ZipDrop exclusivity** under current 28% discount terms | Prevents scaling margin-negative unit economics |


---

## 🛠️ Tech Stack & How to Run
* **Language & Libraries:** Python 3, Pandas, NumPy, Matplotlib, Seaborn
* **Environment:** Google Colab / Jupyter Notebook
1. Clone this repository:
   ```bash
   git clone [https://github.com/](https://github.com/)<your-username>/voltreplay-battery-swapping-analytics.git
