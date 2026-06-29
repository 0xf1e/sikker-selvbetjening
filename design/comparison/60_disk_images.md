# 60 — Disk Images

Whether the prototype builds Disk Images at all, given the design's stated
assumption that Disk-Image building could be avoided.

| Verdict | Count |
|---------|-------|
| ALIGNED | 2 |
| PARTIAL | 1 |
| CONFLICT | 1 |
| WARNING | 0 |
| NOT APPLICABLE | 1 |
| **total** | **5** |

---

### C22 — CONFLICT

Claim: "At this point we work in the assumption that a mitigation using a different Disk Image is acceptable" — i.e. the prototype assumes it does *not* need to build its own Disk Images.
Design: design/image_build_prototype.md:61
Code: sikker-selvbetjening/.github/workflows/build-disk.yml:1-92 (a whole workflow that builds `anaconda-iso` disk images via `osbuild/bootc-image-builder-action`), sikker-selvbetjening/disk_config/iso-gnome.toml:1-45 (a kickstart config for unattended Anaconda install), sikker-selvbetjening/disk_config/disk.toml:1-3 (qcow2 filesystem customization), sikker-selvbetjening/Containerfile:6 (the bootc base image these disks are built from)
Finding: The design's working assumption was that the prototype would *avoid* building Disk Images and instead rely on `bootc switch`/`bootc update --apply` against an external minimal disk image. The code does the opposite: it maintains a full Anaconda-ISO (and qcow2-capable) disk-image build pipeline. This contradicts the stated prototype assumption. (The design did leave the door open — image_build_prototype.md:58, C21 — but its *current stance* at :61 was "mitigation acceptable, don't build.")

### C21 — PARTIAL

Claim: An ISO build is necessary only if stakeholders must see a seamless install *and* the `bootc update` method is unacceptable.
Design: design/image_build_prototype.md:58
Code: sikker-selvbetjening/.github/workflows/build-disk.yml:1-92 (ISO build is implemented)
Finding: The code implements the ISO build, which implicitly treats this conditional as true ("a seamless install must be shown and `bootc update` is unacceptable"). The design hedged this as a conditional possibility; the code committed to building the ISO. Not a hard contradiction (the design allowed it), but the code resolved a design conditional in the affirmative without that decision being recorded in the design prose.

### C20 — ALIGNED

Claim: It is possible to install the OS without a Disk Image, via `bootc switch` / `bootc update --apply`, starting from an external bootc disk image.
Design: design/image_build_prototype.md:57
Code: sikker-selvbetjening/features/bootc-updates/build_files/20-services.sh:14-19 (`bootc upgrade --apply --soft-reboot=auto`), sikker-selvbetjening/Containerfile:6 (a bootc image, which bootc can switch onto / apply onto), sikker-selvbetjening/README.md:75-81 (documents the bootc update-check flow)
Finding: The capability the design describes is present: the image is a bootc image and bootc's check/apply flow is wired up, so the disk-image-less install path is available. The statement (that this is *possible*) holds.

### C62 — ALIGNED

Claim: The Disk Image Store "does not need to be a separate software system" — VCS release attachments are acceptable.
Design: design/image_build_design.md:179
Code: sikker-selvbetjening/.github/workflows/build-disk.yml:91-115 (disk images are uploaded to GitHub job **artifacts** :91-99, or optionally **S3** via rclone :101-115 — no bespoke store built)
Finding: Consistent with the design's permissive note: the code reuses existing distribution channels (GH artifacts / S3) rather than building a separate Disk Image Store. No conflict.

### C45 — NOT APPLICABLE — envisioned

Claim: CI Pipeline generates an OS Disk Image (envisioned detail edge).
Design: design/image_build_design.md:100
Code: sikker-selvbetjening/.github/workflows/build-disk.yml:1-92 (disk generation lives in the **Domain-Context** base-image repo, not the Local-Context CI, and only produces the base `anaconda-iso`, not per-device-group disks)
Finding: An envisioned-production edge. At the prototype level, disk generation exists but in a different context (Domain, not Local) and only for the base image — covered by C22 above. No per-device-group disk generation exists.
