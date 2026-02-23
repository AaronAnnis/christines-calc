# DogCalc Design Document

**Date:** 2026-02-23
**Purpose:** Mobile pet transport quote calculator for an independent minivan operator

## Problem

The operator charges $0.50/mile flat — well below market rates ($0.85-$1.60/mile + base fees). She needs a quick way to calculate proper quotes on her phone while managing shared rides with multiple dogs.

## Solution

A single standalone HTML file that runs on her iPhone like a native app. No internet, no app store, no accounts. Saved to home screen, opens full-screen.

## Pricing Model

Based on industry research (TLC Pet Transport, CitizenShipper, Happy Tails Travel, Signature Pet Transport) and the provided pricing strategy PDF.

### Core Fees (adjustable in Settings)

| Fee | Default | Description |
|-----|---------|-------------|
| Base Booking Fee | $200 | Admin, loading, insurance |
| Per-Mile Rate | $1.00 | Based on direct Google Maps distance |
| Sanitation Fee | $35 | Every booking, non-negotiable |

### Dog Size Surcharges

| Size | Weight Range | Surcharge |
|------|-------------|-----------|
| Small | Under 25 lbs | $0 |
| Medium | 25-50 lbs | $25 |
| Large | 50-80 lbs | $50 |
| XL | 80+ lbs | $100 |

### Litter/Puppy Pricing

| Scenario | Rate |
|----------|------|
| First puppy | Full standard rate |
| Each sibling (under 16 weeks) | $75 flat |
| Each sibling (over 16 weeks) | $100 flat |

### Service Tiers

| Tier | Pricing |
|------|---------|
| Economy (Shared) | Standard rate |
| Private (VIP) | Base fee x2 + $2.00/mile |

### Optional Surcharges

| Surcharge | Rate |
|-----------|------|
| Detour Fee | $1.50/mile beyond 30-mile threshold |
| Wait Time | $1.00/min after 20-min grace period |
| Holiday/Weekend | +20% of total |
| Overnight Stay | $97/night |

## UX Design

### Principles (Apple HIG + research)

- iOS-native feel: San Francisco font via `-apple-system`, system colors, grouped table layout
- Touch targets: minimum 44x44pt (Apple standard)
- Thumb-zone layout: primary actions in bottom 50-60% of screen
- Progressive disclosure: surcharges hidden behind expandable section
- Live-updating total pinned at bottom of screen
- No sliders (research shows they're fiddly on mobile)

### Input Methods (per NN/G research)

- Miles: numeric keypad input (values too large for steppers)
- Dog count: +/- stepper buttons (small bounded values)
- Dog size: large tap-friendly buttons
- Service tier: iOS segmented control
- Surcharges: iOS toggle switches with conditional fields

### Three Screens

1. **Calculator** — miles, dogs, tier, surcharges, live total, "Copy Quote" button
2. **Quote History** — scrollable list of past quotes, tap to view full breakdown
3. **Settings** — adjust all rates, business name for quote footer

### Navigation

iOS-style tab bar pinned at bottom: Calculator | History | Settings

### Data Persistence

Browser localStorage — persists across sessions, no internet needed.

### Quote Output

Clean text breakdown copied to clipboard for texting to customers:

```
--- Pet Transport Quote ---
Trip: 300 miles (Economy)
1x Large Dog

Base Fee:           $200.00
Mileage (300 mi):   $300.00
Size Surcharge:      $50.00
Sanitation:          $35.00
─────────────────────────
TOTAL:              $585.00

Thank you for choosing [Business Name]!
```

### PWA Features

- Add to Home Screen metadata (manifest-like meta tags)
- Full-screen mode (no browser chrome)
- Status bar color matching
- Auto dark mode detection

## Technical

- Single `.html` file, zero dependencies
- All CSS inline in `<style>` tag
- All JS inline in `<script>` tag
- Target: Mobile Safari (iPhone), also works on Android Chrome
- localStorage for persistence
