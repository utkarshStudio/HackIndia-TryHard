# SeaSafe — Captain's Copilot

> **Built under HACKINDIA**
>
> A bridge-console decision-support cockpit for merchant shipping. SeaSafe ingests an advisory (geopolitical threat, cyclone, port congestion), runs a transparent local-first AI orchestrator across maritime tools, and surfaces a recommended course of action — with full legal-data-compliance masking of vessel telemetry under DPDP / GDPR / IMO regimes.

---

## 1. What SeaSafe is

SeaSafe is a **single-page web application** that simulates the workflow of a ship's master being handed an advisory mid-voyage. It models the realities of modern bridge decision-making:

- An advisory arrives (missile threat, piracy, cyclone, drought, port congestion).
- A copilot runs tool calls — chokepoint status, weather hazards, route metrics, route comparisons, port congestion.
- A `Decision Card` proposes a recommended course with quantified deltas (ETA, fuel tons, USD, CO₂).
- The captain accepts, dismisses, or overrides — every action lands in an immutable audit log.
- Throughout the voyage, **Compliance Mode** legally masks the vessel's track to match the jurisdiction it is sailing through.

There is no upstream LLM provider you must pay for, no API key to configure. The orchestrator is deterministic and demo-reliable; an optional local Ollama runtime can author the captain rationale.

---

## 2. Core features

### 2.1 Five built-in scenarios (keys `1`–`5`)
| # | Scenario | Theme |
|---|---|---|
| 1 | **Red Sea Missile Threat** | Suez vs. Cape of Good Hope reroute, severity 4 |
| 2 | **Strait of Hormuz Seizure Risk** | Das Island vs. Fujairah / Khor Fakkan bypass |
| 3 | **Panama Canal Drought** | Panama vs. Suez vs. Cape Horn — drought-driven draft restrictions |
| 4 | **Singapore Strait Piracy** | Malacca vs. Sunda vs. Lombok piracy reroute |
| 5 | **Arabian Sea Cyclone** | Direct vs. southern bypass vs. Indian coast hug |

### 2.2 Custom scenario builder
The header **+ New Scenario** button opens a builder. Provide vessel name, vessel type (container, tanker, bulk, RoRo, general cargo), origin port, destination port, and severity (1–5). SeaSafe runs the route planner, picks crossed danger/threat zones, and synthesises a fully working scenario with routes, advisory, orchestrator script, and decision deltas.

### 2.3 Local-first copilot panel
The right-hand `AgentPanel` streams every step of the orchestrator: thinking → tool calls → writing rationale → done. Each tool call shows arguments, latency, and a typed result row (severity, ETA, deltas, port congestion).

Tools currently wired:
- `check_chokepoint_status`
- `check_weather_hazards`
- `calculate_route_metrics`
- `compare_routes`
- `check_port_congestion`

The captain rationale is streamed token-by-token; if the local model is unavailable, a deterministic rationale is composed instead and a fallback notice is shown.

### 2.4 Decision card with deltas & radar comparison
When the orchestrator finishes, the bottom-right `DecisionCard` shows the recommended route, two alternatives, and per-route deltas vs. the active route (`±h`, `±USD`, `±t CO₂`). The top-left `RouteRadarChart` plots all three routes across **Risk · Cost · Carbon · Speed · Reliability**. Click any spoke or card to make it the captain's call.

### 2.5 Compliance mode (press `C`)
Maritime data does not live in a vacuum — vessel positions are personal data under several regimes. Compliance Mode renders a forward-only legal display window for the active route. The masking engine picks the jurisdiction the vessel is currently in and applies that zone's look-ahead and disposal policy.

| Regime | Code | Look-ahead | Disposal | Legal basis (paraphrased) |
|---|---|---|---|---|
| **India DPDP Act 2023 + IT/SPDI Rules** | `india_dpdp` | 100 nm | 0 h on exit | DPDP §8(7) storage limitation |
| **EU GDPR + e-Privacy** | `eu_gdpr` | 150 nm | 24 h | GDPR Art. 5(1)(c)/(e) |
| **UAE Federal Data Protection Law (FDPL)** | `gulf_uae_fdpl` | 120 nm | 12 h | UAE FDPL Art. 5 purpose limitation |
| **International / IMO default** | `international_imo` | 200 nm | 48 h | SOLAS V/19 + IMO MSC-FAL.1/Circ.3 |

The `ComplianceHUD` (bottom-left) shows the active zone, governing law, look-ahead window, disposal deadline, route progress, and how many waypoints have been disposed of. The vessel-clock advances voyage time at a demo-speed factor and toasts every zone transition.

### 2.6 Audit log
Every accept / dismiss is timestamped with the from/to route, recommendation, rationale snippet, and the full tool-call trace. Rendered in the bottom-left `AuditLog` panel, persistent across scenario switches. A "Captain's call" badge marks decisions that overrode the AI recommendation.

### 2.7 Map & layers
A MapLibre + deck.gl canvas renders:
- **Route paths** (current = blue, alternates = dim green, recommendation crossfades to bright green over 800 ms on accept)
- **Vessel** as a chevron icon with heading
- **Risk polygons** for chokepoints
- **Weather hazard zones** (live polygons keyed off the scenario)
- **Compliance overlays** when Compliance Mode is on

A `flyTo` ease animates between scenarios; route swaps crossfade.

### 2.8 Keyboard shortcuts
| Key | Action |
|---|---|
| `R` | Replay scenario / reset |
| `C` | Toggle compliance mode (where regulated) |
| `A` | Accept selected route (when DecisionCard open) |
| `Esc` | Dismiss DecisionCard |
| `1`–`5` | Switch scenario |

A `?` button bottom-right hovers the full list.

### 2.9 Demo polish
- Inter font, tabular numerics for figures, uppercase tracked labels for HUD chrome.
- Toasts via Sonner for transitions, decisions, and zone entries.
- Tool-call rows stagger in (150 ms each); rationale streams with a caret.
- Live ETA + course in the header; pulsing live-dot next to the vessel name.
- Error states render inline with a Retry button — no silent failures.

---

## 3. Tech stack

| Layer | Choice |
|---|---|
| Framework | **Next.js 16** (App Router, webpack) on **React 19** |
| Language | **TypeScript 5** |
| Styling | **Tailwind v4** + `tw-animate-css` + shadcn primitives |
| State | **Zustand** (single bridge store) |
| Map | **MapLibre GL** + **deck.gl 9** (PathLayer, PolygonLayer, ScatterplotLayer, TextLayer, IconLayer), `react-map-gl` |
| UI primitives | `@base-ui/react`, lucide-react icons |
| Toasts | Sonner |
| AI runtime | OpenAI-compatible client pointed at a local Ollama instance (`http://localhost:11434/v1`) for the captain rationale only |
| Tiles | OpenFreeMap Positron |

No paid APIs. No environment variables required. No keys.

---

## 4. Project layout

```
app/
  api/orchestrator/route.ts    streamed tool-call orchestrator
  api/ai/warmup/route.ts       optional local-LLM warmup ping
  layout.tsx, page.tsx, globals.css, icon.tsx
components/
  BridgeHeader.tsx, ScenarioPicker.tsx
  bridge/   AdvisoryBanner, AgentPanel, DecisionCard, RadarPanel,
            RouteRadarChart, ComplianceHUD, AuditLog,
            KeyboardHints, NewScenarioModal, ToolCallRow
  map/      BridgeMap, routeLayer, vesselLayer, riskZoneLayer
  ui/       button, card, badge, select, sonner
lib/
  scenarios/        red-sea, hormuz, panama, malacca, cyclone (+ index)
  compliance/       zones, scenarioMasks, maskingEngine, vesselClock
  weather/          weatherService, reroute, routeHazardDetection, types
  ai/               llama (Ollama client), generateRationale
  radar/            scores
  hooks/            useOrchestratorStream
  tools/            tools, constants
  customRoutePlanner.ts, scenarioFactory.ts, routeLookup.ts,
  store.ts, types.ts, utils.ts, dangerZones.ts
public/icons/vessel.png
```

---

## 5. Running locally

```bash
npm install
npm run dev      # http://localhost:3000
```

Production build:

```bash
npm run build
npm start
```

Optional — local rationale model (skippable; deterministic fallback runs without it):

```bash
ollama serve
ollama pull llama3.2
```

The app warms the model with `POST /api/ai/warmup` on first load. If Ollama is not running, every other feature still works and the rationale falls back to a deterministic composer.

---

## 6. Demo flow (recommended)

1. Land on `/` — the Red Sea advisory fires after ~2.5 s.
2. Click **Assess**. Watch the tool stream in the right panel.
3. Pick a route on the radar or the decision card; press `A` to accept.
4. Press `C` to enter compliance mode and watch the masked window track the vessel as it sails through Indian/EU/Gulf waters.
5. Press `2`/`3`/`4`/`5` to switch scenarios; press `R` to replay any of them.
6. Open **+ New Scenario** to construct your own voyage end to end.

---

## 7. Credits

**SeaSafe** — built for **HACKINDIA**.

License: project source for hackathon / educational use.
