Local Network Security Scan 
This repository contains the results of a security scan performed on a local /24 network. The objective was to identify active hosts, discover open ports, and assess potential security vulnerabilities.

Key Findings
The scan identified a total of 4 active hosts on the network.

High-Risk Vulnerabilities
Router (192.168.1.1):
Telnet (Port 23) is OPEN: This is a major security risk as Telnet transmits credentials and data in unencrypted plaintext.
Unknown Service (Port 5555) is OPEN: An unidentified service (freeciv) is running, which requires immediate investigation.
Scanning Machine (192.168.1.3):
SMB (Ports 139, 445) is OPEN: Exposing file and printer sharing services increases the risk of unauthorized access and lateral movement attacks within the network.

Full Report
For a complete breakdown of all open ports, discovered hosts, and detailed hardening recommendations, please see the Network_Scan_Results.txt file.


Every Day New Tasks are Uploaded in The same Repository
