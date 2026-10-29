## Overview
"Discount Specials Near Me" is a static, privacy-first single-page application that locates the user, discovers nearby businesses within a 5-mile radius, and surfaces honest, actionable pathways to weekly discounts. Built exclusively for Surge hosting with zero backend or API keys, it relies on OpenStreetMap’s Overpass API for place data and Nominatim for location context. The app deliberately avoids fabricating discount amounts or validity dates. Instead, it bridges users to official chain weekly-ad pages, provides direct contact/site links for independents, and highlights open-day windows to help users time their visits within the requested "today or next five days" window.

## Interface
**Layout & Sections**
- **Header:** App title, dynamic location badge showing town/city name **and** GPS coordinates (lat/lon), subtle paper-texture background.
- **Control Bar:** Sticky top bar containing the `↻ Refresh` button (primary action) and category pills (`Restaurant`, `Grocery`, `Entertainment`, `Pharmacy`). Pills toggle active state; multi-select supported.
- **Status/Note Area:** Contextual messaging for loading, empty, error, and system guidance. Always visible above results.
- **Results List:** Vertical card feed. Each card contains business name, distance, category tag, chain/independent label, external link(s), and a "Next 5 Days Open" chip row.
- **Footer:** Attribution to OSM contributors and static app credits.

**Interactions**
- Tapping a category pill filters the list instantly (client-side).
- `Refresh` clears cached state, re-requests geolocation, runs reverse geocode, hits Overpass, recalculates distances, and re-renders.
- Tapping a business link opens in a new tab (`target="_blank"`).
- Open-day chips are non-interactive but visually indicate visit windows.

**States**
- **Loading:** Spinner + "Requesting location & nearby spots…" (max 8s timeout).
- **Empty:** "No matching businesses within 5 miles. Try adjusting categories or moving to a denser area."
- **Error:** Red banner explaining permission denial, network failure, or Overpass rate limit. Includes retry prompt.
- **Success:** Populated card list with distance-sorted order (nearest first).

**Mobile**
- Responsive flex layout. Cards stack vertically.
- Touch targets ≥44px. Horizontal scroll for category pills if screen is narrow.
- Sticky header collapses to minimal controls on scroll; full header returns on scroll-up.

## Data & behavior
**Sources**
- `Geolocation API` (browser): High-accuracy coordinates with 60s maximum age.
- `Nominatim Reverse Geocode`: `https://nominatim.openstreetmap.org/reverse` → extracts town/city/village.
- `Overpass API`: `https://overpass-api.de/api/interpreter` → fetches nodes/ways within 8047m radius for selected tags.
- `External Links`: Chain weekly-ad URLs curated from official corporate savings pages.

**Transformations**
1. **Radius Filter:** Haversine formula applied to all returned POIs; strictly drops >5.00 miles.
2. **Tag Classification:** Maps OSM tags to the four required categories via explicit matching rules.
3. **Brand/Chain Detection:** Checks `name`, `brand`, and `operator` tags against a local chain dictionary.
4. **Open-Day Projection:** Parses `opening_hours` tokens to map today + next 4 calendar days to open/closed status.
5. **Link Resolution:** Matches chains to verified weekly-ad endpoints. Falls back to `website` → `phone` → "check social" for independents.

**Honesty Rules**
- Zero auto-generated discount text, percentages, or expiration dates.
- All external links are explicitly labeled as `"→ Weekly Ad / Savings Page"` or `"→ Visit Official Site"`.
- Independent businesses receive `"Call or check Instagram/FB for weekly specials"` rather than implied promotions.
- Open-day chips are labeled `"Open next 5 days:"` with today highlighted.

**Caching**
- `localStorage` stores last query payload (`lat, lon, cats, timestamp`).
- On load, if timestamp < 10 minutes, shows cached results instantly while background refresh runs.
- `Refresh` button forces full network bypass of cache.

## Specials model
**Core Principle:** "Where to look, not what we invent." Since no live specials API exists, the app surfaces *access points* and *timing windows* while maintaining strict honesty.

**How "Today or Next 5 Days" is Handled Honestly**
- The app calculates today’s weekday + next 4 days (7-day window).
- It cross-references each business’s `opening_hours` to show which of those 5 days they are open.
- Closed days are rendered muted; open days get a solid fill. Today is highlighted with the accent color.
- A persistent info banner explains: `"Chain weekly ads typically drop on Sundays or Wednesdays. Use the open-day chips below to time your visit. Links go to official ad pages."`
- No fake "Valid through…" or "Today only" text is ever rendered.

**Category-Specific Surfacing**
- **Grocery & Pharmacy:** Highest chain coverage. Each match links to the official weekly ad URL if detected. Independents show `website` + `phone` + "Call for weekly flyers".
- **Restaurant:** Mixed. Major chains link to promos/deals pages. Independents link to site/social or suggest calling. Note: "Restaurants rarely publish fixed weekly specials; check direct channels."
- **Entertainment:** Cinemas & theaters link to promotions/loyalty pages. Bowling alleys, community centers, and indie venues link to site/calendar. Label: "Check for show specials & package deals."

**Real vs. Where-to-Look Labeling**
- Chain matches: `"Official Weekly Ad →"` (direct link)
- Independent matches: `"Site →"` / `"Call →"` / `"Check Social →"`
- All cards include a small footnote: `"Specials cycles vary. Links direct to official savings pages."`

## Acceptance criteria
1. On load, the UI displays both the user’s town/city name **and** GPS coordinates (latitude/longitude).
2. All returned businesses fall within a strict 5.00-mile radius (8047m) of the user’s location.
3. The `↻ Refresh` button triggers a full re-fetch of location, reverse geocode, and POI data without reloading the page.
4. Category pills filter results dynamically; selecting multiple categories shows a union of results.
5. No discount amounts, percentages, or validity dates are auto-generated. All "specials" references are explicitly labeled as external links or direct-to-action prompts.
6. The "Next 5 Days" open-day chips accurately reflect the business’s opening hours, correctly highlight today, and cover exactly 5 calendar days forward.
7. The app runs entirely static (HTML/CSS/JS) with zero backend, zero API keys, and only calls to OSM ecosystem endpoints (Overpass, Nominatim, Geolocation API).

## Visual system
**Palette**
- `--bg: #F7F3EB` (warm off-white, subtle paper tone)
- `--card: #FFFFFF` (clean base)
- `--ink: #2C2824` (soft charcoal, high readability)
- `--accent: #A65E3C` (terracotta/rust, primary action & highlights)
- `--accent-light: #F4E9DE` (chip backgrounds, dividers)
- `--muted: #7A7267` (secondary text, disabled states)
- `--line: #E6DFD3` (card borders, subtle separation)
- `--success: #5B7A6F` (chain ad links, verified indicators)

**Typography**
- System stack: `-apple-system, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif`
- Title: `1.35rem`, `font-weight: 700`, `letter-spacing: -0.02em`
- Body: `0.95rem`, `line-height: 1.55`
- Chips/labels: `0.75rem`, `font-weight: 600`, `text-transform: uppercase`, `letter-spacing: 0.04em`

**Spacing & Structure**
- 8px baseline grid. Padding scales: `0.5rem` (tight UI), `0.8rem` (cards), `1rem` (page edges).
- Cards: `border-radius: 12px`, `box-shadow: 0 2px 8px rgba(44,40,36,0.07)`, `border: 1px solid var(--line)`.
- Subtle background texture: `radial-gradient(var(--line) 1px, transparent 1px)` at `24px` spacing for a faint "weekly flyer paper" feel.

**Tone & Feel**
- Warm, grounded, and glanceable. Avoids corporate tech blues in favor of earthy, print-flyer warmth.
- Hierarchy prioritizes action: distance → name → open days → link.
- Micro-interactions: pill toggle uses `transform: scale(0.97)` on press; link hover adds subtle underline and `--ink` shift.
- Consistent with rural/outdoors aesthetics: clean lines, muted contrasts, ample whitespace, no visual clutter.

## Out of scope
- Real-time dynamic pricing, live coupon scanning, or machine-generated discount values.
- Map visualization (list-only interface per static constraints).
- User accounts, favorites, notifications, or personalized profiles.
- Internationalization, non-Latin scripts, or non-US weekly ad cycle patterns.
- Backend services, databases, CDN asset management, or API key rotation.
- Automated email/SMS alerts or deep-linking to in-app store experiences.