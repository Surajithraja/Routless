# RouteLess: The Last Mile

> **AI-assisted rural student transit optimizer and real-time disruption resilience platform.**
> **Primary Location**: Configurable Indian Transit Corridor (Default: **Telangana, Medak - TS 09**; Preset: **Karnataka, Dakshina Kannada - KA 19**).

---

## 1. Overview
RouteLess is an open-source, resilient, static web application designed to ensure that rural students in government and private bus corridors reach their schools and examination centers safely and reliably even during vehicle breakdowns, seat shortages, route disruptions, or road blockages.

### Key Capabilities:
- **Zero Backend & Zero Paid API Keys**: Completely client-side static web application running locally or on static hosts (GitHub Pages, Netlify).
- **Location-Based Indian Number Plates (Change 1)**: Authentic registration numbers formatted as `[STATE CODE] [RTO DISTRICT CODE] [SERIES LETTERS] [4 DIGITS]` (e.g. `TS 09 Z 1024` for TSRTC govt bus, `TS 09 UB 2048` for private carrier, or `KA 19 F 1024` for KSRTC). Plates are displayed across bus cards, route map tooltips/popups, and the fleet allocation table.
- **Dynamic Location Config**: Top-level configuration (`stateCode`, `district`, `rtoDistrictCode`, `govtSeries`, `villageNames`) that automatically regenerates all bus plates and landmark names.
- **Surprise Hurdle: "The Exam Day Crisis" (Change 2)**: One-click emergency crisis simulator triggering 2 government bus breakdowns, a 40% demand surge with 2-hour exam deadlines, and major road flooding. The engine executes priority reallocations, flood detours, multi-trip relay dispatches, and generates an audit plan with live countdowns and a "Reset" toggle.
- **My Location Assistant (Privacy-First)**: Intelligent GPS transit guidance engine computing walking ETAs, nearest catchable stops, countdowns, and pooled ride fallbacks right on the device.
- **Multi-Modal Route Comparison**: Dijkstra multi-criteria routing comparing **Shortest (Blue)**, **Safest (Green)**, and **Least Traffic (Orange)** paths with safety and lighting scores.
- **Bilingual & Accessible**: Full Kannada (ಕನ್ನಡ) and English localization, WCAG high-contrast badges, and accessible large touch targets (>40px).

---

## 2. Location-Based Number Plates (Change 1)

Every vehicle in the fleet features an authentic Indian registration number plate:
- **Format**: `[STATE CODE] [RTO DISTRICT CODE] [SERIES LETTERS] [4 DIGITS]` (e.g., `"TS 09 Z 1024"`, `"TS 09 UB 2048"`).
- **Government Bus Distinction**: State government carriers strictly utilize the designated government series letter (`"Z"` for TSRTC in Telangana, `"F"` for KSRTC in Karnataka).
- **Dynamic Configuration (`locationConfig`)**:
  ```javascript
  export const locationConfig = {
    stateCode: 'TS',             // Indian State Code
    stateName: 'Telangana',       // State Name
    district: 'Medak',           // District Name
    rtoDistrictCode: '09',       // RTO District Code (2 digits)
    govtSeries: 'Z',             // Government prefix series
    privateSeries: 'UB',         // Private carrier series
    operatorGovtName: 'TSRTC',
    operatorPrivateName: 'Private stage carriage',
    villageNames: ['Medak Rural', 'Haveli Ghanpur', 'Shankarampet', ...],
    schoolNames: ['Govt Junior College, Medak', 'Telangana Model School', ...]
  };
  ```
  Changing `locationConfig` (or toggling the Location Preset dropdown in the top header) automatically recalculates and regenerates all number plates and village/school names in real time.
- **Plate Visibility**:
  1. **Bus Cards & Timings Board**: Displayed in bold font-mono alongside the operator badge.
  2. **Route Map**: Prominently shown on bus markers, interactive hover tooltips, and click popups.
  3. **Fleet Allocation Table**: Listed with operator, seating capacity, and occupancy metrics.

---

## 3. "The Exam Day Crisis" Simulator (Change 2)

A one-click operational challenge designed for rapid hackathon evaluation:

### The Crisis Trigger:
Clicking **`⚡ Trigger Crisis`** activates 4 compounding emergencies simultaneously:
1. **Bus Breakdown**: Two government buses (`TS 09 Z 1024` and `TS 09 Z 3190`) break down immediately, marked in bold red `❌ BREAKDOWN` and removed from service.
2. **Demand Surge**: Student transit demand surges by **40%** (24 extra exam candidates enrolled, total: 84).
3. **Severe Road Flooding**: The primary highway segment between two villages is flooded, marked in dashed red hazard on the map and excluded from routing.
4. **Time Pressure & Countdowns**: Imminent morning examinations scheduled within the next 2 hours (08:30, 08:45, 09:00, 09:30 AM), with live minutes-remaining countdowns.

### Algorithmic Response Logic:
- **Priority Sorting**:
  1. *Priority 1*: Exam candidates with the least time remaining first (closest exam deadlines).
  2. *Priority 2*: Students with the longest travel distance to destination.
  3. *Priority 3*: General students and special needs priority.
- **Detour Routing & Travel Time Recalculation**:
  - The Dijkstra router excludes the flooded road segment and recalculates detoured travel times.
  - A student is certified **On Time** strictly if their estimated arrival time is before their exam start.
- **Multi-Trip Fleet Consolidation**:
  - Remaining 3 operating buses execute **Trip 1 (Early Express Batch)** and **Trip 2 (Relay Loop Batch)** to double seat capacity.
  - Remote and overflow candidates are assigned to **Emergency Community Shared Autos**.
- **Before vs After Audit Plan**:
  - Summary metrics: Buses Available (`5 ➔ 3`), Total Students (`84`), On-Time % (`88%`), At-Risk (`6`), Unserved/Diverted (`4`), Average Delay (`+14m`).
  - Student filter tabs: `All`, `🟢 On Time`, `🟡 At Risk`, `🔴 Unserved / Pooled`.
- **Reset to Normal**:
  - Clicking **`🔄 Reset to Normal Operation`** restores broken down buses, clears the road block, resets demand, and returns the fleet to standard schedules.

---

## 4. Architecture & File Structure

```
Routless/
├── index.html                 # Complete standalone, single-file distribution (HTML + CSS + JS)
├── package.json
├── SPEC.md                    # Data schemas, contracts & algorithm specs
├── README.md                  # Project documentation & crisis walkthrough
├── scripts/
│   └── buildSingleHtml.mjs    # Bundler compiling pure engine, data, and UI into index.html
├── src/
│   ├── config/
│   │   ├── locationConfig.js  # Location config, RTO presets & plate generator (Change 1)
│   │   └── region.js          # Coastal Karnataka bounds, center [13.13, 74.82], school hours
│   ├── data/
│   │   ├── villages.json      # 8 rural villages & feeder hubs
│   │   ├── schools.json       # 4 landmark educational institutions
│   │   ├── buses.json         # 5 transit buses with realistic Indian number plates
│   │   ├── roads.json         # 17-node, 23-edge road graph with lighting & risk profiles
│   │   ├── students.json      # 60 synthetic student ride records
│   │   └── regions.json       # Multi-region registry (Telangana, Coastal KA, Mysuru, etc.)
│   ├── engine/
│   │   ├── crisis.js          # Pure Exam Day Crisis & Emergency Dispatcher (Change 2)
│   │   ├── distance.js        # Geodesic Haversine calculation & spatial neighborhood sorting
│   │   ├── demand.js          # Moving average demand forecasting with exam multipliers
│   │   ├── allocation.js      # Student bus capacity assignment with special needs priority
│   │   ├── disruption.js      # Vehicle cancellation & road segment blockage impact engine
│   │   ├── pooling.js         # Overflow proximity clustering into Shared Autos & Vans
│   │   ├── routingGraph.js    # Graph connectivity (BFS) & multi-criteria Dijkstra
│   │   ├── routing.js         # Multi-modal routing (shortest, safest, least-traffic)
│   │   ├── simulation.js      # Timetable vehicle interpolation, speed & bearing kinematics
│   │   └── location.js        # Pure location guidance, walking ETA, geofencing & jump smoothing
│   ├── services/
│   │   ├── store.js           # Central reactive state store (immutable updates)
│   │   ├── storageService.js  # Resilient localStorage with volatile memory fallback
│   │   └── routingService.js  # OSRM road geometry client with 4-second timeout fallback
│   └── ui/
│       ├── crisisPanel.js     # Exam Day Crisis Simulator UI panel (Change 2)
│       ├── clock.js           # Simulation clock (1x, 5x, 30x, 60x) with tab visibility handling
│       ├── i18n.js            # English and Kannada (ಕನ್ನಡ) translation dictionary
│       ├── map.js             # Leaflet live bus markers, road network & Google Maps layers
│       ├── locationPanel.js   # My Location Assistant UI (consent, GPS tracking, Demo Walker)
│       ├── timings.js         # Village schedule board with number plates & "Next bus for me"
│       ├── disruptionControls.js # Disruption simulation controls
│       └── dashboard.js       # Seat utilization chart & Fleet Plate Allocation Table
└── tests/
    └── runTests.mjs           # Zero-dependency test runner (49 test suites)
```

---

## 5. Running & Testing Locally

### Running Unit & Integration Tests (100% Pass Rate):
```bash
node tests/runTests.mjs
```
*Executes all 49 tests across pure engine calculations, realistic number plate generation, kinematics, road connectivity, location guidance, and the complete Exam Day Crisis multi-trip emergency reallocation workflow.*

### Rebuilding the Standalone Single-File Bundle:
```bash
node scripts/buildSingleHtml.mjs
```
*Compiles all data, styles, engine logic, location configs, and UI components into `index.html`. You can directly double-click `index.html` in Windows Explorer or open it in any web browser without running any build server or backend!*

---

## 6. Deployment Instructions

### Deploy to GitHub Pages
1. Push your repository to GitHub:
   ```bash
   git init
   git add .
   git commit -m "Deploy RouteLess with Location Plates and Exam Crisis Simulator"
   git branch -M main
   git remote add origin https://github.com/<your-username>/routless.git
   git push -u origin main
   ```
2. In your repository on GitHub:
   - Navigate to **Settings** > **Pages**.
   - Under **Build and deployment**, select **Deploy from a branch**.
   - Select Branch: `main` and Folder: `/ (root)`.
   - Click **Save**. Your site will be published at `https://<your-username>.github.io/routless/` with automatic HTTPS.

### Deploy to Netlify
1. Log in to [Netlify](https://app.netlify.com).
2. Go to **Sites** > **Add new site** > **Deploy manually**.
3. Drag and drop the `Routless` project folder into the upload target.
4. Netlify will instantly provide a secure HTTPS URL for your application.
