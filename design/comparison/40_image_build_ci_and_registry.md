# 40 — Image Build, CI & Registry

The prototype's CI edges (fetch baseline, push image, request run) and the two
load-bearing findings: the **build-path open question** the code silently
resolved (C46), and the **no-on-change build trigger** (C56).

| Verdict | Count |
|---------|-------|
| ALIGNED | 3 |
| PARTIAL | 0 |
| CONFLICT | 1 |
| WARNING | 2 |
| NOT APPLICABLE | 1 |
| **total** | **7** |

---

### C15 — ALIGNED

Claim: Prototype edge (4) — ConfigRepo "requests run of pipeline" with the CI Pipeline Runner.
Design: design/image_build_prototype.md:33
Code: sikker-selvbetjening-config/.github/workflows/build.yml:3-4 (`on: workflow_dispatch:`), :13-92 (`prepare` then `build` matrix jobs)
Finding: The config repo's workflow *is* the request to run the pipeline. It is triggered manually (`workflow_dispatch`), which matches "request run" (a human/CI action), even though it is not auto-triggered on config change (see C56).

### C16 — ALIGNED

Claim: Prototype edge (5) — CI Pipeline "fetches baseline from" the Domain Image Build.
Design: design/image_build_prototype.md:35
Code: sikker-selvbetjening-config/.github/workflows/build.yml:8 (`BASE_IMAGE: ghcr.io/os2borgerpc/sikker-selvbetjening:latest`), sikker-selvbetjening-config/scripts/build-device-group-image.sh:26,89,120 (`FROM ${BASE_IMAGE}`, `podman run … ${BASE_IMAGE}`)
Finding: The derived-image build fetches the Domain-Context image (`sikker-selvbetjening:latest`) as its baseline. Matches.

### C17 — ALIGNED

Claim: Prototype edge (6) — CI Pipeline "pushes updated Image Build to" the Image Registry.
Design: design/image_build_prototype.md:36
Code: sikker-selvbetjening-config/scripts/build-device-group-image.sh:116-131 (`podman build -t … :latest`, then `podman tag`/`podman push` for `latest`, `latest.<date>`, `<date>`, `sha-<sha>`), sikker-selvbetjening-config/.github/workflows/build.yml:12 (`IMAGE_REPO: ghcr.io/os2borgerpc/sikker-selvbetjening-config`)
Finding: The CI builds the derived image and pushes it (with multiple tags) to GHCR. Matches.

---

### C35 / C46 — WARNING (code resolved an open question)

Claim (Open Question): build path — (1) each context maintains its own Image Builds and the Local Context refers to them as a base, vs (2) the Local Context fetches the Image Declaration and builds the Domain/Base images itself. Design: "we need to stay uncommittal towards the approach at this point."
Design: design/image_build_design.md:54 (edge "fetches Image Build or Image Declaration from [^3]"), design/image_build_design.md:118-125 (Note 3 open question + "stay uncommittal" at :125)
Code: sikker-selvbetjening/.github/workflows/build.yml:3-8,66-81 (the **Domain Context** builds and publishes its *own* image to GHCR), sikker-selvbetjening-config/.github/workflows/build.yml:8 (`BASE_IMAGE: ghcr.io/os2borgerpc/sikker-selvbetjening:latest`), sikker-selvbetjening-config/scripts/build-device-group-image.sh:120 (`FROM ${BASE_IMAGE}`)
Finding: The code has **committed to a specific answer**: the Domain Context maintains and publishes its own Image Build (`sikker-selvbetjening/build.yml` pushes to GHCR), and the Local Context pulls that pre-built image as `BASE_IMAGE` and layers the configuration on top (`FROM ${BASE_IMAGE}`). That is **option (1)** — "each context maintains its own Image Builds, and the local Image Build process refers to those Image Builds as a base." (The Base Context layer is circumvented with an off-the-shelf Fedora Silverblue stream per C25, so option (2) — Local building the Base/Domain images from their declarations — is *not* what the code does.) Either way, the design deliberately stayed uncommitted and the code silently chose one approach.

---

### C56 — CONFLICT

Claim: "a new Image Build created with every Local Configuration change" (core prototype behavior, contrasted with client-side config apply).
Design: design/image_build_design.md:157-164
Code: sikker-selvbetjening-config/.github/workflows/build.yml:3-4 (`on: workflow_dispatch:` — the **only** trigger; there is no `push:` trigger on `config/`)
Finding: The design makes per-change image builds a defining property of the chosen approach (vs running client software that applies config changes). The code does the opposite of "on every change": the derived-image build is **only** runnable via manual `workflow_dispatch`. A commit to `config/config.yml` produces no image. (The base OS image repo *does* build on push — `sikker-selvbetjening/.github/workflows/build.yml:3-8` — but that is the Domain Context image, not the Local Configuration image.) This contradicts the per-change build behavior.

### C57 — NOT APPLICABLE — deliberately-omitted

Claim: The Image Build also runs "on a regular interval, in order to fetch the latest updates provided through the Domain Context."
Design: design/image_build_design.md:165
Code: sikker-selvbetjening-config/.github/workflows/build.yml:3-4 (no `schedule:` trigger)
Finding: No scheduled rebuild exists in the config repo. "Scheduled Rebuilds" is on the prototype's explicit omission list (image_build_prototype.md:83, claim C30), so its absence is expected at the prototype level. (Note the internal design tension: Note 6 describes regular-interval builds as a property of the system, while the prototype doc defers them. The code follows the prototype-level omission — see C30 in doc 70.)

---

*The "Image Registry" component is present as GHCR (both repos). Signing/SBOM/Disk-image generation edges from the envisioned diagram are handled in docs 45, 60 and 70.*
