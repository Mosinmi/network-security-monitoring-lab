# Network Traffic Analysis

## 1. Overview

This document presents the analysis performed on network traffic captured using Wireshark in a controlled network security monitoring laboratory.

The analysis focused on TCP behavior, service exposure, DNS activity, HTTP traffic, HTTPS/TLS communication, network endpoints, and TCP retransmissions.

All traffic was generated within an authorized laboratory environment.

---

## 2. Analysis Environment

| Component | Description |
|---|---|
| Monitoring Tool | Wireshark 4.6.8 |
| Operating System | Ubuntu Server 22.04.5 LTS |
| Client System | Windows |
| Ubuntu IP | `192.168.0.162` |
| Windows IP | `192.168.0.136` |
| Network | Local bridged network |
| Primary Protocols | TCP, SSH, DNS, HTTP, TLS |

---

## 3. Protocol Hierarchy Analysis

The `recon-analysis.pcap` capture contained 31 packets.

Wireshark identified the following protocol hierarchy:

| Protocol | Packets | Percentage of Packets | Bytes | Percentage of Bytes |
|---|---:|---:|---:|---:|
| IPv4 | 31 | 100% | 620 | 27.8% |
| TCP | 31 | 100% | 948 | 42.5% |
| SSH | 1 | 3.2% | 42 | 1.9% |
| HTTP | 2 | 6.5% | 166 | 7.5% |

The capture was entirely based on IPv4/TCP traffic at the network and transport layers.

The application-layer traffic identified by Wireshark included SSH and HTTP.

**Evidence:** `screenshots/07-protocol-hierarchy.png`

---

## 4. IPv4 Endpoint Analysis

The IPv4 endpoint statistics identified three addresses in the capture:

| IP Address | Packets | Bytes | Observed Role |
|---|---:|---:|---|
| `192.168.0.162` | 31 | ~2 KB | Ubuntu server |
| `192.168.0.136` | 22 | ~1 KB | Windows client |
| `91.189.91.57`  | 9  | 776 bytes | External endpoint |

The majority of the observed traffic involved the Windows client and Ubuntu server.

The external endpoint `91.189.91.57` was also observed communicating with the Ubuntu host during the capture.

**Evidence:** `screenshots/08-ipv4-endpoints.png`

---

## 5. TCP Service Analysis

TCP traffic was analyzed to identify connection attempts against services on the Ubuntu server.

The captured traffic included connection attempts to:

- TCP/22
- TCP/23
- TCP/80
- TCP/443

TCP/22 produced a response from the Ubuntu server and was subsequently associated with SSH traffic.

The other tested ports produced repeated SYN transmissions without a corresponding successful TCP handshake in the captured traffic.

This observation is based only on the captured traffic and does not by itself establish the reason for the lack of response.

**Evidence:** `screenshots/03-tcp-conversations.png`

---

## 6. TCP SYN Retransmission Analysis

The following Wireshark display filter was used:

```text
tcp.analysis.retransmission

Repeated SYN retransmissions were observed for connections directed toward:

192.168.0.162:23
192.168.0.162:80
192.168.0.162:443

The capture also showed a successful TCP response for the SSH service on port 22.

Repeated SYN retransmissions can indicate that a destination is not responding, is filtered, or is otherwise unreachable from the source. The packet capture alone does not establish the exact reason for the retransmissions.

Evidence: screenshots/09-tcp-retransmissions.png

7. SSH Traffic Analysis

The SSH capture demonstrated communication between:

192.168.0.136 → 192.168.0.162:22

The capture contained:

SSH protocol identification
SSH version exchange
Key Exchange Init messages
Elliptic Curve Diffie-Hellman key exchange
New Keys messages
Encrypted packets

The observed SSH traffic demonstrates the transition from protocol negotiation and key exchange to encrypted communication.

The packet capture also showed the SSH software identification strings:

SSH-2.0-OpenSSH_for_Windows_9.5
SSH-2.0-OpenSSH_8.9p1 Ubuntu-3ubuntu0.17

Evidence: SSH analysis capture and corresponding Wireshark screenshots.

8. DNS Traffic Analysis

DNS traffic was captured and examined using Wireshark.

DNS queries and corresponding responses were identified between the Ubuntu host and its DNS resolver.

DNS traffic provides useful network-monitoring visibility because it can reveal:

Domain resolution activity
External destinations requested by a host
Connectivity checks
Application-related DNS activity

DNS activity should be interpreted in context. The presence of a domain in a capture does not by itself establish that the domain is malicious.

Evidence: screenshots/02-dns-analysis.png

9. HTTP Traffic Analysis

The HTTP capture demonstrated readable HTTP request and response traffic.

The capture included:

GET / HTTP/1.1

and:

HTTP/1.1 200 OK

The packet contents allowed HTTP protocol information to be inspected directly in Wireshark.

This demonstrates the reduced confidentiality provided by unencrypted HTTP traffic compared with HTTPS communication.

Evidence: screenshots/05-http-request.png

10. HTTPS/TLS Analysis

The HTTPS capture demonstrated TLS negotiation followed by encrypted application traffic.

The TLS analysis identified:

Client Hello
Server Hello
Cryptographic negotiation
Key exchange information
Encrypted application data

Unlike the HTTP capture, application-layer content was not directly visible as readable HTTP requests and responses.

This demonstrates the confidentiality benefit provided by TLS for application traffic.

Evidence: screenshots/06-tls-analysis.png

11. Security Monitoring Observations

The analysis demonstrated several useful network-monitoring capabilities:

Identification of communicating hosts.
Identification of TCP services being targeted.
Detection of repeated TCP SYN retransmissions.
Identification of SSH negotiation and encrypted traffic.
Visibility into DNS resolution activity.
Inspection of cleartext HTTP traffic.
Identification of encrypted TLS application traffic.
Differentiation between local and external network endpoints.

These observations demonstrate how packet captures can provide visibility into network behavior and support security monitoring activities.

12. Limitations

The captures represent controlled laboratory traffic and are not intended to represent the complete traffic profile of a production network.

The observed TCP retransmissions were analyzed as network behavior and were not automatically classified as malicious activity.

No conclusion about malicious intent was made solely from packet-level observations.

The analysis is also limited to the protocols and traffic present during the capture periods.

13. Conclusion

The Wireshark analysis demonstrated how packet captures can be used to investigate network communications, identify endpoints, analyze TCP behavior, inspect application protocols, and distinguish encrypted from unencrypted traffic.

The laboratory exercise provided practical experience in network security monitoring, packet analysis, protocol investigation, traffic interpretation, and defensive cybersecurity analysis.

The resulting evidence and analysis will form the basis for the project's security findings and final monitoring report.