# OT/ICS Security Operations Centre — Water Treatment Plant

Securing the chemical treatment unit of a desalination plant: building an OT
Security Operations Centre on an industrial-control-system simulation, running
realistic attack scenarios, and assessing compliance with the Saudi **NCA
OTCC-1:2022** controls. (Course: CECY412 — ICS Security)

> **Author:** Rayan Kurdi (Team Lead — Network Security Engineer + SOC Analyst)
> **Team project** with Yazan Alhazmi (incident & compliance lead).
> **Environment:** GRFICSv3 (Dockerized ICS) on Ubuntu 24.04, Apple Silicon host
> **Skills:** OT/ICS security · Modbus/TCP · firewall hardening · IDS rule writing · incident response · NCA OTCC-1:2022 compliance

---

## Scenario
GC Desalination Plant supplies drinking water to a coastal city, so a cyber
incident in its chemical-treatment unit is a public-safety problem. An attacker
who can reach the controller (PLC) can open or close dosing valves and stop
treatment. At the start, the audit found the plant effectively unprotected.

## What was found (before)
- **Allow-all firewall** — any office/DMZ host could reach the PLC, the
  engineering workstation, and the firewall itself.
- **Default credentials** on every device (PLC, HMI, firewall, workstation).
- **Modbus/TCP** with no authentication or encryption — all process values and
  setpoints readable in clear text.
- **No monitoring** — no detection rules, no change-approval process.

## Controls implemented (my tasks — 2, 3, 4)
- **Network segmentation:** replaced the allow-all rule with a **default-deny**
  firewall policy; only the HMI↔PLC Modbus flow (port 502) is permitted, all else
  blocked and logged.
- **Hardening:** rotated all four default passwords to 16-character random values
  and verified the old ones fail; confirmed only the required protocol (Modbus)
  is enabled (S7, DNP3, EtherNet/IP off).
- **Backup & recovery:** backed up the PLC program and configuration off-box and
  proved restore from backup in **~8 seconds**.
- **Monitoring (OT SOC):** wrote **8 custom IDS rules** from the plant's Modbus
  baseline, plus a PLC program-integrity check.

## Results
Four attack scenarios were run twice — a planned run and a **blind run** (a
scheduler started each attack at a random, sealed time):

| Incident | Detected? | Time |
|----------|-----------|------|
| Unplanned write outside change window | Yes | ≤11 s |
| Unapproved PLC program change | Yes | ≤11 s |
| Rogue device on the DMZ | Yes | ≤11 s |
| Old default-password logins | Blocked by new passwords (no log trace) | — |

The plant kept running throughout — the HMI showed live values after every
change and incident.

## Compliance
Performed an **NCA OTCC-1:2022** gap assessment (facility criticality Level 1 —
High) and documented remaining risks with prioritised recommendations
(e.g. IDS blind to intra-control-network traffic, HMI placed in the DMZ).

## Report
The full 31-page deliverable — incident reports (D1), final report (D2), and
appendices A–E (asset inventory, change log, OTCC gap assessment, evidence,
work log) — is in
[`docs/CECY412-ICS-SOC-Report.pdf`](docs/CECY412-ICS-SOC-Report.pdf).

---

## Security note
All credentials shown in the report are **well-known ICS defaults** from the
GRFICSv3 training simulation (e.g. OpenPLC's published `openplc/openplc`), listed
only as assessment findings. The new 16-character passwords set during hardening
were never recorded in the report or committed to this repository.

*All work was performed in an isolated GRFICSv3 lab simulation for educational
purposes.*
