# ocp4_workload_librechat_mcp

Deploy LibreChat with OpenShift MCP server support on Red Hat OpenShift.

## Overview

This workload installs:
- **LibreChat** - AI chat interface that supports multiple LLM providers
- **OpenShift MCP Server** - Model Context Protocol server for OpenShift interactions
- **Required infrastructure** - Namespaces, routes, secrets, and security contexts

## Key Features

- **User-configurable LLM credentials** - No pre-configured API keys required
- **OpenShift native** - Uses Argo CD GitOps for deployment
- **MCP integration** - Connect LibreChat to OpenShift resources via MCP
- **Secure by default** - Auto-generates encryption keys and JWT secrets
- **Educational focus** - Designed for workshops and learning environments

## Requirements

- OpenShift 4.12+
- Cluster admin access for SCC creation
- Valid LiteMaaS API key (added post-installation)

**Note**: This workload automatically installs OpenShift GitOps (Argo CD) if not already present.

## Quick Start

### AgnosticV Integration

Add this workload to your AgnosticV `common.yaml`:

```yaml
workloads:
  - maas_librechat.maas_librechat.ocp4_workload_librechat_mcp

# Optional: Customize variables
ocp4_workload_librechat_mcp_namespace: librechat
ocp4_workload_librechat_mcp_admin_username: instructor
ocp4_workload_librechat_mcp_admin_password: openshift
```

### Standalone Playbook

```yaml
---
- name: Deploy LibreChat with MCP
  hosts: localhost
  tasks:
    - name: Run LibreChat workload
      ansible.builtin.include_role:
        name: maas_librechat.maas_librechat.ocp4_workload_librechat_mcp
      vars:
        ocp4_workload_librechat_mcp_namespace: librechat
        ocp4_workload_librechat_mcp_admin_username: admin
        ocp4_workload_librechat_mcp_admin_password: openshift
```

## Variables

### Core Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `ocp4_workload_librechat_mcp_namespace` | `librechat` | Namespace for LibreChat |
| `ocp4_workload_librechat_mcp_admin_username` | `admin` | Default admin username |
| `ocp4_workload_librechat_mcp_admin_password` | `openshift` | Default admin password |
| `ocp4_workload_librechat_mcp_email_domain` | `example.com` | Email domain for users |

### LiteMaaS Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `ocp4_workload_librechat_mcp_litemaas_url` | `https://litellm.apps.maas.redhatworkshops.io/v1` | LiteMaaS endpoint |
| `ocp4_workload_librechat_mcp_litemaas_api_key` | `""` | LiteMaaS API key (empty by default) |
| `ocp4_workload_librechat_mcp_litemaas_models` | `["llama-scout-17b"]` | Available models |
| `ocp4_workload_librechat_mcp_skip_litemaas` | `false` | Skip LiteMaaS endpoint config |

### MCP Server Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `ocp4_workload_librechat_mcp_enable_openshift_mcp` | `true` | Enable OpenShift MCP server |
| `ocp4_workload_librechat_mcp_openshift_namespace` | `mcp-openshift` | MCP server namespace |
| `ocp4_workload_librechat_mcp_openshift_image` | `quay.io/containers/kubernetes_mcp_server:latest` | MCP server image |

### Advanced Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `ocp4_workload_librechat_mcp_create_scc` | `true` | Create Security Context Constraint |
| `ocp4_workload_librechat_mcp_repo` | `https://github.com/danny-avila/LibreChat` | LibreChat Helm chart repo |
| `ocp4_workload_librechat_mcp_repo_tag` | `v0.8.1` | LibreChat version |

## Post-Installation

After installation, you must add LiteMaaS API keys to enable AI functionality.

See [docs/POST_INSTALL.md](../../docs/POST_INSTALL.md) for detailed instructions.

### Quick Steps

1. Get your LiteMaaS API key
2. Update the secret:
   ```bash
   oc patch secret librechat-credentials-env \
     -n librechat \
     -p '{"stringData":{"CUSTOM_MODEL_KEY":"your-api-key-here"}}'
   ```
3. Restart LibreChat:
   ```bash
   oc rollout restart deployment/librechat -n librechat
   ```

## Architecture

```
┌─────────────────────────────────────────────────┐
│  Ansible Workload                                │
│  - Installs OpenShift GitOps (if needed)         │
│  - Creates Argo CD Applications                  │
└─────────────────┬───────────────────────────────┘
                  │
                  ▼
        ┌─────────────────────┐
        │  OpenShift GitOps   │
        │  (Argo CD)          │
        └─────────┬───────────┘
                  │
        ┌─────────┴─────────┐
        │                   │
        ▼                   ▼
┌──────────────┐    ┌──────────────┐
│  LibreChat   │◄───┤  MCP Server  │
│  Namespace   │    │  Namespace   │
├──────────────┤    ├──────────────┤
│ - LibreChat  │    │ - MCP Proxy  │
│ - MongoDB    │    │ - Route      │
│ - Meilisearch│    └──────────────┘
│ - Route      │
│ - Secrets    │
└──────────────┘
```

## Educational Use

This workload is designed for workshops and training where:
- Students bring their own LLM API keys
- Instructors demonstrate LibreChat + MCP integration
- Each student configures their own LiteMaaS access
- Focus is on using AI with OpenShift, not on credential management

## Troubleshooting

### LibreChat won't start

Check the deployment status:
```bash
oc get deployment librechat -n librechat
oc logs deployment/librechat -n librechat
```

### No models available

Verify the LiteMaaS API key is set:
```bash
oc get secret librechat-credentials-env -n librechat -o jsonpath='{.data.CUSTOM_MODEL_KEY}' | base64 -d
```

### MCP server connection fails

Check the MCP server is running:
```bash
oc get pods -n mcp-openshift
oc logs -n mcp-openshift deployment/mcp-openshift
```

## References

- [LibreChat Documentation](https://www.librechat.ai/docs)
- [Model Context Protocol](https://modelcontextprotocol.io/)
- [OpenShift MCP Server](https://github.com/rhpds/mcp-gitops)
- [AgnosticD v2](https://github.com/agnosticd/)

## Author

Prakhar Srivastava - Red Hat Technical Marketing
