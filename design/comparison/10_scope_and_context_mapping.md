# 10 — Scope & Context Mapping

Which design components map to which repo, which are out of scope, and where the
bounded-context model of the design meets the two-repo reality of the code.

Two repos are in scope:

- `sikker-selvbetjening/` — the **base/container image** repo (builds the OS image and disk artifacts).
- `sikker-selvbetjening-config/` — the **configuration/overlay** repo (renders per-device-group overlays and builds derived images).

| Verdict | Count |
|---------|-------|
| ALIGNED | 3 |
| PARTIAL | 2 |
| CONFLICT | 0 |
| WARNING | 0 |
| NOT APPLICABLE | 8 |
| **total** | **13** |

## Context → repo mapping

The design defines three bounded contexts (image_build_design.md:9): **Local
Context** (Local Administration), **Domain Context** (OS2borgerpc), **Base
Context** (OS2basis). The code has exactly **two** repos and no third "Base
Context" project, which matches the prototype's explicit decision to drop the
Base Context (C23/C25). Mapping:

- **Domain Context** ≈ `sikker-selvbetjening/`. It owns the OS image build
  (`Containerfile`, `features/`) and pushes the base image to GHCR
  (`sikker-selvbetjening/.github/workflows/build.yml:3-8`).
- **Local Context** ≈ `sikker-selvbetjening-config/`. It owns the local
  configuration (`config/config.yml`), renders overlays
  (`playbooks/render-host-overlays.yml`), and builds derived images
  (`scripts/build-device-group-image.sh`).
- **Base Context** ≈ none; circumvented with an off-the-shelf base stream
  (C25).

The **Configuration UI** (Configuration UI node in both diagrams) is **not
present in either repo**. The only trace of it is the JSON-schema contract in
`sikker-selvbetjening/features/schemas/system_files/usr/share/sikker-selvbetjening/schemas/{schema,uischema}.json`,
whose `x-ssb-file` / `x-ssb-autogen` extensions and `downloadUrlTemplate`
pointing at GitHub raw URLs indicate the UI is a **separate "plugin" project**
outside the scope of these two repos. This is consistent with the design
treating the UI as a non-functional/simulated component for the prototype
(image_build_prototype.md:49) and as an envisioned-production component
(image_build_design.md:45, 71).

---

### C23 — ALIGNED

Claim: Pipeline Declaration moved to the Domain context for the prototype; Base Context omitted.
Design: design/image_build_prototype.md:66-68
Code: sikker-selvbetjening/Containerfile:6,13-23 (the OS image "pipeline declaration"/Containerfile lives in the Domain-Context repo sikker-selvbetjening); no Base-Context repo exists.
Finding: The OS-image build recipe lives in the Domain-Context repo. There is no Base Context repo. Matches the prototype decision.

### C24 — PARTIAL

Claim: "no Pipeline Declaration logic is contained in the Local Context".
Design: design/image_build_prototype.md:68
Code: sikker-selvbetjening-config/.github/workflows/build.yml:1-92, sikker-selvbetjening-config/scripts/build-device-group-image.sh:1-132, sikker-selvbetjening-config/playbooks/render-host-overlays.yml:1-101
Finding: The OS-image *build* pipeline (Containerfile) is correctly outside the Local Context. But the Local-Context repo itself carries substantial build/render logic (its own GH Actions workflow, the device-group build script, and the overlay-merging playbook). The principle that the Local Context owns *no* pipeline logic is only partially honoured — the core OS-build logic is external, but the derived-image build/merge logic is local. (See also C63.)

### C25 — ALIGNED

Claim: Base Context Image Declaration circumvented by using an off-the-shelf base Image Stream.
Design: design/image_build_prototype.md:69
Code: sikker-selvbetjening/Containerfile:6 (`FROM quay.io/fedora-ostree-desktops/silverblue:43`)
Finding: The base image is an off-the-shelf Fedora Silverblue stream, not a locally-built Base-Context image. Matches the design. (Minor code-internal inconsistency: README says Silverblue `:42`, Containerfile pins `:43` — not a design↔code issue.)

### C31 — PARTIAL

Claim: Ubiquitous-language definitions (Local Configuration, Image Stream, Image Build, Image Declaration, contexts, Disk Image).
Design: design/image_build_design.md:7-13
Code: sikker-selvbetjening-config/config/config.yml:1-43, sikker-selvbetjening-config/scripts/discover-device-groups.py:1-92, sikker-selvbetjening/Containerfile:6
Finding: The concepts exist but the code uses a different vocabulary: "Local Configuration" → `config/config.yml`; "Image Stream"/"Image Build" → device-group `image_name` images; "Image Declaration" → `Containerfile` + overlay payload; "Image Declaration" pieces appear as `domains`/`policies`/`device_groups`. Concepts map cleanly; the terms themselves drift.

### C6 — ALIGNED

Claim: Prototype logic/discoveries must be reusable for the production build.
Design: design/prototype_strategy.md:19
Code: sikker-selvbetjening/features/sikker-overlay-engine/ (overlay engine decoupled from any specific config), sikker-selvbetjening/features/schemas/ (schema as a published contract), sikker-selvbetjening-config/README.md (documents the schema/helper "compatibility boundary")
Finding: Reuse thinking is visible: a feature-modular Containerfile, a contract-first schema published from the base image, and an overlay engine whose README explicitly frames itself as a reusable, idempotent renderer. No contradiction.

---

## Envisioned-production components (out of scope for the prototype code)

The following claims describe components of the *envisioned production* system.
Their absence in the prototype code is expected and is recorded as
`NOT APPLICABLE — envisioned`.

### C32 — NOT APPLICABLE — envisioned

Local Administrator updates Signing Key/Secrets in the backend (image_build_design.md:49). No secrets backend or signing-key UI exists in code.

### C33 — NOT APPLICABLE — envisioned

Local Administrator updates Local Configuration in the Configuration UI (image_build_design.md:50). No Configuration UI in these repos (separate project).

### C34 — NOT APPLICABLE — envisioned

Backend fetches the Pipeline Declaration from the Base Context (image_build_design.md:53). No Base Context and no fetch step.

### C37 — NOT APPLICABLE — envisioned

Configuration UI pushes change to ConfigRepo (image_build_design.md:91). No UI in repos.

### C38 — NOT APPLICABLE — envisioned

ConfigRepo requests run of the pipeline (image_build_design.md:92). Envisioned restatement; the prototype variant (C15) is checked in doc 40.

### C41 — NOT APPLICABLE — envisioned

CI Pipeline fetches baseline from the Domain Context (image_build_design.md:96). Envisioned restatement; the prototype variant (C16) is checked in doc 40.

### C42 — NOT APPLICABLE — envisioned

CI Pipeline pushes updated Image Build to the Image Registry (image_build_design.md:97). Envisioned restatement; the prototype variant (C17) is checked in doc 40.

### C64 — NOT APPLICABLE — open-question

Who owns the Pipeline Declaration is "up for debate" (Base / Domain / separate) (image_build_design.md:187-191). The prototype design *resolved* this at the prototype level by moving the declaration to the Domain Context (C23, image_build_prototype.md:66-68) — a documented design decision, not a silent code commitment. No further code-level resolution.
