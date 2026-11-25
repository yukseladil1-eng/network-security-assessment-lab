# Final Security Assessment Report

**Project:** Network and Infrastructure Security Assessment Lab  
**Environment:** Two Ubuntu virtual machines in UTM  
**Evidence review date:** 25 November 2026  
**Recorded final scan date:** 23 November 2026  
**Assessment type:** Controlled educational laboratory

## Executive summary

This assessment examined network exposure, administrative access, temporary web services and local file permissions in a small Ubuntu laboratory. A deliberately placed synthetic report was accessible over HTTP. Moving it outside the served directory changed the tested URL to an HTTP 404 response. Temporary web access rules were subsequently removed after the web services had stopped.

The final retained Nmap screenshot shows TCP ports 22, 8000 and 8443 as `filtered` from the assessment VM. A separate target-side listener listing contains SSH and loopback DNS listeners, with no TCP listener on 8000 or 8443. The final firewall screenshot shows UFW active, an incoming default-deny policy and one displayed SSH allow rule restricted to the management interface and source. The participant also confirmed successful key-based management access and rejection of a password-only attempt.

These results demonstrate the recorded improvements on the tested paths. They do not establish that every service, network path or account is secure. This report reconstructs historical evidence; it is not a new live assessment performed on the review date. Findings concern synthetic data and intentional lab conditions, with no production incident or actual business loss claimed.

## Scope and architecture

| Asset | Purpose | Address or access path |
|---|---|---|
| `ubuntu-security` | Assessment VM | `10.10.10.10` |
| `ubuntu-target` | Assessed server | `10.10.10.20` |
| Management workstation | Authorized SSH administration | `127.0.0.1:2222` forwarding to target TCP 22 |
| Target management ingress | Restricted SSH path | `enp0s2`, permitted source `10.0.2.2` |
| Target lab ingress | Temporary HTTP/HTTPS testing | `enp0s1`, test source `10.10.10.10` |

See the [network architecture](network-architecture.md) for a diagram and permission boundaries. Testing covered selected TCP ports, HTTP file exposure, an HTTP/TLS comparison, SSH authentication behavior and a specific local file path. It did not cover all TCP ports, UDP, IPv6 reachability, all SSH users or `Match` conditions, patch completeness, production deployment, or the broader host network. The available records do not prove an air-gapped environment.

## Method and evidence quality

The workflow combined source-side Nmap scans, target-side UFW and socket inspection, browser requests, tcpdump output and access tests. Remediation was followed by bounded repeat checks. Risk labels below are qualitative teaching assessments of the demonstrated condition; no formal CVSS score or production impact estimate was calculated.

The [evidence register](evidence-register.md) distinguishes retained screenshots, source screenshots visually inspected but not retained, user-pasted output, and participant confirmations. Prior guidance and expected outputs are not treated as proof that a command ran. Evidence identifiers link each conclusion to its basis. Lack of a retained original reduces independent reproducibility even when the source was inspected during review.

## Findings overview

| ID | Finding | Lab-context risk | Recorded outcome |
|---|---|---|---|
| F01 | Synthetic private report accessible through the HTTP document root | Medium potential confidentiality impact; synthetic content only | Tested URL returned 404 after relocation |
| F02 | Cleartext HTTP used for a demonstration service | Medium potential exposure on an observed path; no real secret demonstrated | TLS comparison completed; temporary listeners later absent |
| F03 | Temporary web allow rules remained after services stopped | Low, latent exposure if a listener returned | Rules removed; final scan changed web ports from closed to filtered |
| H01 | SSH authentication and management-path restriction | Hardening verification | Key login and password-only rejection reported |
| H02 | Private directory and file permission tightening | Defense in depth | Owner access and non-owner denial reported |

## F01 — Synthetic private report exposed through HTTP

**Condition and evidence.** A browser screenshot from the assessment VM displayed `CONFIDENTIAL - Simulated Company Report` at `http://10.10.10.20:8000/reports/internal-report.txt` (E07). The participant also described reaching the file through the directory listing. This directly supports exposure of the selected synthetic file to the lab client. It does not establish Internet-wide access or exposure of genuine company information.

**Risk.** Placing non-public material beneath a served directory can make it retrievable through the web service, independently of whether an unrelated local user can traverse the owner's home directory. Here the service could serve the file using its own access rights. The potential confidentiality consequence is meaningful, although the actual demonstrated content was deliberately simulated.

**Remediation.** The recorded workflow moved `~/lab-web/reports/internal-report.txt` into `~/company-private/`, outside the HTTP document root. The private directory and file permissions were subsequently tightened as described in H02.

**Retest and status.** A second source screenshot showed the same URL returning `Error code: 404` and `File not found` (E08). The tested disclosure path was therefore remediated at that observation. This single request does not verify the absence of alternate copies, aliases, cached responses or other exposed files. Future service deployment should keep private material outside every served path and repeat the exact retrieval check.

## F02 — Cleartext HTTP demonstration and TLS comparison

**Condition and evidence.** The participant pasted an HTTP client result showing status `200` and `Welcome to CyberSecurity Lab` (E09), and reported seeing readable content in the HTTP capture. The original HTTP capture attachment was not recovered for independent review. The client output proves successful content retrieval; by itself it does not prove what appeared in a packet capture.

**Risk.** The lab demonstrates why unencrypted application traffic is inappropriate for sensitive information on paths where traffic can be observed. No password, genuine confidential record or interception by a third party was demonstrated. The recorded HTTP-versus-TLS comparison is therefore reported with this evidence limitation.

**Remediation and comparison.** A temporary HTTPS service was exercised on TCP 8443 using a self-signed certificate. The participant reported opening the page after accepting its certificate warning (E12). The pasted tcpdump output for that connection was directly reviewed: it showed the TCP exchange and non-readable application payloads, without visible HTTP request lines, response status or page content in the supplied excerpt (E13).

**Retest and status.** The TLS observation is consistent with encrypted application traffic. Readable handshake strings such as `h2` and `http/1.1` do not show that the page body was sent in cleartext. However, this ASCII-rendered capture does not establish the negotiated TLS version, cipher suite, certificate validation or resistance to impersonation. Accepting a self-signed warning did not establish trusted server identity. A trusted, correctly validated certificate would be required for a normal deployment. The final listener output contained neither temporary web service (E03), and their firewall allowances were removed (E15). This was a learning comparison, not a production TLS certification.

## F03 — Unused web access rules after service shutdown

**Condition and evidence.** During testing, UFW allowed source `10.10.10.10` to TCP 8000 and 8443 on `enp0s1`, alongside the restricted SSH management rule (E11). After the temporary web services were no longer listening (E03), the retained 22:51 scan showed TCP 8000 and 8443 `closed` and TCP 22 `filtered` (E01).

**Risk.** An unused allow rule can permit unintended access if a service later binds the same port. A closed port is not itself evidence of an exploitable running service. The concern here was unnecessary retained permission and possible future exposure, not a successful attack.

**Remediation.** The two temporary web allow rules were removed while preserving the management SSH rule. The final UFW screenshot showed `Status: active`, `Default: deny (incoming), allow (outgoing), disabled (routed)` and only the displayed rule `22/tcp on enp0s2 ALLOW IN 10.0.2.2` (E15).

**Retest and status.** The retained 23:01 scan classified all three selected TCP ports as `filtered` from the assessment VM (E02). This is consistent with the recorded firewall cleanup. The participant confirmed that a new authorized management connection still returned the expected lab user and target hostname (E06). Cleanup is supported for the recorded configuration and test path; persistence across a later reboot was not independently checked.

## Hardening verifications

### H01 — SSH key authentication and management access

The participant confirmed effective settings of `PubkeyAuthentication yes`, `PasswordAuthentication no` and `KbdInteractiveAuthentication no`, successful explicit-key access, and `Permission denied (publickey)` when attempting a password-only connection (E14). This supports key-only behavior for the tested account and connection. Final UFW evidence separately restricts the displayed SSH allow rule to the management path (E15). These are complementary controls: authentication checks identity, while filtering limits the permitted path. The evidence does not cover every user, source-dependent setting or possible authentication method. Private key material is excluded from this repository.

### H02 — Private file permissions

The participant reported directory mode `700` and file mode `600`, denied access for `lab_auditor`, and successful owner access (E04–E05). Mode `700` gives the directory owner read, write and traversal permissions; mode `600` gives the file owner read and write permissions. Crucially, `/home/student` was already reported as `750`, restricting traversal by other users. The denied read cannot be attributed exclusively to the later child permission changes, and does not prove that unauthorized local reading had previously succeeded. This is documented as defense in depth, separately from the proven HTTP retrieval.

## Final interpretation and remaining assurance limits

The evidence supports remediation of the demonstrated HTTP URL, removal of temporary web allowances and preservation of reported authorized SSH access. In Nmap, `open` means a service accepts connections, `closed` means a response indicates no accepting service at that port, and `filtered` means probes did not let Nmap determine open versus closed. The final `filtered` results must therefore be considered alongside the target listener listing, not used as proof that every service stopped.

The `UFW inactive` line in E01 belongs to **ubuntu-security**, not the assessed target. The final target firewall state is evidenced separately by E15. Only E01 and E02 are retained source images in this package, with ancillary metadata removed and displayed content unchanged; other evidence has the provenance and limitations stated in the register. The [retest guide](retest-guide.md) provides bounded checks if fresh evidence is needed. No untested environment state is represented as verified.
