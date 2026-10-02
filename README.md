# Awesome Agent Hardening [![Awesome](https://awesome.re/badge.svg)](https://awesome.re)

<p align="center">
  <img src="media/hello.gif" width="420" alt="Pica the magpie admires a shiny key, notices you, hides it under her wing and bows.">
</p>

> Limiting what AI agents can reach, change, and break.

This list is for developers who build agents. It assumes any model output can be the worst possible one: an attacker wrote it, the user asked for the wrong thing, or the model got it wrong on its own. Nothing here relies on the model behaving. Each section takes one thing an agent can be given, lists public incidents with their primary sources, and then lists controls that hold outside the model. Lines in italics are my own comments.

If the list saves you time, give it a star so other people who build agents can find it too.

Each control names the principle it applies:

| Principle         | What holds whatever the model says                                                                 |
| ----------------- | -------------------------------------------------------------------------------------------------- |
| Least privilege   | The agent reaches only what was granted for this task.                                             |
| Isolation         | What the agent runs stays in a sandbox with no secrets, and nothing outside trusts what it writes. |
| Data flow control | Data goes only to people allowed to read everything it came from.                                  |
| Output handling   | Model output is data; the system decides what it may trigger.                                      |
| Recoverability    | Any agent mistake can be rolled back, and the system caps how big it can get.                      |

## Contents

- [Principles](#principles)
- [Model Only](#model-only)
- [Mail and RAG](#mail-and-rag)
- [Tools and MCP](#tools-and-mcp)
- [Shell and Files](#shell-and-files)
- [Browser](#browser)
- [Memory](#memory)
- [Skills and Marketplaces](#skills-and-marketplaces)
- [Unattended Agents](#unattended-agents)
- [Identity and Tokens](#identity-and-tokens)
- [Human Approval and Classifiers](#human-approval-and-classifiers)
- [Testing](#testing)
- [Frameworks](#frameworks)
- [Lectures](#lectures)

## Principles

- [The lethal trifecta for AI agents](https://simonwillison.net/2025/Jun/16/the-lethal-trifecta/) - An agent with private data, untrusted content and a way to send data out can be made to leak it. *Don't try to remove the untrusted leg by sorting inputs into trusted and untrusted, because the user's own request or a plain model mistake produces the worst output too; cut access to private data or the way out instead*
- [Design Patterns for Securing LLM Agents against Prompt Injections](https://arxiv.org/abs/2506.08837) - Six patterns that limit what an agent can still do after it has read untrusted input. *Try to narrow a tool before you refuse it. After a flat no the team wires the same source up locally under its own account, and then nobody sees it at all*
- [Defeating Prompt Injections by Design](https://arxiv.org/abs/2503.18813) - CaMeL takes control flow from the trusted request, so injected text cannot change which tools run.
- [CaMeL](https://github.com/google-research/camel-prompt-injection) - Data flow control: research code that reproduces the paper above on AgentDojo; the authors warn that it is unmaintained and may not be fully secure.
- [Secure Computer System: Unified Exposition and Multics Interpretation](https://csrc.nist.gov/files/pubs/conference/1998/10/08/proceedings-of-the-21st-nissc-1998/final/docs/early-cs-papers/bell76.pdf) - Data flow control: the 1976 Bell and LaPadula report on confidentiality labels, where a subject may not read above its level or write below it. *It's fifty years old and I still send people here first. Once "no write down" clicks, the lethal trifecta stops looking like a new problem*
- [Securing AI Agents with Information-Flow Control](https://arxiv.org/abs/2505.23643) - Data flow control: FIDES from Microsoft Research labels data by confidentiality and integrity and enforces the policy in the planner, outside the model.
- [FIDES](https://github.com/microsoft/fides) - Data flow control: code and a tutorial for the paper above.
- [APPA: Recoverable Information-Flow Control for Real-World LLM Agents](https://arxiv.org/abs/2607.24625) - Data flow control: labels follow what the agent has read, each tool call is checked before it runs and its output before it enters the context, and work on untrusted data continues in a disposable branch.
- [OpenAPPA](https://github.com/archestra-ai/OpenAPPA) - Data flow control: the open-source engine for the paper above. A TOML policy says who may read each piece of data, and a call that would send it to anyone else is refused before it runs. Its README calls it a preview and an RFC whose config and wire formats may still break. *This is the closest I've seen to what I do by hand in a review: take who was allowed to read the data and compare it with who is about to receive it. It's raw, and the benchmark is the authors' own, so I'd try it on one agent before believing the numbers*
- [The Attacker Moves Second](https://arxiv.org/abs/2510.09023) - Adaptive attacks broke twelve published prompt injection defenses, so a filter can tell you something is off but cannot stop the leak.

## Model Only

<p align="center">
  <img src="media/model-only.gif" width="520" alt="Pica pecks a shiny refund button faster and faster, coins pile up, then she notices you and looks away.">
</p>

- [Moffatt v. Air Canada, 2024 BCCRT 149](https://loyaltylobby.com/wp-content/uploads/2024/02/Air-Canada-Tribunal-2024bccrt149.pdf) - A tribunal held the airline to a refund rule its chatbot made up. Prices and policy have to come from the system, not from model text.

## Mail and RAG

<p align="center">
  <img src="media/mail-and-rag.gif" width="520" alt="Pica checks a shiny note that says refund all, hesitates, and the shine wins.">
</p>

### Incidents

- [EchoLeak](https://www.aim.security/lp/aim-labs-echoleak-blogpost) - Zero-click leak from Microsoft 365 Copilot: instructions in an email made the answer carry data in an image URL, and the browser fetched it through a Teams proxy.
- [EchoLeak: The First Real-World Zero-Click Prompt Injection Exploit](https://arxiv.org/abs/2509.10540) - The paper on the same chain, with the classifier bypass and the reference-style Markdown image.
- [CVE-2025-32711](https://msrc.microsoft.com/update-guide/vulnerability/CVE-2025-32711) - Microsoft fixed it on the server side, and customers had nothing to do.

### Controls

- [How Microsoft defends against indirect prompt injection attacks](https://www.microsoft.com/en-us/msrc/blog/2025/07/how-microsoft-defends-against-indirect-prompt-injection-attacks) - Output handling: known ways out, such as Markdown images, are blocked deterministically, while hardened prompts and classifiers are counted as probabilistic layers on top.
- [Best practices to render streamed LLM responses](https://developer.chrome.com/docs/ai/render-llm-responses) - Output handling: treat model output as user-generated content, sanitize the whole accumulated response because a payload can be split across chunks, and stop rendering once the sanitizer removes anything.
- [Practical LLM Security Advice from the NVIDIA AI Red Team](https://developer.nvidia.com/blog/practical-llm-security-advice-from-the-nvidia-ai-red-team/) - Output handling: the three findings the red team keeps seeing (executed model code, loose permissions on RAG stores, rendered active content) and a fix for each, such as an image content security policy limited to known sites. *Read permissions miss one case. A search index serves an approved policy and somebody's draft the same way, the answer drops the author and the page a reader used to judge by, and every access along the way is legitimate. So decide who may write into the corpus, and take the author from the source system's metadata, because a line saying "Author: Compliance" in the body proves nothing*
- [The dangers of AI agents unfurling hyperlinks and what to do about it](https://embracethered.com/blog/posts/2024/the-dangers-of-unfurling-and-what-you-can-do-about-it/) - Output handling: a chat platform that previews links fetches whatever URL the model wrote, so a Slack app should post with `unfurl_links` and `unfurl_media` set to false.
- [A Security Model for Full-Text File System Search in Multi-User Environments](https://www.usenix.org/conference/fast-05/security-model-full-text-file-system-search-multi-user-environments) - Data flow control: a 2005 paper showing that a shared index with permissions applied as a postprocessing step still leaks what is in files a user cannot read; a RAG index shared by many users has the same problem.
- [Implement multitenancy](https://docs.pinecone.io/guides/index-data/implement-multitenancy) - Data flow control: one namespace per tenant keeps each customer's records stored apart and binds every query to a single namespace, which lowers the risk of a bug returning another tenant's data.
- [About Dangerzone](https://dangerzone.rocks/about/) - Isolation: opens an untrusted document inside a gVisor sandbox with no network, turns the pages into pixels and rebuilds a PDF from them outside, so only what the pages look like gets through.
- [DOMPurify](https://github.com/cure53/DOMPurify) - Output handling: an HTML sanitizer that works on the parsed DOM, with allow and forbid lists for tags. *It's built against XSS, so out of the box it keeps `<img src>` pointing anywhere, and that's the channel the leaks above went through. Put `img` and `form` into `FORBID_TAGS` yourself*
- [react-markdown](https://github.com/remarkjs/react-markdown) - Output handling: a React Markdown renderer that is safe by default: it doesn't use `dangerouslySetInnerHTML`, and `disallowedElements` and `urlTransform` let you drop elements and rewrite URLs. *Safe by default means no script runs. A Markdown image still loads from whatever `http` or `https` URL the model wrote, so I'd disallow `img` or run every URL through an allowlist*

## Tools and MCP

<p align="center">
  <img src="media/tools-and-mcp.gif" width="520" alt="Pica wearing a user badge opens a private box and pins the salary sheet on a public board.">
</p>

### Incidents

- [GitHub MCP Exploited: Accessing private repositories via MCP](https://invariantlabs.ai/blog/mcp-github-vulnerability) - A public issue led an agent holding the user's token to copy private repository data into a public pull request.
- [Supabase MCP Security: How Prompt Injection Leaked Private Tables](https://generalanalysis.com/blog/supabase-mcp-blog) - A support ticket told Cursor, connected with the service role key, to read a table of integration tokens and post it back into the ticket. *For SQL that the model writes, a read-only role on the live database is weaker than it looks. The whole query still runs through the parser, the planner and the built-in functions, and the role opens every table it covers. I'd point the agent at a copy that holds only what every one of its users is allowed to see*
- [Asana MCP server incident](https://status.asana.com/incidents/5b9hhtgs3mxs) - A bug could show one organization's data to MCP users in other organizations; Asana kept the server offline for twelve days.
- [Malicious postmark-mcp package](https://postmarkapp.com/blog/information-regarding-malicious-postmark-mcp-package) - A fake Postmark MCP server on npm quietly copied every sent email to the attacker.

### Controls

- [MCP Security Best Practices](https://modelcontextprotocol.io/docs/2026-07-28/tutorials/security/security_best_practices) - Least privilege: why a server must not pass client tokens through or act as a confused deputy. *The tool description is written by the server's developer, who is exactly the party you're restricting, so I wouldn't let it feed an access decision*
- [MCP Authorization Security Considerations](https://modelcontextprotocol.io/specification/2026-07-28/basic/authorization/security-considerations) - Least privilege: what the specification requires of tokens and scopes.
- [Progent: Securing AI Agents with Privilege Control](https://arxiv.org/abs/2504.11703) - Least privilege: every tool call is checked against rules over tool names and arguments, and a policy update that widens access needs explicit approval. *Write the rule as an allow for the values you expect. A deny laid over a general allow falls away as soon as the field is missing or arrives as another type*
- [Before the Tool Call: Deterministic Pre-Action Authorization for Autonomous AI Agents](https://arxiv.org/abs/2603.20953) - Least privilege: a declarative policy is evaluated before each tool call runs; in the author's bounty test social engineering worked on the model 74.6% of the time under a permissive policy and never in 879 attempts under a restrictive one.
- [Authorization API 1.0](https://openid.net/specs/authorization-api-1_0.html) - Least privilege: the OpenID AuthZEN specification for asking an external policy decision point whether a subject may perform an action on a resource, which keeps the decision in one place outside the agent.
- [COAZ-MCP: COAZ Binding for the Model Context Protocol](https://openid.github.io/authzen/authzen-coaz-mcp-binding-1_0.html) - Least privilege: a draft that maps MCP messages onto the Authorization API above, so a gateway or server can authorize a call down to its parameters.
- [Securing MCP: A Control Plane for Agent Tool Execution](https://developer.microsoft.com/blog/securing-mcp-a-control-plane-for-agent-tool-execution) - Least privilege: MCP has no point where policy is checked before a call runs, so Microsoft's open-source toolkit puts one between the client and the servers; with safety instructions in the prompt alone its red-team benchmark recorded a 26.67% policy violation rate.
- [LangChain Tools](https://docs.langchain.com/oss/python/langchain/tools) - Least privilege: a `ToolRuntime` parameter is filled in by the framework and left out of the schema the model sees, so values like the user ID come from the session and the model cannot choose them.
- [Server Side Request Forgery Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet.html) - Output handling: how to check a host or URL before fetching it, which is what a fetch tool has to do with an argument the model wrote: compare against an allowlist, use the parser's output value and switch redirects off. *Show the value to the person, compare it at execution and check it in the tool in one parsed form. Python's `urllib` reads the host of `http://example.com\@evil.com` as `evil.com`, and Node's `URL` reads it as `example.com`*
- [Cedar](https://github.com/cedar-policy/cedar) - Least privilege: an open-source policy language and engine where nothing is allowed without a permit rule and any matching forbid rule wins. *Everyone already knows [OPA](https://www.openpolicyagent.org/). I'd still look at Cedar: the language is small and policies get validated against a schema, so it's harder to write one that does the wrong thing. Do check what it does on an evaluation error, though. Cedar skips a policy that errors, so a broken forbid rule quietly turns into an allow unless you treat any diagnostics as a deny*
- [agentgateway](https://github.com/agentgateway/agentgateway) - Least privilege: an open-source proxy for agents and MCP servers that authorizes each tool call before relaying it. *Authorizing a call by tool name is a solved problem, so I'd take it off the shelf. Its RBAC rules don't see the arguments, though, so limits on values still belong in the tool or the system behind it*
- [ContextForge](https://github.com/IBM/mcp-context-forge) - Least privilege: IBM's open-source gateway, registry and proxy in front of MCP, A2A and REST APIs. *Out of the box it's one permission for every tool, and per-tool or per-argument decisions come from a policy plugin. Check that the token going downstream still names the user: with client credentials it doesn't*
- [Istio Authorization Policy](https://istio.io/latest/docs/reference/config/security/authorization-policy/) - Least privilege: allow and deny rules between workloads by host, method and path, for teams that build their own gateway on a mesh. *A mesh will authorize the request, but it holds no credentials and swaps nothing in, so the token exchange and the tool-level checks are yours to write*

## Shell and Files

<p align="center">
  <img src="media/shell-and-files.gif" width="560" alt="Staging refuses Pica, she digs an old railway token out of her nest, flies past staging, and prod accepts it.">
</p>

### Incidents

- [Your AI wants to nuke your database](https://blog.railway.com/p/your-ai-wants-to-nuke-your-database) - Railway on a coding agent that found an account-scoped API token on the user's machine and deleted a production volume; the data was recovered.
- [AI coding tool Replit wiped a database](https://fortune.com/2025/07/23/ai-coding-tool-replit-wiped-database-called-it-a-catastrophic-failure/) - During a code freeze the agent ran destructive commands against production, then claimed rollback was impossible.
- [AWS-2025-015](https://aws.amazon.com/security/security-bulletins/AWS-2025-015/) - An over-scoped GitHub token let an attacker ship a wiper prompt in the Amazon Q Developer extension, which failed to run.
- [S1ngularity: What Happened, How We Responded, What We Learned](https://nx.dev/blog/s1ngularity-postmortem) - Malicious Nx releases drove local AI coding CLIs to search the machine for secrets.
- [The Week of Sandbox Escapes](https://www.pillar.security/blog/the-week-of-sandbox-escapes) - Pillar Security got out of the sandboxes of Cursor, Codex CLI, Gemini CLI and Antigravity without breaking any of them: the agent wrote a Python interpreter, a Git hook or a task config that the host later ran, or reached the Docker socket.

### Controls

<p align="center">
  <img src="media/isolation.gif" width="520" alt="The same scene under a sandbox dome: the nest holds no tokens and Pica bumps into the dome on her way to prod.">
</p>

- [Making Claude Code more secure and autonomous with sandboxing](https://www.anthropic.com/engineering/claude-code-sandboxing) - Isolation: the filesystem and the network are isolated together, and permission prompts dropped by 84%.
- [Practical Security Guidance for Sandboxing Agentic Workflows and Managing Execution Risk](https://developer.nvidia.com/blog/practical-security-guidance-for-sandboxing-agentic-workflows-and-managing-execution-risk/) - Isolation: three controls NVIDIA's red team treats as mandatory at the OS level: block network egress to unknown hosts, block writes outside the workspace, and block writes to agent configuration files wherever they are. *Hang each rule on something the enforcement point can see. A coding agent commits under the developer's account, so a pipeline rule about "the agent's commits" has nothing to match on*
- [Under the hood: Security architecture of GitHub Agentic Workflows](https://github.blog/ai-and-ml/generative-ai/under-the-hood-security-architecture-of-github-agentic-workflows/) - Isolation: the agent runs in a container with no secrets and firewalled egress, and its writes are staged, capped in number and checked after it exits.
- [How we contain Claude across products](https://www.anthropic.com/engineering/how-we-contain-claude) - Isolation: three containment designs for claude.ai, Claude Code and Cowork, written after users approved about 93% of permission prompts; credentials that never enter the sandbox cannot leave it.
- [gVisor Security Model](https://gvisor.dev/docs/architecture_guide/security/) - Isolation: what a user-space kernel protects against, which is untrusted code exploiting bugs in the host kernel, and what it leaves open, such as hardware side channels.
- [OpenAI Shell tool](https://developers.openai.com/api/docs/guides/tools-shell) - Isolation: the hosted shell reaches the network only through a domain allowlist, and with `domain_secrets` a sidecar adds the real credential for an approved domain while the model sees a placeholder.
- [OpenShell](https://github.com/NVIDIA/OpenShell) - Isolation: NVIDIA's open-source runtime that runs an agent in a kernel-level sandbox, checks every network connection against policy and adds credentials only to requests bound for approved endpoints. It reached version 0.1 in September 2026. *The part I'd take first is the credentials: the agent never sees the real key. It's young, so I wouldn't build a year of policy on it yet*

## Browser

- [Agentic browser security: indirect prompt injection in Perplexity Comet](https://brave.com/blog/comet-prompt-injection/) - Hidden text in a Reddit comment made the browser agent open the user's logged-in email, read a one-time code and post it in the thread.

## Memory

- [Spyware Injection Into Your ChatGPT's Long-Term Memory](https://embracethered.com/blog/posts/2024/chatgpt-macos-app-persistent-data-exfiltration/) - A prompt injection wrote itself into ChatGPT's long-term memory and leaked later conversations through image URLs.

## Skills and Marketplaces

- [Malicious ClawHub skills target OpenClaw users](https://opensourcemalware.com/blog/malicious-clawhub-skills-target-openclaw-users) - 386 fake crypto trading skills talked users into running stealers for exchange keys and wallets.
- [Malicious OpenClaw skills used to distribute Atomic macOS Stealer](https://www.trendaisecurity.com/en-us/resources-insights/trendai-security-blog/malicious-openclaw-skills-used-to-distribute-atomic-macos-stealer) - The setup steps of these skills installed an infostealer on the user's Mac.

## Unattended Agents

- [OpenClaw Defaults Ship Insecure and Shodan Already Found Them](https://www.toxsec.com/p/openclaw-is-a-wildly-insecure) - Gateways open to the internet trusted anything that looked like local traffic.
- [Widespread OpenClaw Exploitation by Multiple Threat Groups](https://flare.io/learn/resources/blog/widespread-openclaw-exploitation) - Flare counted more than 30,000 compromised instances used to steal keys and read messages.
- [Hacking Moltbook](https://www.wiz.io/blog/exposed-moltbook-database-reveals-millions-of-api-keys) - A social network for agents left its database open without row-level security and exposed 1.5 million API keys.
- [What security teams need to know about OpenClaw](https://www.crowdstrike.com/en-us/blog/what-security-teams-need-to-know-about-openclaw-ai-super-agent/) - What a personal super-agent can reach once it is installed.

## Identity and Tokens

<p align="center">
  <img src="media/identity.gif" width="600" alt="Pica puts her own token into a card reader and every drawer opens: alice, bob, carol.">
</p>

### Incidents

- [Salesloft Drift AI supply chain attack](https://www.finra.org/rules-guidance/guidance/salesloft-drift-AI-supply-chain-attack) - Attackers used stolen OAuth tokens of a chat agent integration to export Salesforce data; FINRA counts more than 700 affected organizations.
- [Data theft from Salesforce instances via Salesloft Drift](https://cloud.google.com/blog/topics/threat-intelligence/data-theft-salesforce-instances-via-salesloft-drift) - Google Threat Intelligence on how the integration's tokens were used. The model played no part.
- [Cloudflare's response to the Salesloft Drift incident](https://blog.cloudflare.com/response-to-salesloft-drift-incident/) - One victim's account: its support cases held 104 API tokens that customers had pasted, and Cloudflare rotated them all.

### Controls

- [RFC 8693, OAuth 2.0 Token Exchange](https://www.rfc-editor.org/rfc/rfc8693) - Least privilege: one narrow token that names both the user and the agent acting for them.
- [OWASP Non-Human Identities Top 10](https://owasp.org/www-project-non-human-identities-top-10/) - Least privilege: the usual ways service and agent credentials go wrong.
- [Software and AI agent identity and authorization](https://www.nccoe.nist.gov/sites/default/files/2026-02/accelerating-the-adoption-of-software-and-ai-agent-identity-and-authorization-concept-paper.pdf) - Data flow control: a NIST NCCoE concept paper on identifying and authorizing agents.
- [RFC 8707, Resource Indicators for OAuth 2.0](https://www.rfc-editor.org/rfc/rfc8707) - Least privilege: the client names the resource it wants a token for, so the server can issue one that works at that resource only.
- [RFC 9449, OAuth 2.0 Demonstrating Proof of Possession (DPoP)](https://www.rfc-editor.org/rfc/rfc9449) - Least privilege: a token bound to the client's key, so whoever copies the token also needs the key to use it.
- [Identity Assertion JWT Authorization Grant](https://datatracker.ietf.org/doc/draft-ietf-oauth-identity-assertion-authz-grant/) - Least privilege: an IETF draft, also called Cross-App Access, in which the company identity provider decides whether an application may get a token for another application's API on the user's behalf.
- [Cross-App Access](https://oauth.net/cross-app-access/) - Least privilege: a plain-language explainer of the draft above with a list of identity providers, clients and authorization servers that implement it.
- [Transaction Tokens](https://datatracker.ietf.org/doc/draft-ietf-oauth-transaction-tokens/) - Least privilege: an IETF draft for a short-lived token that carries the user, the workload and the authorization context of one request through the services that handle it.
- [Credential injector](https://www.envoyproxy.io/docs/envoy/latest/configuration/http/http_filters/credential_injector_filter) - Isolation: an Envoy filter that adds the credential to outgoing requests in a sidecar, so the workload behind the proxy sends them without the secret.
- [Credential Brokering Patterns for AI Agents Part 1: Don't Give the Agent the Keys](https://blog.christianposta.com/credential-brokering-patterns-for-ai-agent-egress/) - Isolation: compares short-lived tokens, key-bound tokens and a gateway that attaches the real credential on the way out, and argues for the gateway; the author works for a gateway vendor.

## Human Approval and Classifiers

<p align="center">
  <img src="media/approvals.gif" width="520" alt="Pica approves requests faster and faster with drooping eyelids, approves volumeDelete prod with her eyes shut, and the blast leaves her covered in soot.">
</p>

*A "guardrail" product is usually three things in one box: DLP, an AI safety filter and a WAF. I find that an odd bundle. Security has always looked after confidentiality, integrity and availability. You can stretch that triad, but I wouldn't stretch it into product territory, where the business decides for itself what is acceptable. If the security team ends up answering for "the model said a bad word", something has gone strange*

- [Auto mode is now the default in Claude Code](https://claude.com/blog/auto-mode-default-in-claude-code) - In Anthropic's test people caught 13.6% of dangerous commands and the classifier caught 89%. Both still miss some, so they belong inside a sandbox. *An approval only counts for what a person can check at a glance. When data is sent out, that is the recipient: nobody reads three pages of body text looking for someone else's data*
- [How we built Claude Code auto mode](https://www.anthropic.com/engineering/claude-code-auto-mode) - How the permission classifier is built and which overeager actions it still misses.
- [AuthZEN Access Request and Approval Profile](https://openid.github.io/authzen/authzen-access-request-approval-profile-1_0.html) - Least privilege: a draft in which a denied call can open an approval request, the denial stays a denial until then, and the policy decision point checks again at enforcement time after approval. *Make sure the approver can't also steer what the model reads, otherwise one person can both trigger the action and sign it off*

## Testing

*Most of the time these tools burn compute testing alignment, which is a safety question: will the model say something it shouldn't. The labs that train the models already measure that and publish it, see the [Anthropic system cards](https://www.anthropic.com/system-cards) and the [OpenAI Deployment Safety Hub](https://deploymentsafety.openai.com/). AI security asks what the model's output can reach in your system. So write the threat model first and only then see whether a scanner answers anything in it. If you don't train models yourself, I honestly don't know what you'd need one for in an enterprise*

- [Promptfoo](https://www.promptfoo.dev/docs/red-team/) - Generates adversarial inputs for an LLM application and evaluates the responses.
- [garak](https://github.com/NVIDIA/garak) - NVIDIA's LLM vulnerability scanner.
- [PyRIT](https://github.com/microsoft/PyRIT) - Microsoft's open-source framework for finding risks in generative AI systems.

## Frameworks

- [OWASP Top 10 for LLM Applications](https://genai.owasp.org/llm-top-10/) - The common risk list, handy as a checklist once the controls above are in place.
- [MITRE ATLAS](https://atlas.mitre.org/) - Adversary tactics and techniques against AI systems, in the style of ATT&CK.
- [MAESTRO](https://cloudsecurityalliance.org/blog/2025/02/06/agentic-ai-threat-modeling-framework-maestro) - The Cloud Security Alliance's threat modeling framework for agentic AI.

## Lectures

[Безопасность агентов для тех, кто их строит](lectures/shad-2026-09.md), Yandex School of Data Analysis, 29 September 2026, in Russian, with [slides](lectures/shad-2026-09.pdf). One agent gets its capabilities one at a time, and each one comes with a real incident and the place to stop it.

## Contributing

Contributions are welcome. Please read the [contribution guidelines](contributing.md) first.

## Footnotes

Compiled by [Alexander Goncharov](https://www.linkedin.com/in/alexander-goncharov-600510234/). Pica the magpie is the mascot of his talks, drawn after Charley Harper. Each scene shows the problem its section is about.
