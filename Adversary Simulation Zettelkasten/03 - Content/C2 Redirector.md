---
aliases:
tags:
  - 🏗️Infrastructure
primary-categories:
  - "[[Penetration Test]]"
  - "[[Red Team]]"
secondary-categories:
  - "[[C2 Tradecraft & Profiles]]"
type: Infrastructure
infrastructure-type: Redirector
platforms:
  - Cloud
components:
  - Reverse proxy
  - TLS certificate
  - Reverse tunnel
supports-playbooks:
  - <!-- [[Playbook Note]] -->
supports-tools:
  - <!-- [[Tool Note]] -->
supports-tradecraft:
  - <!-- [[Tradecraft Note]] -->
note-status: ☑️ Ready
---
# [[C2 Redirector]]

---
## Overview

Operators use redirectors to control and restrict the flow of network traffic to their command and control (C2) infrastructure while obfuscating their identities[^1].

This note describes a redirector-centric C2 architecture where implants beacon toward disposable cloud-hosted intermediaries rather than directly exposing operator-controlled infrastructure.

## Architecture Summary

![[c2-redirectors.drawio.png]]

Imagine an implant deployed on a victim network that communicates over an encrypted channel (typically HTTPS) to one or more redirectors hosted in a public cloud platform.

The implant should blend into normal outbound traffic patterns, using standard ports (e.g., 443) and appearing similar to legitimate web, DNS, or API requests. Operators may design the implant to use common hostnames or mimic popular services to evade detection by network monitoring tools.

Operators must manage implant beaconing intervals and jitter to avoid generating suspicious network spikes or patterns. In some cases, defenders may analyze network request timing to detect covert channels, so operators should balance usability and stealth[^2].

Implants use short-haul beacons (frequent check-ins) and long-haul beacons (infrequent communications for stealth). Alternating between modes maintains responsiveness while reducing detection risk.

## Major Components

### Cloud Redirector Edge

**Role**:
- Receive implant traffic over common outbound channels
- Present a disposable public-facing endpoint instead of exposing the team server directly

**Dependencies**:
- Cloud compute, CDN, serverless, or PaaS hosting in a provider such as AWS, Azure, GCP, or Cloudflare
- Domain/DNS records that resolve to the redirector edge
- TLS material that keeps the listener aligned with expected HTTPS behavior

**Security Relevance**:
- The redirector is the most exposed layer in the design and must not contain operator-identifying data or long-lived secrets
- Region choice, provider telemetry, and exposed service metadata all affect traceback and takedown risk
- Multiple redirectors or regions improve survivability but increase coordination and certificate-management overhead

### Traffic Filtering Layer

**Role**: 
- Distinguish expected implant traffic from scanners, crawlers, and blue-team reconnaissance before requests ever reach the backend C2 channel

**Dependencies**:
- Reverse proxy logic such as Apache[^3], NGINX[^4], edge middleware, or custom application handlers
- Rules that evaluate headers, cookies, URI patterns, query strings, or source characteristics
- Optional WAF or DDoS controls such as AWS Shield[^5] or Azure DDoS Protection[^6]

**Security Relevance**:
- Filtering logic is often the first real OPSEC control between public traffic and the operational backend
- Weak filtering exposes the team server to noisy probes, while overly strict filtering can break implants or strand operators
- Logging at this layer can be valuable for spotting scanning patterns, but it must be managed carefully to avoid retaining sensitive data unnecessarily

### Backend C2 Tunnel

**Role**:
- Bridge validated redirector traffic to the actual team server without exposing the server directly to the internet

**Dependencies**:
- Reverse SSH[^7] or VPN-style tunnels that originate from the backend toward the redirector
- Stable routing between the public edge and the operator-controlled service
- Certificate renewal or client-auth workflows when mutual TLS or stronger backend authentication is used[^8][^9]

**Security Relevance**:
- Inbound reachability to the team server should be minimized; the redirector should not need durable private keys for direct backend initiation
- Tunnel failure, certificate expiration, or provider-side outages can sever C2 unexpectedly
- This layer defines how much of the real infrastructure is recoverable if a redirector is seized or fully instrumented

## Trust Boundaries

- The victim environment should only see the redirector-facing edge and never the true team server
- Redirectors must be treated as semi-exposed infrastructure with no operator-identifying data
- Tunnels, certificates, and filtering logic form the most sensitive control points in the design

## Operational Notes

Operators typically establish an encrypted reverse SSH tunnel (or VPN tunnel) from the C2 team server to the redirector. Directly connecting the redirector back to the C2 server is discouraged, as this would require storing sensitive private keys on the redirector and allow inbound access from the public cloud, increasing OPSEC risk[^7].

Operators should continuously monitor redirector uptime and performance to avoid service interruptions. Cloud-native monitoring tools such as AWS CloudWatch and Azure Monitor can track HTTP status codes, error rates, and latency[^10][^11]. Implementing periodic health checks detects if a redirector has been taken offline or misconfigured. Using out-of-band alerting via Slack, SMS, or custom webhooks can quickly notify operators of availability issues or unexpected traffic spikes[^12][^13][^14].

Operators can also deploy chained redirectors to further obscure the true C2 infrastructure. For example, implant traffic may first reach a CDN edge worker, then pass through a cloud-based VM, before finally arriving at the C2 server. This multi-hop architecture frustrates defender traceback efforts but increases operational complexity and requires careful tunnel and credential management to avoid introducing OPSEC risks.

Redirectors, like most C2 infrastructure, should be treated as disposable assets. After an operation, securely destroy redirectors, keys, client data, and configurations[^15]. Automating the creation and teardown of redirectors via Infrastructure as Code (IaC) or cloud automation tools reduces the risk of leftover assets that defenders might later discover and lowers hosting costs[^16].

---

## Related Notes

### Same Classification

#### Similar Infrastructure
```dataview
LIST
FROM "03 - Content"
WHERE type = "Infrastructure"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND file.name != this.file.name
  AND (
    infrastructure-type = this.infrastructure-type OR
    contains(platforms, this.platforms[0])
  )
SORT file.name ASC
LIMIT 10
```

### Typed Relationships

#### Supports Tradecraft
```dataview
LIST
FROM "03 - Content"
WHERE type = "Tradecraft"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND contains(this.supports-tradecraft, file.link)
SORT file.name ASC
```

#### Supports Tools
```dataview
LIST
FROM "03 - Content"
WHERE type = "Tool"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND contains(this.supports-tools, file.link)
SORT file.name ASC
```

#### Supports Playbooks
```dataview
LIST
FROM "03 - Content"
WHERE type = "Playbook"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND contains(this.supports-playbooks, file.link)
SORT file.name ASC
```

---

## Resources

| Reference | Info |
| --------- | ---- |
| [Red Team Tutorial: Design and setup of C2 traffic redirectors, Dmitrijs Trizna](https://ditrizna.medium.com/design-and-setup-of-c2-traffic-redirectors-ec3c11bd227d) | Medium blog post on C2 infrastructure with a focus on redirectors                                                                           |
| [Red Team Ops II, Zero-Point Security](https://training.zeropointsecurity.co.uk/courses/red-team-ops-ii)                                                              | A continuation of ZPS's "Red Team Ops" course; one of the primary learning objectives is the maintenance and hardening of C2 infrastructure |

[^1]: Red Team Tutorial: Design and setup of C2 traffic redirectors, Dmitrijs Trizna, https://ditrizna.medium.com/design-and-setup-of-c2-traffic-redirectors-ec3c11bd227d
[^2]: The Jitter-Trap: How Randomness Betrays the Evasive, Varonis, https://www.varonis.com/blog/jitter-trap
[^3]: Apache HTTP Server Project, Apache Software Foundation, https://httpd.apache.org/
[^4]: nginx, Nginx Inc., https://nginx.org/
[^5]: AWS Shield, Amazon Web Services, https://aws.amazon.com/shield/
[^6]: Azure DDoS Protection, Microsoft, https://learn.microsoft.com/en-us/azure/ddos-protection/ddos-protection-overview
[^7]: Secure Shell, NIST, https://csrc.nist.gov/glossary/term/secure_shell_network_protocol
[^8]: Let's Encrypt, Internet Security Research Group, https://letsencrypt.org/
[^9]: Revisiting Cloudflare Workers for C2 Redirections, byt3bl33d3r, https://byt3bl33d3r.substack.com/p/revisiting-cloudflare-workers-for
[^10]: Amazon CloudWatch, Amazon Web Services, https://aws.amazon.com/cloudwatch/
[^11]: Azure Monitor, Microsoft, https://learn.microsoft.com/en-us/azure/azure-monitor/fundamentals/overview
[^12]: Slack, Slack Technologies, https://slack.com/
[^13]: Short Message Service, NIST, https://csrc.nist.gov/glossary/term/short_message_service
[^14]: Webhooks, Make, https://help.make.com/webhooks
[^15]: Red Team Assessment Phases: Completing Objectives, InfoSec Institute, https://www.infosecinstitute.com/resources/penetration-testing/red-team-assessment-phases-completing-objectives/
[^16]: Infrastructure as Code, NIST, https://csrc.nist.gov/glossary/term/infrastructure_as_code

---

*Created Date*: <%+tp.file.creation_date("MMMM Do YYYY (HH:mm a)")%>  
*Last Modified Date*: <%+tp.file.last_modified_date("MMMM Do YYYY (HH:mm a)")%>
