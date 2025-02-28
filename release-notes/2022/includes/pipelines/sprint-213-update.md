---
author: ckanyika
ms.author: ckanyika
ms.date: 12/8/2022
ms.topic: include
---
### Container Registry service connections can now use Azure Managed Identities

You can use a System-assigned Managed Identity when creating Docker Registry service connections for Azure Container Registry. This allows you to access Azure Container Registry using a Managed Identity associated with a self-hosted Azure Pipelines agent, eliminating the need to manage credentials.

> [!div class="mx-imgBorder"]
> ![New Docker Registry Service Connection for Changes to Approvals](../../media/213-pipelines-05.png)

> [!NOTE]
> The Managed Identity used to access Azure Container Registry will need the appropriate Azure Role Based Access Control (RBAC) assignment, e.g. AcrPull or AcrPush role.

### Audit log events related to pipeline permissions

When you restrict pipeline permissions of a protected resource, such as a service connection, the associated Audit Event Log now correctly states that the resource was *successfully _unauthorized_* for its project.

> [!div class="mx-imgBorder"]
> ![Pipeline Permissions for Changes to Approvals](../../media/213-pipelines-01.png)

> [!div class="mx-imgBorder"]
> ![Successfully Authorized for Changes to Approvals](../../media/213-pipelines-02.png)

### Ensure your organization only uses YAML pipelines

Azure DevOps now lets you ensure your organization only uses YAML pipelines, by disabling the creation of classic build pipelines, classic release pipelines, task groups, and deployment groups. Your existing classic pipelines will continue to run, and you'll be able to edit them, but you won't be able to create new ones.

You can disable creation of classic pipelines at the organization level or project-level, by turning on the corresponding toggles. The toggles can be found in **Project / Organization Settings -> Pipelines -> Settings**. 

> [!div class="mx-imgBorder"]
> ![Disable Creation Of Classic Build and Classic Pipeline for Changes to Approvals](../../media/213-pipelines-03.png)

The toggles state is off by default, and you will need admin rights to change the state. If the toggle is on at the organization-level, the disabling is enforced for all projects. Otherwise, each project is free to choose whether to enforce or not the disablement.

When disabling the creation of classic pipelines is enforced, REST APIs related to creating classic pipelines, task groups, and deployment groups will fail. REST APIs that create YAML pipelines will work.

Disabling creation of classic pipelines is opt-in for existing organizations. For new organizations, it is opt-in for the time being.

### Service hook for job state change

Service hooks allow you to react in response to events related to state changes in your pipeline. Until now, you could configure service hooks for pipeline run and stage state changes. 

With this update, you can configure service hooks that run when the state of a job in your pipeline changes. The payload structure of the new service hook is similar to the following example.

```json
{
    "subscriptionId": "8d91ad83-1db5-4d43-8c5a-9bb2239644b1",
    "notificationId": 29,
    "id": "fcad4962-f3a6-4fbf-9653-2058c304503f",
    "eventType": "ms.vss-pipelines.job-state-changed-event",
    "publisherId": "pipelines",
    "message":
    {
        "text": "Run 20221121.5 stage Build job Compile succeeded.",
        "html": "Run 20221121.5 stage Build job <a href=\"https://dev.azure.com/fabrikamfiber/fabrikamfiber-viva/_build/results?buildId=2710088\">Compile</a> succeeded.",
        "markdown": "Run 20221121.5 stage Build job [Compile](https://dev.azure.com/fabrikamfiber/fabrikamfiber-viva/_build/results?buildId=2710088) succeeded."
    },
    "detailedMessage":
    {
        "text": "Run 20221121.5 stage Build job Compile succeeded.",
        "html": "Run 20221121.5 stage Build job <a href=\"https://dev.azure.com/fabrikamfiber/fabrikamfiber-viva/_build/results?buildId=2710088\">Compile</a> succeeded.",
        "markdown": "Run 20221121.5 stage Build job [Compile](https://dev.azure.com/fabrikamfiber/fabrikamfiber-viva/_build/results?buildId=2710088) succeeded."
    },
    "resource":
    {
        "job":
        {
            "_links":
            {
                "web":
                {
                    "href": "https://dev.azure.com/fabrikamfiber/fabrikamfiber-viva/_build/results?buildId=2710088"
                },
                "pipeline.web":
                {
                    "href": "https://dev.azure.com/fabrikamfiber/fabrikamfiber-viva/_build/definition?definitionId=4647"
                }
            },
            "id": "e87e3d16-29b0-5003-7d86-82b704b96244",
            "name": "Compile",
            "state": "completed",
            "result": "succeeded",
            "startTime": "2022-11-21T16:10:28.49Z",
            "finishTime": "2022-11-21T16:10:53.66Z"
        },
        "stage": { ... },
        "run": { ... },
        "pipeline": { ... },
        "repositories": [ ... ]
    },
    "resourceVersion": "5.1-preview.1",
    "createdDate": "2022-11-21T16:11:02.9207334Z"
}
```

Run, stage, and job state change service hook events now contain a `repository` property that shows source code version information associated to the pipeline run. For example,

```json
"repositories":
[
    {
        "type": "Git",
        "change":
        {
            "author":
            {
                "name": "Fabrikam John",
                "email": "john@fabrikamfiber.com",
                "date": "2022-11-11T15:09:21Z"
            },
            "committer":
            {
                "name": "Fabrikam John",
                "email": "john@fabrikamfiber.com",
                "date": "2022-11-11T15:09:21Z"
            },
            "message": "Added Viva support"
        },
        "url": "https://fabrikamfiber@dev.azure.com/fabrikamfiber/fabrikamfiber-viva/_git/fabrikamfiber"
    },
    {
        "type": "GitHub",
        "change":
        {
            "author":
            {
                "name": "Fabrikam John",
                "email": "john@github.com",
                "date": "2022-08-11T15:05:20Z"
            },
            "committer":
            {
                "name": "Fabrikam John",
                "email": "john@github.com",
                "date": "2022-08-11T15:05:20Z"
            },
            "message": "Added Viva open source REST API library"
        },
        "url": "https://api.github.com/repos/FabrikamFiber/Viva"
    }
]
```

### New PAT scope required to update pipeline General Settings 
Invoking the [General Settings - Update](/azure/devops/build/general-settings/update?view=azure-devops-rest-7.1&preserve-view=true) REST API now requires a PAT with scope **Project and Team -> Read & Write**. 

> [!div class="mx-imgBorder"]
> ![Project and Team](../../media/213-pipelines-07.png)

### Fine-grained access management for agent pools

Agent pools allow you to specify and manage the machines on which your pipelines run. 

Previously, if you used a custom agent pool, managing which pipelines can access it was coarse-grained. You could allow all pipelines to use it, or you could require each pipeline ask for permission. Unfortunately, once you granted a pipeline access permission to an agent pool, you could not revoke it using the pipelines UI.

Azure Pipelines now provides a fine-grained access management for agent pools. The experience is similar to the one for managing pipeline permissions for Service Connections.

> [!div class="mx-imgBorder"]
> ![FabrikamFiber Agent Pool for Changes to Approvals](../../media/213-pipelines-06.png)

### Prevent granting all pipelines access to protected resources

When you create a protected resource such as a service connection or an environment, you have the option to select the **Grant access permission to all pipelines** checkbox. Until now, this option was checked by default. 

While this makes it easier for pipelines to use new protected resources, the reverse is that it favors accidentally granting too many pipelines the right to access the resource.

To promote a secure-by-default choice, Azure DevOps now leaves the checkbox unticked.

> [!div class="mx-imgBorder"]
> ![New Generic Service Connection for Changes to Approvals](../../media/213-pipelines-04.png)

### Improved security when building pull requests from forked GitHub repositories

You can use Azure DevOps to build and test your public GitHub repository. Having a public GitHub repository lets you  collaborate with developers across the world, but comes with [security concerns related to building pull requests (PRs) from forked repos](/azure/devops/pipelines/security/repos?view=azure-devops#forks&preserve-view=true).

To prevent PRs from forked GitHub repositories from making undesired changes to your repositories, Azure DevOps now limits the GitHub access token to have read-only scope.

### Macos-latest label will point to macos-12 image

The macos-12 Monterey image is ready to be the default version for the “macos-latest" label in Azure Pipelines Microsoft-hosted agents. Until now, this label pointed to macos-11 Big Sur agents. 


For a full list of differences between macos-12 and macos-11, visit the [GitHub issue](https://github.com/actions/runner-images/issues/6384). For a full list of software installed on the image, check [here](https://github.com/actions/runner-images/blob/main/images/macos/macos-12-Readme.md).

### Ubuntu-latest label will point to ubuntu-22.04 image

The ubuntu-22.04 image is ready to be the default version for the **ubuntu-latest** label in Azure Pipelines Microsoft-hosted agents. Until now, this label pointed to ubuntu-20.04 agents. 

For a full list of differences between ubuntu-22.04 and ubuntu-20.04, visit the [GitHub issue](https://github.com/actions/runner-images/issues/6399). For a full list of software installed on the image, check [here](https://github.com/actions/runner-images/blob/main/images/linux/Ubuntu2204-Readme.md).