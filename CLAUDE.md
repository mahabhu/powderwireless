# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **POWDER testbed profile** for 5G intra-gNB handover experiments with OCUDU and Open5GS on the conducted RF attenuator matrix, using a single CU/DU node. It is a reduced version of the two-gNB profile in [reference/ocudu-fbs-handover/](reference/ocudu-fbs-handover/) (itself based on `https://gitlab.flux.utah.edu/dmaas/srs-rf-matrix`), with the second gNB (`cudu2` + N300) and the second UE removed. There is no O-RAN RIC or shared VLAN.

The entry point is [profile.py](profile.py).

## Platform Context

Experiments are instantiated via the [POWDER](https://powderwireless.net) portal, not run locally. The repo is checked out at `/local/repository` on every node; scripts in [bin/](bin/) run at boot or manually over SSH.

- **OS**: Ubuntu 22.04 (COTS UE image on the NUCs)
- **Build/config directory on nodes**: `/var/tmp` (OCUDU in `/var/tmp/ocudu`, gNB configs copied to `/var/tmp/etc/ocudu`)
- UHD 4.10 debs are vendored in [debs/](debs/)

## Topology

| Node | Hardware | Role |
|------|----------|------|
| `cn5g` | d430 | Open5GS core (192.168.1.1) |
| `cudu` | d740 + `x310-1` | gNB, gNB ID 411, PCI 1 and 2 (192.168.1.2) |
| `ue1` | `nuc27` | Quectel RM520 COTS UE |

RF paths in the matrix are fixed by POWDER staff; only attenuation values can be changed.

## gNB configs ([etc/ocudu/](etc/ocudu/))

- `gnb_rf_x310_ho.yml` — upstream intra-gNB handover: two cells (PCI 1, 2) on the X310, one per UBX daughter card
- `gnb_rf_x310_ho_phytap.yml` — same, with the UL EPRE PHY tap enabled (needs the `enable_phy_tap` parameter)
- `mimo_usrp.yml` — `cell_cfg` overlay for 2x2 antennas

## Key Scripts

| Script | Node | Purpose |
|--------|------|---------|
| [bin/deploy-open5gs.sh](bin/deploy-open5gs.sh) | cn5g | Install Open5GS, add the two subscribers |
| [bin/deploy-ocudu.sh](bin/deploy-ocudu.sh) | cudu | Build OCUDU, copy gNB configs |
| [bin/setup-cots-ue.sh](bin/setup-cots-ue.sh) | ue1 | Quectel modem setup |
| [bin/update-attens](bin/update-attens) | cudu | Set a path group (`ru1ue1`, `ru2ue1`, `all`) to 0..95 (+30 dB) |
| [bin/handover](bin/handover) | cudu | Fade `ue1` between the X310's two cells |

Run attenuator commands on `cudu`, not the NUCs.
