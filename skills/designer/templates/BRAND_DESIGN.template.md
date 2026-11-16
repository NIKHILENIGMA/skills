# Brand Design Specification (`docs/BRAND_DESIGN.md`)

> **Status:** `[DRAFT | 🟡 AWAITING_DEVELOPER_APPROVAL | 🟢 APPROVED]`  
> **Last Updated:** `YYYY-MM-DD`  
> **Visual Approver:** Developer / User  
> **Stitch Workspace Link:** `[https://stitch.withgoogle.com/projects/<project-id> | Not Linked Yet]`

---

## 1. Brand DNA & Identity Roots

*Who is this product, what does it feel like, and why does this visual identity exist?*

* **Brand Archetype:** `[e.g., High-Trust Fintech, Minimalist Neo-Brutalist, Editorial Luxury, Playful Bento Consumer, Industrial Cyberpunk]`
* **Unique Identity & Visual Vibe:** `[e.g., Surgical precision, zero-fluff, tactile micro-interactions, high-contrast data clarity]`
* **Target Actor Surfaces:** `[From docs/PRD.md — e.g., Customer Web App, Rider Mobile App, Admin Ops Dashboard]`
* **Visual Metaphors & Motifs:** `[e.g., Crisp 1px hairline borders, subtle radial top-glow, tabular monospace numbers]`

---

## 2. Color Architecture & Palette Strategy

*Strict rule: Every color must have a semantic purpose. Enforce the **60-30-10 rule**.*

### 2.1 Palette Strategy
* **Color Scheme Type:** `[Mono (Single Hue + Neutrals) | Bi-Color (Neutral Base + 1 Contrast Accent) | Tri-Color (Base + Secondary Hue + Action Accent)]`
* **The 60-30-10 Distribution:**
  * **60% Dominant Canvas:** `[Main backgrounds, page wrappers, negative space]`
  * **30% Structural Hierarchy:** `[Cards, sidebars, tables, hairline borders, secondary buttons, muted labels]`
  * **10% Intentional Accent:** `[Primary CTAs, active indicators, key metric highlights, focus rings]`

### 2.2 Semantic Color Token Matrix (Light & Dark Modes)

| Semantic Token | Light Mode (Hex / HSL / OKLCH) | Dark Mode (Hex / HSL / OKLCH) | Role & Usage Boundary |
| :--- | :--- | :--- | :--- |
| `background` (60%) | `#FFFFFF` / `0 0% 100%` | `#09090B` / `240 10% 3.9%` | Root page canvas |
| `foreground` | `#09090B` / `240 10% 3.9%` | `#FAFAFA` / `0 0% 98%` | Primary body copy & headings |
| `card` (30%) | `#F4F4F5` / `240 4.8% 95.9%` | `#18181B` / `240 3.7% 15.9%` | Raised container & panel surfaces |
| `card-foreground` | `#09090B` / `240 10% 3.9%` | `#FAFAFA` / `0 0% 98%` | Text on card surfaces |
| `primary` (10% Accent) | `[#HEX / HSL]` | `[#HEX / HSL]` | Primary CTA buttons, active states |
| `primary-foreground` | `[#HEX / HSL]` | `[#HEX / HSL]` | Text/icons inside primary CTA |
| `secondary` (30%) | `[#HEX / HSL]` | `[#HEX / HSL]` | Secondary actions, filter pills |
| `accent` (Tri-Color) | `[#HEX / HSL or N/A]` | `[#HEX / HSL or N/A]` | Secondary highlight / badge hue |
| `muted` | `[#HEX / HSL]` | `[#HEX / HSL]` | Skeletons, disabled backgrounds |
| `muted-foreground` | `[#HEX / HSL]` | `[#HEX / HSL]` | Subtitles, timestamps, helper text |
| `border` | `[#HEX / HSL]` | `[#HEX / HSL]` | 1px card strokes, dividers |
| `ring` | `[#HEX / HSL]` | `[#HEX / HSL]` | Keyboard focus ring (`focus-visible`) |
| `destructive` | `#EF4444` | `#DC2626` | Delete actions, error banners |
| `success` | `#10B981` | `#059669` | Completed badges, positive delta |
| `warning` | `#F59E0B` | `#D97706` | Pending states, caution alerts |

---

## 3. Typography & Font System

* **Heading Font (`--font-heading`):** `[e.g., Geist, Plus Jakarta Sans, Inter, Syne]`
* **Body Font (`--font-sans`):** `[e.g., Inter, Geist Sans, Outfit]`
* **Monospace / Data Font (`--font-mono`):** `[e.g., JetBrains Mono, Geist Mono, Fira Code]` *(Mandatory for IDs, currency, timestamps, and code)*
* **Font Loading Strategy:** `[e.g., next/font/google | CSS @import | Local WOFF2]`

### 3.1 Type Scale & Hierarchy

| Level | Desktop / Mobile Size | Weight | Line Height | Letter Spacing | Usage Context |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Display** | `48px / 36px` | `700` (Bold) | `1.1` | `-0.02em` | Hero headlines |
| **H1** | `32px / 28px` | `600` (SemiBold) | `1.2` | `-0.015em` | Page-level headers |
| **H2** | `24px / 20px` | `600` (SemiBold) | `1.25` | `-0.01em` | Section headers |
| **H3** | `18px / 16px` | `500` (Medium) | `1.3` | `0` | Card & modal titles |
| **Body** | `14px / 14px` | `400` (Regular) | `1.5` | `0` | Default UI text & tables |
| **Small / Micro** | `12px / 11px` | `500` (Medium) | `1.4` | `+0.01em` | Badges, captions, table headers |

---

## 4. Theme Preference & Surface Geometry

* **Default Theme Mode:** `[Pure Dark | Warm Dark | Crisp Light | System Adaptive]`
* **Corner Radius (`--radius`):** `[0px (Sharp Brutalist) | 6px (Subtle Pro) | 8px (Modern Default) | 12px (Soft Consumer)]`
* **Border Treatment:** `[1px solid hairline | Subtle top-border highlight | 2px Neo-Brutalist stroke]`
* **Elevation & Shadow Strategy:** `[Flat border-first | Diffused multi-layer shadow-sm | Glassmorphic backdrop-blur-md]`
* **Layout Density:** `[Compact (High-density SaaS tables) | Comfortable (Standard Web) | Spacious (Consumer/Editorial)]`

---

## 5. Dev Workspace Token Blueprint (For `slice` & `implement`)

*Note: `designer` NEVER edits code files directly. The `implement` skill applies these exact tokens into `globals.css` / `tailwind.config.ts` during execution.*

```css
/* Target: globals.css (Tailwind v3 / v4 / Raw CSS & shadcn/ui compatible) */
:root {
  --background: /* <Light HSL/OKLCH> */;
  --foreground: /* <Light HSL/OKLCH> */;
  --card: /* <Light HSL/OKLCH> */;
  --card-foreground: /* <Light HSL/OKLCH> */;
  --primary: /* <Light HSL/OKLCH> */;
  --primary-foreground: /* <Light HSL/OKLCH> */;
  --secondary: /* <Light HSL/OKLCH> */;
  --secondary-foreground: /* <Light HSL/OKLCH> */;
  --muted: /* <Light HSL/OKLCH> */;
  --muted-foreground: /* <Light HSL/OKLCH> */;
  --border: /* <Light HSL/OKLCH> */;
  --ring: /* <Light HSL/OKLCH> */;
  --radius: /* <e.g., 0.5rem> */;
}

.dark {
  --background: /* <Dark HSL/OKLCH> */;
  --foreground: /* <Dark HSL/OKLCH> */;
  --card: /* <Dark HSL/OKLCH> */;
  --card-foreground: /* <Dark HSL/OKLCH> */;
  --primary: /* <Dark HSL/OKLCH> */;
  --primary-foreground: /* <Dark HSL/OKLCH> */;
  --secondary: /* <Dark HSL/OKLCH> */;
  --secondary-foreground: /* <Dark HSL/OKLCH> */;
  --muted: /* <Dark HSL/OKLCH> */;
  --muted-foreground: /* <Dark HSL/OKLCH> */;
  --border: /* <Dark HSL/OKLCH> */;
  --ring: /* <Dark HSL/OKLCH> */;
}
```

---

## 6. Design Principles, Anti-Slop Rules & Decision Log

1. **Accent Restraint (10% Cap):** Primary accent color is strictly reserved for primary actions and active indicators—never used as a full card or section background.
2. **State Completeness:** Every screen in `docs/UI_SPEC/` must define Loading Skeleton, Empty State, and Error State visuals before implementation.
3. **Accessibility Floor:** Minimum WCAG AA contrast (`4.5:1` normal text, `3:1` large text) and visible `:focus-visible` rings on all interactive elements.
4. **No Raw HTML Pasting:** Stitch preview code is a structural reference only; `implement` must compose clean project primitives (`shadcn/ui`, Tailwind, or modular CSS).
5. **User Decisions Recorded:**
   * `[Decision 1: e.g., Selected Bi-Color dark-first theme over light mode for night-shift ops]`
   * `[Decision 2: e.g., Enforced JetBrains Mono on all order IDs and price columns]`

---

## 7. Visual Anchors & Approved Stitch Screen Registry

*Before designing any new screen on Stitch, the agent MUST inspect existing rows in this registry to maintain layout and navigation consistency across pages.*

* **External Inspiration / Reference URLs:** `[List any reference sites, Figma URLs, or sketches provided by the user]`
* **Stitch Project ID:** `[stitch://projects/<project-id>]`

| Screen # | Feature ID | Screen Name | Stitch Screen ID | UI Spec File (`docs/UI_SPEC/`) | Status |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `001` | `FEAT-01` | `<e.g., Auth Login & OTP>` | `stitch://screen-001` | [`docs/UI_SPEC/001-auth-login.md`](./UI_SPEC/001-auth-login.md) | `🟢 APPROVED` |
| `002` | `FEAT-02` | `<e.g., Main Dashboard>` | `stitch://screen-002` | [`docs/UI_SPEC/002-main-dashboard.md`](./UI_SPEC/002-main-dashboard.md) | `🟡 PENDING_REVIEW` |
