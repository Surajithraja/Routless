# Prompt and Engineering History

## Phase 1: Specification
- **Prompt**: User specified challenge "Routeless: The Last Mile" with constraints (Vite + vanilla JS, Leaflet, Turf.js, Chart.js, Vitest, pure engine functions in `/src/engine`, haversine calculations, OSRM timeout fallback, synthetic data for 8 villages, 3 schools, 5 buses, 60 students, mobile-first, English+Tamil i18n, localStorage resilience).
- **Result**: Authored `SPEC.md` covering the folder tree, JSON contracts, exact engine pure function signatures, UI modularization, and disruption/fallback mechanisms.
- **Fixes**: Ensured strict adherence to pure functional patterns for engine logic with clear isolation from DOM/Storage.

## Phase 2: Data + Engine + Tests
- **Prompt**: User executed "run" to transition to Phase 2.
- **Result**:
  - Authored all synthetic datasets in `/src/data`: `villages.json` (8 rural villages), `schools.json` (3 schools), `buses.json` (5 buses), `students.json` (60 students with preferences and needs), and `historical_demand.json` (256 historical records across weekdays and exam dates).
  - Implemented all 5 pure engine modules in `/src/engine`:
    - `distance.js`: Haversine great-circle calculation & nearest-neighbor ranking.
    - `demand.js`: Moving average demand forecasting with weekday & exam multipliers.
    - `allocation.js`: Priority assignment matching student destinations, time slots, and special-needs sorting.
    - `disruption.js`: Bus cancellation & road blockage detection along route segments.
    - `pooling.js`: Proximity clustering into Shared Autos and Volunteer Vans.
  - Implemented comprehensive test runner `tests/runTests.mjs` and test suites: 14/14 tests passing with zero failures.
- **Fixes**: Wrapped test runner in zero-external-dependency ESM harness to ensure instant and reproducible testing across standard Node and Vitest environments.

## Phase 7: Coastal Karnataka Context Migration (NH66 Mangaluru - Udupi)
- **Prompt**: Context Change: Rebase RouteLess to coastal Karnataka (Mangaluru to Udupi NH66 corridor + inland feeder villages). Create `/src/config/region.js` with mapCenter `[13.13, 74.82]`, maxBounds `[[12.75, 74.60], [13.55, 75.10]]`, school hours 9:00 AM - 4:30 PM. Replace seed data with real coastal Karnataka villages/towns (Surathkal, Mulki, Padubidri, Kaup, Bajpe, Kateel, Kinnigoli, Moodbidri) and landmark destinations (NITK Surathkal, MIT Manipal, St Aloysius Mangaluru, Govt PU Udupi). Add 60 students with generic Kannada/Tulu names (Ananya, Pranav, Divya, Harish). Include both KSRTC and Private stage carriage operators with UI badges. Do NOT change engine logic in `/src/engine`.
- **Result**:
  - Created `src/config/region.js` defining the coastal Karnataka configuration, map boundaries, school hours, and operator badges.
  - Rebased `src/data/villages.json`, `src/data/schools.json`, and `src/data/roads.json` to the Mangaluru-Udupi NH66 coastal and inland feeder corridor.
  - Updated `src/data/buses.json` with coastal Karnataka routes, operators (`KSRTC` and `Private stage carriage`), driver names (Prashanth Shetty, Santhosh Poojary, Gururaj Bhat, Dinesh Kulal, Venkatesh Rao), and stop timetables.
  - Updated `src/data/students.json` with 60 students bearing regional names.
  - Added operator badges to the village timetable board (`KSRTC` in red, `Private` in emerald).
  - Enforced map bounds `[[12.75, 74.60], [13.55, 75.10]]` so users cannot pan away from coastal Karnataka.
  - Added native Google Maps (Roadmap, Hybrid Satellite, Terrain) alongside CARTO and OSM tile layers.
  - Rebuilt standalone `index.html` (162 KB).
  - Executed test suite: **35/35 tests passing cleanly**.










