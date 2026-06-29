# Executive Summary

This report compares three design documents (`design/image_build_design.md`, `design/image_build_prototype.md`, `design/prototype_strategy.md`) with the two code repositories: `sikker-selvbetjening` (the Domain Context / base image) and `sikker-selvbetjening-config` (the Local Context / config repo). Of 67 checkable design claims: **19 ALIGNED**, **12 PARTIAL**, **7 CONFLICT**, **3 WARNING** (open questions the implementation has settled), and **26 NOT APPLICABLE** (envisioned, deliberately omitted, or unresolved). Full per-claim evidence lives in `comparison/10`–`90`.

## Where design and implementation diverge (CONFLICTs)

| # | Finding | Design says | Implementation does | Detail |
|---|---------|-------------|---------------------|--------|
| C60/C61 | Secrets in image | No secret info on builds (`design:172-173`) | psk + mesh groupid stored inline | see below / `45` |
| C22 | Disk images built | Disk Image assumed avoidable (`prototype:61`) | full iso + kickstart pipeline | `60` |
| C52 | Repo model | 1 repo : 1 image stream (`design:142`) | 1 repo → N image streams | `30` |
| C53 | Templating | 1:1 avoids templating (`design:144`) | policy merge/derive logic present | `30` |
| C65 | Portability | Pipeline must be portable (`strategy:34`) | GitHub-Actions-specific | `20` |
| C56 | Auto-rebuild | Config change should trigger build | `workflow_dispatch` only (`config/build.yml:3-4`) | `40` |

The most material of these:

**1. Secrets stored in the image.** WiFi credentials are rendered into the staged image tree and end up in the derived image. In `sikker-selvbetjening/features/wifi/.../tasks/wifi.yml:59-63` the pre-shared key is written straight into the connection file:

```
[wifi-security]
key-mgmt=wpa-psk
psk={{ wifi.psk }}
```

This `.nmconnection` lands in `/etc/NetworkManager/system-connections/<ssid>.nmconnection` inside the image. A MeshCentral group id is likewise stored inline (`sikker-selvbetjening/features/mesh-agent/.../tasks/mesh-agent-enable.yml:29-35`). Both differ from the design principle that *no secret information* be put onto Image Builds (`design/image_build_design.md:172-173`). The schema notes the WiFi psk "should eventually be replaced by a runtime secret reference" (`schema.json:175-177`), signalling the current approach is an interim placeholder rather than the intended end state.

**2. Disk images built, where the design assumed they could be avoided.** The design prototype assumed a separate Disk Image could be avoided (`design/image_build_prototype.md:61`). The implementation instead maintains a full anaconda-iso + qcow2-capable build pipeline (`.github/workflows/build-disk.yml`) with a kickstart config (`disk_config/iso-gnome.toml:1-45`), taking the conditional path the design had left open.

**3. One repo serves many image streams, rather than one.** The design chose a 1:1 mapping of repository to Image Stream (`design/image_build_design.md:142`) specifically to avoid templating logic. The implementation's single config repo produces many image streams — `scripts/discover-device-groups.py:54-86` walks every domain × device-group and emits a build matrix, and `playbooks/render-host-overlays.yml:66-86` merges an ordered policy list with `combine(..., recursive=True)` to derive a combined config per group. This is the derivation logic the 1:1 decision was intended to avoid.

**4. Pipeline is tied to GitHub Actions.** Both repos use GitHub-Actions-specific actions (`build.yml:52-78`: `docker/metadata-action`, `redhat-actions/buildah-build`, `redhat-actions/push-to-registry`; `build-disk.yml:80-89`: `osbuild/bootc-image-builder-action`), differing from the design's portability/interoperability requirement (`design/prototype_strategy.md:34`).

**5. Builds are triggered manually, not on config change.** The config repo CI runs only on `workflow_dispatch` (`sikker-selvbetjening-config/.github/workflows/build.yml:3-4`), so configuration changes do not currently auto-trigger image rebuilds.

## Open questions the implementation has settled (WARNINGs)

| Claim | Open question | Implementation's choice | Detail |
|-------|---------------|-------------------------|--------|
| C35/C46 | Who builds images? (`design:118-125`) | Option (1): Domain builds base, Local layers on top | `40` |
| C59 | How to store secrets? | None / inline (simplest, worth revisiting) | `45` |

**Build path.** The design left open *who* builds images (`design/image_build_design.md:118-125`). The implementation went with option (1): the Domain Context builds and publishes its own base image, and the Local Context pulls it and builds derived device-group images on top — `FROM ${BASE_IMAGE}` at `sikker-selvbetjening-config/scripts/build-device-group-image.sh:120`, with `BASE_IMAGE=ghcr.io/os2borgerpc/sikker-selvbetjening:latest` (`build.yml`).

**Secret storage.** The implementation settled the secret-handling question by choosing *no storage / inline values* — the simplest option, and a choice worth revisiting before production.

## Where implementation and design differ (PARTIALs)

The design recommends ansible be used "in a way that it can later be **seamlessly removed**" and that the decision be "documented carefully" (`design/prototype_strategy.md:37`). In the current implementation, ansible is woven through the config repo and overlay engine, so the "seamlessly removable" goal is not yet met.

- The config repo's overlay rendering is itself a full ansible playbook (`sikker-selvbetjening-config/playbooks/render-host-overlays.yml:15-98`: `hosts: localhost`, `connection: local`, merging policies via the `combine` filter).
- The overlay engine entrypoint depends on it — `sikker-create-overlay:23` runs `command -v ansible-playbook … || die "ansible-playbook is not installed"`, then invokes `ansible-playbook "${PLAYBOOK_FILE}"` (`:28`), which auto-discovers per-feature `tasks/*.yml` files (at least 6) all authored as ansible tasks.
- Ansible is explicitly installed into the image (`features/fedora-packages/build_files/10-packages.sh:14`), and no removal path is documented in either repo (grep for removal/migration/deprecation finds nothing; the overlay `README.md` documents architecture but not how to retire ansible).

Retiring ansible would mean rewriting the config-repo playbook, the entrypoint, and every feature task file, so the design's "seamlessly removable" target is not yet reached. (Note: the `Containerfile` build itself runs bash feature scripts, not ansible — `Containerfile:16-23` — so the build step is already ansible-free; that is benign.) Signature verification is not present (signing was a deliberate omission), so the client pulls updates it cannot currently verify. And the client checks nightly at 02:00 and applies immediately in one shot, rather than the design's suggested "pull frequently without applying" mitigation:

```
ExecStart=/usr/bin/bash -c '/usr/bin/bootc upgrade --check && /usr/bin/bootc upgrade --apply --soft-reboot=auto'
# (timer) OnCalendar=*-*-* 02:00:00   Persistent=true
```

(both at `features/bootc-updates/build_files/20-services.sh:19,31-32`).

## The orphan-code finding

There is a structural gap: **much of `sikker-selvbetjening/` is a fully-formed public-library kiosk product** for which the design documents provide essentially no requirements, hypotheses, or decisions. Examples with no design backing:

- **Accounts/login:** `create-users` sets up a `superuser` with a hardcoded password and NOPASSWD sudo (`05-users.sh:16-21`):

  ```
  echo "superuser:superuser" | chpasswd
  superuser ALL=(ALL) NOPASSWD: ALL
  ```

  `auto-login` logs in the `bruger` user automatically via GDM (`features/auto-login/.../gdm/custom.conf:3-4`).
- **Desktop lockdown:** `firefox-config` ships a locked browser with a hardcoded Sønderborg library homepage (`policies.json:5-6`), plus extensive dconf lockdown.
- **Kiosk operations:** `idle-monitor` (auto-logout; `sikker-selvbetjening/ToDo.txt:1-3` records it as broken), `power-schedule` (library open-hours daemon), and `mesh-agent` — a closed-source MeshCentral remote-management agent that enrols via group id (`features/mesh-agent/.../meshagent-bootstrap.sh:29,41`).

The design flags only two themes (user login, desktop apps) as lacking justification; the rest have no design mention at all.

## Bottom line

The prototype's core image-build and config-overlay pipeline is broadly consistent with the design vision. The divergences cluster into three themes: **(1) secrets handling is left open by the design and handled inline in the implementation**; **(2) several design decisions (1:1 repo, avoid disk images, portable pipeline) differ from what the implementation does**; and **(3) the base image spans a broader kiosk-product scope than the design documents currently address**.
