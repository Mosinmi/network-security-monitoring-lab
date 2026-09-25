# Network Security Monitoring & Wireshark Traffic Analysis Lab

## Overview

This project is a practical Network Security Monitoring (NSM) lab focused on capturing, analyzing, and documenting network traffic using **Wireshark** and **tcpdump**.

The lab demonstrates how a security analyst can inspect network communications, identify protocols and endpoints, analyze TCP connection behavior, detect repeated connection attempts, and distinguish between cleartext and encrypted traffic.

The project was performed in a controlled lab environment using a Windows host and an Ubuntu Server virtual machine.

---

## Objectives

The main objectives of this project were to:

- Capture network traffic using `tcpdump`.
- Analyze packet captures using Wireshark.
- Examine TCP connection behavior.
- Identify TCP SYN packets and retransmissions.
- Analyze SSH traffic.
- Analyze DNS queries and responses.
- Examine HTTP traffic and identify visible application-layer data.
- Examine HTTPS/TLS traffic and identify encrypted communications.
- Analyze protocol distribution and network endpoints.
- Document security observations and findings.
- Demonstrate practical Network Security Monitoring techniques.

---

## Lab Environment

| Component | Details |
|---|---|
| Host Operating System | Windows |
| Analysis Tool | Wireshark |
| Packet Capture Tool | tcpdump |
| Virtualization | Oracle VirtualBox |
| Server Operating System | Ubuntu Server 22.04.5 LTS |
| Ubuntu Interface | `enp0s3` |
| Windows Host IP | `192.168.0.136` |
| Ubuntu Server IP | `192.168.0.162` |
| Network Configuration | Bridged Adapter |

The lab was conducted on systems controlled by the user.

---

## Tools Used

### Wireshark

Wireshark was used for detailed packet-level analysis.

Key features used included:

- Display filters
- TCP Conversations
- TCP stream analysis
- Protocol Hierarchy
- IPv4 Endpoints
- Packet details
- TCP retransmission analysis
- DNS analysis
- HTTP analysis
- TLS analysis

### tcpdump

`tcpdump` was used on the Ubuntu Server to capture traffic directly from the network interface.

Example:

```bash
sudo tcpdump -i enp0s3 -nn -w ~/recon-analysis.pcap 'tcp'

The resulting packet captures were transferred to the Windows host and opened in Wireshark for analysis.

Network Traffic Analysis
1. Protocol Hierarchy Analysis

Wireshark's Statistics → Protocol Hierarchy feature was used to identify the protocols contained in the reconnaissance capture.

The capture contained:

IPv4 traffic
TCP traffic
SSH traffic
HTTP traffic

The protocol hierarchy showed that TCP represented the transport layer traffic observed in the capture, while SSH and HTTP represented application-layer protocols present within the captured traffic.

Evidence

2. IPv4 Endpoint Analysis

Wireshark's Statistics → Endpoints → IPv4 feature was used to identify the hosts communicating in the capture.

The primary endpoints observed were:

Endpoint	Observed Activity
192.168.0.136	Windows host generating connection attempts
192.168.0.162	Ubuntu Server receiving connection attempts
91.189.91.57	External endpoint observed during HTTP communication

The Ubuntu Server was the primary destination for the TCP connection tests performed from the Windows host.

Evidence

TCP Analysis
3. TCP Conversation Analysis

Wireshark's Statistics → Conversations → TCP feature was used to examine TCP conversations.

The following destination ports were observed:

Destination Port	Service/Protocol	Observed Behavior
22	SSH	Successful TCP response observed
23	Telnet	Repeated SYN attempts with no successful response observed
80	HTTP	Repeated SYN attempts with no successful response observed during the test
443	HTTPS	Repeated SYN attempts with no successful response observed during the test

An additional outbound HTTP conversation from the Ubuntu Server to an external endpoint was also observed.

The results demonstrate how TCP conversations can be used to identify attempted service connections and distinguish responsive traffic from connection attempts that did not receive a successful TCP response.

Evidence

4. TCP SYN Analysis

The following Wireshark display filter was used:

tcp.flags.syn == 1 && tcp.flags.ack == 0

This filter displays TCP SYN packets that initiate connection attempts.

The analysis revealed repeated SYN packets directed toward ports 23, 80, and 443 on the Ubuntu Server.

Repeated SYN packets without a corresponding successful response can indicate that a service is not responding, traffic is being filtered, or the destination is otherwise unreachable.

Evidence

5. TCP Retransmission Analysis

The following Wireshark filter was used:

tcp.analysis.retransmission

The analysis showed repeated TCP SYN retransmissions associated with connection attempts to:

TCP port 23
TCP port 80
TCP port 443

These retransmissions demonstrate how packet analysis can reveal unsuccessful or incomplete connection attempts.

From a security monitoring perspective, repeated connection attempts can be useful indicators for investigating network connectivity problems, service availability, firewall behavior, or reconnaissance activity.

Evidence

SSH Traffic Analysis
6. SSH Connection Analysis

SSH traffic was analyzed between the Windows host and Ubuntu Server.

The SSH connection used:

Windows Host: 192.168.0.136
Ubuntu Server: 192.168.0.162
Destination Port: 22

The packet capture showed the SSH protocol and server/client identification information.

The observed SSH software banners included:

SSH-2.0-OpenSSH_for_Windows_9.5

and:

SSH-2.0-OpenSSH_8.9p1 Ubuntu-3ubuntu0.17

The SSH negotiation also exposed information about supported key-exchange algorithms and cryptographic algorithms.

However, the contents of the authenticated SSH session were not available as readable application data because SSH encrypts the session after negotiation.

DNS Traffic Analysis
7. DNS Analysis

DNS traffic was captured and analyzed using Wireshark.

The following display filters were used:

dns

To isolate DNS queries:

dns.flags.response == 0

To isolate DNS responses:

dns.flags.response == 1

To examine queried domain names:

dns.qry.name

The analysis demonstrated how DNS requests and responses can be examined to determine domain-resolution activity.

DNS monitoring can provide useful visibility into the destinations requested by systems on a network and can support further investigation when suspicious or unexpected domains are identified.

Evidence

DNS traffic was captured in:

captures/02-dns-analysis.pcapng
HTTP Traffic Analysis
8. Cleartext HTTP Analysis

HTTP traffic was generated from the Ubuntu Server using:

curl http://example.com

The traffic was captured using:

sudo tcpdump -i enp0s3 -nn -w ~/http-analysis.pcap 'tcp port 80'

The capture contained HTTP requests and responses.

An observed request included:

GET / HTTP/1.1

The capture also showed HTTP response information, including:

HTTP/1.1 204 No Content

and:

HTTP/1.1 200 OK

Because HTTP does not encrypt the application-layer content, information such as HTTP methods, request paths, and response headers can be visible within a packet capture.

Security Significance

Cleartext HTTP traffic can expose application-layer information to someone who can capture the traffic.

This demonstrates why encrypted protocols such as HTTPS are preferred for protecting web communications.

Evidence

Capture:

captures/http-analysis.pcap
HTTPS and TLS Analysis
9. TLS Traffic Analysis

HTTPS traffic was generated using:

curl -I https://example.com

The traffic was captured using:

sudo tcpdump -i enp0s3 -nn -w ~/https-analysis.pcap 'tcp port 443'

Wireshark was then used to inspect the TLS packets.

The following display filter was used:

tls

The analysis showed TLS negotiation traffic, including the client and server handshake messages.

Unlike the HTTP capture, the application data transmitted after the TLS handshake was not available as readable plaintext.

Security Significance

TLS provides encryption for application-layer communication, reducing the visibility of sensitive application data to passive network observers.

Network security monitoring systems can still observe metadata such as:

Source and destination IP addresses
TCP ports
Connection timing
Packet sizes
TLS handshake information

However, the encrypted application payload is not directly readable from the packet capture without appropriate decryption material.

Evidence

Capture:

captures/https-analysis.pcap
Security Findings

The analysis produced several security-relevant observations.

ID	Finding	Severity	Status
NSM-001	TCP service connection behavior	Medium	Observed
NSM-002	Repeated TCP SYN retransmissions	Low	Observed
NSM-003	DNS resolution activity	Informational	Observed
NSM-004	Cleartext HTTP traffic visibility	Medium	Observed
NSM-005	TLS-protected application traffic	Informational	Observed

Detailed findings are documented in:

documentation/findings.md

The complete traffic analysis is documented in:

documentation/analysis.md
Security Monitoring Observations

The lab demonstrated several practical concepts relevant to Network Security Operations and Security Engineering.

Network Visibility

Packet captures provide visibility into:

Network conversations
Protocols
Endpoints
Ports
Connection attempts
Retransmissions
Application-layer traffic
Reconnaissance Detection

Repeated TCP SYN attempts against multiple ports can be identified through packet analysis.

Useful filters include:

tcp.flags.syn == 1 && tcp.flags.ack == 0

and:

tcp.analysis.retransmission
Cleartext Traffic Detection

HTTP traffic can expose application-layer information directly in packet captures.

This makes protocol identification and cleartext traffic detection important components of network monitoring.

Encrypted Traffic Monitoring

HTTPS/TLS demonstrates that encryption protects application payloads while still leaving network metadata available for monitoring.

Security monitoring tools can therefore combine:

IP addresses
Ports
Protocols
Connection timing
Packet characteristics
TLS metadata

to investigate network activity.

Capture Files

The project contains packet captures used during the investigation.

captures/
├── 02-dns-analysis.pcapng
├── http-analysis.pcap
├── https-analysis.pcap
└── recon-analysis.pcap

Additional captures may be included as the lab is expanded.

Screenshots

Evidence screenshots are stored in:

screenshots/

Key evidence includes:

03-tcp-conversations.png
04-tcp-syn-analysis.png
05-http-request.png
06-tls-analysis.png
07-protocol-hierarchy.png
08-ipv4-endpoints.png
09-tcp-retransmissions.png
Limitations

This lab was performed in a controlled environment and does not represent traffic from a production enterprise network.

The following limitations apply:

The capture duration was limited.
The number of monitored hosts was small.
The lab did not include a full enterprise SIEM.
No malicious payloads were used.
Observed SYN retransmissions do not by themselves prove malicious activity.
External IP addresses observed in the captures were not automatically classified as malicious.
Encrypted traffic could not be analyzed at the application-payload level without decryption material.

Therefore, the findings should be interpreted as observations from the controlled lab environment rather than conclusions about a production network.

Skills Demonstrated

This project demonstrates practical experience with:

Network Security Monitoring
Wireshark
tcpdump
Packet capture
Packet analysis
TCP/IP
TCP handshake analysis
TCP SYN analysis
TCP retransmission analysis
DNS analysis
HTTP analysis
HTTPS/TLS analysis
Network endpoint analysis
Protocol identification
Security evidence collection
Security findings documentation
Basic reconnaissance detection
Project Structure
network-security-monitoring-lab/
│
├── README.md
│
├── captures/
│   ├── 02-dns-analysis.pcapng
│   ├── http-analysis.pcap
│   ├── https-analysis.pcap
│   └── recon-analysis.pcap
│
├── screenshots/
│   ├── 03-tcp-conversations.png
│   ├── 04-tcp-syn-analysis.png
│   ├── 05-http-request.png
│   ├── 06-tls-analysis.png
│   ├── 07-protocol-hierarchy.png
│   ├── 08-ipv4-endpoints.png
│   └── 09-tcp-retransmissions.png
│
├── documentation/
│   ├── analysis.md
│   └── findings.md
│
└── reports/
Conclusion

This lab provided practical experience in network traffic capture, packet analysis, protocol identification, and security monitoring.
