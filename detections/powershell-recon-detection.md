# Detection: PowerShell-Based Reconnaissance

## Status
`Draft / Lab` — built for skills demonstration, not production use.

## Trigger
Detection assignments often start with a question rather than a blank page — e.g., "does our environment detect this?" after a CISA advisory is released. This detection was scoped in response to a CISA advisory describing adversary use of PowerShell for post-compromise reconnaissance.

> Replace this with the specific advisory reference (ID/title/link) you used when writing your own version of this detection.

## Behavior We're Trying to Detect
Adversary use of PowerShell to enumerate host, network, or user information as an early step after initial access — a common precursor to further lateral movement or privilege escalation.

## Data Source
- **Log channel**: `Microsoft-Windows-PowerShell/Operational`
- **Event ID**: 4104 (PowerShell Script Block Logging)
- **Why this source**: Event ID 4104 captures the full content of the script block being executed, not just the process name — this is essential for catching obfuscated, encoded, or dynamically constructed PowerShell commands that would be invisible in standard process-creation logging alone.

## Prerequisites for Data Availability
- Script Block Logging must be explicitly enabled via Group Policy or registry on the host — it is **not** enabled by default
- The Elastic Agent must have a custom integration configured against the `Microsoft-Windows-PowerShell/Operational` channel

## Simulated Attack
A reconnaissance-style PowerShell command was executed locally from the command line on the lab's Windows VM to generate a representative Event ID 4104 log entry (e.g., enumerating running processes, local users, or system configuration).

## Detection Logic (KQL, example structure)

```
event.code: "4104" and
winlog.channel: "Microsoft-Windows-PowerShell/Operational" and
powershell.file.script_block_text: (*Get-Process* or *whoami* or *systeminfo* or *Get-LocalUser*)
```

> Tune this list of matched strings/commands to the specific advisory technique you're targeting. Avoid overly broad matches (e.g., a bare `Get-Process`) in a real environment — this example is intentionally simple for lab purposes.

## Testing Steps
1. Confirm Event ID 4104 logging is functioning (see `docs/SETUP.md` validation checklist)
2. Run the simulated reconnaissance command from the VM
3. Confirm the raw event appears in Kibana Discover with the expected script block content
4. Confirm the detection rule fires and produces an alert with the correct host/user/command context
5. Re-run with a slightly modified command to check for basic evasion (e.g., string obfuscation) and note the result

## Known False Positives / Limitations
- Legitimate administrative scripts using the same cmdlets (e.g., routine `Get-Process` monitoring scripts) will also match — this rule is intentionally broad for demonstration purposes and would need allow-listing or additional context (parent process, user, time of day) in a real deployment
- This detection only covers one log source; it does not correlate with process creation, network connections, or authentication events, which would strengthen fidelity and reduce false positives
- Heavily obfuscated or base64-encoded PowerShell may not match simple string-based queries and would require decoding logic or a broader anomaly-based approach

## Next Steps / Tuning Ideas
- Correlate with Sysmon Event ID 1 (process creation) for parent-process context
- Add MITRE ATT&CK technique mapping (e.g., T1059.001 — PowerShell)
- Build an allow-list for known-good administrative scripts
- Expand to cover encoded command execution (`-EncodedCommand` usage)
