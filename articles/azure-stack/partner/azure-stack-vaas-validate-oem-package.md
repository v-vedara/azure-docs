---
title: Validate original equipment manufacturer (OEM) packages in Azure Stack Validation as a Service | Microsoft Docs
description: Learn how to validate original equipment manufacturer (OEM) packages with Validation as a Service.
services: azure-stack
documentationcenter: ''
author: mattbriggs
manager: femila

ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 02/19/2019
ms.author: mabrigg
ms.reviewer: johnhas
ms.lastreviewed: 02/19/2019

ROBOTS: NOINDEX

---

# Validate OEM packages

[!INCLUDE [Azure_Stack_Partner](./includes/azure-stack-partner-appliesto.md)]

You can test a new OEM package when there has been a change to the firmware or drivers for a completed solution validation. When your package has passed the test, it is signed by Microsoft. Your test must contain the updated OEM extension package with the drivers and firmware that have passed Windows Server logo and PCS tests.

[!INCLUDE [azure-stack-vaas-workflow-validation-completion](includes/azure-stack-vaas-workflow-validation-completion.md)]

> [!IMPORTANT]
> Before uploading or submitting packages, review [Create an OEM package](azure-stack-vaas-create-oem-package.md) for the expected package format and contents.

## Managing packages for validation

When using the **Package Validation** workflow to validate a package, you will need to provide a URL to an **Azure Storage blob**. This blob is the test signed OEM package that will be installed as part of the update process. Create the blob using the Azure Storage Account you created during setup (see [Set up your Validation as a Service resources](azure-stack-vaas-set-up-resources.md)).

### Prerequisite: Provision a storage container

Create a container in your storage account for package blobs. This container can be used for all your Package Validation runs.

1. In the [Azure Portal](https://portal.azure.com), go to the storage account created in [Set up your Validation as a Service resources](azure-stack-vaas-set-up-resources.md).
2. On the left blade under **Blob Service**, select on **Containers**.
3. Select **+ Container** in the menu bar and provide a name for the container, e.g., `vaaspackages`.

### Upload package to storage account

1. Prepare the package you want to validate. If your package has multiple files, compress it into a `.zip` file.
2. In the [Azure Portal](https://portal.azure.com), select the package container and upload the package by selecting on **Upload** in the menu bar.
3. Select the package `.zip` file to upload. Keep defaults for **Blob type** (i.e., **Block Blob**) and **Block size**.

> [!NOTE]
> Please ensure that the `.zip` contents are placed at the root of the `.zip` file. There should be no sub-folders in the package.

### Generate package blob URL for VaaS

When creating a **Package Validation** workflow in the VaaS portal, you will need to provide a URL to the Azure Storage blob containing your package.

#### Option 1: Generating a blob SAS URL

Use this option if you do not want to enable public read access to your storage container or blobs.

1. In the [Azure portal](https://portal.azure.com/), go to your storage account, and navigate to the .zip containing your package

2. Select **Generate SAS** from the context menu

3. Select **Read** from **Permissions**

4. Set **Start time** to the current time, and **End time** to at least 48 hours from **Start time**. If you will be running other tests with the same package, consider increasing **End time** for the length of your testing. Any tests scheduled through VaaS after **End time** will fail and a new SAS will need to be generated.

5. Select **Generate blob SAS token and URL**.

Use the **Blob SAS URL** when providing package blob URLs to the portal.

#### Option 2: Grant public read access

> [!CAUTION]
> This option opens up your blob(s) for anonymous read-only access.

1. Grant **public read access for blobs only** to the package container by following the instructions in section [Grant anonymous users permissions to containers and blobs](https://docs.microsoft.com/azure/storage/storage-manage-access-to-resources#grant-anonymous-users-permissions-to-containers-and-blobs).

> [!NOTE]
> If you are providing a package URL to an *interactive test* (for example, Monthly AzureStack Update Verification or OEM Extension Package Verification), you must grant **full public read access** to proceed with testing.

2. In the package container, select the package blob to open the properties pane.

3. Copy the **URL**. Use this value when providing package blob URLs to the portal.

## Apply monthly update

[!INCLUDE [azure-stack-vaas-workflow-section_update-azs](includes/azure-stack-vaas-workflow-section_update-azs.md)]

> [!NOTE]
> After you apply the monthly update, we recommend that you run Test-AzureStack to verify that the update was applied correctly and is in a healthy state. If Test-AzureStack fails, report the issue to Microsoft. Don't proceed with the test pass until the issue is resolved. Information about how to run the Test-Azure Stack command can be found in this [article](https://docs.microsoft.com/azure/azure-stack/azure-stack-diagnostic-test).

## Create a Package Validation workflow

1. Sign in to the [VaaS portal](https://azurestackvalidation.com).

2. [!INCLUDE [azure-stack-vaas-workflow-step_select-solution](includes/azure-stack-vaas-workflow-step_select-solution.md)]

3. Select **Start** on the **Package Validation** tile.

    ![Package validations workflow tile](media/tile_validation-package.png)

4. [!INCLUDE [azure-stack-vaas-workflow-step_naming](includes/azure-stack-vaas-workflow-step_naming.md)]

5. Enter the Azure Storage blob URL to the test signed OEM package requiring a signature from Microsoft. For instructions, see [Generate package blob URL for VaaS](#generate-package-blob-url-for-vaas).

6. [!INCLUDE [azure-stack-vaas-workflow-step_upload-stampinfo](includes/azure-stack-vaas-workflow-step_upload-stampinfo.md)]

7. [!INCLUDE [azure-stack-vaas-workflow-step_test-params](includes/azure-stack-vaas-workflow-step_test-params.md)]

    > [!NOTE]
    > Environment parameters cannot be modified after creating a workflow.

8. [!INCLUDE [azure-stack-vaas-workflow-step_tags](includes/azure-stack-vaas-workflow-step_tags.md)]

9. [!INCLUDE [azure-stack-vaas-workflow-step_submit](includes/azure-stack-vaas-workflow-step_submit.md)]
    You will be redirected to the tests summary page.

## Required tests

Following tests are required for OEM package validation:

- OEM Extension Package Verification
- Cloud Simulation Engine

## Run Package Validation tests

1. In the **Package Validation tests summary** page, you will run a subset of the listed tests appropriate to your scenario.

    In the validation workflows, **scheduling** a test uses the workflow-level common parameters that you specified during workflow creation (see [Workflow common parameters for Azure Stack Validation as a Service](azure-stack-vaas-parameters.md)). If any of test parameter values become invalid, you must resupply them as instructed in [Modify workflow parameters](azure-stack-vaas-monitor-test.md#change-workflow-parameters).

    > [!NOTE]
    > Scheduling a validation test over an existing instance will create a new instance in place of the old instance in the portal. Logs for the old instance will be retained but are not accessible from the portal.  
    Once a test has completed successfully, the **Schedule** action becomes disabled.

2. Select the agent that will run the test. For information about adding local test execution agents, see [Deploy the local agent](azure-stack-vaas-local-agent.md).

3. To complete OEM Extension Package Verification select **Schedule** from the context menu to open a prompt for scheduling the test instance.

4. Review the test parameters and then select **Submit** to schedule OEM Extension Package Verification for execution.

5. Review the result for OEM Extension Package Verification. Once the test has succeeded, schedule Cloud Simulation Engine for execution.

When all tests have successfully completed, send the name of your VaaS solution and Package Validation to [vaashelp@microsoft.com](mailto:vaashelp@microsoft.com) to request package signing.

## Next steps

- [Monitor and manage tests in the VaaS portal](azure-stack-vaas-monitor-test.md)
