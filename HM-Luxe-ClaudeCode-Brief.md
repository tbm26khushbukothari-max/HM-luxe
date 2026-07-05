# HM Luxe — Prototype Build Brief (for Claude Code)

> **How to use this:** Start a fresh Claude Code session in an empty folder, paste this entire brief as your first message (or save it as `BRIEF.md` and say *"Build this to spec"*). Tell Claude Code to read it fully, build in the order given, self-check against the acceptance criteria, and deploy to Vercel at the end. It should not ask you questions unless it is genuinely blocked.

---

## 0. Who will open this — the only quality bar that matters

This prototype will be opened, on a phone, by ultra-high-net-worth investors and their family offices. The reference class is a **private-bank app, an Aman or Oberoi guest app, an Amex Centurion interface** — discreet, quiet, expensive. It is **not** a SaaS dashboard and must never read like one. If a screen could belong to any B2B startup, it is wrong. Restraint is the aesthetic.

---

## 1. What you're building

A **mobile-first, invite-only clickable prototype** of **HM Luxe** — a managed home-operations service for India's top 1%. One accountable home manager runs the household A-to-Z, backed by a 24–48h continuity guarantee and an operating-layer app the principal opens. This build is the app itself: the demo an investor taps through.

- **Front-end only. Mock data. No backend, no auth, no database.**
- Fully clickable: tab navigation, bottom sheets, a working continuity modal with a state change.
- Rendered inside a **phone frame** on desktop; full-bleed on mobile.

---

## 2. Non-negotiables

- Match the design system in §4 **exactly** — colors, fonts, spacing, motion.
- **Zero generic-template tells.** See §10 for the banned list. This is the difference between "investor-grade" and "AI-built."
- Mobile-first; everything works at 390px wide.
- Subtle, intentional motion. Respect `prefers-reduced-motion`.
- Accessibility floor: semantic HTML, visible keyboard focus, sufficient contrast, `aria-label`s on icon-only controls.
- Clean, typed, componentized code. No dead code, no lorem ipsum, no TODOs left in.

---

## 3. Stack & setup

- **Next.js (App Router) + TypeScript + Tailwind CSS.**
- Fonts via `next/font/google`: **Cormorant Garamond** (display) and **Manrope** (UI/body).
- Icons: **lucide-react**, thin stroke (`strokeWidth={1.4}`), or inline SVG. **No emoji, ever.**
- State: React hooks, in-memory only. No localStorage/sessionStorage needed.
- Mock data lives in `lib/data.ts` and is imported by screens — never hard-code strings inside components.
- Deploys to **Vercel**.

```bash
npx create-next-app@latest hm-luxe --typescript --tailwind --app --eslint --src-dir --no-import-alias
```

Wire the fonts in `app/layout.tsx`:

```tsx
import { Cormorant_Garamond, Manrope } from "next/font/google";
const cormorant = Cormorant_Garamond({ subsets: ["latin"], weight: ["500","600"], variable: "--font-display" });
const manrope = Manrope({ subsets: ["latin"], weight: ["400","500","600","700"], variable: "--font-body" });
// add `${cormorant.variable} ${manrope.variable}` to <body> className
```

---

## 4. Design system — lock these exactly

### Colors (extend `tailwind.config.ts` theme)

```ts
colors: {
  bg:        "#12201b",  // deep lacquer green — app background
  panel:     "#1a2c24",  // cards
  panel2:    "#22382d",  // raised surfaces
  ink:       "#efe9dc",  // primary text (warm ivory)
  body:      "#d7d1c3",  // secondary body text
  muted:     "#9aa79c",  // tertiary / captions
  brass:     "#c2a878",  // the single accent — used with restraint
  brassDeep: "#a2895f",
  sage:      "#8fb79a",  // "present / on track" status
  clay:      "#c98f6b",  // "attention / pending" — used sparingly
},
```

Hairlines (borders/dividers): `rgba(239,233,220,0.11)`. Brass hairlines: `rgba(194,168,120,0.24)`.

### Typography

- **Display (`font-display`, Cormorant Garamond):** greetings, names, big figures, section headlines. Weights 500/600.
- **UI/body (`font-body`, Manrope):** everything else. 400–700.
- **Eyebrows/labels:** Manrope, uppercase, `tracking-[0.22em]`, ~11px, `text-brass`, weight 600.
- Big money figures and the manager's name are **always** Cormorant.

### Shape, spacing, motion

- Cards: `rounded-[20px]`, 1px hairline border, background `panel`. **Prefer hairlines over heavy shadows.**
- Generous whitespace. The signature divider is a **thin brass hairline rule**, not a thick line.
- One — and only one — soft brass radial glow, top-right, very low opacity, as ambient depth.
- Motion: screen changes rise+fade ~400ms ease; tap gives a subtle `scale(0.98)`. Nothing bouncy, nothing that reads as "AI-generated flourish." Gate all of it behind `prefers-reduced-motion`.

---

## 5. App shell

- **Phone frame:** centered, `max-width: 404px`, `rounded-[46px]`, 1px brass hairline border, `bg` background, subtle outer shadow. On viewports < 480px it goes full-bleed.
- **Status bar:** minimal — time ("9:41") left, three faint dots right.
- **Bottom tab bar** (4 tabs), brass active state, thin line icons + tiny uppercase labels:
  - Home · Manager · Household · Ledger
- **Overlays:**
  - Bottom **sheets** (slide up, scrim behind): Membership, Pantry, Tasks.
  - A centered **modal**: continuity "Request cover".

---

## 6. Screens — exact specs, copy & mock data

Use this data verbatim (put it in `lib/data.ts`). It is deliberate; do not invent replacements.

```ts
export const principal = { name: "Mr. Malhotra", initials: "VM", address: "Amrita Shergill Marg · New Delhi" };
export const manager = { name: "Aarti Sethi", initials: "AS", tenure: "14 months", since: "March 2024",
  standby: { name: "Priya Nair", initials: "P" } };
export const today = { staffIn: "4/4", tasks: "6/8", budgetUsedPct: 69 };
export const staff = [
  { role: "Cook", name: "Ramesh Yadav", initials: "R", status: "In · 7:10", present: true },
  { role: "Housekeeper", name: "Sunita Devi", initials: "S", status: "In · 8:05", present: true },
  { role: "Driver", name: "Iqbal Khan", initials: "I", status: "In · 8:30", present: true },
  { role: "Gardener · part-time", name: "Mahesh", initials: "M", status: "Mon & Thu", present: false },
];
export const vendors = [
  { label: "Dairy — daily delivery", status: "Done · 6:40", pending: false },
  { label: "Laundry — weekly pickup", status: "Collected", pending: false },
  { label: "AC annual service", status: "Scheduled · Fri", pending: true },
  { label: "Florist — Sat arrangement", status: "Confirmed", pending: true },
];
export const ledger = {
  month: "November", spent: 124300, budget: 180000,
  categories: [
    { name: "Staff wages", amt: 68000, pct: 100 },
    { name: "Groceries & kitchen", amt: 31400, pct: 46 },
    { name: "Utilities", amt: 12900, pct: 19 },
    { name: "Maintenance & repair", amt: 8600, pct: 13 },
    { name: "Household & misc", amt: 3400, pct: 5 },
  ],
  entries: [
    { name: "Organic grocery run", meta: "Approved · today, 10:12", amt: 4210 },
    { name: "AC annual service — advance", meta: "Scheduled · Fri", amt: 6500 },
    { name: "Salary advance — Sunita", meta: "Approved · Wed", amt: 2000 },
  ],
};
export const membership = {
  placement: 75000, monthly: 25000, managerSalary: 55000,
  includes: [
    "A dedicated, vetted home manager",
    "24–48 hour continuity cover, guaranteed",
    "Insurance & bonding on your manager",
    "The operating-layer app — staff, ledger, pantry",
    "Monthly budget review, quarterly service review",
  ],
};
```

### 6.1 Home (default tab)
- Brand row: `HM` (Cormorant) + `LUXE` eyebrow, left; a small round `VM` avatar, right.
- Greeting (Cormorant, ~33px): **"Good evening, Mr. Malhotra."** Sub (muted): the address.
- **Home-manager card** (the hero): brass-ring `AS` avatar, name (Cormorant), "Running your home · 14 months", chips: `● On duty` (sage dot) · `Vetted` · `Insured`. Brass hairline. Then the **continuity strip**: a small shield seal, eyebrow **CONTINUITY ASSURED**, line *"Priya stands by. If Aarti is ever away, cover arrives within 24 hours — guaranteed."*, and a ghost **"Request cover"** button → opens the continuity modal.
- **Today band:** three cells — `4/4 Staff in` · `6/8 Tasks done` (thin meter) · `69% Budget used` (thin meter).
- **Tiles (2×2):** Household (→ tab) · Ledger (→ tab) · Pantry (→ sheet, "2 items to reorder") · Tasks (→ sheet, "2 open today").
- **Membership bar** (dashed brass border): "HM Luxe Membership / Placement · continuity · oversight" → opens Membership sheet.

### 6.2 Manager (tab)
- Large centered profile: `AS` avatar, name (Cormorant), "With your home since March 2024", chips (`● On duty`, "B.Com · Hotel Mgmt dip.").
- **Trust badges (2×2)**, each a brass check + label: Police-verified · References checked · Insured & bonded · HM Academy certified.
- **What Aarti runs** (hairline list): Household budgeting & the monthly ledger · Staff hiring, rosters & vendor management · Inventory, procurement & reordering · Guest, travel & event hosting.
- **Standby block** (subtle gradient panel, brass border): eyebrow "Your standby manager", *"Priya Nair steps in — within 24–48 hours."*, line about her already holding the home's standing notes, a "Meet your standby" ghost → continuity modal.
- Quiet private-note quote: five small brass stars + *"Discreet, exact, unflappable. I stopped thinking about the house."* — "Your private note · Mar 2025".

### 6.3 Household (tab)
- Headline (Cormorant): "The people who keep it running." Sub: "Coordinated by Aarti · attendance live".
- **Staff list** from `staff` (avatar, role eyebrow, name, status; sage dot when present).
- **Vendors & services** from `vendors` (sage when done, clay when pending).
- Foot note: "Rosters, attendance & pay handled by Aarti — always visible to you."

### 6.4 Ledger (tab)
- Header eyebrow: "Home ledger · November".
- **Budget hero** (centered): Cormorant `₹1,24,300`, muted "of ₹1,80,000 monthly budget", thick-ish brass meter at 69%, sage `● On track · 11 days remaining`.
- **Where it went** — category rows with amount (Cormorant) + thin brass bar per `pct`.
- **Recent entries** — from `ledger.entries` (a `₹` chip, name, meta, Cormorant amount).
- Foot note: "Every rupee logged by Aarti, approved by you. Excludes HM Luxe fees."
- Format all money with Indian grouping (e.g., `₹1,24,300`).

### 6.5 Membership (bottom sheet)
- Title "HM Luxe Membership", sub "Transparent, all-in. No agency mark-up on your staff."
- Three priced lines: **Placement — one time ₹75,000** ("We source, vet, and place your manager. Free replacement if the fit isn't right within 90 days."); **Membership — monthly ₹25,000** ("The continuity guarantee, insurance, the app, and monthly budget oversight."); **Manager's salary — from ₹55,000** ("Paid to Aarti in full, billed at cost. You always see the real number.").
- "Every membership includes" box → `membership.includes` (brass checks).
- Fine print: "Underlying household staff and expenses are billed at actuals and managed within your budget. HM Luxe never marks them up."

### 6.6 Pantry & Tasks (bottom sheets)
- **Pantry:** 2 clay "Reorder" items (Coffee — single origin; Olive oil — 2L) + 3 sage "In stock" (Basmati rice; Atta — 10kg; Tea — first flush). Foot: "Reorders are placed by Aarti against your approved vendors and logged to the ledger."
- **Tasks:** "6 done · 2 open." Three sage "Done" (Morning staff briefing; Grocery run & ledger entry; Confirm Fri AC service) + two clay (Saturday dinner — 8 guests / In progress; Florist & table setting / Open). Foot: "You see what's handled without having to ask. That's the point."

### 6.7 Continuity modal (the signature interaction)
- Trigger: any "Request cover" / "Meet your standby" button.
- Content: a shield seal, "Cover, on standby", line about Priya knowing the home and stepping in within 24 hours, a `who` card (Priya Nair · "Available within 24 hours"), a solid brass **"Request Priya"** button + a muted "Not now".
- On **Request Priya** → the modal body swaps to a confirmed state: "Cover requested", *"Priya will confirm within the hour. You'll get a note the moment she's locked in. Nothing on your end changes."*, and a sage "Done" button. Closing resets it to the default state.

---

## 7. File structure (suggested)

```
src/
  app/layout.tsx        // fonts, metadata
  app/page.tsx          // mounts the PhoneShell
  app/globals.css       // tokens, base
  components/PhoneShell.tsx     // frame + status bar + tab bar + overlay host
  components/TabBar.tsx
  components/Sheet.tsx          // reusable bottom sheet
  components/Modal.tsx          // continuity modal
  components/screens/Home.tsx
  components/screens/Manager.tsx
  components/screens/Household.tsx
  components/screens/Ledger.tsx
  components/sheets/Membership.tsx
  components/sheets/Pantry.tsx
  components/sheets/Tasks.tsx
  components/ui/*        // Eyebrow, Card, Chip, Avatar, Meter, GhostButton
  lib/data.ts
  lib/format.ts         // rupee() with Indian digit grouping
```

---

## 8. Acceptance criteria — self-check before you call it done

- [ ] Cormorant Garamond + Manrope actually load and are applied (display vs body correct).
- [ ] Colors match §4 to the hex; no stray Tailwind defaults.
- [ ] Phone frame on desktop; clean full-bleed under 480px.
- [ ] All four tabs render and switch with the rise/fade motion.
- [ ] Membership, Pantry, Tasks sheets open/close; scrim dismisses.
- [ ] Continuity modal works **including the confirmed-state swap and reset**.
- [ ] Money uses Indian grouping (`₹1,24,300`).
- [ ] No emoji anywhere. No console errors. `next build` passes with no type errors.
- [ ] Keyboard focus visible; icon buttons have `aria-label`; `prefers-reduced-motion` respected.
- [ ] Nothing on any screen reads as a generic SaaS dashboard.

---

## 9. Build order

1. Scaffold Next.js + TS + Tailwind; wire fonts and the color/typography tokens; set the dark `bg`.
2. `PhoneShell` (frame, status bar, tab bar) + routing between the four screens via local state.
3. `lib/data.ts` + `lib/format.ts` + shared `ui/*` primitives (Eyebrow, Card, Chip, Avatar, Meter, GhostButton).
4. Screens in order: Home → Manager → Household → Ledger.
5. Overlays: Sheet component, then Membership / Pantry / Tasks; then the continuity Modal with its state swap.
6. Motion + accessibility pass.
7. Run the acceptance checklist, fix, then deploy.

---

## 10. Explicitly avoid (the "cheap tell" list)

Purple/violet or blue→pink gradients · neon or acid accents · default Tailwind blue links · emoji or emoji-style icons · heavy/blurry drop shadows · rounded-full pill buttons everywhere · glassmorphism · stocky illustration · "Dashboard / Analytics / Overview" SaaS chrome · dense borders and boxes · exclamation-mark marketing copy · lorem ipsum · more than one accent color. When unsure, remove the accessory and let the whitespace and the brass hairline carry it.

---

## 11. Deploy to Vercel

```bash
npm i -g vercel      # if needed
vercel               # first run links/creates the project
vercel --prod        # ship the shareable URL
```

Confirm the production URL opens cleanly on a phone. That link is what goes to investors — treat its polish as part of the deliverable.
