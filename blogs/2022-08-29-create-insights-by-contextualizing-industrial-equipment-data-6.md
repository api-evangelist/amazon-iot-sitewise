---
title: "Create insights by contextualizing industrial equipment data using AWS IoT SiteWise (Part 1)"
url: "https://aws.amazon.com/blogs/iot/create-insights-by-contextualizing-industrial-equipment-data-using-aws-iot-sitewise-part-1/"
date: "Mon, 29 Aug 2022 06:26:24 +0000"
author: "Jan Borch"
feed_url: "https://aws.amazon.com/blogs/iot/tag/aws-iot-sitewise/feed/"
---
<h1>Introduction</h1> 
<p>More and more customers in the manufacturing industry want to collect data from machines and robots located in different facilities into a centralized AWS cloud-based IoT data lake. But the data produced by industrial equipment is often raw data points like temperature and pressure time series. Feeding those raw data streams directly into your industrial data lake, will make it difficult for your data analysts to get insights out of the ingested equipment data. A data analyst might need information that is not directly contained in the raw data streams to analyze the performance of industrial equipment. Metadata like the construction year, location or the manufacture of an equipment could have an impact on the performance metrics.</p> 
<p>AWS IoT SiteWise is a managed AWS service that simplifies collecting, organizing, and analyzing industrial equipment data and can help to contextualize the raw data streams captured from your industrial equipment using the AWS IoT SiteWise asset modeling capabilities. In part 1 of this blog series, and based on a fictional industrial use case, we will showcase how customer can use the asset modelling feature of AWS IoT Sitewise to manage such industrial equipment meta-data. And we will see how to use the AWS IoT SiteWise built-in library of operators and functions to perform real-time analytics to compute aggregated metrics. In part 2, we will show how we can export the ingested data to AWS IoT Analytics to perform complex batch analytics by combining the raw, meta and aggregated data to understand the root cause of an observed performance degradation.</p> 
<h2>Sample use case</h2> 
<p>To get you started, let’s consider a simple industrial scenario where the goal is to remotely monitor industrial furnaces. Your company owns furnaces across different production sites that perform the same industrial process like e.g. annealing metal workpieces. You’ve noticed a difference in manufacturing time and quality across your production sites.<br /> You want to model your furnace in AWS IoT SiteWise with the following properties, and you use AWS IoT SiteWise Edge to collect those data points e.g over Modbus TPC from your furnaces.</p> 
<table border="1" style="height: 85px; border-color: #363434; border-collapse: collapse;"> 
 <tbody> 
  <tr style="background-color: #dbd9d9;"> 
   <td><strong>Furnace Asset Model</strong></td> 
  </tr> 
  <tr style="background-color: #dbd9d9;"> 
   <td><strong>Property Name</strong></td> 
   <td><strong>Property Type</strong></td> 
   <td><strong>Property Value Type</strong></td> 
   <td><strong>Unit</strong></td> 
   <td><strong>Sample Data</strong></td> 
  </tr> 
  <tr> 
   <td>Furnace location</td> 
   <td>ATTRIBUTE</td> 
   <td>STRING</td> 
   <td>none</td> 
   <td>Paris factory, Chicago factory</td> 
  </tr> 
  <tr> 
   <td>Furnace manufacturer</td> 
   <td>ATTRIBUTE</td> 
   <td>STRING</td> 
   <td>none</td> 
   <td>Furnace Corp, Heat&amp;Metal Corp</td> 
  </tr> 
  <tr> 
   <td>Furnace temp set point</td> 
   <td>ATTRIBUTE</td> 
   <td>INT</td> 
   <td>C˚</td> 
   <td>760</td> 
  </tr> 
  <tr> 
   <td>Furnace construction year</td> 
   <td>ATTRIBUTE</td> 
   <td>INT</td> 
   <td>Year</td> 
   <td>1999</td> 
  </tr> 
  <tr> 
   <td>Current Kw Power Consumption</td> 
   <td>MEASUREMENT</td> 
   <td>DOUBLE</td> 
   <td>kW</td> 
   <td>51</td> 
  </tr> 
  <tr> 
   <td>Current furnace temperature</td> 
   <td>MEASUREMENT</td> 
   <td>DOUBLE</td> 
   <td>C˚</td> 
   <td>399</td> 
  </tr> 
  <tr> 
   <td>The Furnace state</td> 
   <td>MEASUREMENT</td> 
   <td>STRING</td> 
   <td>none</td> 
   <td>IDLE, HEATING,HOLDING, COOLING</td> 
  </tr> 
  <tr> 
   <td>Last HOLDING cycle duration</td> 
   <td>TRANSFORMATION</td> 
   <td>DOUBLE</td> 
   <td>Duration in s</td> 
   <td>4h5m3s</td> 
  </tr> 
  <tr> 
   <td>Avg Holding cycle last 24h</td> 
   <td>METRIC(1day)</td> 
   <td>DOUBLE</td> 
   <td>Duration in s</td> 
   <td>4h5m3s</td> 
  </tr> 
 </tbody> 
</table> 
<p>You have a suspicion that the efficiency issue is linked to the heterogeneous machine park, so you want to compare the heating and holding duration across all furnaces grouped by manufacture and construction year. The next section shows you step-by-step instructions on how to use AWS IoT SiteWise and AWS IoT Analytics to generate the desired report.</p> 
<h2>Model and create an industrial asset in AWS IoT SiteWise</h2> 
<p>The first section explains on a high level how to create the furnace asset model in AWS IoT SiteWise. For details on how to model industrial assets in AWS IoT SiteWise, see <a href="https://docs.aws.amazon.com/iot-sitewise/latest/userguide/industrial-asset-models.html">Modeling industrial assets</a>.</p> 
<h3>Create a furnace asset model</h3> 
<p>Sign in to the AWS Management Console and navigate to the AWS IoT SiteWise console.<br /> On the navigation bar, choose <strong>Build</strong>, <strong>Model</strong> to create a new Model, call it <code>Furnace</code> and define the static attributes and default value as describe in the table before:</p> 
<p><img alt="SiteWise Attribute Definition" class="alignnone size-full wp-image-7962" height="274" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/05/25/SiteWiseAttributeDefinition.png" width="930" /></p> 
<p>Next define the asset model measurement as depicted below. The furnace operates in four different processing states <code>State</code> moving from <code>IDLE</code> to <code>HEATING</code>, over <code>HOLDING</code> and <code>COOLING</code>. The <code>Temperature</code> measurement shows the current furnace temperature and <code>Power</code> the current power consumption in kW.</p> 
<p><img alt="SiteWise Measurement Definition" class="alignnone size-full wp-image-7963" height="244" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/05/25/SiteWiseMeasurementDefinition.png" width="931" /></p> 
<p>The next step is to define AWS IoT SiteWise transforms to perform computation on the raw measurements. We use some advanced temporal AWS IoT SiteWise functions here to detect the state change from <code>HOLDING</code> to <code>COOLING</code> and store the <code>HOLDING</code> cycle duration into the Metric <code>Last Holding Cycle Time</code> . The formula below is triggered when the <code>State</code> measurement changes value and the previous value was <code>HOLDING</code>: <code>if(pretrigger(State)=="Holding", ...</code> . In this situation, it computes the duration of the holding time by subtracting the current change timestamp from the previous change timestamp: <code>timestamp(State) - timestamp(pretrigger(State)</code>. To learn more about AWS IoT SiteWise temporal functions, see <a href="https://docs.aws.amazon.com/iot-sitewise/latest/userguide/expression-temporal-functions.html">Temporal functions</a></p> 
<p><img alt="SiteWise Transformation Definition" class="alignnone size-full wp-image-7964" height="195" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/05/25/SiteWiseTransformationDefinitions.png" width="1120" /><br /> A furnace operator might be interested in monitoring the evolution of the holding cycle duration over time. To do so, let’s create a last metric to calculate the average <code>Last Holding Cycle Time</code> for a time window of 5-minute, in a real scenario a daily roll-up might be more appropriate to compare variations over a longer time period.</p> 
<p><img alt="SiteWise Metric Definition" class="alignnone size-full wp-image-7965" height="176" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/05/25/SiteWiseMetricDefinition.png" width="959" /><br /> AWS IoT SiteWise allows users to define asset model hierarchies to create logical associations between the asset models in your industrial operation. As a last step, create a model named <code>Factory</code> to represent a factory and create a hierarchy definition pointing to the <code>Furance</code> model. A factory will later on, through a hierarchical structure, represent a group of furnaces placed in one production site. We will use this hierarchy later in AWS IoT SiteWise Monitor to visualize furnace performance metrics within a factory on a dashboard.</p> 
<p><img alt="SiteWise Hierarchy Definition" class="alignnone size-full wp-image-7966" height="181" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/05/25/SiteWiseHierarchyDefinition.png" width="924" /></p> 
<h3>Create the furnace assets</h3> 
<p>Create assets based on the <code>Furnace</code> model by choosing <strong>Build</strong>, <strong>Assets</strong> in the navigation bar and choose <strong>Create asset</strong>. Create for example one <code>Factory</code> Asset named <code>Paris Factory</code> and 4 attached <code>Furnace</code> assets and populate the static asset attributes with random data of your choice.</p> 
<p><img alt="SiteWise Furnace Asset" class="alignnone size-full wp-image-7967" height="773" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/05/25/SiteWiseFurnaceAsset.png" width="1133" /></p> 
<p>This concludes the Asset modelling and creation part, and we can now start analyzing the data captured by AWS IoT SiteWise. In the next section, we will show you how to leverage the built-in AWS IoT SiteWise time-series optimized data store to monitor our furnaces in real-time.</p> 
<h2>Analyzing the near-real time data using AWS IoT Sitewise</h2> 
<p>To test our AWS IoT SiteWise assets, we need to generate some sample data for the furnace temperature, power and state measurements. In this blog post we don’t connect to a real Modbus data source but use a Python based data simulator that you can run on your laptop:&nbsp;<a href="https://github.com/aws-samples/sitewise-iiot-data-simulator">https://github.com/aws-samples/sitewise-iiot-data-simulator</a> . Follow the instructions in the README file to install and run the simulator.</p> 
<p>AWS IoT SiteWise Monitor is an easy way to visualize the measurements, transformations and metrics we defined in our Asset Model. The following screen capture shows what an operational dashboard could look like to compare the performance of two Furnaces in a Factory. AWS IoT SiteWise Monitor allows you to create no-code fully managed web applications by using drag and drop the asset model properties onto the dashboard. This blog post leaves it to the discretion of the reader to design their own dashboard. To get you started, here are some of the widgets we used to create the dashboard depicted below. The dashboard uses the timeline widget to visualize the current and previous state transitions, the line chart to plot the temperature and power consumption and a bar chart to depict the last HOLDING cycle time duration. Several KPI widgets allow operators to have quick glance at key Furnace KPIs. To learn more on how to set up an AWS IoT SiteWise Monitor Dashboard, see <a href="https://docs.aws.amazon.com/iot-sitewise/latest/userguide/monitor-getting-started.html">Getting started with AWS IoT SiteWise Monitor</a>.</p> 
<p><img alt="SiteWise Monitor Dashboard" class="alignnone size-full wp-image-7968" height="1039" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/05/25/SiteWiseMonitorDashboard.png" width="1334" /></p> 
<p>Using the AWS IoT SiteWise Monitor dashboard, we can clearly identify that the <code>Avg. Holding Cycle time</code> metric for <code>Furnace001</code> is longer (87s vs 76.5s) than for <code>Furnace002</code>.<br /> The holding time is also higher compared to the average (82s) across all furnaces in the Paris factory. But a more in-depth analysis is needed to understand the root cause of this discrepancy.</p> 
<h2>Clean up</h2> 
<p>Make sure you stop the furnace data simulator to avoid incurring ongoing charges.</p> 
<h2>Conclusion</h2> 
<p>This concludes the first part of this blog series. In this part we reviewed how AWS IoT SiteWise can be used to enrich raw industrial data streams, perform real-time analytics to detect industrial process boundaries and compute process level metrics like cycle duration and moving averages. Since the dashboard doesn’t allow for direct insights into the cause for the difference in the <code>Avg. Holding Cycle time</code>, we will use the second blog post in this series to dive deeper. In the second part of this blog, we will showcase how we can leverage the AWS IoT SiteWise cold tier storage feature to export the collected historical data to Amazon S3 and use AWS IoT Analytics to perform the root cause analysis and understand what contributes to the low performance of <code>Furnace001</code>.</p> 
<h2>About the author</h2> 
<p><img alt="" class="size-full wp-image-7982 alignleft" height="150" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/05/25/JanBorchSmall-1.jpeg" width="100" /><br /> Jan Borch is a Principal Specialist Solution Architect for IoT at Amazon Web Services (AWS) and has spent the last 10 years helping customers design and build best-in-class cloud solutions on AWS. The last 5 years, he has focused on the intersection of Cloud and IoT, leading the AWS IoT Prototyping Team to co-develop innovative connected IoT solutions with AWS customers in Europe, Middle East and Africa and recently his focal point shifted to customers with strategic IoT workloads on AWS.</p>
