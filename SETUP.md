# Setup Guide

This walks through building the lab end-to-end, following the structure of the source tutorial series (Part 1: infrastructure, Part 2: telemetry + first detection). Adjust specifics to your own environment/credentials — nothing here should be copy-pasted with real secrets into your repo.

## Prerequisites

- A DigitalOcean account (or any cloud VPS provider of your choice)
- Docker & Docker Compose knowledge (basic)
- A hypervisor for the local VM: VirtualBox, VMware Workstation/Fusion, or Hyper-V
- A Windows 10/11 ISO for the victim VM
- Basic familiarity with Kibana / Elastic Stack navigation
- Minimum resources for the local VM: check current Windows minimum specs (typically 4GB+ RAM, 2 vCPUs, 60GB+ disk) — Part 1 has no local requirements since the SIEM itself is cloud-hosted; the VM requirement only applies once you reach Part 2

## Part 1 — Cloud Infrastructure & Elastic Stack

1. **Provision the droplet**
   - Create a new DigitalOcean droplet (a basic/shared CPU plan is sufficient for a lab)
   - Choose a Docker-preinstalled image from the DigitalOcean Marketplace, or install Docker manually via the official Docker install script
   - Secure the droplet: create a non-root user, enable a firewall (UFW or DigitalOcean Cloud Firewall), restrict SSH access, and consider key-based auth only

2. **Deploy the Elastic Stack with Docker**
   - Pull down (or write) a `docker-compose.yml` defining Elasticsearch, Kibana, and Fleet Server
   - Set environment variables for stack version, memory limits, and security settings (enable Elastic security features / TLS as appropriate for your comfort level in a lab)
   - Bring the stack up: `docker compose up -d`
   - Confirm Elasticsearch is healthy and Kibana is reachable over its exposed port

3. **Access Kibana**
   - Browse to `https://<your-droplet-ip>:5601` (or your configured domain/reverse proxy)
   - Log in and verify you can reach the Fleet section under **Management**

4. **Set up Fleet Server**
   - In Kibana, go to **Fleet → Settings** and configure the Fleet Server host (your droplet's public IP/domain)
   - Generate a Fleet Server policy and enrollment token — you'll need this in Part 2 when enrolling the Elastic Agent from the Windows VM

> At the end of Part 1 you should have: a running Elastic Stack in Docker, accessible Kibana, and Fleet Server ready to accept agent enrollments. No local VM is required yet.

## Part 2 — Local VM, Telemetry, and First Detection

5. **Build the Windows VM**
   - Create a new VM in your hypervisor of choice with the Windows ISO
   - Complete the OS install and apply updates
   - Configure networking so the VM can reach your droplet's Fleet Server endpoint (NAT is usually sufficient; avoid bridging it directly onto a production network)

6. **Install and enroll the Elastic Agent**
   - Download the Elastic Agent installer matching your Elastic Stack version
   - Run the enrollment command using the Fleet Server URL and enrollment token generated in step 4
   - Confirm the agent shows as **Healthy** in **Fleet → Agents** in Kibana

7. **Add a custom integration for PowerShell logging**
   - In Kibana, go to **Fleet → Agent Policies** and add a **Custom Windows Event Log** integration
   - Set the channel to `Microsoft-Windows-PowerShell/Operational`
   - This channel captures Event ID 4104 (PowerShell Script Block Logging), which records the full contents of executed PowerShell code — critical for catching obfuscated or encoded attacker commands
   - Ensure Script Block Logging is enabled on the Windows VM itself via Group Policy or the registry (`HKLM\Software\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging`), otherwise Event ID 4104 will never be generated

8. **Generate test telemetry**
   - From an elevated PowerShell prompt on the VM, run a benign "reconnaissance-style" command (e.g., a script enumerating local processes or system info) to simulate early-stage attacker behavior
   - Confirm the corresponding Event ID 4104 entry appears in Kibana under **Discover**, filtered to your PowerShell integration's data view

9. **Scope a detection**
   - Pick a real-world reference point for the detection — this lab used a CISA advisory describing a specific technique
   - Translate the advisory's described behavior into a searchable pattern against the ingested PowerShell log data (command-line strings, script content, parent process, etc.)

10. **Write and test the detection rule**
    - In Kibana, go to **Security → Rules → Create new rule** (or **Alerts** depending on your Kibana version/solution)
    - Choose a Custom Query rule type using KQL
    - Scope the query to the relevant event code and PowerShell command patterns identified in step 9
    - Set a reasonable schedule/lookback and severity
    - Trigger the simulated attack again from the VM and confirm the rule fires an alert in Kibana

11. **Document the detection**
    - Fill out a detection case study (see [`docs/detections/powershell-recon-detection.md`](detections/powershell-recon-detection.md) as a template) covering: the trigger/reference, data source, detection logic, testing steps, and known false positives

## Validation Checklist

- [ ] Elastic Stack containers all report healthy (`docker ps`)
- [ ] Kibana reachable and logged in
- [ ] Fleet Server shows as healthy
- [ ] Elastic Agent on the Windows VM shows **Healthy** in Fleet
- [ ] Script Block Logging enabled on the VM (registry/GPO confirmed)
- [ ] Event ID 4104 entries visible in Kibana Discover after running a test command
- [ ] Detection rule created and fires correctly when the simulated attack is re-run
- [ ] Detection documented in `docs/detections/`

## Common Pitfalls

- **No Event ID 4104 appearing**: Script Block Logging is not enabled on the VM, or the custom integration is pointed at the wrong event channel name
- **Agent shows Unhealthy/Offline in Fleet**: check network connectivity between the VM and the droplet, and confirm the droplet's firewall allows the Fleet Server port
- **Kibana not reachable**: check the droplet's firewall/security group rules for the Kibana port, and confirm the container is actually running (`docker logs <container>`)
- **Detection rule never fires**: double check the KQL query against the actual field names/values in Discover — Elastic Common Schema (ECS) field names can differ from what you expect
