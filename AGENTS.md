# AGENTS.md

Context for continuing work on this repo in a new session.

## What this is

A working draft of HSBC's AppSec software supply chain control, covering software
from ingress to production. It exists as a single self-contained HTML page at
`docs/index.html`, organised into switchable layers.

Audience: engineering leadership, AppSec, and eventually governance and regulators.
It is a control document, not marketing — claims should be defensible and gaps
stated plainly.

## The artefact

**One file, no build step.** `docs/index.html` contains all markup, CSS and JS
inline. Constraints that must hold:

- No external dependencies except Google Fonts (IBM Plex Sans + IBM Plex Mono).
- No frameworks, no bundler, no npm. Hand-written HTML.
- Must render correctly in light and dark mode — colours are CSS custom properties
  redefined under `prefers-color-scheme` and `[data-theme]`.
- Must work on mobile: safe-area insets declared, wide tables wrapped in
  `.scroller` (`overflow-x: auto`), grids collapse under media queries.
- `localStorage` used only to remember the selected layer, wrapped in try/catch.

It is published as a Claude artifact at
`https://claude.ai/artifact/ENJSEyMRFCUkUJyTbahy4C` — republishing to that URL
updates the same page rather than creating a new one.

## Layer architecture

Each layer is a tab plus a panel. The switcher script picks up any element with a
matching `data-layer` attribute, so adding a layer needs no JS change:

```html
<button role="tab" id="tab-NAME" data-layer="NAME" aria-controls="layer-NAME" aria-selected="false">Label</button>
<div class="layer" id="layer-NAME" data-layer="NAME" role="tabpanel" aria-labelledby="tab-NAME" hidden>…</div>
```

Layers deep-link by fragment (`#javaroute`). A disabled "Stages (planned)" tab is a
placeholder — see Planned work.

Current layers: `base`, `foss`, `java`, `python`, `npm`, `go`, `containers`,
`vendor`, `freeware`, `source`, `desktop`, `javaroute`, `journey`, `findings`.

### After any edit, verify

- Tab ids and panel ids match (`sorted(tabs) == sorted(panels)`).
- `<section` opens equal `</section>` closes; same for `<table>`.
- The version pill in the nav and the version note in the footer are both updated.

## Component vocabulary

Reuse these rather than inventing new ones:

| Class | Use |
| --- | --- |
| `.scroller` + `<table>` | Any table. Always wrap — tables have `min-width` and must not break mobile layout. |
| `.base` | Grid of short concept cards (h3 + p). The general-purpose block. |
| `.seams` / `.seamcard` | Risks, gaps and open questions. Has an `.id` chip, `.owner` line for the fix, optional `.threat` line. |
| `.eco` / `.ecocard` | Comparison of parallel things with the same attributes (ecosystems, populations). Uses `<dl>` plus a `.risk` footer. |
| `.clock` / `.clock-row` | Ordered or time-based sequences. Left label, right explanation. |
| `.treat` | The 4×3 findings grid (treatments × horizons). Only used in the findings layer. |
| `.journey` / `.jstep` | The numbered vertical journey with evidence chips. Only used in the journey layer. |
| `.note` | Amber callout for the point that matters most in a section. Use sparingly — one or two per layer. |

Status colours are semantic and consistent everywhere:

- `live` / teal — in place, or achievable now
- `build` / blue — partial, planned, or in progress
- `seam` / amber — a boundary nobody owns
- `gapc` / red — a genuine gap today
- `mvp` / purple — pilot, findings layer only

## Editorial conventions

Path layers follow the same shape, which makes them comparable:

1. Intro section naming the **defining property** of that path in bold.
2. Lifecycle walk table (stage / control / notes).
3. Risks and gaps as `.seamcard`s, each with a `Fix:` line in `.owner`.
4. "First three moves" as a `.base` grid.

Writing style:

- State the failure mode, not just the control. "Fails if…" is more useful than
  "should".
- Name the one thing that matters most per section and say why.
- Prefer concrete consequence over hedged advice.
- Hedge factual claims that are genuinely uncertain: vendor SLSA level claims are
  self-asserted, regulatory mappings are interpretive, unverified platform claims
  are marked as such.
- Avoid inventing statistics or naming specific incidents unless certain.

ID prefixes in use: `S1–S5` seams (base), `F1–F5` FOSS cross-ecosystem, `J2–J4`
Java-specific, `P2–P4` Python, `N2–N4` npm, `G2–G4` Go, `C1–C6` containers,
`V1–V6` vendor, `W1–W6` freeware, `D1–D12` desktop, `J1–J6` journey design points,
`Q1–Q6` Java route open questions, `RF-01…` findings register.

## Substantive decisions the document rests on

These are the arguments the control makes. Changing one means changing several
layers, so treat them as decisions rather than prose.

1. **SLSA v1.2 is the reference version** (approved 24 November 2025). Build track
   L0–L3, Source track L1–L4. There is no Build L4. Build Environment and
   Dependency tracks are drafts and are labelled as such.
2. **The SBOM of record is generated at build time from the resolved graph**, before
   shading, bundling or minification. Manifests state intent; binaries lose detail.
   Everything else on the FOSS paths follows from this.
3. **Context is part of the policy decision.** An SBOM carries the context it was
   produced in (manifest only / lock file / build-resolved / binary only / image /
   vendored), and a clean scan in a degraded context is not equivalent to a clean
   scan in a good one.
4. **`not_affected` and a waiver are different things.** The first is a falsifiable
   engineering claim with a justification label and no expiry. The second is a risk
   decision on something genuinely affected, with scope, owner and expiry.
   Conflating them is how exception processes rot.
5. **No waivers at ingress.** There is no consumer, exposure or data class at
   ingress, so no risk decision is possible. Packages are admitted with
   `under_investigation` as a travelling obligation, discharged at the release gate
   where context exists.
6. **Promotion is the enforcement point, not deployment**, because deployment
   control is incomplete across the estate. Three promotion paths: standard,
   protected (requires a gated deployer), blocked.
7. **Location is derived state.** The attestation set decides which path an artefact
   sits in; a reconciler moves it. Movement between paths is itself attested.
8. **`builder.id` is composite** — orchestrator plus executor class. Orchestration
   does not raise the level of what it orchestrates.
9. **Build L2 means the platform generates and signs.** Three shortcuts fail it:
   signing in a pipeline step with credentials, the job composing provenance for the
   platform to sign, and the job supplying the digest.
10. **The CVE clock starts at advisory publication**, not at the next build. Because
    verification is point-in-time, VSAs carry an expiry policy.
11. **Containers have a second clock** — base image currency. OS-layer findings are
    inherited and owned by the platform; application-layer findings are owned by the
    service team. Every finding routes to one or the other.
12. **Exploitability is a decision, not a score.** CVSS, KEV, EPSS, reachability and
    exposure feed an SSVC tree that emits an action. VEX records the outcome.
13. **The control gives something back.** Evidence level unlocks entitlements:
    deployment scope, waiver self-service, emergency path speed, triage volume. This
    is deliberate — a control that only says no gets routed around.
14. **A prebuilt binary of an open-source project is on the binary path**, not the
    FOSS path. Public source does not mean the binary was built from it.
15. **Findings use four treatments across three horizons** (golden path, identify
    outside the path, general mitigations, MVP). Empty cells are the output, not a
    documentation gap.
16. **Detection before enforcement.** A gate without detection only constrains the
    people who were already compliant.

## Environment facts established

Confirmed by the user, and the practical layers depend on them:

- **Build**: Jenkins plus other build technologies, orchestrated by **DevX pipelines**.
- **Signing**: a **SLSA signing agent** already exists. Whether it *generates*
  provenance or only *signs what it is handed* is unresolved and decides the level.
- **SBOM inventory**: **Spectra**, storing SBOMs at stages. Being stood up.
- **Repository / proxy**: **Nexus**.
- **Component registry**: **ICE** — components and business apps, drives entitlement.
- **Runtime**: mixed — internal Kubernetes/OpenShift, public cloud managed services,
  VMs and app servers.
- Also referenced in the journey layer: an exploitability engine, a waiver system, a
  change record system, a release page, a deployment service, and a production
  dynamic inventory.

Assumptions still unverified: Maven and Gradle; SAST and quality gates already in the
pipelines; some SCA coverage; no artefact signing or SBOM of record in production use
today; mobile apps distributed via app stores rather than enterprise side-loading.

## Open questions

Java route (`Q1–Q6`): does the signing agent generate or sign; how are signing
callers authenticated; what proportion of builds pass through DevX; which executor
classes exist; does Spectra key to digests; who operates the verification service.

Journey (`J1–J6`): transitive scope enforcement; developer identity at the Nexus
proxy; maximum `under_investigation` window; revocation blast action for running
instances; VSA versus raw provenance at deploy; fix availability as a revocation
trigger.

Q1 and Q3 are the two that most change the plan.

## Planned work

- **Stages layer** — the disabled tab. Undecided whether stages are a third dimension
  or a filter over the existing path layers. Worth settling before building.
- More path layers as needed (mobile, mainframe, data/ML artefacts have all been
  implied but not written).
- Reconcile `RF-xx` finding IDs with the existing enterprise risk register.
- Replace placeholder control IDs with real AppSec control library references.

## Working practice

- Edit `docs/index.html` directly. Large layer additions are easiest via a Python
  script doing a string replace on `<footer>`.
- Commit one change per version, with the version note in the footer updated to match.
- The published artifact should be updated at the same URL when the page changes.
