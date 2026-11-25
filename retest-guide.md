# Retest guide

Use these steps only in the owned UTM lab. They inspect the selected controls and perform bounded connection checks; they do not change firewall rules, permissions or server configuration. Record the date, source VM, target address, command and actual output for each check. Historical results are a comparison point, not an expected output to copy into evidence.

## 1 Check the target locally

In the `ubuntu-target` console or its existing administrative session:

```bash
date -Is
hostname
ip -brief address
sudo ss -ltnp
sudo ufw status verbose
sudo ufw status numbered
```

`ss -ltnp` lists listening TCP sockets and their processes. It does not test remote reachability or UDP. `ufw status verbose` shows UFW's status and default policies; `status numbered` lists its user-facing rules. Record the interface, source restriction and destination port of the SSH rule exactly as shown. Do not delete a rule while collecting evidence.

The recorded target listener output contained SSH and loopback DNS, without a TCP listener on 8000 or 8443. A different current result should be investigated and documented rather than rewritten to match the historical record.

Review `ip` output before publication: retain the lab addresses needed to explain the test, and remove unrelated addresses or host identifiers from any public copy.

## 2 Check the assessment path

In `ubuntu-security`:

```bash
date -Is
nmap -sT -Pn -p 22,8000,8443 10.10.10.20
```

This uses TCP connect scanning (`-sT`), skips host discovery (`-Pn`) and checks only the three named ports. The recovered final result was `filtered` for all three.

| State | What this observation supports |
|---|---|
| `open` | A service accepted connections on that tested port/path at that time. |
| `closed` | The port responded as closed; no connection-accepting service was observed there. |
| `filtered` | The probes did not allow Nmap to determine whether the port was open or closed. |

Compare this result with the local listener and firewall outputs. A `filtered` SSH port can coexist with a running SSH service and successful management access through a different permitted path.

## 3 Check that administrative SSH still works

Open a new terminal on the UTM host. Use the existing lab key locally; never copy its contents into an evidence file or upload it.

```bash
ssh -i ~/.ssh/ubuntu_target_lab_ed25519 \
  -o IdentitiesOnly=yes \
  -o PreferredAuthentications=publickey \
  -p 2222 student@127.0.0.1
```

If the key requires a passphrase, enter it privately in the terminal. In the new session:

```bash
whoami
hostname
exit
```

The recorded identities were `student` and `ubuntu-target`. Success demonstrates management access using a public key on this connection. It does **not** prove that password and keyboard-interactive authentication are disabled on the server.

To inspect SSH's effective baseline configuration on the target:

```bash
sudo sshd -T
```

Review `pubkeyauthentication`, `passwordauthentication`, `kbdinteractiveauthentication`, `authenticationmethods` and `permitrootlogin`. Preserve only the relevant lines in public evidence. Configuration using `Match` blocks needs a connection-specific evaluation with the actual client address and account; the baseline output alone must not be described as applying to every possible connection. A separate negative authentication test is necessary if the report claims password-based access was rejected.

The participant previously confirmed the first three settings as `yes`, `no` and `no`, respectively, and a password-only attempt rejected with `Permission denied (publickey)`. For a bounded repeat of that negative test from the host:

```bash
ssh -o PubkeyAuthentication=no \
  -o PreferredAuthentications=password,keyboard-interactive \
  -o NumberOfPasswordPrompts=0 \
  -p 2222 student@127.0.0.1
```

Record the complete response. `Permission denied (publickey)` supports that the tested connection offered only public-key authentication. A generic failure, timeout or refusal does not prove password authentication is disabled. This command supplies no password and does not change server settings.

## 4 Check the private report boundary

On `ubuntu-target`:

```bash
namei -l /home/student/company-private/internal-report.txt
stat -c '%a %U %G %n' /home/student/company-private
stat -c '%a %U %G %n' /home/student/company-private/internal-report.txt
id lab_auditor
sudo -u lab_auditor test -r /home/student/company-private/internal-report.txt
echo "auditor_read_check_exit=$?"
test -r /home/student/company-private/internal-report.txt
echo "current_user_read_check_exit=$?"
```

`test -r` checks readability without printing file contents. Exit status `0` means readable; nonzero means the check failed. Run the current-user check as `student`, not as root. If `lab_auditor` does not exist, record that limitation; do not create or alter accounts merely to complete this read-only collection.

The historical report described directory mode `700`, file mode `600`, auditor access denied and owner access allowed. Examine **all parent directories** before attributing a denial to a specific mode change. The existing restriction on `/home/student` is material to the interpretation.

## 5 Review historical HTTP and TLS evidence

The temporary HTTP and HTTPS services were no longer listening in the recovered final target listing. Do not interpret a failed request to stopped services as proof that a particular URL was remediated or that TLS is configured correctly.

For the HTTP exposure finding, a complete evidence chain identifies the actual served directory, a successful pre-remediation request to the simulated private content, the move outside that directory, and a repeat request while the service is running that no longer returns the content.

The historical before/after screenshots were directly inspected at `http://10.10.10.20:8000/reports/internal-report.txt`: simulated confidential content appeared before relocation, and `404 File not found` afterward. Those images were not retained, so any new screenshot should be labelled as a new test rather than a replacement for an original.

For the plaintext/TLS comparison, distinguish application content visible in an HTTP capture from encrypted TLS application records. Record the capture interface, endpoints, protocol and test request. A readable TLS handshake or certificate is not evidence that the application payload is plaintext. A self-signed certificate and any use of `curl -k` must be disclosed: bypassing verification does not demonstrate trusted server identity.

The inspected TLS tcpdump excerpt showed traffic between `10.10.10.10` and `10.10.10.20:8443`, binary payloads and readable handshake strings such as `h2` and `http/1.1`, without a readable HTTP request, response status or page body. This excerpt does not establish a specific TLS version, cipher suite or trusted identity. The HTTP client response was inspected directly; plaintext visibility in the HTTP capture remains participant-reported because that original capture text was not recovered.

Use only reviewed historical captures or a separately planned controlled rerun. Do not publish raw packet captures without inspecting their complete contents for credentials, unrelated traffic and personal data.

## Evidence record

For each result, retain the observation and its limitations together:

```text
Evidence ID:
Observation date and timezone:
Source VM or management client:
Target address and port:
Command:
Actual output:
Finding or control tested:
Interpretation:
Limitations:
Redactions in the public copy:
```

Keep originals privately. Public evidence should contain only the material required to support the finding. Do not replace an original screenshot with a reconstructed terminal image or present an expected result as captured output.
