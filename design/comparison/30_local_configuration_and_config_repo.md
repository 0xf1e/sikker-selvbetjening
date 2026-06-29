# 30 — Local Configuration & Config Repo

How the prototype's "change local configuration centrally and see the effect"
story, the (non-)Configuration UI, and the "one repository per Local
Configuration" decision hold up against the config repo.

| Verdict | Count |
|---------|-------|
| ALIGNED | 7 |
| PARTIAL | 2 |
| CONFLICT | 2 |
| WARNING | 0 |
| NOT APPLICABLE | 3 |
| **total** | **14** |

All 14 claims assigned to this document (C1–C4, C10–C13, C19, C50–C54) are analysed below.

---

### C1 — ALIGNED

Claim (Concern 1): show a live example of changing the local configuration and seeing the effect on the Client PC.
Design: design/prototype_strategy.md:14
Code: sikker-selvbetjening-config/config/config.yml:3-35 (editable policies: desktop background, printer, wifi, favorite apps, firefox urls, mesh agent), sikker-selvbetjening-config/playbooks/render-host-overlays.yml:75-101 (renders a normalized overlay per device group), sikker-selvbetjening/features/{background-image,favorite-apps,wifi,auto-browser}/system_files/usr/libexec/sikker-overlay/tasks/*.yml (overlay → concrete dconf/NetworkManager/script changes on the Client PC)
Finding: A change in `config/config.yml` flows through render → overlay → concrete filesystem changes that are visible on the Client PC (background image, favorite apps, wifi profile, browser start URLs). The end-to-end "change config → see effect" path exists.

### C2 — ALIGNED

Claim (Concern 2a): show local configuration applied centrally (not via USB sticks).
Design: design/prototype_strategy.md:15
Code: sikker-selvbetjening-config/.github/workflows/build.yml:1-92 (central CI builds derived images), sikker-selvbetjening-config/scripts/build-device-group-image.sh:116-131 (pushes to GHCR), sikker-selvbetjening/features/bootc-updates/build_files/20-services.sh:1-42 (client pulls updates from the registry)
Finding: Configuration is delivered through centrally-built images published to a registry and pulled by the client. No USB-stick distribution path is involved.

### C3 — PARTIAL

Claim (Concern 2b): the configuration change "involves only little manual labor".
Design: design/prototype_strategy.md:15
Code: sikker-selvbetjening-config/.github/workflows/build.yml:3-4 (`on: workflow_dispatch:` — builds are triggered *manually*), image_build_prototype.md:49 (design itself states the ConfigUI backend action is "simulated by manually pushing the changes")
Finding: Once triggered, the build is fully automated. But the trigger is manual (`workflow_dispatch` only — see C56), and the design's own Note 1 concedes the UI backend action is simulated by a manual push. For the prototype this is acceptable; it is not yet the low-manual-labor production flow.

### C4 — PARTIAL

Claim (Concern 3): change the local configuration via an interface the admin "feels comfortable" with.
Design: design/prototype_strategy.md:16
Code: sikker-selvbetjening/features/schemas/system_files/usr/share/sikker-selvbetjening/schemas/schema.json:23-54 (`x-ssb-file` with `downloadUrlTemplate: https://raw.githubusercontent.com/{owner}/{repo}/{branch}/{path}` — points at a separate GitHub-hosted UI), schema.json:95 (`x-ssb-autogen: policy-id` custom extension), uischema.json:1-65 (a json-schema-form `ui:` widget/disabled hint contract) — **no consuming UI exists in either in-scope repo**
Finding: No Configuration UI exists in either in-scope repo. A schema/uischema contract is published from the base image, implying the actual UI is a separate "plugin" project outside scope. The prototype design explicitly makes the UI non-functional/simulated (C19), so the absence is consistent with the prototype — but the *concern* ("comfortable interface") is not met by code in these repos.

### C10 — ALIGNED

Claim: The config repo "exposes a lot of parameters that are not relevant to the intended user".
Design: design/prototype_strategy.md:38
Code: sikker-selvbetjening-config/config/config.yml:1-43 (exposes `id`, `image_name`, `domain`, device-group `id`/mesh ids, `mesh_agent.groupid`), sikker-selvbetjening/features/schemas/.../schema.json:90-95 (`policy id` with `x-ssb-autogen: policy-id`), :232-234 (`device_group id` accepts arbitrary mesh ids), :262-267 (`image_name` identifier-safe key)
Finding: The design's observation holds: the config exposes structural/automation fields (`id`, `image_name`, mesh ids) that a local IT administrator would not care about. The code matches the described state.

### C11 — ALIGNED

Claim: Keep the config-repo/interface cleanup at a lower priority for now.
Design: design/prototype_strategy.md:38
Code: sikker-selvbetjening-config/config/config.yml:1-43 (unchanged — the irrelevant parameters are still exposed), sikker-selvbetjening-config/README.md (documents the schema/helper boundary but does not add a presentation layer)
Finding: The recommendation to defer cleanup was followed — the code was not refactored to hide irrelevant parameters.

### C12 — NOT APPLICABLE — prototype-simulated

Prototype edge (1): PrototypePresenter updates Local Configuration in the Configuration UI (image_build_prototype.md:29). No UI exists in code; the prototype design states it is simulated. See C19.

### C13 — ALIGNED

Claim: Prototype edge (2) — push the change to the Local Configuration Backend (manually simulated).
Design: design/image_build_prototype.md:30
Code: sikker-selvbetjening-config/config/config.yml (a normal file in a git repo — changes are committed/pushed manually)
Finding: The Local Configuration Backend is a git repository; changes are pushed to it, consistent with the "simulate by manually pushing" model.

### C19 — ALIGNED

Claim (Note 1): The Configuration UI is non-functional; its backend action is simulated by manually pushing changes.
Design: design/image_build_prototype.md:49
Code: (no UI in repos; `config/config.yml` edited and pushed directly), sikker-selvbetjening/features/schemas/.../uischema.json (UI schema present but no consuming UI in scope)
Finding: Matches exactly — there is no functional UI; configuration is changed by editing and pushing the config file. The "manual push simulation" is literally how it works.

### C50 — ALIGNED

Claim: Local Configuration stored as a text file in a VCS.
Design: design/image_build_design.md:139
Code: sikker-selvbetjening-config/config/config.yml (YAML text file under git; `.git/` present)
Finding: The Local Configuration is a text file in a git repository. Matches.

### C51 — NOT APPLICABLE — open-question

Auth between Configuration UI and ConfigRepo; a generic "system" account is a no-go (image_build_design.md:140). No Configuration UI exists in code, so the auth question is neither resolved nor violated here.

### C52 — CONFLICT

Claim: "1:1 mapping between Local Configurations and their associated Image Streams … for every Local Configuration, there is a separate repository".
Design: design/image_build_design.md:142
Code: sikker-selvbetjening-config/config/config.yml:1-43 (a **single** repo holds one `domains` list able to contain many domains, each with many `policies` and many `device_groups`), sikker-selvbetjening-config/scripts/discover-device-groups.py:54-86 (iterates *all* domains × device_groups and emits a **matrix** of multiple images from this one repo), sikker-selvbetjening-config/.github/workflows/build.yml:74-92 (matrix build of many device-group images from one repo)
Finding: The design **decided** on one repository per Local Configuration (per Image Stream). The code uses **one repository that produces many image streams** — `discover-device-groups.py` walks every domain/device-group and the CI builds each as a separate matrix entry (`image_name`). This is a 1-repo-to-N-image-streams architecture, directly contradicting the 1:1 decision. (Today only the `demo`/`Nordborg` group is populated, but the structure and discovery logic are built for many.)

### C53 — CONFLICT

Claim: Consequence of 1:1 — "we don't need to maintain a templating logic for deriving multiple Local Configurations from a single template".
Design: design/image_build_design.md:144
Code: sikker-selvbetjening-config/playbooks/render-host-overlays.yml:66-86 (builds a `policies_by_name` lookup and **merges** an ordered policy list with `combine(..., recursive=True)` to derive a combined configuration per device group), sikker-selvbetjening-config/templates/section.yml.j2 (a reusable template fragment), sikker-selvbetjening-config/scripts/discover-device-groups.py:54-86 (derives multiple image builds from shared policies)
Finding: The exact templating/derivation logic the design said 1:1 would *avoid* is present: device groups compose multiple reusable `policies` (a templating mechanism) into derived configurations. This is the direct downstream consequence of breaking the 1:1 decision (C52).

### C54 — NOT APPLICABLE — open-question

How to give Local Admins a test/staging system (image_build_design.md:147). No staging mechanism exists in code; the question is unresolved (and the 1-to-N repo model in C52 would actually enable per-device-group staging, but none is implemented).
