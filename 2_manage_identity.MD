# Manage identity for projects, pipelines, and agents

## Introduction

- Security and access control are crucial in managing projects, pipelines, and agents within Azure DevOps.The importance of properly configuring agents, service connections, and managed identities to prevent unauthorized access, reduce security risks, and ensure appropriate access levels for team members.

# Configure a Microsoft Hosted Pool

- Microsoft-hosted agents are virtual machines maintained by Microsoft, and agent pools enable you to organize and manage these agents efficiently.

## Types of Azure DevOps Agents

- Azure DevOps provides two types of agents:

    - Microsoft-hosted agents: These are agents managed by Microsoft. They are pre-configured and ready to use, with various software already installed. Microsoft-hosted agents are updated regularly and provide a highly scalable option for running your pipelines.

    - Self-hosted agents: These are agents that you manage on your infrastructure. They provide more control over the environment and can be customized to your requirements. Self-hosted agents are ideal for scenarios where you need specific tools or configurations not available on Microsoft-hosted agents.


## Agent Pools 

- Agent pools are groups of agents that can be shared across multiple projects and pipelines. By organizing your agents into pools, you can manage their access and permissions more efficiently, ensuring that your projects and pipelines remain secure.

## Applying the Principle of least Priviledge in Azure DevOps

- To maintain a secure environment in Azure DevOps, it's essential to follow security best practices when configuring Microsoft-hosted agents and agent pools. Some key security considerations include:

    - Role-based access control (RBAC): Azure DevOps uses RBAC to manage access to agent pools. Roles such as "Administrator," "User," and "Reader" can be assigned to users or groups to grant different levels of access to agent pools. Ensure that you only grant the necessary permissions to team members to maintain a secure environment.

    - Limiting scope: When creating service connections for your pipelines, it's important to limit the scope of access. This helps prevent unauthorized access and reduces the potential impact of a security breach. Always grant the minimum set of permissions required for a specific task.

    - Securing secrets: Ensure that secrets like API keys, passwords, and tokens are securely stored using Azure Key Vault or another secure storage solution. Avoid storing secrets in plain text or in code repositories.

    - Regularly review permissions: Periodically review and update permissions for users and groups to ensure that they have the appropriate level of access based on their current roles and responsibilities.


## Configure Permissions for Microsoft Hosted Agent Pools

- To configure permissions for Microsoft-hosted agent pools:

    1. In Azure DevOps, navigate to the Organization settings.
    2. Under Pipelines, select Agent pools.
    3. Select the agent pool for which you want to configure permissions.
    4. Select the Security tab.
    5. Here, you can add users or groups and assign them specific roles like "Administrator," "User," or "Reader." Make sure to grant the least privilege necessary for each team member to perform their tasks.
    6. You can also configure permissions for specific pipelines by clicking on the Pipelines tab and selecting the pipeline for which you want to configure permissions.

## Using Azure Pipelines Agent Pool in your Pipelines

```yml

trigger:
- main

pool:
  vmImage: ubuntu-latest # This is the default if you don't specify a pool or vmImage.

steps:
- script: echo "Hello, World!"
  displayName: 'Run a simple script'

```

- Replace 'ubuntu-latest' with the name of the agent pool you want. This YAML file is a simple pipeline configuration that uses the specified agent pool to run a single script step, which prints "Hello, World!".

- With this YAML configuration, when the pipeline is triggered, it runs on an agent from the specified agent pool. Make sure that the agent pool you use has the necessary tools and capabilities to execute the tasks defined in your pipeline.

# Configure Agents for Projects

- Agents are responsible for running the tasks specified in your pipelines, and configuring them correctly is essential for ensuring smooth execution and maintaining a secure environment.

## Agent Selection in Azure Pipelines

- When configuring agents for your projects, consider the following factors:

- Resource requirements: Microsoft-hosted agents may not always have the necessary resources or software to execute specific tasks. In such cases, you might need to use self-hosted agents with custom configurations.

- Security and compliance: Some organizations have strict security or compliance requirements that necessitate the use of self-hosted agents. These agents can be deployed within your organization's infrastructure, providing better control over security and data handling.

## Configure Agents for Projects

- To configure agents for your projects:

    - Determine whether you need to use Microsoft-hosted agents or self-hosted agents based on your project requirements.

    - Select the appropriate agent pool and configure permissions.

    - Update your pipeline YAML file to specify the desired agent pool using the pool property.

```yml

trigger:
- main

pool:
  vmImage: 'ubuntu-latest'

```

- Configure permissions for your pipeline, ensuring that the principle of least privilege is followed.

# Configure Agent Identities

- Configuring self-hosted agent identities using Managed Identities provides a secure alternative to personal access tokens (PATs) for authentication and authorization in Azure DevOps.

## Understand Agent Identities

- When an Azure DevOps agent runs a job, it uses the agent identity to authenticate itself to other services and perform actions on behalf of the user. In other words, agent identities define what an agent can and can't do, effectively controlling resource access.

You can create a more secure pipeline by carefully managing agent identities and their permissions. For example, you can limit the potential damage from a compromised agent by ensuring that its identity only has the minimum necessary permissions.

## Importance of Agent Identities

- In Azure DevOps, Agent Identity is fundamental to ensuring the secure and efficient execution of tasks. Agents are the building blocks that carry out the work specified in your pipelines. They run the tasks defined in a stage of your pipeline, and they do so with a specific identity.

- This identity is crucial for several reasons:

    - Security: The agent's identity determines what resources the agent can access during operations. It controls permissions and access rights to different parts of your Azure DevOps environment and other linked systems.

    - Auditing: Agent Identity is a key component in tracking and auditing the actions performed in your DevOps environment. You can trace back operations to specific agents, making it easier to understand the flow of tasks and identify any issues.

    - Resource Management: The agent's identity can be used to determine which tasks it should perform. You can allocate certain tasks to specific agents, improving efficiency and resource use.

- You can configure agent identities in Azure DevOps using a personal access token (PAT).

# Configure the Scope of Service Connection

- Service connections enable your agents to interact with external services, such as source control repositories, cloud services, or package registries.

## Understanding Service Connection Scope 

- The scope of a service connection determines the level of access and permissions the connection has within the external service. By properly configuring the scope, you can ensure that your pipelines have the necessary access to complete their tasks, while minimizing the risk of unauthorized access and potential security breaches.

### Best Practices for configuring Service connection Scope

- Least privilege: Grant the minimum set of permissions required for the specific tasks the service connection will be used for. Avoid giving broader access than necessary.

- Limit the number of service connections: Reduce the number of service connections to minimize the potential attack surface. Reuse service connections when appropriate, but ensure that the access levels are suitable for all the pipelines using the connection.

- Regularly review and update permissions: Periodically review and update the permissions and scope of your service connections to ensure they remain appropriate and secure.

- Monitor usage: Keep track of service connection usage across your pipelines to identify any unusual activity or unauthorized access.

- Use managed identities when possible: Managed identities provide a secure way to authenticate with Azure services without having to store credentials in your pipeline. Use managed identities for Azure resources whenever possible.

## Configure the Scope of a Service Connection

1. Sign in to your Azure DevOps portal, navigate to your project, and then go to 'Project settings' and 'Service connections.'

2. Click on 'New service connection' and choose the type of connection you want to create, for example, 'Azure Resource Manager.'

3. In the 'Authentication method' section, select 'Service principal (automatic).' This option will automatically create and manage a service principal in your Microsoft Entra ID, which will be used for authentication.

4. Configure the scope of the service connection by selecting the appropriate subscription, resource group, or resource. The scope determines which resources the service connection has access to. Limit the scope as much as possible to reduce potential security risks.
    - Subscription: Grants access to all resources within the specified Azure subscription.
    - Resource Group: Grants access only to resources within a specific resource group in the Azure subscription.
    - Resource: Grants access to a single resource, such as a storage account or a web app, within the Azure subscription.

5. Provide a name and description for the service connection, and then click 'Save.'

6. Once the service connection is created, open your new service connection, click on the ellipsis (three dots) next to the Edit button and select 'Security.'

7. Review the 'Roles' section to ensure that the service principal has been granted the least privilege necessary for its purpose. If needed, modify the roles or add custom roles to further limit the access, and actions it can perform.

8. Save your changes.

# Understand and convert to a Managed Identity

- Managed Identity and Service Principals are two ways to authenticate and authorize access to Azure resources. Both provide a secure way to grant your Azure Pipelines access to Azure services without having to store credentials in the pipeline configuration.

## Benefits of using managed identities

- Managed identities provide several benefits when used in Azure DevOps:

    - Improved security: They eliminate the need to store sensitive credentials in your pipeline, reducing the risk of credential exposure.
    - Simplified management: You can manage identities through Azure, reducing the overhead of identity management in Azure DevOps.
    - Fine-grained access control: Managed identities allow you to define the exact permissions and access levels required for your pipelines and agents.
    - Auditing and monitoring: Authentication and authorization events are logged, making it easier to track access and detect potential security threats.

## Create a managed identity and add it to Azure DevOps

- Create your Managed Identity in Azure and assign the appropriate permissions to it. Then, configure the agent to use the Managed Identity.

1. In the Azure portal, navigate to the Managed Identities resource.

2. Click + Create and configure the Managed Identity as appropriate.

3. Assign the necessary permissions to the Managed Identity, following the principle of least privilege.

4. In Azure DevOps, navigate to your organization and select Organization settings.

5. Click on Users under General.

6. Add the Managed Identity as a user.

7. Choose the appropriate access level for the Managed Identity and projects.

8. Click to Add.

9. Click on Security under Pipelines.

10. Click to Add a new security group or choose an existing one.

11. Add the Managed Identity to the security group.

## Convert to Managed Identity

- To convert an existing service connection to use a managed identity, follow these steps:

1. Navigate to your Azure DevOps project, go to 'Project settings' and then 'Service connections.'
2. Identify the service connection you want to convert to use a managed identity. Click on Edit and make a note of its settings, such as the scope and roles assigned to it. Also, make sure you know the pipelines using this service connection.
3. Delete the existing service connection by clicking on the ellipsis (three dots) next to the service connection and selecting 'Delete.'
4. Create a new service connection by clicking on 'New service connection' and selecting 'Azure Resource Manager' as the connection type (or choose the connection you need).
5. In the 'Authentication method' section, select 'Managed identity.'
6. Configure the scope and permissions for the managed identity using the settings from your previous service connection. This ensures that the managed identity has the same access and functionality as the original service connection.
7. Provide a name and description for the new service connection, and then click 'Save.'
8. Update your pipeline to use the new service connection with the managed identity. To do this, locate the service connection reference in your pipeline YAML file and replace the old service connection name with the new one (if it has changed).
9. Save and commit the updated pipeline YAML file.