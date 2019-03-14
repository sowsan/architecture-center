<!-- TEMPLATE FILE - DO NOT ADD METADATA -->
<!-- markdownlint-disable MD002 MD041 -->
> [!NOTE]
>In the event of changes to your business requirements, Azure Management Groups allow you to easily reorganize your management hierarchy and subscription group assignments. However, keep in mind that Azure Policies and RBAC assignments applied to a management group are inherited by all subscriptions underneath that group in the hierarchy. If you plan to reassign subscriptions between management groups, make sure that you are aware of any policy and RBAC changes that may result. See the [Azure Management Groups documentation](/azure/governance/management-groups/) for more information.

### Governance of resources

A set of global Azure Policies and RBAC assignments will provide a baseline level of governance enforcement. 

Azure provides several built-in policies and role assignments that you can assign to any management group, subscription, or resource group. However, to meet the Cloud Governance team's policy requirements, implementation of the governance MVP requires completing the following tasks:

1. Define required custom Azure Polices at the root of the Management Group hierarchy.
2. Create an Azure Blueprint definition using these custom policies and RBAC settings required by the governance MVP.
3. Apply policies and configuration globally by assigning the Blueprint to all subscriptions.

#### Create custom policies

Custom policy definitions are saved to either a management group or a subscription and are inherited through the management group hierarchy. If a policy definition's save location is a management group, that custom policy is available to assign to any of that group's child management groups or subscriptions.

Since the policies required to support the governance MVP are meant to apply to all current subscriptions, the following custom policy definitions will be created in the root management group:

1. Restrict the list available RBAC assignments to a set of built-in Azure roles authorized by your Cloud Governance team.
2. Require the use of the following tags on all resources: Department/Billing Unit, Geography, Data Classification, Criticality, SLA, Environment, Application Archetype, Application, and Application Owner.
3. Require that the Application tag for resources should match the name of the relevant Resource Group.

For information on defining custom policies see the [Azure Policy documentation](/azure/governance/policy/tutorials/create-custom-policy-definition). For guidance and examples of custom policies, consult the [Azure Policy samples site](/azure/governance/policy/samples/) and the associated [GitHub repository](https://github.com/Azure/azure-policy).

#### Assign Azure Policy and RBAC roles using Azure Blueprints

Although the policy requirements defined in this governance MVP apply to all current subscriptions, it's very likely that future deployments will require exceptions or alternative policies. As a result, assigning policy using management groups, with all child subscriptions inheriting these assignments, may not be flexible enough to support these scenarios. 

[Azure Blueprints](/azure/governance/blueprints/overview) allow the consistent assignment of policy and roles, application of Resource Manager templates, and deployment of Resource Groups across multiple subscriptions. As with policy definitions, blueprint definitions are saved to management groups or subscriptions, and are available through inheritance to any children in the management group hierarchy.

The Cloud Governance team has decided that enforcement of required Azure Policy and RBAC assignments across subscriptions will be implemented through Azure Blueprints and associated artifacts:

1. In the root management group, create a blueprint named `governance-baseline`.
2. Add the following artifacts to the blueprint:
    1. Policy assignments for the custom Azure Policy definitions defined at the management group root.
    2. Resource Group definitions for any groups required for all subscriptions by Governance MVP.
    3. Standard subscription and resource group RBAC assignments.
3. Publish the blueprint.
4. Assign the `governance-baseline` blueprint to all subscriptions.

See the [Azure Blueprints documentation](/azure/governance/blueprints/overview) for more information on creating and using blueprints.

### Demilitarized Zone (DMZ)

Specific subscriptions often require some level of access to on-premise resources. This is common in migration scenarios or dev scenarios where dependent resources reside in the on-premises datacenter.  

Until trust in the cloud environment is fully established it's important to tightly control and monitor any allowed communication between the on-premises environment and cloud workloads, and that the on-premises network is secured against potential unauthorized access from cloud-based resources. To support these scenarios, the governance MVP adds the following best practices:

1. Establish a cloud DMZ.
    1. The [Cloud DMZ reference architecture](/azure/architecture/reference-architectures/dmz/secure-vnet-hybrid) establishes a pattern and deployment model for creating a VPN Gateway in Azure.
    2. Validate that on-premises security and traffic management mechanisms are configured to only allow access to and from authorized resources and services hosted in the cloud.
    3. Validate that proper DMZ connectivity is in place for a local edge device in the on-premises datacenter.
    4. Validate that the local edge device is compatible with Azure VPN Gateway requirements.
    <!-- 5. Once connection to the on-premisess VPN has been verified, capture the Resource Manager template created by that reference architecture. -->
1. Create a second blueprint named `dmz`.
    1. Add the Resource Manager template for the VPN Gateway to the blueprint.
1. Apply the DMZ blueprint to any subscriptions requiring on-premises connectivity. This blueprint should be applied in addition to the governance MVP blueprint.

One of the biggest concerns raised by IT security and traditional governance teams is the risk that early stage cloud adoption will compromise existing assets. The above approach allows cloud adoption teams to build and migrate hybrid solutions, with reduced risk to on-premises assets. As trust in the cloud environment increases, later evolutions may remove this temporary solution.

> [!NOTE]
> The above is a starting point to quickly create a baseline governance MVP. This is only the beginning of the governance journey. Further evolution will be needed as the company continues to adopt the cloud and takes on more risk in the following areas:
>
> - Mission-critical workloads
> - Protected data
> - Cost management
> - Multi-cloud scenarios
>
>Moreover, the specific details of this MVP are based on the example journey of a fictitious company, described in the articles that follow. We highly recommend becoming familiar with the other articles in this series before implementing this best practice.
