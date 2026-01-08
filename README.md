# MaaS LibreChat Ansible Collection

Ansible collection for deploying LibreChat with OpenShift MCP (Model Context Protocol) server support on Red Hat OpenShift.

## Overview

This collection provides a workload that installs LibreChat - an open-source AI chat interface - along with OpenShift MCP server integration. Designed for educational and workshop environments where users bring their own LLM API keys.

## Key Features

- **No pre-configured credentials** - Users add their own LiteMaaS or OpenAI API keys
- **OpenShift native** - Uses Helm for deployment, no GitOps dependencies
- **MCP integration** - Connects LibreChat to OpenShift resources via Model Context Protocol
- **Simple installation** - Single workload role, minimal configuration
- **Educational focus** - Perfect for workshops, labs, and learning environments

## What's Included

### Roles

- `ocp4_workload_librechat_mcp` - Main workload role that deploys:
  - LibreChat web interface
  - MongoDB database
  - Meilisearch for vector search
  - OpenShift MCP server (optional)
  - OpenShift routes and security contexts

## Requirements

- Ansible 2.15+
- OpenShift 4.12+
- `kubernetes.core` collection
- Cluster admin access (for Security Context Constraints)

## Installation

Install this collection:

```bash
ansible-galaxy collection install maas_librechat.maas_librechat
```

Or add to `requirements.yml`:

```yaml
---
collections:
  - name: maas_librechat.maas_librechat
    version: ">=1.0.0"
```

## Quick Start

### AgnosticV Integration

Add to your AgnosticV `common.yaml`:

```yaml
workloads:
  - maas_librechat.maas_librechat.ocp4_workload_librechat_mcp

# Optional customization
ocp4_workload_librechat_mcp_namespace: my-librechat
ocp4_workload_librechat_mcp_admin_username: instructor
ocp4_workload_librechat_mcp_admin_password: secure-password
```

### Standalone Playbook

```yaml
---
- name: Deploy LibreChat with MCP
  hosts: localhost
  connection: local
  gather_facts: false

  tasks:
    - name: Include LibreChat MCP workload
      ansible.builtin.include_role:
        name: maas_librechat.maas_librechat.ocp4_workload_librechat_mcp
      vars:
        ocp4_workload_librechat_mcp_namespace: librechat
        ocp4_workload_librechat_mcp_admin_username: admin
        ocp4_workload_librechat_mcp_admin_password: openshift
```

Run the playbook:

```bash
ansible-playbook deploy-librechat.yml
```

## Post-Installation

After deployment, users must add their own LiteMaaS API keys:

```bash
# Update the secret with your LiteMaaS API key
oc patch secret librechat-credentials-env \
  -n librechat \
  -p '{"stringData":{"CUSTOM_MODEL_KEY":"sk-your-api-key"}}'

# Restart LibreChat to pick up the new key
oc rollout restart deployment/librechat -n librechat
```

See [docs/POST_INSTALL.md](docs/POST_INSTALL.md) for detailed instructions.

## Configuration

### Common Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `ocp4_workload_librechat_mcp_namespace` | `librechat` | Target namespace |
| `ocp4_workload_librechat_mcp_admin_username` | `admin` | Admin username |
| `ocp4_workload_librechat_mcp_admin_password` | `openshift` | Admin password |
| `ocp4_workload_librechat_mcp_enable_openshift_mcp` | `true` | Enable MCP server |
| `ocp4_workload_librechat_mcp_litemaas_api_key` | `""` | LiteMaaS API key (empty by default) |

For complete variable list, see [role documentation](roles/ocp4_workload_librechat_mcp/README.md).

## Architecture

This collection deploys:

1. **LibreChat** - AI chat interface supporting multiple LLM providers
2. **MongoDB** - Database for chat history and user data
3. **Meilisearch** - Vector search engine for RAG capabilities
4. **OpenShift MCP Server** - Connects LibreChat to OpenShift APIs

All components run in OpenShift with proper security contexts and are accessible via OpenShift routes.

## Use Cases

### Workshop/Training Labs

Perfect for instructors teaching AI + OpenShift integration:
- Students bring their own LiteMaaS API keys
- Instructor deploys the infrastructure
- Focus on using AI with Kubernetes, not credential management

### Development Environments

Quick setup for developers testing AI applications:
- No need for cloud accounts or payment methods upfront
- Add API keys when ready
- Test MCP integrations with OpenShift

### Education Modules

Part of Red Hat Demo Platform (RHDP) learning content:
- Consistent deployment via AgnosticD
- Integrates with other RHDP workloads
- Clear, repeatable installation process

## Documentation

- [Role README](roles/ocp4_workload_librechat_mcp/README.md) - Full role documentation
- [Post-Install Guide](docs/POST_INSTALL.md) - Adding API keys and configuration
- [LibreChat Docs](https://www.librechat.ai/docs) - Upstream documentation
- [MCP Documentation](https://modelcontextprotocol.io/) - Model Context Protocol

## Support

This collection is maintained by Red Hat Technical Marketing as part of the RHDP initiative.

For issues and questions:
- GitHub Issues: https://github.com/rhpds/maas-librechat/issues
- Internal: Reach out to Prakhar Srivastava

## License

Apache License 2.0 - see LICENSE file for details.

## Author

**Prakhar Srivastava**
Manager, Technical Marketing
Red Hat, Inc.
