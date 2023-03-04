# Azure Data Factory Deploy Action

GitHub Action that performs a side-effect free deployment of Azure Data Factory entities in a Data Factory instance.

## How it works

The GitHub Action uses [pre and post-deployment scripts](https://docs.microsoft.com/en-us/azure/data-factory/continuous-integration-deployment#script) to prevent the deployment from potential side effects, such as:

* Execution of active triggers during the deployment process that could corrupt resources relationships or have pipelines in undesired states.
* Availability of unused resources that could bring confusion to data engineers and reduce maintainability.

![Architecture Design](./images/architecture-design.png)

It is designed to run the following steps sequentially:

1. A pre-deployment task checks for all active triggers and stop them.
2. An ARM template deployment task is executed.
3. A post-deployment task deletes all resources that have  been removed from the ARM template (triggers, pipelines, dataflows, datasets, linked services, Integration Runtimes) and restarts the active triggers.

## When to use

The action is useful on Continuous Deployment (CD) scenarios, where a step can be added in a workflow to deploy the Data Factory resources.

## Getting Started

### Prerequisites

* A GitHub repository integrated with an existing Azure Data Factory. For more info, see [Source control in Azure Data Factory](https://docs.microsoft.com/en-us/azure/data-factory/source-control).
* An Azure service principal with `Contributor` role added as a secret on your GitHub repository. For more info, see [Create a service principal and add it to GitHub secret](https://docs.microsoft.com/azure/developer/github/connect-from-azure#create-a-service-principal-and-add-it-to-github-secret).

If your GitHub Actions workflows are running on a [self-hosted runner](https://docs.github.com/en/actions/hosting-your-own-runners/about-self-hosted-runners), ensure you have installed:

* [PowerShell 7.1](https://docs.microsoft.com/en-us/powershell/scripting/install/installing-powershell?view=powershell-7.1) with [Azure Az PowerShell module](https://docs.microsoft.com/en-us/powershell/azure/install-az-ps?view=azps-6.4.0) 
* [GitHub Actions Runner](https://github.com/actions/runner) `v2.280.3` or later.

### Example Usage

```yml
steps:
  - name: Login via Az module
    uses: azure/login@v1
    with:
      creds: ${{ secrets.AZURE_CREDENTIALS }}
      enable-AzPSSession: true 

  - name: Deploy resources
    uses: Azure/data-factory-deploy-action@v1.2.0
    with:
      resourceGroupName: myResourceGroup
      dataFactoryName: myDataFactory
      armTemplateFile: myArmTemplate.json
      # armTemplateParametersFile: myArmTemplateParameters.json [optional]
      # additionalParameters: 'key1=value key2=value keyN=value' [optional]
      # skipAzModuleInstallation: true [optional]
      # cleanupResources: true [optional]
```

### Inputs

| Name | Description | Required | Default value |
| --- | --- | --- | --- |
| `resourceGroupName` | Data Factory resource group name | true | |
| `dataFactoryName` | Data Factory name | true |  |
| `armTemplateFile` | Data Factory ARM template file | false | `ARMTemplateForFactory.json`  |
| `armTemplateParametersFile` | Data Factory ARM template parameters file | false | `ARMTemplateParametersForFactory.json`  |
| `additionalParameters` | Data Factory custom parameters. Key-values must be splitted by space. | false |
| `skipAzModuleInstallation` | Skip `Az` powershell module installation. | false | false |
| `cleanupResources` | Whether or not to run the cleanup step to remove deleted factory elements. | false | true |

## Contributing

This project welcomes contributions and suggestions.  Most contributions require you to agree to a
Contributor License Agreement (CLA) declaring that you have the right to, and actually do, grant us
the rights to use your contribution. For details, visit https://cla.opensource.microsoft.com.

When you submit a pull request, a CLA bot will automatically determine whether you need to provide
a CLA and decorate the PR appropriately (e.g., status check, comment). Simply follow the instructions
provided by the bot. You will only need to do this once across all repos using our CLA.

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/).
For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or
contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.

## Trademarks

This project may contain trademarks or logos for projects, products, or services. Authorized use of Microsoft 
trademarks or logos is subject to and must follow 
[Microsoft's Trademark & Brand Guidelines](https://www.microsoft.com/en-us/legal/intellectualproperty/trademarks/usage/general).
Use of Microsoft trademarks or logos in modified versions of this project must not cause confusion or imply Microsoft sponsorship.
Any use of third-party trademarks or logos are subject to those third-party's policies.
