# Network and Infrastructure Security Assessment Lab

A practical security assessment of two Ubuntu virtual machines running in UTM. This project documents how network access, running services, SSH management and Linux file permissions were investigated, hardened and checked again in a controlled learning environment.

The final recovered scan shows TCP ports **22, 8000 and 8443 filtered** from the assessment VM. A separate target-side listener listing shows SSH running and no HTTP or HTTPS listeners on 8000 or 8443 at that observation. These results describe the recorded tests, not a live security guarantee.

## Read the project

- [Final Security Assessment Report](final-security-assessment-report.md) — findings, risk, evidence, remediation and retest results.
- [Network architecture](network-architecture.md) — assessment and management paths.
- [Retest guide](retest-guide.md) — bounded verification steps and interpretation limits.
- [Evidence register](evidence-register.md) — sources, retention and evidence strength.
- [Turkish terminology guide](glossary-tr.md) — optional plain-language explanations for readers learning the terminology.

## Environment and scope

| Component | Role | Lab address or path |
|---|---|---|
| `ubuntu-security` | Assessment VM | `10.10.10.10` |
| `ubuntu-target` | Assessed Ubuntu server | `10.10.10.20` |
| UTM host | Administrative SSH client | Local forwarding through `127.0.0.1:2222` |
| SSH management ingress | Target UFW rule | `enp0s2`, source `10.0.2.2` |
| Temporary web ingress | Target UFW rules during HTTP/TLS testing | `enp0s1`, source `10.10.10.10` |

Tests were confined to the lab. The project is an educational assessment, with no production client, breach or business impact claimed. Private IP addresses and generic lab account names are retained because they explain the tests. Personal host details and credential material are excluded.

## Recovered verification results

| Check | Recorded result | Evidence strength |
|---|---|---|
| Nmap on 23 November 2026 at 22:51 +0200 | `22/tcp filtered`; `8000/tcp closed`; `8443/tcp closed` | Screenshot directly inspected |
| Nmap on 23 November 2026 at 23:01 +0200 | All three tested ports `filtered` | Screenshot directly inspected |
| Target TCP listeners | SSH on IPv4 and IPv6 wildcard addresses; DNS on loopback; no 8000/8443 listener in the supplied listing | User-pasted command output |
| Final target firewall | Active; default deny incoming, allow outgoing; only SSH on `enp0s2` from `10.0.2.2` allowed | Screenshot directly inspected; original image not retained |
| Private report over HTTP | The exact report URL returned simulated confidential content before relocation, then `404 File not found` afterward | Before/after screenshots directly inspected; original images not retained |
| SSH authentication | Participant confirmed public-key enabled, password and keyboard-interactive disabled; explicit-key login succeeded and password-only attempt returned `Permission denied (publickey)` | User-reported configuration and test results; raw authentication logs not retained |
| TLS traffic | Supplied capture contained TCP exchange and binary payloads; no readable request, HTTP status or page body appeared in the inspected excerpt | User-pasted tcpdump output; TLS version/cipher and trusted identity not established |
| Private report access | `lab_auditor` denied; owner `student` able to read | User-reported result |
| File path permissions | Private directory `700`, report file `600`; parent `/home/student` already restricted other users | User description of path inspection |
| Management access after cleanup | New SSH connection returned the expected account and hostname | User-reported result |

`filtered` means that Nmap could not determine whether the port was open or closed because the probes did not provide a decisive response. It does not prove that the service is stopped. The report keeps network reachability and target-side process state separate.

The `UFW inactive` line in the first Nmap screenshot belongs to **ubuntu-security**, where that command was run. It is not evidence that the firewall on **ubuntu-target** was inactive.

## Assessment topics

- Service discovery and interpretation of open, closed and filtered TCP ports.
- Firewall allow rules, default ingress policy and removal of temporary access.
- SSH management access and authentication configuration.
- Controlled HTTP directory exposure and moving private content outside the served directory.
- HTTP plaintext compared with TLS-protected application traffic.
- Directory traversal permissions and file read/write permissions.

The controlled HTTP exposure was demonstrated at `http://10.10.10.20:8000/reports/internal-report.txt`. The same URL returned a 404 after the report was moved to `~/company-private`. This is distinct from the later shutdown of the temporary web services.

The SSH conclusion combines participant confirmation of `sshd -T` settings with positive key authentication and a negative password-only test. It is not an independently retained authentication-log audit. The HTTP client response was directly inspected; the participant's report of seeing its content in tcpdump is retained as a reported observation because the original HTTP capture text was not recovered. TLS used a self-signed certificate, so confidentiality observations must not be presented as proof of trusted server identity.

## Technical lessons

Stopping a service and removing its firewall rule address different conditions. The former removes a listener; the latter restricts permitted network access if a service later uses the same port.

Linux checks the directories along a file path as well as the file itself. The denied `lab_auditor` read must not be presented as proof of a newly fixed local disclosure: the parent home directory already restricted traversal before the additional permission tightening.

Packet confidentiality, certificate trust and server authentication are separate properties. Likewise, an observed service banner or port label is not proof of an exploitable software vulnerability.

## Tools used

UTM, Ubuntu, OpenSSH, UFW, Nmap, `ss`, `namei`, `chmod`, Python HTTP services, OpenSSL and tcpdump were used across the recorded lab workflow. Nmap **7.98** is visible in the recovered final scan screenshots. Versions of other tools are not asserted without an accompanying record.

## Publication boundaries

This repository contains documentation and selected, reviewed evidence. SSH private keys, TLS private keys, passwords, session tokens, raw packet captures and full VM images are excluded. `.gitignore` is a preventive aid; it does not remove data already committed to a repository.

No claim is made about untested ports, UDP, IPv6 reachability, the wider host network, patch completeness or production readiness. See the report for finding-specific evidence and limitations.
