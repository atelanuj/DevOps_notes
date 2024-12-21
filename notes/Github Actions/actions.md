# GitHub Actions CI/CD

## [**The components of GitHub Actions**](https://docs.github.com/en/actions/about-github-actions/understanding-github-actions#the-components-of-github-actions)

- You can configure a GitHub Actions **workflow** to be triggered when an **event** occurs in your repository, such as a pull request being opened or an issue being created.
- Each job will run inside its own virtual machine **runner**, or inside a container, and has one or more **steps** that either run a script that you define or run an **action**, which is a reusable extension that can simplify your workflow.

![image](https://docs.github.com/assets/cb-25535/mw-1440/images/help/actions/overview-actions-simple.webp)

!https://docs.github.com/assets/cb-25535/mw-1440/images/help/actions/overview-actions-simple.webp

## [**Workflows**](https://docs.github.com/en/actions/about-github-actions/understanding-github-actions#workflows)

- Workflows are defined by a YAML file checked in to your repository and will run when triggered by an **event in your repository**, or they can be **triggered manually**, or at a defined **schedule**.
- Workflows are defined in the `.github/workflows`
- **Use cases:**
    - Building and testing pull requests
    - Deploying your application every time a release is created
    - Adding a label whenever a new issue is opened

## [**Events**](https://docs.github.com/en/actions/about-github-actions/understanding-github-actions#events)

- specific activity in a repository that triggers a **workflow** run
    - pull request, opens an issue, or pushes a commit to a repository. You can also trigger a workflow to run on a [schedule](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#schedule), by [posting to a REST API](https://docs.github.com/en/rest/repos/repos#create-a-repository-dispatch-event), or manually.
- Examples:

```yaml
on:
  pull_request:
    types: [opened, reopened]
    
 on:
  schedule:
    # * is a special character in YAML so you have to quote this string
    - cron:  '30 5,17 * * *'
    
 on:
  push
 
 # You can manually trigger a workflow run using the GitHub API, GitHub CLI, or GitHub browser interface.
 on: workflow_dispatch
```

[Workflow Dispatch](https://www.notion.so/Workflow-Dispatch-1636293bee7280909fa9c96c6ecdcf52?pvs=21)