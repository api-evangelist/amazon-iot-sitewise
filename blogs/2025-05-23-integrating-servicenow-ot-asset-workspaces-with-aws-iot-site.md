---
title: "Integrating ServiceNow OT Asset Workspaces with AWS IoT SiteWise Asset Models"
url: "https://aws.amazon.com/blogs/iot/integrating-servicenow-ot-asset-workspaces-with-aws-iot-sitewise-asset-models/"
date: "Fri, 23 May 2025 20:58:24 +0000"
author: "Maria El Khoury"
feed_url: "https://aws.amazon.com/blogs/iot/tag/aws-iot-sitewise/feed/"
---
<p>In today’s digital enterprise landscape, organizations increasingly rely on asset management solutions to streamline their operations. Companies often find themselves managing the same physical assets across multiple IT and operational technology (OT) systems. One of the services that help IT teams track and manage enterprise IT assets is <a href="https://www.servicenow.com/" rel="noopener noreferrer" target="_blank">ServiceNow</a>. This service handles everything from hardware and software inventory to service requests, license compliance, and the complete lifecycle of technology resources. For OT, <a href="https://aws.amazon.com/iot-sitewise/" rel="noopener noreferrer" target="_blank">AWS IoT SiteWise</a> is a managed service that enables companies to collect, organize, and analyze industrial equipment data at scale. This service provides a unified repository of live and historical operational data, empowering organizations to make data-driven decisions that enhance production efficiency and optimize asset maintenance.</p> 
<p>A common challenge that organizations face using ServiceNow and AWS IoT SiteWise together is maintaining consistent asset information across their systems. When an asset hierarchy is updated in ServiceNow, operations teams must manually replicate these changes in AWS IoT SiteWise, leading to duplicate work and potential inconsistencies. This process is also time-consuming, error-prone, and creates unnecessary overhead to manage the same assets in both environments.This blog post presents an approach to synchronize asset data between ServiceNow and AWS IoT SiteWise. By implementing this integration pattern, you can eliminate manual updates, reduce errors, and maintain consistent asset hierarchies across your IT and OT platforms.</p> 
<h2>Solution overview</h2> 
<p>This solution uses AWS services to create an automated integration between ServiceNow and AWS IoT SiteWise. When changes occur in ServiceNow’s asset management system, they can automatically flow through to AWS IoT SiteWise, ensuring both systems remain synchronized.</p> 
<p><img alt="Architecture" class=" wp-image-16730 aligncenter" height="327" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2025/05/21/IOTB-835_1_Architecture.png" style="margin: 10px 0px 10px 0px;" width="1020" /></p> 
<p style="text-align: center;">Figure 1: Architecture</p> 
<p>Figure 1 shows a two-phase data flow. In the first phase (Ingest), data moves from ServiceNow through Amazon AppFlow to in <a href="https://aws.amazon.com/pm/serv-s3/?trk=fecf68c9-3874-4ae2-a7ed-72b6d19c8034&amp;sc_channel=ps&amp;ef_id=Cj0KCQjwh_i_BhCzARIsANimeoERoEnC94DM5XOVtkPcj1PeePw7UAHM89HxZju9yOs32soV4xVTpnoaAiUHEALw_wcB:G:s&amp;s_kwcid=AL!4422!3!536452728638!e!!g!!amazon%20s3!11204620052!112938567994&amp;gclid=Cj0KCQjwh_i_BhCzARIsANimeoERoEnC94DM5XOVtkPcj1PeePw7UAHM89HxZju9yOs32soV4xVTpnoaAiUHEALw_wcB" rel="noopener noreferrer" target="_blank">Amazon Simple Storage Service (Amazon S3)</a>. In the second phase (Import), the data flows through AWS Glue and back to Amazon S3 before reaching AWS IoT SiteWise. Both phases serve different purposes:</p> 
<p><strong>Ingest phase:</strong></p> 
<ul> 
 <li><a href="https://aws.amazon.com/appflow/" rel="noopener noreferrer" target="_blank">Amazon AppFlow</a> pulls asset data from ServiceNow tables: Operations Technology (OT), OT Entity, OT Entity Type.</li> 
 <li>The data is then stored in Amazon S3 and in <a href="https://parquet.apache.org/" rel="noopener noreferrer" target="_blank">parquet format</a>.</li> 
</ul> 
<p><strong>Import phase:</strong></p> 
<ul> 
 <li><a href="https://aws.amazon.com/glue/" rel="noopener noreferrer" target="_blank">AWS Glue</a> transforms the parquet files to a <a href="https://docs.aws.amazon.com/iot-sitewise/latest/userguide/bulk-operations-schema.html" rel="noopener noreferrer" target="_blank">JSON format</a> that AWS IoT SiteWise can import.</li> 
 <li>Transformed JSON files are stored in Amazon S3.</li> 
 <li>AWS IoT SiteWise imports the asset information to create or update asset models and hierarchies.</li> 
</ul> 
<p><strong>Implementation overview</strong></p> 
<p>This post presents the following stages to implement this integration:</p> 
<ol> 
 <li>Configure the ServiceNow connector in Amazon AppFlow to ingest asset data into Amazon S3.</li> 
 <li>Create AWS Glue jobs to transform the data from parquet to JSON to match the required AWS IoT SiteWise import format.</li> 
 <li>Set up AWS IoT SiteWise asset import from Amazon S3.</li> 
</ol> 
<h3>Prerequisites</h3> 
<p>Before implementing this solution, you’ll need:</p> 
<ul> 
 <li>A ServiceNow instance with access to your asset tables. In this example, we use: 
  <ul> 
   <li>Operations Technology (OT) (<code>cmdb_ci_ot</code>): Contains operational technology device records from ServiceNow. These records include basic attributes like name, serial number, model number, manufacturer, and location information.</li> 
   <li>OT Entity (<code>cmdb_ot_entity</code>): Contains records that define the OT entity instances and their relationships. It also represents how devices connect to each other in the operational hierarchy.</li> 
   <li>OT Entity Type (<code>cmdb_ot_entity_type</code>): Contains records that define the types or categories of OT entities (such as, Area, Process Cell, Unit, and Equipment Module). It also defines the allowed parent-child relationships in the operational hierarchy.</li> 
  </ul> </li> 
 <li>Three tables work together to provide a complete picture of OT assets: 
  <ul> 
   <li><code>cmdb_ci_ot</code> handles the physical device information (configuration Items).</li> 
   <li><code>cmdb_ot_entity</code> manages the instances and relationships of these devices.</li> 
   <li><code>cmdb_ot_entity_type</code> defines the hierarchy structure rules and categories.</li> 
  </ul> </li> 
 <li>An AWS account with permissions to use Amazon AppFlow, Amazon S3, AWS Glue, and AWS IoT SiteWise.</li> 
 <li>ServiceNow credentials for a system-only user with permission to read your asset tables.</li> 
</ul> 
<h2>Implementation</h2> 
<h3>Configure the ServiceNow connector</h3> 
<p>In this section, you will set up Amazon AppFlow to pull data from ServiceNow and configure AWS Glue to catalog the data.</p> 
<p><strong>Create a ServiceNow connection in Amazon AppFlow</strong></p> 
<ol> 
 <li>Navigate to the Amazon AppFlow console.</li> 
 <li>In the left menu, under <strong>Connections</strong>, choose <strong>ServiceNow</strong> from the Connectors dropdown.</li> 
 <li>Choose <strong>Create connection</strong>.</li> 
 <li>In the <strong>Connect to ServiceNow</strong> pop up, see Figure 2, enter the following: 
  <ol type="a"> 
   <li>Select either <strong>Basic Auth</strong> or <strong>OAuth2</strong> as needed.</li> 
   <li>Fill in the necessary information according to the <a href="https://docs.aws.amazon.com/appflow/latest/userguide/servicenow.html" rel="noopener noreferrer" target="_blank">user guide</a>. 
    <ol type="i"> 
     <li>If you choose OAuth2 fill in the Client ID, Client secret and Instance URL for your ServiceNow Instance.</li> 
     <li>If you choose Basic Auth fill in the Username, Password and Instance URL for your ServiceNow Instance.</li> 
    </ol> </li> 
   <li>Click connect once all information is filled in.</li> 
  </ol> </li> 
</ol> 
<p><img alt="Connect to ServiceNow" class="aligncenter wp-image-16731" height="599" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2025/05/21/IOTB-835_2_Connect-to-ServiceNow-1024x1024.png" style="margin: 10px 0px 10px 0px;" width="600" /></p> 
<p style="text-align: center;">Figure 2: Connect to ServiceNow</p> 
<p><strong>Create flows for each table</strong></p> 
<ol> 
 <li>Navigate to the Amazon AppFlow console.</li> 
 <li>In the left menu, under <strong>Flows</strong>, choose <strong>Create flow</strong>.</li> 
 <li>Enter a <strong>Flow Name</strong> (for example: <code>cmdb_ci_ot</code>), see Figure 3, and select <strong>Next</strong>.</li> 
</ol> 
<p><img alt="Create flow" class="aligncenter wp-image-16732 " height="322" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2025/05/21/IOTB-835_3_Create-flow.png" style="margin: 10px 0px 10px 0px;" width="600" /></p> 
<p style="text-align: center;">Figure 3: Create flow</p> 
<ol start="4"> 
 <li>In the <strong>Source details </strong>dialog box, see Figure 4, enter the following: 
  <ol type="a"> 
   <li>For <strong>Source Name,</strong> select <strong>ServiceNow.</strong></li> 
   <li>Be sure that the connection you created earlier is selected under <strong>ServiceNow connection</strong>. The reference will have the name of your ServiceNow instance, this example uses “dev287617”.</li> 
   <li>For <strong>ServiceNow object</strong>, select <strong>Operational Technology (OT)</strong>.</li> 
  </ol> </li> 
 <li>Navigate to the <strong>Destination details </strong>dialog box, see Figure 4, and enter the following: 
  <ol type="a"> 
   <li>For <strong>Destination name</strong>, choose <strong>Amazon S3.</strong></li> 
   <li>Under <strong>Bucket details</strong>, either choose your destination bucket or <a href="https://docs.aws.amazon.com/AmazonS3/latest/userguide/create-bucket-overview.html" rel="noopener noreferrer" target="_blank">create one</a> through the <a href="https://console.aws.amazon.com/s3/" rel="noopener noreferrer" target="_blank">Amazon S3 console</a>. This example uses the bucket prefix <code>cmdb_ci_ot</code>.</li> 
  </ol> </li> 
 <li>Choose <strong>Next</strong>.</li> 
</ol> 
<p><img alt="Flow source and destination" class="aligncenter wp-image-16733 " height="345" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2025/05/21/IOTB-835_4_Flow-source-and-destination.png" style="margin: 10px 0px 10px 0px;" width="599" /></p> 
<p style="text-align: center;">Figure 4: Flow source and destination</p> 
<ol start="7"> 
 <li>In the <strong>Source to destination field mapping </strong>dialog box, see Figure 5, enter the following: 
  <ol type="a"> 
   <li>Under <strong>Source field name</strong>, choose <strong>Map all fields directly</strong>.</li> 
   <li>Choose <strong>Next</strong> and then choose <strong>Next</strong> again.</li> 
   <li>Finish by choosing <strong>Run flow</strong>.</li> 
  </ol> </li> 
</ol> 
<p><img alt="Run flow" class=" wp-image-16734 aligncenter" height="125" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2025/05/21/IOTB-835_5_Run-flow.png" style="margin: 10px 0px 10px 0px;" width="594" /></p> 
<p style="text-align: center;">Figure 5: Run flow</p> 
<p>Repeat the “Create flows for each table” procedure to create a flow for each of your tables to create connections with the other ServiceNow objects:</p> 
<ol type="a"> 
 <li>Flow name and Amazon S3 prefix: <code>cmdb_ot_entity</code>, ServiceNow object: OT Asset.</li> 
 <li>Flow name and Amazon S3 prefix: <code>cmdb_ot_entity_type</code>, ServiceNow object: OT Asset Type.</li> 
 <li></li> 
</ol> 
<p><strong>Set up and run AWS Glue Crawler to identify the schema</strong></p> 
<ol> 
 <li>Navigate to AWS Glue console.</li> 
 <li>In the left menu, under <strong>Data Catalog</strong>, choose <strong>Crawlers</strong>.</li> 
 <li>In the <strong>Crawlers</strong> dialog box, see Figure 6, choose <strong>Create crawler</strong>.</li> 
</ol> 
<p><img alt="AWS Glue crawlers" class=" wp-image-16735 aligncenter" height="114" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2025/05/21/IOTB-835_6_AWS-Glue-crawlers.png" style="margin: 10px 0px 10px 0px;" width="594" /></p> 
<p style="text-align: center;">Figure 6: AWS Glue crawlers</p> 
<ol start="4"> 
 <li>For <strong>Crawler name</strong>, use <strong>ServiceNow Crawler</strong> and choose <strong>Next</strong>.</li> 
</ol> 
<p><img alt="Crawler properties" class=" wp-image-16736 aligncenter" height="196" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2025/05/21/IOTB-835_7_Crawler-properties.png" style="margin: 10px 0px 10px 0px;" width="599" /></p> 
<p style="text-align: center;">Figure 7: Crawler properties</p> 
<ol start="5"> 
 <li>Choose <strong>Add a data source</strong>.</li> 
 <li>In the <strong>Add data source </strong>dialog box, see Figure 8, enter the following: 
  <ol type="a"> 
   <li>For <strong>Data source</strong>, choose <strong>S3</strong>.</li> 
   <li>For <strong>S3 path</strong>, choose <code>&lt;your_bucket_name&gt;</code>.</li> 
   <li>Select <strong>Add an S3 data source</strong>.</li> 
  </ol> </li> 
</ol> 
<p style="text-align: center;"><img alt="Crawler data source" class="aligncenter wp-image-16737 " height="268" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2025/05/21/IOTB-835_8_Crawler-data-source-777x1024.png" style="margin: 10px 0px 10px 0px;" width="203" /><br /> Figure 8: Crawler data source</p> 
<ol start="7"> 
 <li>Choose <strong>Next</strong>.</li> 
 <li>Under IAM role, select <strong>Create new IAM role, </strong>see Figure 9.</li> 
</ol> 
<p><img alt="Crawler IAM Role" class=" wp-image-16738 aligncenter" height="243" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2025/05/21/IOTB-835_9_Crawler-IAM-Role.png" style="margin: 10px 0px 10px 0px;" width="481" /></p> 
<p style="text-align: center;">Figure 9: Crawler IAM Role</p> 
<ol start="9"> 
 <li>Choose a name for the role. In this example, we use <code>AWSGlueServiceRole-ServiceNowCrawler</code>.</li> 
 <li>Select <strong>Next.</strong></li> 
 <li>Under Target database, choose an AWS Glue Database. This example uses the default database.</li> 
</ol> 
<p><img alt="AWS Glue Database" class=" wp-image-16739 aligncenter" height="258" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2025/05/21/IOTB-835_10_AWS-Glue-Database-Selection.png" style="margin: 10px 0px 10px 0px;" width="509" /></p> 
<p style="text-align: center;">Figure 10: AWS Glue Database Selection</p> 
<ol start="12"> 
 <li>Choose <strong>Next</strong>.</li> 
 <li>Choose <strong>Create</strong>.</li> 
 <li>Run the crawler. It should take around two minutes to complete.</li> 
</ol> 
<p>The ServiceNow parquet data has now been successfully imported into Amazon S3.</p> 
<h3>Transform the JSON files</h3> 
<p>In this section, you will set up the AWS Glue job to transform the parquet files to JSON format suitable for AWS IoT SiteWise and import the data into AWS IoT SiteWise.</p> 
<p><strong>Create AWS Glue Job</strong></p> 
<ol> 
 <li>Navigate to AWS Glue console.</li> 
 <li>In the menu to the left, under <strong>ETL Jobs</strong>, choose <strong>Visual ETL.</strong></li> 
 <li>In <strong>Create Job</strong>, select <strong>Visual ETL.</strong></li> 
</ol> 
<p><img alt="AWS Glue studio" class=" wp-image-16740 aligncenter" height="242" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2025/05/21/IOTB-835_11_AWS-Glue-studio.png" style="margin: 10px 0px 10px 0px;" width="666" /></p> 
<p style="text-align: center;">Figure 11: AWS Glue studio</p> 
<ol start="4"> 
 <li>Create a Source node. Select the blue plus (+) button, see Figure 12, and select <strong>Amazon S3</strong>.</li> 
</ol> 
<p><img alt="Visual ETL" class=" wp-image-16741 aligncenter" height="293" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2025/05/21/IOTB-835_12_Visual-ETL.png" style="margin: 10px 0px 10px 0px;" width="659" /></p> 
<p style="text-align: center;">Figure 12: Visual ETL</p> 
<ol start="5"> 
 <li>For <strong>Name</strong>, choose any name for the node, see Figure 13. In this example, we will use cmdb_ot_entity.</li> 
 <li>For <strong>S3 source type</strong>, select <strong>Data Catalog table</strong>.</li> 
 <li>For <strong>Database, </strong>choose the target database you previously selected when setting up your AWS Glue Crawler.</li> 
 <li>For <strong>Table</strong>, choose the first table <code>cmbd_ot_entity</code>. Repeat this step for each of the tables: <code>cmdb_ci_ot</code> and <code>cmdb_ot_entity_type</code>.</li> 
</ol> 
<p><img alt="Adding source node" class="aligncenter wp-image-16742" height="263" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2025/05/21/IOTB-835_13_Adding-source-node.png" style="margin: 10px 0px 10px 0px;" width="300" /></p> 
<p style="text-align: center;">Figure 13: Adding source node</p> 
<p><strong>Map assets to the AWS IoT SiteWise import format</strong></p> 
<ol> 
 <li>Create a new Source node by selecting the blue “+” button as shown in Figure 12.</li> 
 <li>Add a <strong>Transform</strong> node and select <strong>SQL Query</strong>.</li> 
 <li>For <strong>Name</strong>, use <strong>“assets”</strong>.</li> 
 <li>For <strong>Node parents</strong>, choose the source nodes <code>cmdb_ot_entity</code> and <code>cmdb_ci_ot</code>.</li> 
 <li>For Input sources and SQL Aliases, choose <code>cmdb_ot_entity</code> and <code>cmdb_ci_ot</code> for each, as shown in Figure 14.</li> 
</ol> 
<p><img alt="assets transform" class="aligncenter wp-image-16743" height="263" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2025/05/21/IOTB-835_14_assets-transform.png" style="margin: 10px 0px 10px 0px;" width="300" /></p> 
<p style="text-align: center;">Figure 14: assets transform</p> 
<ol start="6"> 
 <li>For <strong>SQL Query</strong>, copy and paste the following query:</li> 
</ol> 
<div class="hide-language"> 
 <div class="hide-language"> 
  <pre><code class="lang-sql">SELECT DISTINCT
    parent.sys_id as assetExternalId,
    parent.name as assetName,
    parent.ot_asset_type as assetModelExternalId,
    COLLECT_LIST(
        CASE 
            WHEN child.sys_id IS NOT NULL THEN 
                STRUCT(
                    array_join(array(parent.ot_asset_type, child.ot_asset_type), '-') as externalId,
                    child.sys_id as childAssetExternalId
                )
        END
    ) as assetHierarchies,
    (
        CASE 
            WHEN ot.sys_id IS NOT NULL THEN 
                array(
                    STRUCT('name' as externalId, ot.name as attributeValue),
                    STRUCT('serial_number' as externalId, ot.serial_number as attributeValue),
                    STRUCT('manufacturer' as externalId, ot.manufacturer as attributeValue),
                    STRUCT('model_number' as externalId, ot.model_number as attributeValue),
                    STRUCT('firmware_version' as externalId, ot.firmware_version as attributeValue),
                    STRUCT('hardware_version' as externalId, ot.hardware_version as attributeValue),
                    STRUCT('asset_tag' as externalId, ot.asset_tag as attributeValue),
                    STRUCT('category' as externalId, ot.category as attributeValue),
                    STRUCT('environment' as externalId, ot.environment as attributeValue),
                    STRUCT('short_description' as externalId, ot.short_description as attributeValue)
                )
            ELSE array()
        END
    ) as assetProperties
FROM cmdb_ot_entity as parent
LEFT JOIN cmdb_ci_ot as ot
    ON parent.ot_asset = ot.sys_id
LEFT JOIN cmdb_ot_entity as child
    ON parent.sys_id = child.parent
GROUP BY parent.sys_id, parent.name, parent.ot_asset_type, ot.sys_id, ot.name, ot.serial_number, ot.manufacturer, ot.model_number, ot.firmware_version, ot.hardware_version, ot.asset_tag, ot.category, ot.environment, ot.short_description</code></pre> 
 </div> 
</div> 
<p><strong>Map the asset model</strong></p> 
<ol start="7"> 
 <li>Create a new Source node by selecting the blue “+” button as shown in Figure 12.</li> 
 <li>Add a new <strong>Transform</strong> node and select <strong>SQL Query</strong>.</li> 
 <li>For <strong>Name</strong>, use “<strong>assetModels</strong>”.</li> 
 <li>For <strong>Node Parents, </strong>choose the source node <code>cmdb_ot_entity_type</code>.</li> 
 <li>For Input sources and SQL Aliases, use <code>cmdb_ot_entity_type</code> for each, as shown in Figure 15.</li> 
</ol> 
<p><img alt="assetModel transform" class="aligncenter wp-image-16744" height="365" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2025/05/21/IOTB-835_15_assetModel-transform.png" style="margin: 10px 0px 10px 0px;" width="300" /></p> 
<p style="text-align: center;">Figure 15: assetModel transform</p> 
<ol start="12"> 
 <li>For <strong>SQL Query</strong>, copy and paste the following query:</li> 
</ol> 
<div class="hide-language"> 
 <div class="hide-language"> 
  <pre><code class="lang-sql">SELECT DISTINCT
    parent.sys_id as assetModelExternalId,
    parent.label as assetModelName,
    (
        CASE 
            WHEN parent.ot_table IS NOT NULL THEN 
                from_json(
                '[{"dataType":"STRING","externalId":"name","name":"Name","type":{"attribute":{"defaultValue":"-"}},"unit":"-"},{"dataType":"STRING","externalId":"serial_number","name":"Serial Number","type":{"attribute":{"defaultValue":"-"}},"unit":"-"},{"dataType":"STRING","externalId":"manufacturer","name":"Manufacturer","type":{"attribute":{"defaultValue":"-"}},"unit":"-"},{"dataType":"STRING","externalId":"model_number","name":"Model Number","type":{"attribute":{"defaultValue":"-"}},"unit":"-"},{"dataType":"STRING","externalId":"firmware_version","name":"Firmware Version","type":{"attribute":{"defaultValue":"-"}},"unit":"-"},{"dataType":"STRING","externalId":"hardware_version","name":"Hardware Version","type":{"attribute":{"defaultValue":"-"}},"unit":"-"},{"dataType":"STRING","externalId":"asset_tag","name":"Asset Tag","type":{"attribute":{"defaultValue":"-"}},"unit":"-"},{"dataType":"STRING","externalId":"category","name":"Category","type":{"attribute":{"defaultValue":"-"}},"unit":"-"},{"dataType":"STRING","externalId":"environment","name":"Environment","type":{"attribute":{"defaultValue":"-"}},"unit":"-"},{"dataType":"STRING","externalId":"short_description","name":"Short Description","type":{"attribute":{"defaultValue":"-"}},"unit":"-"}]',
                    'array&amp;amp;lt;struct&amp;amp;lt;dataType:string,externalId:string,name:string,type:struct&amp;amp;lt;attribute:struct&amp;amp;lt;defaultValue:string&amp;amp;gt;&amp;amp;gt;,unit:string&amp;amp;gt;&amp;amp;gt;'
                )
            ELSE array()
        END
    ) as assetModelProperties,
    COLLECT_LIST(
        CASE 
            WHEN child.sys_id IS NOT NULL THEN 
                STRUCT(
                    array_join(array(parent.sys_id, child.sys_id), '-') as externalId,
                    child.name as name,
                    child.sys_id as childAssetModelExternalId
                )
        END
    ) as assetModelHierarchies
FROM cmdb_ot_entity_type as parent
LEFT JOIN cmdb_ot_entity_type as child
    ON parent.sys_id = child.parent_type
GROUP BY parent.sys_id, parent.name, parent.label, parent.ot_table</code></pre> 
 </div> 
</div> 
<p><strong>Combine the assets and assetModels</strong></p> 
<ol start="13"> 
 <li>Create a new Source node by selecting the blue “+” button as shown in Figure 12.</li> 
 <li>Add a new node <strong>Transform</strong> and select <strong>SQL Query</strong>.</li> 
 <li>For <strong>Name</strong>, use <strong>“assetModelHierarchy”</strong>.</li> 
 <li>For <strong>Node parents</strong>, choose the source nodes <strong>assets</strong> and <strong>assetModels</strong>.</li> 
 <li>For Input sources and SQL aliases, use <strong>assets </strong>and <strong>assetModels</strong> for each, as shown in Figure 16.</li> 
</ol> 
<p><img alt="assetModelHierarchy transform" class="aligncenter wp-image-16745" height="263" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2025/05/21/IOTB-835_16_assetModelHierarchy-transform.png" style="margin: 10px 0px 10px 0px;" width="300" /></p> 
<p style="text-align: center;">Figure 16: assetModelHierarchy transform</p> 
<ol start="18"> 
 <li>For <strong>SQL Query</strong>, copy and paste the following query:</li> 
</ol> 
<div class="hide-language"> 
 <pre><code class="lang-sql">SELECT (
    SELECT COLLECT_LIST(STRUCT(assetModels.*)) as assetModels
    FROM assetModels
) as assetModels,
(
    SELECT COLLECT_LIST(STRUCT(assets.*)) as assets
    FROM assets
) as assets</code></pre> 
</div> 
<p>Now, add the target of the transform so you can store the result of our AWS Glue Job to be used for importing into AWS IoT SiteWise.</p> 
<p><strong>Add the target of the transform</strong></p> 
<ol start="19"> 
 <li>Create a new Source node by selecting the blue “+” button as shown in Figure 12.</li> 
 <li>Add a new node <strong>Transform</strong> and select <strong>Amazon S3 </strong>from the targets.</li> 
 <li>For <strong>Name</strong>, use anything. This example usesAmazon S3.</li> 
 <li>For <strong>Node Parents</strong>, choose the source node <strong>assetModelHierarchy</strong>.</li> 
 <li>For <strong>Format</strong>, choose <strong>JSON</strong>.</li> 
 <li>For <strong>Compression Type</strong>, choose <strong>None</strong>.</li> 
 <li>For <strong>S3 Target Location</strong>, select <code>&lt;your_destination_bucket&gt;</code>.</li> 
</ol> 
<p><img alt="Adding target node" class="aligncenter wp-image-16746" height="358" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2025/05/21/IOTB-835_17_Adding-target-node.png" style="margin: 10px 0px 10px 0px;" width="300" /></p> 
<p style="text-align: center;">Figure 17: Adding target node</p> 
<p>Once complete, you should see the ETL that’s shown in Figure 18. Choose <strong>Save</strong>.</p> 
<p>Then select <strong>Run </strong>and wait for it to finish.</p> 
<p><img alt="Asset hierarchy" class=" wp-image-16747 aligncenter" height="290" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2025/05/21/IOTB-835_18_Asset-hierarchy.png" style="margin: 10px 0px 10px 0px;" width="644" /></p> 
<p style="text-align: center;">Figure 18: Asset hierarchy</p> 
<h3>Import into AWS IoT SiteWise</h3> 
<p>In this section, you will confirm the creation of the JSON file in Amazon S3 and import the ServiceNow assets into AWS IoT SiteWise.</p> 
<ol> 
 <li>First, confirm JSON file was created by doing the following: 
  <ol type="a"> 
   <li>Open the Amazon S3 console.</li> 
   <li>Select <code>&lt;your_destination_bucket&gt;</code><strong>.</strong></li> 
   <li>Select the <code>run-&lt;timestamp&gt;-part-r-00000</code> file, then choose <strong>Actions.</strong></li> 
   <li>Select <strong>Rename Object</strong>, then rename it to <code>sitewise-import.json</code>. In order to import it to AWS IoT SiteWise, the object must have the json extension added to the name.</li> 
  </ol> </li> 
 <li>To import into AWS IoT SiteWise, open the AWS IoT SiteWise console. 
  <ol type="a"> 
   <li>In the navigation pane, choose <strong>Bulk Operations</strong>.</li> 
   <li style="text-align: left;">Select <strong>New Import</strong><img alt="AWS IoT SiteWise bulk operations" class=" wp-image-16748 aligncenter" height="273" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2025/05/21/IOTB-835_19_AWS-IoT-SiteWise-bulk-operations.png" style="margin: 10px 0px 10px 0px;" width="634" /> 
    <div style="text-align: center;">
     Figure 19: AWS IoT SiteWise bulk operations
    </div> </li> 
   <li style="text-align: left;">In the Import metadata dialog box, for S3 URI, select <code>&lt;your_destination_bucket&gt;</code> then the <code>sitewise-import.json file.</code><img alt="S3 import" class=" wp-image-16749 aligncenter" height="271" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2025/05/21/IOTB-835_20_S3-import.png" style="margin: 10px 0px 10px 0px;" width="630" /> 
    <div style="text-align: center;">
     Figure 20: S3 import
    </div> </li> 
   <li style="text-align: left;">Select <strong>Import</strong> and wait for the import to finish.</li> 
  </ol> </li> 
</ol> 
<h2>Validate your work</h2> 
<p>You can now view the different models and model properties as shown in Figure 21. You can also view the different assets and asset properties as shown in Figures 22, 23, and 24. Your ServiceNow hierarchy is now successfully replicated into AWS IoT SiteWise.</p> 
<p><img alt="AWS IoT SiteWise models" class=" wp-image-16750 aligncenter" height="176" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2025/05/21/IOTB-835_21_AWS-IoT-SiteWise-models.png" style="margin: 10px 0px 10px 0px;" width="669" /></p> 
<p style="text-align: center;">Figure 21: AWS IoT SiteWise models</p> 
<p><img alt="AWS IoT SiteWise model properties" class=" wp-image-16751 aligncenter" height="345" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2025/05/21/IOTB-835_22_AWS-IoT-SiteWise-model-properties.png" style="margin: 10px 0px 10px 0px;" width="667" /></p> 
<p style="text-align: center;">Figure 22: AWS IoT SiteWise model properties</p> 
<p><img alt="AWS IoT SiteWise assets" class=" wp-image-16752 aligncenter" height="305" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2025/05/21/IOTB-835_23_AWS-IoT-SiteWise-assets.png" style="margin: 10px 0px 10px 0px;" width="672" /></p> 
<p style="text-align: center;">Figure 23: AWS IoT SiteWise assets</p> 
<p><img alt="AWS IoT SiteWise asset properties" class=" wp-image-16753 aligncenter" height="351" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2025/05/21/IOTB-835_24_AWS-IoT-SIteWise-asset-properties.png" style="margin: 10px 0px 10px 0px;" width="678" /></p> 
<p style="text-align: center;">Figure 24: AWS IoT SiteWise asset properties</p> 
<h2>Cleanup</h2> 
<p>To clean up the work outlined in this blog, navigate to the Amazon AppFlow console to delete the flows and ServiceNow connection. Delete any user and user credentials created for ServiceNow. In AWS Glue, delete the crawler, job, and tables from the AWS Glue Data Catalog. Remove the assets and asset models from AWS IoT SiteWise. Finally, delete both the parquet and transformed JSON files from your Amazon S3 buckets.</p> 
<h2>Conclusion</h2> 
<p>This blog presented a process to integrate ServiceNow asset data with AWS IoT SiteWise, a practice that allows organizations to maintain consistent asset information across their IT and OT asset management solutions. To fully automate this integration, schedule your Amazon AppFlow flows to run at regular intervals and configure your AWS Glue job with a schedule trigger. When setting up the AWS Glue job, you can also add the ‘.json’ extension to the output file via the ETL script. Both solutions eliminate manual data entry and ensure consistency between IT and OT systems.</p> 
<p>Try implementing this solution and let us know about your experience in the comments section below. Want to learn more about AWS IoT SiteWise? See the <a href="https://docs.aws.amazon.com/iot-sitewise/latest/userguide/what-is-sitewise.html" rel="noopener noreferrer" target="_blank">AWS IoT SiteWise Developer Guide</a> for more information.</p> 
<hr /> 
<h3>About the authors</h3> 
<p style="clear: both;"><img alt="" class="alignleft wp-image-4649 size-thumbnail" height="121" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2025/05/22/IOTB-835_Maria-El-Khoury.jpg" width="100" /><br /> <strong>Maria El Khoury</strong> is a Solutions Architect at AWS supporting manufacturing customers in their digital transformation journey. With a background of building IoT and computer vision solutions, Maria is especially interested in applying AWS in the fields of Industrial IoT and supply chain.</p> 
<p style="clear: both;"><img alt="" class="alignleft size-thumbnail wp-image-16778" height="121" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2025/05/22/IOTB-835_Brent-Van-Wynsberge.jpg" width="100" /><br /> <strong>Brent Van Wynsberge</strong> is a Solutions Architect at AWS supporting enterprise customers. He accelerates the cloud adoption journey for organizations by aligning technical objectives to business outcomes and strategic goals. Brent is an IoT enthusiast, specifically in the application of IoT in manufacturing, he is also interested in DevOps, data analytics and containers.</p>
