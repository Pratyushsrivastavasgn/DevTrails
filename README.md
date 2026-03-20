---------------------------------
 Detail out the requirement with persona based scenarios and the workflow
for your application.
-----------------------------






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
○ Detail your plans of integrating AI/ML into the workflow (Premium
calculation, Fraud Detection and so on).
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
 
### ML Microservice
| Layer | Technology | Purpose |
|---|---|---|
| API Framework | Python + FastAPI | Hosts all ML model endpoints |
| Premium Model | XGBoost + scikit-learn | Risk scoring and premium calculation |
| Fraud Model | Isolation Forest (scikit-learn) | Anomaly-based fraud scoring |
| NLP | spaCy | Curfew/strike detection from news |
| Forecasting | Facebook Prophet | Next-week claim volume prediction |
 
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
 




