# UI Context

Source: Esplenin Hotel Project Documentation v0.4 (S04). Design references R1 to R11 are the 11 images Monty provided.

## Theme

Two surfaces with different moods.
- **Guest site**: light, spacious, large photos, one accent color for buttons. A rounded search bar overlaps the hero image (R5, R8, R10). Fully responsive from phone to desktop
- **Staff dashboards**: left sidebar with icons plus a top bar (search, notification bell, profile). Rounded white cards with soft shadows on a light grey background (R2, R3, R11). Desktop and tablet first
- Light and dark mode switch for staff is optional (COULD priority)

## Colors

Define as CSS custom properties. Components use tokens only, no hardcoded hex values. The palette is navy, warm beige and soft gold (guest-site mood), with green, mauve and slate for states. Hex values below are the single source; define them once as tokens.

| Role | CSS Variable | Value | Source name |
| --- | --- | --- | --- |
| Page background (light) | `--bg-base` | `#FCF7F0` | Light cream |
| Surface (cards) | `--bg-surface` | `#FDFDFD` | White |
| Primary text | `--text-primary` | `#1F2A44` | Navy |
| Muted text | `--text-muted` | `#6B6C70` | Darkened grey |
| Primary accent (buttons) | `--accent-primary` | `#C6A75E` | Soft gold |
| Text on accent | `--accent-primary-foreground` | `#1F2A44` | Navy |
| Secondary / dark surface | `--bg-inverse` | `#1F2A44` | Navy |
| Border | `--border-default` | `#E8DCC8` | Warm beige |
| Error | `--state-error` | `#A64044` | Dusty mauve |
| Success | `--state-success` | `#2A6151` | Moderate green |

Contrast notes (WCAG 2.1 AA): navy on gold and navy on cream pass. `--text-muted` (#6B6C70) is about 4.9:1 on cream and 5.2:1 on white (hand-calculated; confirm with a contrast checker). The original grey #77787B was about 4.3:1 and failed, so it is kept only for `--status-grey` (used on chips with text and a border). Do not put gold text on cream or white.

Status colors are fixed and the same everywhere (tokens `--status-green`, `--status-yellow`, `--status-red`, `--status-blue`, `--status-grey`):

| Token | Value | Source name |
| --- | --- | --- |
| `--status-green` | `#2A6151` | Moderate green |
| `--status-yellow` | `#C6A75E` | Soft gold |
| `--status-red` | `#A64044` | Dusty mauve |
| `--status-blue` | `#AAB1C5` | Pale slate |
| `--status-grey` | `#77787B` | Grey |

| Status type | Status | Color |
| --- | --- | --- |
| Room | Available | green |
| Room | Reserved | yellow |
| Room | Occupied | red |
| Room | Cleaning | blue |
| Room | Unavailable | grey |
| Reservation | Pending | yellow |
| Reservation | Approved | green |
| Reservation | Cancelled | grey |
| Reservation | No Show | red |

Status chips must also show the status text, never color alone.

Extended palette (defined as tokens now, no role assigned yet; assign a role here before using one in a component):

| Token | Value | Source name |
| --- | --- | --- |
| `--palette-deep-green` | `#0A3B25` | Deep bluish green |
| `--palette-laurel-green` | `#B2B7AA` | Laurel green |
| `--palette-tuscan-gold` | `#D8C2A4` | Tuscan gold |
| `--palette-warm-beige` | `#E8DCC8` | Warm beige (same as `--border-default`) |
| `--palette-space-indigo` | `#222648` | Space indigo |
| `--palette-faded-copper` | `#8C7B5E` | Faded copper |
| `--palette-khaki-beige` | `#BFAF92` | Khaki beige |
| `--palette-olive-bark` | `#675B1F` | Olive bark |
| `--palette-dark-coffee` | `#3D2523` | Dark coffee |
| `--palette-lime-moss` | `#A9B03F` | Lime moss |
| `--palette-night-bordeaux` | `#490214` | Night bordeaux |

Charts use one consistent color set with a Day / Week / Month / Year toggle.

## Typography

| Role | Font | Variable |
| --- | --- | --- |
| UI and body text | Plus Jakarta Sans (free, OFL) | `--font-sans` |
| Large headings (optional accent) | Cormorant Garamond (free, OFL) | `--font-display` |
| Japanese / Chinese fallback | Noto Sans JP / Noto Sans SC (free, OFL) | in the `--font-sans` stack after Plus Jakarta Sans; load only for `ja` and `zh` |
| Code/mono | JetBrains Mono (free, OFL) | `--font-mono` |
| Last-resort stack | `system-ui, -apple-system, "Segoe UI", sans-serif` | end of every stack |

Load all fonts with `next/font/google` (self-hosted at build, no request to Google from the browser). Read the Next.js 16 font guide in `frontend/node_modules/next/dist/docs/` first. Check French accent coverage on the Google Fonts pages before use.

## Border Radius

Rounded cards and a rounded search bar are required. Exact scale not defined yet.

| Context | Class |
| --- | --- |
| Inline / small UI | `rounded-[0.5rem]` |
| Cards / panels | `rounded-[1rem]` |
| Modals / overlays | `rounded-[1.5rem]` |

Radius values are proposed, not confirmed. Define them as tokens (`--radius-sm`, `--radius-md`, `--radius-lg`) in `rem`.

## Scaling Units

- Use `rem` for font sizes, spacing, radius and component sizes, so everything follows the user's browser font size
- Use `vw` and `vh` only for layout-level sizing (hero height, section width, fluid type via `clamp()`). Prefer `dvh` over `vh` on mobile so browser bars do not cut the layout
- Fluid type uses `clamp(min rem, preferred vw + rem, max rem)`, never `vw` alone, so text stays readable and zoomable (WCAG 2.1 AA)
- No `px` for text or spacing, except 1px borders and hairlines

## Component Library

shadcn/ui on top of Tailwind CSS. Components live in `components/ui/`. Add new ones with the CLI rather than writing from scratch. Charts: Recharts. Icons: Lucide React (the shadcn/ui default).

## Layout Patterns

- Guest top bar: Logo, Home, Rooms, My Bookings, language switch, Book Now button
- Home: hero with floating search bar (Branch, Check-in, Check-out, Guests, Theme), room type cards, "Why book with us" icon row, reviews, footer
- Room pages: photo gallery (4 images, slider), amenity icon grid, price-per-night box with Book Now (R10, R9)
- Booking: two columns (form left, summary card right), stacked on mobile
- Staff layout: left sidebar (sections of the same page, not separate pages), top bar with branch name, search, notifications, theme switch, profile menu
- Dashboards: KPI cards at the top (number, small label, optional trend), then charts, then tables
- Sidebar items are sections of one page per role: `/staff/front-desk`, `/staff/supervisor`, `/staff/manager`, `/admin`
- Super Admin top bar has a branch filter. Picking a branch shows that branch's Manager sections
- Modals for actions: Assign room, Reschedule, New Reservation, Record Payment, Room Transfer
- Disabled actions show the reason
- Every list and page has empty, loading and error states

## Shared Components (build once, reuse everywhere)

Search bar (Branch / Dates / Guests / Theme), room type card, price breakdown card, status chip (reservation, payment, room), KPI card, chart card, data table (search, filters, pagination), side panel and modal, language switch.

## Left Out On Purpose

Blog, offers and packages, Instagram feed, wedding and event pages, newsletter signup, wishlist, price comparison, discounts, in-app chat, orders, channel manager, housekeeping menu, destination maps.
