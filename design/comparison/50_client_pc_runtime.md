# 50 — Client PC Runtime

bootc self-update behaviour, the "signed & updated" check, the
simulated-only observability story, and the optional rollback mitigation.

| Verdict | Count |
|---------|-------|
| ALIGNED | 2 |
| PARTIAL | 3 |
| CONFLICT | 0 |
| WARNING | 0 |
| NOT APPLICABLE | 3 |
| **total** | **8** |

---

### C5 — PARTIAL

Claim (Concern 4): show a live example of monitoring the health of the Client PCs.
Design: design/prototype_strategy.md:17
Code: sikker-selvbetjening/features/mesh-agent/ (MeshCentral remote-management agent — real remote access, not a health dashboard), sikker-selvbetjening/features/bootc-updates/build_files/20-services.sh:1-42 (bootc update status), (no observability panel/dashboard in either repo)
Finding: No health/observability dashboard exists in code. There *is* a real remote-management channel (the MeshCentral agent) and bootc exposes update status, but neither is the "monitor health of Client PCs" panel the concern describes. The design itself says the panel is simulated for the prototype (C26), so the absence is expected — but Concern 4 is only demonstrable via a hand-simulated panel, not via code in these repos.

### C18 — PARTIAL

Claim: Client PC "checks for signed & updated Image Build in" the Image Registry.
Design: design/image_build_prototype.md:38
Code: sikker-selvbetjening/features/bootc-updates/build_files/20-services.sh:14-19 (`ExecStart=… bootc upgrade --check && bootc upgrade --apply --soft-reboot=auto`), :25-31 (timer), (no signature verification — see C27)
Finding: The "checks for … updated Image Build" part is implemented (bootc checks the registry). The "**signed**" part is **not** — image signing is on the omission list (C27) and bootc here performs no signature verification. Half the claim is met.

### C47 — ALIGNED (with a documentation defect)

Claim: The Client PC "regularly checks the Image Stream for new versions".
Design: design/image_build_design.md:131
Code: sikker-selvbetjening/features/bootc-updates/build_files/20-services.sh:31 (`OnCalendar=*-*-* 02:00:00`), :32 (`Persistent=true`)
Finding: The client checks regularly — nightly at 02:00, with `Persistent=true` to catch missed runs. That satisfies "regularly checks." (Documentation defect worth flagging: `sikker-selvbetjening/README.md:77` claims the timer "checks every 5 minutes and applies updates," which contradicts the actual `OnCalendar=*-*-* 02:00:00`. The code is nightly, not 5-minute. This is a code-internal README↔code mismatch, not a design↔code one, but it directly bears on this claim.)

### C48 — PARTIAL

Claim (suggested mitigation): have the Client PC "pull new updates at high frequency, without applying them right away" so a reboot triggers the update.
Design: design/image_build_design.md:132
Code: sikker-selvbetjening/features/bootc-updates/build_files/20-services.sh:14-19,31 (the timer runs once nightly and the service does `--check && --apply --soft-reboot=auto` in one shot — it does *not* pull frequently without applying)
Finding: The design's suggested "pull frequently, defer apply, reboot to apply" mitigation is **not** implemented. The code chose a different cadence/strategy: nightly check-and-apply with a soft reboot. The self-update capability exists, but the specific staged-apply mitigation does not.

### C49 — NOT APPLICABLE — open-question

Claim: Risk of manual `bootc rollback`; "(If needed, a mechanism can be developed that checks for network health on boot and automatically rolls back)."
Design: design/image_build_design.md:133
Code: (no network-health-on-boot auto-rollback mechanism; sikker-selvbetjening/features/reboot/system_files/usr/libexec/reboot-handler.sh is an unrelated "nuclear" force-reboot, not a health-gated rollback)
Finding: The optional auto-rollback mechanism was explicitly future/conditional ("if needed, a mechanism can be developed"). It is absent, as expected for an open optional item. The underlying risk (manual `bootc rollback` if a network-breaking update is pushed) stands and is not mitigated by code.

### C26 — ALIGNED

Claim: Observability Panel is part of the prototype but only simulated ("dummy", not connected).
Design: design/image_build_prototype.md:79
Code: (no observability panel code in either repo)
Finding: Matches — there is no real observability panel in code, consistent with the prototype decision to simulate it during the presentation.

### C36 — NOT APPLICABLE — envisioned

Client PC checks for a *signed* & updated Image Build (envisioned coarse diagram, image_build_design.md:56). The "signed" aspect is envisioned/omitted; the prototype self-update variant is C18.

### C67 — NOT APPLICABLE — envisioned

Observability Panel: a non-technical interface showing machine health and rollout status (image_build_design.md:199). Envisioned-production component; absent in code, consistent with C26's simulated treatment.
