# slsa-controls

Working draft of the AppSec software supply chain control: ingress to production.

The control is a single self-contained HTML page at [`docs/index.html`](docs/index.html).
It has no build step and no dependencies — open it in a browser, or serve it via
GitHub Pages from the `/docs` folder.

## Layers

The page is organised as switchable layers. Each is a tab at the top of the page,
and deep-links with a fragment (e.g. `index.html#javaroute`).

| Layer | Covers |
| --- | --- |
| Control overview | Stages, pillars, seams, SLSA spine, CVE clock, exploitability, waivers, promotion paths |
| FOSS overview | SBOM of record, context matrix, licence scanning, ecosystem comparison |
| Java / Python / npm / Go | Per-ecosystem lifecycle, risks and opening moves |
| Containers | Base images, digest pinning, base image currency clock |
| Vendor binaries | Contract levers, containment controls, auto-update risk |
| Freeware & shareware | Catalogue, build-agent exposure, licence traps |
| Source & forks | Harvested source, fork debt, contributing back |
| SDKs, IDEs & plugins | Toolchain as build input, desktop SDK, extension risk |
| Java route (practical) | Current state, target state and sequence for the Java estate |
| Golden path journey | A CVSS 9+ dependency traced end to end through the control |
| Risk findings | Treatment model: golden path, detection, mitigation, MVP across horizons |

## Status

Draft for review. Control IDs are placeholders pending mapping to the AppSec
control library. SLSA references are to v1.2 (approved 24 November 2025).
Platform level claims are vendor-asserted rather than independently certified.

## History

Each commit corresponds to a document version, oldest first. `git log --oneline`
gives the change summary; `git diff <sha>~ <sha> -- docs/index.html` shows what a
given version added.

## Editing

The page is hand-written HTML with inline CSS and a small layer-switching script.
To add a layer, add a `<button role="tab" data-layer="NAME">` to the nav and a
matching `<div class="layer" id="layer-NAME" data-layer="NAME" hidden>` below.
The script picks up anything with matching `data-layer` attributes — no JS changes
needed.
