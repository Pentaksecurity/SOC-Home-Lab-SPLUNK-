SOC Home Lab: Attack Simulation & Splunk Telemetry Analysis
Objective

The purpose of this lab was to simulate a cyber intrusion in a controlled environment, generate telemetry data, and analyze the resulting indicators of compromise (IOCs) using Splunk. By replaying a simple attack scenario, we demonstrate how defenders can trace malicious activity, investigate attacker actions, and validate detection capabilities.

1. Lab Environment

Two virtual machines were configured in VirtualBox on an Internal Network (no external access):

Attacker: Kali Linux – 192.168.20.11

Victim: Windows 11 – 192.168.20.10

This setup ensured that communication occurred only between the two hosts, isolating the test environment from external networks.

<img width="709" height="207" alt="VM Networking" src="https://github.com/user-attachments/assets/668b4917-04fa-4a43-9562-7c96813aee9d" />
2. Attack Simulation
Reconnaissance

The attacker initiated a scan of the victim using Nmap, identifying open ports including 3389 (RDP) — a potential entry point for exploitation.

<img width="1516" height="597" alt="Nmap Scan" src="https://github.com/user-attachments/assets/4e8329b4-bd40-4a29-8a08-ccd77c42f4ca" />
Weaponization

A malicious payload was generated with msfvenom, disguised as a seemingly legitimate file:
Invoice.pdf.exe.

<img width="1111" height="183" alt="Payload Creation" src="https://github.com/user-attachments/assets/15aae95a-76a6-44cb-b1e4-76f23076eb9d" />
Delivery

The payload was hosted on a simple Python HTTP server, simulating a phishing delivery method.

python3 -m http.server 9999

<img width="673" height="178" alt="Payload Delivery" src="https://github.com/user-attachments/assets/a258fafa-ae2e-4a38-8550-86fe43a99b8f" />
Exploitation

The attacker configured a Meterpreter reverse TCP listener. Once the victim executed the payload, a reverse shell session was established.

<img width="1080" height="848" alt="Meterpreter Shell" src="https://github.com/user-attachments/assets/bad57630-606b-4831-9572-b78c581eda2c" /> <img width="646" height="597" alt="Shell Commands" src="https://github.com/user-attachments/assets/60d7be8a-f038-41ae-a7ff-a49b5f246459" />

To generate forensic artifacts, commands such as net user, ipconfig, and net localgroup were executed within the session. These actions would later be traced in Splunk.

3. Victim System Observation

On the Windows host, the following was observed:

Netstat (netstat -anob) revealed an active connection from the Kali attacker on port 4444.

<img width="1139" height="559" alt="Netstat Connection" src="https://github.com/user-attachments/assets/54e07abf-3133-44a4-89b0-af9a43e54053" />

The associated Process ID (PID 6648) was traced in Task Manager and linked to Invoice.pdf.exe.

<img width="1066" height="541" alt="Task Manager Process" src="https://github.com/user-attachments/assets/18728f15-6f9f-4259-9a5b-1caec880de4b" />

This validated that the malicious executable had successfully established persistence on the victim.

4. Splunk Telemetry & IOC Investigation

Windows Sysmon logs were ingested into Splunk via a custom endpoint index.

Key Findings:

Searching for the attacker IP revealed execution of Invoice.pdf.exe.

<img width="1024" height="768" alt="Splunk IP Search" src="https://github.com/user-attachments/assets/1d06219d-4b1e-49c6-852a-fea599da6c1b" />

Searching specifically for Invoice.pdf.exe confirmed suspicious execution.

<img width="1024" height="768" alt="Splunk File Search" src="https://github.com/user-attachments/assets/6d1b4a17-c322-4626-a71e-deb0d4f4afbb" />

Sysmon revealed that Invoice.pdf.exe spawned cmd.exe — a strong indicator of malicious behavior.

<img width="1024" height="768" alt="Splunk CMD Spawn" src="https://github.com/user-attachments/assets/56d764be-72e3-4ac0-803c-9c8bb199a19c" />

By pivoting on the Process GUID, we reconstructed the full process chain and attacker commands with:

| table _time, ParentImage, Image, CommandLine

<img width="1024" height="768" alt="Splunk Process GUID" src="https://github.com/user-attachments/assets/9a9389fa-b083-417f-9056-19a6bca12478" /> <img width="1024" height="768" alt="Splunk Command History" src="https://github.com/user-attachments/assets/edac3b64-abe1-4a5c-9aca-e38c712addb7" />

This confirmed the commands executed from the Meterpreter shell, validating that Splunk successfully captured attacker behavior.

5. Conclusion

This exercise demonstrated the full attack lifecycle and the defensive visibility achieved through Splunk + Sysmon:

Telemetry Creation: Attacker actions generated detectable artifacts.

IOC Identification: Suspicious process execution (Invoice.pdf.exe spawning cmd.exe) was flagged.

Attack Tracing: Process lineage and command history were reconstructed via Splunk queries.

Defensive Application: Analysts can use these techniques to recognize, investigate, and respond to real-world intrusions.

This lab highlights how defenders can not only identify malicious activity but also map attacker behavior to the intrusion kill chain — enabling effective detection and incident response.
