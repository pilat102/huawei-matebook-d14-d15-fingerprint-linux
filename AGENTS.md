# Maintainer instructions — Huawei MateBook 13 Linux

## Fork divergence (MateBook D14/D15 WAX9)

This fork carries two board-specific changes on top of upstream; keep them
when merging upstream work:

1. `fingerprint/driver/goodix51a0/gx51_transport.c` (`gx51_open_irq_gpio48`):
   probes the IRQ line as GPIO48 first, falls back to GPIO279, with
   `GXFP_IRQ_LINE` env override. On WAX9 boards GPIO48 is `speakers-enable`.
   `SOURCE_MANIFEST.sha256` is updated accordingly.
2. `contrib/60-wax9-goodix51a0.conf`: systemd drop-in setting
   `GXFP_FDT_DOWN_PROOF_QA=0` for WAX9 (single FDT-down reply). Documented in
   `README-MATEBOOK-D.md`.

Before changing, building or installing any GXFP51A0 candidate, **read**
`fingerprint/docs/HALL_OF_FAME.md`, `fingerprint/docs/HARDWARE_TRIALS.csv`,
and the private canonical Cloud9 workspace handoff (not tracked by Git).
Use the measured winner's *code* as a new experimental base, with separately
validated KDE/desktop corrections; also benchmark the currently installed
version. Never promote an acceleration in READY→DETECT when first-touch
matching regresses. Keep both failed trials and the rollback package.
A record is not a reliability certification; compare the same end-to-end
metric under real user-attested LOCK, S3 and S4 trials.


This is the public source repository. Read README.md, docs/PROJECT_MAP.md, and the private canonical workspace handoff before modifying driver or installers. The current installer driver is QA63.2, while the latest *published* reference release is rel71.74 (matebook13-rel71.74); two deep-S3 v12 recoveries have evidence, but repeated first-touch deep-S3, S4, cold boot and battery usage are NOT certified. Historical rel71.69 and older branches, trial documents and old README sections are NOT current installation instructions.

## Changes
- Preserve native fprintd/PAM/password fallback, enrolled templates, matcher threshold 7 and authenticated TLS-GCM. No speculative firmware erase/UPFW, GPIO112 manipulation or bypassing PAM/global sudo policy.
- Work within an isolated development environment; preserve source and library ABI. Test fingerprint/research offline and the entire fingerprint verify suite. Any reliability/speed claim requires actual user-attested LOCK / real S3 evidence; unit CI alone does not certify hardware.
- Keep version, package configuration, all EN/FR/ZH README translations, Hall of Fame evidence, docs and the private canonical handoff consistent. Respect CONTRIBUTING.md and SECURITY.md; never commit biometric images/templates, PMK/PSK, Windows proprietary blobs, personal paths/IPs or credentials.
- Prefer native driver/library/platform features over services, polling, prestart GPIO hacks or extra sleep hooks. Rejected rel71.65/66/68 reset/timing experiments are historical.
- Use published rel71.74 tags and SHA256 evidence only for those tagged assets, never as hashes or validation claims for newer QA63.2 source. Test and document changes before merging. Keep public documentation accessible to humans and agents with links, not accumulated stale checkpoint headlines.
