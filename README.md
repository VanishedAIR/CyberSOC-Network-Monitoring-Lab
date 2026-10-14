# CyberSOC Network Monitoring Lab

**Personal project, in progress (September 2026 to present)**

A three-VM lab for practicing network monitoring and detection. Client traffic crosses a Linux router that captures packets and runs Suricata, and I use it to write and test detection rules against traffic I generate myself.

## Objective

Build a small segmented network, capture HTTP traffic crossing it, and detect a specific request pattern with a custom Suricata rule. The aim is to understand the full path from routing and firewall configuration to packet capture, rule writing, alert generation and log inspection.

## Architecture

Three Ubuntu VMs run in VMware. The monitor sits between two subnets and routes traffic from one to the other.

| VM | Role | Address |
|---|---|---|
| Client | Generates HTTP requests | 10.77.10.10 |
| Monitor | Router and sensor, client side | 10.77.10.1 |
| Monitor | Router and sensor, server side | 10.77.20.1 |
| Server | Web server on port 8080 | 10.77.20.20 |

Client and server use separate subnets, `10.77.10.0/24` and `10.77.20.0/24`. HTTP requests between them pass through the monitor.

## Tools

VMware, Ubuntu, nftables, tcpdump, curl, Suricata 7.0.3 and jq.

## Completed Milestones

1. Built the three VMs with separate client and server subnets and addressed each interface.
2. Configured static routes and Linux IPv4 forwarding on the monitor, plus nftables rules.
3. Sent client-to-server HTTP traffic on port 8080 and captured it on the monitor.
4. Analyzed the capture offline with Suricata.
5. Wrote a custom HTTP GET detection rule (SID 1000001).
6. Validated the rule with GET and HEAD requests.
7. Ran Suricata for live detection and enabled it as a systemd service.
8. Rebooted and confirmed that live alerts still fired.
9. Inspected Suricata's EVE JSON event and alert records with jq.

## Detection Validation

I sent both a GET and a HEAD request from the client to the server. Suricata logged both as HTTP transactions, and the custom rule alerted on the GET request only. That confirms the rule matches the method it was written for and does not fire on a different HTTP method.

After enabling Suricata as a service and rebooting, I confirmed that live alerts were still generated.

## Evidence

Screenshots from the lab document the setup and results. Configuration files, packet captures and EVE logs remain in the lab VMs and still need to be exported into this repository.

## Limitations

- This is a small lab with traffic I generate myself, not a production SOC.
- There is no SIEM integration, centralized dashboard or incident response workflow.
- Detection covers one custom rule. The nftables setup is a lab configuration, not a hardened policy.

## Next Steps

- Export the Suricata rule file, nftables ruleset, interface and routing configuration, and a sample PCAP into this repository.
- Document the exact commands used to generate and verify each test.
