---
title: Deploy templates with the command line in Azure Stack | Microsoft Docs
description: Learn how to use the cross-platform command line interface (CLI) to deploy templates from inside the ClientVM or after using the VPN to connect to Azure Stack.
services: azure-stack
documentationcenter: 
author: heathl17
manager: byronr
editor: 
ms.assetid: 9584177f-4af3-4834-864d-930b09ae0995
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/26/2016
ms.author: helaw
translationtype: Human Translation
ms.sourcegitcommit: 219dcbfdca145bedb570eb9ef747ee00cc0342eb
ms.openlocfilehash: b9af860ad2a6b97d4ae38037c088a6f77a4d4c2e


---
# <a name="deploy-templates-in-azure-stack-using-the-command-line"></a>Deploy templates in Azure Stack using the command line
Use the command line to deploy Azure Resource Manager templates to the Azure Stack POC. Azure Resource Manager templates deploy and provision all the resources for your application in a single, coordinated operation.

## <a name="download-template"></a>Download template
To test a deployment with the CLI, download the files azuredeploy.json and azuredeploy.parameters.json from the [create storage account example template](https://github.com/Azure/AzureStack-QuickStart-Templates/tree/master/101-create-storage-account).

## <a name="deploy-template"></a>Deploy template
Navigate to the folder where these files were downloaded and run the following command to deploy the template:

    azure group create "cliRG" "local" –f azuredeploy.json –d "testDeploy" –e azuredeploy.parameters.json

This command deploys the template to the resource group **cliRG** in the Azure Stack POC’s default location.

## <a name="validate-template-deployment"></a>Validate template deployment
To see this resource group and storage account, use the following commands:

    azure group list

    azure storage account list

## <a name="next-steps"></a>Next steps
[Manage user permissions](azure-stack-manage-permissions.md)




<!--HONumber=Nov16_HO3-->


