# 45 — Secrets In Image

A dedicated document for the strongest conflict cluster in this comparison:
the design explicitly warns against putting secret information (e.g. a wifi
password) into Image Builds, and the code does exactly that.

| Verdict | Count |
|---------|-------|
| ALIGNED | 0 |
| PARTIAL | 0 |
| CONFLICT | 2 |
| WARNING | 1 |
| NOT APPLICABLE | 2 |
| **total** | **5** |

---

### C60 — CONFLICT

Claim: "As far as feasible, we should therefore try to not put any secret information onto the Image Builds, i.e. treat them as if they could just as well be distributed publicly" — with a wifi password given as the example of sensitive data to avoid.
Design: design/image_build_design.md:173
Code: sikker-selvbetjening/features/wifi/system_files/usr/libexec/sikker-overlay/tasks/wifi.yml:45 (`dest: "{{ output_root }}/etc/NetworkManager/system-connections/{{ wifi.ssid }}.nmconnection"`), wifi.yml:59-63 (writes `key-mgmt=wpa-psk` and `psk={{ wifi.psk }}` into that file), sikker-selvbetjening-config/scripts/build-device-group-image.sh:116-119 (the staged `etc/` tree is `COPY`'d into the derived image via `COPY build/${IMAGE_NAME}/etc/ /etc/`)
Finding: The wifi pre-shared key is rendered into a NetworkManager connection file inside the staged overlay tree and then baked into the derived container image. This is precisely the case the design names — "sensitive information, like a wifi password" becoming "part of an Image Build" — which the design says to avoid so the image need not be treated as sensitive. The schema even flags this: `schema.json:175-178` describes `psk` as "sensitive data [that] should eventually be replaced by a runtime secret reference." It is not yet replaced.

### C61 — CONFLICT

Claim: "We assume that entering [secrets] is only needed in situations where it is acceptable to expect the Client PC user to enter a secret (like a secret wifi password) on demand … in a public PC context, no secrets need to be entered."
Design: design/image_build_design.md:174
Code: sikker-selvbetjening/features/wifi/system_files/usr/libexec/sikker-overlay/tasks/wifi.yml:59-63 (psk baked at build time, no on-demand entry), sikker-selvbetjening/features/mesh-agent/system_files/usr/libexec/sikker-overlay/tasks/mesh-agent-enable.yml:29-35 (writes `MESH_AGENT_GROUPID` into `/etc/meshagent/meshagent.env`, baked into the image), sikker-selvbetjening-config/config/config.yml:30-31 (demo config carries a `mesh_agent.groupid` token)
Finding: The design's hypothesis is that public-PC secrets are entered on demand by the user and that no secrets are needed in the public-PC context. The code pre-bakes the wifi psk (and the MeshCentral enrollment group id) into the image at build time rather than collecting them on demand. This contradicts the on-demand hypothesis.

### C59 — WARNING (code resolved an open question)

Claim (Open Question): "we want to not commit on a specific secrets storage implementation or secrets management design pattern."
Design: design/image_build_design.md:172
Code: sikker-selvbetjening/features/wifi/.../wifi.yml:59-63 (psk stored inline in the image), sikker-selvbetjening/features/mesh-agent/.../mesh-agent-enable.yml:29-35 (groupid stored inline), sikker-selvbetjening-config/config/config.yml:25-31 (secrets carried as plain YAML values), (no secrets manager / vault / runtime-secret reference anywhere in either repo)
Finding: The design deliberately declined to choose a secrets-storage implementation. The code has effectively chosen one: **no secrets store — secrets are carried as plain values in the Local Configuration and baked into the image.** This silently resolves the open question in the simplest (and least secure) direction, exactly the kind of code-level commitment the comparison is meant to surface.

### C58 — NOT APPLICABLE — envisioned

Claim: The model is "agnostic towards how the signing key or other secrets are actually stored and mananged."
Design: design/image_build_design.md:171
Code: (no signing key is stored or used — see C27; the only "secrets" are the inline wifi psk / mesh groupid above)
Finding: An envisioned-production principle. At the prototype level it is not violated by *having* a storage choice (C59), but the agnosticism is undermined by the inline approach — recorded as a WARNING under C59 rather than double-counted here.

### C40 — NOT APPLICABLE — envisioned

Claim: CI Pipeline reads Secrets (envisioned detail edge).
Design: design/image_build_design.md:95
Code: sikker-selvbetjening-config/.github/workflows/build.yml:1-92 (the CI reads no external secret store; the only build-time secrets used are GitHub Actions defaults like `secrets.GITHUB_TOKEN` for registry login, and `secrets.S3_*` in the *base* repo's disk workflow)
Finding: The envisioned "CI reads Secrets" edge is not implemented as a secrets-backend read. Secrets enter the image through the inline overlay values (C60/C61), not through a build-time secret fetch.
