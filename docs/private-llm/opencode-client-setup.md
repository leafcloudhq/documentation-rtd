# Configuring OpenCode with a Private LLM Server

This guide shows how to configure [OpenCode](https://opencode.ai) to use your team's private LLM server.

## Prerequisites

You'll need from your administrator:

- **Server IP or URL** - The address of the vLLM server
- **API Key** - For authentication
- **Model name** - The model running on the server (e.g., `nm-testing/Qwen3-Coder-30B-A3B-Instruct-W4A16-awq`)

---

## Step 1: Install OpenCode

If you haven't installed OpenCode yet:

```bash
curl -fsSL https://opencode.ai/install | bash
```

Verify the installation:

```bash
opencode --version
```

---

## Step 2: Create the Configuration File

Create the OpenCode configuration directory:

```bash
mkdir -p ~/.config/opencode
```

Create or edit `~/.config/opencode/opencode.json`:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "provider": {
    "leafcloud": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "Leafcloud Team LLM",
      "options": {
        "baseURL": "https://<SERVER-IP>/v1",
        "apiKey": "<YOUR-API-KEY>"
      },
      "models": {
        "nm-testing/Qwen3-Coder-30B-A3B-Instruct-W4A16-awq": {
          "name": "Qwen3-Coder (AWQ)",
          "limit": {
            "context": 32768,
            "output": 16384
          }
        }
      }
    }
  },
  "model": "leafcloud/nm-testing/Qwen3-Coder-30B-A3B-Instruct-W4A16-awq"
}
```

Replace:

- `<SERVER-IP>` - The IP address or domain of your vLLM server
- `<YOUR-API-KEY>` - The API key provided by your administrator

!!! tip "Different Model?"
    If your server runs a different model, update the `models` section and the `model` default accordingly.

!!! info "About the Quantized Model"
    We use `nm-testing/Qwen3-Coder-30B-A3B-Instruct-W4A16-awq`, a **4-bit AWQ quantized** version of Qwen3-Coder. This reduces memory usage by ~4x while maintaining nearly identical quality, allowing the model to run efficiently on a single A100 GPU. The "W4A16" means 4-bit weights with 16-bit activations.

---

## Step 3: Test the Connection

### 3.1 Test the API Endpoint

Verify you can reach the server:

```bash
curl -k https://<SERVER-IP>/v1/models \
  -H "Authorization: Bearer <YOUR-API-KEY>"
```

You should see a JSON response listing the available model:

```json
{
  "object": "list",
  "data": [
    {
      "id": "nm-testing/Qwen3-Coder-30B-A3B-Instruct-W4A16-awq",
      "object": "model",
      ...
    }
  ]
}
```

### 3.2 Test with OpenCode

Start OpenCode in any project:

```bash
cd your-project
NODE_TLS_REJECT_UNAUTHORIZED=0 opencode
```

!!! warning "Self-Signed Certificate Workaround"
    The server currently uses a self-signed TLS certificate. The `NODE_TLS_REJECT_UNAUTHORIZED=0` environment variable disables certificate verification in Node.js. This is safe for internal/private servers but should not be used for public endpoints. Proper TLS with trusted certificates is planned for a future update.

**Tip:** Add an alias to your shell profile (`~/.bashrc` or `~/.zshrc`) for convenience:

```bash
alias opencode='NODE_TLS_REJECT_UNAUTHORIZED=0 opencode'
```

OpenCode should automatically use your configured Leafcloud provider.

---

## Step 4: Start Coding

You're ready to use OpenCode with your team's private LLM server!

**Tips for best results:**

- Use **Plan mode** (Tab key) for complex tasks - it helps the model think through the problem
- Provide context by mentioning relevant files with `@filename`
- The model excels at agentic coding tasks (multi-step, tool calling)

---

## Configuration Options

### Context and Output Limits

Adjust the limits based on your server's configuration:

```json
"limit": {
  "context": 32768,   // Max input tokens
  "output": 16384     // Max output tokens
}
```

!!! note
    These should match your server's `vllm_max_model_len` setting.

### Multiple Models

If your server runs multiple models:

```json
"models": {
  "nm-testing/Qwen3-Coder-30B-A3B-Instruct-W4A16-awq": {
    "name": "Qwen3-Coder AWQ (30B)",
    "limit": { "context": 32768, "output": 16384 }
  },
  "Qwen/Qwen2.5-Coder-14B-Instruct": {
    "name": "Qwen2.5-Coder (14B)",
    "limit": { "context": 32768, "output": 8192 }
  }
}
```

### Custom Provider Name

Change the display name:

```json
"name": "My Company AI"
```

---

## Troubleshooting

### "Connection refused" or timeout

- Check if the server IP is correct
- Verify your IP is whitelisted in the server's security group
- Ask your admin if the server is running

### "Unauthorized" or 401 error

- Double-check your API key
- Make sure there are no extra spaces or newlines in the key

### "Model not found"

- Verify the model name matches exactly what's running on the server
- Check with `curl -k https://<SERVER-IP>/v1/models`

### Certificate warnings / TLS errors

The server uses self-signed certificates. This is normal for internal servers.

- For **curl**: use the `-k` flag to skip certificate verification
- For **OpenCode**: use `NODE_TLS_REJECT_UNAUTHORIZED=0 opencode` (see [Step 3.2](#32-test-with-opencode))

### Slow responses

- Large context windows use more GPU memory and take longer
- Complex tasks may require multiple tool calls
- Check with your admin if the server is under heavy load

---

## Security Notes

1. **Keep your API key secret** - Don't commit it to git or share it publicly
2. **Use environment variables** - For extra security, use:
   ```json
   "apiKey": "{env:LEAFCLOUD_API_KEY}"
   ```
   Then set `export LEAFCLOUD_API_KEY=your-key` in your shell profile

3. **VPN recommended** - For sensitive codebases, access the server through a VPN

---

## Related Tutorials

- [Deploy Team OpenCode Server](team-opencode-vllm.md) - Set up the server (for admins)
- [OpenCode Documentation](https://opencode.ai/docs) - Full OpenCode docs
