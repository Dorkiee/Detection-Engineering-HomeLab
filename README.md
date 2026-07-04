# Detection Engineering Lab

A self-hosted, cloud-backed **Detection Engineering Lab** built to practice the full detection lifecycle: log collection, telemetry forwarding, detection rule authoring, and validation against simulated attacker behavior.
---

## Why this project

Detection engineering requires a safe environment to mimic real-world Tactics, Techniques, and Procedures (TTPs), build detection logic, and validate it before shipping to production. This lab is that environment, scoped small enough to run on a single low-cost cloud VPS plus a local VM.

This repo exists to:

- Document a repeatable, minimal-cost detection lab build (cloud + local hybrid)
- Practice the Detection Development Life Cycle (DDLC): idea → data validation → rule logic → testing → deployment → tuning
- Provide a portfolio artifact for interviews demonstrating hands-on SOC/Detection Engineering skills (SIEM administration, log source onboarding, Windows telemetry, rule writing, containerization)

## Tech Stack

| Layer | Tool | Purpose |
|---|---|---|
| Cloud host | DigitalOcean Droplet | Low-cost VPS to host the SIEM stack |
| Containerization | Docker / Docker Compose | Consistent, reproducible deployment of the Elastic Stack |
| SIEM | Elastic Stack (Elasticsearch, Kibana, Fleet Server) | Log storage, search, detection rule engine, dashboards |
| Agent | Elastic Agent | Ships host telemetry to the Elastic Stack via Fleet |
| Simulated victim host | Windows VM (VirtualBox/VMware/Hyper-V) | Generates real Windows event log telemetry |
| Log source | Windows Event Log — `Microsoft-Windows-PowerShell/Operational` (Event ID 4104) | Captures PowerShell Script Block Logging |
| Detection logic | Kibana / KQL detection rules | Alerts on simulated adversary behavior |
| Reference | CISA Advisory | Real-world detection use case used to scope the first rule |

## Repository Structure

```
detection-engineering-lab/
├── README.md                     # You are here
├── ARCHITECTURE.md               # System/data-flow diagram + design decisions
├── docs/
│   ├── SETUP.md                  # Full step-by-step build guide
│   └── detections/
│       └── powershell-recon-detection.md   # Documented detection case study
├── detections/                   # Exported Kibana/KQL detection rule definitions
├── scripts/                      # Helper scripts (attack simulation, deployment)
└── LICENSE
```

## Quick Start

See [`docs/SETUP.md`](docs/SETUP.md) for the full walkthrough. High level:

1. Provision a DigitalOcean droplet and install Docker
2. Deploy the Elastic Stack (Elasticsearch, Kibana, Fleet Server) via Docker Compose
3. Stand up a local Windows VM as the telemetry-generating host
4. Install the Elastic Agent on the VM and enroll it in Fleet
5. Configure a custom integration to capture PowerShell Script Block Logging (Event ID 4104)
6. Simulate attacker behavior on the VM (e.g., a PowerShell reconnaissance command)
7. Confirm the log arrives in Elasticsearch/Kibana
8. Write and test a detection rule against the CISA-advisory-referenced technique

## Example Detection Case

A worked example is documented in [`docs/detections/powershell-recon-detection.md`](docs/detections/powershell-recon-detection.md), covering:

- The trigger (a CISA advisory referencing a known TTP)
- The data source needed (PowerShell Script Block Logging, Event ID 4104)
- The simulated attack (a reconnaissance command run from the VM)
- The resulting detection rule logic
- Known false positives / tuning notes

## What This Demonstrates (for interviews)

- **Infrastructure**: standing up cloud infrastructure (DigitalOcean) and containerized services (Docker)
- **SIEM engineering**: deploying and configuring Elastic Stack, Fleet, and custom integrations
- **Windows telemetry**: understanding of Windows Event Logs, PowerShell logging, and what visibility gaps look like without proper configuration
- **Detection engineering workflow**: going from a real advisory → data requirement → simulated telemetry → working detection logic → documentation
- **Documentation discipline**: every detection is written up with its rationale, data requirements, and limitations — the same way a production detection engineering team tracks detection content

## Roadmap / Next Steps

- [ ] Add Sysmon for deeper endpoint visibility (process creation, network connections, registry)
- [ ] Map detections to MITRE ATT&CK techniques
- [ ] Introduce Atomic Red Team for repeatable attack simulation
- [ ] Automate detection rule deployment via GitHub Actions (Detection-as-Code)
- [ ] Add a second log source (e.g., Sysmon or Zeek) for cross-source correlation

## Disclaimer

This lab is for educational and personal skills-development purposes only. It is not intended to represent a production security architecture. Credentials, IPs, and any sensitive lab-specific values in this repo are placeholders — replace them with your own before deploying.

## Credits

- Tutorial series by [Bastradamus](https://bastradamus.com/)

## License

[MIT](LICENSE) — feel free to fork and adapt for your own lab.
