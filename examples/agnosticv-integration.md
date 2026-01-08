# AgnosticV Integration Guide

This guide shows how to integrate the `maas_librechat.maas_librechat` collection into AgnosticV.

## Collection Installation

### Option 1: Install from Galaxy (When Published)

Add to your `requirements.yml`:

```yaml
---
collections:
  - name: maas_librechat.maas_librechat
    version: ">=1.0.0"
```

### Option 2: Install from Git (Development)

Add to your `requirements.yml`:

```yaml
---
collections:
  - name: https://github.com/rhpds/maas-librechat.git
    type: git
    version: main
```

### Option 3: Local Development

For testing locally:

```bash
# Build the collection
cd /Users/psrivast/work/code/maas-librechat
ansible-galaxy collection build

# Install it
ansible-galaxy collection install maas_librechat-maas_librechat-1.0.0.tar.gz -f
```

## AgnosticV Configuration

### Directory Structure

In your AgnosticV environment definition:

```
my-environment/
├── common.yaml
├── dev.yaml
├── prod.yaml
└── requirements.yml
```

### requirements.yml

```yaml
---
collections:
  - name: maas_librechat.maas_librechat
    version: ">=1.0.0"

  # Other required collections
  - name: kubernetes.core
    version: ">=2.3.0"
```

### common.yaml

Add the workload using FQCN:

```yaml
---
# Basic configuration
workloads:
  - maas_librechat.maas_librechat.ocp4_workload_librechat_mcp

# Workload variables with FQCN prefix
ocp4_workload_librechat_mcp_namespace: librechat
ocp4_workload_librechat_mcp_admin_username: instructor
ocp4_workload_librechat_mcp_admin_password: openshift

# Optional: Configure LiteMaaS
ocp4_workload_librechat_mcp_litemaas_url: https://litellm.apps.maas.redhatworkshops.io/v1
ocp4_workload_librechat_mcp_litemaas_models:
  - llama-scout-17b
  - granite-3-2-8b-instruct

# Optional: Enable/disable features
ocp4_workload_librechat_mcp_enable_openshift_mcp: true
ocp4_workload_librechat_mcp_create_scc: true
```

### Multi-User Environment Example

For workshop with multiple users:

```yaml
---
# common.yaml
workloads:
  - maas_librechat.maas_librechat.ocp4_workload_librechat_mcp

# Base configuration
ocp4_workload_librechat_mcp_namespace: librechat-user1
ocp4_workload_librechat_mcp_admin_username: user1
ocp4_workload_librechat_mcp_admin_password: "{{ lookup('password', '/dev/null length=16') }}"
ocp4_workload_librechat_mcp_email_domain: workshop.example.com

# LiteMaaS configuration (users add keys post-install)
ocp4_workload_librechat_mcp_litemaas_api_key: ""
ocp4_workload_librechat_mcp_skip_litemaas: false
```

### Using with Other Workloads

Combine with other RHDP workloads:

```yaml
---
# common.yaml
workloads:
  # REQUIRED: Install OpenShift GitOps first
  - ocp4_workload_openshift_gitops

  # Install LibreChat with MCP
  - maas_librechat.maas_librechat.ocp4_workload_librechat_mcp

  # Optional: Install Gitea (if you want Gitea MCP server too)
  - rhpds.gitea.ocp4_workload_gitea_operator

# LibreChat configuration
ocp4_workload_librechat_mcp_namespace: librechat
ocp4_workload_librechat_mcp_admin_username: admin
ocp4_workload_librechat_mcp_admin_password: openshift
```

### Passing Variables from Other Workloads

Example: Use LiteMaaS virtual keys from another workload:

```yaml
---
# common.yaml
workloads:
  # First, provision LiteMaaS virtual keys
  - rhpds.litellm_virtual_keys.ocp4_workload_litellm_virtual_keys

  # Then, install LibreChat and pass the key
  - maas_librechat.maas_librechat.ocp4_workload_librechat_mcp

# Pass the LiteMaaS key from the previous workload
ocp4_workload_librechat_mcp_litemaas_api_key: "{{ lookup('agnosticd_user_data', 'litellm_virtual_key') | default('', true) }}"
ocp4_workload_librechat_mcp_litemaas_url: "{{ lookup('agnosticd_user_data', 'litellm_api_base_url') | default('https://litellm.apps.maas.redhatworkshops.io/v1', true) }}"
```

## Using FQCN in Playbooks

### Direct Role Import

```yaml
---
- name: Deploy LibreChat
  hosts: localhost
  tasks:
    - name: Include LibreChat workload using FQCN
      ansible.builtin.include_role:
        name: maas_librechat.maas_librechat.ocp4_workload_librechat_mcp
      vars:
        ocp4_workload_librechat_mcp_namespace: librechat
        ocp4_workload_librechat_mcp_admin_username: admin
```

### Using import_role

```yaml
---
- name: Deploy LibreChat
  hosts: localhost
  tasks:
    - name: Import LibreChat workload using FQCN
      ansible.builtin.import_role:
        name: maas_librechat.maas_librechat.ocp4_workload_librechat_mcp
      vars:
        ocp4_workload_librechat_mcp_namespace: librechat
```

## Variable Precedence

Variables can be set at different levels:

```yaml
# 1. AgnosticV common.yaml (lowest priority)
ocp4_workload_librechat_mcp_namespace: librechat

# 2. AgnosticV dev.yaml (overrides common.yaml)
ocp4_workload_librechat_mcp_namespace: librechat-dev

# 3. Extra vars at runtime (highest priority)
# ansible-playbook ... -e "ocp4_workload_librechat_mcp_namespace=my-custom-ns"
```

## Environment-Specific Configurations

### dev.yaml

```yaml
---
# Development environment
ocp4_workload_librechat_mcp_namespace: librechat-dev
ocp4_workload_librechat_mcp_admin_password: dev123
ocp4_workload_librechat_mcp_litemaas_api_key: ""  # Users add their own
```

### prod.yaml

```yaml
---
# Production environment
ocp4_workload_librechat_mcp_namespace: librechat-prod
ocp4_workload_librechat_mcp_admin_password: "{{ vault_librechat_password }}"
ocp4_workload_librechat_mcp_create_scc: true
ocp4_workload_librechat_mcp_enable_openshift_mcp: true
```

## Verification

After AgnosticV deployment:

```bash
# Check the workload was applied
ansible-playbook -i inventory pre-infra.yml infra.yml default-vars.yml -e @configs/my-config.yml

# Verify in OpenShift
oc get pods -n librechat
oc get route librechat -n librechat
```

## User Data Display

The workload automatically saves user data that will be displayed via AgnosticV:

```
LibreChat URL: https://librechat-librechat.apps.cluster.example.com
Username: admin@example.com
Password: openshift
```

This appears in:
- RHDP catalog provisioning emails
- AgnosticV showroom display
- Babylon UI

## Troubleshooting

### Collection Not Found

```
ERROR! couldn't resolve module/action 'maas_librechat.maas_librechat.ocp4_workload_librechat_mcp'
```

**Fix:**
```bash
# Ensure requirements.yml includes the collection
ansible-galaxy collection install -r requirements.yml -f

# Verify installation
ansible-galaxy collection list | grep maas_librechat
```

### Wrong Variable Names

If variables aren't taking effect, check:
- Variable names match exactly (including the `ocp4_workload_librechat_mcp_` prefix)
- No typos in the FQCN: `maas_librechat.maas_librechat.ocp4_workload_librechat_mcp`
- Variables are in the correct YAML file

### Role Not Running

Check the workload is in the workloads list:

```yaml
workloads:
  - maas_librechat.maas_librechat.ocp4_workload_librechat_mcp  # ✅ Correct
  # NOT:
  # - ocp4_workload_librechat_mcp  # ❌ Missing FQCN
```

## Complete Example

Here's a complete AgnosticV configuration:

```yaml
---
# common.yaml for a LibreChat workshop

# Environment metadata
env_type: ocp4-workshop-librechat-mcp
cloud_provider: ec2
software_to_deploy: openshift4

# OpenShift cluster configuration
ocp4_installer_version: "4.14"

# Collections to install
requirements_content:
  collections:
    - name: maas_librechat.maas_librechat
      version: ">=1.0.0"
    - name: kubernetes.core

# Workloads to deploy
workloads:
  - maas_librechat.maas_librechat.ocp4_workload_librechat_mcp

# LibreChat workload configuration
ocp4_workload_librechat_mcp_namespace: librechat
ocp4_workload_librechat_mcp_admin_username: instructor
ocp4_workload_librechat_mcp_admin_password: openshift
ocp4_workload_librechat_mcp_email_domain: workshop.redhat.com

# LiteMaaS configuration (students add their own keys)
ocp4_workload_librechat_mcp_litemaas_url: https://litellm.apps.maas.redhatworkshops.io/v1
ocp4_workload_librechat_mcp_litemaas_api_key: ""
ocp4_workload_librechat_mcp_litemaas_models:
  - llama-scout-17b
  - granite-3-2-8b-instruct

# MCP server configuration
ocp4_workload_librechat_mcp_enable_openshift_mcp: true
ocp4_workload_librechat_mcp_openshift_namespace: mcp-openshift

# Security
ocp4_workload_librechat_mcp_create_scc: true

# Student info (displayed to users)
student_info: |
  Your LibreChat environment is ready!

  Access it at: {{ librechat_url }}
  Username: {{ librechat_admin_user }}
  Password: {{ librechat_admin_password }}

  Remember to add your LiteMaaS API key following the instructions
  in the workshop guide.
```

## Next Steps

1. **Test locally first** - Build and install the collection in your dev environment
2. **Create AgnosticV config** - Add to your environment's common.yaml
3. **Deploy to dev cluster** - Test with AgnosticV in development
4. **Document for users** - Include post-install steps in your workshop guide
5. **Deploy to production** - Once tested, deploy to RHDP catalog

## Resources

- [AgnosticD Documentation](https://github.com/redhat-cop/agnosticd)
- [Ansible Collections Guide](https://docs.ansible.com/ansible/latest/user_guide/collections_using.html)
- [RHDP Developer Guide](https://red.ht/rhdp-docs)
