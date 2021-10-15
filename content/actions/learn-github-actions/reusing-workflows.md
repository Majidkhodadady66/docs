---
title: Reusing workflows
shortTitle: Reusing workflows
intro: 'Learn how to avoid duplication when creating a workflow by reusing existing workflows.'
miniTocMaxHeadingLevel: 3
versions:
  fpt: '*'
  ghes: '>=3.4'
  ghae: 'issue-4757'
  ghec: '*'
type: how_to
topics:
  - Workflows
---

{% data reusables.actions.enterprise-beta %}
{% data reusables.actions.enterprise-github-hosted-runners %}
{% data reusables.actions.ae-beta %}

{% note %}

**Note:** Reusable workflows are currently in beta and subject to change.

{% endnote %}

## Overview

Rather than copying and pasting from one workflow to another, you can make workflows reusable. You and anyone with access to the reusable workflow can then call the reusable workflow from another workflow. 

Reusing workflows avoids duplication. This makes workflows easier to maintain and allows you to create new workflows more quickly by building on the work of others, just as you do with actions. Workflow reuse also promotes best practice by helping you to use workflows that are well designed, have already been tested, and have been proved to be effective. Your organization can build up a library of reusable workflows that can be centrally maintained. 

A workflow that uses another workflow is referred to as a "caller" workflow. The reusable workflow is a "called" workflow. One caller workflow can use multiple called workflows. Each called workflow is referenced in a single line. The result is that the caller workflow file may contain just a few lines of YAML, but may perform a large number of tasks when it's run. When you reuse a workflow, the entire called workflow is used, just as if it was part of the caller workflow.

If you reuse a workflow from a different repository, any actions in the called workflow run as if they were part of the caller workflow. For example, if the called workflow uses `actions/checkout`, the action checks out the contents of the repository that hosts the caller workflow, not the called workflow.

When a reusable workflow is triggered by a caller workflow, the `github` context is always associated with the caller workflow. For more information about the `github` context, see "[Context and expression syntax for GitHub Actions](/actions/reference/context-and-expression-syntax-for-github-actions#github-context)."

## Access to reusable workflows

A reusable workflow can be used by another workflow if any of the following is true:

* Both workflows are in the same repository.
* The called workflow is stored in a public repository.
* The called workflow is stored in an internal repository and the settings for that repository allow it to be accessed. For more information, see "[Managing {% data variables.product.prodname_actions %} settings for a repository](/repositories/managing-your-repositorys-settings-and-features/enabling-features-for-your-repository/managing-github-actions-settings-for-a-repository#allowing-access-to-components-in-an-internal-repository)."

## Limitations

* Reusable workflows can't call other reusable workflows.
* Reusable workflows stored within a private repository can only be used by workflows within the same repository.
* Any environment variables set in an `env` context defined at the workflow level in the caller workflow are not propagated to the called workflow. For more information about the `env` context, see "[Context and expression syntax for GitHub Actions](/actions/reference/context-and-expression-syntax-for-github-actions#env-context)."
  
The following limitations will be removed when workflow reuse moves out of beta:
* You can't set the concurrency of a called workflow from the caller workflow. For more information about `jobs.<job_id>.concurrency`, see "[Workflow syntax for GitHub Actions](/actions/learn-github-actions/workflow-syntax-for-github-actions#jobsjob_idconcurrency)."
* Outputs generated by a called workflow can't be accessed by the caller workflow.

## Creating a reusable workflow

Reusable workflows are YAML-formatted files, very similar to any other workflow file. As with other workflow files, you locate reusable workflows in the `.github/workflows` directory of a repository. Subdirectories of the `workflows` directory are not supported.

For a workflow to be reusable, the values for `on` must include `workflow_call`:

```yaml
on: 
  workflow_call:
```

You can define inputs and secrets, which can be passed from the caller workflow and then used within the called workflow. The following example, from a reusable workflow, defines two inputs (called "ring" and "environment") and one secret (called "token"):

```yaml
on:
  workflow_call:
    inputs:
      ring:
        description: 'Identifier for the target deployment ring'
        default: 'ring-0'
        required: false
        type: string
      environment:
        required: false
        type: string
    secrets:
      token:
        required: false
```

For details of the syntax for defining inputs and secrets, see [on.workflow_call.inputs](/actions/reference/workflow-syntax-for-github-actions#onworkflow_callinputs) and [on.workflow_call.secrets](/actions/reference/workflow-syntax-for-github-actions#onworkflow_callsecrets).

### Example reusable workflow

This reusable workflow file named `workflow-B.yml` (we'll refer to this later) takes an input string and a secret from the caller workflow and uses them in an action.

{% raw %}
```yaml{:copy}
name: Reusable workflow example

on:
  workflow_call:
    inputs:
      username:
        required: true
        type: string
    secrets:
      token:
        required: true

jobs:
  example_job:
    name: Pass input and secrets to my-action
    runs-on: ubuntu-latest
    steps:
      - uses: ./.github/actions/my-action@v1
        with:
          username: ${{ inputs.username }}
          token: ${{ secrets.token }}      
```
{% endraw %}

## Calling a reusable workflow

You call a reusable workflow by using the `uses` keyword. Unlike when you are using actions within a workflow, you call reusable workflows directly within a job, and not from within job steps.

[`jobs.<job_id>.uses`](/actions/reference/workflow-syntax-for-github-actions#jobsjob_iduses)

You reference reusable workflow files using the syntax: 

`{owner}/{repo}/{path}/{filename}@{ref}`

You can call multiple workflows, referencing each in a separate job.

{% data reusables.actions.uses-keyword-example %}

### Passing inputs and secrets to a reusable workflow

Use the `with` keyword in a job to pass named inputs to the called workflow. Use the `secrets` keyword to pass named secrets. The inputs and secrets you pass must be defined in the called workflow. For inputs, the data type of the input value must match the type specified for that input in the called workflow (boolean, number, or string).

{% raw %}
```yaml
with:
  username: mona
secrets:
  token: ${{ secrets.TOKEN }}
```
{% endraw %}

### Supported keywords for jobs that call a reusable workflow

When you call a reusable workflow, you can only use the following keywords in the job containing the call:

* [`jobs.<job_id>.name`](/actions/reference/workflow-syntax-for-github-actions#jobsjob_idname)
* [`jobs.<job_id>.uses`](/actions/reference/workflow-syntax-for-github-actions#jobsjob_iduses)
* [`jobs.<job_id>.with`](/actions/reference/workflow-syntax-for-github-actions#jobsjob_idwith)
* [`jobs.<job_id>.with.<input_id>`](/actions/reference/workflow-syntax-for-github-actions#jobsjob_idwithinput_id)
* [`jobs.<job_id>.secrets`](/actions/reference/workflow-syntax-for-github-actions#jobsjob_idsecrets)
* [`jobs.<job_id>.secrets.<secret_id>`](/actions/reference/workflow-syntax-for-github-actions#jobsjob_idsecretssecret_id)
* [`jobs.<job_id>.needs`](/actions/reference/workflow-syntax-for-github-actions#jobsjob_idneeds)
* [`jobs.<job_id>.if`](/actions/reference/workflow-syntax-for-github-actions#jobsjob_idif)
* [`jobs.<job_id>.permissions`](/actions/reference/workflow-syntax-for-github-actions#jobsjob_idpermissions)

   {% note %}

   **Note:** 

   * If `jobs.<job_id>.permissions` is not specified in the calling job, the called workflow will have the default permissions for the `GITHUB_TOKEN`. For more information, see "[Authentication in a workflow](/actions/reference/authentication-in-a-workflow#permissions-for-the-github_token)."
   * The `GITHUB_TOKEN` permissions passed from the caller workflow can be only downgraded (not elevated) by the called workflow.

   {% endnote %}

### Example caller workflow

This workflow file calls two workflow files. The second of these, `workflow-B.yml` (shown above), is passed an input, `username`, and a secret, `token`.

{% raw %}
```yaml{:copy}
name: Call a reusable workflow

on:
  pull_request:
    branches:
      - main

jobs:
  call-workflow:
    uses: octo-org/example-repo/.github/workflows/workflow-A.yml@v1

  call-workflow-passing-data:
    uses: octo-org/example-repo/.github/workflows/workflow-B.yml@main
    with:
      username: mona
    secrets:
      token: ${{ secrets.TOKEN }}
```
{% endraw %}

## Next steps

To continue learning about {% data variables.product.prodname_actions %}, see "[Events that trigger workflows](/actions/learn-github-actions/events-that-trigger-workflows)."