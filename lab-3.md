
# Lab 3 

The lab includes steps for the user to follow along with associated
screenshots that provide visual aid. In each screenshot, sections are
highlighted with orange boxes to indicate the area(s) user should focus
on.

## Introduction

In this lab, you will create another Eventstream to ingest additional
data into our existing Eventhouse. We will see how to include
transformations within the Eventstream to control what data we want to
be adding to the KQL database.

By the end of this lab, you will have learned:

- Eventstream Processing and Transformation
- Writing KQL Queries to Join Data from an External Database
- Exploring KQL Data within Power BI

## Fabric Eventstreams

### Task 1: Create an Eventstream

1. Open the **Fabric workspace** you have been using for the course today.

    ![A screenshot of a computer Description automatically generated](./media/image62.png)

2. There is additional streaming data for us to ingest related to our
   e-commerce store. For this Eventstream, though, we will want to
   transform the data prior to loading it into the Eventhouse. Instead
   of going to the **"Real-Time Hub"** we can create a new Eventstream
   directly from the workspace. From the **+ New** dropdown menu,
   create a new **Eventstream**.

    ![A screenshot of a computer Description automatically generated](./media/image67.png)

3. Give the new Eventstream the name, **es_Fabrikam_ClickEvents**,
   check the **"Enhanced Capabilities"** option, and then click on
   **Create**.

    ![A screenshot of a computer](./media/image68.png)

4. Under the Home ribbon, click on the **Add source** dropdown and then
   select **External sources**.

    ![A screenshot of a computer Description automatically generated](./media/image69.png)

5. Similarly to the previous lab, we will be connecting to an Azure
   Event Hub which has data being streamed to from a Python notebook.
   Click on the **"Azure Event Hubs"** tile.

    ![A screenshot of a web page Description automatically generated](./media/image70.png)

6. Create a **New connection**.

    ![A screenshot of a computer Description automatically generated](./media/image71.png)

7. From your environment details page, copy and paste all the necessary
   connection settings into the appropriate fields.

    - Event Hub namespace: **realtimeconsumersiad**
    - Event Hub: **rta-iad-clicks**
    - Shared Access Key Name: **rti-reader**
    - Shared Access Key: **H/5gTWuI+lQLT9HYBzuqzPdKaR6YWXCRw+AEhE6k1gE=**

    ![A screenshot of a computer Description automatically generated](./media/image72.png)

8. Once all the properties have been filled out, click on **Connect**.

9. In the configuration of the Azure Event Hub data source, you will
   need to modify the **Consumer group** of the Event Hub to ensure
   that you gain access to a unique access point to the stream of data.
   Within your **Environment details** find the property that lists
   what your consumer group name will be and place that into the field
   box. It will appear something like "**cg-xx**"

    ![A screenshot of a computer Description automatically generated](./media/image73.png)

10. Click on **Next**.

11. On the review and create window, verify that everything is configured correctly and click **Add**.

    ![A screenshot of a computer Description automatically generated](./media/image74.png)

12. Once the stream is configured, you will be able to see a preview of
    the data coming from the Event Hub.

    ![A table of numbers and numbers Description automatically generated](./media/image75.png)

13. Examine the data being received. There are two types of events that
    are logged from the e-commerce website: clicks and impressions.

    - **IMPRESSION** - An impression event is recorded each time an
      advertisement or a product listing is displayed to a user.
      Impressions are a measure of how many times an item (ad or product)
      is viewed, regardless of whether it is interacted with.

    - **CLICK** - A click event is recorded when a user interacts with an
      item by clicking on it. This usually indicates a higher level of
      engagement compared to an impression.

    In addition to the clicks and impression events that are logged, there
    are details about what product the click or impression was for, what
    device and browser the webpage was loaded from, and what IP address
    accessed the page and how long the page took to load.

### Task 2: Transform the Eventstream

1. You will now take this stream of data and transform it to fit into
   your KQL Database in a way that can be easily understood for
   analysts looking to derive insights from this data. Within the
   Eventstream canvas, click on the dropdown for the **Transform
   events** object.

    ![A screenshot of a chat](./media/image76.png)

2. From the list of available operations, select the **Manage fields**
   option.

    ![A screenshot of a search box Description automatically generated](./media/image77.png)

3. On the new icon that appears called **Manage_fields1**, click on the
   **pencil icon** to select what fields you wish to add to your stream
   from the source.

    ![A screen shot of a computer Description automatically generated](./media/image78.png)

4. In the flyout pane that appears, click on the button the option to
   **Add all fields**.

    ![A screenshot of a computer Description automatically generated](./media/image79.png)

5. From the list of fields, select the one called **PartitionId** and
   click on the ellipses (...) that appears when you hover over the field.

    ![A screenshot of a computer Description automatically generated](./media/image80.png)

6. Choose the option to **Remove** that field. For this stream of data
   coming from the Event Hub, partitioning is not being used, so this
   column is not helpful to us, thus we are removing it.

    ![A screenshot of a computer Description automatically generated](./media/image81.png)

7. Remove all the following fields that will not be needed for this
   stream:

    - userAgent
    - page_loading_seconds
    - EventProcessedUtcTime
    - EventEnqueredUtcTime

    You should be left with the following fields:

    ![A screenshot of a computer Description automatically generated](./media/image82.png)

8. Hover over the **eventDate** field and when an ellipses (...) appears on
   the right-hand side of the window, click it.

    ![A screenshot of a computer Description automatically generated](./media/image83.png)

9. Choose the option, **Edit**.

    ![A screen shot of a computer Description automatically generated](./media/image84.png)

10. Click on the **Change type toggle** to modify the data type of this
    field. The original type is a String; you need to modify the
    **Converted Type** to **DateTime**. After you have finished, click on
    **Save**.

    ![A screenshot of a computer Description automatically generated](./media/image85.png)

11. Click on the **Refresh** button in your Eventstream Test result
    pane. These are the fields that you want to load to a KQL Database.

    ![A screenshot of a computer Description automatically generated](./media/image86.png)

### Task 3: Split Eventstream and Load Two Destinations

1. While you could load this stream of data to a KQL Database for
   analysis, you may want to have another way to consume this data to
   differentiate CLICK events and IMPRESSION events. Add another
   transformation activity to the user interface by hovering over the
   end of the **Manage_Fields1** transform.

    ![A screen shot of a computer screen Description automatically generated](./media/image87.png)

2. Choose the **Filter** transform from the available list of
   operations.

    ![A screenshot of a computer Description automatically generated](./media/image88.png)

3. Click on the **pencil icon** on the new transformation, **Filter1**.

    ![A screen shot of a computer Description automatically generated](./media/image89.png)

4. In the flyout that appears on the right-hand side of the screen,
   customize the filter conditions to reflect a way to only return
   CLICK values by using the values below. It is important to note that
   the Filter transform is case sensitive.

    - **Operation name** - Clicks
    - **Select a field to filter on** - eventType
    - **Keep events when the value** - equals -- CLICK **(Important! This
      is a case sensitive field, ensure to input in all capitals for this
      example)**

    ![A screenshot of a screen Description automatically generated](./media/image90.png)

5. Choose the **Save** option to keep your changes.

6. Click on the **Refresh** button again to verify the data has been
   filtered to CLICK eventTypes.

    ![A screenshot of a computer Description automatically generated](./media/image91.png)

7. This may be the only field that you're interested in sending to a
   table, but another option is to instead create two separate streams
   to route different information to two or more tables. From the
   **Home** ribbon of the Eventstream, click on the **Transform events**
   dropdown and then select **Filter**.

    ![A screenshot of a computer Description automatically generated](./media/image92.png)

8. A new object called **Filter1 (Name may differ)** will appear on
   your canvas. You will need to connect the **Manage_fields1 stream**
   to the new filter transformation. Drag a line from the green dot on
   one transform to another to make that connection.

    ![A screenshot of a computer Description automatically generated](./media/image93.png)

9. Click on the **pencil icon** for **Filter2** to edit its settings.

    ![A screenshot of a computer Description automatically generated](./media/image94.png)

10. In the flyout that appears on the right-hand side of the screen,
    customize the filter conditions to reflect a way to only return
    IMPRESSION values by using the values below. Remember that the
    Filter transform is case sensitive.

    - **Operation name** - Impressions
    - **Select a field to filter on** - eventType
    - **Keep events when the value** - equals -- IMPRESSION **(Important!
      This is a case sensitive field, ensure to input in all capitals for
      this example)**

    ![A screenshot of a screen Description automatically generated](./media/image95.png)

11. Choose the **Save** option to keep your changes.

12. Before we load the data into new tables within our KQL Database, we
    can remove additional columns which are not needed. In this case, for
    the stream of data filtered for our "CLICK" records, we no longer need
    the "eventType" column since every row holds the same value. For our
    "IMPRESSION" stream of data, we can remove the "eventType" column for
    the same reasons mentioned above, and we can also remove the "referrer"
    column as it is empty for every row in this table.

13. Click on the **+ icon** after the **Clicks** filter operation.

    ![A white board with writing on it Description automatically generated](./media/image96.png)

14. In the dropdown menu, select **Manage fields**.

    ![A screenshot of a computer Description automatically generated](./media/image97.png)

15. Click on the **pencil icon** to select what fields you wish to
    add/remove to your stream.

    ![A white rectangular sign with black text Description automatically generated](./media/image98.png)

16. Rename the operation to **Manage_Clicks**. Also, select **Add all
    fields** then remove **eventType**.

    ![A screenshot of a computer](./media/image99.png)

17. Next, add another **Manage fields** transform connected to the
    **Impressions** filter as seen below.

    ![A diagram of a rectangle with a rectangle and a rectangle with a rectangle with a rectangle with a rectangle with a rectangle with a rectangle with a rectangle with](./media/image100.png)

18. Click on the **pencil icon** to select what fields you wish to
    add/remove to your stream.

    ![A white card with a pen and a red box Description automatically generated](./media/image101.png)

19. Rename the operation to **Manage_Impressions**. Then select **Add all
    fields** then remove **eventType** and **referrer**. Your **Manage fields**
    transform should look like the following:

    ![A screenshot of a computer Description automatically generated](./media/image102.png)

20. Now that you have cleaned up the data for the streams for each of
    the types of events, you need to load each stream into a new table
    on the KQL Database. Click on the **+ icon** after the
    **Manage_Clicks** manage fields operation.

    ![A screen shot of a computer Description automatically generated](./media/image103.png)

21. In the dropdown list that appears, go to the **Destinations** and select
    **KQL Database**.

    ![A screenshot of a computer Description automatically generated](./media/image104.png)

22. Click on the **pencil icon** for the **KQL_Database1** operation.

    ![A screenshot of a computer Description automatically generated](./media/image105.png)

23. For this destination, configure the following properties:

    - **Destination name** -- dbo-Clicks
    - **Workspace** - RTI_username
    - **KQL Database** - eh_Fabrikam
    - **Destination Table** - Create a new table called **Clicks**

    ![A screenshot of a computer Description automatically generated](./media/image106.png)

24. Click on **Save** at the bottom of the flyout.

25. Do the same thing for the Impressions table with the following
    information configured as below.

    ![A screenshot of a computer Description automatically generated](./media/image107.png)

26. Save your changes.

27. This Eventstream is now ready to begin streaming. Click on
    **Publish** to begin that stream.

    ![A screenshot of a computer Description automatically generated](./media/image108.png)

28. With the Eventstream now running, you should see the Eventstream
    user interface slightly change to signify that you are streaming the
    data from Event Hub, transforming and splitting that data stream, and
    loading it into two separate KQL Database tables.

    ![A screenshot of a computer Description automatically generated](./media/image109.png)

## Adding More Data to KQL Database

### Task 4: Validate Event Data Tables

1. Return to your **RTI_username** workspace.

2. Open the **eh_Fabrikam** KQL Database.

    ![A screenshot of a computer Description automatically generated](./media/image110.png)

3. With the Eventstream running, you should now see two new tables on
   the KQL Database Overview page. After letting the Eventstream run
   for several hours, you will see that the **Top tables** within the
   KQL Database will be displayed on the Overview page and show how
   much data is stored within the table.

    ![A screenshot of a computer Description automatically generated](./media/image111.png)

4. Click on the **Impressions** table. This table receives about 1.5
   million records every 24 hours. There are many more impressions than
   Clicks, so this will be your largest table for the purposes of this
   class.

    ![A screenshot of a computer Description automatically generated](./media/image112.png)

### Task 5: Create KQL Database Shortcuts for Dimension tables

Up to this point you have been working with streaming data, but are still missing some critical elements to be able to derive intelligence from the data you have brought in. In this task we will bring in data from an external Azure SQL Database which will serve as dimension tables
within our KQL Database. This will allow us to better describe the data we are streaming currently. For example, all our tables contain a form of Product ID which is a numeric field, but it would be better if we had some sort of Product name to be able to display. The data we need to support this is currently in an external Azure SQL Database, let's see how easy it is to make connections to some of these dimension tables.

1. **Create a New KQL Queryset**
   - In the **eh_Fabrikam** database, click on the drop-down menu called **New related item**.
   - Select the option **KQL Queryset**.
   ![New KQL Queryset](./media/image113.png)

2. **Name and Create the Queryset**
   - Name the KQL Queryset **Create Tables**.
   - Click the **Create** button.
   ![Name and Create Queryset](./media/image114.png)

3. **Connect to the KQL Database**
   - The OneLake data hub will open. Select the **"eh_Fabrikam"** KQL database.
   - Click **"Connect"**.
   ![Connect to Database](./media/image115.png)

4. **Prepare the Query Window**
   - In the new interface, click within the query window.
   - Use the keyboard shortcut **Ctrl + A** to highlight all text and delete it.
   ![Prepare Query Window](./media/image116.png)

5. **Enter the KQL Script**
   - Paste the following KQL script into the query window. This script creates shortcuts to external Azure SQL Database tables, making them available in your KQL database.
   ![Enter KQL Script](./media/image117.emf)
   ![Script Execution](./media/image118.png)

6. **Execute the Script**
   - Click the **Run** button to execute the script.
   ![Run Script](./media/image119.png)

7. **Verify Shortcuts**
   - In the Database Explorer window, check for a new folder called **Shortcuts**.
   - Within this folder, you should see two tables linked from the external Azure SQL Database.
   ![Verify Shortcuts](./media/image120.png)

8. **Run a KQL Query**
   - To validate, run a KQL query to view data from one of the linked tables.
   ![Run KQL Query](./media/image121.emf)

9. **Build a Power BI Report**
   - Highlight your query results.
   - Click the **Build Power BI report** button to create a report using this data.
   - Explore briefly and then close the report builder by clicking the **X button**.
   ![Build Power BI Report](./media/image123.png)
   ![Close Report Builder](./media/image124.png)

10. **Return to the KQL Database**
    - Navigate back to the **eh_Fabrikam** KQL Database.
    - Click on **Shortcuts** in the navigation pane to view all created shortcuts.
    ![Return to Database](./media/image125.png)

## Summary

In this task, you connected your KQL Database to dimension tables from an external Azure SQL Database. This allows you to enrich your streaming data with additional context, such as product names, enhancing your ability to analyze and report on the data. You also explored creating a Power BI report from your enriched data.

## References

For additional resources and learning, you can explore the following links:

- [Microsoft Fabric GA announcement](https://aka.ms/Fabric-Hero-Blog-Ignite23)
- [Fabric Guided Tour](https://aka.ms/Fabric-GuidedTour)
- [Microsoft Fabric free trial](https://aka.ms/try-fabric)
- [Microsoft Fabric website](https://aka.ms/microsoft-fabric)
- [Fabric Learning modules](https://aka.ms/learn-fabric)
- [Fabric technical documentation](https://aka.ms/fabric-docs)
- [Fabric e-book](https://aka.ms/fabric-get-started-ebook)
- [Fabric community](https://aka.ms/fabric-community)

For deeper insights, check out the related blogs:

- [Data Factory experience in Fabric](https://aka.ms/Fabric-Data-Factory-Blog)
- [Synapse Data Engineering experience in Fabric](https://aka.ms/Fabric-DE-Blog)
- [Synapse Data Science experience in Fabric](https://aka.ms/Fabric-DS-Blog)
- [Synapse Data Warehousing experience in Fabric](https://aka.ms/Fabric-DW-Blog)
- [Real-Time Intelligence experience in Fabric](https://blog.fabric.microsoft.com/en-us/blog/category/real-time-intelligence)
- [Power BI announcement blog](https://aka.ms/Fabric-PBI-Blog)
- [Data Activator experience in Fabric](https://aka.ms/Fabric-DA-Blog)
- [Administration and governance in Fabric](https://aka.ms/Fabric-Admin-Gov-Blog)
- [OneLake in Fabric blog](https://aka.ms/Fabric-OneLake-Blog)
- [Dataverse and Microsoft Fabric integration blog](https://aka.ms/Dataverse-Fabric-Blog)

© 2024 Microsoft Corporation. All rights reserved.

By using this demo/lab, you agree to the following terms:

The technology/functionality described in this demo/lab is provided by Microsoft Corporation for purposes of obtaining your feedback and to provide you with a learning experience. You may only use the demo/lab to evaluate such technology features and functionality and provide feedback to Microsoft. You may not use it for any other purpose. You may not modify, copy, distribute, transmit, display, perform, reproduce, publish, license, create derivative works from, transfer, or sell this demo/lab or any portion thereof.

COPYING OR REPRODUCTION OF THE DEMO/LAB (OR ANY PORTION OF IT) TO ANY OTHER SERVER OR LOCATION FOR FURTHER REPRODUCTION OR REDISTRIBUTION IS EXPRESSLY PROHIBITED.

THIS DEMO/LAB PROVIDES CERTAIN SOFTWARE TECHNOLOGY/PRODUCT FEATURES AND FUNCTIONALITY, INCLUDING POTENTIAL NEW FEATURES AND CONCEPTS, IN A SIMULATED ENVIRONMENT WITHOUT COMPLEX SET-UP OR INSTALLATION FOR THE PURPOSE DESCRIBED ABOVE. THE TECHNOLOGY/CONCEPTS REPRESENTED IN THIS DEMO/LAB MAY NOT REPRESENT FULL FEATURE FUNCTIONALITY AND MAY NOT WORK THE WAY A FINAL VERSION MAY WORK. WE ALSO MAY NOT RELEASE A FINAL VERSION OF SUCH FEATURES OR CONCEPTS. YOUR EXPERIENCE WITH USING SUCH FEATURES AND FUNCTIONALITY IN A PHYSICAL ENVIRONMENT MAY ALSO BE DIFFERENT.

**FEEDBACK**. If you give feedback about the technology features, functionality and/or concepts described in this demo/lab to Microsoft, you give to Microsoft, without charge, the right to use, share and commercialize your feedback in any way and for any purpose. You also give to third parties, without charge, any patent rights needed for their products, technologies and services to use or interface with any specific parts of a Microsoft software or service that includes the feedback. You will not give feedback that is subject to a license that requires Microsoft to license its software or documentation to third parties because we include your feedback in them. These rights survive this agreement.

MICROSOFT CORPORATION HEREBY DISCLAIMS ALL WARRANTIES AND CONDITIONS WITH REGARD TO THE DEMO/LAB, INCLUDING ALL WARRANTIES AND CONDITIONS OF MERCHANTABILITY, WHETHER EXPRESS, IMPLIED OR STATUTORY, FITNESS FOR A PARTICULAR PURPOSE, TITLE AND NON-INFRINGEMENT. MICROSOFT DOES NOT MAKE ANY ASSURANCES OR REPRESENTATIONS WITH REGARD TO THE ACCURACY OF THE RESULTS, OUTPUT THAT DERIVES FROM USE OF DEMO/ LAB, OR SUITABILITY OF THE INFORMATION CONTAINED IN THE DEMO/LAB FOR ANY PURPOSE.

**DISCLAIMER**

This demo/lab contains only a portion of new features and enhancements in Microsoft Power BI. Some of the features might change in future releases of the product. In this demo/lab, you will learn about some, but not all, new features.