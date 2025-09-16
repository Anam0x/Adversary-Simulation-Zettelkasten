---
aliases:
tags:
  - 🏗️Infrastructure
primary categories:
  - "[[Penetration Test]]"
  - "[[Red Team]]"
secondary categories:
  - "[[C2 Tradecraft & Profiles]]"
type: Infrastructure
---
# [[C2 Redirector]]

***

## Overview

Operators use redirectors to control and restrict the flow of network traffic to their [command and control (C2)](https://csrc.nist.gov/glossary/term/command_and_control) infrastructure while obfuscating their identities[^1].

![[c2-redirectors.drawio.png]]

### Victim Network

Imagine an implant deployed on a victim network that communicates over an encrypted channel (typically HTTPS) to one or more redirectors hosted in a public cloud platform.

The implant should blend into normal outbound traffic patterns, using standard ports (e.g., 443) and appearing similar to legitimate web, DNS, or API requests. Operators may design the implant to use common hostnames or mimic popular services to evade detection by network monitoring tools.

Operators must manage implant beaconing intervals and jitter to avoid generating suspicious network spikes or patterns. In some cases, defenders may analyze network request timing to detect covert channels, so operators should balance usability and stealth[^2].

Implants use short-haul beacons (frequent check-ins) and long-haul beacons (infrequent communications for stealth). Alternating between modes maintains responsiveness while reducing detection risk.
### Public Cloud Network

A redirector must not expose any operator-identifying data or metadata, since it resides outside the operator's controlled environment.

A redirector can take many forms:

* [Virtual machine (VM)](https://csrc.nist.gov/glossary/term/virtual_machine) in AWS, Azure, GCP, etc.
* [Content Delivery Network (CDN)](https://csrc.nist.gov/glossary/term/content_delivery_networks) endpoint or edge function
* Serverless computing with AWS Lambdas[^3], Azure Functions[^4], and/or Cloudflare Workers[^5]
* [Platform-as-a-Service (PaaS)](https://csrc.nist.gov/glossary/term/platform_as_a_service) container with custom proxy logic
* And many more...

Operators should filter out unwanted traffic (e.g., scanners, crawlers, or blue‑team reconnaissance) by deploying a lightweight reverse proxy (e.g., Apache[^6] and/or NGINX[^7]) with rules evaluating:

* [`User-Agent`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/User-Agent) HTTP headers
* [Cookie](https://csrc.nist.gov/glossary/term/cookie) values
* [URI](https://csrc.nist.gov/glossary/term/uniform_resource_identifier) path patterns or file extensions
* Query‑string parameters

Suspicious requests can be dropped or redirected to benign content. Other solutions for filtering include [Web Application Firewalls (WAFs)](https://csrc.nist.gov/glossary/term/web_application_firewall) that block known scanning signatures and automated exploit attempts before they reach the proxy. Combining these with [distributed denial of service (DDoS)](https://csrc.nist.gov/glossary/term/web_application_firewall) protection services (e.g., AWS Shield[^8] and/or Azure DDoS Protection[^9]) improves survivability against takedowns and large-scale scans. Rate limiting by IP or region further restricts exposure[^10].

Using a valid, trusted certificate (such as one provisioned through a service like Let’s Encrypt[^11]) ensures traffic blends in with typical enterprise HTTPS and avoids generating certificate errors that may tip off defenders. In some scenarios, [mutual TLS (mTLS)](https://csrc.nist.gov/glossary/term/mutual_tls) or client certificate authentication can help authenticate implants explicitly[^12]. Automating certificate renewal reduces operational risk and downtime.

Operators often deploy multiple redirectors for scalability and redundancy. [Auto-scaling](https://www.ibm.com/think/topics/autoscaling) groups (for VMs) or [concurrency-based scaling](https://www.toucantoco.com/en/glossary/automatic-concurrency-scaling.html) (for serverless functions) allow redirectors to handle unexpected load without crashing. Distributing redirectors across different regions mitigates single points of failure and complicates defender takedowns. Operators should ensure that redirectors can fail over gracefully without interrupting implant communication.

### Additional Considerations

Operators typically establish an encrypted reverse [SSH](https://csrc.nist.gov/glossary/term/secure_shell_network_protocol) tunnel (or [VPN tunnel](https://csrc.nist.gov/glossary/term/tunnel_vpn)) from the C2 team server to the redirector. Directly connecting the redirector back to the C2 server is discouraged, as this would require storing sensitive private keys on the redirector and allow inbound access from the public cloud, increasing OPSEC risk.

Operators should continuously monitor redirector uptime and performance to avoid service interruptions. Cloud-native monitoring tools (e.g., AWS CloudWatch[^13] and/or Azure Monitor[^14]) can track HTTP status codes, error rates, and latency. Implementing periodic health checks detects if a redirector has been taken offline or misconfigured. Using out-of-band alerting (e.g., via Slack[^15], [SMS](https://csrc.nist.gov/glossary/term/short_message_service), or custom [webhooks](https://help.make.com/webhooks)) can quickly notify operators of availability issues or unexpected traffic spikes.

Operators can also deploy chained redirectors to further obscure the true C2 infrastructure. For example, implant traffic may first reach a CDN edge worker, then pass through a cloud-based VM, before finally arriving at the C2 server. This multi-hop architecture frustrates defender traceback efforts but increases operational complexity and requires careful tunnel and credential management to avoid introducing OPSEC risks.

Redirectors, like most C2 infrastructure, should be treated as disposable assets. After an operation, securely destroy redirectors, keys, client data, and configurations[^16]. Automating the creation and teardown of redirectors via [Infrastructure as Code (IaC)](https://csrc.nist.gov/glossary/term/infrastructure_as_code) or cloud automation tools reduces the risk of leftover assets that defenders might later discover and lowers hosting costs.

***

## Resources

| Hyperlink                                                                                                                                                             | Info                                                                                                                                        |
| --------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| [Red Team Tutorial: Design and setup of C2 traffic redirectors, Dmitrijs Trizna](https://ditrizna.medium.com/design-and-setup-of-c2-traffic-redirectors-ec3c11bd227d) | Medium blog post on C2 infrastructure with a focus on redirectors                                                                           |
| [Red Team Ops II, Zero-Point Security](https://training.zeropointsecurity.co.uk/courses/red-team-ops-ii)                                                              | A continuation of ZPS's "Red Team Ops" course; one of the primary learning objectives is the maintenance and hardening of C2 infrastructure |

[^1]: Red Team Tutorial: Design and setup of C2 traffic redirectors, Dmitrijs Trizna, https://ditrizna.medium.com/design-and-setup-of-c2-traffic-redirectors-ec3c11bd227d
[^2]: The Jitter-Trap: How Randomness Betrays the Evasive, Varonis, https://www.varonis.com/blog/jitter-trap
[^3]: AWS Lambda, Amazon Web Services, https://aws.amazon.com/lambda
[^4]: Azure Functions, Microsoft, https://learn.microsoft.com/en-us/azure/azure-functions/functions-overview
[^5]: Cloudflare Workers, Cloudflare, https://workers.cloudflare.com/
[^6]: Apache HTTP Server Project, Apache Software Foundation, https://httpd.apache.org/
[^7]: nginx, Nginx Inc., https://nginx.org/
[^8]: AWS Shield, Amazon Web Services, https://aws.amazon.com/shield/
[^9]: Azure DDoS Protection, Microsoft, https://learn.microsoft.com/en-us/azure/ddos-protection/ddos-protection-overview
[^10]: C2 Redirectors, xbz0n, https://xbz0n.sh/blog/c2-redirectors
[^11]: Let's Encrypt, Internet Security Research Group, https://letsencrypt.org/
[^12]: Revisiting Cloudflare Workers for C2 Redirections, byt3bl33d3r, https://byt3bl33d3r.substack.com/p/revisiting-cloudflare-workers-for
[^13]: Amazon CloudWatch, Amazon Web Services, https://aws.amazon.com/cloudwatch/
[^14]: Azure Monitor, Microsoft, https://learn.microsoft.com/en-us/azure/azure-monitor/fundamentals/overview
[^15]: Slack, Slack Technologies, https://slack.com/
[^16]: Red Team Assessment Phases: Completing Objectives, InfoSec Institute, https://www.infosecinstitute.com/resources/penetration-testing/red-team-assessment-phases-completing-objectives/

***

*Created Date*: <%+tp.file.creation_date("MMMM Do YYYY (HH:mm a)")%>  
*Last Modified Date*: <%+tp.file.last_modified_date("MMMM Do YYYY (HH:mm a)")%>
