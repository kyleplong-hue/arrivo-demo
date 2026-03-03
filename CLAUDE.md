# Arrivo Concierge — Feature Build

## Project
Two single-file React concierge apps (Babel in-browser, no build step):
- `montepulciano/index.html` — Tuscan villa (Molino Nobile), 7-day itinerary
- `concierge/index.html` — Shanghai food week (UnTour)

Both share the same architecture: React 18 via CDN, inline data arrays, localStorage persistence, Google Maps links, owner dashboard with PIN.

## Feature 1: Link Sharing & Permissions

### Current state
- Owner creates guest via dashboard → stores in React state + localStorage
- "Copy Link" encodes entire itinerary as base64 in URL query param `?g=<id>&d=<base64>`
- Anyone with the link can edit (swap cards, delete items)

### What to build
Add three permission levels to the guest link system:

1. **Owner** (PIN-protected, as now): Full edit + template saving + card editor
2. **Editor** (guest who receives link): Can swap cards, reorder, add notes, request bookings. URL: `?g=<id>&role=editor&d=<base64>`
3. **Viewer** (guest forwards to friends): Read-only — no swap buttons, no delete, no reorder. URL: `?g=<id>&role=viewer&d=<base64>`

### Implementation details

**In the Owner Dashboard (OwTab):**
- When owner clicks "Copy Link", generate TWO links:
  - Editor link (primary, auto-copied)
  - Viewer link (shown as secondary option, "Copy view-only link")
- Show both in a small popup/toast after clicking

**In the Guest view:**
- Parse `role` from URL params (default to `editor` for backwards compat if no role param)
- If `role=viewer`:
  - Hide ALL swap (↻) buttons on cards
  - Hide ALL delete (✕) buttons on cards  
  - Hide the booking request form (show bookings read-only)
  - Show a subtle banner: "View-only — ask your host for an editor link"
- If `role=editor`:
  - Show swap and delete as now
  - Add a "Share with your party" button in the header that copies the VIEWER version of the current link
  - Booking form works as now

**For both concierge files:**
- The `role` param should be read from URL in the App component
- Pass `role` down as prop to Card, WkTab, BkTab, Header
- Card component: conditionally render swap/delete buttons based on role
- Header: show "Share" button for editors

### Important constraints
- Keep everything in a single HTML file (no external JS files)
- Keep the existing design/styling exactly as-is
- Don't break existing URL format — old links without `role` param should still work (default to editor)
- Test by checking that viewer links hide all edit controls

## Feature 2: Google Maps Travel Times + Flexible Starting Point

### What to build
Add travel time estimates to every activity card, calculated from a configurable starting point.

**Starting Point Selector:**
- Add a small dropdown/input at the top of the "Your Week" tab
- Default: the property address (Molino Nobile for Montepulciano, central Shanghai for concierge)
- User can type a new address — use Google Places Autocomplete
- Store selected starting point in localStorage
- For Montepulciano: default coords [43.0891, 11.7868] (Molino Nobile area)
- For Shanghai: default coords [31.2200, 121.4580] (central Shanghai)

**Travel Time on Cards:**
- Each card already has a `m` (map query) field with the destination
- Add coordinate data for each venue (extend the existing PL/COORDS objects)
- Calculate haversine distance from starting point to each venue
- Display as pill badge on each card:
  - Walking: 🚶 X min (show if ≤30 min, assume 5 km/h walking speed)
  - Driving: 🚗 X min (always show, assume 40 km/h average for rural Tuscany, 25 km/h for Shanghai)
- Use haversine for MVP (no Google API calls needed)
- Place the travel badge next to the existing Map badge

**Coordinate Data to Add:**
For Montepulciano (montepulciano/index.html), add a COORDS object mapping venue names to [lat, lng]:
- Molino Nobile (starting point): [43.0891, 11.7868]
- De Ricci: [43.0937, 11.7866]
- Bindella: [43.0810, 11.7750]
- Salcheto: [43.0750, 11.7920]
- Podere il Casale: [43.0780, 11.6860]
- Acquachetta: [43.0930, 11.7870]
- La Grotta: [43.0935, 11.7860]
- Montepulciano centro: [43.0937, 11.7866]
- Pienza: [43.0763, 11.6790]
- Bagno Vignoni: [43.0520, 11.6210]
- San Quirico: [43.0570, 11.6060]
- Montalcino: [43.0580, 11.4900]
- Sant'Antimo: [43.0370, 11.5200]
- Siena: [43.3188, 11.3308]
- Theia: [43.0580, 11.8310]
- La Foce: [43.0270, 11.7170]
- Cugusi: [43.0763, 11.6790]
- Olmaia: [43.0800, 11.8100]
- Rosso Vivo: [43.0580, 11.8310]
- Caffe Poliziano: [43.0937, 11.7866]
- Coop Chianciano: [43.0530, 11.8280]
- Buongusto: [43.0937, 11.7866]
- Dopolavoro: [43.0270, 11.7170]
- Contucci: [43.0937, 11.7866]

These are approximate — good enough for travel time estimates.

### Important
- Haversine function: standard formula, returns km
- Walking: distance_km / 5 * 60 = minutes
- Driving: distance_km / 40 * 60 = minutes (Tuscany), / 25 * 60 (Shanghai)
- Round to nearest 5 minutes for cleanliness
- Don't add Google API calls — pure client-side math for now

## Build order
1. Do montepulciano/index.html FIRST (both features)
2. Then replicate to concierge/index.html (adapt coordinates and defaults)
3. Commit after each file is done

## When finished
Run: openclaw system event --text "Done: Arrivo concierge features — link permissions + travel times on both pages" --mode now
