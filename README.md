# Enterprise SIEM Lab: Brute-Force Detection & SIEM

## Objective

The goal of this project was to build an isolated virtual network from scratch then use it to simulate a real brute-force attack against a Windows endpoint with Splunk on Ubuntu Server serving as the central SIEM for log collection and analysis. Kali Linux was used for mimicking a brute force attack on the Windows endpoint.

Core focus areas were SIEM deployment and configuration (Splunk), collecting logs using Splunk Universal Forwarder on Windows, attacking via Kali using Hydra gaining practical experience operating on both sides of an attack.

---

## Tools & Technologies

| Role | Tool / Technology |
|---|---|
| Defence SIEM Setup | Splunk Enterprise (on Ubuntu Server 26.04) |
| User Endpoint (Victim VM) | Windows 10 with Splunk Universal Forwarder |
| Attacker Machine | Kali Linux |
| Attack Tools | Hydra, Nmap |

---

## Architecture & Environment Setup

The home lab runs on VMware Workstation on a fully isolated network consisting of three virtual machines each having a specific role:

- **Win10 (Target)** — The simulated victim machine. The Splunk Universal Forwarder was installed and configured via `inputs.conf` to monitor the Windows Security Event Log.
- **Ubuntu (SIEM / Splunk)** — Hosts the Splunk Enterprise instance and is configured to receive forwarded log data on port `9997`.
- **Kali (Attacker)** — Serves as the attack machine, the origin point of all attack activity during the simulation, including Nmap scans and Hydra brute force.

---

## The Attack Simulation

RDP was enabled on the Windows 10 target to replicate a common and realistic exposure found across corporate environments. A custom wordlist consisting of frequently used usernames and weak passwords was created on the VM and used to launch a credential brute-force attack against port `3389` using Hydra.

The objective was to:

1. **Show failed logon attempts** — generate a high volume of failed authentication attempts that would produce a clear, detectable pattern within the Windows Security Event Log.
2. **Show a successful login attempt** — and to understand what the usual Event ID codes are for both successful and failed attempts.

### Brute-Force Execution

```bash
# Failed attempt
hydra -L users.txt -P passwords.txt rdp://10.0.2.x

# Successful attempt
hydra -L real_users.txt -P real_pass.txt rdp://10.0.2.x
```

The attack produces a flood of **Event ID 4625** entries (failed logon attempts), all originating from a single source IP within a narrow time window. It is precisely this pattern that a SIEM should surface and what the detection and analysis phase of this lab was designed to validate.

The attack later produces a successful **Event ID 4624** login attempt after using valid credentials in txt files used for Hydra brute force.
