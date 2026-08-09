# Developer Documentation

### Ansible Commands

Run from `pj-clusterintent/ansible_agdev`:

- **Update autolab nodes**:
  ```bash
  ansible-playbook -i inventories/agautolab.yml playbooks/agent/setup_autolab_node.yml
  ``
