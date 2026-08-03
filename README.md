# Network Security Audit Against ISO/IEC 27001 Annex A Controls

**Course:** CY376 — Network Monitoring, Security and Auditing
**Track:** Blue Team
**Author:** Aboyinga Ahmed Atogyine (FCM.41.018.004.23), Class CY3A

End-of-semester project auditing a controlled lab network against selected ISO/IEC 27001 Annex A control domains, with technical evidence, findings, and a demonstrated remediation.

## Overview

This project simulates a network security audit for a fictional small business, **Meridian Bookkeeping Ltd.**, whose single internal server (**FIN-SRV-01**) hosts client financial records, a document upload portal, and file-sharing services. The server is represented in the lab by a **Metasploitable2** VM, audited from a **Kali Linux** workstation, both isolated on a VirtualBox host-only network with no route to the internet or host machine.

The audit assessed four Annex A control domains:

- **A.8** — Asset Management
- **A.9** — Access Control
- **A.12** — Operations Security
- **A.13** — Communications Security

## Lab Environment

| Host | Role | Address |
|---|---|---|
| Kali Linux | Auditor workstation | 192.168.56.3 |
| Metasploitable2 (FIN-SRV-01) | Audited asset | 192.168.56.4 |

Network: VirtualBox host-only (`vboxnet0`), 192.168.56.2–192.168.56.199, isolated from the internet and host production network.

**Tools used:** Nmap (service/version discovery), iptables (host-based access control).

## Key Findings

| Control | Issue | Risk | Recommendation |
|---|---|---|---|
| A.13 | Telnet (port 23) — cleartext credentials | High | Disable; enforce SSH only |
| A.9 | Unauthenticated root shell (port 1524) | High | Remove service; treat as potential incident |
| A.12 | vsftpd 2.3.4 (port 21) — known backdoor (CVE-2011-2523) | High | Upgrade or disable FTP |
| A.9/A.13 | MySQL/PostgreSQL reachable over network | Medium | Restrict to localhost/trusted hosts |
| A.9 | Legacy r-services (rlogin/rsh/rexec, ports 512–514) | Medium | Disable; use SSH |
| A.13 | No host-based firewall (iptables policy ACCEPT, no rules) | High | Default-deny inbound policy (remediated below) |

30 open TCP ports were identified in total via a full Nmap scan; six were documented as formal findings above.

## Remediation Demonstrated

A default-deny inbound firewall policy was implemented and verified on FIN-SRV-01:

```bash
sudo iptables -P INPUT DROP
sudo iptables -A INPUT -i lo -j ACCEPT
sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
sudo iptables -A INPUT -p tcp -s 192.168.56.3 -j ACCEPT
```

This changes the INPUT chain from policy ACCEPT (no rules) to policy DROP, explicitly allowing only loopback traffic, established/related connections, and traffic from the auditor's trusted workstation.

## Repository Contents

- `ISO27001_Audit_Report_FINAL.docx` — full project report (Abstract, Methodology, Implementation, Findings, Analysis, Recommendations, Appendices)
- Appendix A — raw Nmap scan output (`scan_full.txt`)
- Appendix B — full iptables ruleset (before/after remediation)

## Scope and Limitations

This was a one-month academic project against a single lab server, not a full organizational audit. A production engagement would additionally include staff interviews, policy review, physical security controls, and testing across multiple hosts/segments. Findings here are illustrative of the kinds of gaps a full audit would surface, not an exhaustive assessment.

## References

- ISO/IEC 27001:2022 — Information security, cybersecurity and privacy protection — ISMS Requirements
- [Rapid7 — Metasploitable2](https://github.com/rapid7/metasploitable2)
- [Nmap Reference Guide](https://nmap.org/book/man.html)
- [Netfilter/iptables Project](https://www.netfilter.org/projects/iptables/)
