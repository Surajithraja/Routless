# RouteLess: The Last Mile — System Specification

## 1. Project Overview & Folder Structure

RouteLess is a resilient, static web application designed to optimize rural school transit during bus shortages, high demand, and route disruptions. It functions completely client-side without paid APIs or mandatory backend services.

```
routless/
├── index.html
├── package.json
├── vite.config.js
├── vitest.config.js
├── README.md
├── PROMPTS.md
├── SPEC.md
├── public/
│   ├── favicon.ico
│   └── locales/
│       ├── en.json
│       └── ta.json
└── src/
    ├── main.js
    ├── style.css
    ├── data/
    │   ├── villages.json
    │   ├── schools.json
    │   ├── buses.json
    │   ├── students.json
    │   └── historical_demand.json
    ├── engine/
    │   ├── distance.js
    │   ├── demand.js
    │   ├── allocation.js
    │   ├── disruption.js
    │   └── pooling.js
    ├── services/
    │   ├── routingService.js
    │   └── storageService.js
    └── ui/
        ├── map.js
        ├── dashboard.js
        ├── requestForm.js
        ├── disruptionControls.js
        ├── recommendations.js
        └── i18n.js
```

---

## 2. JSON Schemas & Data Contracts

### 2.1 Village (`villages.json`)
```json
[
  {
    "id": "vil_01",
    "name": "Alanganallur",
    "nameTa": "அலங்காநல்லூர்",
    "lat": 10.0435,
    "lng": 78.0945,
    "population": 1200
  }
]
```

### 2.2 School (`schools.json`)
```json
[
  {
    "id": "sch_01",
    "name": "Govt Higher Secondary School, Vadipatti",
    "nameTa": "அரசு மேல்நிலைப் பள்ளி, வாடிப்பட்டி",
    "lat": 10.0832,
    "lng": 77.9620,
    "operatingHours": { "start": "08:30", "end": "16:30" }
  }
]
```

### 2.3 Bus (`buses.json`)
```json
[
  {
    "id": "bus_101",
    "routeNumber": "24B",
    "capacity": 40,
    "status": "ACTIVE",
    "routeStops": ["vil_01", "vil_02", "sch_01"],
    "timeSlots": ["07:30", "15:30"]
  }
]
```

### 2.4 Student (`students.json`)
```json
[
  {
    "id": "stu_001",
    "name": "Kavitha M",
    "villageId": "vil_01",
    "schoolId": "sch_01",
    "preferredTimeSlot": "07:30",
    "specialNeeds": false
  }
]
```

### 2.5 Historical Demand (`historical_demand.json`)
```json
[
  {
    "villageId": "vil_01",
    "schoolId": "sch_01",
    "timeSlot": "07:30",
    "date": "2026-11-10",
    "dayOfWeek": 2,
    "isExamDay": false,
    "studentCount": 32
  }
]
```

---

## 3. Pure Engine Function Signatures (`/src/engine`)

All functions in this directory are pure (deterministic, free of side effects, no DOM or localStorage calls).

### 3.1 Distance & Geodesics (`distance.js`)
```javascript
/**
 * Calculates great-circle distance between two coordinates in kilometers using Haversine formula.
 * @param {{lat: number, lng: number}} coord1
 * @param {{lat: number, lng: number}} coord2
 * @returns {number} Distance in kilometers
 */
export function haversineDistance(coord1, coord2) {}

/**
 * Finds the nearest points from an origin among candidates.
 * @param {{lat: number, lng: number}} origin
 * @param {Array<{id: string, lat: number, lng: number}>} candidates
 * @param {number} [maxDistanceKm]
 * @returns {Array<{candidate: Object, distanceKm: number}>}
 */
export function findNearestNeighbors(origin, candidates, maxDistanceKm) {}
```

### 3.2 Demand Forecasting (`demand.js`)
```javascript
/**
 * Forecasts demand using simple moving average adjusted by day-of-week and exam multiplier.
 * @param {Array<{studentCount: number, dayOfWeek: number, isExamDay: boolean}>} history
 * @param {{dayOfWeek: number, isExamDay: boolean, examMultiplier?: number}} factors
 * @returns {{predictedDemand: number, confidenceInterval: [number, number]}}
 */
export function predictDemand(history, factors) {}
```

### 3.3 Bus & Capacity Allocation (`allocation.js`)
```javascript
/**
 * Allocates student ride requests to available buses based on route alignment and seat capacity.
 * Overflow students are tagged for shared pooling.
 * @param {Array<Object>} requests - List of student requests
 * @param {Array<Object>} buses - List of available buses with capacities
 * @param {Array<Object>} routes - Lookup map of route stops
 * @returns {{
 *   allocations: Array<{busId: string, studentIds: string[], occupancy: number, capacity: number}>,
 *   unassignedStudents: Array<Object>
 * }}
 */
export function allocateStudentsToBuses(requests, buses, routes) {}
```

### 3.4 Disruption Simulation & Re-routing (`disruption.js`)
```javascript
/**
 * Simulates bus cancellation or road blockage, triggering route updates and student reassignment.
 * @param {{
 *   type: 'BUS_CANCEL' | 'ROAD_BLOCK',
 *   targetId: string,
 *   blockedSegment?: [string, string]
 * }} disruption
 * @param {Array<Object>} currentBuses
 * @param {Array<Object>} currentAllocations
 * @returns {{
 *   updatedBuses: Array<Object>,
 *   displacedStudents: Array<string>,
 *   affectedRoutes: Array<string>
 * }}
 */
export function applyDisruption(disruption, currentBuses, currentAllocations) {}
```

### 3.5 Overflow Pooling & Volunteer Rides (`pooling.js`)
```javascript
/**
 * Clusters unassigned or displaced students by spatial proximity (village lat/lng) and time window.
 * Groups students into shared auto/van pools (capacity: 4 to 7).
 * @param {Array<Object>} unassignedStudents
 * @param {Object} villageCoordMap - Map of villageId -> {lat, lng}
 * @param {{maxWalkDistKm: number, maxPoolCapacity: number}} options
 * @returns {Array<{
 *   poolId: string,
 *   pickupLocation: {lat: number, lng: number, villageId: string},
 *   schoolId: string,
 *   timeSlot: string,
 *   studentIds: string[],
 *   vehicleType: 'SHARED_AUTO' | 'VOLUNTEER_VAN'
 * }>}
 */
export function clusterOverflowStudents(unassignedStudents, villageCoordMap, options) {}
```

---

## 4. UI Architecture & State Management

- **State Store**: Single reactive store holding `villages`, `schools`, `buses`, `requests`, `disruptions`, and `allocations`.
- **Storage Service**: Safe wrapper around `localStorage` (`get`, `set`, `remove`) with `try...catch` and fallback to in-memory state.
- **Routing Service**: Network client attempting OSRM routing with a strict 2-second timeout, falling back gracefully to straight-line Haversine geodesics.
- **Leaflet Map**: Visualizes villages, schools, active bus paths, blocked roads, and pooled ride pickup points.
- **Chart.js Dashboard**: Real-time seat occupancy gauge, utilization rate per route, and overflow counts.
- **Localization (i18n)**: Instant English / Tamil toggle switching DOM labels without reloads.

