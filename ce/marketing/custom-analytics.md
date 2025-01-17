---
title: "Prepare for analytic reporting with Power BI (Dynamics 365 Marketing) | Microsoft Docs"
description: "Describes how to set up data sources in Dynamics 365 Marketing to make them available to Power BI, and how to download and connect a Power BI template to them."
ms.date: 01/28/2021
ms.service: dynamics-365-marketing
ms.custom: 
  - dyn365-marketing
ms.topic: article
author: alfergus
ms.author: alfergus
manager: shellyha
search.audienceType: 
  - admin
  - customizer
  - enduser
search.app: 
  - D365CE
  - D365Mktg
---

# Prepare for analytic reporting with Power BI

[!INCLUDE[cc-data-platform-banner](../includes/cc-data-platform-banner.md)]

Dynamics 365 Marketing provides a wide selection of built-in analytics throughout the application. But you can also create your own custom analytics and reports from your Dynamics 365 Marketing data by using Power BI. We provide endpoints that you can use to connect Power BI to Dynamics 365 Marketing, plus a downloadable Power BI template that you can open in Power BI Desktop, connect to your Dynamics 365 data sources and then customize as needed. When you're done setting it up, you can publish and share your Power BI report using the standard Power BI online tools.

<a name="data-sources"></a>

## Data sources for custom analytics

To create custom analytics, you can connect two different data sources, each of which provides a different type of data:

- **Profile data** is stored in the organizational database and includes the entities and records that you see, edit, and create when working directly in Dynamics 365 Marketing. These include common entities like contacts, accounts, leads, events, customer journeys, and more. You'll use the Power BI data connector called "Common Data Services for Apps" for this type of data.
- **Interaction data** is stored in the marketing-insights service database and includes information about how your contacts interacted with your marketing initiatives, including email opens, email clicks, event registrations, page submissions, and more. You can see this type of information when you look at the insights built into Dynamics 365 Marketing, but you can't create these records nor view them directly. In the current release, you'll use the Power BI "Azure Blob Storage" connector for this type of data. In future releases, you'll also be able use the dataflow connector.

You'll be able to connect directly to your Dynamics 365 Marketing  database from Power BI to fetch your profile data, but to access interaction data you'll need to set up Azure Blob Storage, configure Dynamics 365 Marketing to save interaction data there, and then connect Power BI to your blob storage.

<!-- [kfm: this video is being edited and will be hosted elsewhere. Revive this section and update the embed when it's ready]

Watch [the following video](https://www.youtube.com/watch?v=pBrB1BohUrE)
for a quick overview of all the data that is available for your marketing analytical reports.<br/><br/>

<div class="embeddedvideo"><iframe width="560" height="315" src="https://www.youtube.com/embed/pBrB1BohUrE" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></div> -->

<a name="connect-blob"></a>

## Set up Azure Blob storage and connect it to Marketing

1. Sign into [portal.azure.com](https://portal.azure.com) using the same account where you are running Dynamics 365 Marketing.

1. If you don't already have one, then create a general-purpose storage account in the Azure Blob storage as described in [Quickstart: Upload, download, and list blobs using the Azure portal](/azure/storage/blobs/storage-quickstart-blobs-portal).

    > [!NOTE]
    > Your general-purpose storage account can be either V1 or V2. Both will work with the Power BI template supplied by Microsoft, but we recommend using V2 if possible because it will support more advanced Power BI capabilities in the future.

1. Create a blob storage container in your storage account, also as described in [Quickstart: Upload, download, and list blobs using the Azure portal](/azure/storage/blobs/storage-quickstart-blobs-portal).

1. Launch the Microsoft Azure Storage Explorer, which is a desktop app, and connect it to your Azure account. If  you don't already have it, you can download if for free from [https://azure.microsoft.com/features/storage-explorer/](https://azure.microsoft.com/features/storage-explorer/).

1. In the Storage Explorer, navigate down to the blob storage container you just created.

    ![Azure Storage Explorer](media/custom-analytics-storage-explorer.png "Azure Storage Explorer")

1. Right-click on your blob storage container and then select **Get shared access signature** from the context menu. The **Shared Access Signature** dialog opens.

    ![The Shared Access Signature dialog](media/custom-analytics-sas.png "The Shared Access Signature dialog")

    Make the following settings:

    - Choose a **Start time** and **Expiry time** to establish the period during which your signature will remain valid. Note that the signature must remain valid for as long as you intend to run the export&mdash;the export will stop immediately when the signature expires.
    - Enable all four **Permissions** by selecting their check boxes.

1. Select **Create** to create the signature. The dialog refreshes to show a **URL** and **Query string**. Select the **Copy** button to copy the **URL** shown here and paste in a temporary text file so you can use it later in this procedure.

    ![Copy the URL](media/custom-analytics-sas-created.png "Copy the URL")

1. Sign into Dynamics 365 Marketing and go to **Settings** > **Advanced settings** > **Marketing settings** > **Marketing analytics configuration**. A list of marketing analytics configurations opens. If a record is already listed here, then select it to open it; otherwise, select **New** from the command bar to create a new record.

1. The **Marketing analytics configuration** page opens.

    ![New marketing analytics configuration](media/custom-analytics-configuration-new.png "New marketing analytics configuration")

    Make the following settings:

    - **Name**: Enter a name to identify this configuration record.
    - **Export to blob storage sas token**: Paste the URL you copied earlier in this procedure.
    - **Export from date**: Optional. When left empty, all interactions available in the Marketing application will be exported. If a date value is specified, then only the interactions that happened after this date will be exported. This is useful for reducing the amount of data exported if you are not interested in creating reports for older interactions.

    > [!NOTE]
    > Parallel exports are not supported, only one export configuration is allowed.

1. Save your settings.

    ![Saved marketing analytics configuration](media/custom-analytics-configuration-running.png "Saved marketing analytics configuration")

    > [!NOTE]
    > The URL you pasted here has two parts: a storage URL identifying your storage account and container name, and an SAS token that allows Dynamics 365 Marketing to connect to your blob storage. To prevent other people from being able to connect to your blob storage, only the storage URL will be saved in the Dynamics 365 database. The full URL, including the SAS token, will be stored securely in Dynamics 365 Marketing. This is why you can never see the full URL in this form.

1. After a short time, Dynamics 365 Marketing will begin to export interaction data as individual files to your blob storage container, where you'll be able to pick it up using Power BI. You'll also be able to see these files and an associated JSON file in the Azure Storage Explorer.

    > [!NOTE]
    > Interactions are exported on the fly, but only until the provided SAS token expires. You can update the SAS token at any time by pasting a new URL in the **Export to blob storage sas token** field and saving the configuration. The already stopped export will be automatically restarted. This is only possible for the same storage account or container. If you need to export to a different storage account and container, stop the current export, delete the configuration, and create a new one.

## Get Power BI templates and connect them your Marketing data

Once your Azure Blob storage is set up and connected to Dynamics 365 Marketing, you're ready to start working in Power BI to connect to your data sources and design your analytics. Do help get you started, we provide a [gallery of pre-built templates and sample reports](marketing-analytics/analytics-gallery-start.md) that you can download, explore, customize, and connect to your Dynamics 365 Marketing instance.

More information: [Download and use marketing analytics templates and sample reports for Power BI](marketing-analytics/analytics-gallery-start.md)

![A collage of various Power BI reports](media/pbi-gallery-overview.png)


[!INCLUDE[footer-include](../includes/footer-banner.md)]