---
name: n8n-tools
description: "Build, validate, and debug n8n workflows using the n8n MCP server. Provides search_nodes, get_node, validate_node, validate_workflow for node discovery and validation; search_templates, get_template for template reuse; and n8n_create_workflow, n8n_get_workflow, n8n_list_workflows, n8n_test_workflow, n8n_deploy_template for workflow lifecycle management. Use when building n8n automations, validating workflow configurations, debugging failed n8n executions, or deploying workflow templates."
---

# n8n Automation Tools

Build, validate, and deploy n8n workflows using the n8n MCP server.

## Workflow

1. **Discover** — Search for the right nodes with `search_nodes`, inspect with `get_node`
2. **Scaffold** — Find similar workflows via `search_templates` / `get_template`, or create from scratch with `n8n_create_workflow`
3. **Validate** — Run `validate_node` on each node and `validate_workflow` on the full workflow before deploying
4. **Test** — Execute the workflow with `n8n_test_workflow` and verify output
5. **Deploy** — Push to production with `n8n_deploy_template` or manage with `n8n_list_workflows` / `n8n_get_workflow`

## Available MCP Tools

| Tool | Purpose |
|------|---------|
| `search_nodes` | Find n8n nodes by name or category |
| `get_node` | Get detailed node configuration and parameters |
| `validate_node` | Validate a single node's configuration |
| `validate_workflow` | Validate an entire workflow definition |
| `search_templates` | Search the n8n template library |
| `get_template` | Retrieve a specific template's full definition |
| `n8n_create_workflow` | Create a new workflow |
| `n8n_get_workflow` | Retrieve an existing workflow by ID |
| `n8n_list_workflows` | List all workflows in the instance |
| `n8n_test_workflow` | Execute a workflow in test mode |
| `n8n_deploy_template` | Deploy a template as a live workflow |

## Related Skills

Use these n8n-specific skills for deeper guidance: `n8n-node-configuration`, `n8n-code-javascript`, `n8n-code-python`, `n8n-workflow-patterns`, `n8n-expression-syntax`, `n8n-validation-expert`.
