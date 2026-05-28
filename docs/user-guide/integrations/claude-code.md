# Claude Code|
| `saturn_ssh_setup` | One-step setup: generate a Saturn-dedicated keypair locally and register the public key with your account |
| `saturn_ssh_key_list` | List SSH keys registered with Saturn Cloud |
| `saturn_ssh_key_register` | Register an existing public key with Saturn Cloud |
| `saturn_ssh_key_delete` | Remove a registered key by ID |
| `saturn_get_ssh_url` | Get the SSH URL of a running workspace or deployment (requires `start_ssh: true` in the recipe) |

Combined with the bundled skill, this means you can ask Claude things like:

- *"Create a Jupyter workspace with a GPU and the latest PyTorch image."*
- *"Schedule my `nightly-etl` job to run every day at 2am UTC."*
- *"Stop every workspace I'm not using right now."*
- *"Show me the last 100 log lines from my failing deployment."*
- *"Bump the instance type on my training job to an `xlarge` GPU."*
- *"Set up SSH to Saturn, then connect me to my `dev` workspace."*

Claude reads the recipe schema from the skill, so it authors valid recipes and explains trade-offs (instance sizes, disk space, image choices) without you needing to remember the field names.

## Updating the plugin

Update the plugin from within Claude Code:

```
/plugin update saturn-cloud@saturn-local
```

Restart Claude Code afterwards. `uvx` picks up the new plugin code on the next launch; if dependencies changed it will refresh its cache automatically.

## Troubleshooting

- **`/plugin install` fails with "marketplace not found".** Make sure you ran `/plugin marketplace add saturncloud/claude-plugin` first.
- **The MCP server fails to start.** Confirm `uv` is installed and on the `PATH` of the shell that launched Claude Code (`uvx --version` should succeed). The MCP server is invoked as `uvx --from <plugin-dir> saturn-mcp`.
- **Tools return authentication errors.** Re-check `SATURN_BASE_URL` and `SATURN_TOKEN` are exported in the shell that launched Claude Code. The token must belong to the same Saturn installation as `SATURN_BASE_URL`.
- **`saturn_get_ssh_url` returns "no ssh_url".** The target resource must be running and its recipe must set `start_ssh: true`. Update the recipe with `saturn_apply_recipe`, restart the resource, and try again.

## Feedback and contributions

Bug reports, feature requests, and pull requests are welcome on the [GitHub repository](https://github.com/saturncloud/claude-plugin). For general Saturn Cloud questions, email [support@saturncloud.io](mailto:support@saturncloud.io).
