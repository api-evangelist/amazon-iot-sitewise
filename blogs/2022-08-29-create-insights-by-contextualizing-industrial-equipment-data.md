---
title: "Create insights by contextualizing industrial equipment data using AWS IoT SiteWise (Part 2)"
url: "https://aws.amazon.com/blogs/iot/create-insights-by-contextualizing-industrial-equipment-data-using-aws-iot-sitewise-part-2/"
date: "Mon, 29 Aug 2022 06:28:04 +0000"
author: "Jan Borch"
feed_url: "https://aws.amazon.com/blogs/iot/tag/aws-iot-sitewise/feed/"
---
<h1>Introduction</h1> 
<p>In the first part of this blog series <a href="https://aws.amazon.com/blogs/iot/create-insights-by-contextualizing-industrial-equipment-data-using-aws-iot-sitewise-part-1/">Create insights by contextualizing industrial equipment data using AWS IoT SiteWise (Part 1)</a> we focused on asset modelling and real-time analytics in AWS IoT SiteWise. We created a dashboard in AWS IoT SiteWise Monitor to get a real-time overview of our furnace heating cycles. But we concluded that a more in-depth analysis was needed to find the root cause of the abnormal heating cycle of <code>Furnace1</code>. In this second part of the blog, we will show how customers can use the AWS IoT SiteWise cold tier storage feature to export the raw, aggregated and meta-data to AWS IoT Analytics for further analysis.</p> 
<h2>Enable AWS IoT SiteWise cold tier storage and AWS IoT Analytics export</h2> 
<p>AWS IoT SiteWise cold tier storage feature makes it easy to consume historical data in downstream AWS analytic services. Additionally, it will also lower your storage cost on AWS IoT SiteWise by exporting historical data to Amazon S3. Customer can freely define how long the data will be kept in the time-series optimized AWS IoT SiteWise data before being exported into S3 by setting a data retention threshold.</p> 
<h3>Enabling AWS IoT SiteWise cold tier storage</h3> 
<p>To enable AWS IoT SiteWise S3 export, open the <a href="https://console.aws.amazon.com/iotsitewise/home">AWS IoT SiteWise console</a>, choose <strong>Settings, Storage, Edit</strong> in the navigation pane and check <strong>Enable Cold tier storage</strong>,</p> 
<p><img alt="SiteWise Storage Tiers" class="alignnone size-full wp-image-7994" height="740" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/05/27/SiteWiseStorageTiers.png" width="2432" /></p> 
<p>Enter an existing <strong>S3 bucket location</strong> in the same AWS region,</p> 
<p><img alt="SiteWise Storage Tiers S3" class="alignnone size-full wp-image-7995" height="1060" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/05/27/SiteWiseStorageTiersS3.png" width="2436" /></p> 
<p>Check <strong>Enable AWS IoT Analytics data store</strong>, type <code>iotsitewise</code> as the <strong>Data store name</strong> and choose <strong>Save</strong></p> 
<p><img alt="SiteWise Storage Tiers IoT Analytics" class="alignnone size-full wp-image-7996" height="752" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/05/27/SiteWiseStorageTiersIoTAnalytics.png" width="2436" /></p> 
<p>For this walk through we will use AWS IoT Analytics to query the data and visualize it in Amazon QuickSight.<br /> The AWS IoT SiteWise S3 export feature exports information on asset properties from the asset model into the <code>asset-metadata</code> S3 prefix when the model change. Once the status of the S3 export is enabled, you should see a line delimited JSON file per asset in your S3 bucket. The raw data is exported every 6 hours and will be placed into the <code>raw</code> S3 prefix. For more details on the export format and location, see <a href="https://docs.aws.amazon.com/iot-sitewise/latest/userguide/file-path-and-schema.html">File paths and schemas of data saved in the cold tier</a>.</p> 
<h3>Create an AWS IoT Analytics dataset to analyze the raw data</h3> 
<p>We can now start to analyze the AWS IoT SiteWise exported data with AWS IoT Analytics. Open the <a href="https://console.aws.amazon.com/iotanalytics/home">AWS IoT Analytics console</a>, choose <strong>Datasets</strong> in the navigation menu and choose <strong>Create dataset</strong>, <strong>Create SQL</strong>. This will open a wizard that will guide you through the Dataset creation. On the first screen name your data set and choose the <code>iotsitewise</code> data store that was created by the AWS IoT SiteWise cold tier export wizard.</p> 
<p><img alt="IoT Analytics Dataset Name" class="alignnone size-full wp-image-8005" height="1182" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/05/27/IoTAnalyticsDatasetName.png" width="1786" /></p> 
<p>Choose <strong>Next</strong> to open the <strong>Author SQL query</strong> dialog, copy and past the sample query below.</p> 
<p>The query used some advanced Athena SQL features and also demonstrated how AWS IoT Analytics Dataset queries can be used to join data from different AWS IoT Analytics data stores. For this analysis, we want to query the metric <code>Last Holding Cycle Time</code> for all our assets in the current month. To accomplish this, the query starts from the raw data store filtered by a specific month. It joins the <code>asset_metadata</code> data store to retrieve the property metadata like the asset name. Finally, it joins the <code>asset_metadata</code> again, but this time grouped by <code>asset ID</code>. This last <code>JOIN</code> statement retrieves all static attribute of the corresponding AWS IoT SiteWise asset and adds it to the result row. This data is crucial for our analysis, because we will use it in our last step as dimensional data.</p> 
<pre><code class="lang-sql">
SELECT 
    from_unixtime(data.timeinseconds + (data.offsetinnanos / 1000000000)) ts, 
    metadata.assetname,  metadata.assetpropertyname, metadata.assetpropertydatatype, 
    data.doublevalue,  
    latesValue['Location'] as Location , latesValue['Manufacturer'] as Manufacturer, 
    latesValue['YearOfConstruction'] as YearOfConstruction, latesValue['Setpoint'] as Setpoint
FROM iotsitewise.raw  as data
-- Join the meta data table 
INNER JOIN iotsitewise.asset_metadata as metadata
ON data.seriesid = metadata.timeseriesid
-- Join sub query that retrieves all asset attributes and latest values    
LEFT JOIN (
  SELECT assetid, map_agg(assetpropertyname, latestvalue) latesValue from (
    SELECT assetid,  assetpropertyid, assetpropertyname,
            coalesce (
            max_by(data.stringvalue, data.timeinseconds),
            max_by(cast(data.integervalue as VARCHAR), data.timeinseconds),
            max_by(cast(data.doublevalue as  VARCHAR), data.timeinseconds)
            ) latestvalue
    FROM iotsitewise.raw data  
    INNER JOIN iotsitewise.asset_metadata metadata
    ON data.seriesid = metadata.timeseriesid
    GROUP BY assetid, assetpropertyid, assetpropertyname) 
  GROUP BY assetid) as dim
ON metadata.assetid = dim.assetid
WHERE data.startyear = year(current_date)
AND data.startmonth = month(current_date)
AND metadata.assetpropertyname = 'Last Holding Cycle Time'
</code></pre> 
<p>To test the query, choose <strong>Test query.</strong> If the query contains no syntax errors, you should see a preview of the data in the <strong>Result preview</strong> section.</p> 
<p><img alt="IoT Analytics Test Query" class="alignnone size-full wp-image-8014" height="1320" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/05/27/IoTAnalyticsTestQuery.png" width="2026" /></p> 
<p>Leave the rest of the steps 3-6 with the default value by choosing <strong>Next</strong> and choose <strong>Create dataset</strong> on the last review step 7.</p> 
<p>To validate if everything is correctly setup, navigate to your newly created dataset</p> 
<p><img alt="IoT Analytics Dataset Test" class="alignnone size-full wp-image-8006" height="1132" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/05/27/IoTAnalyticsDatasetTest.png" width="1794" /></p> 
<p>Choose <strong>Run now</strong> and wait until the result content appears on the <strong>Content</strong> tab with <code>Succeeded</code>. When you choose the Result link, the console shows you a preview of the query result:</p> 
<p><img alt="IoT Analytics Dataset Result" class="alignnone size-full wp-image-8007" height="246" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/05/27/IoTAnalyticsDatasetResult.png" width="2526" /></p> 
<p>The result shows the <code>Last Holding Cycle Time</code> metric by time. The query also added the AWS IoT SiteWise model information, like the asset name and model name, and the asset attribute values to each row. Such flattened data rows make it easier to analyze the data in BI tools. In the next step we will use Amazon QuickSight to analyze the IoT Analytics dataset.</p> 
<h3>Analyze the result in Amazon QuickSight</h3> 
<p>As a final step, we will analyze the data in Amazon QuickSight. Amazon QuickSight comes with a built-in connector for AWS IoT Analytics, so it’s easy to visualize the data.<br /> Open the <a href="https://quicksight.aws.amazon.com/sn/start">Amazon QuickSight console</a> and chose <strong>New Analysis</strong>, when prompted for a data sources, create a new one by choosing <strong>New Dataset</strong>. Choose AWS IoT Analytics and select the SiteWise AWS IoT Analytics dataset <code>holdingcycletimereport</code> we created in the previous step. To create the data source choose <strong>Create Data Source</strong>.</p> 
<table cellspacing="5"> 
 <tbody> 
  <tr> 
   <td><img alt="QuickSight Create Analysis" class="wp-image-8008 alignnone" height="307" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/05/27/QuickSightCreateAnalysis.png" width="706" /></td> 
   <td><img alt="" class="wp-image-8009 alignnone" height="327" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/05/27/QuickSightChooseDataset.png" width="643" /></td> 
  </tr> 
 </tbody> 
</table> 
<p>Choose <strong>Visualize</strong> to start using Amazon QuickSight Visual Types to display the data set.</p> 
<p><img alt="QuickSight Visualize" class=" wp-image-8010 alignnone" height="318" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/05/27/QuickSightVisualize.png" width="712" /></p> 
<p>In this specific use case, we want understand how the Manufacture and the Construction Year influences the HOLDING cycle duration, the Amazon QuickSight heat map is a good choice to visualize this data.</p> 
<p><img alt="QuickSigth Heat Map" class="alignnone wp-image-8011" height="551" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/05/27/QuickSigthHeatMap.png" width="787" /></p> 
<p>And from this view, we can clearly identify that the furnaces manufactured by Furnace Corp in 1999 have the longest cycle time (&gt;=88) and need to be prioritized for replacement.</p> 
<h2>Conclusion</h2> 
<p>This concludes the two part blog series on how to use AWS IoT SiteWise and AWS IoT Analytics to contextualize your industrial equipment data. We started by ingesting raw time series data into AWS IoT SiteWise. Next, we used the AWS IoT SiteWise asset model to add context about the industrial equipment that produced the time series data. Finally, we demonstrated how to use dataset queries in AWS IoT Analytics to combine the time series data points and context data into a flattened format that is easy to consume in BI tools like Amazon QuickSights.</p> 
<h2>About the author</h2> 
<p><img alt="" class="size-full wp-image-7982 alignleft" height="150" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/05/25/JanBorchSmall-1.jpeg" width="100" /></p> 
<p>Jan Borch is a Principal Specialist Solution Architect for IoT at Amazon Web Services (AWS) and spent the last 10 years helping customers design and build best-in-class cloud solutions on AWS. The last 5 years, he focused on the intersection of Cloud and IoT, leading the AWS IoT Prototyping Team to co-develop innovative connected IoT solutions with AWS customers in Europe, Middle East and Africa and recently his focal point shifted to customers with strategic IoT workloads on AWS.</p>
