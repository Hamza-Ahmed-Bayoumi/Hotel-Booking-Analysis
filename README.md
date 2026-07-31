# 🏨 Hotel Booking Data Analysis

An end-to-end data analytics project that cleans, explores, and engineers features from a real hotel-bookings dataset, then translates the findings into an interactive Power BI dashboard and a set of quantified business recommendations.

---

## 1. 📋 Project Overview

Online travel agencies now generate the majority of hotel bookings, but a growing share of those bookings never arrive. This project analyzes hotel reservation data from a **City Hotel** and a **Resort Hotel** to understand booking behavior, cancellations, customer segments, and revenue.

**Main objective:** turn raw booking records into clear, evidence-based answers to core business questions — which guests and channels cancel the most, how much revenue is genuinely at risk, which bookings should be flagged, and how those findings can be delivered as a self-serve BI dashboard.

---

## 2. 📊 Dataset Overview

| Item | Detail |
|---|---|
| Raw size | 119,390 rows × 32 columns |
| Coverage | City Hotel & Resort Hotel, Portugal — arrivals July 2015 to August 2017 |

**Main data categories:**
- **Booking details** — hotel, lead time, arrival date, market segment, distribution channel, deposit type
- **Stay details** — nights (week/weekend), reserved vs. assigned room type, meal plan
- **Guest profile** — adults, children, babies, country, repeat-guest flag, customer type
- **Commercial fields** — ADR, special requests, booking changes, agent/company, reservation status

**Key columns:**

| Column | Meaning |
|---|---|
| `is_canceled` | Target variable — 1 if the booking was cancelled/no-show, 0 if the guest stayed |
| `lead_time` | Days between booking and arrival |
| `adr` | Average Daily Rate — the room's price per night |
| `market_segment` / `distribution_channel` | How and through whom the booking was made |
| `deposit_type` | Whether a deposit was required (No Deposit / Non Refund / Refundable) |
| `customer_type` | Transient, Contract, Group, or Transient-Party |
| `agent` / `company` | Booking agent / corporate account, where applicable |

---

## 3. 🧹 Data Cleaning

- **Duplicates:** 31,994 exact duplicate rows (26.8% of the raw file) were identified and removed. Left in place, they inflated the cancellation rate to 37.0% instead of the true 27.5% — the single biggest correction in the project.
- **Missing values** were handled with business-driven rules rather than blind imputation:

| Column | Missing | Logic Used |
|---|---|---|
| `company` | 82,137 | Corporate segment + missing → `"Unknown Company"`; all other segments → `"No Company"` (most segments never carry a company ID by nature) |
| `agent` | 12,193 | Direct segment + Direct channel + missing → `"Direct Booking"`; all other cases → `"Unknown Agent"` |
| `country` | 452 | Filled as `"Unknown"` — no reliable inference available |
| `children` | 4 | Filled with `0` (a count field, so missing plausibly means none) |

- **Invalid/unrealistic records:** a booking with an ADR of 5,400 (a keying error), one negative ADR (floored at 0), 166 bookings with zero total guests, and 651 bookings with zero total nights were identified as non-real stays and corrected or removed (see Outlier Analysis below).

---

## 4. 🔍 Outlier Analysis

Outliers were screened statistically (Tukey IQR fence) but only removed when supported by business context — not automatically.

| Column | Unusual Values Found | Treatment |
|---|---|---|
| `adr` | 1 record at 5,400 (10x the next-highest rate) | **Removed** — data entry error |
| `adr` | 1 record at -6.38 | **Corrected** — floored at 0 |
| `adr` | 1,778 records at exactly 0 | **Kept** — 91% are Complementary segment (real comp/staff stays) |
| `adults` | 13 records with 10–55 adults | **Kept + flagged** — genuine tour-operator group blocks |
| `adults`/`children`/`babies` | 166 records with zero total guests | **Removed** — no occupancy, no revenue |
| `children`/`babies` | 3 records with 9–10 infants/children | **Capped** to realistic maximums |
| `stays_in_*_nights` | 651 zero-night bookings | **Removed** — administrative artifacts |
| `stays_in_*_nights` | 23 stays over 30 nights | **Kept** — genuine discounted long-stay contracts |

**Result:** only 758 rows (0.87%) were removed, all provable non-stays. ADR skewness dropped from **10.92 to 0.93** without applying any mathematical transformation.

---

## 5. 🔎 Exploratory Data Analysis (EDA)

- **Numerical analysis:** distribution profiling of `lead_time`, `adr`, and stay-length variables, with skewness/kurtosis used to guide cleaning decisions.
- **Categorical analysis:** composition of `hotel`, `market_segment`, `distribution_channel`, `customer_type`, `deposit_type`, and `meal`.
- **Statistical & correlation analysis:** Pearson and Spearman correlation of numeric fields against `is_canceled`, showing that the strongest real drivers are categorical, not linear-numeric.

**Main patterns and insights:**
- **Lead time** is the strongest continuous driver — cancellation rises from 6.0% (same-day) to 41.0% (over a year out).
- **Non-refundable deposits** show a 94.7% cancellation rate — not because deposits cause cancellations, but because they are applied to bookings already judged risky.
- **Special requests** are the cleanest protective signal — cancellation falls steadily from 33.5% (0 requests) to 5.6% (5 requests).
- **Guest origin × channel** interaction is the hidden pattern: domestic (Portuguese) Groups and Offline TA/TO bookings cancel dramatically more (up to 89x) than the identical channel sold internationally.
- **`reservation_status`, `required_car_parking_spaces`, and `room_changed`** all leak the outcome (known only after arrival) and must never be used as predictors.

---

## 6. 🛠️ Feature Engineering

35 new features were engineered to support deeper analysis and visualization, including:

- **Stay & value:** `total_nights`, `total_guests`, `adr_per_guest`, `total_revenue`, `guest_composition` (Solo/Couple/Family/etc.)
- **Temporal:** `arrival_date`, `booking_season`, `is_high_season`, `is_weekend_arrival`
- **Behavioral & commercial:** `previous_cancel_ratio`, `is_repeat_customer`, `booking_source_group` (collapses market segment × channel into 6 managed categories)
- **Data-integrity flags:** `is_group_block`, `is_child_only_record`, `is_zero_rated`
- **Composite `cancellation_risk_score`:** a leakage-free, weighted score built from booking-time signals, grouping bookings into **Low / Moderate / High / Very High** risk tiers with an **18.2x** cancellation-rate lift between tiers

These features power the segment comparisons, trend charts, and risk-tier views used throughout both the notebook and the Power BI dashboard.

---

## 7. 📈 Power BI Dashboard

A 4-page interactive report, filterable by **hotel, month, customer type, and country**.

| Page | Main KPIs & Visuals | Key Insight |
|---|---|---|
| **Customer Analysis** | Market segment, meal preference, distribution channel, repeat-guest rate (3.9%) | Guest base is dominated by Transient (independent) travelers; repeat guests are a small share |
| **Cancellation Analysis** | Cancellation rate by lead time, deposit type, and market segment, tracked month by month | Cancellation risk climbs sharply with lead time and varies enormously by channel |
| **Revenue & Room Analysis** | ADR trends over time, revenue by market segment, room-type performance by hotel | Online OTA generates the most revenue but is also the least reliable channel |


An **Executive Overview** page also anchors the report with headline KPIs: total bookings, overall cancellation rate, total revenue, average ADR, and average stay length.

---

## 8. 💡 Key Business Insights

- **€11.24M** of booked revenue is genuinely exposed to cancellation, and **83%** of that exposure sits in a single channel — Online OTA.
- **Online TA** is both the largest booking source (59.2%) and the highest-cancelling channel (35.6%).
- **Domestic (Portuguese) Groups and Offline TA/TO bookings** cancel up to 89x more than the same channel sold internationally.
- The two hotels are fundamentally different businesses: the **City Hotel** is a volume-and-occupancy business with steady demand; the **Resort** is a yield-management business with extreme seasonality (ADR ranges from ~50 to ~189 across the year).
- The **Very High** risk tier (15.3% of bookings) carries €6.8M in revenue at a 66% cancellation rate — small enough to manage proactively.

---

## 9. ✅ Business Recommendations

- **Reduce cancellations:** require a partial deposit or reconfirmation for domestic Group / Offline TA/TO bookings, the most targetable high-risk segment.
- **Improve retention:** proactively solicit preferences from long-lead bookers — special requests are shown to cut cancellation risk nearly in half.
- **Optimize booking channels:** shift a portion of Online OTA volume toward Direct bookings, which convert far more reliably at a comparable rate.
- **Revenue & pricing:** move from a flat deposit policy to a **seasonal** one, since peak-revenue months also carry peak cancellation risk.
- **Operations:** actively manage the "Very High" risk tier with reconfirmation calls or planned overbooking, and resolve data-quality issues (e.g., duplicate country codes) at the source system.

---

## 10. 🧰 Tools & Technologies

| Category | Tools |
|---|---|
| Language | Python |
| Data manipulation | Pandas, NumPy |
| Visualization | Matplotlib, Seaborn |
| Development environment | Jupyter Notebook |
| Business Intelligence | Power BI, DAX |
| Version control | GitHub |

---

## 11. 🔄 Project Workflow

```
Data Collection → Data Understanding → Data Cleaning → Outlier Analysis →
EDA → Feature Engineering → Data Visualization → Power BI Dashboard →
Business Insights → Recommendations
```

---

*For the full detailed analysis — including complete decision registers, statistical tests, and code-level documentation — see the project notebook and the accompanying detailed documentation.*
