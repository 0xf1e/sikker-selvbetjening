# 90 — Orphan Code (reverse pass)

Code/features that **no design claim covers**. Reverse direction of the rest of
this comparison: this catches drift that a design→code pass misses — most
notably a large "kiosk hardening" layer that the design never describes, which
the design itself half-acknowledges (C7).

| Verdict | Count |
|---------|-------|
| ALIGNED | 1 |
| (orphan features carry no claim ID — listed thematically below) |
| **claim total** | **1** |

---

### C7 — ALIGNED

Claim (meta-note): The "in-progress user login functionality and the setup of desktop apps" features lack justifications; "I recommend evaluating the underlying hypotheses behind the development of those features also."
Design: design/prototype_strategy.md:21
Code: sikker-selvbetjening/features/create-users/build_files/05-users.sh:6-24 (creates `bruger` kiosk user + `superuser` admin with hardcoded password `superuser:superuser` and `NOPASSWD: ALL` sudo), sikker-selvbetjening/features/auto-login/system_files/etc/gdm/custom.conf:1-12 (GDM auto-login for `bruger`), sikker-selvbetjening/features/firefox-config/system_files/etc/firefox/policies/policies.json:1-90 (locked browser config), sikker-selvbetjening/features/gnome-settings/... (heavy GNOME lockdown), sikker-selvbetjening/features/gnome-extensions/..., sikker-selvbetjening/features/favorite-apps/...
Finding: The design's observation is accurate — these features are present and in progress, and the design provides no claim/justification for them. The recommendation to "evaluate the underlying hypotheses" is not reflected anywhere in the repos (no design note, no ADR). ALIGNED with the *claim* (the features exist as described); the recommended evaluation remains unaddressed.

---

## Orphan features (no design claim at all)

The design documents describe a build/configuration pipeline and four concerns.
They say essentially nothing about the **on-device kiosk experience**. The base
image repo nonetheless contains a large, elaborate kiosk-hardening layer. These
features have **no design backing** and are grouped by theme.

### A. User accounts & login (maps to C7's "user login")

- `features/create-users/` — `bruger` (uid 1000, passwordless) + `superuser` (uid 1001, **hardcoded password `superuser`**, passwordless sudo). No design claim about accounts, admin users, or credentials.
- `features/auto-login/` — GDM auto-login/timed-login for `bruger`. No design claim about login behaviour.
- `features/reset-users/` — wipes the `bruger` home directory on every boot and on session stop (`kiosk-reset.sh`, `reset-on-boot.service`, `user@.service.d/kiosk-reset.conf`). No design claim about session/data reset.

### B. Desktop apps & lockdown (maps to C7's "desktop apps")

- `features/firefox-config/` — locked Firefox policies including a **hardcoded homepage `https://biblioteket.sonderborg.dk`**, Danish locale, disabled sync/passwords/telemetry/studies, website filtering, sanitize-on-shutdown.
- `features/gnome-settings/` — extensive dconf lockdown (disable command line, lock screen, save-to-disk, user switching; single workspace; power/sleep disabled; fingerprint/smartcard auth disabled).
- `features/gnome-extensions/` — dash-to-panel, apps-menu, just-perfection; taskbar locked.
- `features/favorite-apps/` — pinned apps + a dconf **lock** preventing user changes.
- `features/auto-browser/` — auto-launches Firefox with configured URLs on login.
- `features/logout-button/` — Danish logout confirmation dialog as a pinned app.
- `features/background-image/` — default desktop background (the *config-driven* part has backing via C1; the *default* asset does not).

### C. Kiosk operations (no design mention whatsoever)

- `features/idle-monitor/` — auto-logs out idle users after 1 hour (`kiosk-monitor.sh`, Danish prompt). **`sikker-selvbetjening/ToDo.txt:1-3` records this feature as broken** ("not working … methods used to fetch the time idle just return a 0"). A broken health/protection feature with no design claim.
- `features/power-schedule/` — library open-hours power management daemon (`power-scheduler.py`, `power-schedule.json` with `rtcwake`/`vbetool` suspend/blank), hardcoded to a Monday–Friday schedule.
- `features/usb-handler/` — desktop notifications on USB insert/remove.
- `features/pc-name/` — sets hostname from hardware serial / MAC on first boot.
- `features/quiet-boot/` — kernel args for quiet/splash boot.
- `features/reboot/` — a "nuclear shutdown handler" that kills all non-essential processes before reboot.
- `features/mesh-agent/` — **MeshCentral remote-management agent** (downloads and runs a closed-source agent, enrols via group id). This is real remote monitoring/management infrastructure — the design's only monitoring claim (C5/C26) explicitly says observability is *simulated* and provides no remote-management design.
- `features/keyboard-layout/` (`dk`), `features/locale/` (`da_DK`, `Europe/Copenhagen`), `features/fedora-packages/` (LibreOffice, Danish langpacks, gnome extensions) — Danish-oriented defaults with no design claim.

### D. Hardcoded / environment-specific specifics (no design claim, but baked in)

- `sikker-selvbetjening-config/config/config.yml` — `demo` domain, `Nordborg` device group, `mesh/demo/...` id, `borgernet` wifi SSID, `sikkerselvbetjening.org/demo/` mesh URL, Sønderborg printer — concrete demo data with no design backing.
- `firefox-config` Sønderborg library homepage; `power-schedule` weekday hours — municipality-specific values baked into the base image.

---

## Why this matters

The design↔code passes in docs 10–70 confirm the *build/configuration pipeline*
matches the design well (with the noted conflicts). This reverse pass shows the
pipeline is a minority of the codebase: the bulk of `sikker-selvbetjening/` is a
fully-formed **public-library kiosk product** (locked browser, locked GNOME,
auto-login, session reset, idle logout, open-hours power management, remote
agent) for which the design documents provide essentially no requirements,
hypotheses, or decisions. The design itself flags only two of these themes (C7:
user login + desktop apps); the rest — idle-monitor, power-schedule, mesh-agent,
reset-users, usb-handler, pc-name, reboot — appear with no design mention at all.

These are the strongest candidates for "code that should either be documented
back into the design or explicitly scoped out," and `ToDo.txt`'s note that
idle-monitor is broken underscores that some of this orphan code is not yet in a
working state.
