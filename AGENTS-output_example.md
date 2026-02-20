# AGENTS.md – Figma-to-Code Integration Rules

> Comprehensive rules for translating Figma designs into production code using the Model Context Protocol (MCP) in this codebase.

---

## 1. Token Definitions

### 1.1 Color Tokens

Colors are defined in [`tailwind.config.js`](tailwind.config.js) under `theme.extend.colors`:

```js
colors: {
  'mid-grey':     '#C4C4C4',
  'info-blue':    '#3FA2F7',
  'success-green':'#56C568',
  'primary-cyan': '#00BCD4',
  'page-bg':      '#EFEFEF',
}
```

**Rules:**
- When a Figma color matches an existing token, **always use the token** (e.g. `bg-success-green`, `text-primary-cyan`).
- When a Figma color does not match any token, use Tailwind arbitrary values (e.g. `bg-[#f1a501]`, `text-[#5e6282]`).
- If a new color appears in **3+ components**, add it as a named token in `tailwind.config.js`.

### 1.2 Typography Tokens

Font families are defined in [`tailwind.config.js`](tailwind.config.js) under `theme.extend.fontFamily`:

```js
fontFamily: {
  'roboto':      ['Roboto', 'sans-serif'],
  'poppins':     ['Poppins', 'sans-serif'],
  'volkhov':     ['Volkhov', 'serif'],
  'open-sans':   ['Open Sans', 'sans-serif'],
  'google-sans': ['Google Sans', 'Product Sans', 'Poppins', 'sans-serif'],
  'montserrat':  ['Montserrat', 'sans-serif'],
}
```

Fonts are loaded via Google Fonts `<link>` tags in [`index.html`](index.html).

**Rules:**
- Map Figma font families to the corresponding Tailwind class: `font-roboto`, `font-poppins`, `font-volkhov`, `font-open-sans`, `font-google-sans`, `font-montserrat`.
- Font sizes use Tailwind arbitrary values matching Figma exactly: `text-[18px]`, `text-[50px]`, etc.
- Font weights use standard Tailwind utilities: `font-normal`, `font-medium`, `font-semibold`, `font-bold`.
- Line heights and letter spacing use arbitrary values when needed: `leading-[30px]`, `tracking-[-3.36px]`.

### 1.3 Spacing Tokens

No custom spacing scale is defined. The project uses:
- Tailwind's default spacing scale (`px-8`, `py-6`, `gap-5`, etc.).
- Arbitrary pixel values for Figma-precise spacing: `px-[141px]`, `mt-[105px]`, `gap-[44px]`.

**Rules:**
- Prefer standard Tailwind spacing utilities when the value aligns with the default scale (multiples of 4px).
- Use arbitrary values (`px-[NNpx]`) for Figma-exact measurements that don't match the scale.

---

## 2. Component Library

### 2.1 Component Location

All shared UI components live in [`src/components/`](src/components/):

| Component | File | Description |
|---|---|---|
| `ActionChip` | [`src/components/ActionChip.tsx`](src/components/ActionChip.tsx) | Rounded pill with icon + text (maps to Figma "Chips / Action") |
| `CircleGauge` | [`src/components/CircleGauge.tsx`](src/components/CircleGauge.tsx) | 3-segment donut gauge (maps to Figma "Circle Chart - 3 data points") |
| `ToggleSwitch` | [`src/components/ToggleSwitch.tsx`](src/components/ToggleSwitch.tsx) | Material-style toggle (maps to Figma "Selection Controls / Switch") |
| `LocationIcon`, `DateIcon`, `PersonIcon` | [`src/components/Icons.tsx`](src/components/Icons.tsx) | Inline SVG icon components |

### 2.2 Component Architecture

```
src/
  components/          ← Shared, reusable UI components
    ActionChip.tsx
    CircleGauge.tsx
    Icons.tsx
    ToggleSwitch.tsx
  pages/
    travel/
      TravelLandingPage.tsx   ← Page-level component with local sub-components
```

**Rules:**
- **Shared components** go in `src/components/` as named default exports.
- **Page-specific sub-components** (e.g. `ServiceCard`, `DestinationCard`, `StepItem`, `FooterColumn`) are defined in the same file as the page, below the main export.
- Every component receives a TypeScript `interface` for its props.
- JSDoc comments reference the Figma component name for traceability (e.g. `/** Matches Figma "Chips / Action" */`).
- Components are **functional components** using React hooks where needed.

### 2.3 Component Patterns

```tsx
// Props interface
interface ActionChipProps {
  icon: ReactNode;
  children: ReactNode;
  color?: string;           // optional with default
}

// Default export, functional component
export default function ActionChip({ icon, children, color = 'bg-success-green' }: ActionChipProps) {
  return (
    <div className={`flex items-center gap-2 px-4 h-14 rounded-3xl ${color}`}>
      <span className="flex-shrink-0">{icon}</span>
      <span className="text-lg font-bold text-white font-roboto">{children}</span>
    </div>
  );
}
```

**When creating a new component from Figma:**
1. Create a TypeScript interface for all props.
2. Use default export with a named function.
3. Add a JSDoc comment referencing the Figma node/component name.
4. Accept `className` prop for style overrides when appropriate.
5. Use `ReactNode` for slot-like props (icons, children).

---

## 3. Frameworks & Libraries

| Category | Technology | Version |
|---|---|---|
| UI Framework | React | ^19.0.0 |
| Language | TypeScript | ~5.7.0 |
| Routing | react-router-dom | ^7.13.0 |
| Styling | Tailwind CSS | ^3.4.0 |
| Build System | Vite | ^6.2.0 |
| CSS Processing | PostCSS + Autoprefixer | ^8.5.0 / ^10.4.20 |

**Rules:**
- Use **React functional components** with hooks — no class components.
- All files use **TypeScript** (`.tsx` / `.ts`) with strict mode enabled.
- Routing is via `react-router-dom` v7 (`BrowserRouter` + `Routes` + `Route`).
- No state management library — use React's built-in `useState` / `useReducer`.
- No component library (e.g. MUI, Chakra) — all UI is hand-built with Tailwind.

---

## 4. Asset Management

### 4.1 Figma MCP Assets

The primary asset source is the **Figma MCP asset endpoint**. Assets are referenced as URL constants at the top of page files:

```tsx
const imgDecore2 = "https://www.figma.com/api/mcp/asset/99f896f0-...";
const imgTraveller1 = "https://www.figma.com/api/mcp/asset/ac577d10-...";
```

**Rules:**
- **Always use Figma MCP asset URLs directly** — do not download and re-host unless explicitly required.
- If the Figma MCP Server returns a `localhost` source for an image/SVG, use that source directly.
- **Do NOT create placeholders** if a localhost source is provided.
- Name asset constants descriptively using `img` prefix + Figma layer name: `imgRectangle14`, `imgGroup10`, `imgLogo`.
- Group all asset constants at the top of the file before the component.

### 4.2 Static Assets

- Minimal static assets in the project (only `vite.svg` favicon in `public/`).
- No CDN configuration or image optimization pipeline.

---

## 5. Icon System

### 5.1 Icon Storage & Format

Icons are inline SVG components defined in [`src/components/Icons.tsx`](src/components/Icons.tsx).

### 5.2 Icon Pattern

```tsx
export function LocationIcon({ className = 'w-9 h-9' }: { className?: string }) {
  return (
    <svg className={className} viewBox="0 0 37 37" fill="none" xmlns="http://www.w3.org/2000/svg">
      <path d="..." fill="white" />
    </svg>
  );
}
```

**Rules:**
- Each icon is a **named export** function component.
- Icons accept a `className` prop with a sensible default size.
- SVG attributes use `viewBox` for scalability, `fill="none"` on root, specific fills on paths.
- **Do NOT import external icon packages** (no `react-icons`, `lucide`, `heroicons`, etc.).
- All icons must come from the Figma payload or be hand-crafted SVG.
- Naming convention: `PascalCase` + `Icon` suffix (e.g. `LocationIcon`, `DateIcon`, `PersonIcon`).

### 5.3 Figma-sourced Icons

For page-specific icons, use Figma MCP asset URLs as `<img>` tags rather than converting to SVG components:

```tsx
<img src={imgGroup11} alt="" className="w-[67px] h-[70px]" />
```

---

## 6. Styling Approach

### 6.1 CSS Methodology

**Utility-first Tailwind CSS** — no CSS Modules, Styled Components, or BEM.

### 6.2 Global Styles

Defined in [`src/index.css`](src/index.css):

```css
@tailwind base;
@tailwind components;
@tailwind utilities;

body {
  margin: 0;
  font-family: 'Roboto', sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

#root {
  width: 100%;
  min-height: 100vh;
}
```

**Rules:**
- Default body font is `Roboto` — override per-component with `font-poppins`, `font-volkhov`, etc.
- Keep global styles minimal; all component styling goes in Tailwind classes.
- **No inline `style` attributes** unless absolutely necessary (e.g. dynamic `backgroundColor`).

### 6.3 Styling Patterns

| Pattern | Usage |
|---|---|
| Tailwind utility classes | Primary styling method for all components |
| Arbitrary values `[NNpx]` | Figma-exact spacing, sizes, colors |
| Template literals | Dynamic class composition: `` `${active ? 'z-10' : ''}` `` |
| `style={{ }}` attribute | Only for truly dynamic values (e.g. color from props) |
| Shadow arbitrary values | Complex shadows: `shadow-[0px_20px_60px_rgba(0,0,0,0.06)]` |

### 6.4 Responsive Design

Currently, the codebase uses **fixed pixel layouts** (`max-w-[1440px]`, fixed paddings). No responsive breakpoints are implemented.

**Rules for new pages:**
- Match Figma dimensions exactly using fixed/arbitrary values.
- Use `max-w-[1440px] mx-auto` for page-width containment.
- If the Figma design includes responsive variants, implement them using Tailwind responsive prefixes (`sm:`, `md:`, `lg:`, etc.).

---

## 7. Project Structure

```
figma-demo/
├── index.html                    # Entry HTML with Google Fonts links
├── package.json                  # Dependencies & scripts
├── tailwind.config.js            # Design tokens (colors, fonts)
├── postcss.config.js             # PostCSS + Tailwind + Autoprefixer
├── tsconfig.json                 # TypeScript strict config
├── vite.config.ts                # Vite + React plugin
└── src/
    ├── main.tsx                  # App entry: BrowserRouter + Routes
    ├── App.tsx                   # Dashboard page (route: /)
    ├── index.css                 # Global styles + Tailwind directives
    ├── vite-env.d.ts             # Vite type references
    ├── components/               # Shared reusable components
    │   ├── ActionChip.tsx
    │   ├── CircleGauge.tsx
    │   ├── Icons.tsx
    │   └── ToggleSwitch.tsx
    └── pages/                    # Page-level components
        └── travel/
            └── TravelLandingPage.tsx
```

### 7.1 Routing

Routes are defined in [`src/main.tsx`](src/main.tsx):

```tsx
<BrowserRouter>
  <Routes>
    <Route path="/" element={<App />} />
    <Route path="/travel" element={<TravelLandingPage />} />
  </Routes>
</BrowserRouter>
```

**Rules for adding new pages:**
1. Create the page component in `src/pages/<feature>/<PageName>.tsx`.
2. Add the route in `src/main.tsx`.
3. Extract reusable sub-components into `src/components/` if they appear in 2+ pages.
4. Keep page-specific sub-components in the same file as the page.

### 7.2 File Naming Conventions

| Type | Convention | Example |
|---|---|---|
| Page components | `PascalCase.tsx` | `TravelLandingPage.tsx` |
| Shared components | `PascalCase.tsx` | `ActionChip.tsx`, `CircleGauge.tsx` |
| Icon components | Grouped in `Icons.tsx` | `LocationIcon`, `DateIcon` |
| Config files | lowercase | `tailwind.config.js`, `vite.config.ts` |

---

## 8. Figma MCP Workflow

When implementing a Figma design in this project, follow this sequence:

### Step 1: Fetch Design Context
```
get_design_context → structured node representation
```

### Step 2: Get Visual Reference
```
get_screenshot → visual reference of the target node
```

### Step 3: Fetch Metadata (if needed)
```
get_metadata → high-level node map (for large/truncated responses)
```

### Step 4: Download Assets
- Collect all image/SVG asset URLs from the design context.
- Use Figma MCP asset endpoints directly (do not re-host).

### Step 5: Implement
1. Create asset constants at the top of the file.
2. Build the component using Tailwind utility classes.
3. Map Figma font families → `font-*` tokens.
4. Map Figma colors → existing color tokens or arbitrary values.
5. Match Figma spacing/sizing exactly using arbitrary values.
6. Add JSDoc referencing the Figma node ID/name.

### Step 6: Validate
- Compare rendered output against the Figma screenshot.
- Verify 1:1 visual parity for layout, typography, colors, and spacing.

---

## 9. Key Conventions Summary

| Rule | Detail |
|---|---|
| **No external icon packages** | All icons from Figma payload or inline SVG |
| **No component libraries** | Hand-built UI with Tailwind only |
| **No inline styles** | Unless dynamic values require it |
| **No placeholder assets** | Use Figma MCP URLs directly |
| **TypeScript strict mode** | All files `.tsx` / `.ts` |
| **Functional components** | No class components |
| **JSDoc Figma references** | Document which Figma node a component maps to |
| **Token-first colors** | Use named tokens over hex where available |
| **Arbitrary values OK** | For Figma-exact measurements not in the scale |
| **Default font: Roboto** | Override per-component as needed |
| **1440px max width** | Standard page container width |
