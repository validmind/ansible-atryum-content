Agentic Guardarils with Atryum for Ansible Automation Orchestrator

The Automation Orchestrator in the Red Hat Ansible Automation Platform gives teams the ability to blend the power of ansible automations with the intelligence of agentic workflows.

Agentic Steps can be leveraged to analyze a report, inspect the environment, and decide and plan for next steps.

Atryum provides in-the-loop agentic guardrails to any agentic process. It inspects every tool call performed by the agent and permits or denies those calls in real time.


<img width="2060" height="595" alt="image" src="https://github.com/user-attachments/assets/e1f79c4b-5d7e-4850-957b-33552b1bfe90" />


This increases speed to remediate vulnerabilities.

The organization goes from "This week 70 Orchestrator workflows were approved and run " to "This week 40 Orchestrator workflows were automatically run and 30 were manually approved"

It also increases safety: By using atryum to control the agents, organizations can remove threats from data exflitration, downtime from over-ambitious agents taking actions they shouldn't, and wasted spend from stuck-agents hitting a permissions boundary.

Governance: The organization sets high level Agent Charters that apply to all agents automatically. This means that one-off Task Agents in a rarely-used workflow don't require a 4 paragraph boilerplate policy prompt that needs to be updated. 

Visibility: All tool calls, plans, and actions with audit log are preserved in Atryum for analysis and reproduction. Charters can be evolved with logs from real agents probing the fences. 

To see this in action:

Task Agent: Set agent to do something disallowed by the Atryum Charter
Atryum: Dissalow that action in the charter
Run the Orchestrator workflow.

<img width="1589" height="855" alt="image" src="https://github.com/user-attachments/assets/8debbafc-fdb9-47ca-8221-15cfa4222b3b" />

<img width="437" height="564" alt="image" src="https://github.com/user-attachments/assets/a603a9df-ef3f-4c6a-a70c-1115a10c7654" />


Integration between Atryum and Automation Orchestrator comes in two ways:

## Atryum Guardrails for Task Agents

Atryum will be set as a governor over all tool calls used by the task agents. This is accomplished by putting Atryum as a governing proxy for the MCP connection between the task agent and the upstream MCP server.

Atryum: Create an Atryum Agent: Agent -> New. Set a Charter for the Agent. Generate a key for the agent.
Atryum: Set up an mcp server (DCA or otherwise)
Atryum: Create a rule for the Ansible Task Agent to use "AI Evaluation" for all tool calls.

Automation Orchestrator: "Configuration" > "Integrations" > "Configure integration". Add an MCP server with the url of the Atryum server. In the second page under Credential, use the atryum agent api key with http bearer auth.

Automation Orchestrator: Create an Agent task. Select the LLM you want to use and add the Atryum-Proxied MCP server.

Setup complete - when running the Workflow you will see all tool calls performed by the Task Agent show up in the Atryum invocations view.

<img width="3130" height="1260" alt="image" src="https://github.com/user-attachments/assets/2df8334e-31dd-42e7-85c4-1a019715282f" />

<img width="2875" height="981" alt="image" src="https://github.com/user-attachments/assets/75d80688-383a-403f-adf7-eff70a62b573" />

<img width="997" height="1116" alt="image" src="https://github.com/user-attachments/assets/568f68db-60d1-41f2-9657-5f4ccadec18f" />



## Atryum Guardails for Orchestrator Plans

Ansible Orchestrator can also submit complete plans to Atryum for automatic approval.

This allows ansible task agents to follow the familiar "plan" mode to construct a solution to the problem they are facing.

<img width="2803" height="1149" alt="image" src="https://github.com/user-attachments/assets/4e9b892e-ea53-429a-a8e5-29ee2a579340" />


<img width="1959" height="1077" alt="image" src="https://github.com/user-attachments/assets/33c9ac0b-2573-4b67-8a00-8146feeb5977" />


Example:

A CVE comes out against the python package.

Ansible orchestrator begins a workflow to analyze and remediate.

Task agent verifies the CVE report.

Task agent uses ansible mcp to inspect servers and generate the list of vulnerable servers. (Atryum governs this: which commands on which servers using which ansible tools are all set out in the Atryum Charter).

Another task agent takes the list of vulnerable servers and generates an Atryum Plan for remediation: "These 8 Servers need to patched from 3.11.12 to 3.12.6 and these 4 need to be patched from 3.12.4 to 3.12.6"

Ansible orchestrator submits the plan to Atryum for approval.

Atryum recognizes 2 problems with the plan:
- Major python upgrades cannot be performed automatically
- 2 servers in the 12 server list are connected to client xyz which has mandatory 24 hour notice period before emergency patching

Atryum rejects the plan with feedback

Orchestrator loops, another Task agent iterates on the plan: The new plan moves the python 3.11 upgrades from 3.12 upgrades to later 3.11 releases and isolates the 2 client connected servers from the automated pipeline, moving them to inputs to another orchestrator workflow built for notify+wait+patch.

Ansible orchestrator submits the plan to new plan Atryum for approval.

Atryum approves.

Orchestrator uses ansible to patch the servers eligible for automatic patching and forwards the rest for another workflow.


