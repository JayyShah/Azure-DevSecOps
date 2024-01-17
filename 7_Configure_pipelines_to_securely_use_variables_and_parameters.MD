# Configure pipelines to securely use variables and parameters

## Introduction

- Using variables and parameters is a crucial part of Azure DevOps. They allow you to store and manage values that can be used across your pipelines, making managing your resources and configurations easier. However, if not used securely, they can pose a significant risk to your organization's security and confidentiality.

# Ensure Parameter and variable types

- Parameters and variables are essential components for customizing pipeline behavior. These enable passing runtime values to pipeline tasks and scripts and defining variables for use across different pipeline stages. When using parameters and variables in pipelines, it's crucial to define their data types correctly to avoid any unexpected behavior that could affect the project's security or pipeline execution.

## Parameter and Variable Type

- Parameters and variables are critical to pipeline customization, as they allow for flexibility and enable pipeline scripts to be more dynamic. You can use them to store values such as connection strings, environment variables, and other sensitive data. Properly defining their data types is also essential to avoid unexpected errors or vulnerabilities arising from incorrect usage.

- Add parameter or Variable to your pipeline

```yaml

parameters:
- name: myParameterName
  type: myDataType
  default: myDefaultValue

variables:
- name: myReadOnlyVar
  value: myValue
  readonly: true

```

- Replace myParameterName, myDataType, myDefaultValue, myReadOnlyVar, and myValue with your desired values.

## Use the correct data types when defining your parameters

- Azure Pipelines supports the following data types:

    - string
    - boolean
    - number
    - object
    - step

- Unlike variables, pipeline parameters can't be changed by a pipeline while it's running. Parameters have data types such as number and string, and they can be restricted to a subset of values. Restricting the parameters is useful when a user-configurable part of the pipeline should take a value only from a constrained list. The setup ensures that the pipeline won't take arbitrary data.


## Define secrets as secret variables or as a part of a variable group

```yaml

variables:
 - name: <name>
   value: myValue

# You can define variable groups to reuse variables across pipelines
# and to manage sensitive data centrally.

variables:
- group: <Group_name>
- name: ConnectionStrings.CatalogConnection
  value: '$(CatalogConnectionToken)'

```

## Use the ReadOnly property to ensure that variables aren't changed by a pipeline while it's running

- It's useful when you want to ensure that a variable isn't changed by a pipeline while it's running. For example, you can use this property to ensure that a variable isn't changed by a pipeline while it's running.

```yaml

variables:
- name: <Name>
  value: myValue
  readonly: true

```

# Identify and restrict insecure use fo parameters and variables

- In Azure Pipelines, parameters and variables can be used to pass runtime values to pipeline tasks and scripts and to define variables that can be used across different pipeline stages. However, if not correctly secured, parameters and variables can become a security vulnerability, as they can store sensitive information such as connection strings, API keys, and other credentials.

- Secure usage of parameters and variables is essential to ensure the security of your pipeline and the projects it supports. Insecure parameters and variables can lead to data breaches, unauthorized access, and other security risks. They can also lead to unexpected behavior or errors that can impact the reliability and stability of your pipeline.

## Identify insecure use of parameters and variables

- Here are some steps to identify insecure use of parameters and variables in Azure Pipelines:

    - Check your pipeline YAML file for any parameters or variables storing sensitive information, such as connection strings or credentials.
    - Check that the values of these parameters or variables aren't hard-coded in the pipeline tasks or script used by your tasks, for example, bash, PowerShell, etc. Instead, they should be defined as secure pipeline inputs, such as a secure file or variable group.
    - Use the Azure DevOps Pipeline Audit log to monitor the usage of parameters and variables in your pipeline and identify any potential security risks or vulnerabilities.
    - Check your template files for any parameters or variables storing sensitive information.
    - Check your repository for any parameters or variables storing sensitive information, for example, appconfig.json, appsettings.json, secrets.json, etc.

## Restrict insecure use of parameters and variables

- Here are some steps to restrict the insecure use of parameters and variables in Azure Pipelines:

    - Define your parameters and variables as secure pipeline inputs, such as a secure file, secret variables, or variable group.
    - Use Azure Key Vault to store sensitive data, such as connection strings, API keys, or certificates and then reference them in your pipeline script.
    - Link your Azure Key Vault to your Azure DevOps organization and then use the Azure Key Vault task to retrieve the secrets from your Azure Key Vault and use them in your pipeline.
    - Use service connections with service principal authentication.
    - Restrict access to sensitive data by setting appropriate permissions and access control policies. For example, limit access to pipeline variables and parameters to specific users or groups.

# Move parameters into a YAML file

- In Azure Pipelines, you can use YAML files to define your pipeline's configuration as code. Parameters in YAML files help you to reuse the pipeline and keep the code clean. 

- Benefits of using parameters in YAML files:

    - It helps in reusing the pipeline configuration.
    - It enables you to define the pipeline as code, allowing you to track pipeline changes over time.
    - It keeps the code clean and organized.

- You can enforce that a pipeline extends from a particular template to increase security.

- Create a file named parameters.yml in your repository, or other repository specific for your templates.

```yaml

# File: parameters.yml
parameters:
- name: buildSteps # the name of the parameter is buildSteps
  type: stepList # data type is StepList
  default: [] # default value of buildSteps
stages:
- stage: secure_buildstage
  pool:
    vmImage: windows-latest
  jobs:
  - job: secure_buildjob
    steps:
    - script: echo This happens before code 
      displayName: 'Base: Pre-build'
    - script: echo Building
      displayName: 'Base: Build'

    - ${{ each step in parameters.buildSteps }}:
      - ${{ each pair in step }}:
          ${{ if ne(pair.value, 'CmdLine@2') }}:
            ${{ pair.key }}: ${{ pair.value }}       
          ${{ if eq(pair.value, 'CmdLine@2') }}: 
            # Step is rejected by raising a YAML syntax error: Unexpected value 'CmdLine@2'
            '${{ pair.value }}': error         

    - script: echo This happens after code
      displayName: 'Base: Signing'

```

- Create a file named azure-pipelines.yml, and reference the parameters.yml file:

```yaml

# File: azure-pipelines.yml
trigger:
- main

extends:
  template: parameters.yml
  parameters:
    buildSteps:  
      - bash: echo Test #Passes
        displayName: succeed
      - bash: echo "Test"
        displayName: succeed
      # Step is rejected by raising a YAML syntax error: Unexpected value 'CmdLine@2'
      - task: CmdLine@2
        inputs:
          script: echo "Script Test"
      # Step is rejected by raising a YAML syntax error: Unexpected value 'CmdLine@2'
      - script: echo "Script Test"

```

- The pipeline is extended from the template parameters.yml. The parameter buildSteps is passed to the template. The template checks if the value of the parameter buildSteps is a stepList type. If it is, the pipeline continues. If it isn't, the pipeline fails.

# Limit the Queue time variables

- In some scenarios, you may want to allow certain variables to be changed when you run a YAML pipeline. For example, you may have a variable that controls the target environment, such as dev, test, or prod. By allowing this variable to be set at queue time, you can reuse the same pipeline definition for different deployments without modifying the YAML file.

- However, allowing variables to be set at queue time can also introduce some risks. For example, someone could change a variable value to access a sensitive resource, bypass a security check, or alter the behavior of the pipeline in an unexpected way. To prevent these risks, you should limit the variables that can be set at queue time to only those that are necessary and safe.

- The setting is designed to work at the organization and project levels.

- In Azure DevOps, you have a setting to limit variables that can be set at queue time. With this setting enabled, only those variables that are explicitly marked as "Settable at queue time" can be set. In other words, you can set any variables at queue time unless this setting is enabled.

    - Organization: When the setting is on, it enforces that, for all pipelines in all projects in the organization, only those variables that are explicitly marked as "Settable at queue time" can be set. When the setting is off, each project can choose to restrict variables set at queue time. The setting is a toggle under Organization Settings > Pipelines > Settings. Only Project Collection Administrators can enable or disable it.

    ![queue](/source/image-17.png)

    - Project: When the setting is on, it enforces that, for all pipelines in the project, only those variables that are explicitly marked as "Settable at queue time" can be set. If the setting is on at the organization level, it is on for all projects and can't be turned off. The setting is a toggle under Project Settings > Pipelines > Settings. Only Project Administrators can enable or disable it.

# Validate Mandatory Variables

- In YAML pipelines, variables store and retrieve values during pipeline runs. Ensuring all the required variables are present and set correctly is essential to ensure pipeline security.

## Define and Validate mandatory variables

- One way to validate mandatory variables is by using the assert expression function in YAML to validate required variables. The assert function allows us to check if a condition is true or false and return an error message if the condition is false. We can use the assert function to validate that the mandatory variables are present and set correctly.

```yaml

variables:
- name: myVariable
  value: ''

steps:
- script: echo 'Validate required variable!'
  condition: eq(variables['myVariable'], 'myRequiredValue')

```

- The variable 'myVariable' is set to an empty string in this example. The condition expression checks if the variable is equal to 'myRequiredValue'. The script task won't be executed if the variable isn't set to this value.

- output: Evaluating: eq(variables['myVariable'], 'myRequiredValue') Expanded: eq('', 'myRequiredValue') Result: False

- You can also configure your pipeline to fail if the variable isn't set correctly (false).