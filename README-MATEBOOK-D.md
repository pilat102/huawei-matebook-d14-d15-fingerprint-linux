# Goodix GXFP51A0 fingerprint on Huawei MateBook D14 / D15 (Intel WAX9)

Working fingerprint login (enroll + verify) on Linux for the MateBook **D14/D15**
Intel models — e.g. **NBLB-WAX9N** (D15 2020, Comet Lake) — with the Goodix
**GXFP51A0** sensor on SPI.

This repository is a **fork of [GodsQuantum/Huawei-Matebook-13-Linux](https://github.com/GodsQuantum/Huawei-Matebook-13-Linux)**
(the driver was reverse-engineered and validated there on the MateBook 13) with
the minimal changes needed for the D14/D15 board. All credit for the driver,
matcher, protocol work and tooling goes to the upstream authors; see
`fingerprint/PROVENANCE.md` and `LICENSES.md`. The fingerprint driver code is
`LGPL-2.1-or-later`.

> **Status: works, experimental.** Enroll, verify, KDE lock-screen unlock and
> `fprintd-verify` confirmed working on a MateBook D15 NBLB-WAX9N running
> Manjaro (kernel 7.3). Sleep (S3) and hibernation (S4) resume are **not
> certified** — keep your password as a fallback. This is reverse-engineered
> biometric software without a security certification; do not rely on it as a
> high-assurance factor.

## What differs from the MateBook 13 upstream

Two hardware-integration differences were found and fixed (full story below):

1. **Fingerprint IRQ is GPIO 279, not GPIO 48.**
   On the D14/D15 the line upstream uses (`INT34BB:00` line 48) is wired to the
   speaker amplifier (`speakers-enable`); the sensor's interrupt is on line
   **279** (`UART2_CTSB` pad — like the HONOR BBR-WAX9 sibling board). The
   driver now probes line 48 first and falls back to 279, so one binary serves
   both boards. Override with `GXFP_IRQ_LINE=<n>` if your board differs.

2. **The sensor answers FDT-down (cmd 0x32) with a single reply.**
   The upstream QA configuration (`GXFP_FDT_DOWN_PROOF_QA=1`) demands two
   replies (ACK + FDT report) and treats anything else as a protocol desync,
   forcing a full MCU/TLS session recovery on every capture — enrollment and
   verification crawl and eventually give up. Install the provided systemd
   drop-in to relax only that check:
   ```
   sudo mkdir -p /etc/systemd/system/fprintd.service.d
   sudo cp contrib/60-wax9-goodix51a0.conf /etc/systemd/system/fprintd.service.d/
   sudo systemctl daemon-reload && sudo systemctl restart fprintd
   ```

Everything else — protocol, TLS-PSK session, image pipeline, FAST-9/BRIEF/RANSAC
matcher, fprintd integration — is upstream's validated code. The sensor firmware
`GF_ST411SEC_APP_14115` is identical to the upstream reference unit.

## Install (Arch / Manjaro / CachyOS)

```bash
git clone https://github.com/pilat102/huawei-matebook-d14-d15-fingerprint-linux.git
cd huawei-matebook-d14-d15-fingerprint-linux
./fingerprint/install-arch.sh          # builds libfprint-goodix51a0 + fprintd
sudo cp contrib/60-wax9-goodix51a0.conf /etc/systemd/system/fprintd.service.d/
sudo systemctl daemon-reload && sudo systemctl restart fprintd
./fingerprint/gxfp51a0-doctor.sh       # all checks should pass
```

Then enroll in **System Settings → Users → Fingerprints** (about 20 touches,
slightly moving the finger each time), or use the guided wizard
`gxfp51a0-enroll`. First touch after boot can take a couple of seconds
(cold TLS/background preparation); steady-state unlock is ~1 s.

Other distros: see `fingerprint/README.md` (the portable `./fingerprint/install.sh`
path) — but note the IRQ fallback and the drop-in above apply there too.

## Tips for daily use (small 80x64 mm sensor)

- **Enroll the same physical finger under two names** (e.g. right-index and
  right-middle). Identify compares a scan against all enrolled templates, so
  40 reference views noticeably improve first-press acceptance compared to 20.
- Hold the finger still for ~2-3 s per touch. A failed press costs a
  ~1 s background re-calibration before the next attempt, so repeated quick
  taps feel much slower than one steady press.
- Expect **~2-3 s from touch to unlock** on a warm session (capture alone is
  ~0.6 s; this matches the upstream reference machine's measured
  finger-to-positive time). The first touch after boot or resume can take
  longer while the driver re-establishes the TLS session and background.
- Plasma's lock screen shows only a small fingerprint icon near the password
  field — there is no Windows/Android-style progress animation in stock KDE,
  and upstream's experimental QML integration is quarantined after a lockout
  incident. The driver logs every stage in
  `journalctl -b -u fprintd` if you want to see what it is doing.

## How the IRQ line was found (for other boards)

- `lsusb` shows nothing — the sensor hangs off **SPI** (`spi-GXFP51A0:00`,
  ACPI HID `GXFP51A0`), with the reset on GPIO 264 (same as MateBook 13).
- The ACPI `_CRS` names the interrupt via a runtime firmware field, and
  `/dev/mem` is restricted, so the pin was identified **electrically**:
  `goodix-re`-style tooling (see [szlukabence/goodix-fingerprint-spi-linux](https://github.com/szlukabence/goodix-fingerprint-spi-linux))
  confirmed the sensor answers (`GF_ST411SEC_APP_14115`), while a GPIO watcher
  showed line 279 is the one the sensor actually drives (high while the MCU is
  in reset, low when running, pulses after `EVK`).
- `tools/` in the reverse-engineering repo above has the probe scripts
  (`cshigh_repro.py --irq 279`, `irqwatch.py`).

If you have a different D14/D15 revision where this driver fails with
`GXFP51A0 GPIO48 IRQ input is not available`, your IRQ is neither 48 nor 279 —
re-run the discovery with the probe scripts and set `GXFP_IRQ_LINE`.

## Troubleshooting

- `Open failed with error: GXFP51A0 GPIO48 IRQ input is not available` —
  IRQ fallback missed your board; see above.
- Enrollment/verification very slow, log spam `FDT_DOWN_PROOF incomplete` —
  you skipped the systemd drop-in (difference #2).
- `journalctl -b -u fprintd -f` is your friend; the driver is verbose.
- Keep password authentication enabled at all times (PAM never gets a global
  `pam_fprintd` from this installer, so password login always remains).

## Credits

- [GodsQuantum/Huawei-Matebook-13-Linux](https://github.com/GodsQuantum/Huawei-Matebook-13-Linux) — the driver this fork is based on (LGPL-2.1-or-later driver, GPL tooling/docs).
- [szlukabence/goodix-fingerprint-spi-linux](https://github.com/szlukabence/goodix-fingerprint-spi-linux) — GXFP51A0 protocol reverse engineering, first-contact and discovery tooling.
- [fprint.freedesktop.org](https://fprint.freedesktop.org/) — libfprint/fprintd.
