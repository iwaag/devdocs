# Developer Documentation

## Updating Autolab Nodes (`ansible_agdev`)

The `setup_autolab_node.yml` Ansible playbook deploys and updates `agautolab` code checkout and its HTTP mission gateway service (`autolab-gateway`) on designated job-runner nodes.

### Playbook Details

- **Playbook**: `pj-clusterintent/ansible_agdev/playbooks/agent/setup_autolab_node.yml`
- **Inventory**: `pj-clusterintent/ansible_agdev/inventories/agautolab.yml`
- **Role**: `autolab_node` (`pj-clusterintent/ansible_agdev/roles/autolab_node/`)

### What It Does

1. **Repository Update**: Pulls the latest code for `agautolab` from the local Gitea repo (`http://agstudio.local:3000/autodev/agautolab.git`).
2. **Directory & Binaries**: Ensures directory structure (`.local/agent/sessions`, `serve`, `gateway`) and `claude` binary reference exist.
3. **Gateway Token Management**: Generates a secure gateway bearer token if missing, slurps it, and copies it to the controller machine (`~/.local/state/autolab-gateway/<node>.token`).
4. **Systemd Service**: Deploys/reloads the `autolab-gateway` systemd user service and ensures it is enabled and running.
5. **Health Verification**: Verifies HTTP probe response from `http://127.0.0.1:8791/healthz`.

### Commands

Run from `pj-clusterintent/ansible_agdev`:

- **Update all autolab nodes**:
  ```bash
  ansible-playbook -i inventories/agautolab.yml playbooks/agent/setup_autolab_node.yml
  ```

- **Update a specific target node** (e.g. `agautolab1`):
  ```bash
  ansible-playbook -i inventories/agautolab.yml playbooks/agent/setup_autolab_node.yml --limit agautolab1
  ```
