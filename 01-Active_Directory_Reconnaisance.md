# Black-Box Penetration Testing: Active Directory Reconnaissance

## 1. Network Discovery

The first step in a Black-Box penetration test is network discovery. During this phase, we identify reachable hosts, exposed ports, and potentially vulnerable services.

Although Nessus and other vulnerability scanners can help identify exposed services and known vulnerabilities, Nmap is an excellent starting point. It is also included by default in Kali Linux.

**TCP Port Discovery:**

```bash
sudo nmap -sS -p- --open -n -Pn <IP> -oN allPorts
```

**Command breakdown:**

- `-sS`: Performs a TCP SYN scan.
- `-p-`: Scans all TCP ports (1–65535).
- `--open`: Displays open or potentially open ports.
- `-n`: Disables DNS resolution.
- `-Pn`: Skips host discovery and treats the target as online.
- `-oN allPorts`: Saves the results in normal output format.

The next steps depend on the discovered ports, service versions, and available information.

In many Windows-based enterprise environments, SMB (TCP/445) is commonly exposed on domain controllers and other domain-joined systems. This service should receive particular attention during the enumeration phase.

## 2. Username Enumeration and OSINT

Before conducting authentication security assessments, we should establish a list of valid usernames.

One approach is to perform Open Source Intelligence (OSINT) research. Publicly available information, such as corporate websites, LinkedIn profiles, and employee directories, can help identify potential username patterns and organizational email conventions.

However, publicly identified names should be considered potential usernames until their validity has been confirmed.

### Identifying the Domain Name

NetExec can help enumerate information exposed through SMB services.

Example:

```bash
netexec smb <DOMAIN_CONTROLLER_IP>
```

Depending on the server configuration and the information exposed, the output may provide useful details about the target, including its domain and operating system.

## 3. Kerberos Username Enumeration with Kerbrute

After building an initial username list, we can use Kerbrute to validate potential domain accounts through Kerberos.

**Repository:**

https://github.com/ropnop/kerbrute

### Installation

Clone the repository:

```bash
git clone https://github.com/ropnop/kerbrute.git
cd kerbrute
```

Install the required Go development tools and compile the project following the repository's instructions.

Alternatively, download a precompiled binary from the project's official GitHub releases.

### User Enumeration

Example:

```bash
./kerbrute_linux_amd64 userenum \
  --dc x.x.x.x \
  -d example.local \
  users.txt
```

**Parameters:**

- `userenum`: Executes the username enumeration function.
- `--dc`: Specifies the Domain Controller IP address.
- `-d`: Specifies the Active Directory domain.
- `users.txt`: Contains the candidate usernames.

The output identifies usernames that Kerbrute successfully validates through Kerberos responses.

Keep in mind that enumeration results should be interpreted in the context of the domain configuration and Kerberos behavior.

## 4. Password Security Assessment

Password spraying is one possible technique for evaluating weak credentials in Active Directory environments.

Unlike traditional brute forcing, password spraying tests a small number of candidate passwords against multiple accounts rather than trying numerous passwords against a single account.

Before performing this assessment, it is essential to understand the organization's password policy and account lockout configuration.

Consider the following factors:

- Minimum password length.
- Complexity requirements.
- Password expiration policies.
- Account lockout thresholds.
- Authentication monitoring and security controls.

A password such as `Enterprise2026!` illustrates a common organizational password pattern: a company-related term, a year, and a special character. However, this is only an example and should not be interpreted as evidence that a particular organization uses it.

## 5. Kerberos Attack Surface Assessment

Kerberos provides several opportunities for security assessment in Active Directory environments.

Two important techniques are Kerberoasting and AS-REP Roasting.

### Kerberoasting

Kerberoasting involves requesting service tickets (TGS) for accounts associated with Service Principal Names (SPNs).

The resulting ticket material may allow offline password-cracking attempts, depending on the encryption type and password strength.

Impacket provides the `GetUserSPNs.py` script for identifying potentially vulnerable service accounts and requesting their service tickets.

### AS-REP Roasting

AS-REP Roasting targets accounts configured without the Kerberos pre-authentication requirement.

In these circumstances, an attacker may obtain AS-REP response material that can be subjected to offline password-cracking attempts.

These two techniques target different Kerberos configurations and should not be confused.

## 6. NTLM Assessment with Responder

Responder is a tool used to assess exposure to LLMNR, NBT-NS, and other name-resolution and authentication-related mechanisms.

In certain network configurations, it can capture NTLM challenge-response material when clients attempt to authenticate to an attacker-controlled service.

The captured NetNTLMv1 or NetNTLMv2 material may be analyzed offline using password-cracking tools such as:

- Hashcat.
- John the Ripper.

**Important distinction:** Captured NetNTLM challenge-response material is not equivalent to obtaining the user's NT password hash. Its recovery depends on factors such as password complexity, protocol configuration, and available computational resources.

## 7. Conclusion

Active Directory reconnaissance in a Black-Box penetration test involves several complementary techniques.

Network discovery, service enumeration, OSINT, username validation, authentication security assessments, and Kerberos analysis help build an understanding of the organization's exposed attack surface.

The objective is not simply to execute tools or collect information, but to identify potential attack paths, validate security weaknesses, and evaluate their impact within the authorized assessment scope.
