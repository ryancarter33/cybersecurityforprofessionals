# Meridian Trust Bank Multi-Day Security Case Study

This multi-day cybersecurity lab followed a simulated incident at Meridian Trust Bank from vulnerability assessment through forensic investigation, incident scoping, threat hunting, detection engineering, and recovery planning.

## Part 1: Vulnerability Assessment and Digital Forensics

I reviewed vulnerability scan results for the Meridian Trust Bank environment and manually validated reported software versions against affected vulnerability ranges. I confirmed legitimate findings, removed false positives, checked CVEs against the CISA Known Exploited Vulnerabilities catalog, and reprioritized remediation based on confirmed exploitation rather than CVSS score alone.

I then produced a vulnerability assessment report containing prioritized findings, compensating controls, remediation guidance, scan-cadence recommendations, and an executive summary.

The investigation then transitioned into digital forensics after MTB-WEB01 was confirmed to be compromised. I analyzed volatile captures, authentication records, Apache logs, recovered files, network connections, and filesystem timestamps to reconstruct the attack.

I verified evidence integrity using SHA-256, decoded a Base64-encoded artifact using CyberChef, created a consolidated indicator-of-compromise list, and reconstructed the attack chain from initial exploitation through privilege escalation, persistence, command and control, data staging, and attempted evidence removal.

The investigation concluded with documentation of the root cause, incident timeline, evidence-handling process, indicators of compromise, and prevention recommendations.

[View Part 1 – Vulnerability Assessment and Digital Forensics](Vulnerability-Assessment-and-Digital-Forensics-Case-Study.pdf)

## Part 2: Incident Scoping, Threat Hunting, and Detection Engineering

The second lab extended the investigation using retained perimeter firewall records. This evidence moved the beginning of attacker activity back to August 28, demonstrating that the compromise began several days earlier than the original disk evidence suggested.

I hunted across the server environment and determined that MTB-FILE01 was also compromised after an SSH key from MTB-WEB01 was used for lateral movement. I also evaluated MTB-APP01 and MTB-DC01 and found no related indicators, while periodic outbound activity from MTB-DVWA01 was determined to be legitimate monitoring traffic.

Indicators from the incident were reorganized using the Pyramid of Pain to distinguish easily changed indicators, such as hashes and IP addresses, from more durable behavioral indicators.

I also mapped attacker activity to MITRE ATT&CK, developed four Sigma detection rules, and documented when those detections could have identified the attack.

The investigation concluded with containment, eradication, and recovery planning and a post-incident review covering the detection gap, root causes, control failures, data at risk, and specific recommendations with assigned owners and deadlines.

[View Part 2 – Incident Scoping, Threat Hunting, and Detection Engineering](Incident-Scoping-Threat-Hunting-and-Detection-Case-Study.pdf)

## Key Lessons Learned

1. **A scanner result or single evidence source cannot be treated as the complete answer.** Manual validation removed false positives, while correlation across disk, authentication, endpoint, and firewall evidence revealed several additional days of attacker activity.

2. **Behavioral indicators are more durable than hashes, filenames, or IP addresses.** Although the attacker changed infrastructure and tooling on the second compromised host, behaviors such as persistence, long-lived outbound communication, privileged access, and lateral movement remained detectable.

3. **Collecting security data is only valuable when organizations can turn it into action.** Firewall records captured hours of suspicious beaconing, but insufficient correlation and alerting allowed attacker activity and data transfer to continue before a human response occurred.