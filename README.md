# SAP HANA Operational Reporting Templates for SAP ECC

![Sample deployment architecture](https://github.com/SAPDocuments/Tutorials/blob/master/tutorials/haas-ecc-operational-reporting-sample-data/prod.png)


## Description
The current repository contains templates with Calculation Views for SAP HANA and models and setup instructions for connecting to this content from SAP Analytics Cloud.

The Calculation Views and other design-time artifacts can be cloned into SAP Web IDE Full Stack or SAP Web IDE for SAP HANA. The artifacts can be deployed into SAP Cloud Platform, SAP HANA Service in Cloud Foundry or SAP HANA, extended application services, advanced model (XS Advanced).

## Available templates

1.	**Master Data**:  Material Master List, Customer Master List, Vendor Master List
2.	**Sales**: Sales Order List (Header), Sales Order List (Items), Sales Order List (Scheduling), Sales Organization Analysis, Credit Memo Analysis, Billing Document List, Return Analysis (SD), Perfect Order Fulfillment, Credit Memo vs. Billing Document Ratio Report, Stock Overview (SD)
3.	**Purchasing**: Purchase Orders Analysis, Goods Receipts / Goods Issues , Logistic Invoice Analysis, Stock Overview (MM) , Order History Overview
4.	**Shipping**: Flexible customer open item reporting (Debtor), Flexible vendor open items  reporting (Creditor), Overdue item reporting, Customer open item analysis (Day sales outstanding )
5.	**Accounting**: Outbound Delivery Overview, Outbound Delivery Items Overview, Outbound Deliveries for Picking, Outbound Delivery Items for Picking



## Requirements
The current repository contains sample data so that the models can be tested without configuring the replication through Smart Data Integration. 

The replication from the SAP Netweaver system is done through the activation of Operational Data Provisioning extractors and the ABAP adapter.

- For replication:
    - These templates leverage the Operational Data Provisioning (ODP) extractors embedded in the BW component in an ECC system. You can get further information about them in the [introduction to ODP](https://wiki.scn.sap.com/wiki/display/BI/Introduction+to+Operational+Data+Provisioning)
    - Follow instructions in note [1931427 - ODP Data Replication API 2.0](https://launchpad.support.sap.com/#/notes/1931427) to check or fulfill prerequisites in the source system
    - Version 2.3.5.2 or higher of the Data Provisioning Agent available in the [SAP Development Tools](https://tools.hana.ondemand.com/#cloudintegration) or the [SAP Software Downloads Center](https://launchpad.support.sap.com/#/softwarecenter/search/dpagent)

- To clone the calculation views:
    - SAP Cloud Platform, SAP HANA Service in Cloud Foundry (requires a productive account) **or** SAP HANA (on-premises) with XS Advanced (including, SAP HANA, express edition). The trial account in Cloud Foundry can only be used with the test data and does not allow for integration with SAP Analytics Cloud.
    - SAP Web IDE Full Stack in SAP CLoud Platform **or** SAP Web IDE for SAP HANA in XS Advanced
    - Permissions to create a user-provided service in the same organization and space in which the HDI container will be deployed
- To expose the models through Information Access
    - The latest version of the SAP HANA Analytics Adapter, available in [SAP Development Tools](https://tools.hana.ondemand.com/#hanatools)

## Download and Installation

### Using Sample data
You can find step-by-step instructions in the following tutorial: [https://developers.sap.com/tutorials/haas-ecc-operational-reporting-sample-data.html](https://developers.sap.com/tutorials/haas-ecc-operational-reporting-sample-data.html)
1. Check these instructions to set up the SAP HANA Service and connect through SAP Web IDE Full Stack: [Get Started with SAP Cloud Platform, SAP HANA Service](https://developers.sap.com/mission.haas-get-started.html)
2. In SAP Web IDE, right-click on the Workspace and choose `Git -> Clone Repository`. Paste the URL for the current repository and clone.
3. Remove the folders with names `CONFIGURE_ME` in `db` and in `db/src`

   ![Remove configurable files](https://github.com/SAPDocuments/Tutorials/blob/master/tutorials/haas-dm-connect-sdi/remove.png)
   
4. Right-click on the `db` folder and choose **Build**. After a successful build, the models can be executed.

### Using Smart Data Integration
The integration requires the publication of ODP extractors from the ABAP system using the ABAP adapter in Smart Data Integration. The Calculation Views will be deployed in an HDI container and the virtual tables will access a remote source. The remote source is accessed in the same way as a plain or replicated schema. Further context about this process is provided in this [blog post](https://blogs.sap.com/2019/02/23/smart-data-integration-cross-container-access-and-the-sap-hana-service/).

1. Check these instructions to set up the SAP HANA Service and connect through SAP Web IDE Full Stack: [Get Started with SAP Cloud Platform, SAP HANA Service](https://developers.sap.com/mission.haas-get-started.html)
2. Install the Data Provisioing Agent in the source ABAP system
3. Connect the Data Provisioning Agent to the SAP HANA System. For the SAP HANA Service in Cloud Foundry, use JDBC: Web Sockets  as in [Step 3 in this tutorial](https://developers.sap.com/tutorials/haas-dm-connect-sdi.html#7528fa4a-b1c9-4113-9323-006da3688291).
4. Register the ABAP Adapter with the SAP HANA instance. 
5. Create a Remote Source based on the agent you have registered. Call the remote source **SAPECC** unless you want to adapt the references to it in virtual tables. See [Step 6 in this tutorial](https://developers.sap.com/tutorials/haas-dm-connect-sdi.html#8c5cdacb-24ee-433d-b30b-d1f93f63ac6a) for reference.
6. Create a user with permissions to the remote source. Here is a sample SQL statement creating a user and setting permissions to a source with name `SAPECC`
  ```
  CREATE USER PLUSR PASSWORD "HanaRocks01" NO FORCE_FIRST_PASSWORD_CHANGE ;
  CREATE ROLE CCROLE;
  grant "CREATE VIRTUAL TABLE", "DROP", "CREATE REMOTE SUBSCRIPTION", "PROCESS REMOTE SUBSCRIPTION EXCEPTION"  on remote source "SAPECC" to CCROLE with grant option;
  grant  CCROLE to PLUSR with admin option;
  ```
7. Create a user-provided service to provide the credentials for access to the remote source form the HDI Container. Call it **CC_ACCESS** unless you want to adapt references to it. See sample steps in [Step 3 in this tutorial](https://developers.sap.com/tutorials/haas-dm-access-cross-container-schema.html#874fa741-8693-4407-80bb-16a53f3c6c16)
8. Adapt the mta.yaml file to include the user-provided service as a resource for the HDI Container. See [Step 4 in this tutorial](https://developers.sap.com/tutorials/haas-dm-access-cross-container-schema.html#3c27eccb-a523-412c-81de-302798bfceaa) for an example.  
9. Remove `_CONFIGURE_ME` from the folders `cfg`, `src/loads` and from the file `remote.hdbgrants`. Be sure to **Save all** the modified artifacts

  ![Remove configure me](https://github.com/SAPDocuments/Tutorials-Contribution/blob/master/tutorials/haas-dm-connect-sdi/conf1.png)
  
  > Note: The virtual tables use a remote source called `SAPECC`, adapt them if the name of your remote source is different. The `.hdbgrants` file refers to a user-provided service called `CC_ACCESS` and also references the remote source `SAPECC`. Adapt them if necessary.
10. Delete the folder `demo_loads`.
11. Right-click on the database module and choose **Build**. 
12. You can now execute the flowgraphs and preview data on calculation views
13. It is recommended to apply performance tweaks to the Calculation Views once they have been adapted to your reporting needs. Check the [SAP Help](https://help.sap.com/viewer/9de0171a6027400bb3b9bee385222eff/2.0.04/en-US/fe76cf4d9daa4cd7865bf93b25993e70.html) for suggestions.

### Deploy the SAP HANA Advanced Analytics Adapter
The SAP HANA Advanced Analytics adapter allows for the consumption of Calculation Views in HDI Containers through Information Access. SAP Analytics Cloud uses Information Access for Live connection. Other tools, such as Analysis for Office work with this connector too.

Follow the instructions in this [blog post series for installation on SAP Cloud Foundry](https://blogs.sap.com/2019/04/24/connecting-the-sap-hana-service-on-cloud-foundry-to-sap-analytics-cloud-the-lazy-approach-pt1/) or these instructions for the [installation on on-premises systems](https://blogs.sap.com/2019/01/22/from-zero-to-analytics-setting-up-a-user-for-the-hana-analytics-adapter/).

### Configure SAP Analytics Cloud
1. Create a connection to your HANA Analytics Adapter entry endpoint called `HANALIVE`. **Note that the models currently depend on the connection being called HANALIVE**. This can be adapter later. 

# LICENSE
Copyright (c) 2018 SAP SE or an SAP affiliate company. All rights reserved. This file is licensed under the Apache Software License, version 2.0 except as noted otherwise in the [LICENSE file](LICENSE).
