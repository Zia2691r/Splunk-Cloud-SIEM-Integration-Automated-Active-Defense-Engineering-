# Splunk Cloud SIEM Integration and Automated Active Defense Engineering

## Project Overview

This project demonstrates the integration of Splunk Cloud SIEM with Fail2ban and IPTables to build an automated threat detection and response workflow. The lab was designed to simulate a real-world Security Operations Center (SOC) environment where security events are collected, analyzed, and acted upon automatically.

The project focuses on detecting SSH brute-force attacks against an Ubuntu system, forwarding authentication logs to Splunk Cloud for analysis, and automatically blocking malicious IP addresses using Fail2ban.

---

## Project Information

**Project Title:** Splunk Cloud SIEM Integration and Automated Active Defense Engineering

**Project Type:** Independent Cybersecurity Engineering Project

**Author:** Mohammed Zia Ur Rahman

**Role:** Cybersecurity Researcher | SOC Analyst Enthusiast

**Domain:** Cybersecurity and Security Operations

**Implementation Year:** 2026

---

## Objectives

The main objectives of this project were:

- Build a centralized log ingestion pipeline using Splunk Cloud.
- Simulate brute-force attacks against an SSH service.
- Analyze authentication logs using SIEM techniques.
- Implement automated threat detection and response.
- Configure Fail2ban to dynamically block attackers.
- Validate incident response and remediation procedures.

---

## Technologies Used

### SIEM Platform
- Splunk Cloud Platform

### Operating Systems
- Ubuntu Desktop 24.04 LTS
- Kali Linux
- Windows Host System

### Security Tools
- Fail2ban
- IPTables
- OpenSSH Server
- THC Hydra

### Virtualization
- Oracle VirtualBox

---

## Lab Environment

The project was built in an isolated virtualized environment consisting of:

### Attacker Machine
- Kali Linux Virtual Machine
- Used to launch brute-force attacks using Hydra

### Target Machine
- Ubuntu Desktop 24.04 LTS
- Hosted the SSH service
- Generated authentication logs

### Host System
- Windows Operating System
- Used as a staging layer for log transfer and Splunk Cloud access

### SIEM Platform
- Splunk Cloud Sandbox Instance
- Used for log ingestion, analysis, and visualization

---

## Architecture Workflow

The workflow of the project is shown below:

1. Kali Linux launches a brute-force attack against the Ubuntu SSH service.
2. Failed login attempts are recorded in the Ubuntu authentication logs.
3. Fail2ban continuously monitors authentication logs.
4. When the configured threshold is exceeded, Fail2ban automatically creates firewall rules using IPTables.
5. The attacker's IP address is blocked.
6. Authentication logs are copied from Ubuntu to the Windows host.
7. Logs are uploaded to Splunk Cloud.
8. Splunk is used to analyze and visualize attack activity.

---

## Challenges Encountered

### Ubuntu Browser Failure

The Firefox browser inside the Ubuntu virtual machine stopped functioning properly, preventing direct access to Splunk Cloud.

#### Solution

A staging workflow was created:

- Authentication logs were copied to a shared folder.
- The Windows host accessed the logs.
- The logs were uploaded to Splunk Cloud from Windows.

---

### SSH Service Not Running

Initial attack attempts failed because Ubuntu Desktop did not have an active SSH server.

#### Solution

The OpenSSH Server package was installed and configured manually.

```bash
sudo apt update
sudo apt install -y openssh-server

sudo systemctl enable --now ssh
sudo systemctl status ssh
```

---

### Splunk Search Returning No Results

After uploading logs to Splunk Cloud, searches and visualizations were not producing results.

#### Cause

Splunk Smart Mode did not automatically extract fields from the uploaded log file.

#### Solution

The search mode was changed from Smart Mode to Verbose Mode, allowing Splunk to perform search-time field extraction.

---

## Phase 1: Preparing the Target System

The Ubuntu system was configured to accept SSH connections.

### Install OpenSSH Server

```bash
sudo apt update
sudo apt install -y openssh-server

sudo systemctl enable --now ssh
sudo systemctl status ssh
```

This enabled the SSH service and opened port 22 for remote connections.

---

## Phase 2: Simulating a Brute-Force Attack

Hydra was used from the Kali Linux virtual machine to perform a dictionary attack against the SSH service.

### Hydra Command

```bash
hydra -l ziaurrahman -P /usr/share/wordlists/rockyou.txt ssh://192.168.1.9 -t 4 -V
```

### Outcome

The attack generated a large number of failed authentication attempts within a short period of time.

These events were recorded in:

```bash
/var/log/auth.log
```

---

## Phase 3: Log Extraction and Splunk Ingestion

Authentication logs were copied from the Ubuntu machine and prepared for upload.

### Copy Authentication Logs

```bash
rm -f ~/attack_logs.txt

sudo cp /var/log/auth.log ~/attack_logs.txt
```

### Modify Permissions

```bash
sudo chmod 666 ~/attack_logs.txt
```

The file was then transferred through a VirtualBox shared folder and uploaded to Splunk Cloud.

### Splunk Upload Configuration

Source Type:

```
linux_secure
```

Category:

```
Operating System
```

Destination Index:

```
Default Index
```

---

## SIEM Analysis

### Search Query

```spl
index=* sourcetype=linux_secure "Failed password"
```

This query was used to identify failed SSH login attempts.

### Visualization Query

```spl
index=* sourcetype=linux_secure "Failed password"
| timechart count
```

### Findings

The visualization showed a significant spike in failed authentication events occurring within a very short period of time.

This confirmed the presence of an automated brute-force attack.

---

## Active Defense Implementation

Fail2ban was deployed to automatically detect and block attackers.

### Install Fail2ban

```bash
sudo apt update
sudo apt install -y fail2ban
```

### Configure Fail2ban

Create:

```bash
/etc/fail2ban/jail.local
```

Configuration:

```ini
[sshd]
enabled = true
port = ssh
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
findtime = 600
bantime = 3600
```

### Apply Configuration

```bash
sudo systemctl restart fail2ban
```

---

## Defensive Configuration

### maxretry

```text
3
```

The attacker is allowed only three failed login attempts.

### findtime

```text
600 seconds
```

The failures are monitored within a ten-minute window.

### bantime

```text
3600 seconds
```

The offending IP address is blocked for one hour.

---

## Testing the Defense Mechanism

After configuring Fail2ban, the Hydra attack was launched again.

### Observed Results

- Login attempts slowed significantly.
- Hydra became ineffective.
- The attacker's IP address was blocked automatically.
- The SSH service remained protected.

---

## Verifying the Ban

The status of the Fail2ban jail was checked using:

```bash
sudo fail2ban-client status sshd
```

The output confirmed that one IP address had been banned.

---

## Remediation and Cleanup

After verification, the blocked IP address was removed from the blacklist.

### Unban Command

```bash
sudo fail2ban-client set sshd unbanip 192.168.1.8
```

The command returned:

```text
1
```

This confirmed successful removal of the IP address from the ban list.

---

## Key Skills Demonstrated

### Security Information and Event Management (SIEM)

- Log ingestion
- Data source configuration
- Event analysis
- Search optimization
- Visualization creation

### Threat Detection and Monitoring

- Brute-force attack detection
- Authentication log analysis
- Security event investigation

### Active Defense Engineering

- Fail2ban configuration
- Automated response workflows
- Dynamic firewall rule management

### Linux Administration

- OpenSSH configuration
- Log management
- System service management
- IPTables integration

### Virtualization and Lab Design

- VirtualBox networking
- Multi-system lab deployment
- Log transfer workflows

---

## Project Outcomes

This project successfully demonstrated:

- Secure log collection and ingestion into Splunk Cloud.
- Detection of brute-force authentication attacks.
- SIEM-based investigation and visualization of security events.
- Automated IP blocking using Fail2ban and IPTables.
- Validation of incident response and recovery procedures.

The project provided practical experience in SIEM operations, log analysis, threat detection, active defense engineering, and SOC-style incident response workflows.

---

## Screenshots

Add all project screenshots inside a folder named:

```
screenshots
```

Example:

```text
project-folder/
│
├── README.md
│
└── screenshots/
    ├── architecture.png
    ├── hydra_attack.png
    ├── splunk_upload.png
    ├── splunk_search.png
    ├── visualization.png
    ├── fail2ban_config.png
    ├── fail2ban_ban.png
    └── unban_process.png
```

Then reference them in GitHub like:

```markdown
## Architecture

![Architecture]
![image alt](https://github.com/Zia2691r/Splunk-Cloud-SIEM-Integration-Automated-Active-Defense-Engineering-/blob/d05115f3001acb312614619376272ff2f82dc36f/config/screenshots/screenshotsarchitecture.png)

## Hydra Attack

![Hydra Attack](screenshots/hydra_attack.png)

## Splunk Visualization

![Visualization](screenshots/visualization.png)

## Fail2ban Configuration

![Fail2ban Configuration](screenshots/fail2ban_config.png)
```

---
