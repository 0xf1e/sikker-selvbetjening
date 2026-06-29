# 20 — Build Pipeline & Ansible

How the OS image is built, where ansible actually runs, and how the "Pipeline
Declaration" claim about portability and ownership holds up against the code.

| Verdict | Count |
|---------|-------|
| ALIGNED | 1 |
| PARTIAL | 4 |
| CONFLICT | 1 |
| WARNING | 0 |
| NOT APPLICABLE | 2 |
| **total** | **8** |

---

### C8 — PARTIAL

Claim: "Code execution during the Container built is currently managed through ansible."
Design: design/prototype_strategy.md:37
Code: sikker-selvbetjening/Containerfile:16-23 (the build loop `find … build_files/*.sh … bash "$script"` — pure bash, no ansible)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;sikker-selvbetjening/features/fedora-packages/build_files/10-packages.sh:14 (installs ansible *into* the image, does not use it to build)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;sikker-selvbetjening-config/playbooks/render-host-overlays.yml:1-101 (ansible runs here, in the config repo, at overlay-render time)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;sikker-selvbetjening/features/sikker-overlay-engine/system_files/usr/libexec/sikker-overlay/sikker-create-overlay.yml:1-26 (ansible runs inside the image at *overlay-apply* time, invoked by the config build)
Finding: Ansible is genuinely used in the system, but **not** during the Containerfile build. The Containerfile build loop executes numbered `*.sh` bash scripts (`Containerfile:16-23`). Ansible is used (a) in the config repo to merge policies/render the overlay, and (b) inside the base image's `/usr/libexec/sikker-overlay` engine when the config repo applies the overlay. The claim's placement of ansible ("during the Container build") does not match the code; the underlying sentiment ("the team uses ansible") is true, the locus is wrong.

### C9 — PARTIAL

Claim: Ansible must be usable "in a way that it can later be seamlessly removed" and the decision "documented carefully".
Design: design/prototype_strategy.md:37
Code: sikker-selvbetjening/features/sikker-overlay-engine/system_files/usr/libexec/sikker-create-overlay:21-29 (the overlay entrypoint hard-runs `ansible-playbook`), sikker-selvbetjening/features/sikker-overlay-engine/system_files/usr/libexec/sikker-overlay/tasks/*.yml (every overlay domain is an ansible task file), sikker-selvbetjening-config/playbooks/render-host-overlays.yml (config repo also depends on ansible), sikker-selvbetjening/features/sikker-overlay-engine/system_files/usr/libexec/sikker-overlay/README.md:1-60 (documents the engine but states no removal path)
Finding: Documentation exists (the overlay README and the strategy note itself), but the "seamlessly removable" goal is **not** achieved: ansible is a hard runtime dependency of the overlay engine (`sikker-create-overlay` shells out to `ansible-playbook`), every configuration domain is implemented as an ansible task file, and the config-repo render step also requires ansible. Removing ansible would require rewriting the entire overlay/render layer. One of the two requirements (documented) is partially met; the other (removable) is not.

### C14 — PARTIAL

Claim: Prototype edge (3) — ConfigRepo "fetches" the Pipeline Declaration.
Design: design/image_build_prototype.md:32
Code: sikker-selvbetjening-config/.github/workflows/build.yml:33-42 (the config repo does `podman create`/`podman cp` to extract the **schema**, not a pipeline declaration), sikker-selvbetjening-config/.github/workflows/build.yml:3-4 (the repo runs its *own* embedded workflow — no declaration is fetched from the Domain Context)
Finding: The config repo does not fetch a "Pipeline Declaration" from the Domain Context. What it *does* fetch from the base image is the JSON **schema** (`build.yml:33-42`, `BASE_IMAGE_SCHEMA_PATH`), which is a validation contract, not a pipeline. The build pipeline itself is hard-coded inside the config repo's own workflow and script. The "fetch pipeline declaration" edge is effectively unimplemented; a schema fetch happens instead.

### C39 — NOT APPLICABLE — envisioned

CI Pipeline fetches the Pipeline Declaration from the Base Context (image_build_design.md:94). Envisioned-production edge; no Base Context and no such fetch in code.

### C55 — ALIGNED

Claim: The CI Pipeline Runner is part of the Local context (image lifetime management is the Local Administration's concern).
Design: design/image_build_design.md:151-155
Code: sikker-selvbetjening-config/.github/workflows/build.yml:1-92 (the derived-image CI lives in the Local-Context repo), sikker-selvbetjening-config/scripts/build-device-group-image.sh:116-131 (local repo builds/tags/pushes derived images)
Finding: The device-group image build CI is owned by the Local-Context repo, matching the design. (The base OS image build CI lives in the Domain-Context repo, which is the domain's own concern — also consistent.)

### C63 — PARTIAL

Claim: "The Local Context does not own any pipeline logic … they should not need to modify the pipeline."
Design: design/image_build_design.md:183-185
Code: sikker-selvbetjening-config/.github/workflows/build.yml:1-92, sikker-selvbetjening-config/scripts/build-device-group-image.sh:1-132, sikker-selvbetjening-config/playbooks/render-host-overlays.yml:1-101
Finding: Same nuance as C24. The OS-image build pipeline (`Containerfile`) is correctly outside the Local Context, but the Local-Context repo carries its own image-build workflow, build script, and policy-merge playbook — pipeline-like logic that is tightly coupled to the local configuration (`config/config.yml`). The spirit of the principle is half-met.

### C65 — CONFLICT

Claim: "The Pipeline Declaration should be written in a format that can easily be transferred between CI Pipeline systems (Interoperability) … There is little to gain from using modules that are specific to one CI system."
Design: design/image_build_design.md:194
Code: sikker-selvbetjening/.github/workflows/build.yml:52-78 (`docker/metadata-action` :53, `redhat-actions/buildah-build` :68, `redhat-actions/push-to-registry` :77 — GitHub-Actions-specific), sikker-selvbetjening/.github/workflows/build-disk.yml:80-89 (`osbuild/bootc-image-builder-action@main` at :82), sikker-selvbetjening-config/.github/workflows/build.yml:1-92 (GitHub Actions workflow with `actions/checkout@v4`, matrix strategy, `GITHUB_TOKEN`/`GITHUB_SHA` intrinsics)
Finding: Both pipelines are written as GitHub Actions workflows using GitHub-specific actions and intrinsics (`GITHUB_SHA`, `GITHUB_TOKEN`, `ghcr.io`). They are **not** portable across CI systems; moving to another runner would require rewriting the workflows. This directly contradicts the interoperability design note. (The Containerfile itself is portable, but the *pipeline declaration* — the orchestration — is not.)

### C66 — NOT APPLICABLE — open-question

How to ensure the Pipeline Declaration is authentic (image_build_design.md:195). Not resolved by the code. Workflows are authenticated via GitHub's `GITHUB_TOKEN`/`id-token: write` but there is no mechanism attesting the *contents* of the pipeline declaration itself, which is what the open question asks about.
