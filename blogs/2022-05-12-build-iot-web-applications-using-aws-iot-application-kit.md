---
title: "Build IoT web applications using AWS IoT Application Kit"
url: "https://aws.amazon.com/blogs/iot/build-iot-applications-using-aws-iot-application-kit/"
date: "Thu, 12 May 2022 20:59:02 +0000"
author: "Sudhir Jena"
feed_url: "https://aws.amazon.com/blogs/iot/tag/aws-iot-sitewise/feed/"
---
<p><span style="font-size: 16px;">On Mar 1, 2022, </span><a href="https://aws.amazon.com/about-aws/whats-new/2022/03/aws-iot-sitewise-development-library-web-industrial-data/" style="font-size: 16px;">we announced AWS IoT Application Kit</a><span style="font-size: 16px;">, an open-source UI components library for IoT application developers. With AWS IoT Application Kit, developers can build rich interactive web applications leveraging data from </span><a href="https://aws.amazon.com/iot-sitewise/" style="font-size: 16px;">AWS IoT SiteWise</a><span style="font-size: 16px;">. IoT application developers can deliver customized user experiences like industrial asset monitoring applications using web front-end frameworks like ReactJS, Vue.js or vanilla JavaScript along with reusable components from AWS IoT Application Kit.</span></p> 
<div class="wp-caption aligncenter" id="attachment_7713" style="width: 1034px;">
 <a href="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/05/09/aws-iot-app-kit-demo-screely.png"><img alt="Screenshot of sample ReactJS application built with AWS IoT Application Kit" class="wp-image-7713 size-large" height="563" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/05/09/aws-iot-app-kit-demo-screely-1024x563.png" width="1024" /></a>
 <p class="wp-caption-text" id="caption-attachment-7713">Figure 1: Screenshot of sample ReactJS application built with AWS IoT Application Kit</p>
</div> 
<h2>What is AWS IoT Application Kit?</h2> 
<p>AWS IoT Application Kit is an open-source, client-side library that enables IoT application developers to simplify the development of complex IoT applications. It leverages performant, reusable components that abstract critical technical considerations expected from a real-time web application; for example, handling streaming data, caching, preloading data, dynamic aggregation, and preventing request fragmentation. This abstraction allows IoT application developers to focus on building custom user experiences and worry less about underlying technical complexities.</p> 
<p>In cases where customers require integrating and enriching their&nbsp;existing web applications for visualizing IoT data from AWS IoT SiteWise, AWS IoT Application Kit also allows customers to integrate the included components into their existing web application.</p> 
<h2>Getting started with AWS IoT Application Kit</h2> 
<p>AWS IoT Application Kit is currently available as a npm package – <a href="https://www.npmjs.com/package/@iot-app-kit/components"><code>@iot-app-kit/components</code></a>. You can install this package with:</p> 
<h3>Using npm</h3> 
<pre><code class="lang-bash">npm install @iot-app-kit/components</code></pre> 
<p>For additional details, please refer to the <a href="https://github.com/awslabs/iot-app-kit">technical documentation</a> for AWS IoT Application Kit.</p> 
<h2>Building with AWS IoT Application Kit</h2> 
<p>In this blog post, we’ll build a ReactJS web application with AWS IoT Application Kit and AWS IoT SiteWise for monitoring an industrial juice bottling line, displaying the telemetry (such as Machine Status and Production Count) from each of the constituent machines in the bottling line.</p> 
<h2>Walkthrough</h2> 
<h3>Prerequisites</h3> 
<p>The following is required to build this solution:</p> 
<ul> 
 <li><a href="https://aws.amazon.com/cli/">AWS CLI</a></li> 
 <li><a href="https://docs.aws.amazon.com/cdk/v2/guide/getting_started.html">AWS CDK</a></li> 
 <li>An AWS CLI profile with permissions to deploy stacks via AWS CloudFormation</li> 
 <li>A default VPC present in your AWS account</li> 
</ul> 
<h3>Step 1: Simulate telemetry of an industrial bottling line</h3> 
<p>The industrial juice bottling line we want to model is comprised of the following interconnected machines (in order):</p> 
<table border="1" cellpadding="4" cellspacing="0" style="border-color: #ededed; border-collapse: collapse;"> 
 <caption>
  Table 1: Ordered list of interconnected machines in simulated industrial juice bottling line
 </caption> 
 <thead> 
  <tr style="background-color: #e0e0e0;"> 
   <td style="width: 67px;" width="67"><strong>Order</strong></td> 
   <td width="187"><strong>Machine Name</strong></td> 
   <td style="text-align: center;" width="93"><strong>Machine ID</strong></td> 
   <td width="584"><strong>Description</strong></td> 
  </tr> 
 </thead> 
 <tbody> 
  <tr> 
   <td>1st</td> 
   <td>Washing Machine</td> 
   <td style="text-align: center;">UN01</td> 
   <td>Washes, sanitizes and dries each incoming empty bottle.</td> 
  </tr> 
  <tr style="background-color: #f7f7f7;"> 
   <td>2nd</td> 
   <td>Filling Machine</td> 
   <td style="text-align: center;">UN02</td> 
   <td>Fills each incoming sanitized bottle to the configured quantity.</td> 
  </tr> 
  <tr> 
   <td>3rd</td> 
   <td>Capping Machine</td> 
   <td style="text-align: center;">UN03</td> 
   <td>Caps and seals each incoming filled bottle.</td> 
  </tr> 
  <tr style="background-color: #f7f7f7;"> 
   <td>4th</td> 
   <td>Labelling Machine</td> 
   <td style="text-align: center;">UN04</td> 
   <td>Attaches and prints the product label on each capped bottle.</td> 
  </tr> 
  <tr> 
   <td>5th</td> 
   <td>Case Packing Machine</td> 
   <td style="text-align: center;">UN05</td> 
   <td>Packs configured group of labelled bottles into a single case.</td> 
  </tr> 
  <tr style="background-color: #f7f7f7;"> 
   <td>6th</td> 
   <td>Palletizing Machine</td> 
   <td style="text-align: center;">UN06</td> 
   <td>Palletizes multiple cases of processed bottles into a pallet for shipment.</td> 
  </tr> 
 </tbody> 
</table> 
<div class="wp-caption aligncenter" id="attachment_7710" style="width: 3246px;">
 <a href="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/05/09/bottling-line-illustration.jpg"><img alt="Bottline line illustration" class="wp-image-7710 size-full" height="1978" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/05/09/bottling-line-illustration.jpg" width="3236" /></a>
 <p class="wp-caption-text" id="caption-attachment-7710">Figure 2: Representation of machines in the industrial bottling line simulated with this demo</p>
</div> 
<p>Each of these machines emits the following data measurements as telemetry:</p> 
<table border="1" cellpadding="4" cellspacing="0" style="border-color: #ededed; border-collapse: collapse;"> 
 <caption>
  Table 2: List of modeled OPC-UA tags
 </caption> 
 <thead> 
  <tr style="background-color: #e0e0e0;"> 
   <td width="159"><strong>Measurement Name</strong></td> 
   <td style="text-align: center;" width="147"><strong>Measurement Unit</strong></td> 
   <td width="85"><strong>Data Type</strong></td> 
   <td width="288"><strong>Modeled Tag</strong></td> 
   <td width="700"><strong>Description</strong></td> 
  </tr> 
 </thead> 
 <tbody> 
  <tr> 
   <td width="159">Machine State</td> 
   <td style="text-align: center;" width="147">None</td> 
   <td style="text-align: center;" width="85">Integer</td> 
   <td width="288">{Machine_ID}/Status/StateCurrent</td> 
   <td width="700">Current operational state of the machine. Possible values are listed in Table 3: Machine States Description.</td> 
  </tr> 
  <tr style="background-color: #f7f7f7;"> 
   <td width="159">Machine Mode</td> 
   <td style="text-align: center;" width="147">None</td> 
   <td style="text-align: center;" width="85">Integer</td> 
   <td width="288">{Machine_ID}/Status/ModeCurrent</td> 
   <td width="700">The mode under which the machine is operating. Possible values are listed in Table 4: Machine Operating Modes.</td> 
  </tr> 
  <tr> 
   <td width="159">Current Speed</td> 
   <td style="text-align: center;" width="147">Bottles per minute</td> 
   <td style="text-align: center;" width="85">Double</td> 
   <td width="288">{Machine_ID}/Status/CurMachSpeed</td> 
   <td width="700">Current operational speed of the machine measured in bottles processed per minute.</td> 
  </tr> 
  <tr style="background-color: #f7f7f7;"> 
   <td width="159">Blocked</td> 
   <td style="text-align: center;" width="147">None</td> 
   <td style="text-align: center;" width="85">Boolean</td> 
   <td width="288">{Machine_ID}/Status/Blocked</td> 
   <td width="700">Indicating whether the machine is blocked from operating due to downstream machine(s) conditions.</td> 
  </tr> 
  <tr> 
   <td width="159">Starved</td> 
   <td style="text-align: center;" width="147">None</td> 
   <td style="text-align: center;" width="85">Boolean</td> 
   <td width="288">{Machine_ID}/Status/Starved</td> 
   <td width="700">Indicating whether the machine is starved from operating due to upstream intake conditions.</td> 
  </tr> 
  <tr style="background-color: #f7f7f7;"> 
   <td width="159">Stop Reason</td> 
   <td style="text-align: center;" width="147">None</td> 
   <td style="text-align: center;" width="85">Integer</td> 
   <td width="288">{Machine_ID}/Admin/StopReasonCode</td> 
   <td width="700">Machine Stop Reason Code.</td> 
  </tr> 
  <tr> 
   <td width="159">Processed Count</td> 
   <td style="text-align: center;" width="147">None</td> 
   <td style="text-align: center;" width="85">Integer</td> 
   <td width="288">{Machine_ID}/Admin/ProcessedCount</td> 
   <td width="700">Incremental counter of bottles processed by the machine, either successfully or unsuccessfully.</td> 
  </tr> 
  <tr style="background-color: #f7f7f7;"> 
   <td width="159">Defective Count</td> 
   <td style="text-align: center;" width="147">None</td> 
   <td style="text-align: center;" width="85">Integer</td> 
   <td width="288">{Machine_ID}/Admin/DefectiveCount</td> 
   <td width="700">Incremental counter of bottles processed unsuccessfully by the machine.</td> 
  </tr> 
 </tbody> 
</table> 
<table border="1" cellpadding="4" cellspacing="0" style="border-color: #ededed; border-collapse: collapse; width: 100%;"> 
 <caption>
  Table 3: Machine States Description
 </caption> 
 <thead> 
  <tr style="background-color: #e0e0e0;"> 
   <td style="text-align: center;" width="157"><strong>StateCurrent Values</strong></td> 
   <td style="text-align: center;" width="177"><strong>Implied Machine State</strong></td> 
  </tr> 
 </thead> 
 <tbody> 
  <tr> 
   <td style="text-align: center;">1</td> 
   <td style="text-align: center;">PRODUCING</td> 
  </tr> 
  <tr style="background-color: #f7f7f7;"> 
   <td style="text-align: center;">2</td> 
   <td style="text-align: center;">IDLE</td> 
  </tr> 
  <tr> 
   <td style="text-align: center;">3</td> 
   <td style="text-align: center;">STARVED</td> 
  </tr> 
  <tr style="background-color: #f7f7f7;"> 
   <td style="text-align: center;">4</td> 
   <td style="text-align: center;">BLOCKED</td> 
  </tr> 
  <tr> 
   <td style="text-align: center;">5</td> 
   <td style="text-align: center;">CHANGEOVER</td> 
  </tr> 
  <tr style="background-color: #f7f7f7;"> 
   <td style="text-align: center;">6</td> 
   <td style="text-align: center;">STOPPED</td> 
  </tr> 
  <tr> 
   <td style="text-align: center;">7</td> 
   <td style="text-align: center;">FAULTED</td> 
  </tr> 
 </tbody> 
</table> 
<table border="1" cellpadding="4" cellspacing="0" style="border-color: #ededed; border-collapse: collapse; width: 100%;"> 
 <caption>
  Table 4: Machine Operating Modes
 </caption> 
 <thead> 
  <tr style="background-color: #e0e0e0;"> 
   <td style="text-align: center;" width="157"><strong>ModeCurrent Values</strong></td> 
   <td style="text-align: center;" width="177"><strong>Implied Machine Mode</strong></td> 
  </tr> 
 </thead> 
 <tbody> 
  <tr> 
   <td style="text-align: center;">1</td> 
   <td style="text-align: center;">AUTOMATIC</td> 
  </tr> 
  <tr style="background-color: #f7f7f7;"> 
   <td style="text-align: center;">2</td> 
   <td style="text-align: center;">MAINTENANCE</td> 
  </tr> 
  <tr> 
   <td style="text-align: center;">3</td> 
   <td style="text-align: center;">MANUAL</td> 
  </tr> 
 </tbody> 
</table> 
<p>We will use <a href="https://nodered.org/">Node-RED</a> hosted on an <a href="https://aws.amazon.com/ec2/">Amazon EC2</a> instance to create a flow which simulates an OPC-UA server allowing to read the modeled tags mentioned in Table 2: List of modeled OPC-UA tags for each of the machines in the industrial juice bottling line. To quickly setup the Node-RED environment, clone the accompanying AWS CDK infrastructure as code from github.</p> 
<ul> 
 <li>Clone the application to your local machine.</li> 
</ul> 
<pre><code class="lang-bash">git clone https://github.com/aws-samples/aws-iot-app-kit-bottling-line-demo.git iot-app-kit-demo</code></pre> 
<ul> 
 <li>Change to the project directory.</li> 
</ul> 
<pre><code class="lang-bash">cd iot-app-kit-demo</code></pre> 
<ul> 
 <li>Install dependencies for the AWS CDK. Note, this is for the infrastructure only.</li> 
</ul> 
<pre><code class="lang-bash">npm ci</code></pre> 
<ul> 
 <li>Configure your account and region for CDK deployment<br /> <span style="color: #c0392b;">Note</span><strong>:</strong> Please use an AWS region where AWS IoT SiteWise is <a href="https://docs.aws.amazon.com/general/latest/gr/iot-sitewise.html#iot-sitewise_region">available</a>.</li> 
</ul> 
<pre><code class="lang-bash">cdk bootstrap aws://&lt;ACCOUNT-NUMBER&gt;/&lt;REGION&gt;</code></pre> 
<ul> 
 <li>Deploy the cdk stack named OpcuaSimulatorStack. When prompted with “Do you wish to deploy these changes (y/n)?” Enter Y.</li> 
</ul> 
<pre><code class="lang-bash">cdk deploy OpcuaSimulatorStack</code></pre> 
<div class="wp-caption aligncenter" id="attachment_7680" style="width: 1034px;">
 <a href="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/05/08/blog-arch-diagram.jpg"><img alt="Iot Application Kit Bottling Line Demo - AWS Architecture Diagram" class="wp-image-7680 size-full" height="576" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/05/08/blog-arch-diagram.jpg" width="1024" /></a>
 <p class="wp-caption-text" id="caption-attachment-7680">Figure 3: Architecture diagram of AWS IoT Application Kit Bottling Line Demo</p>
</div> 
<p>Successful deployment of the <strong>OpcuaSimulatorStack </strong>should create an OPC-UA server, AWS IoT Greengrass V2 core, a corresponding AWS IoT SiteWise gateway along with asset models and derived assets (representing the machines in the juice bottling line). All of the application components i.e., OPC-UA Server, AWS IoT Greengrass V2 core and AWS IoT SiteWise gateway are deployed in an Ubuntu EC2 Instance created through the&nbsp;OpcuaSimulatorStack.</p> 
<p>Deploying the&nbsp;<strong>OpcuaSimulatorStack </strong>should take a few minutes and will be indicated by the output of the cdk deploy&nbsp;command. In Step 2,&nbsp;we will be building a&nbsp;ReactJS web application to monitor the assets created for the juice bottling line.</p> 
<h3>Step 2: Build a custom application to visualize the industrial bottling line operation</h3> 
<p>The cloned code repository <code>aws-iot-app-kit-bottling-line-demo.git</code> contains a starter ReactJS application in the directory named <code>assets/react-app</code>. In this step, we will be adding our AWS IoT Application Kit components to the starter ReactJS application in incremental steps.</p> 
<ul> 
 <li>Change to the ReactJS application directory.</li> 
</ul> 
<pre><code class="lang-bash">cd assets/react-app</code></pre> 
<ul> 
 <li>Install required NPM dependencies</li> 
</ul> 
<pre><code class="lang-bash">npm ci</code></pre> 
<ul> 
 <li>Create a <code>.env</code> file in the root directory of the react-app i.e., <code>assets/react-app/.env</code></li> 
</ul> 
<pre><code class="lang-bash">touch .env</code></pre> 
<ul> 
 <li>Edit the <code>.env</code> file and add your AWS IAM credentials for programmatic access as environment variables prefixed with <code>REACT_APP_</code> as shown in the snippet. The value for <code>REACT_APP_AWS_SESSION_TOKEN</code> is only required if you are using short-lived IAM credentials for programmatic access.</li> 
</ul> 
<pre><code class="lang-bash">REACT_APP_AWS_ACCESS_KEY_ID=&lt;replace-with-aws-access-key-id&gt;
REACT_APP_AWS_SECRET_ACCESS_KEY=&lt;replace-with-aws-access-key&gt;
REACT_APP_AWS_SESSION_TOKEN=&lt;replace-with-aws-session-token&gt;
</code></pre> 
<ul> 
 <li>Save the <code>.env</code> file after editing.</li> 
</ul> 
<p>From here, we will begin adding&nbsp;AWS IoT Application Kit components one by one to demonstrate the usage of each component.</p> 
<ul> 
 <li>Add AWS IoT Application Kit NPM packages to ReactJS application dependencies.</li> 
</ul> 
<pre><code class="lang-bash">npm install @iot-app-kit/components @iot-app-kit/react-components @iot-app-kit/source-iotsitewise</code></pre> 
<ul> 
 <li>Open and edit <code>src/App.tsx</code> to import installed&nbsp;AWS IoT Application Kit components between the comment lines <code>/* --- BEGIN: AWS @iot-app-kit and related imports*/</code>and <code>/* --- END: AWS @iot-app-kit and related imports*/</code>&nbsp;as shown below. Replace the value of <code>awsRegion</code>&nbsp;with the actual AWS region (where&nbsp;<strong>OpcuaSimulatorStack </strong>was&nbsp;deployed in Step 1).</li> 
</ul> 
<pre><code class="lang-ts">...
/* --- BEGIN: AWS @iot-app-kit and related imports*/
import { initialize } from "@iot-app-kit/source-iotsitewise";
import { fromEnvReactApp } from "./fromEnv";
import {
    BarChart,
    LineChart,
    StatusTimeline,
    ResourceExplorer,
    WebglContext,
    StatusGrid,
    Kpi,
} from "@iot-app-kit/react-components";
import { COMPARISON_OPERATOR } from "@synchro-charts/core";

import "./App.css";

const { defineCustomElements } = require("@iot-app-kit/components/loader");

const { query } = initialize({
    awsCredentials: fromEnvReactApp(),
    awsRegion: "&lt;replace-with-aws-region&gt;",
});

defineCustomElements();
/* --- END: AWS @iot-app-kit and related imports*/
...</code></pre> 
<ul> 
 <li>Refer to the AWS IoT SiteWise console&nbsp;to populate the respective asset property ids between the comment lines <code>/* --- BEGIN: Asset Id and Asset Property Ids from AWS IoT SiteWise*/</code>&nbsp;and&nbsp;<code>/* --- END: Asset Property Ids from AWS IoT SiteWise*/</code> that need to be displayed with AWS IoT Application Kit</li> 
</ul> 
<pre><code class="lang-ts">...
/* --- BEGIN: Asset Id and Asset Property Ids from AWS IoT SiteWise*/
    
// Asset Id of the AWS IoT SiteWise asset that you want to display by // default
const DEFAULT_MACHINE_ASSET_ID = '&lt;replace-with-sitwise-asset-id&gt;';
const [ assetId, setAssetId ] = useState(DEFAULT_MACHINE_ASSET_ID);
const [ assetName, setAssetName ] = useState('&lt;replace-with-corresponding-sitwise-asset-name&gt;');
    
// Asset Property Ids of the AWS IoT SiteWise assets that you want to // query data for

// Refer AWS IoT SiteWise measurements
const OEE_BAD_COUNT_PROPERTY = '&lt;replace-with-corresponding-sitwise-asset-property-id&gt;';
const OEE_TOTAL_COUNT_PROPERTY = '&lt;replace-with-corresponding-sitwise-asset-property-id&gt;';
const CURRENT_SPEED_PROPERTY = '&lt;replace-with-corresponding-sitwise-asset-property-id&gt;';
const MACHINE_STOP_REASON_CODE_PROPERTY = '&lt;replace-with-corresponding-sitwise-asset-property-id&gt;';

// Refer IoT SiteWise transforms
const MACHINE_STATE_ENUM_PROPERTY = '&lt;replace-with-corresponding-sitwise-asset-property-id&gt;';
const MACHINE_MODE_ENUM_PROPERTY = '&lt;replace-with-corresponding-sitwise-asset-property-id&gt;';
const STARVED_INDICATOR_PROPERTY = '&lt;replace-with-corresponding-sitwise-asset-property-id&gt;';
const BLOCKED_INDICATOR_PROPERTY = '&lt;replace-with-corresponding-sitwise-asset-property-id&gt;';
    

/* --- END: Asset Property Ids from AWS IoT SiteWise*/
...
</code></pre> 
<ul> 
 <li>Since we have several assets in our juice bottling line, let us first implement the&nbsp;<strong>ResourceExplorer</strong> component to allow&nbsp;filtering, sorting, and pagination of our assets. Add the following code between the comment lines <code>{/* --- BEGIN: `ResourceExplorer` implementation*/}</code> and <code>{/* --- END: `ResourceExplorer` implementation*/}</code> in <code>src/App.tsx</code></li> 
</ul> 
<pre><code class="lang-markup">...
{/* --- BEGIN: `ResourceExplorer` implementation*/}
&lt;ResourceExplorer
    query={query.assetTree.fromRoot()}
    onSelectionChange={(event) =&gt; {
        console.log("changes asset", event);
        props.setAssetId((event?.detail?.selectedItems?.[0] as any)?.id);
        props.setAssetName((event?.detail?.selectedItems?.[0] as any)?.name);
                }}
    columnDefinitions={columnDefinitions}
/&gt;
{/* --- END: `ResourceExplorer` implementation*/}
...
</code></pre> 
<ul> 
 <li>Next, we will implement&nbsp;<strong>StatusTimeline</strong> component to visualize the&nbsp;<strong>Machine State</strong> asset property&nbsp;of our various assets. Add the following code between the comment lines&nbsp;&nbsp;<code>{/* --- BEGIN: `StatusTimeline` implementation*/}</code> and&nbsp;<code>{/* --- END: `StatusTimeline` implementation*/}</code>.</li> 
</ul> 
<pre><code class="lang-markup">...
{/* --- BEGIN: `StatusTimeline` implementation*/}
 &lt;div style={{ height: "170px" }}&gt;
    &lt;StatusTimeline
        viewport={{ duration: '15m' }}
        annotations={{
            y: [
                { color: '#1D8102', comparisonOperator: COMPARISON_OPERATOR.EQUAL, value: 'PRODUCING' },
                { color: '#0073BB', comparisonOperator: COMPARISON_OPERATOR.EQUAL, value: 'IDLE' },
                { color: '#D45200', comparisonOperator: COMPARISON_OPERATOR.EQUAL, value: 'STARVED' },
                { color: '#DA4976', comparisonOperator: COMPARISON_OPERATOR.EQUAL, value: 'BLOCKED' },
                { color: '#5951D6', comparisonOperator: COMPARISON_OPERATOR.EQUAL, value: 'CHANGEOVER' },
                { color: '#455A64', comparisonOperator: COMPARISON_OPERATOR.EQUAL, value: 'STOPPED' },
                { color: '#AB1D00', comparisonOperator: COMPARISON_OPERATOR.EQUAL, value: 'FAULTED' }
            ]
        }}
        queries={[
            query.timeSeriesData({
                assets: [{
                    assetId: props.assetId,
                    properties: [{
                        propertyId: props.machineStatePropertyId
                    }]
                }]
            })
        ]}
    /&gt;
&lt;/div&gt;
{/* --- END: `StatusTimeline` implementation*/}
...
</code></pre> 
<ul> 
 <li>Next, we will implement a&nbsp;<strong>LineChart </strong>component to visualize the following metrics defined in AWS IoT SiteWise for each of the machines in the juice bottling line: 
  <ul> 
   <li><strong>Total Count</strong> of bottles processed every 15 minutes</li> 
   <li><strong>Bad Count</strong> of bottles processed every 15 minutes</li> 
  </ul> </li> 
</ul> 
<p style="padding-left: 40px;">Add the following code between the comment lines <code>{/* --- BEGIN: `LineChart` implementation*/}</code> and&nbsp;<code>{/* --- END: `LineChart` implementation*/}</code>.</p> 
<pre><code class="lang-markup">...
{/* --- BEGIN: `LineChart` implementation*/}
&lt;div style={{ height: "170px" }}&gt;
    &lt;LineChart
        viewport={{ duration: "15m" }}
        queries={[
            query.timeSeriesData({
                assets: [
                    {
                        assetId: props.assetId,
                        properties: [
                            {
                                propertyId: props.badPartsCountPropertyId,
                                refId: "bad-parts-count",
                            },
                            {
                                propertyId: props.totalPartsCountPropertyId,
                                refId: "total-parts-count",
                            },
                        ],
                    },
                ],
            }),
        ]}
        styleSettings={{
            "bad-parts-count": { color: "#D13212", name: "Bad Count" },
            "total-parts-count": { color: "#1D8102", name: "Total Count" },
        }}
    /&gt;
&lt;/div&gt;
{/* --- END: `LineChart` implementation*/}
...
</code></pre> 
<ul> 
 <li>Add <strong>WebglContext </strong>component&nbsp;between the comment lines&nbsp;<code>{/* --- BEGIN: `WebglContext` implementation*/}</code> and&nbsp;<code>{/* --- END: `WebglContext` implementation*/}</code>.<br /> <span style="color: #c0392b;">Note</span><strong>: </strong><strong>WebglContext</strong> <strong>should be</strong> <strong>declared only once</strong> throughout your ReactJS component tree.</li> 
</ul> 
<pre><code class="lang-markup">...
{/* --- BEGIN: `WebglContext` implementation*/}
&lt;WebglContext/&gt;
{/* --- END: `WebglContext` implementation*/}
...
</code></pre> 
<ul> 
 <li>Start a local development server and view the revised ReactJS application&nbsp;by navigating to <code>http://localhost:3000</code>. Once launched, browse through the juice bottling line asset hierarchy and select the asset you want to monitor using the&nbsp;<strong>ResourceExplorer </strong>component. Upon selecting a particular asset, you can view the <strong>Machine State </strong>measurements<strong>&nbsp;</strong>in the displayed&nbsp;<strong>StatusTimeline&nbsp;</strong>component and&nbsp;<strong>Total Count </strong>and<strong> Good Count </strong>metrics<strong>&nbsp;</strong>in the&nbsp;<strong>LineChart </strong>components.</li> 
</ul> 
<pre><code class="lang-bash">npm start</code></pre> 
<p>AWS IoT Application Kit also includes components for the following visualization widgets:</p> 
<ul> 
 <li><strong>BarChart</strong></li> 
 <li><strong>Kpi</strong></li> 
 <li><strong>ScatterChart</strong></li> 
 <li><strong>StatusGrid</strong></li> 
</ul> 
<p>The starter ReactJS application also contains sample implementations of <strong>BarChart</strong>, <strong>Kpi </strong>and&nbsp;<strong>StatusGrid </strong>components in the file <code>src/App.tsx</code>. You can refer to <a href="https://github.com/awslabs/iot-app-kit/blob/main/docs/WhatIs.md">AWS IoT Application Kit documentation</a> for details on how to use these components in your ReactJS application.</p> 
<div class="wp-caption aligncenter" id="attachment_7704" style="width: 1034px;">
 <a href="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/05/08/AWS-IoT-App-Kit-Demo-Screenshot-1.png"><img alt="Screenshot of demo application" class="size-large wp-image-7704" height="602" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/05/08/AWS-IoT-App-Kit-Demo-Screenshot-1-1024x602.png" width="1024" /></a>
 <p class="wp-caption-text" id="caption-attachment-7704">Figure 4: Screenshot of demo application</p>
</div> 
<p>You can also refer to the sample file <code>src/App.completed.tsx</code> for a completed implementation of AWS IoT Application Kit.</p> 
<p>You can also host the ReactJS application built in this walkthrough with <a href="https://aws.amazon.com/amplify/">AWS Amplify</a>. You can refer to <a href="https://aws.amazon.com/getting-started/hands-on/build-react-app-amplify-graphql/module-one/?e=gs2020&amp;p=build-a-react-app-intro">AWS Amplify getting started hands-on guide</a>&nbsp;to get started.</p> 
<h2>Cleaning up</h2> 
<p>Delete the created AWS resources setup in this walkthrough by changing directory to the project directory and executing the following stack deletion commands.&nbsp;When prompted with <code>“Are you sure you want to delete: (y/n)?”</code> Enter Y.</p> 
<pre><code class="lang-bash">cd iot-app-kit-demo 
cdk destroy OpcuaSimulatorStack
</code></pre> 
<h2>Conclusion</h2> 
<p>AWS IoT Application Kit provides abstraction, simplicity, and independence in building web applications to meet custom UI/UX requirements. You can learn more by visiting <a href="https://github.com/awslabs/iot-app-kit">AWS IoT Application Kit</a> to get started building real-time IoT web applications to monitor and visualize IoT data from AWS IoT SiteWise.</p>
