# 00 — Claim Inventory

Full list of every checkable claim extracted from the three design documents.
Each claim has an ID (`C1`–`C67`), a short paraphrase + key quote, its source
location, and a category. No analysis here — analysis lives in the topic
documents (10–90). The "Doc" column records which topic document carries the
verdict (each claim appears in exactly one).

Categories: `proto` = prototype-scope · `env` = envisioned-production ·
`omit` = deliberately-omitted · `open` = open-question.

## prototype_strategy.md

| ID | Paraphrase / quote | Source | Cat | Doc |
|----|--------------------|--------|-----|-----|
| C1 | Concern 1 — show live example of changing the local configuration *and* seeing the effect on the Client PC. | design/prototype_strategy.md:14 | proto | 30 |
| C2 | Concern 2a — show local configuration applied centrally (not via USB sticks). | design/prototype_strategy.md:15 | proto | 30 |
| C3 | Concern 2b — the configuration change "involves only little manual labor". | design/prototype_strategy.md:15 | proto | 30 |
| C4 | Concern 3 — change the local configuration via an interface the admin "feels comfortable" with. | design/prototype_strategy.md:16 | proto | 30 |
| C5 | Concern 4 — show live example of monitoring the health of the Client PCs. | design/prototype_strategy.md:17 | proto | 50 |
| C6 | Quality characteristic — prototype logic/discoveries must be reusable for the production build. | design/prototype_strategy.md:19 | proto | 10 |
| C7 | Note — user-login and desktop-apps features are "in-progress" but lack justification; "recommend evaluating the underlying hypotheses". | design/prototype_strategy.md:21 | proto | 90 |
| C8 | Tactical — "Code execution during the Container built is currently managed through ansible." | design/prototype_strategy.md:37 | proto | 20 |
| C9 | Tactical — ansible must be usable "in a way that it can later be seamlessly removed" and the decision "documented carefully". | design/prototype_strategy.md:37 | proto | 20 |
| C10 | Tactical — config repo "exposes a lot of parameters that are not relevant to the intended user". | design/prototype_strategy.md:38 | proto | 30 |
| C11 | Tactical — keep config-repo/interface cleanup at a lower priority for now. | design/prototype_strategy.md:38 | proto | 30 |

## image_build_prototype.md

| ID | Paraphrase / quote | Source | Cat | Doc |
|----|--------------------|--------|-----|-----|
| C12 | Prototype edge (1) — PrototypePresenter updates Local Configuration in the Configuration UI. | design/image_build_prototype.md:29 | proto | 30 |
| C13 | Prototype edge (2) — push the change to the Local Configuration Backend (manually simulated). | design/image_build_prototype.md:30 | proto | 30 |
| C14 | Prototype edge (3) — ConfigRepo "fetches" the Pipeline Declaration. | design/image_build_prototype.md:32 | proto | 20 |
| C15 | Prototype edge (4) — ConfigRepo "requests run of pipeline" with the CI Pipeline Runner. | design/image_build_prototype.md:33 | proto | 40 |
| C16 | Prototype edge (5) — CI Pipeline "fetches baseline from" the Domain Image Build. | design/image_build_prototype.md:35 | proto | 40 |
| C17 | Prototype edge (6) — CI Pipeline "pushes updated Image Build to" the Image Registry. | design/image_build_prototype.md:36 | proto | 40 |
| C18 | Prototype edge — Client PC "checks for signed & updated Image Build in" the Image Registry. | design/image_build_prototype.md:38 | proto | 50 |
| C19 | Note 1 — Configuration UI is non-functional; backend action is simulated by manually pushing changes. | design/image_build_prototype.md:49 | proto | 30 |
| C20 | Note 2 — possible to install the OS without a Disk Image via `bootc switch` / `bootc update --apply`. | design/image_build_prototype.md:57 | proto | 60 |
| C21 | Note 2 — an ISO build is necessary only if a seamless install must be shown and `bootc update` is unacceptable. | design/image_build_prototype.md:58 | proto | 60 |
| C22 | Note 2 — current assumption: "a mitigation using a different Disk Image is acceptable" (i.e. do not build own disk images). | design/image_build_prototype.md:61 | proto | 60 |
| C23 | Note 3 — Pipeline Declaration moved to the Domain context for the prototype; Base Context omitted. | design/image_build_prototype.md:66-68 | proto | 10 |
| C24 | Note 3 — "no Pipeline Declaration logic is contained in the Local Context". | design/image_build_prototype.md:68 | proto | 10 |
| C25 | Note 3 — Base Context Image Declaration is circumvented by using an off-the-shelf base Image Stream. | design/image_build_prototype.md:69 | proto | 10 |
| C26 | Note 4 — Observability Panel is part of the prototype but only simulated (a "dummy", not connected). | design/image_build_prototype.md:79 | proto | 50 |
| C27 | Other omitted — Image Signing. | design/image_build_prototype.md:83 | omit | 70 |
| C28 | Other omitted — Secret Management. | design/image_build_prototype.md:83 | omit | 70 |
| C29 | Other omitted — SBOM Generation. | design/image_build_prototype.md:83 | omit | 70 |
| C30 | Other omitted — Scheduled Rebuilds. | design/image_build_prototype.md:83 | omit | 70 |

## image_build_design.md

| ID | Paraphrase / quote | Source | Cat | Doc |
|----|--------------------|--------|-----|-----|
| C31 | Ubiquitous-language definitions (Local Configuration, Image Stream, Image Build, Image Declaration, contexts, Disk Image). | design/image_build_design.md:7-13 | proto | 10 |
| C32 | Env edge (1) — Local Administrator updates Signing Key/Secrets in the backend. | design/image_build_design.md:49 | env | 10 |
| C33 | Env edge (2) — Local Administrator updates Local Configuration in the Configuration UI. | design/image_build_design.md:50 | env | 10 |
| C34 | Env edge (4) — backend fetches the Pipeline Declaration from the Base Context. | design/image_build_design.md:53 | env | 10 |
| C35 | Env edge (5) / Note 3 — backend fetches an Image Build *or* an Image Declaration from the Domain Context (which one is the open question). | design/image_build_design.md:54 | open | 40 |
| C36 | Env edge (~) — Client PC checks for a *signed* & updated Image Build. | design/image_build_design.md:56 | env | 50 |
| C37 | Env detail edge (1) — Configuration UI pushes change to ConfigRepo. | design/image_build_design.md:91 | env | 10 |
| C38 | Env detail edge (2) — ConfigRepo requests run of the pipeline. | design/image_build_design.md:92 | env | 10 |
| C39 | Env detail edge (3) — CI Pipeline fetches Pipeline Declaration from the Base Context. | design/image_build_design.md:94 | env | 20 |
| C40 | Env detail edge (4) — CI Pipeline reads Secrets. | design/image_build_design.md:95 | env | 45 |
| C41 | Env detail edge (5) — CI Pipeline fetches baseline from the Domain Context. | design/image_build_design.md:96 | env | 10 |
| C42 | Env detail edge (6) — CI Pipeline pushes updated Image Build to the Image Registry. | design/image_build_design.md:97 | env | 10 |
| C43 | Env detail edge (7) — CI Pipeline requests signing of the updated Image Build. | design/image_build_design.md:98 | env | 70 |
| C44 | Env detail edge (8) — CI Pipeline generates an SBOM. | design/image_build_design.md:99 | env | 70 |
| C45 | Env detail edge (9) — CI Pipeline generates an OS Disk Image. | design/image_build_design.md:100 | env | 60 |
| C46 | Note 3 (open question) — build path (1) each context builds its own images vs (2) Local Context builds them; "stay uncommittal towards the approach". | design/image_build_design.md:118-125 | open | 40 |
| C47 | Note 4 — Client PC "regularly checks the Image Stream for new versions". | design/image_build_design.md:131 | proto | 50 |
| C48 | Note 4 — suggested mitigation: pull updates at high frequency without applying, reboot to apply. | design/image_build_design.md:132 | proto | 50 |
| C49 | Note 4 — risk of manual `bootc rollback`; optional mechanism to check network health on boot and auto-rollback. | design/image_build_design.md:133 | proto | 50 |
| C50 | Note 5 — Local Configuration stored as a text file in a VCS. | design/image_build_design.md:139 | proto | 30 |
| C51 | Note 5 (open question) — auth between Configuration UI and ConfigRepo; a generic "system" account is a no-go. | design/image_build_design.md:140 | open | 30 |
| C52 | Note 5 — "1:1 mapping between Local Configurations and their associated Image Streams … for every Local Configuration, there is a separate repository". | design/image_build_design.md:142 | proto | 30 |
| C53 | Note 5 — consequence: "we don't need to maintain a templating logic for deriving multiple Local Configurations from a single template". | design/image_build_design.md:144 | proto | 30 |
| C54 | Note 5 (open question) — how to give Local Admins a test/staging system. | design/image_build_design.md:147 | open | 30 |
| C55 | Note 6 — the CI Pipeline Runner is part of the Local context (image lifetime management is the Local Administration's concern). | design/image_build_design.md:151-155 | proto | 20 |
| C56 | Note 6 — a new Image Build is created with every Local Configuration change (vs client-side config apply). | design/image_build_design.md:157-164 | proto | 40 |
| C57 | Note 6 — Image Build also runs "on a regular interval, in order to fetch the latest updates provided through the Domain Context". | design/image_build_design.md:165 | proto | 40 |
| C58 | Note 7 — model is "agnostic towards how the signing key or other secrets are actually stored and mananged". | design/image_build_design.md:171 | env | 45 |
| C59 | Note 7 (open question) — "we want to not commit on a specific secrets storage implementation". | design/image_build_design.md:172 | open | 45 |
| C60 | Note 7 — "we should therefore try to not put any secret information onto the Image Builds" (e.g. a wifi password). | design/image_build_design.md:173 | proto | 45 |
| C61 | Note 7 — hypothesis: secrets entered on-demand by the user; in a public-PC context no secrets are needed. | design/image_build_design.md:174 | proto | 45 |
| C62 | Note 8 — Disk Image Store need not be separate software; VCS "release" attachments are acceptable. | design/image_build_design.md:179 | env | 60 |
| C63 | Note 9 — Pipeline Declaration is a separate component outside the Local Context; "the Local Context does not own any pipeline logic". | design/image_build_design.md:183-185 | proto | 20 |
| C64 | Note 9 (open question) — who owns the Pipeline Declaration is "up for debate" (Base / Domain / separate). | design/image_build_design.md:187-191 | open | 10 |
| C65 | Note 9 — Pipeline Declaration should be in a portable, interoperable format; avoid CI-system-specific modules. | design/image_build_design.md:194 | proto | 20 |
| C66 | Note 9 (open question) — how to ensure the Pipeline Declaration is authentic. | design/image_build_design.md:195 | open | 20 |
| C67 | Note 10 — Observability Panel: a non-technical interface showing machine health and rollout status. | design/image_build_design.md:199 | env | 50 |

## Counts by category

- prototype-scope: 33 · envisioned: 18 · deliberately-omitted: 4 · open-question: 9 · (C7 is a meta-note on features) = 67 claims total (incl. C7).

## Coverage check

Every claim C1–C67 is assigned to exactly one topic document in the "Doc" column.
Topic documents: 10, 20, 30, 40, 45, 50, 60, 70, 90.
