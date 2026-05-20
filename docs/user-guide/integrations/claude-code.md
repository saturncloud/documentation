# Claude Code|
| `saturn_list_resources` | List your workspaces, jobs, and deployments |
| `saturn_get_resource` | Fetch a resource's full recipe |
| `saturn_apply_recipe` | Create or update a resource from a recipe |
| `saturn_delete_resource` | Delete a resource |
| `saturn_start_resource` / `saturn_stop_resource` | Control runtime state |
| `saturn_schedule_job` | Set a cron schedule on a job |
| `saturn_get_logs` | Fetch logs for a resource |
| `saturn_list_instance_types` | List available compute sizes |

Combined with the bundled skill, this means you can ask Claude things like:

- *"Create a Jupyter workspace with a GPU and the latest PyTorch image."*
- *"Schedule my `nightly-etl` job to run every day at 2am UTC."*
- *"Stop every workspace I'm not using right now."*
- *"Show me the last 100 log lines from my failing deployment."*
- *"Bump the instance type on my training job to an `xlarge` GPU."*

Claude reads the recipe schema from the skill, so it will author valid recipes and explain trade-offs (instance sizes, disk space, image choices) without you needing to remember the field names.

## Updating the plugin

Update the plugin from within Claude Code:

```
/plugin update saturn-cloud@saturn-local
```

If the update changes Python dependencies, re-sync the environment:

```sh
cd ~/.claude/plugins/marketplaces/saturncloud/claude-plugin
uv sync
```

Restart Claude Code afterwards so it loads the refreshed environment.

## Troubleshooting

- **`/plugin install` fails with "marketplace not found"** — make sure you ran `/plugin marketplace add saturncloud/claude-plugin` first.
- **The MCP server fails to start** — confirm `uv sync` ran successfully inside the installed plugin directory and that `.venv/bin/python` exists there. Restart Claude Code after running `uv sync`.
- **Tools return authentication errors** — re-check `SATURN_BASE_URL` and `SATURN_TOKEN` are exported in the shell that launched Claude Code. The token must belong to the same Saturn installation as `SATURN_BASE_URL`.

## Feedback and contributions

Bug reports, feature requests, and pull requests are welcome on the [GitHub repository](https://github.com/saturncloud/claude-plugin). For general Saturn Cloud questions, email [support@saturncloud.io](mailto:support@saturncloud.io).
