# Evidence register

Reviewed on **25 November 2026** from the participant's historical lab records. This register preserves observations needed to assess the report without publishing the full conversation, personal host details or credentials. Review did not execute fresh tests against the VMs.

## Evidence types

- **A — Retained source image:** screenshot inspected and included in this repository with its original pixel content; ancillary metadata removed from the public copy.
- **B — Inspected source:** screenshot or pasted command output inspected during review; the original is not packaged. The entry is a transcription or observation, not a replacement raw artifact.
- **C — Participant confirmation:** a result explicitly reported by the participant; no corresponding raw result independently inspected.

These types describe provenance, not a numerical confidence score. Commands requested by prior guidance and expected outputs are not evidence of execution. Formatting in the excerpts below is normalized for readability; omitted data are identified.

| ID | Type | Observation | Supports / limitation |
|---|---|---|---|
| E01 | A | Nmap, 23 Nov 2026 22:51 +0200: TCP 22 filtered; 8000 and 8443 closed | Selected-port state before rule cleanup; source VM UFW inactive is not target status |
| E02 | A | Nmap, 23 Nov 2026 23:01 +0200: TCP 22, 8000 and 8443 filtered | Selected-port state after cleanup; does not prove absent listeners |
| E03 | B | Target `sudo ss -ltnp` pasted output shows SSH and loopback DNS, no 8000/8443 | TCP listeners at the recorded observation only |
| E04 | C | Owner read succeeded; `lab_auditor` read returned permission denied | Access-test result; not proof that access was previously possible |
| E05 | C | Participant described `namei -l`: home 750, private directory 700, file 600 | Parent traversal restriction already existed |
| E06 | C | Management SSH after cleanup returned expected `student` / `ubuntu-target` | Reported management access preservation |
| E07 | B | Browser screenshot displayed synthetic report at the HTTP URL | Direct visual evidence of the selected disclosure; image not retained |
| E08 | B | Browser screenshot showed 404 at that same URL after relocation | Direct visual evidence of the selected retest; image not retained |
| E09 | B + C | Pasted HTTP client returned 200 and lab page body; participant reported readable captured content | Client output inspected; original HTTP packet-capture attachment not recovered |
| E10 | B | OpenSSL version and Python HTTP-server help pasted | TLS command support present in that environment; not a security assessment of tool versions |
| E11 | B | Pasted target UFW list active with SSH and two source-restricted web rules | Temporary rule state during TLS testing |
| E12 | C | HTTPS page opened after accepting a self-signed certificate warning | Reported usability; does not establish trusted identity |
| E13 | B | Pasted TCP 8443 tcpdump text inspected | Consistent with encrypted application traffic; no raw packet file or complete TLS analysis |
| E14 | C | SSH effective settings confirmed; explicit-key success and password-only rejection confirmed | Tested account/connection; raw authentication output not retained |
| E15 | B | Final target UFW screenshot directly inspected: active, incoming deny, one SSH allow rule | Final displayed configuration; original image not retained |

## E01–E02: retained Nmap images

The public PNG copies have ancillary text and EXIF metadata removed. Their image-data chunks are unchanged; no displayed output was edited, reconstructed or redacted. The unmodified source downloads are kept outside this repository.

Command visible in both screenshots:

```text
nmap -sT -Pn -p 22,8000,8443 10.10.10.20
```

The assessment was run from `ubuntu-security`. Nmap version **7.98** is visible. Service names such as `http-alt` and `https-alt` are labels in this scan, not independent application identification.

| Screenshot | Recorded result |
|---|---|
| [Before rule cleanup](01-nmap-before-rule-cleanup.png) | `22/tcp filtered ssh`; `8000/tcp closed http-alt`; `8443/tcp closed https-alt` |
| [After rule cleanup](02-nmap-after-rule-cleanup.png) | `22/tcp filtered ssh`; `8000/tcp filtered http-alt`; `8443/tcp filtered https-alt` |

E01 also contains `sudo ufw status numbered` with `Status: inactive` on the **assessment VM**. It must not be used to describe `ubuntu-target`.

[SHA256SUMS.txt](SHA256SUMS.txt) records the SHA-256 digest of each retained image. These digests help detect later file changes; they do not independently authenticate when or how an image was captured.

## E03: target listener output

Selected fields transcribed from the participant's pasted `sudo ss -ltnp` output; queue values, process IDs and file descriptors omitted:

```text
State   Local Address:Port   Process
LISTEN  0.0.0.0:22           sshd / systemd
LISTEN  127.0.0.53%lo:53     systemd-resolve
LISTEN  127.0.0.54:53        systemd-resolve
LISTEN  [::]:22             sshd / systemd
```

No TCP 8000 or 8443 listener appeared in the supplied listing. This says nothing about UDP, startup configuration or a later execution state.

## E04–E06: permissions and final management access

The participant confirmed that the private directory and file had the requested modes, that an attempted read as `lab_auditor` failed with permission denied, and that `student` could read the synthetic report. The reported path inspection was:

```text
/                                      755
/home                                  755
/home/student                          750
/home/student/company-private          700
/home/student/company-private/internal-report.txt  600
```

This is a normalized summary of the participant's description, not a raw `namei` capture. The home directory already restricted other-user traversal. After firewall cleanup, the participant separately confirmed that a new SSH session produced the expected lab username and hostname. No claim is made that these confirmations were fresh tests on the review date.

## E07–E08: HTTP exposure and retest

Both source screenshots were visually inspected at the exact URL:

```text
http://10.10.10.20:8000/reports/internal-report.txt
```

Before relocation, the browser showed:

```text
CONFIDENTIAL - Simulated Company Report
```

After the recorded move from `~/lab-web/reports/` to `~/company-private/`, it showed:

```text
Error response
Error code: 404
Message: File not found.
Error code explanation: 404 - Nothing matches the given URI.
```

The images were not successfully retained in the package. These are manual transcriptions of inspected screenshots, not generated replacement screenshots. The 404 result verifies only the tested retrieval path at that time.

## E09–E10: HTTP response and TLS tooling

The participant pasted the result of a Python HTTP request to `http://10.10.10.20:8000/?demo=1`:

```text
200
Welcome to CyberSecurity Lab
```

The participant also reported being able to see content and other details in the HTTP traffic capture. The raw HTTP capture was attached separately in the historical record and was not recovered during this review. Quotations of it in prior assistant explanations are not promoted to direct evidence here.

Pasted tooling output identified `OpenSSL 3.5.5 27 Jan 2026`. The `python3 -m http.server --help` output included `--tls-cert`, `--tls-key` and `--tls-password-file`. This supports availability of the demonstrated server options. No exact Ubuntu release or full Python version is inferred from these excerpts.

## E11: temporary target firewall rules

Normalized transcription of participant-pasted `sudo ufw status numbered` output during TLS testing:

```text
Status: active
[1] 22/tcp on enp0s2    ALLOW IN  10.0.2.2
[2] 8000/tcp on enp0s1  ALLOW IN  10.10.10.10
[3] 8443/tcp on enp0s1  ALLOW IN  10.10.10.10
```

This is the intermediate rule state, not the final configuration.

## E12–E13: HTTPS and captured traffic

The participant reported opening the HTTPS page after accepting the self-signed certificate warning. That is a recorded lab action, not a recommendation to bypass certificate validation in normal use.

The following capture command and its resulting pasted text were inspected:

```text
sudo tcpdump -i enp0s1 -nn -A 'host 10.10.10.20 and tcp port 8443'
```

The text showed traffic between `10.10.10.10:59108` and `10.10.10.20:8443`, including SYN, SYN-ACK and ACK at approximately `16:59:08`, followed by payloads rendered mostly as non-readable ASCII. The reviewed excerpt contained no visible HTTP `GET` line, response status or `Welcome to CyberSecurity Lab` body. It did contain readable strings such as `h2` and `http/1.1`, consistent with visible protocol-negotiation data.

This record is ASCII-rendered packet output, not a retained `.pcap` file or a decoded TLS session report. It does not establish a particular TLS version, cipher, certificate chain, complete capture coverage or authenticated server identity. The raw dump is not republished; only this bounded observation is retained.

## E14: SSH authentication tests

The participant explicitly confirmed these effective settings after the configuration checks:

```text
PubkeyAuthentication yes
PasswordAuthentication no
KbdInteractiveAuthentication no
```

They then confirmed both outcomes: an explicit-key connection to `student@127.0.0.1` on forwarded TCP 2222 succeeded, while a password-only attempt with public-key authentication disabled returned `Permission denied (publickey)`. These statements are participant confirmations, not independently inspected raw command output. They support the tested connection; they do not audit all users, `Match` contexts or authentication methods. No private key or actual credential is included.

## E15: final target UFW configuration

The final source screenshot was visually inspected on review. It showed `sudo ufw status verbose` and `sudo ufw status numbered` in the `student@ubuntu-target` terminal. Normalized transcription:

```text
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), disabled (routed)
New profiles: skip

To                    Action    From
22/tcp on enp0s2       ALLOW IN  10.0.2.2

Status: active
[1] 22/tcp on enp0s2   ALLOW IN  10.0.2.2
```

The image download did not complete, so the original is not included. This is a transcription of directly inspected evidence. No TCP 8000 or 8443 allow rule appeared in the displayed final list. This configuration observation and the retained E02 scan support the recorded cleanup; neither is a fresh check of current VM state.
