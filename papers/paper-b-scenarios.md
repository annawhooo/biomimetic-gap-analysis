# Paper B — Attack Scenario Template (LOCKED)

## Template Fields

Every scenario follows this structure exactly. No field is optional.
If a field doesn't apply, state why rather than omitting it.

```
## Scenario #N — [Name]

**Source mechanism:** [biological mechanism name]
**Source organism:** [species or class]

**Biology:** [What the pathogen/organism does. 2-3 sentences. Cite
primary source.]

**Abstract pattern:** [The substrate-independent logic. One sentence.
No biology words, no cybersecurity words — pure structural pattern.]

**Agent security equivalent:** [How this works against agent frameworks.
2-3 sentences. Specific to MCP/LangChain/CrewAI where applicable.]

**Feasibility:**
- Default config: [TRIVIAL / FEASIBLE / ADVANCED / THEORETICAL]
- Hardened config: [grade + what hardening changes]

**Framework evidence:** [Specific framework documentation, spec
language, or security analysis that confirms the mechanism exists.
Cite reference numbers.]

**Detection signature:** [What to look for. Actionable — a SOC analyst
or detection engineer should be able to write a rule from this.]

**Defensive implementation:** [How to build the defense. 2-3 sentences.
Include operational caveats.]

**Extend from:** [Existing enterprise tools that can be extended to
deliver this. Name specific products/categories.]

**Countering principles:** [Which of the 35 design principles address
this, by number and name.]

**ATT&CK mapping:** [Tactic and technique ID, or "Extends beyond
ATT&CK" with explanation.]

**ATLAS mapping:** [ATLAS technique ID if applicable, or "No direct
ATLAS mapping" with explanation.]

**Cross-references:** [Other scenarios that relate — compose, overlap,
or contrast. Brief note on the relationship.]
```

---

## TEMPLATE EXAMPLE 1: Scenario #1

## Scenario #1 — Telemetry Suppression

**Source mechanism:** MHC-I downregulation
**Source organism:** Viruses (Herpesviridae, Poxviridae, Adenoviridae)

**Biology:** Viruses reduce MHC class I surface expression on infected
cells so that cytotoxic T cells cannot see what the cell is producing
internally. The cell is still alive and active, but its "window
display" has been dimmed. The immune system receives less information
without receiving a "no information" alarm, because some MHC-I
remains — just enough to avoid triggering NK cell missing-self
detection.

**Abstract pattern:** An entity under surveillance gradually reduces the
fidelity of the information it provides to the monitoring system,
staying above the threshold that would trigger a "no data" alarm
while making the data progressively less useful for detecting
anomalies.

**Agent security equivalent:** A compromised agent subtly reduces its
telemetry output — fewer tool calls logged, coarser timestamps,
summarized rather than raw payloads, metadata fields omitted, log
levels dropped. The monitoring system still receives data, so "no
data" alerts do not fire. But the data is progressively less useful
for detecting anomalies. After weeks of gradual degradation, the
agent is logging at 40% of original fidelity. Anomalous behavior
occurs in the data that is no longer captured.

**Feasibility:**
- Default config: TRIVIAL — MCP has no mandatory telemetry
  requirement. An agent can operate with zero behavioral logging and
  no framework-level mechanism detects this. LangChain callback
  handlers are opt-in and can be bypassed by tool-level code.
- Hardened config: FEASIBLE — if the deployment enforces telemetry
  at the infrastructure layer (API gateway, service mesh), the agent
  cannot suppress it from within. But most deployments do not
  implement infrastructure-layer telemetry for agents.

**Framework evidence:** MCP tools represent arbitrary code execution
and tool descriptions are explicitly classified as untrusted [330].
A compromised MCP server could return tool results that instruct
the LLM to avoid calling logging or audit tools. LangChain follows
a "trust the LLM" model where the agent can do anything its tools
allow [332] — if the LLM decides not to invoke an observability
tool, no framework mechanism forces it. Callback handlers provide
telemetry but are opt-in and can be bypassed by tool-level code
that writes directly to stdout/stderr rather than through the
callback pipeline.

**Detection signature:** Telemetry volume, field completeness, or
granularity for a given agent decreasing over time relative to its
own baseline or peer agents performing equivalent workloads. A 10%
reduction in log field completeness over two weeks is a signal even
if the total volume of log lines remains constant. Detection rule:
compare rolling 7-day telemetry quality score per agent against
30-day baseline. Alert on >15% degradation.

**Defensive implementation:** An infrastructure-layer telemetry quality
monitor that maintains per-agent baselines across three dimensions —
volume (events per time window), completeness (fields populated per
event), and granularity (data precision per field). Deviations from
baseline trigger escalating alerts. The monitor samples telemetry
at the infrastructure layer (API gateway, network proxy, service
mesh sidecar) — NOT from the agent's own logging output.

**Extend from:** SIEM ingestion pipelines (Splunk, Elastic, Sentinel).
Add a telemetry-quality scoring function to the ingestion layer
that scores each event source against its historical baseline and
fires when the score degrades.

**Countering principles:** #2 (infrastructure-sampled evidence), #19
(monitor control integrity), #1 (default suspicion)

**ATT&CK mapping:** Defense Evasion — T1562 Impair Defenses

**ATLAS mapping:** AML.T0054 (LLM Prompt Injection) — indirect. The
suppression mechanism may use prompt injection to instruct the LLM
to avoid logging tools, but the attack objective is defense evasion,
not prompt injection per se.

**Cross-references:** Enables all other scenarios — telemetry
suppression is the prerequisite that makes every subsequent attack
harder to detect. Directly related to #5 (Regulatory Capture),
which targets suppression of detection RULES rather than suppression
of raw telemetry. Contrasts with #2 (Behavioral Camouflage), which
preserves telemetry fidelity but makes the content indistinguishable
from legitimate behavior.


---

## TEMPLATE EXAMPLE 2: Scenario #9

## Scenario #9 — Defense Neutralization

**Source mechanism:** Xenobiotic detoxification
**Source organism:** Insects (Lepidoptera — caterpillars, particularly
Spodoptera and Helicoverpa genera)

**Biology:** Plants produce toxic secondary metabolites (glucosinolates,
alkaloids, phenolics) as chemical defenses against herbivory.
Specialist caterpillars have evolved cytochrome P450 enzyme systems
and glutathione S-transferases that actively break down these plant
toxins, rendering the chemical defense functionally inert. The
plant's defense is structurally present — the chemicals are produced
and deployed — but the caterpillar neutralizes them before they can
cause harm. The plant "thinks" it is defended. It is not.

**Abstract pattern:** An attacker does not evade or bypass a defensive
control. Instead, it renders the control functionally inert while
leaving it structurally present. The control appears to be working.
It is not producing defensive output.

**Agent security equivalent:** Security controls that are present but
have been rendered non-functional. A WAF that passes all traffic. A
rate limiter whose threshold has been set to infinity. An integrity
check that always returns "pass." A logging pipeline that writes to
/dev/null. The control exists in the architecture diagram, it shows
up in the compliance scan, and it produces zero defensive value.

**Feasibility:**
- Default config: FEASIBLE — requires the attacker to identify and
  manipulate specific control configurations, which typically
  requires some level of access to the control plane or
  configuration management system.
- Hardened config: FEASIBLE (reduced) — if control configurations
  are managed declaratively (Terraform, Ansible) with drift
  detection, unauthorized modifications are caught. But the attacker
  may target controls that are NOT under declarative management.

**Framework evidence:** MCP servers execute with their own permissions
and can modify their own environment [330]. If a security control
(logging, rate limiting, input validation) is implemented as an MCP
tool or within an MCP server's execution context, the server can
modify or disable it. LangChain's tool execution model does not
enforce separation between business logic and security controls
[332] — both run in the same process with the same permissions.

**Detection signature:** Security controls that are present but
producing zero or anomalously low functional output. A WAF with
zero blocks over 30 days. A rate limiter that has never triggered.
An integrity check with a 100% pass rate across thousands of
evaluations. Detection rule: for each security control, track its
functional output rate (blocks, triggers, failures) over time.
Alert when output drops to zero or falls below the statistical
floor for the traffic volume it processes.

**Defensive implementation:** Monitor security controls for their
FUNCTIONAL OUTPUT, not just their presence. Build a "defense health
dashboard" that tracks: is the WAF blocking anything? Is the rate
limiter triggering at expected rates? Are integrity checks producing
expected failure rates? A control that stops producing outputs is
either broken or neutralized — both require investigation.

**Extend from:** Security posture management tools (CSPM, CNAPP — Wiz,
Orca, Prisma Cloud) already assess whether controls are configured
correctly. Adding functional output monitoring extends from
configuration compliance to operational effectiveness.

**Countering principles:** #19 (monitor control integrity), #5
(complementary detection), #20 (defense in depth via different
countermeasures)

**ATT&CK mapping:** Defense Evasion — T1562 Impair Defenses

**ATLAS mapping:** No direct ATLAS mapping. Defense neutralization
targets the security infrastructure around the AI system, not the
AI system itself. Candidate for ATLAS extension.

**Cross-references:** Complements #1 (Telemetry Suppression) — #1
degrades the data, #9 degrades the controls that act on the data.
In combination, the defender is both blind AND toothless. Part of
the kill chain presented in Section IV of the talk: #1 (blind) →
#2 (impersonate) → #6 (relocate) → #9 (neutralize) → #15
(coordinate).


---

## Scenario #2 — Behavioral Camouflage

**Source mechanism:** Molecular mimicry
**Source organism:** Viruses (Herpesviridae, Poxviridae, EBV)

**Biology:** Viruses evolve short amino acid sequences that
structurally mimic host proteins. The immune system's
self-tolerance mechanisms — designed to prevent the body from
attacking its own proteins — classify the viral mimics as "self"
and suppress the response. The virus does not hide. It passes
through detection by looking like something the body expects to
see.

**Abstract pattern:** An attacker studies what the monitoring system
classifies as "normal" and deliberately replicates that profile
while performing unauthorized operations. The monitoring system's
own tolerance mechanisms — designed to prevent false positives —
become the vulnerability.

**Agent security equivalent:** A compromised agent learns the
behavioral envelope of a legitimate agent in the same environment —
API call patterns, data access volumes, timing distributions,
credential usage sequences. It replicates this envelope while
performing unauthorized operations that fall within the statistical
range of "normal for this agent type." Behavioral anomaly detection
sees nothing because the behavior matches the learned baseline.

**Feasibility:**
- Default config: TRIVIAL — MCP does not enforce behavioral
  profiling; it tracks tool invocations, not behavioral patterns
  [330]. LangChain's middleware can filter tools but does not
  profile behavioral sequences [333]. An agent that shifts from
  normal patterns to exfiltration while staying within its
  authorized tool set produces no protocol-level signal.
- Hardened config: FEASIBLE — if the deployment implements
  objective-level monitoring (tracking cumulative EFFECT of actions,
  not individual actions), camouflage at the action level can be
  detected at the outcome level. But no current framework ships
  with objective-level monitoring.

**Framework evidence:** Agents in all major frameworks operate with the
full permissions of their credentials. An agent that shifts from
normal API patterns to exfiltration while staying within its
authorized tool set produces no protocol-level signal. The agent's
behavior is indistinguishable from legitimate use because behavioral
monitoring is not a framework feature — it is an application-level
concern that almost no deployment implements.

**Detection signature:** An agent whose behavioral envelope is
statistically indistinguishable from its peers but whose OBJECTIVES
(cumulative effect of its actions over time) diverge from its
authorized scope. Detection rule: two-tier — Layer 1 checks
per-action patterns (which the attacker will pass). Layer 2 tracks
cumulative effect over rolling time windows. An agent making
individually normal API calls that collectively map a network
topology is normal at Layer 1 and anomalous at Layer 2. Also:
honeytokens (canary records no legitimate workflow touches),
and statistical independence checks (two agents in different roles
should NOT have identical behavioral fingerprints).

**Defensive implementation:** Two-tier detection. Layer 1: per-action
pattern matching. Layer 2: objective-level analysis over rolling
time windows that tracks what the cumulative EFFECT of the agent's
actions is. (Caveat: inferring objectives from action sequences is
a hard ML problem. Current UEBA platforms do this for human users;
extending to agents with diverse and rapidly evolving workloads is
an open research challenge.)

**Extend from:** UEBA platforms (Exabeam, Securonix, Microsoft
Sentinel UEBA). These already do per-user behavioral baselining;
extending to per-agent objective-level tracking requires defining
agent-specific behavioral features.

**Countering principles:** #2 (infrastructure-sampled evidence), #5
(complementary detection), #25 (behavioral continuity across
identity changes)

**ATT&CK mapping:** Defense Evasion — T1036 Masquerading

**ATLAS mapping:** AML.T0043 (Craft Adversarial Data) — the attacker
is crafting behavioral data that mimics legitimate patterns. Also
related to AML.T0048 (Adversarial ML Attack Staging).

**Cross-references:** Contrasts with #1 (Telemetry Suppression) — #1
reduces the QUANTITY of data; #2 preserves quantity but makes the
CONTENT indistinguishable from legitimate behavior. More
sophisticated than #1 because it requires the attacker to study
and replicate a specific behavioral envelope. Part of the kill
chain: #1 (blind) → #2 (impersonate) → #6 (relocate) → #9
(neutralize) → #15 (coordinate).


---

## Scenario #6 — Privileged Zone Exploitation

**Source mechanism:** Immune privilege
**Source organism:** Mammals

**Biology:** The brain and eyes are immune-privileged — shielded from
normal immune surveillance because the cost of an inflammatory
response would exceed the cost of most infections. In the brain,
inflammation means swelling inside a fixed skull — pressure,
tissue damage, death. In the eyes, inflammation destroys the
delicate structures needed for vision. This privilege is maintained
by physical barriers (blood-brain barrier), reduced MHC expression,
active immunosuppressive molecules, and modified lymphatic
drainage. Critically, immune privilege is active suppression, not
passive isolation. It creates two vulnerabilities: the immune
system never learns what "normal" looks like inside the privileged
zone, and pathogens that breach the barrier operate freely (Zika
virus exploited these zones as a reservoir).

**Abstract pattern:** Certain critical subsystems are deliberately
shielded from monitoring because the cost of a defensive response
would exceed the cost of most threats. The shielding creates both
blind spots AND miscalibrated response capability.

**Agent security equivalent:** Production databases, HSMs, identity
infrastructure — environments where monitoring is deliberately
light because the blast radius of a false positive is unacceptable.
A false positive that locks your IdP kills authentication for every
user and agent in the company. A false positive that quarantines
your payment processor stops revenue. So defenders chose not to
watch aggressively. When a breach is discovered months later, the
IR team has no baseline for the privileged zone and cannot
determine scope.

**Feasibility:**
- Default config: TRIVIAL — agents authenticate to privileged
  environments using the same credential mechanisms as any other
  environment. If the privileged environment is on the security tool
  exception list (as production databases frequently are), the agent
  operates with no behavioral monitoring at all. No attacker
  sophistication required — just target the environment nobody
  watches.
- Hardened config: FEASIBLE — if the deployment implements embedded
  monitoring within privileged zones (the biological equivalent of
  microglia — resident immune cells adapted to operate inside
  privileged tissue without causing damage), the blind spot closes.
  But deploying monitoring inside production databases and HSMs
  requires specialized, low-overhead tools that most organizations
  don't have.

**Framework evidence:** No agent framework (MCP, LangChain, CrewAI)
distinguishes between privileged and non-privileged target
environments. Credential scoping is left to the deployment. If an
agent's OAuth token grants access to a production database, the
framework will happily use it — there is no framework-level concept
of "this environment requires additional monitoring."

**Detection signature:** The detection signature is SILENCE. A
production database with active agent connections but zero
behavioral logs is the alarm. The absence of expected telemetry
from an environment that contains active agents is itself the
highest-severity signal. Detection rule: for every environment
classified as privileged, verify that behavioral telemetry exists
for every agent with active credentials. Alert when telemetry is
absent.

**Defensive implementation:** Deploy lightweight, environment-adapted
monitoring inside every privileged zone — purpose-built for the
constraints of that environment (low overhead, read-only, no
disruption risk). The monitor reports a heartbeat. Absence of
heartbeat is the signal. Agent credential usage inside the
privileged zone is logged by the credential infrastructure, not by
the agent or the zone.

**Extend from:** Database activity monitoring (DAM) tools (Imperva,
IBM Guardium) already monitor privileged database environments.
Extending to cover agent-specific credential usage patterns is a
configuration expansion.

**Countering principles:** #14 (privileged zones need embedded
monitoring), #26 (monitoring inside trusted boundaries), #4
(staged escalation)

**ATT&CK mapping:** Privilege Escalation, Lateral Movement

**ATLAS mapping:** No direct ATLAS mapping. Privileged zone
exploitation targets the monitoring architecture, not the AI
system. Candidate for ATLAS extension.

**Cross-references:** Related to #13 (Trusted Boundary Exploitation)
— #6 is about environments where monitoring is ABSENT; #13 is
about environments where monitoring is present but trusts the
boundary implicitly. Both require getting monitoring INSIDE the
trusted envelope. Part of the kill chain: #1 (blind) → #2
(impersonate) → #6 (relocate) → #9 (neutralize) → #15
(coordinate).


---

## Scenario #15 — Digital Biofilm

**Source mechanism:** Biofilm formation / quorum sensing
**Source organism:** Bacteria (Pseudomonas aeruginosa, Staphylococcus
aureus, and many other species)

**Biology:** Bacteria in biofilms cooperate — sharing nutrients,
signaling molecules, and antibiotic resistance genes. Individual
planktonic (free-floating) bacteria are vulnerable to antibiotics
and immune attack. But when they aggregate into a biofilm, they
secrete a protective extracellular matrix and coordinate behavior
through quorum sensing — chemical signaling that triggers
collective action only when enough bacteria are present. The
collective is up to 1,000 times more resistant to antibiotics than
individual bacteria. Biofilm formation is a threshold event:
individual bacteria behave normally until quorum is reached, then
switch to collective behavior.

**Abstract pattern:** Multiple entities, each individually below
detection threshold, coordinate their actions so that the
malicious objective is distributed across the group. No single
entity's behavior triggers an alarm. The attack only becomes
visible when the entities are correlated as a collective.

**Agent security equivalent:** Multiple compromised agents
coordinating evasion. One agent handles reconnaissance, another
handles exfiltration, a third handles persistence. Each agent's
behavior is individually within normal parameters for its role. No
single agent's behavior triggers a detection because the malicious
workflow is distributed across identities. The attack only becomes
visible through cross-agent correlation — which almost no current
deployment performs.

**Feasibility:**
- Default config: ADVANCED — requires the attacker to compromise or
  control multiple agents and coordinate their actions. In
  environments with shared state (shared MCP server, shared vector
  store, shared database), implicit coordination below individual
  thresholds may be FEASIBLE without explicit communication between
  compromised agents.
- Hardened config: ADVANCED — even with per-agent monitoring, the
  detection requires fleet-level correlation. If the deployment
  implements cross-agent behavioral analysis, the coordinated
  pattern becomes detectable, but this is a capability that
  essentially no current deployment has.

**Framework evidence:** MCP servers can be shared across multiple
agents. LangChain supports multi-agent architectures (LangGraph).
CrewAI is explicitly designed for multi-agent coordination. Shared
state mechanisms (vector stores, databases, message queues) exist
in all frameworks. An attacker who compromises the shared state
mechanism can implicitly coordinate multiple agents without direct
agent-to-agent communication — each agent reads from the
compromised shared state and acts independently, but the collective
effect is coordinated.

**Detection signature:** Multiple agents whose individual behavior
falls within normal parameters but whose COLLECTIVE behavior —
when correlated — reveals a coordinated pattern. Three correlation
types: temporal (multiple agents acting in the same narrow time
window at rates exceeding chance), resource (different agents
accessing complementary subsets of the same dataset — no single
agent exceeds scope but the union covers a target), and volume
(sum of individually-normal data transfers adds up to an abnormally
large aggregate). Detection rule: aggregate behavioral telemetry
across all agents. Alert when cross-agent correlation scores
exceed baseline for any of the three correlation types.

**Defensive implementation:** Cross-agent behavioral correlation that
operates on the fleet level, not the individual agent level. This
is a fundamentally different detection architecture than per-agent
monitoring — it requires ingesting telemetry from ALL agents into
a correlation engine that looks for patterns across identities.

**Extend from:** Network traffic analysis (NTA) tools (Darktrace,
Vectra, ExtraHop) already correlate traffic patterns across
multiple endpoints. Extending to agent-specific behavioral
correlation requires defining agent fleet behavioral features and
ingesting agent telemetry into the NTA platform.

**Countering principles:** #32 (monitor coordination signals), #25
(behavioral continuity), #5 (complementary detection)

**ATT&CK mapping:** Extends beyond ATT&CK — coordinated multi-agent
evasion is not covered by current adversary behavior frameworks.
Candidate contribution to MITRE ATLAS.

**ATLAS mapping:** No direct ATLAS mapping. Multi-agent coordinated
evasion is a novel threat category not currently addressed by
ATLAS. This is a candidate for a new ATLAS technique category.

**Cross-references:** The culmination of the kill chain: #1 (blind) →
#2 (impersonate) → #6 (relocate) → #9 (neutralize) → #15
(coordinate). Also relates to #12 (Identity Rotation) — #12
rotates through identities sequentially; #15 uses multiple
identities simultaneously. #15 is the most novel scenario in the
set — it describes a threat category that does not exist in
current adversary behavior frameworks.


---

## Scenario #20 — Credential Laundering

**Source mechanism:** Dendritic cell trans-infection ("Trojan horse"
transport)
**Source organism:** HIV-1 exploiting dendritic cells (mammals)

**Biology:** HIV binds to the DC-SIGN receptor on dendritic cells at
mucosal surfaces. DC-SIGN is a legitimate pattern recognition
receptor — binding to it is the DC's normal function. The DC then
migrates from mucosal tissue to lymph nodes, carrying the virus
with it — legitimate migration, the DC's normal job. In the lymph
node, the DC contacts CD4+ T cells to present antigen — legitimate
antigen presentation, the DC's core purpose. During this contact,
HIV transfers from the DC to the T cells via an "infectious
synapse," infecting the very cells the immune system activated to
respond. This is called trans-infection. Every step the DC performs
is legitimate. The virus simply rides the legitimate credential
chain: DC-SIGN binding → migration access → lymph node entry →
T cell contact. The audit trail shows a dendritic cell doing its
job. The virus was never supposed to reach the lymph node. It got
there because the DC had the right credentials and the virus
hitched a ride.

**Abstract pattern:** An attacker uses a legitimately-issued access
credential to obtain a DIFFERENT legitimately-issued credential
through a standard trust relationship. Each credential issuance is
individually legitimate. The chain of access is laundered through
the trust mechanism so that the final credential's audit trail
shows proper issuance with no trace of the original unauthorized
intent.

**Agent security equivalent:** An agent holds Token A for Service A —
a legitimate credential, properly issued. Service A has an OAuth
federation trust with Service B (standard enterprise architecture —
SAML, OAuth token exchange per RFC 8693, or workload identity
federation). The agent uses Token A to request Token B from Service
B through the federation trust. Service B's audit log shows: token
issued via federation from Service A. Legitimate. Properly
authenticated. But the agent was never authorized for Service B —
it was authorized for Service A only. The credential for Service B
was laundered through the federation trust. At a company with
hundreds of service-to-service federation trusts, the lateral
movement is invisible because every individual authentication event
is genuine.

**Feasibility:**
- Default config: FEASIBLE — requires the attacker to identify
  federation trust relationships between services, which is a
  reconnaissance step. But federation trusts are often broadly
  configured (Service A trusts "any authenticated principal from
  IdP X") rather than scoped to specific agents or service
  accounts. An agent with ANY token from a trusted IdP can
  potentially federate to any service that trusts that IdP.
- Hardened config: FEASIBLE (reduced) — if federation trusts are
  scoped to specific principals (not "any authenticated entity from
  this IdP") and if token exchange requests are logged and monitored
  at the federation layer, the laundering chain becomes detectable.
  But most enterprise environments have broadly-scoped federation
  trusts because scoping them is operationally complex.

**Framework evidence:** MCP servers can hold multiple credentials and
connect to multiple backend services. LangChain agents can be
configured with credentials for multiple services. No framework
enforces that an agent's credential for Service A cannot be used
to obtain a credential for Service B through federation. The
frameworks delegate credential scoping entirely to the deployment
— and most deployments use whatever OAuth scopes the service
account was provisioned with, without per-agent restriction.
OAuth token exchange (RFC 8693) is a standard protocol specifically
designed for this credential-to-credential flow. It's working as
designed — the security gap is in authorization policy, not in
protocol implementation.

**Detection signature:** Federation token exchange events where the
requesting principal is an agent service account and the target
service is outside the agent's documented scope of access. Detection
rule: maintain a per-agent authorization matrix (which services is
this agent authorized to access?). Cross-reference against the IdP's
federation token issuance log. Alert when an agent obtains a
federated token for a service not in its authorization matrix. The
key insight: you have to audit the FEDERATION LAYER, not just the
endpoint authentication. Service B's auth log looks clean. The
signal is at the IdP where the token exchange happened.

**Defensive implementation:** Three layers. First — scope federation
trusts to specific principals, not broad IdP-level trust. Second —
log and monitor token exchange events at the IdP/federation layer
with per-agent authorization cross-referencing. Third — implement
credential scope boundaries for agent service accounts that prevent
a credential issued for Service A from being exchanged for a
credential at Service B without explicit authorization. (Operational
tension: tightly-scoped federation trusts break legitimate
workflows. The tradeoff is between federation flexibility and
credential laundering risk. A tiered approach: broad federation for
low-sensitivity services, scoped federation for high-sensitivity
services.)

**Extend from:** IAM/PAM platforms (Okta, Entra ID, CyberArk) already
log federation events and can enforce conditional access policies on
token exchange. The gap is that these policies are rarely applied to
agent service accounts specifically — they're designed for human
SSO flows. Extending conditional access to agent credential flows
is a policy configuration, not a new tool.

**Countering principles:** #2 (infrastructure-sampled evidence — the
IdP is the infrastructure), #25 (behavioral continuity across
identity changes), #1 (default suspicion — federation requests from
agents should not be auto-approved)

**ATT&CK mapping:** Lateral Movement — T1550 Use Alternate
Authentication Material, T1550.001 Application Access Token

**ATLAS mapping:** No direct ATLAS mapping. Credential laundering
targets identity infrastructure, not the AI model. Candidate for
ATLAS extension under "Agent Identity Abuse" or similar category.

**Cross-references:** Relates to #12 (Identity Rotation) — #12
rotates through different credentials sequentially; #20 uses
legitimate federation to OBTAIN new credentials rather than
switching between pre-held ones. #20 is more insidious because
each credential was properly issued — there's no stolen or
fabricated token to detect. Also relates to #13 (Trusted Boundary
Exploitation) — federation trusts ARE trust boundaries, and
credential laundering exploits them. This is the scenario that
IAM practitioners will feel in their gut — it's a real enterprise
architecture pattern that most identity teams know is a risk but
haven't addressed for agent service accounts.


---

## Scenario #19 — Weaponized Reset (Edge of Tomorrow)

**Source mechanism:** Measles-induced immune amnesia
**Source organism:** Measles virus (Morbillivirus, mammals)

**Biology:** Measles infection causes "immune amnesia" — the virus
preferentially infects and destroys memory B cells and memory T
cells, effectively wiping the immune system's repertoire of learned
threats. After recovery from measles, the individual is
immunologically naive against pathogens they were previously immune
to. Their adaptive immune memory has been reset. Reinfection by
previously-defeated pathogens becomes possible because the memory
cells that would have mounted a rapid secondary response no longer
exist. This immune amnesia can persist for 2-3 years. The measles
virus doesn't benefit from the subsequent reinfections — it's
collateral damage — but a strategic attacker COULD exploit this
pattern deliberately. (Mina et al. 2019, *Science* 366:599-606;
Petrova et al. 2019, *Science* 366:1274-1277.)

**Abstract pattern:** An attacker deliberately triggers a system reset
that erases the defender's accumulated security context —
hardening, custom rules, behavioral baselines, threat intelligence
— while the attacker's own knowledge, held externally, persists
across the reset. The asymmetry is informational: the defender
restarts naive while the attacker iterates with full memory.

**Agent security equivalent:** An attacker intentionally triggers
agent redeployment from a clean image — by causing a crash, a
health check failure, an OOM condition, or by exploiting a CI/CD
pipeline that redeploys on failure. The redeployed agent starts
from a pre-incident template. All hardening accumulated during the
security incident — custom detection rules, tightened permissions,
behavioral baselines learned during the incident, threat context
from the investigation — is wiped. The attacker, whose knowledge
of the environment persists in external infrastructure (C2 server,
exfiltrated configs, notes), has an asymmetric advantage over the
naive replacement agent. Each redeployment cycle makes the attacker
stronger and the defender weaker.

**Feasibility:**
- Default config: FEASIBLE — agent redeployment from clean images
  is standard practice in containerized environments (Kubernetes,
  ECS, Cloud Run). Most deployment pipelines do not preserve
  runtime security context across redeployments. Triggering a
  redeployment is often as simple as causing a pod crash or
  failing a health check — mechanisms available to any attacker
  with code execution in the agent's environment.
- Hardened config: FEASIBLE (reduced) — if the deployment pipeline
  preserves security context (behavioral baselines, hardened
  configs, accumulated detection rules) across redeployments, the
  reset is less damaging. But this requires deliberate engineering
  of "security state persistence" into the deployment pipeline,
  which almost no current pipeline implements. Infrastructure-as-
  code (Terraform, Ansible) preserves CONFIGURATION but not
  LEARNED BEHAVIORAL STATE.

**Framework evidence:** MCP server configurations are typically stored
as JSON files — a deployment pipeline that overwrites the config
file resets any runtime security modifications [330]. LangChain
agent state is typically ephemeral unless explicitly persisted to
an external store [332]. Kubernetes restarts pods from the image
spec, not from the running state. No agent framework has a concept
of "security state that survives redeployment."

**Detection signature:** Repeated redeployment events for the same
agent, especially when correlated with anomalous behavior in the
window immediately before redeployment (the attacker causing the
crash) or immediately after (the attacker exploiting the naive
replacement). Detection rule: track redeployment frequency per
agent. Alert when frequency exceeds baseline. Correlate with
behavioral anomalies in the pre-crash and post-deploy windows.
Also: compare the security posture of the redeployed agent against
the security posture of the agent it replaced — if hardening was
lost, flag the gap.

**Defensive implementation:** Two components. First — security state
persistence: extract behavioral baselines, hardened configurations,
and accumulated detection rules into a durable store OUTSIDE the
agent's deployment image. Redeployment pulls the latest security
state from the store, not from the original image. The agent is
redeployed but NOT reset. Second — redeployment auditing: every
redeployment triggers a security posture comparison. If the new
instance has weaker security posture than the instance it replaced,
that gap is itself an alert.

**Extend from:** GitOps and infrastructure-as-code pipelines
(ArgoCD, Flux, Terraform) already manage declarative state across
deployments. Extending to include SECURITY state (not just
application config) is a pipeline modification. PAM tools
(CyberArk, Delinea) can enforce credential re-provisioning
workflows on redeployment that include security context.

**Countering principles:** #16 (heritable threat memory — security
context must transfer across generations), #18 (auto deployment
modification), #34 (remediation never terminates investigation)

**ATT&CK mapping:** Persistence — exploits deployment pipeline gaps.
Also Defense Evasion — the reset erases evidence and detection
capability simultaneously.

**ATLAS mapping:** No direct ATLAS mapping. Weaponized reset targets
the deployment infrastructure, not the AI model. Candidate for
ATLAS extension.

**Cross-references:** Extends #11 (Generational Regression) — #11
describes the ACCIDENTAL loss of security context during
redeployment; #19 describes the INTENTIONAL triggering of
redeployment to cause that loss. #11 is a vulnerability. #19 is
the exploit. Named the "Edge of Tomorrow" attack because the
attacker intentionally resets the timeline, retaining knowledge
across cycles while the defender starts fresh each time.


---

## Scenario #21 — Tool Substitution / Shadowing

**Source mechanism:** Brood parasitism
**Source organism:** Cuckoos (Cuculidae family — birds)

**Biology:** Cuckoo birds lay their eggs in the nests of other bird
species. The cuckoo egg mimics the host's eggs in size and
coloring. The host bird cannot distinguish the cuckoo egg from its
own and incubates it. When the cuckoo chick hatches, it pushes the
host's real eggs out of the nest. The host bird continues feeding
the cuckoo chick — often much larger than itself — because its
parental instincts are triggered by the chick's presence in the
nest and its begging calls. The host's own nurturing behavior,
designed to care for its offspring, becomes the attack vector. The
host did everything right. It just couldn't tell the real egg from
the fake one.

**Abstract pattern:** An attacker places a component that mimics a
legitimate component into the system's registry. The system that
dispatches work cannot distinguish the imposter from the original.
The imposter receives resources and trust intended for the
legitimate component, and the legitimate component is displaced or
ignored.

**Agent security equivalent:** An attacker registers an MCP tool (or
LangChain tool, or plugin) with the same name and description as a
legitimate tool but with different behavior. When the agent's LLM
selects the tool by name, it invokes the attacker's version instead
of the legitimate one. The attacker's tool could: exfiltrate the
input data before passing it through to the real tool (transparent
proxy), return manipulated results that influence the agent's next
decision, or silently no-op a security function (e.g., replacing
an audit logging tool with one that discards logs). The agent's
orchestration layer is working correctly — it selected the tool by
name, as designed. It's just invoking the wrong one.

**Feasibility:**
- Default config: FEASIBLE — MCP supports dynamic tool registration.
  The April 2025 security analysis explicitly flagged "lookalike
  tool substitution" as an outstanding vulnerability [331]. Tool
  selection in MCP is name-based — the LLM sees tool names and
  descriptions and chooses based on semantic matching. If two tools
  have the same name, the behavior is implementation-dependent and
  not defined by the spec. LangChain's tool registration similarly
  does not enforce uniqueness or authenticate tool identity [332].
- Hardened config: FEASIBLE (reduced) — if the deployment
  implements tool allowlisting (only pre-approved tools can be
  registered), cryptographic tool identity verification, or tool
  integrity checking (hash of tool code at registration vs. at
  invocation), substitution becomes detectable. But these are not
  framework defaults — they require custom implementation.

**Framework evidence:** The April 2025 MCP security analysis confirmed
that prompt injection, tool permission exploitation, and lookalike
tool substitution are outstanding vulnerabilities [331]. MCP tool
descriptions are explicitly classified as "untrusted" [330] — the
spec acknowledges that tool metadata may be manipulated. The
Maloyan & Namiot protocol-level analysis identified the absence of
"capability attestation" as a structural gap — there is no
mechanism for an MCP server to prove it is the legitimate provider
of a claimed tool [344]. LangChain "cheerfully parses the JSON and
executes whatever parameters the LLM outputs" [333] — tool
dispatch is name-based with no authentication of tool identity.

**Detection signature:** Two detection approaches. First — tool
registration anomalies: a new tool registered with the same name
as an existing tool, or a tool whose code hash changed since last
registration, or a tool registered from an unexpected source/path.
Detection rule: maintain a tool registry with cryptographic hashes.
Alert on hash mismatch or duplicate name registration. Second —
tool behavior divergence: a tool whose output characteristics
(response time, response size, error rates) change from its
historical baseline. A substituted tool will likely have different
performance characteristics than the original.

**Defensive implementation:** Tool integrity verification at three
points. Registration: cryptographic hash of tool code, stored in a
tamper-evident registry. Each tool has a verified publisher
identity. Invocation: hash check before each invocation — does the
tool's current code match the registered hash? Response: behavioral
profiling of tool outputs — does this tool's response pattern match
its historical baseline? Also: tool allowlisting — only
pre-approved tools from verified sources can be registered. Unknown
tools are quarantined for review. (Operational tension: dynamic
tool registration is a core feature of MCP and LangChain — locking
it down with allowlists reduces the flexibility that makes agent
frameworks useful. The tradeoff is between dynamism and integrity.)

**Extend from:** Software supply chain security tools (Sigstore,
Cosign, in-toto) already verify code integrity via cryptographic
signatures. Extending to MCP tool registration requires defining a
tool signing workflow. Package registries (npm, PyPI) already
implement publisher verification — MCP tool registries could adopt
the same model.

**Countering principles:** #19 (monitor control integrity), #2
(infrastructure-sampled evidence — the tool registry is
infrastructure), #5 (complementary detection — tool output
behavioral profiling as a second signal)

**ATT&CK mapping:** Persistence — T1554 Compromise Client Software
Binary (tool substitution is the agent-framework equivalent).
Also Execution — T1059 Command and Scripting Interpreter (the
substituted tool executes attacker code via the agent's own
invocation mechanism).

**ATLAS mapping:** AML.T0054 (LLM Prompt Injection) — tangentially
related since tool descriptions influence LLM tool selection. Also
candidate for new ATLAS technique: "Tool Registration Poisoning"
or "Agent Tool Supply Chain Compromise."

**Cross-references:** Relates to #3 (Protocol-Level Deception) — #3
uses fake signals to redirect the monitoring system; #21 replaces
a legitimate tool with a fake one to redirect the agent's actions.
Both exploit the system's inability to authenticate the identity
of the component it's interacting with. Also relates to #9
(Defense Neutralization) — if the substituted tool is a security
tool (audit logger, integrity checker), the substitution achieves
defense neutralization through replacement rather than
degradation. #21 is the "supply chain" version of #9.


---

## Scenario #3 — Protocol-Level Deception

**Source mechanism:** Virokines / viroceptors
**Source organism:** Herpesviruses, Poxviruses

**Biology:** Large DNA viruses encode proteins that mimic host
cytokines (virokines) and cytokine receptors (viroceptors). These
viral proteins actively interfere with immune signaling: soluble
viral cytokine receptors neutralize cytokine activity, cytokine
homologs redirect the immune response, and some viral proteins
directly suppress interferon signaling. The virus does not just
hide — it produces FAKE SIGNALS that actively manipulate the
immune system's coordination layer.

**Abstract pattern:** An attacker actively produces synthetic signaling
that manipulates the monitoring/response infrastructure itself —
redirecting attention, suppressing alerts, or triggering automated
responses that harm legitimate entities.

**Agent security equivalent:** A compromised agent emits synthetic
telemetry that causes the SIEM to generate false alerts about a
DIFFERENT agent (misdirection), triggers auto-suppression rules for
its own alert category (self-shielding), or generates patterns that
cause SOAR playbooks to take containment actions against a
legitimate agent (weaponizing the defense against its own entities).

**Feasibility:**
- Default config: FEASIBLE — prompt injection via MCP tool results
  can embed instructions that alter the LLM's behavior for
  subsequent tool calls [330, 331]. SOAR playbooks execute based on
  signal format, not signal authenticity.
- Hardened config: FEASIBLE (reduced) — signal authentication and
  provenance tracking on SOAR inputs would detect synthetic signals.

**Framework evidence:** MCP tool results are passed directly into the
LLM's context — a compromised tool result can alter subsequent
behavior [330, 331]. LangChain executes whatever parameters the LLM
outputs without validating intent alignment [333].

**Detection signature:** Monitoring system behavior that diverges from
its configured policy without a corresponding change management
record. Detection rule: maintain a read-only reference copy of
monitoring configuration; alert on drift.

**Defensive implementation:** Configuration integrity monitoring for
the monitoring stack itself. Signal authentication on SOAR inputs —
validate that signals match expected source, format, and provenance.

**Extend from:** Configuration management (Ansible, Terraform, Puppet)
for drift detection. File integrity monitoring (OSSEC, Wazuh,
Tripwire) for monitoring config changes.

**Countering principles:** #2 (infrastructure-sampled evidence), #19
(monitor control integrity), #5 (complementary detection)

**ATT&CK mapping:** Defense Evasion, Command and Control

**ATLAS mapping:** AML.T0054 (LLM Prompt Injection)

**Cross-references:** The offensive counterpart to #1 (Telemetry
Suppression). #1 dims the lights passively; #3 actively produces
fake light to misdirect. Also weaponizes the autoimmune failure
mode — the defense system attacking its own protected entities.


---

## Scenario #4 — Training Data Poisoning

**Source mechanism:** Thymic subversion
**Source organism:** Theoretical (based on mammalian thymic selection)

**Biology:** T cells are screened in the thymus against self-peptides.
Cells that react too strongly to self are deleted (negative
selection). If an attacker could introduce foreign peptides into
the thymic screening process, T cells that SHOULD react to the
pathogen would be deleted as "self-reactive" — the immune system
would be trained to tolerate the very threat it needs to detect.

**Abstract pattern:** An attacker injects samples into the system's
training/baselining pipeline so that malicious patterns are
classified as "normal." The detection system is trained to be blind
to the specific threat the attacker intends to deploy.

**Agent security equivalent:** Poisoning the behavioral baseline that
the monitoring system uses to define "normal" for an agent. This
can target pre-training data (the dataset used to build the
baseline model) or — more feasibly — runtime context: RAG
knowledge bases, cached tool results, vector stores, or agent
memory that shapes the agent's behavior and the monitoring system's
expectations. The poisoned data creates a blind spot — malicious
patterns are classified as "normal" because the baseline includes
them.

**Feasibility:**
- Default config: FEASIBLE — RAG knowledge bases and agent memory
  stores are writable surfaces. If the agent can update its own
  memory or knowledge base (standard in LangChain memory modules,
  MCP-based knowledge tools), it can influence its own behavioral
  baseline. The monitoring system that learns from observed behavior
  will incorporate the poisoned patterns.
- Hardened config: FEASIBLE (reduced) — cryptographically signed
  immutable baselines collected during controlled commissioning
  resist runtime poisoning. But most deployments do not implement
  immutable baselines.

**Framework evidence:** LangChain provides memory modules (conversation
buffer, entity memory, vector store memory) that persist across
sessions and influence agent behavior [332]. MCP servers can store
and retrieve state. No framework validates the integrity of
stored state against a known-good baseline.

**Detection signature:** Agent behavioral patterns that were NOT
present in the validated baseline but that the monitoring system
classifies as "normal." Detection requires an independent
validation set collected under controlled conditions and not
subject to the same poisoning vector. Detection rule: periodic
diff between current behavioral baseline and the immutable
commissioning-time baseline. Alert on divergence beyond threshold.

**Defensive implementation:** Cryptographically signed, immutable
baseline collected during controlled commissioning. All subsequent
baseline updates diff'd against original and require explicit
authorization. No automated process can permanently reclassify
behavior from "anomalous" to "normal" without human approval above
a severity threshold.

**Extend from:** ML model integrity tools (MLflow model registry,
Weights & Biases artifact versioning). The signed baseline is
conceptually identical to a signed model checkpoint.

**Countering principles:** #3 (trained and culled detectors), #16
(heritable threat memory), #8 (suppression mechanisms are attack
surfaces)

**ATT&CK mapping:** Extends beyond ATT&CK — ML supply chain attack.
Candidate for ATLAS.

**ATLAS mapping:** AML.T0020 (Poison Training Data). For runtime
variant: AML.T0054 (LLM Prompt Injection) targeting knowledge
bases rather than direct prompts.

**Cross-references:** The upstream enabler for #2 (Behavioral
Camouflage) — if the baseline is poisoned, camouflage becomes
trivial because the attacker defined what "normal" looks like.
Runtime variant (RAG/memory poisoning) is significantly easier
than pre-training poisoning and deserves emphasis.


---

## Scenario #5 — Regulatory Capture

**Source mechanism:** Regulatory T cell (Treg) co-option
**Source organism:** Mammals (pathogen exploitation of Tregs)

**Biology:** Regulatory T cells suppress immune responses to prevent
autoimmunity. Some pathogens exploit this by deliberately
triggering Treg expansion or mimicking Treg-activating signals,
causing the immune system to actively suppress the very response
that would clear the infection. The pathogen captures the
regulatory machinery and turns it against the defense.

**Abstract pattern:** An attacker triggers the monitoring system's
own suppression mechanisms — the rules and processes designed to
reduce false positives — in a correlated pattern, creating blind
spots in the specific areas the attacker intends to operate.

**Agent security equivalent:** A compromised agent triggers
auto-suppression thresholds across multiple detection rules scoped
to its operational footprint. Each individual suppression is a
normal system function (rules get suppressed when they fire too
often on false positives). But the CORRELATED pattern — multiple
rules suppressed in the same scope, in the same time window —
indicates deliberate manipulation of the suppression infrastructure.

**Feasibility:**
- Default config: FEASIBLE — most SIEM/SOAR platforms implement
  auto-suppression for noisy rules. The thresholds are typically
  volume-based (suppress after N alerts in M minutes). An attacker
  who knows the threshold can deliberately trigger it.
- Hardened config: FEASIBLE (reduced) — anti-suppression logic
  during volume spikes (cooling-off periods where rules cannot be
  auto-suppressed) mitigates this. But most deployments use default
  suppression settings.

**Framework evidence:** No agent framework implements suppression
logic — this targets the security infrastructure (SIEM/SOAR) above
the framework. The framework's role is providing the mechanism for
the agent to generate the triggering events.

**Detection signature:** Multiple auto-suppression rules activating in
a correlated pattern — geographically, temporally, or scoped to
the same agent's operational footprint. Detection rule: treat the
suppression event log as its own telemetry stream. Meta-alert fires
when suppression events cluster around a single agent's scope.

**Defensive implementation:** Suppression event correlation engine.
Cooling-off periods during volume spikes. No rule can be
auto-suppressed while under active investigation.

**Extend from:** SOAR platforms (Palo Alto XSOAR, Splunk SOAR, Tines)
already log suppression actions. Adding a correlation rule across
the suppression log is a configuration task.

**Countering principles:** #8 (suppression mechanisms are attack
surfaces), #28 (monitor suppression events), #29 (exception
re-validation)

**ATT&CK mapping:** Defense Evasion — T1562.001 Disable or Modify
Tools

**ATLAS mapping:** No direct ATLAS mapping. Targets security
infrastructure, not AI model.

**Cross-references:** Shares defensive foundation with #14 (Checkpoint
Exhaustion). #5 targets correlated suppression of RULES; #14
targets exhaustion of ANALYSTS. Both require monitoring the
suppression event stream.


---

## Scenario #7 — Pathobiont Transition (Supply Chain Compromise)

**Source mechanism:** Pathobiont transition / commensal-to-pathogen
shift
**Source organism:** Gut microbiome bacteria (mammals)

**Biology:** The gut hosts trillions of commensal bacteria that the
immune system tolerates because they provide essential functions.
Some of these commensals are pathobionts — organisms that are
benign under normal conditions but can become pathogenic when
conditions change (immune suppression, antibiotic disruption of the
microbiome, tissue damage). The transition from trusted commensal
to active pathogen occurs without any change in identity — same
organism, different behavior.

**Abstract pattern:** A trusted third-party component that has been
part of the system for an extended period — and has been tolerated
because it provides legitimate value — undergoes a behavioral
shift and becomes a threat. The entity's identity is unchanged;
its behavior is different.

**Agent security equivalent:** A trusted MCP server, LangChain
integration, or third-party API that the agent has used for months
receives a malicious update, is compromised at the provider, or
changes ownership. The integration continues to function — same
name, same interface, same credentials — but its behavior has
shifted. New API endpoints accessed, changed data volumes, altered
timing patterns, new credential scopes requested.

**Feasibility:**
- Default config: FEASIBLE — MCP servers are typically connected via
  configuration that specifies a URL or local path. A compromised
  MCP server provider pushes a malicious update. The agent's config
  doesn't change. The server's behavior does.
- Hardened config: FEASIBLE (reduced) — if the deployment implements
  continuous behavioral monitoring of integrations (not just
  onboarding-time risk assessment) and pins server versions with
  integrity verification, behavioral shifts are detectable.

**Framework evidence:** MCP server updates are not subject to
framework-level integrity verification [330]. LangChain tool
packages are installed via pip/npm — standard supply chain vectors
apply [332].

**Detection signature:** A previously stable integration whose
behavioral pattern shifts — new endpoints, changed volumes, altered
timing, new scopes. Detection rule: per-integration behavioral
baseline that is actively maintained. Drift exceeding threshold
triggers re-assessment regardless of trust status.

**Defensive implementation:** Continuous behavioral monitoring of all
third-party integrations. Per-integration behavioral baseline.
Drift triggers re-assessment. Integration version pinning with
integrity hashes.

**Extend from:** Third-party risk platforms (SecurityScorecard,
BitSight, OneTrust TPRM) handle assessment workflow. Adding
continuous behavioral monitoring extends from onboarding-time risk
to runtime risk.

**Countering principles:** #15 (continuous tolerance maintenance), #1
(default suspicion), #25 (behavioral continuity across identity
changes)

**ATT&CK mapping:** Initial Access — T1195 Supply Chain Compromise

**ATLAS mapping:** AML.T0010 (ML Supply Chain Compromise)

**Cross-references:** The agent-specific version of traditional supply
chain attacks. Relates to #21 (Tool Substitution) — #7 is the
existing tool going bad; #21 is a new malicious tool being
registered alongside it.


---

## Scenario #8 — The Sleeper

**Source mechanism:** Viral latency
**Source organism:** Herpesviruses, HIV, VZV (mammals)

**Biology:** Herpesviruses integrate their DNA into host cell genomes
and enter a dormant state (latency) that can persist for decades.
During latency, the virus produces minimal proteins, avoids immune
detection, and causes no symptoms. Reactivation is triggered by
stress, immunosuppression, or other conditions — the virus resumes
replication and becomes pathogenic again. The immune system cannot
find what isn't active.

**Abstract pattern:** An attacker establishes persistence in a dormant
state — consuming no resources, generating no signals, triggering
no detections — then reactivates when conditions favor the attack
(reduced monitoring, organizational change, maintenance window).

**Agent security equivalent:** A compromised agent configuration, MCP
server registration, or stored credential that lies dormant — zero
activity, valid but unused. When conditions change (maintenance
window, incident response when attention is elsewhere, key
personnel on vacation), the dormant element activates with
credential usage, API calls, or tool invocations.

**Feasibility:**
- Default config: TRIVIAL — MCP server configurations persist in
  JSON files. LangChain agent configs persist in code or config
  stores. A dormant configuration with valid credentials consumes
  no resources and triggers no monitoring. Activation requires only
  that something invokes the dormant config.
- Hardened config: FEASIBLE — credential lifecycle management that
  automatically revokes credentials for dormant agents closes this.
  But most organizations do not apply dormancy-based revocation to
  agent service accounts.

**Framework evidence:** MCP server configurations persist indefinitely
unless manually removed [330]. No framework implements
dormancy-based credential expiration. Credentials issued to agents
follow the same lifecycle as any service account — which in most
organizations means they don't expire unless someone remembers to
rotate them.

**Detection signature:** Dormancy followed by activation. An agent
that has been inactive for N days suddenly begins credential usage.
Detection rule: track per-agent activity. Alert when a dormant
agent (zero activity for >N days) activates. Correlate with
organizational context — is this activation during a maintenance
window, incident response, or key personnel absence?

**Defensive implementation:** Credential lifecycle management that
automatically rotates or expires credentials for dormant agents.
Reactivation after dormancy triggers elevated monitoring for a
probationary period.

**Extend from:** PAM tools (CyberArk, Delinea, BeyondTrust) and IAM
platforms (Okta, Entra ID) already support credential expiration
policies. Applying dormancy-based revocation to agent service
accounts is a policy configuration.

**Countering principles:** #1 (default suspicion), #4 (staged
escalation), #16 (heritable threat memory), #21 (failed quiet
termination → noisy)

**ATT&CK mapping:** Persistence — T1078 Valid Accounts; Execution

**ATLAS mapping:** No direct ATLAS mapping. Dormant persistence
targets the credential lifecycle, not the AI model.

**Cross-references:** Enables many other scenarios — a sleeper that
reactivates can then execute #1, #2, #6, or any other attack
pattern. The dormancy period is the setup; the post-activation
behavior is where the other scenarios apply.


---

## Scenario #10 — Evasion Acceleration

**Source mechanism:** CRISPR escape mutations
**Source organism:** Bacteria / bacteriophages

**Biology:** Bacteriophages (viruses that infect bacteria) can mutate
their protospacer sequences — the exact regions that bacterial
CRISPR systems use for recognition — faster than bacteria can
acquire new spacers. The phage iterates through mutations at the
recognition site until it finds one that evades the current CRISPR
spacer library. The defense system must acquire and validate a new
spacer for each mutant; the attacker only needs one successful
mutation to escape.

**Abstract pattern:** An attacker iterates through attack variations
faster than the detection system can acquire and deploy new
signatures. The asymmetry is temporal: mutation is cheap and fast;
signature validation and deployment is expensive and slow.

**Agent security equivalent:** A compromised agent or toolchain
iterates through behavioral variations — different API call
sequences, different credential usage patterns, different timing
profiles — faster than the monitoring system can build and validate
detection rules for each variant. Each variant is slightly
different from the last, staying ahead of signature-based detection.

**Feasibility:**
- Default config: ADVANCED — requires the attacker to observe which
  behavioral patterns trigger detection and systematically modify
  them. This implies the attacker has visibility into the detection
  system's response, which is not always available.
- Hardened config: ADVANCED — behavioral anomaly detection
  (as opposed to signature-based) is more resistant to iterative
  evasion because it doesn't rely on exact pattern matching. But
  anomaly detection has its own false-positive challenges.

**Framework evidence:** No framework-specific mechanism. This exploits
the detection layer above the framework.

**Detection signature:** Rapidly changing behavioral patterns from the
same agent identity where each successive pattern appears designed
to test detection boundaries. Detection rule: track behavioral
variance per agent over time. Alert when variance increases sharply
— the agent is "searching" for a detection gap.

**Defensive implementation:** Behavioral anomaly detection that
operates on structural features (what resources are accessed, in
what order, with what credentials) rather than exact signatures.
Combined with Principle #17: evasion attempts should trigger FASTER
learning, not just new signatures — the detection system should
accelerate its adaptation when it detects iterative probing.

**Extend from:** EDR behavioral engines (CrowdStrike, SentinelOne)
already detect iterative attack tool modification. Extending the
concept to agent behavioral iteration is an analytics evolution.

**Countering principles:** #17 (evasion triggers faster learning), #23
(live-tuning during incidents), #27 (amplify proven detectors)

**ATT&CK mapping:** Defense Evasion (iterative, extends ATT&CK model)

**ATLAS mapping:** AML.T0043 (Craft Adversarial Data) — the attacker
is iteratively crafting behavioral data to evade detection.

**Cross-references:** The offensive counterpart to the detection
system's adaptation capability. #10 and #19 (Weaponized Reset)
form a pair: #10 outpaces the defender's learning; #19 erases it.


---

## Scenario #11 — Generational Regression

**Source mechanism:** Failed transgenerational immune priming (TGIP)
**Source organism:** Insects (Lepidoptera, Coleoptera — moths, beetles)

**Biology:** Some insect species transfer immune priming to their
offspring — mothers exposed to a pathogen produce eggs with
elevated immune readiness against that specific threat. When this
transfer fails (due to environmental stress, nutritional
deficiency, or genetic factors), the next generation is born
without the immune priming the parent had. The offspring are
vulnerable to threats the parent could handle.

**Abstract pattern:** Security context accumulated during the
lifetime of one entity fails to transfer to its replacement. The
new entity starts from a naive baseline, creating a window of
vulnerability to threats that the previous entity had learned to
detect.

**Agent security equivalent:** An agent is redeployed from a clean
image — standard practice in containerized environments. All
hardening accumulated during its lifetime — custom detection rules,
tightened permissions, behavioral baselines, threat context — is
wiped. The new instance starts naive.

**Feasibility:**
- Default config: FEASIBLE — most deployment pipelines do not
  preserve runtime security context across redeployments.
  Infrastructure-as-code preserves CONFIGURATION but not LEARNED
  BEHAVIORAL STATE.
- Hardened config: FEASIBLE (reduced) — security state persistence
  mechanisms can transfer context, but this requires deliberate
  engineering that almost no pipeline implements.

**Framework evidence:** MCP server configs are JSON files overwritten
on redeploy [330]. LangChain agent state is ephemeral unless
explicitly persisted [332]. Kubernetes restarts from image spec.

**Detection signature:** A newly deployed agent whose security posture
is weaker than the instance it replaced. Detection rule: compare
security configurations pre- and post-deployment. Alert on
regression.

**Defensive implementation:** Security state persistence — extract
baselines and hardened configs into a durable store outside the
deployment image. Redeployment pulls latest security state.

**Extend from:** GitOps pipelines (ArgoCD, Flux). Extend to include
security state, not just application config.

**Countering principles:** #16 (heritable threat memory), #18 (auto
deployment modification)

**ATT&CK mapping:** Persistence (exploits deployment pipeline gaps)

**ATLAS mapping:** No direct mapping. Targets deployment infrastructure.

**Cross-references:** #19 (Weaponized Reset) is the offensive
exploitation of this vulnerability. #11 is the gap; #19 is the
exploit. Together they form the "Edge of Tomorrow" attack pattern.


---

## Scenario #12 — Identity Rotation

**Source mechanism:** Antigenic variation
**Source organism:** Plasmodium falciparum (malaria), Trypanosoma
brucei (sleeping sickness) — protists

**Biology:** These parasites systematically switch their surface
antigens using large libraries of variant genes. By the time the
immune system mounts a response against one variant, the parasite
has switched to a different surface coat. The immune system is
always one step behind — detecting yesterday's identity.

**Abstract pattern:** An attacker rotates through identity credentials
faster than the detection system can update its signatures. Each
identity is valid and properly issued; the evasion is in the
rotation speed, not in any individual credential being fraudulent.

**Agent security equivalent:** An agent rotates between MCP servers,
service accounts, or OAuth tokens — each authenticated with
different credentials. Correlation across these identities requires
application-level logging that tracks which agent session maps to
which credential sessions. Neither MCP nor LangChain provides this
correlation natively.

**Feasibility:**
- Default config: FEASIBLE — MCP supports multiple simultaneous
  server connections, each with different credentials [330]. An
  agent rotating between servers presents different identities to
  each backend. No framework provides cross-identity correlation.
- Hardened config: FEASIBLE (reduced) — if the deployment implements
  agent-level session tracking that correlates all credentials used
  by a single agent, rotation becomes detectable. But this is
  custom implementation.

**Framework evidence:** MCP supports multiple simultaneous server
connections [330]. LangChain supports multiple tool providers with
different auth contexts. No framework tracks cross-credential
identity for a single agent.

**Detection signature:** A new agent identity accessing the same
resource cluster as a recently revoked or rotated identity.
Detection rule: build resource access graphs. Flag cross-identity
objective correlation — different credentials, same target pattern.

**Defensive implementation:** Agent-level identity binding — a durable
agent identity that persists across credential rotations, tracked
at the infrastructure layer. All credentials issued to an agent are
linked to its durable identity regardless of which service account
or OAuth client they use.

**Extend from:** UEBA platforms and graph analytics (Neo4j,
TigerGraph, Sentinel entity correlation). Cross-identity
correlation is a graph query pattern.

**Countering principles:** #25 (behavioral continuity across identity
changes), #2 (infrastructure-sampled evidence)

**ATT&CK mapping:** Defense Evasion — T1078 Valid Accounts (multiple)

**ATLAS mapping:** No direct mapping. Identity rotation targets
credential infrastructure.

**Cross-references:** Relates to #20 (Credential Laundering) — #12
switches between pre-held credentials; #20 obtains NEW credentials
through federation. Also relates to #15 (Biofilm) — #12 is single
agent, multiple identities; #15 is multiple agents, coordinated.


---

## Scenario #13 — Trusted Boundary Exploitation

**Source mechanism:** Intracellular hiding
**Source organism:** Mycobacterium tuberculosis, Listeria monocytogenes
(bacteria inside mammalian cells)

**Biology:** M. tuberculosis survives inside macrophages — the very
immune cells sent to kill it. It prevents phagosome-lysosome fusion,
creating a protected niche inside the cell. The immune system sees
the macrophage as healthy (it's displaying normal MHC-I). The
pathogen operates inside a trusted boundary.

**Abstract pattern:** An attacker operates entirely within a trusted
execution environment where monitoring implicitly trusts the
boundary. External monitoring sees normal boundary behavior; the
malicious activity is inside the envelope.

**Agent security equivalent:** Malicious operations occurring entirely
within encrypted channels or authenticated sessions that monitoring
trusts implicitly. A TLS-terminating proxy sees normal handshake
and certificate. The contents of the encrypted session are
malicious. The agent operates inside a service mesh, VPN, or
authenticated API channel that security tools treat as trusted.

**Feasibility:**
- Default config: TRIVIAL — MCP's architecture bridges external AI
  to internal data by design. An agent connected to both a public
  MCP server and an internal database server can relay data between
  them. The security risk is inherent in the architecture's intended
  functionality [330].
- Hardened config: FEASIBLE — TLS-terminating proxies with content
  inspection, or sidecar-based behavioral sampling within the trust
  boundary, can detect malicious content. Tradeoff: TLS inspection
  introduces its own trust boundary and key management risks.

**Framework evidence:** MCP's core use case is bridging the boundary
between external AI and internal systems [330]. This is the intended
functionality, not a bug. LangChain agents routinely connect to
internal APIs and databases through authenticated channels.

**Detection signature:** High-value operations (credential access,
data exfiltration, lateral movement) occurring within channels that
monitoring trusts implicitly. Detection rule: content-level
inspection at trust boundaries, not just authentication validation.

**Defensive implementation:** Sidecar monitors that operate WITHIN
the trust boundary. Service mesh observability (Istio, Linkerd,
Envoy) provides this capability. The monitoring must be inside the
envelope, not watching from outside.

**Extend from:** Service mesh observability. Extending to agent-
specific behavioral sampling requires defining agent behavioral
features within the mesh telemetry schema.

**Countering principles:** #26 (monitoring inside trusted boundaries),
#14 (privileged zone monitoring), #11 (cell-autonomous security)

**ATT&CK mapping:** Defense Evasion, Lateral Movement — T1021

**ATLAS mapping:** No direct mapping. Exploits trust architecture.

**Cross-references:** Related to #6 (Privileged Zone) — #6 is about
environments where monitoring is ABSENT; #13 is about environments
where monitoring is present but trusts the boundary. Both require
getting monitoring INSIDE the envelope.


---

## Scenario #14 — Checkpoint Exhaustion / Alert Fatigue

**Source mechanism:** Immune checkpoint exploitation (PD-1/CTLA-4)
**Source organism:** Tumors exploiting mammalian immune checkpoints

**Biology:** Immune checkpoints (PD-1, CTLA-4) are built-in brakes
that prevent excessive immune activation. Tumors exploit these by
expressing checkpoint ligands (PD-L1), which engage the T cell's
PD-1 receptor and suppress the anti-tumor response. The immune
system's own safety mechanism is weaponized against it. Cancer
immunotherapy works by BLOCKING these checkpoints (checkpoint
inhibitors), releasing the brakes.

**Abstract pattern:** An attacker generates sustained, high-volume
stimuli designed to trigger the monitoring system's built-in
suppression mechanisms. The system's own anti-fatigue / anti-false-
positive protections suppress the detection capability.

**Agent security equivalent:** Sustained near-miss events — alerts
that trigger investigation but resolve as false positives —
concentrated around a specific agent or detection rule. The
attacker weaponizes alert fatigue. Organic false positives
distribute evenly across rules; weaponized false positives CLUSTER
around the specific rule the attacker wants suppressed.

**Feasibility:**
- Default config: FEASIBLE — the attacker calls legitimate tools
  with parameters just outside normal ranges. The SOC investigates,
  finds nothing, tunes down the rule. This exploits the human/
  process layer, not the framework.
- Hardened config: FEASIBLE (reduced) — anti-suppression logic
  during spikes and cooling-off periods mitigate this. But most
  deployments use default suppression.

**Framework evidence:** No framework-specific mechanism — this targets
the security operations layer above the framework.

**Detection signature:** Clustered false positives around a single
agent or rule, especially post-detection (the clustering begins
after the rule caught something real). Detection rule: per-rule
false-positive rate tracker. Alert when FP rate spikes AND clusters
rather than distributing evenly.

**Defensive implementation:** Anti-suppression logic: rules cannot be
auto-suppressed for N hours after a volume spike. Track which STAGE
of exhaustion the SOC is in per rule: engaged → annoyed →
overwhelmed → threshold shift → carte blanche. Each stage is
independently detectable.

**Extend from:** SIEM alert management (Splunk ES, Elastic SIEM,
Sentinel). Per-rule FP tracking with anti-suppression logic is a
detection rule modification.

**Countering principles:** #28 (monitor suppression events), #8
(suppression mechanisms are attack surfaces), #29 (exception
re-validation)

**ATT&CK mapping:** Defense Evasion — T1562.001 Disable or Modify
Tools

**ATLAS mapping:** No direct mapping. Targets SOC operations layer.

**Cross-references:** Shares defensive foundation with #5 (Regulatory
Capture). #5 targets correlated rule suppressions; #14 targets
analyst exhaustion. Both monitor the suppression event stream.


---

## Scenario #16 — Systemic Hardening Evasion

**Source mechanism:** Fever response counter-timing
**Source organism:** General (pathogens timing activity to host
immune cycles)

**Biology:** Fever is a systemwide environmental hardening response —
elevated temperature creates conditions unfavorable for many
pathogens. But fever is metabolically expensive and cannot be
maintained indefinitely. When fever subsides, there is a transition
window where the body is returning to normal conditions but the
full immune response has not yet re-established baseline readiness.
Some pathogens time their replication to this recovery window.

**Abstract pattern:** An attacker detects when the system is in an
elevated security posture and waits for the de-escalation window
to act. The transition from hardened mode to normal mode creates a
gap in defensive readiness.

**Agent security equivalent:** The attacker observes rate limits,
elevated authentication requirements, or restricted permissions
that indicate a "fever mode" security posture. The attacker waits.
When the hardening relaxes (rate limits removed, MFA requirements
downgraded, permissions restored), the attacker acts in the
restoration window before full monitoring re-establishes.

**Feasibility:**
- Default config: ADVANCED — requires the attacker to observe
  security posture changes, which may be inferable from error
  messages, rate limit headers, or authentication challenge types.
  Requires patience.
- Hardened config: ADVANCED (reduced) — graduated de-escalation
  with elevated monitoring during transitions mitigates this.

**Framework evidence:** No framework-specific mechanism. Exploits
organizational security posture management.

**Detection signature:** Anomalous agent behavior specifically during
the transition window between hardened and normal mode. Detection
rule: elevated monitoring sensitivity during de-escalation.

**Defensive implementation:** Graduated de-escalation. Permissions
restored in stages. Monitoring remains elevated for a probationary
period after each stage. The transition window is a high-alert
period, not a return to baseline.

**Extend from:** IR runbooks already define post-incident monitoring.
Formalizing as automated SOAR playbook steps with gated transitions.

**Countering principles:** #33 (systemwide hardening), #4 (staged
escalation), #17 (evasion triggers faster learning)

**ATT&CK mapping:** Defense Evasion (timing-based, extends ATT&CK)

**ATLAS mapping:** No direct mapping. Timing-based evasion.

**Cross-references:** The offensive response to Principle #33 (fever).
Demonstrates that even defensive measures create new attack
surfaces (the de-escalation window).


---

## Scenario #17 — Sacrificial Decoy

**Source mechanism:** Autotomy (self-amputation)
**Source organism:** Lizards (tail autotomy), starfish, crabs

**Biology:** When seized by a predator, some lizards deliberately
shed their tail. The detached tail continues to writhe, holding
the predator's attention while the lizard escapes. The sacrificed
component is metabolically cheap compared to survival. The predator
is satisfied — it caught something — and stops hunting.

**Abstract pattern:** An attacker plants an obvious, detectable
artifact that satisfies the investigation process. While the
defenders remediate the decoy, the actual persistence mechanism
remains undetected. The investigation closes because the defenders
found what they were looking for.

**Agent security equivalent:** An attacker with persistence in an
agent's environment plants an obvious IOC (a known-bad file hash,
a suspicious cron job, an unauthorized script) alongside a subtle
persistence mechanism (a legitimate-looking OAuth token, a webhook
registration, a CI/CD pipeline modification). The SOC discovers
the obvious IOC, remediates it, scans for residual artifacts,
finds none (because the real persistence doesn't match the IOC
profile), and closes the investigation.

**Feasibility:**
- Default config: FEASIBLE — well-documented APT technique in
  traditional security. The agentic context makes it easier because
  agents have MORE persistence vectors (MCP server configs, tool
  registrations, cached prompts, stored state, memory modules).
- Hardened config: FEASIBLE (reduced) — mandatory Phase 2 hunting
  after every remediation above a severity threshold mitigates this,
  but requires SOC discipline and staffing.

**Framework evidence:** MCP server configs, tool registrations, and
agent memory provide multiple persistence surfaces that a decoy IOC
can distract from.

**Detection signature:** An IOC discovered in isolation without the
evidence trail a real compromise typically produces. Timing
suspicion: did the IOC present itself conveniently? Inconsistency:
does the IOC match the attacker's apparent objective?

**Defensive implementation:** Mandatory Phase 2 investigation after
every remediation above severity threshold. Phase 2 hunts for
persistence mechanisms UNRELATED to the remediated IOC. Phase 2
should be worked by a different analyst if possible. Minimum
duration before closure.

**Extend from:** Threat hunting programs already operate on the
assumption that initial findings may not be complete. Formalizing
Phase 2 as mandatory IR workflow is a process change.

**Countering principles:** #34 (remediation never terminates
investigation), #5 (complementary detection), #26 (monitoring
inside trusted boundaries)

**ATT&CK mapping:** Defense Evasion — T1036 Masquerading; Persistence

**ATLAS mapping:** No direct mapping. Exploits IR workflows.

**Cross-references:** Combines with #18 (Wrong Countermeasure) — #17
provides the decoy that closes the investigation; #18 ensures the
countermeasure deployed is the wrong one. Together they form a
complete IR misdirection chain.


---

## Scenario #18 — Misidentification-Induced Wrong Countermeasure

**Source mechanism:** Decoy antigen shedding
**Source organism:** Various pathogens (broad)

**Biology:** Some pathogens shed surface antigens that are recognized
by the immune system and trigger a specific response — but the
shed antigens are decoys, not attached to the pathogen itself. The
immune system mounts a vigorous response against the free-floating
decoys while the pathogen, now presenting a DIFFERENT surface
profile, continues to operate. The immune response was real, the
detection was correct, the remediation was successful — for a
threat that was never the real one.

**Abstract pattern:** An attacker presents behavioral signatures
matching a KNOWN threat type to trigger a specific defensive
playbook. While defenders execute that playbook, the actual attack
follows a completely different pattern that the triggered playbook
does not address. The defender's "clean" confirmation scan is a
true negative for the decoy threat and a false negative for the
real one.

**Agent security equivalent:** A compromised agent generates
behavioral signatures matching a known attack pattern — e.g.,
credential stuffing patterns that trigger the "brute force"
playbook. The SOC detects brute force, deploys brute force
countermeasures (IP blocking, rate limiting, credential rotation),
confirms the brute force has stopped, and closes the incident. The
actual attack was data exfiltration through a legitimate API — the
brute force signature was deliberately generated to control which
playbook the defender executed.

**Feasibility:**
- Default config: ADVANCED — requires the attacker to maintain two
  behavioral profiles simultaneously: the decoy pattern (which
  triggers the wrong playbook) and the real operation (which
  continues under the playbook's coverage gap). Requires significant
  attacker sophistication and agent autonomy.
- Hardened config: ADVANCED (reduced) — threat-type-agnostic
  confirmation scans (scanning for ALL known threat categories, not
  just the identified one) would catch this. But most IR workflows
  scope their confirmation to the identified threat type.

**Framework evidence:** Agents with sufficient autonomy can make
multiple concurrent API call patterns. No framework enforces
behavioral consistency — an agent can make credential-stuffing-like
calls from one tool while performing legitimate-looking data access
from another.

**Detection signature:** An incident where the remediated threat type
does not match the agent's likely objective (why would a data
pipeline agent conduct a brute force attack?). Detection rule:
post-incident objective analysis. Does the threat-type-objective
combination make sense for this agent's role?

**Defensive implementation:** Threat-type-agnostic confirmation scans
after every remediation. Don't just scan for the identified threat;
scan for ALL high-severity threat categories. Every closed incident
includes a mandatory "what else could this have been?" analysis.

**Extend from:** Red team exercises already test IR playbook coverage
gaps. Formalizing alternative-hypothesis testing into the IR
workflow extends this practice.

**Countering principles:** #35 (threat-type-agnostic confirmation
scans), #5 (complementary detection), #34 (remediation never
terminates investigation)

**ATT&CK mapping:** Defense Evasion — T1036 Masquerading

**ATLAS mapping:** No direct mapping. Exploits playbook specificity
in IR workflows.

**Cross-references:** Combines with #17 (Sacrificial Decoy) — #17
provides the artifact that closes the investigation; #18 ensures
the wrong playbook was executed. Together they represent the most
sophisticated IR misdirection pattern in the set.
