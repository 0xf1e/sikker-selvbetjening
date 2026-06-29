# 70 — Deliberate Omissions

The prototype doc (image_build_prototype.md:81-85) deliberately defers four
sub-domains: **Image Signing, Secret Management, SBOM Generation, Scheduled
Rebuilds**. Each must be verified absent. Each absence here is expected
(`NOT APPLICABLE — deliberately-omitted`); a *present* implementation would be
a notable finding.

| Verdict | Count |
|---------|-------|
| ALIGNED | 0 |
| PARTIAL | 0 |
| CONFLICT | 0 |
| WARNING | 0 |
| NOT APPLICABLE (deliberately-omitted) | 6 |
| **total** | **6** |

---

### C27 — NOT APPLICABLE — deliberately-omitted (with a leftover hint)

Claim omitted: Image Signing.
Design: design/image_build_prototype.md:83
Code: sikker-selvbetjening/.github/workflows/build.yml:66-81 (build then push — **no** `cosign sign` step), sikker-selvbetjening/.github/workflows/build-disk.yml:1-92 (no signing), sikker-selvbetjening-config/scripts/build-device-group-image.sh:116-131 (tag/push only — no signing)
Finding: Image signing is absent from both build pipelines, as expected. **Notable hint:** `sikker-selvbetjening/.gitignore:1` lists `cosign.key`, indicating a signing key was at some point scaffolded (likely a leftover from the ublue bootc template), but no signing logic actually runs. Not a violation — signing is correctly omitted — but the stale `cosign.key` entry is worth noting as drift between the omission and the repository skeleton.

### C43 — NOT APPLICABLE — deliberately-omitted

Claim omitted (envisioned edge): CI Pipeline requests signing of the updated Image Build.
Design: design/image_build_design.md:98
Code: (no signing request in either workflow — see C27)
Finding: Absent, as expected for the prototype.

### C44 — NOT APPLICABLE — deliberately-omitted

Claim omitted (envisioned edge): CI Pipeline generates an SBOM.
Design: design/image_build_design.md:99
Code: sikker-selvbetjening/.github/workflows/build.yml:1-92, sikker-selvbetjening-config/.github/workflows/build.yml:1-92 (no `syft`/`spdx`/SBOM-generation step in either pipeline)
Finding: SBOM generation is absent, as expected.

### C29 — NOT APPLICABLE — deliberately-omitted

Claim omitted: SBOM Generation.
Design: design/image_build_prototype.md:83
Code: (no SBOM tooling anywhere — see C44)
Finding: Absent, as expected.

### C28 — NOT APPLICABLE — deliberately-omitted

Claim omitted: Secret Management.
Design: design/image_build_prototype.md:83
Code: (no secrets-manager/vault/runtime-secret mechanism in either repo)
Finding: No Secret Management *subsystem* exists, as expected. **Important nuance:** while no secrets-management component is present, secrets *are* handled — by being baked inline into images (wifi psk, mesh groupid). That is a separate, substantive conflict documented in doc 45 (C59/C60/C61), not a violation of this omission. The omission is about a *dedicated secret-management sub-domain*; that is correctly absent.

### C30 — NOT APPLICABLE — deliberately-omitted

Claim omitted: Scheduled Rebuilds.
Design: design/image_build_prototype.md:83
Code: sikker-selvbetjening-config/.github/workflows/build.yml:3-4 (`on: workflow_dispatch:` — no `schedule:` and no `push:` trigger)
Finding: No scheduled rebuild exists, as expected. (Ties to C57 in doc 40: the design's Note 6 mentions regular-interval rebuilds as a system property, but the prototype defers them, and the code follows the prototype-level omission. The derived-image build is *only* manual today.)

---

## Omission summary

| Sub-domain | Omitted? | Notes |
|------------|----------|-------|
| Image Signing | ✅ absent | stale `cosign.key` in `.gitignore` (C27) |
| Secret Management subsystem | ✅ absent | but secrets are baked inline (doc 45) |
| SBOM Generation | ✅ absent | — |
| Scheduled Rebuilds | ✅ absent | derived build is `workflow_dispatch` only (C30/C57) |

All four deliberately-omitted sub-domains are correctly absent from the build
pipelines. The only drift is the stale `cosign.key` gitignore entry and the
inline-secrets behaviour (the latter is a genuine conflict, handled in doc 45).
