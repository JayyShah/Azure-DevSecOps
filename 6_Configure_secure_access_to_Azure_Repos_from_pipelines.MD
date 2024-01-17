# Configure secure access to Azure Repos from pipelines

## Introduction

- It's essential to have a secure environment where pipelines can access resources, such as packages, secrets, and services, without exposing sensitive information. 

# Configure Pipeline access to packages

- Configure pipeline access to packages stored in Azure Artifacts. It involves creating and managing feed permissions, which allow you to control who can access and manage your packages. By controlling access to these packages, you can ensure that only authorized users can access and use the packages in your projects.

## Configure Pipeline access to packages in YAML

1. Navigate to your Azure DevOps organization and select the project that contains the Azure Artifacts repository you want to configure.

2. In the left-side menu, select Artifacts.

3. If you don't have any feed, Create a feed.

4. In the Artifacts menu, select Feed Settings.

5. Click on the Permissions tab.

6. Add users or groups to the repository.

7. Select the permissions you want to assign to each user or group (for example, Owner, Reader, Contributor, or Collaborator).

![packages_in_yaml](/source/image-15.png)

8. Select Add users/groups, and then add your build identity as a Contributor. The project-level build identity is named as follows: [Project name] Build Service ([Organization name]). Example: Implement security through a pipeline using DevOps Build Service (contoso).

9. Save Your changes

## Consume Packages from Azure Artifacts in a Pipeline 

- In Azure Pipelines, you can use the classic editor or the YAML tasks to publish your NuGet or other packages within your pipeline to your Azure Artifacts feed or public registries such as nuget.org.

```yaml

steps:
    - task: NuGetCommand@2
      inputs:
        command: 'restore'
        restoreSolution: '**/*.sln'
        feedsToUse: 'select'
        vstsFeed: 'SecurePipelineFeed'

```

- Replace your Azure Artifacts feed name with the name of your Azure Artifacts feed and your solution file.sln with the name of your solution file.

------------------------------------------------------------------------------------------------------------------

# Configure Pipeline access to credential Secret

## Use variables to store values or encrypted secrets

1. Open your Azure DevOps project and navigate to the Pipelines section.

2. Click on Pipelines in the left-hand menu.

3. Open your pipeline, or create a new one.

4. Click the Edit button in the right-top corner to edit your pipeline.

5. Click the Variables button.

6. Click the New variable button to create a new variable.

7. Enter the name and value for your secret.

8. Check the "Keep this value secret" checkbox to encrypt your secret.

9. (Optional) Check the "Let users override this value when running this pipeline" checkbox to allow users to override the value of your variable at queue time.

10. Click the OK button to save your variable.

11. Click the Save button to save your pipeline.


## Create a Secret in Azure DevOps variable Group

1. Open your Azure DevOps project and navigate to the Pipelines section.

2. Click on Library in the left-hand menu.

3. Open your variable group, or create a new one.

4. Click the Add button to add a new variable.

5. Give your variable a name (for example "<name> Secret Key").

6. Enter the value for your secret in the Value field.

7. Click the Save button to save your variable group.

## Allow variable groups use in your pipeline

1. Open your Variable Group.

2. Click on the Pipeline permissions button.

3. Add the pipelines that will use this Variable Group.

4. Click the Save button to save your Variable Group.

## Access secrets within your pipeline 

1. Open your pipeline YAML file.

2. Add the following code to the top of your YAML file:

```yaml
variables:
  - group: <group_name>
```

3. Use the following syntax to access your secrets within your pipeline

```yaml
$(<group_name> Secret Key)
```

4. If you want to use your pipeline variables, you can use the following:

```yaml
$(New Credential Secret)
```

5. Save your YAML file

- Secret variables are encrypted at rest with a 2048-bit RSA key. Secrets are available on the agent for tasks and scripts to use. Be careful about who has access to alter your pipeline.

- You must decide whether to use the Variable Groups or the pipeline UI variables. The advantage of using the Variable Groups is that you can use the same variables in multiple pipelines. The advantage of using the pipeline UI variables is that you can override the variable's value at queue time.

------------------------------------------------------------------------------------------------------------------

# Configure pipeline access to secrets for services

- Securing access to services is essential when working with pipelines in Azure DevOps. Service connections allow you to store your pipelines' credentials to access external resources, such as databases, web APIs, and other systems.

## Create A Service Connection

1. Go to your Azure DevOps project.

2. Navigate to the Project settings.

3. Click on Service connections under Pipelines.

4. Click on New service connection.

5. Select the type of service connection you want to create (for example, Azure Service Bus, Kubernetes, Apple App Store or other).

6. Enter the required information for the service connection.

7. Click on Save.

![Service_connection](/source/image-16.png)

## Store the service connection using Variables

1. Go to your pipeline definition.

2. Click on Edit.

3. Click on Variables.

4. Create a new variable with the name that represents the service connection (for example, service_bus_connection).

5. Enter the value of the service connection.

6. Check the checkbox "Keep this value secret" to encrypt the variable.

7. Click on Save.

- *Unlike normal variables, they are not automatically decrypted into script environment variables. You need to map secret variables explicitly.*

## Use Service connection in YAML

```yaml

steps:
    - powershell: |
        Write-Host "Using the mapped env var for this task works and is recommended: $env:MY_MAPPED_ENV_VAR"
      env:
        MY_MAPPED_ENV_VAR: $(service_bus_connection) # the recommended way to map to an env variable
    
    - task: PublishToAzureServiceBus@1
      inputs:
        azureSubscription: $(service_bus_connection)
        messageBody: '"hello world!"'
        signPayload: false
        waitForCompletion: true

```

------------------------------------------------------------------------------------------------------------------

# Use Azure Key Vault to secure secrets

- Securing access to sensitive information, such as passwords and API keys, is essential to DevOps.

## Create an Azure Key Vault and Service Principal

- The first step in securing access to credential secrets is to store them in Azure Key Vault. This service allows you to store and manage secrets, keys, and certificates securely and provides you with the ability to control access to these secrets.

    1. To create an Azure Key Vault, go to the Azure portal and click on the "Create a resource" button.

    2. Search and select the "Key Vault" option, click create and then fill out the required information to create a new vault.

    3. Create a new Service Principal in Microsoft Entra ID to grant access to the Key Vault.

    4. Assign the service Principal to the Key Vault 

    5. *The service principal that you created will need to have Secret permissions access ("Get, List") to the Key Vault. If the service principal does not have access to the Key Vault, you will see an error message when you try to link the Variable Group to the Key Vault.*

    6. Once you've created your Key Vault, you need to store the secrets that you want to use in your pipeline. You can create secrets directly in your Key Vault, or from the Azure DevOps.


## Create Secrets in Azure Key Vault

1. In the Azure portal, go to the Azure Key Vault that you created in step 1.

2. Open the "Secrets" option and click on the "Generate/Import" button.

3. From the Azure Key Vault you can create manual secrets, or upload a certificate.

4. Once you've created your secret, you can use it in your pipeline.

## Create Secrets in Azure DevOps

1. In Azure DevOps, go to the Azure DevOps organization and project that you want to use.

2. Click on Library and then open your Variable group.

3. Toggle the "Link to Key Vault" option and select the Key Vault and secret that you want to use.

4. Click "Authorize" to enable Azure Pipelines to set these permissions or manage secret permissions in the Azure portal.

5. When authorization is complete, click Add under Variables to add the secret from your linked Key Vault to your Variable group.

6. Select the secret that you want to use in your pipeline and click OK to add it to your Variable group.

7. Save the Variable group.

## Grant Azure DevOps Access to Key Vault

- Now that you've stored your secrets in Azure Key Vault, you need to grant Azure DevOps access to the Key Vault so that your pipeline can retrieve the secrets.

    1. In the Azure portal, go to the Azure DevOps organization and project.
    2. Go to the "Project Settings" and then "Service connections".
    3. Click the "New service connection" button, and then select "Azure Resource Manager".
    4. Fill out the required information to create the connection, including the name of the Key Vault and the secrets that you want to use in your pipeline.
    5. After you've created the service connection, you'll need to grant Azure DevOps access to the Key Vault. To do this, go to the Azure Key Vault and click on the "Access policies" option.
    6. Add a new policy, and then select the Azure DevOps service connection that you created in step 4.
    7. Assign the "Get" and "List" permissions to the service connection.

## Use Secrets in Your Pipeline

```yaml

steps:
- task: AzureKeyVault@2
  inputs:
    azureSubscription: '<your_azure_subscription_name>'
    KeyVaultName: '<your_key_vault_name>'
    SecretsFilter: |
      <secret_name>

```

- Replace <your_azure_subscription_name> with the name of your Azure subscription, <your_key_vault_name> with the name of your Key Vault, and <secret_name> with the name of the secret that you want to use in your pipeline.

- Save the pipeline definition, and then run the pipeline. The secret should now be available in your pipeline, and you can use it as needed.

------------------------------------------------------------------------------------------------------------------

# Explore and Secure Log Files

## Access Log files in Azure Pipelines

- The job details page provides detailed information about the pipeline run, including the tasks executed, their status, and any output generated.

- You can access the logs for a specific pipeline run by following these steps:

    1. In your Azure DevOps project, navigate to the pipelines section, under pipelines menu.
    2. Select the pipeline for which you want to view the logs.
    3. Click on a specific run of the pipeline.
    4. In the run details page, find the Jobs tab and click on the job for which you want to view the logs.

- You can also access the logs for a specific task by clicking on the task name, or download logs for the entire job by clicking on the "Download logs" link.

## Secure Log Files

- Securing log files in Azure Pipelines is crucial to ensure that sensitive information, such as secrets and credentials, isn't displayed in plain text. Azure Pipelines attempts to scrub secrets from logs wherever possible. This filtering is on a best-effort basis and can't catch every way that secrets can be leaked. Avoid echoing secrets to the console, using them in command line parameters, or logging them to files.

- There are many ways to secure log files in Azure Pipelines, including:

    - By using the issecret=true command in a script or task, you can ensure that specific values aren't displayed in the logs. When issecret is set to true, the variable's value is saved as secret and masked out from the log. Secret variables aren't passed into tasks as environment variables and must instead be passed as inputs.

```yaml

steps:
    - pwsh: |
        Write-Host "##vso[task.setvariable variable=nonSecretVar;]Now you can see me!"
        Write-Host "##vso[task.setvariable variable=secretVar;issecret=true]Now you don't!"
      name: SetVariables
```

- Read the variables 

```yaml

- pwsh: |
        Write-Host "The magician says: $env:NONSECRETVAR = Not a secret."
        Write-Host "The magician says: $env:SECRETVAR = Yes, it's hidden, can't you see it? =)"
        Write-Host "The magician says: $(secretVar) = It's encrypted."

```

- By using the isoutput=false command in a script or task, the variable's value is hidden out from the log.

```yaml

steps:
    - pwsh: |
        Write-Host "##vso[task.setvariable variable=outputVarTrue;isoutput=true]No, it's not a secret!"
        Write-Host "##vso[task.setvariable variable=outputVarFalse;isoutput=false]Yes, it's a secret!"
      name: SetVariables

```

- Read the Variables 

```yaml

- pwsh: |
        Write-Host "Hidden out from the log: $env:SETVARIABLES_OUTPUTVARTRUE"
        Write-Host "Hidden out from the log: $(SetVariables.outputVarTrue)"
        Write-Host "Hidden out from the log: $env:SETVARIABLES_OUTPUTVARFALSE = Yes, it's hidden."

```

- A few other ways to secure log files in Azure Pipelines include:

    - Use Secure Files type to upload a file to Azure Pipelines and then download it to the pipeline using the "Download secure file" task. This is useful for uploading certificates and other files that are required by tasks in the pipeline, but shouldn't be displayed in plain text.

    - Azure Pipelines will automatically delete log files after a certain amount of time, or by the retention settings. This is useful for ensuring that secrets and other sensitive information aren't stored indefinitely.

    - Azure Key Vault integration is another way to secure your secrets from log files.
    
    - Secure files, Secret variables, and Variable groups are another way to secure log files in Azure Pipelines.