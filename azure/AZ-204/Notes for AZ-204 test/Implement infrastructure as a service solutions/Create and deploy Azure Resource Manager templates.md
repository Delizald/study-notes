# Learning objectives

- Describe what role Azure Resource Manager (ARM) has in Azure and the benefits of using ARM templates.
- Explain what happens when ARM templates are deployed and how to structure them to support your solution.
- Create a template with conditional resource deployments.
- Choose the correct deployment mode for your solution.
- Create and deploy an ARM template by using Visual Studio Code.

# Explore Azure Resource Manager

When a user sends a request from any of the Azure tools, APIs, or SDKs, Resource Manager receives the request. It authenticates and authorizes the request. Resource Manager sends the request to the Azure service.

(azure-resource-manager.png)


## Why choose Azure Resource Manager templates?
consider the following advantages of using templates:

`Declarative syntax`: Azure Resource Manager templates allow you to create and deploy an entire Azure infrastructure declaratively. For example, you can deploy not only virtual machines, but also the network infrastructure, storage systems, and any other resources you may need.

`Repeatable results`: Repeatedly deploy your infrastructure throughout the development lifecycle and have confidence your resources are deployed in a consistent manner. Templates are idempotent, which means you can deploy the same template many times and get the same resource types in the same state.

`Orchestration`: You don't have to worry about the complexities of ordering operations. Resource Manager orchestrates the deployment of interdependent resources so they're created in the correct order.

## Template file
(https://docs.microsoft.com/en-us/azure/azure-resource-manager/templates/template-functions)

Within your template, you can write template expressions that extend the capabilities of JSON. These expressions make use of the functions provided by Resource Manager.

Parameters - Provide values during deployment that allow the same template to be used with different environments. (https://docs.microsoft.com/en-us/azure/azure-resource-manager/templates/parameters)

Variables - Define values that are reused in your templates. They can be constructed from parameter values. (https://docs.microsoft.com/en-us/azure/azure-resource-manager/templates/variables)

User-defined functions - Create customized functions that simplify your template. (https://docs.microsoft.com/en-us/azure/azure-resource-manager/templates/user-defined-functions)

Resources - Specify the resources to deploy. (https://docs.microsoft.com/en-us/azure/azure-resource-manager/templates/resource-declaration)

Outputs - Return values from the deployed resources. (https://docs.microsoft.com/en-us/azure/azure-resource-manager/templates/outputs)

# Deploy multi-tiered solutions

When you deploy a template, Resource Manager converts the template into REST API operations. For example, when Resource Manager receives a template with the following resource definition:

```
"resources": [
  {
    "type": "Microsoft.Storage/storageAccounts",
    "apiVersion": "2019-04-01",
    "name": "mystorageaccount",
    "location": "westus",
    "sku": {
      "name": "Standard_LRS"
    },
    "kind": "StorageV2",
    "properties": {}
  }
]
```

It converts the definition to the following REST API operation, which is sent to the Microsoft.Storage resource provider:

```
PUT
https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Storage/storageAccounts/mystorageaccount?api-version=2019-04-01
REQUEST BODY
{
  "location": "westus",
  "sku": {
    "name": "Standard_LRS"
  },
  "kind": "StorageV2",
  "properties": {}
}
```

You can deploy a template using any of the following options:

- Azure portal
- Azure CLI
- PowerShell
- REST API
- Button in GitHub repository
- Azure Cloud Shell

## Defining multi-tiered templates
You can use a single template but it also makes sense to divide your deployment requirements into a set of targeted, purpose-specific templates. You can easily reuse these templates for different solutions. To deploy a particular solution, you create a master template that links all the required templates.

## Share templates 
Template specs enable you to store a template as a resource type. You use role-based access control to manage access to the template spec. Users with read access to the template spec can deploy it, but not change the template. (https://docs.microsoft.com/en-us/azure/azure-resource-manager/templates/template-specs)

# Explore conditional deployment

Sometimes you need to optionally deploy a resource in an Azure Resource Manager template.
Use the `condition element` to specify whether the resource is deployed. 

**Conditional deployment doesn't cascade to child resources. If you want to conditionally deploy a resource and its child resources, you must apply the same condition to each resource type.**

## New or existing resource
The following example shows how to use condition to deploy a new storage account or use an existing storage account.
It contains a parameter named `newOrExisting` which is used as a condition in the resources section.
```
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "storageAccountName": {
      "type": "string"
    },
    "location": {
      "type": "string",
      "defaultValue": "[resourceGroup().location]"
    },
    "newOrExisting": {
      "type": "string",
      "defaultValue": "new",
      "allowedValues": [
        "new",
        "existing"
      ]
    }
  },
  "functions": [],
  "resources": [
    {
      "condition": "[equals(parameters('newOrExisting'), 'new')]",
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "2019-06-01",
      "name": "[parameters('storageAccountName')]",
      "location": "[parameters('location')]",
      "sku": {
        "name": "Standard_LRS",
        "tier": "Standard"
      },
      "kind": "StorageV2",
      "properties": {
        "accessTier": "Hot"
      }
    }
  ]
}
```

When the parameter `newOrExisting` is set to new, the condition evaluates to true. The storage account is deployed. However, when newOrExisting is set to existing, the condition evaluates to false and the storage account isn't deployed.

## Runtime functions

Use the `if` function to make sure the function is only evaluated for conditions when the resource is deployed.

# Set the correct deployment mode
When deploying your resources, you specify that the deployment is either an incremental update or a complete update.

For both modes, Resource Manager tries to create all resources specified in the template. If the resource already exists in the resource group and its settings are unchanged, no operation is taken for that resource. If you change the property values for a resource, the resource is updated with those new values. If you try to update the location or type of an existing resource, the deployment fails with an error. Instead, deploy a new resource with the location or type that you need.

## Complete mode
In complete mode, Resource Manager deletes resources that exist in the resource group but aren't specified in the template.

If your template includes a resource that isn't deployed because condition evaluates to false, the result depends on which REST API version you use to deploy the template. If you use a version earlier than 2019-05-10, the resource isn't deleted. With 2019-05-10 or later, the resource is deleted. The latest versions of Azure PowerShell and Azure CLI delete the resource.

Be careful using complete mode with copy loops. Any resources that aren't specified in the template after resolving the copy loop are deleted.

## Incremental mode

when redeploying an existing resource in incremental mode, the outcome is different. Specify all properties for the resource, not just the ones you're updating. A common misunderstanding is to think properties that aren't specified are left unchanged. If you don't specify certain properties, Resource Manager interprets the update as overwriting those values.