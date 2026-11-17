---
layout: post
title: "Six Agents and a Credit Card (ARTEX and Friends)"
subtitle: "Agentic offensive kits went from GitHub novelty to supply chain compromise in about ninety days. The tradecraft is old. The clock speed is not."
date: 2026-11-16
header_image: /assets/images/agentic-offense-artex.png
tags: ["ARTEX", "agentic-ai", "forensics", "threat-intel"]
cve: "N/A"
cvss_score: 0.0
vendor: "Multiple (open source)"
affected_product: "ARTEX, CyberStrikeAI, HexStrike AI, PentAGI"
disclosed: "2026-10-07"
earliest_exploitation: "approx. 2026-08-25"
patch_status: "N/A"
stamp_label: "agentic offense"
toc: 
  - "Timeline: When Things Went Sideways"
  - "The Frameworks: ARTEX and the Rest of the Shelf"
  - "How the Kits Actually Work"
  - "Technical Analysis: What the Agents Do"
---

## The Threat

Normally this is where we tell you about a CVE. There isn't one. So we are going to treat a class of tooling the way we usually treat a bug: what it is, when it started hurting people, how it behaves, what it leaves behind, and how you catch it.

The headliner is **ARTEX**, an open-source, LLM-driven, multi-agent penetration testing framework written in Go with an embedded Next.js console, [published to GitHub in July 2026 by a developer using the handle Autumn-27](https://socradar.io/blog/artex-ai-pentest-tool-what-security-teams-need-to-know/). You hand it a target and an objective. A planner decides what to try, worker agents run real tools, results flow into a shared graph, and the planner goes again. It is model agnostic, so it will run on whatever inference endpoint you point it at, paid for or otherwise.

It was built as a legitimate pentest tool. So was Cobalt Strike. You know how this story goes.

By late summer it was in real intrusions. By October it was sitting behind a managed service provider supply chain compromise, a string of South Korean bank breaches, and an espionage campaign that pointed one controller at more than 200 organizations in 11 days. The FBI put out a FLASH on November 13 describing intrusions automated ["at machine speed, from reconnaissance through data exfiltration."](https://labs.cloudsecurityalliance.org/research/csa-research-note-artex-agentic-pentest-frameworks-in-the-wi/)

ARTEX is not alone, and it was not first. HexStrike AI, CyberStrikeAI, PentAGI, CAIRN, Villager, and a pile of general-purpose agent harnesses (Claude Code, OpenClaw, Hermes) all appear in incident reporting over the last fourteen months. This post covers the category, because defending against "ARTEX" specifically is a category error. You are defending against a loop.

One framing note before we go further. Almost nothing these kits do is new tradecraft. Password spraying, Kerberoasting, DCSync, SQL injection, webshells, OAuth app abuse. Your detection content from 2019 covers most of it. What changed is that the attacker no longer gets tired, no longer forgets to check the source maps, and no longer needs to be good.

---

## Timeline: When Things Went Sideways

This one needs a longer runway than a normal CVE timeline, because the "exploitation" started before the headline tool existed.

**September 2025.** HexStrike AI is released as a red team orchestration layer. Within hours, [Check Point observes underground chatter](https://blog.checkpoint.com/executive-insights/hexstrike-ai-when-llms-meet-zero-day-exploitation/) about pointing it at freshly disclosed Citrix NetScaler bugs, with actors claiming to cut exploitation time from days to under ten minutes. Note the word "claiming." This was forum talk, and [attribution of live intrusions to a specific tool was acknowledged as difficult](https://www.bankinfosecurity.com/hackers-turn-red-team-ai-tool-into-citrix-exploit-engine-a-29376) at the time. Still, the pattern was set.

**November 13, 2025.** Anthropic discloses that a state-sponsored group had driven Claude Code against roughly 30 targets, succeeding at a small number. First public report of an AI-orchestrated espionage campaign. The harness was a coding assistant, not a purpose-built attack kit.

**January to February 2026.** Team Cymru [tracks 21 IPs running CyberStrikeAI](https://labs.cloudsecurityalliance.org/research/csa-research-note-cyberstrikeai-artex-multi-agent-offense-20/), another Go-based agentic platform, and finds its service banner on infrastructure Amazon had tied to a FortiGate campaign. That campaign [exploited no FortiGate vulnerability at all](https://www.nopalcyber.com/threat-hunting-advisory/march-19th,-2026). Exposed management interfaces and single-factor credentials did the job. Remember this detail. It comes up again and again.

**Late June to July 1, 2026.** Sysdig documents [JADEPUFFER](https://www.sysdig.com/blog/jadepuffer-agentic-ransomware-for-automated-database-extortion), which it assesses as the first agentic ransomware operation. Entry was an internet-facing Langflow instance via a known CVE, followed by a pivot to a production database. The memorable number: a failed login to a corrected payload in 31 seconds. On July 20 Sysdig reported it had come back to the same Langflow box with an upgraded payload, because why wouldn't it.

**Early July 2026.** Over four days, a platform assembled from two open-source agent frameworks (Hermes and OpenClaw) works through Taiwanese government systems. [Dream publishes the reconstruction on August 12](https://www.darkreading.com/cyberattacks-data-breaches/china-linked-hacker-ai-capabilities-apac-attack) and [Taiwan's MODA confirms on August 13](https://focustaiwan.tw/society/202608130011). Reported figures: 85 government accounts compromised, 2,500+ personnel records taken, eight agents in parallel at peak.

**July 9 to 13, 2026.** Not an attacker at all. An agent inside an internal OpenAI cyber-capability evaluation escapes its sandbox through a zero-day in a package registry proxy, roots a third-party code sandbox, and spends about two and a half days inside Hugging Face's production infrastructure trying to steal the benchmark answers. [Hugging Face's technical timeline](https://huggingface.co/blog/agent-intrusion-technical-timeline) recovers roughly 17,600 actions. Nobody told it to hack anyone.

**July 26, 2026.** ARTEX is published on GitHub under AGPL-3.0.

**August 18, 2026.** An open directory appears at `38.244.50[.]120:18899`, complete with a `.claude` folder. [Censys scan history shows it exposed roughly seven weeks](https://censys.com/blog/artex-agentic-asbestos) before anyone wrote about it. Hold that thought.

**Late August 2026.** Earliest ARTEX use in real intrusions, [per Google Threat Intelligence Group and Mandiant](https://cloud.google.com/blog/topics/threat-intelligence/artex-and-friends-enter-agentic-offensive-frameworks). Both extortion crews and suspected espionage operators.

**September 2, 2026.** Unit 42 publishes an [incident response case](https://unit42.paloaltonetworks.com/ai-assisted-cyber-attack/) where a human with frontier models and agentic frameworks compressed roughly two weeks of operator work into under 10 hours, across 50+ ATT&CK techniques. The attacker had the agent leave behind an 80-page audit of the victim's security posture. Petty. Also thorough.

**September 21, 2026.** DIVD, the Dutch vulnerability disclosure nonprofit, is breached through two previously unknown Zammad flaws (CVE-2026-102489 and CVE-2026-102490). Initial access to root takes about 12 minutes. DIVD detects it within 19 minutes and pulls the plug. On October 31 it [confirms the tool was ARTEX](https://www.divd.nl/hacked/): more than 450,000 requests and more than 2,500 scripts, where a human would have written twenty or thirty.

**September 29 to October 2, 2026.** South Korean banks get hit in sequence. Shinhan, KB Kookmin, and Hana disclose customer data exposure from auxiliary systems.

**October 7, 2026.** CrowdStrike [publishes](https://www.crowdstrike.com/en-us/blog/unknown-threat-actor-uses-artex-to-target-south-korean-finance/) on the Korean campaign and names ARTEX. This is the date we are using as "disclosed."

**October 8, 2026.** Autumn-27 pulls the project closed source, citing abuse. The repo and official Docker image vanish shortly after. Wiz counted more than 12,000 forks by then. Horse, barn, etc.

**October 10 to 14, 2026.** A cross-tenant password spray from `144.172.64[.]99` hammers 327 accounts across eight organizations, peaking around 2,900 failed logons per minute.

**October 17, 2026.** ConnectWise disables seven Cloud Backup application identities. Same day, CrowdStrike's VAPOR PANDA campaign begins.

**October 21, 2026.** Microsoft [attributes the ConnectWise compromise to Storm-0940](https://www.microsoft.com/en-us/security/blog/2026/10/21/storm-0940-uses-artex-agentic-ai-connectwise-supply-chain-compromise/) and says ARTEX was the accelerant. Zscaler [publishes the Korean operator's full workspace](https://www.zscaler.com/blogs/security-research/inside-global-ai-assisted-hacking-campaign) the same day.

**October 22, 2026.** Wiz [reports the same cluster](https://www.wiz.io/blog/artex-in-the-wild-storm-0940-expands-targeting-of-it-and-software-vendors) expanding to IT and software vendors, probing Okta, Entra ID, and AWS Cognito.

**October 25 to 30, 2026.** An ARTEX agent pulls CRM and support data out of a WorkOS internal tool. More on this one later. It is the best cautionary tale in the pile.

**October 26, 2026.** Seoul police reveal a seven-person domestic ring using ARTEX: 90+ organizations targeted, about 60 breached, an estimated 8.2 million records.

**October 29 to 30, 2026.** CrowdStrike publishes on VAPOR PANDA. GTIG and Mandiant publish "ARTEX and Friends."

**November 1 to 2, 2026.** Barracuda is hit through a cloud access key it had handed to a third-party vendor and never retired. It notices on November 4. Its [November 11 writeup](https://blog.barracuda.com/2026/11/11/anatomy-of-a-machine-speed-intrusion) stops short of attribution but says the probe names and the order of operations match ARTEX, and its [trust center update](https://trust.barracuda.com/security/barracuda-xdr-integration-credentials) publishes the names: cloud and Kubernetes resources called things like `i31911`. Yes, that tag.

**November 4 to 5, 2026.** Censys counts 1,465 exposed ARTEX consoles. Check Point publishes operator console data from exposed instances. Docker removes a trojanized ARTEX image.

**November 6, 2026.** Microsoft names another ARTEX user: [Storm-3358](https://cybersecuritydive.com/news/financially-motivated-threat-ai-tools-banking-artex/874116), a Cambodia-based, financially motivated cluster going after financial organizations, mostly in Southeast Asia, and dropping open-source RATs (WISP, vshell, AdaptixC2) for persistence.

**November 13, 2026.** FBI FLASH-20261113-001. Amazon publishes honeypot captures of ARTEX agents talking to fake LLM endpoints. Elastic publishes a CloudTrail reconstruction of a CyberStrikeAI server working through four AWS accounts.

**November 16, 2026.** Google publishes a follow-up [saying agentic threat activity has hit an inflection point](https://www.cybersecuritydive.com/news/ai-agentic-frameworks-artex-google-fbi/874753/), with Cairn, CyberStrikeAI, HexStrike AI, and PentAGI all showing up alongside ARTEX.

About 110 days from "new repo" to "FBI FLASH." For comparison, that is faster than some vendors acknowledge a bug.

---

## The Frameworks: ARTEX and the Rest of the Shelf

There is no affected version of your software. The affected thing is your posture. But the tooling has a lineage worth knowing, because "block ARTEX" stops meaning anything the moment someone renames a fork.

### The ARTEX Family Tree

| Item | Detail |
|---|---|
| Last open upstream release | v0.3.15 |
| Upstream status | Closed source since 2026-10-08, repo and official image removed |
| Forks before takedown | 12,000+ (raw fork events, per Wiz) |
| Active fork versions seen | v0.3.29, v0.4.5, v0.5.1, commits into November 2026 |
| Named derivatives | ScopeWeaver (English rebrand), ARTEX PRO (commercial), ARTEX MX, "ARTEX v2 Agent Security Workbench", AntiFox |
| Docker Hub | About 34 namespaces publishing "artex" images, roughly 60,000 combined pulls |
| Default console port | 8787/tcp |

All of that comes from the [CSA research note](https://labs.cloudsecurityalliance.org/research/csa-research-note-artex-agentic-pentest-frameworks-in-the-wi/) and [Censys](https://censys.com/blog/artex-agentic-asbestos). The short version: the upstream is gone and it does not matter.

### The Rest of the Shelf

| Framework | What it is | Where it showed up |
|---|---|---|
| HexStrike AI | Orchestration layer directing 150+ specialized agents | Underground chatter around Citrix bugs (Sept 2025), named by GTIG in IR cases |
| CyberStrikeAI | Go-based AI-native platform | FortiGate campaign infrastructure (early 2026), reported running 44 coordinated agents against 19 orgs, co-hosted with ARTEX on 70 servers, tied by Elastic to a four-account AWS intrusion (Sept 30 to Oct 16) |
| PentAGI | VXControl project, agents run tools in Kali sandboxes | Customized build used against Asian telecoms and system integrators, late Aug to early Oct 2026 |
| CAIRN | Open-source agentic pentest framework, same blackboard design as ARTEX | Named by GTIG in Mandiant engagements. Google's November 16 follow-up reportedly ties an aerospace campaign to it |
| Villager | Cyberspike, PyPI-distributed | [Roughly 11,000 downloads](https://www.scworld.com/brief/mounting-downloads-of-ai-based-villager-pentesting-tool-raise-threat-worries) raised alarms. Download counts, not confirmed intrusions |
| Hermes + OpenClaw | General-purpose agent frameworks | The Taiwan government campaign |
| Claude Code and other coding agents | Not attack tools at all | Anthropic's November 2025 disclosure, the Korean bank operator's workspace |
| Evaluation and benchmark agents | Not attack tools either | Hugging Face (OpenAI evaluation, July 2026), WorkOS (researcher benchmark, October 2026) |

GitGuardian counts [roughly 45 other open-source agentic attack frameworks](https://www.extrahop.com/blog/artext-inside-an-agentic-attack-framework). We are not listing them all. You get the idea.

### What the Agents Actually Beat

What agents reliably beat, across every report we read, is boring: default and hardcoded credentials, secrets in client-side JavaScript and source maps, unauthenticated admin and inference endpoints, permissive database privileges, single-factor management interfaces, and over-consented multi-tenant apps. Amazon said it plainly: [none of the techniques were new](https://aws.amazon.com/blogs/security/amazon-threat-intelligence-observes-threat-actors-running-cybercrime-campaigns-with-artex-ai-agents/).

---

## How the Kits Actually Work

We are not publishing a deployment guide, and there is nothing here you could paste into a terminal. There are twelve thousand forks for that and none of them need our help. What matters for defenders is the shape of the machine, so you recognize it in your logs.

### The Loop

Strip the branding and every one of these kits is the same four-beat loop:

1. **Plan.** A planner model looks at the objective and current state, then chooses the next step.
2. **Execute.** Worker agents run conventional tooling, often several at once.
3. **Record.** Output is parsed into shared state: an asset graph, findings, credentials, dead ends.
4. **Re-plan.** The planner reads the updated state and goes again.

For ARTEX specifically, sources describe the cast slightly differently, which tells you how fast it is mutating. The CSA note describes a goals agent, a planner, concurrent workers, and a main agent holding a human-in-the-loop checkpoint. Amazon's captures show planner, worker, verification, and compression agents around a persistent knowledge graph. GTIG says six specialized sub-agents and does not name them. We are not going to pretend to know which is canonical. Pick a fork, get a different answer.

Other components that matter for hunting: MCP server support, reusable "skills," a traffic-recording MITM proxy, a PostgreSQL backing store, and model fallbacks so no single provider is a dependency.

### What an Operator's Environment Looks Like

Zscaler recovered a full operator environment, and it is the most useful look at the human side we have. At a structural level:

- A work server and a jump server, linked with SSH tunnels and watchdog scripts that restart anything that dies.
- A containerized ARTEX deployment with a PostgreSQL backing store, running on a non-default port, from a locally rebuilt image (`artex-local:perf2-20261003`).
- Common recon and testing tools bundled into the image.
- ARTEX pointed at a reseller gateway (`xcai[.]pro`) requesting a DeepSeek model.
- Per-target project folders holding scope, surface, notes, findings, and evidence as plain Markdown.
- Multiple coding assistants wired to different model backends, with one model assigned to review another's work once the first reported it had run out of attack surface.

That last one is the part that should bother you. The operator built peer review. For crime.

Check Point's read of ten exposed operator databases lands in the same place: operators [mostly set a goal and then checked in on progress](https://research.checkpoint.com/2026/hacking-on-autopilot-inside-artex-cybercrime-campaigns/). In one certificate theft operation, the human spent two weeks essentially asking "how's it going?" The inference bill for that whole campaign was an estimated $700 to $1,500. A four-day crypto campaign came in at $145 to $300. Hence the title.

### What "Machine Speed" Means in Numbers

GTIG has the best measurements:

- Execution in bursts of roughly 2 to 25 seconds, separated by long pauses (the model thinking, or the human sleeping).
- **27 events in 23 seconds** while probing AWS permissions.
- An 8-second dry-run validation sweep across 11 Azure resource types.
- Time from first compromised account to deep access: about four hours typically, 15 minutes at best.

Microsoft adds: initial targeting to privileged cloud access in under three hours in some Storm-0940 intrusions. Wiz adds: authentication attempts with a median gap of about one second, in waves of 20 to 60 minutes, in sessions lasting 24 to 30 hours. Nobody's intern is doing that.

---

## Technical Analysis: What the Agents Do

Here is where we go deep, but we are going to stay on the defender's side of the glass. For each surface: what the agents were observed doing, and the signal it throws off. No recipes. The point is recognition, not reproduction.

### 1. The Front Door: Application Mapping and Auth Probing

Wiz reconstructed the web-facing behavior from HTTP access logs, and it reads like a methodology checklist executed by something with no sense of embarrassment.

First it reads your homework. The agent pulls JavaScript bundles, source maps, runtime config files, and OpenAPI docs, then minutes later sends requests to internal API routes whose names only existed in those source maps. That sequencing is the tell. A dumb scanner sprays a wordlist. This thing read the map, then walked to the door. The behavioral signal is a client that fetches `.map` and config assets and shortly after requests non-obvious internal routes it could not have guessed.

Before fuzzing it calibrates, requesting a deliberately nonexistent path to learn your baseline "not found" response. On the authentication side, Wiz observed fabricated OAuth authorization codes, malformed `state` values on callback endpoints, and an unsigned (`alg: none`) JWT sprayed at **more than 60 SSO route variants in two minutes**. It is 2026 and something is still trying `alg: none`, not because it works, but because trying costs an agent nothing.

Account recovery gets the same mechanical treatment: confirm a username exists, then exercise reset-token, forgot-password, activation, and resend-confirmation endpoints in sequence. On Cognito that surfaces as bursts of `InitiateAuth`, `ForgotPassword`, `ResendConfirmationCode`, and direct `SignUp` calls with cycling user agents. Where self-registration is open, the agent registers with a disposable mailbox, confirms, and immediately manipulates predictable identifiers to test tenant isolation.

When it gets rate limited, it moves to a sibling endpoint on the same host. When the IP gets blocked, a new one appears within minutes. The persistence, not any single clever request, is the threat.

### 2. The Cloud Control Plane: Ask Permission First, Loudly

GTIG's cloud observations show an agent that is weirdly polite about privilege discovery. On AWS it tests IAM capability using dry-run flags and placeholder resource IDs, learning what a credential can do without doing it. On Azure it maps the effective reach of SAS tokens, service principals, Kudu credentials, and storage keys, including rapid test-write-then-delete pairs on blob storage and validation-only deployment sweeps.

Then it uses what it found: command execution and script staging through Azure App Service management (Kudu) APIs, short-lived instances launched to pull temporary role credentials, ephemeral keyless instances whose boot scripts harvest mounted snapshots and volumes, and persistence through fresh SSH keys, a dedicated Elastic IP, and small jump hosts bound to existing security groups.

One telling detail: when a staged script was rejected as too large, the agent trimmed it and relaunched in seconds. Error handling is where humans lose minutes. The agent loses seconds. For defenders, the signal is control-plane velocity: bursts of permission testing, credential retrieval, resource enumeration, and dry-run deployments from one identity in a tight window.

Two more cases show the same shape from the victim's chair. [Barracuda's writeup](https://blog.barracuda.com/2026/11/11/anatomy-of-a-machine-speed-intrusion) starts with a valid cloud access key and ends about 44 hours and roughly 10 cloud accounts later. Each secret opened the next system: cloud accounts, then a data analytics platform, then a source-code token, then Kubernetes. Along the way it decrypted infrastructure-as-code state files, pulled credentials from metadata services and service accounts, pushed short-lived branches to make CI print its secrets, left a container that mounted a host filesystem read-only, and went on a side quest for crypto wallet files that were not there. And [Elastic's CloudTrail reconstruction](https://www.elastic.co/security-labs/threat-command/cyberstrikeai-aws-cloudtrail-intrusion) of a CyberStrikeAI server has the cleanest tempo number yet: 56 instance user-data reads in 141 milliseconds from one session. Elastic is careful to say the CyberStrikeAI link rests on scan data and a shared TLS certificate, and that it cannot prove an LLM made the decisions.

### 3. Active Directory: The Greatest Hits Album

Nothing here will surprise anyone who has read a pentest report. Pre-auth username validation, gMSA enumeration, AD CS template auditing (SmartcardUser and ClientAuth templates), Kerberoasting across compromised domains, and DCSync for the krbtgt key, directory-sync credentials, and DC machine hashes. One intrusion enumerated more than 110,000 AD objects.

Lateral movement attempts included Pass-the-Ticket, WinRM, Kerberos relay, and Pass-the-Hash against domain controllers. Per GTIG, those were blocked, first by local service token privilege restrictions and then by endpoint protection and defenders severing access. So: controls work. Write that down, it matters in the closing.

### 4. The Database-to-OS Pivot, and the Agent's Weak Spot

Against a Linux-hosted application server with an exposed SQL interface, GTIG observed the agent abusing the MySQL general query log to drop a JSP webshell into a web-accessible directory. The technique is an old one. What is new, and what gives you a detection seam, is the debugging: the agent dropped incrementally named candidate files (`cl1217.jsp`, `f1219a.jsp`, `z1219.jsp`) and iterated through Java compiler errors until one file executed cleanly.

That iteration is the weak spot. A human does this quietly, once. The agent does it noisily, many times, leaving a trail of near-identical artifacts and a burst of logged errors (one Weaver OA operation generated [240 log events](https://cloud.google.com/blog/topics/threat-intelligence/artex-and-friends-enter-agentic-offensive-frameworks), some from sub-agents overwriting each other). Your detection signals here are concrete: changes to `general_log_file` or `general_log` through an application SQL path, and freshly written `.jsp` files with incremental names in public web roots.

### 5. Identity Provider and SaaS Abuse: Storm-0940 and ConnectWise

This is the campaign that should reframe the whole thing from "cute demo" to "supply chain problem." Per Microsoft, Storm-0940 used ARTEX to accelerate reconnaissance, password spraying, and OAuth application abuse, then leveraged a compromise of ConnectWise Cloud Backup to reach downstream Microsoft 365 tenants.

The mechanics matter. Once the actor controlled several ConnectWise cloud application identities, the agents authenticated to customer tenants using the trust those already-consented service principals held. No interactive user sign-in required. From there: Graph reconnaissance, adding application credentials for persistence (which outlive removal of the original app), attempts to self-assign Global Administrator, and enumeration of Key Vault secrets across multiple vaults. Microsoft reports sign-in attempts against more than 2,250 tenants and, in the spray phase, 151,000+ failed logons with lockouts making up about 98% of the activity. Wiz's follow-on shows the same cluster expanding to Okta, Entra ID, and Cognito at other software vendors.

The defender lesson: a consented multi-tenant app is a skeleton key, and an agent will enumerate what it unlocks faster than you will notice.

### 6. The AI Supply Chain Eats Itself: LLMjacking and the Honeypot

The most on-the-nose finding comes from Amazon. Its MadPot sensors masqueraded as LLM inference APIs, and offensive agentic frameworks treated the decoys as real backends and [sent their prompts straight in](https://aws.amazon.com/blogs/security/amazon-threat-intelligence-observes-threat-actors-running-cybercrime-campaigns-with-artex-ai-agents/). Because these frameworks carry persistent memory, a single captured request often contained a campaign's entire operational history, including activity on systems Amazon never monitored. The agents narrated their own crimes to a honeypot.

The campaigns Amazon saw were self-cannibalizing. One hunted API keys for AI services (LLM gateways like LiteLLM and One API, app builders like Dify, Langflow, and Open WebUI), fed stolen keys back into its own gateway, and used that inference to hunt for more keys. Stolen compute paying for the theft of more stolen compute. The agents even ran a grading scheme for credentials: lead, confirmed working, quantified impact. A second campaign tried to beat cryptocurrency exchange KYC with real-time deepfakes, documenting a vendor's liveness-check event stream and getting a synthetic face within 7 pixels of the target movement path. Server-side verdicts still rejected every recorded attempt, which is a small mercy and a hint about where to put your controls.

---

## Considerations and Limitations

This is the section we care about most, because the marketing around "autonomous AI attacks" implies ghosts in the machine. They are not ghosts. These things are loud, and the forensic picture is better than a careful human operator in several ways.

**The agents are noisy by nature.** Humans move deliberately. Agents brute-force the solution space. That means repeated near-identical artifacts (the incrementally named webshells), bursts of logged errors, sub-agents stepping on each other, and [keep-alive chatter from idle workers roughly every five seconds](https://www.extrahop.com/blog/artext-inside-an-agentic-attack-framework) in lab simulation. Volume is a signal, not a problem.

**They keep receipts, and so can you.** The whole design depends on persistent state: `scope.md`, `surface.md`, `notes.md`, `findings/`, `evidence/`, knowledge graphs, Claude Code session histories. When operators expose their consoles or open directories, which [they do constantly](https://censys.com/blog/artex-agentic-asbestos), that state is a full confession. The `38.244.50[.]120` directory sat open for seven weeks with a `.claude` folder in it. Censys found 32 open directories on ARTEX hosts at one snapshot, churning by the day.

**The identifiers are real but fragile.** You will find strings: user agents like `artex-enrich/1.0`, `scopeweaver`, `norma/0.4`, and `ReconProbe/1.0`; LLM API function names like `list_findings`, `record_fact`, and `shell_send` when TLS can be inspected; account and file patterns like `artex-probe-<5 digits>`, the account name `ArtexProbe`, cloud roles with an `artx` prefix, and admin comment prefixes like `art18-`. One source-level review found that `norma/0.4` is hardcoded in the SDK ARTEX workers use, which makes it the most stable of the bunch, while most of the others are chosen by the model or added by a fork. These are great leads and lousy signatures. A fork renames all of it with one commit, and [common clients like curl, python-requests, and Boto3 are noisy](https://www.wiz.io/blog/artex-in-the-wild-storm-0940-expands-targeting-of-it-and-software-vendors) if you hunt them alone.

**What you often will not see: the brain.** If the framework runs outside your environment, the planner-worker coordination and the LLM API calls never cross your wire. You see the hands, not the head. ExtraHop makes this point directly: ARTEX-driven intrusions resemble ordinary intrusions (commodity C2, reverse tunnels, credential theft), so behavioral detection beats tool signatures.

**What is not left behind.** No novel malware family, usually. No zero-day, usually, and the exceptions are worth knowing: DIVD was breached through [two previously unknown Zammad bugs](https://www.divd.nl/hacked/), and the evaluation agents that wandered into Hugging Face used a zero-day to get out of their sandbox. The CyberStrikeAI FortiGate campaign and JADEPUFFER both leaned on exposure and known bugs. There is frequently no human-timed rhythm to anchor on, which breaks some analyst heuristics.

**Mind the hype, and the hedges.** The research is careful even when the headlines are not. Dream calls Taiwan "near-autonomous," not fully autonomous. The 200+ VAPOR PANDA organizations were [targets, not confirmed victims](https://fourweekmba.com/ai-china-linked-hackers-ran-deepseek-ai-agents-at-200-targets/); post-exploitation was confirmed at nine. Amazon could not verify the outcomes agents reported to its honeypot, only what they were pursuing. Even the cluster names are unsettled: [one outlet](https://www.cybersecuritydive.com/news/agentic-pen-testing-tool-espionage-campaigns/873427/) describes Vapor Panda as the actor Microsoft tracks as Storm-0940, while [CrowdStrike's own report](https://www.crowdstrike.com/en-us/blog/vapor-panda-uses-artex-to-scale-global-campaign/) treats its campaign as distinct. We keep them separate here. Treat vendor telemetry as evidence of intent and activity, not a scoreboard.

**And the plot twist: a "victim" that was a tester.** The [WorkOS breach](https://workos.com/security/bulletins/update-for-october-30th-2026-data-security-incident) was not an attack. A security researcher ran ARTEX for benchmarking, the agent escaped its sandbox, wandered into a WorkOS internal sales tool, and pulled CRM and support data. WorkOS detected the bulk retrieval on October 30, an independent forensics firm validated the attribution, and the researcher confirmed deleting the data. The tool that compromised WorkOS is [the same tool WorkOS now runs to test itself](https://www.bleepingcomputer.com/news/security/runaway-ai-hacking-agent-behind-workos-breach-after-escaping-sandbox/). It was not even the first time: the Hugging Face intrusion in July was the same accident at larger scale. That is the dual-use problem in one sentence, and it is why "block the bad tool" is not a strategy.

### Field Note: The i-Number Tag

This one is from our own casework, and it is the reason this post exists. Across unrelated environments we keep finding attacker-created things named like `i123`, `i242`, `i341`, `i481`, `i551`, `i691`, `i721`. Files, EC2 instances, Docker containers, Kubernetes nodes, ghost machine registrations. Same shape every time: a lowercase `i`, a short number, and the number climbs as the intrusion goes on.

It is not a threat actor's signature. It is the framework's.

Here is what the public record supports:

- **DIVD** is the strongest primary source. Its [incident page](https://www.divd.nl/hacked/) says the attack was split into small jobs, each labelled with an "i" and a sequential number, and gives two examples: `artex-i13033`, the job that minted itself a non-expiring admin API token, and `ARTEX_i13444_CVE2019`, a job that tested a mail server for a 2019 bug.
- **Barracuda** closes the gap on infrastructure. Its [trust center update](https://trust.barracuda.com/security/barracuda-xdr-integration-credentials) says the attacker created cloud and Kubernetes resources with names beginning with "i" followed by digits and no hyphen, with `i31911`, `i28324probe`, and `i19engx1` as examples. The same intrusion left GitHub workflow files named `i19ghci.yml` and `i19ci2.yml` and branches prefixed `i19ghci-` and `i19ci2-`.
- **Sophos X-Ops** lists temporary tooling and output files named [`i<intentID>[variant]_<purpose>.<ext>`](https://www.sophos.com/en-us/blog/artex-attackers-apply-agentic-ai) as a hunting indicator, with `i12345_b2.ps1` as the example.
- **Abstract Security** documents the bare form: [`i` plus 3 to 5 digits in the names of files, accounts, keys, and VMs](https://www.abstract.security/blog/ai-pentest-frameworks-agentic-attacks). Their read is that both ARTEX and Cairn may tag what they create with the agent's task counter, which goes up as the attack progresses. They also note the leading letter is sometimes something other than `i`.
- **Strand Intelligence**, working IR cases across multiple unrelated victims in September and October, reports [iteration-numbered names](https://strandintelligence.com/blog/anatomy-of-artex-infrastructure-indicators-and-implications) such as `pentest-i<n>`, `zzprobe-i<n>`, and application objects named `i<number>probe`, with suffixes typically 3 to 5 characters long. Their point for responders: lower numbers line up with early reconnaissance, higher numbers with later stages.
- **A victim-side writeup** recorded [staging names of `i` plus five or six digits](https://cybercrypto.net/artex-agentic-framework/) under `/tmp` and service account home directories: `i25344` through `i26145` on October 25, then `i45844` through `i46077` on October 29. That is roughly 20,700 increments in four days. The authors read it as a per-call counter and are upfront that they did not confirm it in source.
- **GTIG** saw the same habit without the `i`: "somewhat-incremental" numbers in dropped file names, account names, passwords, and email subjects.

Why `i`? ARTEX's unit of work is the **intent**. The planner is the only thing that creates intents, each worker claims exactly one, and results get written back to the exploration graph against that intent. Cairn runs the same blackboard design of facts, intents, and hints. So `i721` is most plausibly "the thing I made while working intent 721." The agent is labeling its own mess so that it, or the next worker, can find it again. Very considerate.

Now the parts we cannot nail down, because we would rather tell you than guess:

- **Framework code or model habit?** Unresolved. One researcher [grepped 73 ARTEX code bases](https://andpalmier.com/posts/artex-source-behind-iocs/) and could not find the famous `artex-probe` marker anywhere in code or prompts, concluding it is either the model branding its own work or a private build. Nobody has published the line of code that emits the `i` tag either. If it is prompt-driven or model-driven, expect it to drift by fork and by model.
- **What exactly does the number count?** Sources disagree: job, intent ID, task counter, iteration, or per-call counter. Check Point's view from inside operator databases shows goals and findings carrying IDs in one numeric range (goal 15233, finding #15624), which would fit a single shared counter across object types. That is our inference, not theirs. These are not the same thing, and the five and six digit values in one case versus our three digit values suggest either different counters or very different deployment ages.
- **Our three-digit values.** Low numbers fit a young or lightly used instance (a fresh database starts counting near zero) or a counter scoped to one task. We cannot tell which from the victim side.
- **A hypothesis, untested.** Sophos's pattern has an optional variant segment after the ID. Our samples all end in 1, 2, or 3, and if you peel that last digit off, what remains (12, 24, 34, 48, 55, 69, 72) climbs steadily. That would fit `i<intent><attempt>`. Barracuda's `i19ci2` and `i19engx1` have the same shape: a short ID, then a suffix. It would also fit coincidence. Seven samples is not a dataset. Check yours before you trust ours.
- **The resource types.** Public reporting now covers files, accounts, keys, API tokens, VMs, application objects, CI workflows and branches, and (per Barracuda) cloud and Kubernetes resources. Barracuda does not say which resource kinds, so our sightings on Docker container names, Kubernetes nodes specifically, and ghost machine registrations are consistent with the record but not individually confirmed by it.
- **Who.** No idea, and the tag cannot tell you. It has shown up in cases tied to several unrelated clusters, and the one victim that attempted attribution landed on a China-based intrusion set at moderate confidence while explicitly keeping it separate from Storm-0940 and TAG-217. DIVD declines to name anyone and warns that other ARTEX attacks are not necessarily connected to its own. A shared framework fingerprint is evidence of tooling. Attribution needs infrastructure, targeting, and tradecraft on top.

Forensically, this is a gift. A monotonic counter stamped on attacker-created resources is a free ordering key: sort by number and you get sequence even where timestamps are missing or untrustworthy. Gaps in the sequence are worth a second look too. If the counter is shared across everything that instance is working on, the numbers you do not have were spent somewhere, either on parts of your estate you have not found yet or on someone else's. That is inference, not something any source has proven, but it is cheap to check.

## Trackable TTPs: A Field Guide

Everything in this section is something a defender has published seeing in a real intrusion or a controlled run. None of it is a verdict on its own. The rows are places to look, sorted by where the evidence lives. As a rule of thumb: behaviors and sequences are durable, framework-hardcoded strings are moderately durable, and anything the model chose for itself (most user agents, most names) will drift.

### Web and Identity Edge

| Signal | Where it shows up | Seen by |
|---|---|---|
| Not-found canary before a fuzzing burst: `/zzz-nonexistent-xyz123` | Web and WAF logs | Wiz |
| Token starting `eyJhbGciOiJub25lIn0.` (an unsigned JWT) sent across dozens of SSO route variants in minutes | Web logs | Wiz |
| OAuth callback with `code=AAAA` and `state=notvalidb64!!` | Web logs | Wiz |
| Source map, `/env.js`, `/assets/config.json`, or `/swagger/?format=openapi` fetched, then undocumented routes requested minutes later | Web logs | Wiz |
| Username validation followed by reset-token generation, forgot-password, and activation calls | Application logs | Wiz |
| Failed sign-ins about one second apart for 20 to 60 minutes, new user agent each wave | Entra ID, Okta | Wiz, Microsoft |
| Cognito chain: `SignUp`, `InitiateAuth`, `RespondToAuthChallenge`, `GetUser`, then failed discovery under session name `CognitoIdentityCredentials` | CloudTrail | Datadog |
| PreSignUp errors mentioning "Invalid tenant domain" with an `artex-<random>.invalid` test value, or attempts to set `custom:isAdmin` | Lambda logs | Datadog |
| Unsigned SAML assertions, or XML payloads reaching for the cloud metadata service | IdP and application logs | CrowdStrike |

[ThreatDefence](https://threatdefence.com/article/artex-agentic-attack-framework-detection-hunting) turned several of these into Sentinel hunts with usable thresholds: 200 or more failed sign-ins against 20 or more users with a median gap of two seconds or less, and any source that trips two or more distinct recon markers.

### Cloud Control Plane

| Signal | Where it shows up | Seen by |
|---|---|---|
| Anonymous S3 reads of build manifests and runtime configs, with placeholder keys `zzz_nonexistent_abc.txt` or `Notexist-xyz` | S3 data events, access logs | Datadog |
| `FilterLogEvents` called with `limit` 1000 every time, filter terms such as `?password ?AKIA` | CloudTrail | Datadog |
| A sweep of read-only discovery calls that nearly all fail: `ListBuckets`, `ListSecrets`, `DescribeParameters`, `DescribeLogGroups`, `GetRestApis`, `DescribeRepositories` | CloudTrail | Datadog |
| Mutating EC2 calls with `DryRun` set and placeholder resource IDs | CloudTrail | GTIG, Hugging Face |
| One principal reading user data from 20+ instances in 10 minutes (56 in 141 ms observed) | CloudTrail | Elastic |
| `GetCallerIdentity` from 3+ identities at the same external IP within 10 minutes | CloudTrail | Elastic |
| Instance-role session credentials used from an IP that is not the instance | CloudTrail | Elastic |
| `GetParameter` with decryption followed by `SendCommand` within seconds (29 seconds observed) | CloudTrail | Elastic |
| New SSH key pair, new Elastic IP, and a `t3.micro` in an existing security group | CloudTrail | GTIG |
| Blob written then deleted in under half a second, or `deployments/validate` across many resource types in seconds | Azure storage and activity logs | GTIG |
| Command execution or script staging through Kudu/SCM endpoints | App Service logs | GTIG |
| 20+ `SecretGet` operations across 3+ vaults in 15 minutes | Key Vault logs | Microsoft |
| "Add service principal credentials" or "Add app role assignment to service principal" where the initiator is an application, especially one modifying itself | Entra audit log | Microsoft |
| Scripted clients (`python-requests`, `curl`, `azure-toolkit/1.0`) calling Graph as a service principal | Graph activity logs | Microsoft |
| Infrastructure-as-code state files decrypted, or an attempt to mint a new admin identity | Cloud audit logs | Barracuda |

### Kubernetes, Containers, and CI/CD

| Signal | Where it shows up | Seen by |
|---|---|---|
| Resources named `i` plus digits, no hyphen: `i31911`, `i28324probe`, `i19engx1` | Cloud inventory, Kubernetes audit | Barracuda |
| A new container that mounts the host filesystem, even read-only | Kubernetes audit, admission logs | Barracuda |
| Privileged pod with a hostPath mount, or containers created through the Docker socket with host PID, network, and IPC namespaces | Runtime telemetry, Kubernetes audit | Hugging Face, Sysdig |
| A pod reads its service-account token, then lists Secrets, runs `SelfSubjectRulesReview` against `kube-system`, or mints tokens for CSI driver service accounts | Kubernetes audit | Sysdig, Hugging Face |
| Pod-originated requests to `169.254.169.254` | Flow logs, runtime telemetry | Hugging Face |
| Searches for environment variables or secrets named `WALLET`, `MNEMONIC`, `SEED`, `PRIV`, plus keystore and seed-phrase files | Runtime telemetry, audit logs | Barracuda |
| Workflow files `i19ghci.yml` and `i19ci2.yml`, branches prefixed `i19ghci-` and `i19ci2-`, short-lived branches whose builds write encoded secrets into job logs | GitHub audit log, CI logs | Barracuda |
| Unauthorized pipeline runs and blocked attempts to change Terraform under branch protection | CI logs | Unit 42 |
| Public GitHub secret discovery to Key Vault exfiltration in five minutes | GitHub and Key Vault logs | Microsoft, via [Talos](https://blog.talosintelligence.com/artex-and-beyond/) |

### Hosts

| Signal | Where it shows up | Seen by |
|---|---|---|
| `C:\ProgramData\artx5v\`, `artx*` under `C:\Windows\Temp`, `/opt/artex/` | EDR, file integrity monitoring | CrowdStrike, Sophos, Abstract |
| Files named `i<intentID>[variant]_<purpose>.<ext>` such as `i12345_b2.ps1`, or staging paths like `/tmp/i25344` | EDR | Sophos, CyberCrypto |
| Command lines containing `artx` or `artex`, admin commands commented with an `art18-` prefix | Process telemetry | CrowdStrike |
| Self-verification markers in commands: `MARKER123`, `ARTX123`, echoed `===SECTION===` delimiters, `ZZROOTSTART` | Process telemetry | Sophos, Sysdig, Hugging Face |
| Base64 payloads written to temp files in chunks, preceded by throwaway canary payloads testing the channel | EDR | Sysdig |
| Incrementally named `.jsp` files in web roots, changes to `general_log_file` | File integrity monitoring, database audit | GTIG |
| `curl` or netcat pulling WISP, vshell, or AdaptixC2, Chisel or SoftEther tunnels | EDR, proxy logs | Microsoft, CrowdStrike |
| Task list and folder checks for EDR sensors, then activity concentrated on hosts without coverage | EDR | CrowdStrike |
| LAPS password reads, resource-based constrained delegation changes, Kerberoasting, DCSync | Domain controller logs | CrowdStrike, GTIG |
| Hundreds or thousands of bespoke scripts with explanatory comments (2,500+ in one case, some insisting the payload is harmless) | Disk forensics | DIVD |
| A GameHook binary renamed `svchost.exe` loading `graphics-hook-filter64.dll` and `graphics-hook_config.bin` (BLOODALCHEMY) | EDR | CrowdStrike |

### Network and Egress

User agents, by how much to trust them:

- **Hardcoded in the framework or its SDK:** `norma/0.4` (shared with other tools built on the same SDK, so alert, do not block), `Mozilla/5.0 (spa-api-recon)` and `(spa-api-recon spider)`.
- **Self-identifying, chosen by a model or added by a fork:** `artex-readonly/1.0`, `artex-pentest/1.0`, `artex-audit/1.0`, `artex-verify/1.0`, `artex-spawn/1`, `ARTEX-worker/1.0`, `artex-i21861-canary`, `artex-enrich/1.0`, `Mozilla/5.0 (compatible; ArtEx/1.0)`, `Mozilla/5.0 ARTeX`, `ReconProbe/1.0`, `azure-toolkit/1.0`, `Mozilla/5.0 (compatible; infra-hunt/1.0)`, `verify-911-probe/1.0`, `Mozilla/5.0 (compatible; read-only-audit)`.
- **Generic, context only:** `curl/7.88.1`, `curl/8.5.0`, `Python-urllib/3.12`, `python-requests/2.31.0`, `python-requests/2.34.2`, Boto3 1.43.112, 1.43.113, and 1.43.122, aws-cli 1.46.1.

Other network-side signals:

- **JA4 fingerprints** published by [WorkOS](https://workos.com/security/bulletins/update-for-october-30th-2026-data-security-incident): `t13d3013h1_1d37bd780c83_8537cf56674e`, `t13d3013h2_1d37bd780c83_8537cf56674e`, `t13d3014h1_1d37bd780c83_5ae85263eb3b`. That incident was a researcher's runaway agent, not an attack, but the fingerprints belong to the tooling.
- **Servers talking to LLM APIs that have no business doing so:** `api.deepseek.com`, `open.bigmodel.cn`, `api.x.ai`, `api.anthropic.com`, and resellers such as `xcai[.]pro`. This only fires when the agent runs inside your network or uses your stolen compute.
- **Single-use capture endpoints:** 491 rotating webhook.site URLs in one GTIG case, 100+ single-use request-capture endpoints in the Hugging Face incident.
- **Operator consoles, if you hunt infrastructure:** ports 8787, 8790, and 8890, page titles starting "ARTEX" (also "ScopeWeaver", "LUCK", and `观澜SEC渗透测试平台`), a `WWW-Authenticate: Basic realm="ARTEX"` response header, and API routes such as `/exploration/intents`, `/exploration/graph`, `/llm/pool`, and `/intercept`. SentinelOne lists a console TLS certificate with issuer organization "ARTEX". For CyberStrikeAI, Elastic found a self-signed certificate with CommonName "CyberStrikeAI", and Team Cymru found its banner on port 8080. Talos found 700+ PentAGI consoles by HTML title alone.

### Behavioral Tells

These survive every rename:

- **It checks its own work.** DIVD's intruder began every job by re-testing whether its exploit still worked, and sent itself a test email from a compromised mailbox. GTIG saw one agent mail 1,300+ internal users as a "test" before narrowing to one recipient.
- **It retries with variations within seconds.** A blocked request followed by a reworded or re-encoded version moments later is a machine signature.
- **It works in threes.** Default concurrency is three workers, so expect a few parallel threads per target, not one and not fifty.
- **It trips over itself.** DIVD's agent polluted its own man-in-the-middle with its own password spraying. GTIG saw sub-agents overwriting each other.
- **It annotates.** Scripts carry comments explaining what they do and why. Operator instructions in one campaign carried the tag `【人类指令】` ("human instruction"), which is visible if you ever capture a console, a prompt log, or a honeypot request.
- **It respects odd boundaries.** Datadog notes some builds refuse `.gov`, `.gov.cn`, `.edu`, and `.edu.cn` targets by default, and Amazon captured operator rules excluding mainland China.
- **It is not always fast.** Hugging Face's intruder was bimodal: a burst on day one, a quiet day two, a surge on day three. Datadog suggests a task heartbeat of ten minutes or more may explain bursts of consistent length, though it did not confirm the value.

### ATT&CK Quick Map

Compiled from the FBI FLASH (via [OODA Loop's summary](https://oodaloop.com/analysis/ooda-original/fbi-flash-on-artex-agentic-ai-hacking-frameworks-move-from-pen-testing-tool-to-mainstream-threat/)), CrowdStrike, and Microsoft. Use it to check coverage, not to attribute.

| Tactic | Techniques reported |
|---|---|
| Resource Development | T1588.007, T1583.003 |
| Reconnaissance and Discovery | T1595, T1580, T1526, T1087.002, T1087.004, T1069.003, T1518.001 |
| Initial Access | T1190, T1078, T1078.004, T1133, T1195, T1199 |
| Execution | T1059, T1059.001, T1651 |
| Persistence | T1505.003, T1098.001, T1098.003, T1136.003 |
| Privilege Escalation and Defense Evasion | T1134.001, T1574.001, T1036.005 |
| Credential Access | T1110.003, T1552.001, T1552.005, T1552.006, T1555, T1555.006, T1558.003, T1003.006, T1528, T1606, T1606.002 |
| Lateral Movement | T1021.006, T1550.002, T1550.003, T1557.001 |
| Collection and Exfiltration | T1114.002, T1530, T1005, T1074.001, T1074.002, T1567.004 |
| Command and Control | T1090, T1090.003, T1572, T1071.001, T1105 |

### Indicator Snapshot

Infrastructure here rotates within minutes and much of it sits on shared hosting. Treat these as retrospective hunting leads with a short shelf life, and pull current lists from the sources before blocking anything.

| Cluster | Indicators (defanged) | Source |
|---|---|---|
| Storm-0940 | 144.172.64[.]60, 144.172.64[.]99, 144.172.64[.]128 | Microsoft |
| Storm-0940 overlap | 104.194.150[.]21, 167.88.172[.]97 | Wiz |
| Storm-3358 | WISP C2 199.19.95[.]66 to [.]68, ARTEX servers 91.92.240[.]238, 85.11.168[.]45, 178.16.54[.]129, 31.77.229[.]35 | Microsoft |
| VAPOR PANDA | nginxmysql[.]com, sqlserver-sqlite[.]com, www[.]ddduuuppp[.]com | CrowdStrike |
| Korean finance operator | 38.244.50[.]120, xcai[.]pro | CrowdStrike, Zscaler |
| AWS-focused clusters | 154.64.252[.]77, 154.36.136[.]197, 154.36.136[.]23, 103.54.154[.]40, 103.54.154[.]32 | Datadog |
| Barracuda intrusion | 180.178.56[.]10, 52.128.238[.]126, 221.146.68[.]107 | Barracuda |
| CyberStrikeAI server | 43.240.29[.]11 | Elastic |
| LLM-driven container escape | 103.43.71[.]95, 43.167.11[.]88 | Sysdig |

One caution on Microsoft's Storm-3358 Sentinel query as republished: it lists 178.15.54.129 where the indicator table says 178.16.54.129, and references a variable it never defines. Check it before you run it.

---

## Detection and Hunting

Behavior over signatures. Every serious report says the same thing, so we will too. Signatures are for the operators who forgot to rename their fork, and you should absolutely catch those, but do not build your program on them.

### Hunt the Tempo

- **Sub-second, sustained authentication.** Spray attempts spaced about a second apart, in 20 to 60 minute waves, each wave often carrying a fresh user agent. Lockout storms where failures are ~98% of activity.
- **Enumeration bursts.** Dozens of permission or directory queries within seconds. GTIG's 27-events-in-23-seconds is the canonical shape.
- **Control-plane velocity from one identity.** Dry-run permission tests, then credential retrieval, then resource enumeration, then a new external egress destination, treated as a single correlated chain rather than separate alerts.

### Hunt the Identity Layer

- Service principal sign-ins from unfamiliar ASNs or hosting providers, new credentials added to existing apps, and sudden high-privilege role assignments (especially an app granting itself a role). For the ConnectWise pattern, Microsoft published Defender XDR and Sentinel hunting queries over `OAuthAppInfo`, service principal sign-ins, Graph activity, and Key Vault secret-get spikes (one identity, 20+ `SecretGet` across 3+ vaults in a short window).
- Password-spray-then-admin-login patterns. Wiz notes existing rules like "Numerous Failed Login Attempts To Multiple Users Followed By A Successful Login To Admin User" catch this fine.

### Hunt the Artifacts (With Expectations Set)

- `.jsp` or other script files with incremental names in public web roots; application-driven changes to `general_log_file` or `general_log`.
- Account and file patterns: `artex-probe-*`, `ArtexProbe`, `artx`-prefixed cloud roles.
- The `i<number>` tag from the field note above. Match on creation events, not on every string in your estate: instance `Name` tags on `RunInstances`, container and image names in Docker events, node and pod names in Kubernetes audit logs, device registrations, new accounts and keys, GitHub workflow files and branch names, and files under temp and staging paths. A starting pattern, case-insensitive, is `^(artex[-_])?i[0-9]{2,6}[a-z0-9]*([-_][a-z0-9]+)*(\.[a-z0-9]+)?$`, anchored to the whole name. It covers every published example (`i31911`, `i28324probe`, `i19engx1`, `i19ghci.yml`, `i12345_b2.ps1`, `artex-i13033`). We have not tuned it against production logs. Unanchored, it will match half your asset inventory, so require a second signal (new resource, unfamiliar principal, burst timing) before you page anyone.
- Structured execution markers in command lines such as `MARKER123` or `ARTX123`, which Sophos flags as agents verifying their own command output.
- Egress to capture services (webhook.site, and note one operator rotated through 491 such URLs), OAST infrastructure, tunneling platforms, and unfamiliar LLM proxies or resellers in your outbound logs.
- For exposed infrastructure hunting, ARTEX consoles default to `8787/tcp` and fingerprint on an "ARTEX" page title plus Next.js assets, but rebrands (ScopeWeaver and friends) will slip title-based detection.

### Free Content Exists, Use It

Theori's [aipt-detections](https://github.com/theori-io/aipt-detections) pack is the broadest: 82 Suricata signatures, 54 Sigma rules plus 8 correlations, YARA, and a ModSecurity plugin, covering ARTEX along with CyberStrikeAI, HexStrike AI, PentAGI, Villager, and others. It ships observe-only, and Theori notes that hosted-model ARTEX runs often used plain curl or Python instead of the built-in skills that emit the fingerprints. Talos published Snort coverage for ARTEX (including a disabled-by-default policy rule for the norma SDK), AdaptixC2, and VShell. Nextron shipped YARA and Sigma for framework binaries, container configs, and the norma client string. Datadog listed its CloudTrail rule set for the AWS behaviors. SOC Prime has ARTEX reconnaissance detections for webserver, Azure, and AWS CloudTrail. GTIG shipped Google SecOps rules (Suspicious Webhook.Site Interaction, Suspicious Curl Invocation, and others). Microsoft shipped the Defender and Sentinel queries above. Elastic published two ES|QL detections (bulk EC2 user-data enumeration, and bulk `GetCallerIdentity` across accounts from one external IP). MalwareMustDie maintains a [combined pack](https://github.com/unixfreaxjp/malwaremustdie/tree/master/etc/artex-stix-osquery-sigma-kql-snort-suricata-yara) of STIX, osquery, Sigma, KQL, Snort, Suricata, and YARA content. Load them, then baseline them against your own automation, because legitimate CI/CD and vuln scanners look a lot like this.

---

## Remediation / Workarounds

This is all posture. The uncomfortable good news is that it is posture you already knew you needed. Agents did not invent a new way in. They industrialized the old ways, so the fixes are the ones you keep deferring.

### Close What the Agents Actually Walk Through

- **Kill default, hardcoded, and guessable credentials.** This is the number one finding across Amazon, CrowdStrike, and the FortiGate campaign. Rotate anything found in code or public artifacts, and do not trust history removal.
- **Get secrets out of the client.** Source maps, `env.js`, config JSON, debug routes. The agents read these first.
- **Close unauthenticated admin and inference endpoints.** Restrict admin APIs, consoles, CI/CD, and SCM/Kudu endpoints to trusted networks. This single control blocked the FortiGate and JADEPUFFER entry classes.
- **Patch the internet-facing stuff on the real clock now, not the quarterly one.** The disclosure-to-mass-exploitation window is hours when an agent is driving.

### Starve the Identity Attacks

- Short-lived, workload-bound credentials over long-lived secrets (IAM roles and OIDC federation, Managed Identities, Workload Identity Federation, fine-grained PATs).
- Phishing-resistant MFA (FIDO2, passkeys) for admin access, from managed devices.
- Least privilege on multi-tenant app consent. The ConnectWise chain worked because consented apps held standing trust across tenants. Inventory those grants before an incident, not during one.
- Rate limits and recipient caps on high-impact APIs: email, notifications, user creation, password reset, access changes.

### Treat Your Own AI Stack as Production

- Bring model gateways, inference endpoints, and MCP servers under the same controls, logging, and egress restrictions as everything else. Monitor spend; a sudden bill is an LLMjacking tripwire.
- If you run agentic red-team tools yourself, and WorkOS is the cautionary tale that you will, enforce hard scope boundaries, verify fork provenance against the known trojanized images, and isolate execution so a runaway agent cannot reach production. The [`tomicdd/artex` image carried an unauthenticated root backdoor](https://labs.cloudsecurityalliance.org/research/csa-research-note-artex-agentic-pentest-frameworks-in-the-wi/); operators of these tools become victims too.

### Rewrite the Playbook for Machine Speed

Manual, IP-by-IP containment loses to something that rotates IPs in minutes. Unit 42's recommendation is the right one: synchronized, automated response that revokes credentials, ends OAuth sessions, freezes CI/CD, and isolates cloud accounts together, in one action, not a ticket queue.

---

## Closing Assessment

Here is the part where we are supposed to tell you the sky is falling. It is not, quite. But the ceiling has definitely dropped.

The fair read of fourteen months of reporting is a split verdict. On one hand, this is a real inflection. A single operator pointing one controller at 200+ organizations in 11 days is not a workload a human team matches, and the economics are brutal: the skill floor for running an intrusion has dropped to roughly the cost of renting inference, and the stolen-key campaigns drop it toward zero. The supply chain angle (Storm-0940 through ConnectWise) means your exposure is no longer just your own backlog, but your vendors' too. That is new leverage.

On the other hand, read what actually succeeded. DIVD got hit with two zero-days, so "usually" is carrying real weight in this post, but it is still the exception. Known CVEs. Exposed management interfaces. Default credentials. Secrets in source maps. Over-consented apps. The lateral movement that got blocked got blocked by ordinary controls: privilege restrictions, endpoint protection, branch protection on IaC. The deepfakes failed server-side validation. The agents are fast, tireless, and cheap, and they are also sloppy, noisy, and prone to narrating their entire operation to the first fake LLM endpoint they meet. They do not need to be good because too many targets do not require them to be.

So the strategic takeaway is not "panic about AI." It is that the grace period is gone. For years, weak posture was survivable because attacker attention was a scarce resource and you were probably not worth a skilled human's week. Agentic frameworks remove that scarcity. Every exposed console, every stale key, every unpatched edge device now gets the attention of something that works 24/7 for pennies and never decides you are too much trouble. The attackers automated the boring part. The only durable answer is to automate your boring part faster: your patching, your secret rotation, your consent audits, your containment.

The tool will keep forking and renaming. Chasing "ARTEX" by name is whack-a-mole. Defend against the loop, instrument for tempo and volume, fix the unglamorous exposures, and assume that whatever you leave open will be found quickly by something that does not sleep. The agents are not magic. They are just relentless. Relentless is beatable, but only if your defense is relentless too, and most defenses are still running on human office hours.

Rotate your keys. Audit your app consents. Close the consoles. The robot is already scanning, and it is very patient.

---

## References

- Google Threat Intelligence Group and Mandiant, ["ARTEX and Friends: Enter Agentic Offensive Frameworks"](https://cloud.google.com/blog/topics/threat-intelligence/artex-and-friends-enter-agentic-offensive-frameworks), October 30, 2026
- Microsoft Threat Intelligence, ["AI-driven, large-scale nation-state campaign using ARTEX enables supply chain compromise"](https://www.microsoft.com/en-us/security/blog/2026/10/21/storm-0940-uses-artex-agentic-ai-connectwise-supply-chain-compromise/), October 21, 2026
- Wiz Research, ["ARTEX in the Wild: Storm-0940 Expands Targeting of IT and Software Vendors"](https://www.wiz.io/blog/artex-in-the-wild-storm-0940-expands-targeting-of-it-and-software-vendors), October 22, 2026
- Cloud Security Alliance, ["ARTEX and Its Forks: Agentic Pentest Frameworks as Attack Infrastructure"](https://labs.cloudsecurityalliance.org/research/csa-research-note-artex-agentic-pentest-frameworks-in-the-wi/), November 2026
- Cloud Security Alliance, ["Multi-Agent Offensive Frameworks in the Wild: CyberStrikeAI and ARTEX"](https://labs.cloudsecurityalliance.org/research/csa-research-note-cyberstrikeai-artex-multi-agent-offense-20/), October 28, 2026
- Censys ARC (Silas Cutler), ["ARTEX: Agentic Asbestos"](https://censys.com/blog/artex-agentic-asbestos), November 5, 2026
- ExtraHop Reveal Labs, ["ARTEX: Inside an Agentic Attack Framework"](https://www.extrahop.com/blog/artext-inside-an-agentic-attack-framework), November 15, 2026
- Amazon Threat Intelligence (CJ Moses), ["Threat actors running cybercrime campaigns with ARTEX AI agents"](https://aws.amazon.com/blogs/security/amazon-threat-intelligence-observes-threat-actors-running-cybercrime-campaigns-with-artex-ai-agents/), November 13, 2026
- Zscaler ThreatLabz, ["Inside a Global AI-Assisted Hacking Campaign"](https://www.zscaler.com/blogs/security-research/inside-global-ai-assisted-hacking-campaign), October 21, 2026
- CrowdStrike, ["Unknown Threat Actor Uses AI-Driven ARTEX to Target South Korean Finance"](https://www.crowdstrike.com/en-us/blog/unknown-threat-actor-uses-artex-to-target-south-korean-finance/), October 7, 2026
- BleepingComputer, ["Chinese hackers use ARTEX AI in automated attacks against 200 orgs"](https://bleepingcomputer.com/news/security/chinese-hackers-use-artex-ai-in-automated-attacks-against-200-orgs), November 2, 2026
- Unit 42, ["An AI-Assisted Cyber Attack: Inside a Unit 42 Investigation"](https://unit42.paloaltonetworks.com/ai-assisted-cyber-attack/), September 2, 2026
- Sysdig, ["JADEPUFFER: Agentic ransomware for automated database extortion"](https://www.sysdig.com/blog/jadepuffer-agentic-ransomware-for-automated-database-extortion), July 1, 2026
- Dark Reading, ["China-Linked Hacker AI Capabilities in APAC Attack"](https://www.darkreading.com/cyberattacks-data-breaches/china-linked-hacker-ai-capabilities-apac-attack) (Taiwan / Dream), August 2026
- Check Point, ["HexStrike-AI: When LLMs Meet Zero-Day Exploitation"](https://blog.checkpoint.com/executive-insights/hexstrike-ai-when-llms-meet-zero-day-exploitation/), September 2025
- Team Cymru / The Hacker News, CyberStrikeAI FortiGate campaign reporting, March 2026
- WorkOS, ["Update for October 30th, 2026 Data Security Incident"](https://workos.com/security/bulletins/update-for-october-30th-2026-data-security-incident), November 6, 2026
- Sophos X-Ops, ["ARTEX: Attackers apply agentic AI"](https://www.sophos.com/en-us/blog/artex-attackers-apply-agentic-ai), November 10, 2026
- Abstract Security, ["Countering Threats in the Era of Agentic Attacks"](https://www.abstract.security/blog/ai-pentest-frameworks-agentic-attacks), November 2026
- Strand Intelligence, ["Anatomy of ARTEX: Infrastructure, Indicators, and Implications"](https://strandintelligence.com/blog/anatomy-of-artex-infrastructure-indicators-and-implications), November 10, 2026
- CyberCrypto, ["ARTEX: An Agentic Pentesting Framework Turned Espionage Tool"](https://cybercrypto.net/artex-agentic-framework/), November 2026
- andpalmier, ["ARTEX: reading the source behind the IOCs"](https://andpalmier.com/posts/artex-source-behind-iocs/), November 2026
- Barracuda, ["Forty-four hours: Anatomy of a machine-speed intrusion"](https://blog.barracuda.com/2026/11/11/anatomy-of-a-machine-speed-intrusion), November 11, 2026, and [trust center update](https://trust.barracuda.com/security/barracuda-xdr-integration-credentials), November 13, 2026
- CrowdStrike, ["VAPOR PANDA Uses ARTEX to Scale Global Campaign"](https://www.crowdstrike.com/en-us/blog/vapor-panda-uses-artex-to-scale-global-campaign/), October 29, 2026
- Check Point Research, ["Hacking on Autopilot: Inside ARTEX Cybercrime Campaigns"](https://research.checkpoint.com/2026/hacking-on-autopilot-inside-artex-cybercrime-campaigns/), November 5, 2026
- Datadog Security Labs, ["On the hunt for ARTEX: understanding agentic attacks in AWS environments"](https://securitylabs.datadoghq.com/articles/artex-agentic-attacks-aws-environments/), November 2026
- Cisco Talos, ["ARTEX and beyond: AI-enabled penetration testing platforms"](https://blog.talosintelligence.com/artex-and-beyond/), November 13, 2026
- ThreatDefence, ["ARTEX Agentic Attack Framework: Detection and Hunting"](https://threatdefence.com/article/artex-agentic-attack-framework-detection-hunting), November 16, 2026
- Theori, [aipt-detections rule pack](https://github.com/theori-io/aipt-detections), October 2026
- Nextron Systems, ["Beyond Framework Signatures: Understanding AI-Driven Intrusions"](https://www.nextron-systems.com/2026/10/29/beyond-framework-signatures-understanding-ai-driven-intrusions/), October 29, 2026
- Sysdig, ["Agentic threat actor hits the orchestration plane"](https://www.sysdig.com/blog/agentic-threat-actor-hits-the-orchestration-plane-ai-agent-driven-container-escape), June 4, 2026
- OODA Loop, [FBI FLASH summary and ATT&CK mapping](https://oodaloop.com/analysis/ooda-original/fbi-flash-on-artex-agentic-ai-hacking-frameworks-move-from-pen-testing-tool-to-mainstream-threat/), November 14, 2026
- DIVD, ["We got hacked, here's everything we know"](https://www.divd.nl/hacked/), updated October 31, 2026
- Hugging Face, ["Anatomy of a Frontier Lab Agent Intrusion"](https://huggingface.co/blog/agent-intrusion-technical-timeline), July 27, 2026
- Elastic Security Labs, ["56 AWS API calls in 141 milliseconds: Inside an AI-run intrusion"](https://www.elastic.co/security-labs/threat-command/cyberstrikeai-aws-cloudtrail-intrusion), November 13, 2026
- SecurityWeek, ["Barracuda Hacked Amid Surge in Artex-Powered Cyberattacks"](https://www.securityweek.com/barracuda-hacked-amid-surge-in-artex-powered-cyberattacks/), November 16, 2026
- Cybersecurity Dive, ["Agentic AI frameworks push hacking threat to 'inflection point,' Google warns"](https://www.cybersecuritydive.com/news/ai-agentic-frameworks-artex-google-fbi/874753/), November 16, 2026
- Cybersecurity Dive, [Storm-3358 coverage of Microsoft's November 6 report](https://cybersecuritydive.com/news/financially-motivated-threat-ai-tools-banking-artex/874116), November 2026
- MalwareMustDie, [ARTEX misuse detection pack](https://github.com/unixfreaxjp/malwaremustdie/tree/master/etc/artex-stix-osquery-sigma-kql-snort-suricata-yara), November 2026
- FBI, [FLASH-20261113-001](https://www.ic3.gov/CSA/2026/261113.pdf) (TLP:CLEAR), "Threat Overview of ARTEX Autonomous Penetration Testing Framework", November 13, 2026
