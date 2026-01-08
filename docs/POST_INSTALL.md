# Post-Installation Configuration

After installing LibreChat with the `ocp4_workload_librechat_mcp` role, you need to configure your LLM API keys to enable AI functionality.

## Why Configure API Keys Post-Installation?

This workload is designed for educational environments where:
- **Students bring their own keys** - No shared credentials or pre-paid access
- **Instructors focus on teaching** - Not on managing API key distribution
- **Clear cost ownership** - Each user controls their own LLM usage and billing

## Getting Your LiteMaaS API Key

### Option 1: Red Hat LiteMaaS (Recommended for RHDP)

If you have access to Red Hat's MaaS platform:

1. Navigate to the LiteMaaS portal
2. Generate a new API key for your user
3. Copy the key (starts with `sk-` or similar)

### Option 2: Other LLM Providers

You can also use other OpenAI-compatible providers:
- **OpenAI** - https://platform.openai.com/api-keys
- **Anthropic** - https://console.anthropic.com/settings/keys
- **Azure OpenAI** - Azure portal
- **Local LLMs** - Ollama, LocalAI, etc.

## Adding Your API Key

### Method 1: OpenShift CLI (Recommended)

Update the secret with your API key:

```bash
# Replace YOUR_API_KEY with your actual key
oc patch secret librechat-credentials-env \
  -n librechat \
  -p '{"stringData":{"CUSTOM_MODEL_KEY":"YOUR_API_KEY"}}'
```

Restart LibreChat to pick up the change:

```bash
oc rollout restart deployment/librechat -n librechat
```

Wait for the new pod to be ready:

```bash
oc rollout status deployment/librechat -n librechat
```

### Method 2: OpenShift Web Console

1. Navigate to **Workloads → Secrets** in the `librechat` namespace
2. Find the `librechat-credentials-env` secret
3. Click **Actions → Edit Secret**
4. Find the `CUSTOM_MODEL_KEY` field
5. Replace `REPLACE_WITH_YOUR_LITEMAAS_API_KEY` with your actual key
6. Click **Save**
7. Go to **Workloads → Deployments**
8. Find the `librechat` deployment
9. Click **Actions → Restart rollout**

### Method 3: Edit Secret YAML Directly

Edit the secret:

```bash
oc edit secret librechat-credentials-env -n librechat
```

Find the `CUSTOM_MODEL_KEY` field and update its value (it's base64 encoded):

```yaml
stringData:
  CUSTOM_MODEL_KEY: "YOUR_API_KEY"
```

Save and exit. Then restart:

```bash
oc rollout restart deployment/librechat -n librechat
```

## Verifying the Configuration

### Check the Secret Value

Verify your key is set (be careful - this shows the key in plain text):

```bash
oc get secret librechat-credentials-env -n librechat \
  -o jsonpath='{.data.CUSTOM_MODEL_KEY}' | base64 -d
echo
```

### Test LibreChat

1. Get the LibreChat URL:
   ```bash
   oc get route librechat -n librechat -o jsonpath='{.spec.host}'
   ```

2. Open the URL in your browser

3. Login with your credentials:
   - Email: `admin@example.com` (or whatever you configured)
   - Password: From your workload variables

4. Start a new chat

5. Select **LiteMaaS** as the model

6. Send a test message like "Hello, can you hear me?"

If you see a response, your API key is working!

## Using Other LLM Providers

### OpenAI

Update the secret with both URL and key:

```bash
oc patch secret librechat-credentials-env -n librechat -p '{
  "stringData": {
    "CUSTOM_MODEL_URL": "https://api.openai.com/v1",
    "CUSTOM_MODEL_KEY": "sk-your-openai-key"
  }
}'

oc rollout restart deployment/librechat -n librechat
```

### Anthropic (Claude)

Anthropic requires a different endpoint format. Update the LibreChat config:

```bash
# This requires editing the Helm values and redeploying
# See LibreChat documentation for Anthropic integration
```

### Local LLMs (Ollama)

If you have Ollama running in your cluster:

```bash
oc patch secret librechat-credentials-env -n librechat -p '{
  "stringData": {
    "CUSTOM_MODEL_URL": "http://ollama.ollama.svc.cluster.local:11434/v1",
    "CUSTOM_MODEL_KEY": "not-needed"
  }
}'

oc rollout restart deployment/librechat -n librechat
```

## Adding Multiple LLM Providers

LibreChat supports multiple providers simultaneously. To add more:

1. You can add additional API keys via the LibreChat UI:
   - Login to LibreChat
   - Go to **Settings → API Keys**
   - Add keys for OpenAI, Anthropic, etc.

2. Or configure them in the Helm values by re-running the workload with updated variables.

## Troubleshooting

### "No models available" Error

This means the API key is not configured or invalid.

**Check:**
```bash
# Verify the secret exists
oc get secret librechat-credentials-env -n librechat

# Check the key value
oc get secret librechat-credentials-env -n librechat \
  -o jsonpath='{.data.CUSTOM_MODEL_KEY}' | base64 -d
```

**Fix:**
- Ensure the key is set correctly
- Verify the key is valid (test with curl or in LiteMaaS portal)
- Restart the deployment

### "Unauthorized" or "Invalid API Key" Error

Your key is configured but not valid.

**Check:**
```bash
# Test the API key directly
curl https://litellm.apps.maas.redhatworkshops.io/v1/models \
  -H "Authorization: Bearer YOUR_API_KEY"
```

**Fix:**
- Regenerate your API key in the LiteMaaS portal
- Update the secret with the new key
- Restart the deployment

### LibreChat Won't Start After Updating Secret

**Check:**
```bash
# View pod logs
oc logs -n librechat deployment/librechat --tail=50

# Check pod status
oc get pods -n librechat
```

**Fix:**
- Ensure the secret is valid YAML
- Check for typos in the key value
- Verify MongoDB and Meilisearch are running

### Changes Not Taking Effect

**Fix:**
```bash
# Force delete the pod (it will recreate)
oc delete pod -n librechat -l app.kubernetes.io/name=librechat

# Or do a full restart
oc rollout restart deployment/librechat -n librechat
```

## Security Best Practices

### Protect Your API Keys

- **Never commit keys to Git** - Always use secrets
- **Use separate keys per user** - Don't share keys in workshops
- **Rotate keys regularly** - Especially in shared environments
- **Monitor usage** - Check your LiteMaaS dashboard for unexpected usage
- **Revoke compromised keys immediately** - Regenerate in the portal

### RBAC Considerations

By default, all users with access to the `librechat` namespace can view secrets.

Restrict access:

```bash
# Remove default view permissions
oc policy remove-role-from-group view system:authenticated -n librechat

# Grant specific users access
oc policy add-role-to-user view instructor -n librechat
```

## Workshop Scenarios

### Scenario 1: Instructor-Provided Key (Demo Mode)

Instructor adds one shared key for demos:

```bash
oc patch secret librechat-credentials-env -n librechat \
  -p '{"stringData":{"CUSTOM_MODEL_KEY":"'$INSTRUCTOR_KEY'"}}'
oc rollout restart deployment/librechat -n librechat
```

Students use LibreChat with the shared key during demos.

### Scenario 2: Student Self-Configuration (Hands-On)

Each student adds their own key:

1. Instructor provides these instructions
2. Students get their own LiteMaaS keys
3. Students update the secret (need edit permissions)
4. Each student tests their own configuration

### Scenario 3: Pre-Configured Per-User Namespaces

For multi-user deployments (future enhancement):
- Deploy one LibreChat instance per student
- Each in their own namespace: `librechat-user1`, `librechat-user2`, etc.
- Students only have access to their own namespace and secret

## Additional Configuration

### Changing Admin Password

Update the user password:

```bash
oc patch secret librechat-credentials-env -n librechat \
  -p '{"stringData":{"USER_PASSWORD":"new-secure-password"}}'
```

Note: This doesn't change existing users, only affects initial setup.

To change password for existing user:
- Login to LibreChat
- Go to **Settings → Account**
- Change password in the UI

### Adding More Models

Edit the list of available models:

```bash
# This requires updating the Helm values and redeploying
# See role variables: ocp4_workload_librechat_mcp_litemaas_models
```

### Enabling/Disabling Features

Configure LibreChat features via the Helm values:
- Registration: `ALLOW_REGISTRATION`
- Email login: `ALLOW_EMAIL_LOGIN`
- Social auth: Configure in `librechat.configYamlContent`

See [LibreChat documentation](https://www.librechat.ai/docs/configuration) for all options.

## Next Steps

Once your API key is configured:

1. **Explore LibreChat** - Try different models and features
2. **Test MCP Integration** - Use the OpenShift MCP server to interact with cluster resources
3. **Create Conversations** - Build AI assistants, try RAG capabilities
4. **Share Knowledge** - Help other students get set up

## Resources

- [LibreChat Documentation](https://www.librechat.ai/docs)
- [LiteMaaS Portal](https://litellm.apps.maas.redhatworkshops.io) (Red Hat internal)
- [Model Context Protocol](https://modelcontextprotocol.io/)
- [OpenShift Secrets Guide](https://docs.openshift.com/container-platform/latest/nodes/pods/nodes-pods-secrets.html)

## Getting Help

If you're stuck:
- Check the troubleshooting section above
- Review LibreChat logs: `oc logs -n librechat deployment/librechat`
- Ask your instructor or workshop facilitator
- Open an issue: https://github.com/rhpds/maas-librechat/issues
