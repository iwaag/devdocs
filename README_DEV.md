# Rule

- Don't expose local pc/cluster info in non-ignored files.

# Adopted Deveopment Style

Standard AG Style

# refs
devpolicy/terms.md ... Read only when you need to check terminologies

# In-System Agents

## cagent (pj-clusterintent)

- Responds to requests for explaining/observing/changing desired state and/or actual state of the cluster.
- In-System workflow shold be designed so that cagent receive a report when cagent's explanation of the cluster found invalid.

## autolab agent(pj-agdev/agautolab)

- Responds to request for explaining/observing/developing projects.

## devworld assistent(pj-agdev/agdevworld)

- Responds to any requests from Human, mainly through agdevworld frontend.

## forge agent(pj-agdev/agforge)

- Responds to requests for providing media assets with characteristics specified in the requests.

# Adopted Policies for Development of In-System Agents

## Favorable

Easier Next Time
Single Entrance
Tool Giving
Evidence-Driven Guidance
Failure Farming

## Unfavorable

Anxiety-Driven Guidance
Tool Implantation
Unexplained Chainsaw

## Agent ≠ Model
- The backend model/harness is a swappable parameter of an agent, never its identity.
- Every agentic run records which backend served it. See devpolicy/agent_records.md for the common record.

# Other policies

## Deus Ex Machina note
- When the Omni Agent performs work that belongs to an in-system agent, leave a one-line note in the episode doc: "did X for agent Y — handoff candidate".
- Perhaps positive for the mission, perhaps negative for workflow growth; the note is the whole obligation.

# Ansible Commands

Run from `pj-clusterintent/ansible_agdev`:

- **Update autolab nodes**:
  ```bash
  uv run --project ../nctl nctl render production --out inventories/generated
  ansible-playbook -i inventories/generated/production.yml playbooks/agent/setup_autolab_node.yml --limit agautolab1
  ```
