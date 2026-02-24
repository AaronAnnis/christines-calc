# DogCalc v2 Design Document

**Date:** 2026-02-23
**Purpose:** Feature expansion — customer management, auto-mileage routing, multi-stop trips, and quote enhancements

## 1. Customer System

### Data Model

New localStorage key: `dogcalc_customers`

```json
{
  "id": "cust_1708700000000",
  "name": "Jane Smith",
  "notes": "Has two golden retrievers. Prefers morning pickups.",
  "createdAt": "2026-02-23T14:30:00.000Z"
}
```

IDs generated as `cust_` + `Date.now()`.

### Navigation

Fourth tab added to bottom nav bar: **Calculator | History | Customers | Settings**

- SVG person icon for Customers tab, using `currentColor`, same stroke style as existing tab icons
- All four tabs must fit comfortably at 414pt width (iPhone 11)

### Customers Screen

- Scrollable list of saved customers, sorted alphabetically by name
- Each row shows customer name and note preview (first line, truncated)
- "+" button in top-right corner to add a new customer
- Tap a customer row to open their detail view
- Swipe-to-delete (iOS pattern) or long-press to remove a customer

### Customer Detail View

- Full-screen view pushed on top of Customers tab
- Back arrow in top-left to return to customer list
- Shows: name (editable), notes (editable textarea), creation date
- Below: chronological list of all quotes associated with this customer
- Each quote row shows date, route/miles, total, and status (if set)
- Tap a quote to expand its full breakdown (same as History detail)

### Calculator Integration

- New "Customer" text input at the top of the calculator form, above miles
- As the user types, a dropdown shows matching saved customers (case-insensitive substring match on name)
- Selecting a customer from the dropdown fills the input and stores the customer ID on the quote
- If the typed name doesn't match any saved customer, the quote saves with just the name string (no ID)
- "Copy Quote" saves the customer association in the quote history entry

### History Integration

- History list rows show the customer name (if associated) below the date/miles line
- History search (see Section 4) filters by customer name

## 2. Auto-Mileage via OSRM

### Overview

Replace the single "Miles" numeric input with location-based routing. The user types origin and destination addresses; the app geocodes them and calculates driving distance automatically.

### APIs

| Service | Endpoint | Purpose |
|---------|----------|---------|
| Nominatim | `https://nominatim.openstreetmap.org/search?q={query}&format=json&countrycodes=us&limit=5` | Geocode addresses to lat/lng |
| OSRM | `https://router.project-osrm.org/route/v1/driving/{lng1},{lat1};{lng2},{lat2}?overview=false` | Calculate driving distance |

Both are free, no API key required. Rate-limit Nominatim to 1 request/second per their usage policy.

### UI Changes

- **From** text input: address/city for origin
- **To** text input: address/city for destination
- As the user types in either field (debounced 500ms), show a dropdown of up to 5 address suggestions from Nominatim
- When the user selects a suggestion (or blurs the field after typing), geocode the address
- Once both From and To have valid coordinates, fire the OSRM route request
- Auto-fill the miles field with the result (rounded to nearest whole number)
- Show a small loading spinner inline while the route is being calculated

### Manual Override

- The miles numeric input remains visible below From/To
- If the user types a number directly into miles, it overrides the calculated value
- If the user then changes From or To, the override is cleared and recalculated
- This ensures the app still works fully offline (just type miles manually)

### Error Handling

- If Nominatim returns no results: show "No addresses found" in the dropdown
- If OSRM fails or is unreachable: leave miles field empty, user enters manually
- If the device is offline: From/To fields are still usable but suggestions and routing won't work; manual miles input works as before

## 3. Multi-Stop Trips

### UI

- "Add Stop" button between the From and To fields
- Each tap adds a new location text input between the previous stops and the To field
- Each stop has an "x" button to remove it
- Maximum 5 stops (practical limit for a minivan trip)
- Stops use the same Nominatim auto-suggest as From/To

### Route Calculation

Route is calculated as a chain: **From -> Stop 1 -> Stop 2 -> ... -> To**

- Each leg is a separate OSRM request (or a single OSRM request with waypoints: `{lng1},{lat1};{lng2},{lat2};{lng3},{lat3}`)
- Total miles = sum of all legs
- Miles field auto-fills with the total

### Per-Dog Mileage

- By default, every dog rides the full route (total miles)
- Each dog card gains an optional "Miles" override field
- If a dog is only going part of the route (e.g., pickup at Stop 1, dropoff at Stop 2), the user can enter that dog's specific mileage
- Per-dog mileage is used for that dog's mileage charge calculation instead of the total

### Quote Output

The copied quote text includes the route:

```
--- Pet Transport Quote ---
Route: Portland, ME -> Boston, MA -> Hartford, CT
Total: 250 miles (Economy)
1x Large Dog (250 mi)
1x Small Dog (120 mi — Boston, MA to Hartford, CT)

Base Fee:           $200.00
Mileage (250 mi):   $250.00
Mileage (120 mi):   $120.00
Size Surcharge:      $50.00
Sanitation:          $35.00
─────────────────────────
TOTAL:              $655.00
```

## 4. Quote Enhancements

### Notes Field

- Free text `<textarea>` on the calculator screen, placed below the dogs section and above the surcharges accordion
- Placeholder text: "Trip notes (optional)"
- Max 500 characters
- Saved with the quote in history
- Displayed in the history detail view
- Included in the copied quote text (if non-empty), placed after the line items and before the total

### Quote Status

Each quote in history can carry a status:

| Status | Badge Color | Meaning |
|--------|-------------|---------|
| (none) | No badge | Default — just created |
| Sent | Gray (`systemGray`) | Quote texted to customer |
| Accepted | Green (`systemGreen`) | Customer confirmed |
| Declined | Red (`systemRed`) | Customer passed |

- Small colored badge/pill shown on the history list row
- Tap the badge to cycle: none -> Sent -> Accepted -> Declined -> none
- Status is persisted in the quote's localStorage entry
- Status is purely informational — no logic depends on it

### Quick Re-Quote

- Each quote in the history detail view has a "Re-quote" button
- Tapping it:
  1. Switches to the Calculator tab
  2. Pre-fills all fields: miles, From/To (if stored), dogs (count, sizes), service tier, surcharges, customer, notes
  3. Does NOT auto-copy — the user reviews, adjusts if needed, then taps "Copy Quote" to generate a new quote
- The original quote is unmodified; re-quoting creates a new history entry

### Search History

- Search bar pinned at the top of the History screen (iOS-style search field with magnifying glass icon)
- Filters quotes by customer name — case-insensitive substring match
- Results update live as the user types
- If no quotes match, show centered "No results" text
- If the search bar is empty, show all quotes (default behavior)

## Technical Considerations

### Still a Single HTML File

All new features stay within the single `index.html` file. No external dependencies, no build step.

### localStorage Schema

| Key | Contents |
|-----|----------|
| `dogcalc_settings` | Existing — rates, business name |
| `dogcalc_history` | Existing — array of quote objects (now with `customerId`, `customerName`, `notes`, `status`, `route` fields) |
| `dogcalc_customers` | New — array of customer objects |

### Offline Behavior

- Customer system, quote status, notes, search, and re-quote all work fully offline
- Auto-mileage and multi-stop routing require internet for Nominatim/OSRM calls
- Manual miles input always works as fallback
- The app must never break or show errors when offline — routing features gracefully degrade

### Migration

- Existing quotes in history (from v1) will have no `customerId`, `customerName`, `notes`, `status`, or `route` fields — these should all be treated as absent/null and display correctly
- No data migration script needed; new fields are simply absent on old entries
