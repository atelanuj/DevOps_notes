# Workflow Dispatch

- To enable a workflow to be triggered manually, you need to configure the `workflow_dispatch` event. You can manually trigger a workflow run using the GitHub API, GitHub CLI, or GitHub browser interface. For more information, see [Manually running a workflow](https://docs.github.com/en/actions/managing-workflow-runs/manually-running-a-workflow).

```yaml
on: workflow_dispatch
```

## Providing Inputs

- To manually provide the Input to the workflow you can use the `inputs` inside the `workflow_dispatch`
- Example:

```yaml
name: WorkFlow Dispatch
on:
    workflow_dispatch:
      inputs:
        logLevel:
          description: 'Log level'
          required: true
          default: 'warning'
          type: choice
          options:
          - info
          - warning
          - debug
        tags:
          description: 'Test scenario tags'
          required: false
          type: boolean
        environment:
          description: 'Environment to run tests against'
          type: choice
          required: true
          options:
            - Dev
            - Prod
            - UAT
            - non-prod
  
jobs:
    log-the-inputs:
      runs-on: ubuntu-latest
      steps:
        - run: |
            echo "Log level: $LEVEL"
            echo "Tags: $TAGS"
            echo "Environment: $ENVIRONMENT"
          env:
            LEVEL: ${{ inputs.logLevel }}
            TAGS: ${{ inputs.tags }}
            ENVIRONMENT: ${{ inputs.environment }}
```

![image](https://docs.github.com/assets/cb-78157/mw-1440/images/help/actions/workflow-dispatch-inputs.webp)

!https://docs.github.com/assets/cb-78157/mw-1440/images/help/actions/workflow-dispatch-inputs.webp