## Persona-Based Scenarios and Workflow

This section outlines how **GigGotYou** operates in real-world situations using **persona-driven scenarios**, demonstrating both successful payouts and edge cases involving system limitations or fraud detection.

---

### Scenario 1: Genuine Disruption → Successful Payout

**Rider:** Ravi, 24. Zepto delivery partner, Bengaluru. Faces heavy rainfall during evening shift.

**What GigGotYou detects:**
1. Rainfall threshold breach confirmed in Ravi’s delivery zone at 6:10 PM.
2. GPS validation shows Ravi was active in assigned delivery area before disruption.
3. Platform activity drops significantly post rainfall (no completed orders in next 2 hours).
4. Disruption window calculated from 6:15 PM to 8:15 PM.
5. Income loss estimated based on Ravi’s average hourly earnings.

**Outcome:**
Ravi’s payout is **automatically processed and credited instantly**.  
Ravi receives: *"A disruption was detected in your area. ₹XXX has been credited as income protection."*

---

### Scenario 2: System Limitation → Missed Trigger (False Negative)

**Rider:** Ravi, 24. Zepto delivery partner, Bengaluru. Faces moderate rainfall during shift.

**What GigGotYou detects:**
1. Rainfall detected but **does not cross predefined threshold values**.
2. GPS validation shows reduced activity, but not fully inactive.
3. Platform activity shows partial order completion.
4. No parametric trigger is activated due to threshold conditions not being met.

**Outcome:**
No payout is triggered despite **partial income loss**.  
Ravi receives: *"No qualifying disruption detected during your active hours."*

**Insight:**
Highlights dependency on **strict parametric thresholds** and need for **adaptive calibration**.

---

### Scenario 3: Fraud Detection → Claim Blocked

**Rider:** Suresh, 34. Blinkit partner, Andheri East. Claims slot disruption due to local flooding.

**What GigGotYou detects:**
1. Rainfall threshold breach confirmed in Andheri East at 2:15 PM.
2. GPS validation shows movement consistent with **active deliveries between 2:00 PM and 4:00 PM**.
3. Platform activity check (mock Blinkit API) shows **3 completed orders during claimed disruption window**.
4. Activity pattern contradicts claimed income loss.
5. Fraud score calculated at **0.87**, exceeding risk threshold.

**Outcome:**
Claim is **automatically flagged and routed for admin review**.  
Suresh’s payout is held.  
Suresh receives: *"Your claim is under review. Our team will contact you within 24 hours."*

**Insight:**
Demonstrates **AI-based fraud detection**, ensuring system integrity and preventing misuse.

---

### Core Workflow Reflection
Across all scenarios, GigGotYou operates on:
→ **Real-time trigger detection**  
→ **Location and activity validation**  
→ **Automated decision-making (approve / reject / review)**  
→ **Instant or conditional payout handling**  

This ensures a balance between **automation, accuracy, and fraud prevention** in income protection.


## 2. Weekly Premium Model
 
### Philosophy
 
Q-Commerce riders operate week-to-week. They don't think in monthly or annual terms — their income resets every week, and so should their insurance. A weekly model also lets riders **opt out during low-risk periods** (e.g., a clear summer week) and opt back in before monsoon season, giving them genuine control.
 
### Premium Formula
 
```
weekly_premium = BASE_RATE × zone_risk_multiplier × seasonal_factor × loyalty_discount
```
 
**BASE_RATE:** ₹30/week (the lowest-risk, lowest-coverage starting point)
 
**Zone Risk Multiplier** (derived from ML model, updated monthly):
 
| Zone Risk Tier | Example Areas | Multiplier |
|---|---|---|
| Low | Gurgaon Cyber City, Pune Kalyani Nagar | 0.85 |
| Medium | Bengaluru Whitefield, Chennai Anna Nagar | 1.0 |
| High | Mumbai Andheri/Kurla, Hyderabad Kondapur | 1.5 |
| Very High | Delhi Dwarka/Yamuna flood zones | 2.2 |
 
**Seasonal Factor** (rule-based, calendar-driven):
- June–September (monsoon): 1.3×
- October–November (Delhi pollution season): 1.2×
- All other months: 1.0×
 
**Loyalty Discount:**
- 0–4 weeks active: 0% discount
- 5–12 weeks active: 5% discount
- 13+ weeks active (no fraud flags): 10% discount
 
**Example Calculation — Arjun, Malad West, July:**
```
₹30 × 1.5 (High Risk) × 1.3 (Monsoon) × 1.0 (New) = ₹58.50/week → rounded to ₹59/week
```
 
**Coverage Cap:** Always set at 20× the weekly premium (e.g., ₹59/week → ₹1,180 max payout/week). This ensures actuarial sustainability.
 
### Payment
Premium is auto-debited every Monday from the rider's linked UPI ID. If the debit fails, coverage is paused for that week and the rider is notified. No penalties, no lock-ins.
 
---
 
## 3. Parametric Triggers
 
Parametric insurance pays out when a **pre-defined, objectively measurable condition is met** — not when a claim is manually assessed. This is the core innovation: no forms, no agents, no disputes.
 
GigGotYou monitors **5 triggers** in real time:
 
### Trigger 1 — Heavy Rainfall
- **Data Source:** Open-Meteo API (free tier) + IMD district-level alerts
- **Threshold:** Rainfall ≥ 35mm/hr sustained for ≥ 30 minutes in the rider's operative zone
- **Rationale:** At this level, most delivery platforms suspend outdoor operations. Validated against Zepto/Blinkit historical suspension data.
- **Geographic Resolution:** Ward-level polygon matching (rider's dark store mapped to municipal ward)
 
### Trigger 2 — Extreme Heat
- **Data Source:** Open-Meteo API
- **Threshold:** Temperature ≥ 44°C during an active slot window
- **Rationale:** At 44°C, outdoor physical labour is a health hazard. Several states have issued advisories restricting outdoor work at this threshold.
 
### Trigger 3 — Air Quality Emergency
- **Data Source:** CPCB AQI API (Government of India, free)
- **Threshold:** AQI ≥ 400 (Severe+), sustained for ≥ 2 hours
- **Rationale:** GRAP Stage IV (invoked at AQI 450+) mandates suspension of non-essential outdoor activities in Delhi NCR. We trigger slightly earlier (400) to match when platforms actually halt operations.
 
### Trigger 4 — Curfew / Local Strike / Zone Closure
- **Data Source:** NLP pipeline processing news feeds (NewsAPI free tier) + manual admin override
- **Threshold:** Verified curfew or bandh affecting the rider's registered zone, confirmed by at least 2 independent sources or admin confirmation
- **Rationale:** Unplanned curfews (post-incidents, political events) are a real disruption in Indian metros. This trigger requires human-in-the-loop confirmation to avoid false positives from rumours.
 
### Trigger 5 — Platform Outage
- **Data Source:** Mock platform heartbeat API (simulated for hackathon; real integration would use platform webhooks)
- **Threshold:** Zero order-dispatch signals for ≥ 45 consecutive minutes during a rider's active slot
- **Rationale:** A 45-minute blackout during peak hours represents a meaningful and verifiable income loss. Short blips (< 45 min) are excluded to prevent noise-triggered payouts.
 
---
 
## 4. Platform Choice: Web + Mobile (PWA + React Native)
 
### Why Both?
 
**The rider interface is mobile-first by necessity.** Q-Commerce riders operate entirely from their phones. They register, check coverage, and receive payouts on mobile. A native-feeling mobile experience is non-negotiable for rider adoption.
 
**The admin/insurer dashboard is web-first by necessity.** Insurance administrators reviewing fraud queues, monitoring live disruption maps, and analysing loss ratios need a desktop-grade data interface. A responsive web dashboard serves this perfectly.
 
### Implementation Strategy
 
Rather than building two entirely separate codebases, we use a **shared-logic architecture**:
 
- **React.js (Web):** Admin dashboard + rider web portal (PWA-capable, installable on Android home screen)
- **React Native (Mobile):** Rider-facing app — onboarding, policy status, payout notifications, coverage toggle
- **Shared:** Business logic, API calls, and state management patterns are mirrored between the two, reducing duplication
 
### Justification for PWA over pure native (for riders)
- Q-Commerce riders often use low-to-mid range Android phones
- PWA eliminates Play Store friction for onboarding (critical for adoption)
- Push notifications, offline caching, and home-screen install work natively in modern Android browsers
- React Native app serves users who want the full native experience
 

-------------------------------------

## 5. AI/ML Integration and Workflow

## 1. Calculating Weekly Premiums

XGBoost will be used for premium calculations. It is a machine learning model that has proven to be very effective when working with structured datasets. By analyzing and predicting what delivery zone a rider delivers from, the projected rainfall for the upcoming week, the current air quality index (AQI), what season it is in the current year, and determining whether or not the delivery zone has a history of prior disruptions with no disruptions in a certain period of time, we will be able to identify the likelihood of the delivery rider losing income based on the number of deliveries per week. This will allow us to create a "risk score" for each rider that we will be able to use to determine their weekly premium. The XGBoost model is a good choice for this use case because it has demonstrated a high level of accuracy and has performed very well with all the real-world scenarios required in this use case.

## 2. Detecting Fraud 

We will be using an anomaly detection model called Isolation Forest to identify unusual patterns of claims that may be fraudulent in nature. For example, if a rider's GPS location does not match the delivery location, if there are multiple claims associated with one disruption event, if there are multiple claims with high frequency for a single rider, or if there are claims during a disruption event for riders who actually did not work during that disruption event. The best usage of the Isolation Forest model is to identify fraud since the number of fraud cases is so small, and the chance of obtaining a statistically significant quantity of labeled fraud claims to use for training is nearly impossible, which makes it impractical to use a traditional classification model for fraud.

## 3. Estimating Income Loss
The rider’s anticipated income loss during an outage will be calculated using regression models to produce a more accurate estimate for payouts. To calculate these estimates, the model will input data such as hours worked, order volume for the area, dependence on peak hours, and historical earnings. This will enable the platform to calculate payouts based on estimated income loss rather than use a flat amount for all riders. As a result, the platform will be more equitable and individualized.

## 4. Recommending Coverage
Using Artificial Intelligence and/or Machine Learning, the system will also be able to recommend the optimal coverage plan on a weekly basis to each rider based upon their work pattern, risk by location, and average earnings on a weekly basis. This would help riders identify whether they should elect for basic, standard or enhanced coverage when they onboard with the platform. This will provide a better onboarding experience and simplify the process of determining which coverage option works best for a rider that may not understand what level of coverage works best for them.

There are many reasons why this AI and ML approach is a powerful force in bringing together various models to create an intelligent and practical platform that can:
•	Logistically reason by fair and personalized pricing on a weekly basis using XGBoost
•	Avoid fraudulent claims and the number of false claims using Isolation Forest
•	Provide accurately calculated payouts based on a transparent way to estimate loss of income via data analysis and predictive modeling
•	Elevate user experience when being onboarded through recommendations that match insurance coverage with their unique situation


Therefore, AI and ML are not just an enhancement but rather the foundation for the way the platform rates risks, protects workers and pprovides rapid, equitable payouts for verified income loss caused by external disruptions.
---------------------------------


## 6. Tech Stack
 
### Frontend
| Layer | Technology | Purpose |
|---|---|---|
| Web App | React.js | Admin dashboard + rider web portal |
| Mobile App | React Native | Rider-facing native mobile app |
| Styling | Tailwind CSS | Responsive UI, consistent design system |
| Charts | Recharts | Dashboard analytics visualizations |
| State | Redux Toolkit | Shared app state (policy, claims, auth) |
| Maps | Leaflet.js | Disruption zone visualization on admin map |
 
### Backend
| Layer | Technology | Purpose |
|---|---|---|
| API Server | Node.js + Express.js | Core REST API — riders, policies, claims, payouts |
| Database | MongoDB + Mongoose | Document store for rider profiles, policies, claims |
| Auth | JWT + bcrypt | Secure rider and admin sessions |
| Job Scheduler | node-cron | Weekly premium debit jobs, trigger polling loop |
| Notifications | Twilio SMS API (trial) | Payout and disruption SMS to riders |
 
### AI/ML Microservice
| Layer | Technology | Purpose |
|---|---|---|
| API Framework | Python + FastAPI | Hosts and serves all AI/ML endpoints for pricing, fraud checks, and risk analysis |
| Premium & Risk Model | XGBoost + scikit-learn | Predicts weekly income-loss risk and calculates dynamic weekly premium |
| Fraud Detection Model | Isolation Forest + scikit-learn | Detects suspicious claim behavior such as GPS mismatch, duplicates, and abnormal claim frequency |
| Income Loss Estimation | XGBoost Regressor / scikit-learn | Estimates likely wage loss during a qualifying disruption using work pattern and zone activity |
| Coverage Recommendation | Rule-based scoring + ML features | Suggests the most suitable weekly protection plan for each rider |
| Feature Processing | pandas + NumPy | Cleans, transforms, and prepares structured weather, AQI, and rider activity data |
| Model Persistence | joblib / pickle | Saves trained models for fast reuse during prediction |
| External Data Integration | Weather API + AQI API + mock platform data | Supplies real-time and historical inputs for pricing, trigger validation, and payout support |
| Monitoring & Logging | FastAPI logs + Python logging | Tracks model requests, fraud flags, and prediction flow for debugging and transparency |
 
### External Integrations
| Service | API | Tier |
|---|---|---|
| Weather | Open-Meteo | Free, no key required |
| Air Quality | CPCB AQI (data.gov.in) | Free |
| News | NewsAPI | Free developer tier |
| Payments | Razorpay | Test/sandbox mode |
| SMS | Twilio | Trial credits |
| Platform Heartbeat | Mock REST API (custom) | Simulated |
 
### Infrastructure
| Component | Tool |
|---|---|
| Version Control | GitHub |
| Backend Hosting | Render (free tier) |
| Frontend Hosting | Vercel |
| ML Service Hosting | Render (separate service) |
| Database | MongoDB Atlas (free tier) |
 
---
 
## 7. Development Plan
 
### Phase 1 — Ideation & Foundation (March 4–20) ✅
- Persona research, scenario definition
- Parametric trigger logic designed
- Weekly premium model formula finalized
- Tech stack decided
- Repository created, README + Idea Document written
- Basic UI wireframes sketched
 
### Phase 2 — Automation & Protection (March 21 – April 4)
**Week 3:**
- Rider registration + onboarding flow (React + React Native)
- MongoDB schema: Rider, Policy, Claim, Payout collections
- Node.js API: `/auth`, `/policy`, `/claims` routes
- ML microservice scaffold (FastAPI) + XGBoost premium model v1
 
**Week 4:**
- 5 parametric triggers integrated (Open-Meteo, CPCB, mock platform API)
- Auto-claim initiation pipeline (trigger → fraud check → payout)
- Basic fraud rules (GPS zone + activity validation)
- Mock Razorpay payout flow
- Phase 2 demo video
 
### Phase 3 — Scale & Optimise (April 5–17)
**Week 5:**
- Isolation Forest fraud model integrated into FastAPI
- Admin dashboard: live disruption map (Leaflet), fraud review queue
- Worker dashboard: earnings protected, coverage status, payout history
- Predictive risk forecast widget (Prophet)
 
**Week 6:**
- End-to-end disruption simulation (trigger fake monsoon → auto-payout demo)
- Performance testing, edge case handling
- Final pitch deck (PDF)
- 5-minute walkthrough video recorded and uploaded
 
---
 
## 8. Why GigGotYou Will Work (Business Viability Notes)
 
**Market Size:** India has ~12 million gig delivery workers. Q-Commerce alone accounts for ~2 million active riders across top 10 cities. Even 5% adoption at ₹50/week average premium = ₹5 crore/week in premium volume.
 
**Unit Economics:**
- Average premium: ₹50/week
- Expected claim frequency: ~15% of weeks (based on disruption data from IMD/CPCB historical records)
- Average payout per claim: ₹600
- Expected loss ratio: (0.15 × ₹600) / ₹50 = 1.8 — i.e., claims would exceed premium at this rate
 
This tells us our base rates need calibration upward OR coverage caps need tightening, which is exactly what the ML risk model does over time. The model is designed to converge toward a sustainable 60–70% loss ratio as it accumulates real claim data.
 
**Distribution Moat:** Partnering directly with Zepto/Blinkit dark store managers for bulk onboarding (workers enrol at the dark store, premium split between rider and platform as a benefit) dramatically reduces CAC.
 
**Regulatory Path:** Parametric insurance in India falls under IRDAI's sandbox framework for InsurTech innovations, which has an expedited approval pathway for micro-insurance products below ₹200/week premium. GigGotYou is designed specifically to operate within this bracket.
 
---
 
*GigGotYou — Phase 1 Idea Document | Guidewire DEVTrails 2026*
*Coverage: Loss of Income ONLY | Weekly Pricing | Q-Commerce Persona*
 




