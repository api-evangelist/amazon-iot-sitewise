---
title: "AWS IoT SiteWise adds support for 10 new industrial protocols with Domatica EasyEdge integration"
url: "https://aws.amazon.com/blogs/iot/aws-iot-sitewise-adds-support-for-10-new-industrial-protocols-with-domatica-easyedge-integration/"
date: "Thu, 09 Nov 2023 19:37:40 +0000"
author: "Seibou Gounteni"
feed_url: "https://aws.amazon.com/blogs/iot/tag/aws-iot-sitewise/feed/"
---
<h2>Introduction</h2> 
<p>Today, we <a href="https://aws.amazon.com/about-aws/whats-new/2023/11/aws-iot-sitewise-edge-easy-edge-protocol-support/">announced</a> the general availability of extended industrial protocol support for&nbsp;<a href="https://aws.amazon.com/iot-sitewise/"> AWS IoT SiteWise</a> – a managed service that makes it easy to collect, store, organize and monitor data from industrial equipment at scale to help you make data-driven decisions. <a href="https://aws.amazon.com/iot-sitewise/sitewise-edge/">AWS IoT SiteWise Edge</a>, a feature of AWS IoT SiteWise, extends the cloud capabilities to collect, organize, process, and monitor equipment data on-premises. Through a new integration with AWS Partner <a href="https://www.domatica.io/">Domatica</a>, customers now have the ability ingest data from 10 additional industrial protocols including Modbus (TCP &amp; RTU), Ethernet/IP, Siemens S7, KNX, LoRaWAN, MQTT, Profinet, Profibus, BACnet and Restfull (REST API) interfaces, in addition to native OPC UA support. Previously, ingesting data from these protocols required acquiring, provisioning, and configuring infrastructure and middleware for data collection and translation resulting in additional cost and time to value.</p> 
<p>In this blog post, we will walk through the installation and configuration of <a href="https://aws.amazon.com/iot-sitewise/sitewise-edge/">AWS IoT SiteWise Edge</a> gateway software with Domatica EasyEdge data collector to ingest equipment data from a Siemens S7 PLC into <a href="https://aws.amazon.com/iot-sitewise/">AWS IoT SiteWise</a>. Refer to Domatica <a href="https://docs.easyedge.io/">documentation</a> on how to connect additional data sources.</p> 
<h2>Solution overview</h2> 
<p>Through the AWS Console, users can simply add AWS Partner <a href="https://www.easyedge.io/easyedge-for-aws/">Domatica’s EasyEdge</a> software as a data source on their existing <a href="https://aws.amazon.com/iot-sitewise/sitewise-edge/">AWS IoT SiteWise Edge</a> gateway. AWS IoT SiteWise Edge provides on-premises software to extend the cloud capabilities in AWS IoT SiteWise to the industrial edge. Users then configure the protocols, desired data flows, and data conditioning in the partner application. After configurations are deployed, the equipment data flows seamlessly to <a href="https://aws.amazon.com/iot-sitewise/sitewise-edge/">AWS IoT SiteWise Edge</a> for local monitoring, storage, and access at the edge. The data flow It is also sent to AWS IoT SiteWise for integration with other industrial data and usage in other AWS Cloud services.</p> 
<p>Once the data is ingested into AWS SiteWise, you can visualize the collected data with <a href="https://docs.aws.amazon.com/iot-sitewise/latest/appguide/what-is-monitor-app.html">IoT SiteWise Monitor</a>, a feature of AWS IoT SiteWise; it provides portals in the form of managed web applications where you can create dashboards. You can also leverage <a href="https://aws.amazon.com/grafana/">Amazon Managed Grafana</a> to visualize and monitor data in dashboards by <a href="https://docs.aws.amazon.com/grafana/latest/userguide/using-iotsitewise-in-AMG.html">using the AWS IoT SiteWise data source</a>; or store your data in hot and cold storage tiers of <a href="https://aws.amazon.com/iot-sitewise/">AWS IoT SiteWise</a>: a hot tier optimized for real-time applications and frequently accessed data with lower write-to-read latency, and a cold tier optimized for analytical applications with less-frequently accessed data and historical data, such as business intelligence (BI) dashboards, artificial intelligence (AI) and machine learning (ML) training, historical reports, and backups. For more information on AWS IoT SiteWise, you can visit the <a href="https://docs.aws.amazon.com/iot-sitewise/latest/userguide/what-is-sitewise.html">AWS IoT SiteWise user guide</a>.</p> 
<p>These sections summarize how to create a SiteWise Edge gateway and include detailed instructions for steps that are specific to adding EasyEdge data source and connect to a Siemens S7 PLC. For this demonstration, we are using a virtual Siemens S7 PLC from this <a href="https://github.com/gijzelaerr/python-snap7">open source GitHub repo</a>.</p> 
<p>There are four key steps to consider when building this solution:</p> 
<ol> 
 <li><strong>Create</strong> <a href="https://aws.amazon.com/iot-sitewise/sitewise-edge/"><strong>AWS IoT SiteWise Edge</strong></a> <strong>Gateway</strong></li> 
 <li><strong>Add data source for EasyEdge</strong></li> 
 <li><strong>Connect EasyEdge to a Siemens S7 PLC</strong></li> 
 <li><strong>Verify data flow into</strong> <a href="https://aws.amazon.com/iot-sitewise/"><strong>AWS IoT SiteWise</strong></a></li> 
</ol> 
<h2>Prerequisites</h2> 
<ul> 
 <li>At a minimum, <a href="https://aws.amazon.com/iot-sitewise/sitewise-edge/">AWS IoT SiteWise Edge</a> requires an industrial computer running Linux with a x86 64 bit quad-core processor, 16GB RAM, and 256GB in disk space. The gateway device must allow inbound traffic on port 443 and it must allow outbound traffic on ports 443 and 8883.</li> 
 <li>A Siemens S7 PLC</li> 
 <li><a href="https://www.easyedge.io/">EasyEdge Studio</a> account.</li> 
</ul> 
<h2>Solution Architecture</h2> 
<p><img alt="" class="alignnone wp-image-14288 size-full" height="395" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2023/11/08/b1.png" width="964" /></p> 
<h2>Walkthrough</h2> 
<h3>1. Create AWS IoT SiteWise Gateway</h3> 
<p>In the AWS Management Console, create and install the SiteWise Edge gateway with a Linux machine, following these <a href="https://docs.aws.amazon.com/iot-sitewise/latest/userguide/configure-gateway-ggv2.html">instructions</a>. During this process, skip the data source as it will be configured after the gateway is connected.</p> 
<h3>2. Configure EasyEdge Data source</h3> 
<p>In this step, we will add EasyEdge as a data source in the SiteWise Edge gateway.</p> 
<p>Before adding the EasyEdge data source, confirm that the SiteWise Edge gateway is connected by navigating to the AWS console (<strong><strong>Services→AWS</strong> <strong>IoT</strong> <strong>SiteWise→Edge→Gateways) </strong></strong>→ <strong>select your gateway).</strong></p> 
<p>The <strong>Gateway configuration</strong> section of the <strong>Overview</strong> tab should now say “<strong>Connected</strong>.”</p> 
<div class="wp-caption alignnone" id="attachment_14289" style="width: 1034px;">
 <img alt="" class="wp-image-14289 size-large" height="386" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2023/11/08/b2-1024x386.jpg" width="1024" />
 <p class="wp-caption-text" id="caption-attachment-14289">Figure 1: SiteWise Gateway</p>
</div> 
<p>Click on the SiteWise Edge gateway you created and choose <strong>Add data source:</strong></p> 
<div class="wp-caption alignnone" id="attachment_14290" style="width: 1034px;">
 <img alt="" class="wp-image-14290 size-large" height="295" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2023/11/08/b3-1024x295.jpg" width="1024" />
 <p class="wp-caption-text" id="caption-attachment-14290">Figure 2: SiteWise data source</p>
</div> 
<p>Select “<strong>EasyEdge</strong>” from the “<strong>Source</strong> <strong>Type</strong>“ drop-down list, provide a name for the data source, and select ”<strong>Authorize</strong>“ and ”<strong>Update components</strong>“. <strong>Note:</strong> EasyEdge requires that <a href="https://docs.aws.amazon.com/iot-sitewise/latest/userguide/connect-partner-app.html">Docker is installed</a> on the Edge Gateway, and that you <a href="https://accounts.easyedge.io/signup?partner=aws">create an account with EasyEdge</a>.</p> 
<div class="wp-caption alignnone" id="attachment_14291" style="width: 967px;">
 <img alt="" class="wp-image-14291 size-large" height="1024" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2023/11/08/b4-957x1024.jpg" width="957" />
 <p class="wp-caption-text" id="caption-attachment-14291">Figure 3: EasyEdge Data Source</p>
</div> 
<p>After clicking “<strong>Save</strong>” you will be redirected to the <a href="https://studio.easyedge.io/">EasyEdge Studio</a> site. Proceed to Configure your EdgeNode and click <strong>Next</strong> on the following screen:</p> 
<div class="wp-caption alignnone" id="attachment_14292" style="width: 963px;">
 <img alt="" class="wp-image-14292 " height="667" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2023/11/08/b5-1024x717.jpg" width="953" />
 <p class="wp-caption-text" id="caption-attachment-14292">Figure 4: Edge Node Configuration</p>
</div> 
<p>Review the settings and click “<strong>Finish</strong>” to complete creating the EdgeNode:</p> 
<div class="wp-caption alignnone" id="attachment_14293" style="width: 951px;">
 <img alt="" class="wp-image-14293 " height="629" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2023/11/08/b6-1024x685.jpg" width="941" />
 <p class="wp-caption-text" id="caption-attachment-14293">Figure 5: EasyEdge Data Source Summary</p>
</div> 
<p>The EdgeNode status will be updated to “<strong>Online</strong>” which confirms a successful connectivity between the SiteWise Edge gateway and EasyEdge.</p> 
<div class="wp-caption alignnone" id="attachment_14294" style="width: 1034px;">
 <img alt="" class="wp-image-14294 size-large" height="347" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2023/11/08/b7-1024x347.jpg" width="1024" />
 <p class="wp-caption-text" id="caption-attachment-14294">Figure 6: EasyEdge Online</p>
</div> 
<p>Confirm that the SiteWise Edge deployment was successful: on the AWS Console, navigate to <strong>IoT Greengrass → Greengrass Devices → Deployments</strong> and confirm that the deployment status is “<strong>Completed</strong>”:</p> 
<div class="wp-caption alignnone" id="attachment_14295" style="width: 1034px;">
 <img alt="" class="wp-image-14295 size-large" height="193" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2023/11/08/b8-1024x193.jpg" width="1024" />
 <p class="wp-caption-text" id="caption-attachment-14295">Figure 7: AWS IoT Greengrass Deployment</p>
</div> 
<h3>3. Connect EasyEdge to the Siemens S7 PLC</h3> 
<p>In this step, we will connect the Siemens S7 PLC as a device to the EdgeNode configured in <strong>Step 2.</strong></p> 
<p>Navigate back to the EasyEdge Studio console and click on “<strong>Devices</strong>” on the left navigation menu. Click on “<strong>Add</strong>” in the “<strong>Add Device</strong>” section and configure your device connection. Select “<strong>Siemens S7 Client</strong>” from the list:</p> 
<div class="wp-caption alignnone" id="attachment_14296" style="width: 1034px;">
 <img alt="" class="wp-image-14296 size-large" height="392" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2023/11/08/b9-1024x392.jpg" width="1024" />
 <p class="wp-caption-text" id="caption-attachment-14296">Figure 8: Add EasyEdge Device</p>
</div> 
<div class="wp-caption alignnone" id="attachment_14297" style="width: 1034px;">
 <img alt="" class="wp-image-14297 size-large" height="754" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2023/11/08/b10-1024x754.jpg" width="1024" />
 <p class="wp-caption-text" id="caption-attachment-14297">Figure 9: Add Siemens S7 Engine</p>
</div> 
<p>Select “<strong>Create from scratch</strong>”:</p> 
<div class="wp-caption alignnone" id="attachment_14298" style="width: 1034px;">
 <img alt="" class="wp-image-14298 size-large" height="656" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2023/11/08/b11-1024x656.jpg" width="1024" />
 <p class="wp-caption-text" id="caption-attachment-14298">Figure 10: Siemens S7 Engine Configuration</p>
</div> 
<p>Add the <strong>datapoints/register</strong> that you would like to collect from the S7 PLC and click “<strong>Next</strong>”:</p> 
<div class="wp-caption alignnone" id="attachment_14299" style="width: 1034px;">
 <img alt="" class="wp-image-14299 size-large" height="456" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2023/11/08/b12-1024x456.jpg" width="1024" />
 <p class="wp-caption-text" id="caption-attachment-14299">Figure 11: Siemens S7 Datapoints Configuration</p>
</div> 
<p>Give the connection a name and location (optional) and select “<strong>Next</strong>”:</p> 
<div class="wp-caption alignnone" id="attachment_14300" style="width: 1034px;">
 <img alt="" class="wp-image-14300 size-large" height="558" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2023/11/08/b13-1024x558.jpg" width="1024" />
 <p class="wp-caption-text" id="caption-attachment-14300">Figure 12: Siemens S7 Configuration</p>
</div> 
<p>Configure the details for the S7 PLC and click “<strong>Next</strong>”:</p> 
<div class="wp-caption alignnone" id="attachment_14301" style="width: 1034px;">
 <img alt="" class="wp-image-14301 size-large" height="579" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2023/11/08/b14-1024x579.jpg" width="1024" />
 <p class="wp-caption-text" id="caption-attachment-14301">Figure 13: Siemens S7 IP Configuration</p>
</div> 
<p>Select the list of datapoints that you want to enable:</p> 
<div class="wp-caption alignnone" id="attachment_14302" style="width: 1034px;">
 <img alt="" class="wp-image-14302 size-large" height="564" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2023/11/08/b15-1024x564.jpg" width="1024" />
 <p class="wp-caption-text" id="caption-attachment-14302">Figure 14: Siemens S7 Tag Configuration</p>
</div> 
<p>Review the settings and click “<strong>Finish</strong>” to create your device connection:</p> 
<div class="wp-caption alignnone" id="attachment_14303" style="width: 1034px;">
 <img alt="" class="wp-image-14303 size-large" height="767" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2023/11/08/b16-1024x767.jpg" width="1024" />
 <p class="wp-caption-text" id="caption-attachment-14303">Figure 15: Siemens S7 Tag Configuration Summary</p>
</div> 
<p>Click the <strong>deployment</strong> icon on the top navigation bar:</p> 
<div class="wp-caption alignnone" id="attachment_14304" style="width: 429px;">
 <img alt="" class="wp-image-14304 size-full" height="164" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2023/11/08/b17.jpg" width="419" />
 <p class="wp-caption-text" id="caption-attachment-14304">Figure 16: EdgeNode Deployment</p>
</div> 
<p>Make sure the EdgeNode device you created earlier is selected, and click on “<strong>Deploy</strong>:</p> 
<div class="wp-caption alignnone" id="attachment_14305" style="width: 1034px;">
 <img alt="" class="wp-image-14305 size-large" height="468" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2023/11/08/b18-1024x468.jpg" width="1024" />
 <p class="wp-caption-text" id="caption-attachment-14305">Figure 17: EdgeNode Deployment</p>
</div> 
<p>You will see the following popup with a progress indicator; click “<strong>Close</strong>” after the deployment finishes successfully:</p> 
<div class="wp-caption alignnone" id="attachment_14306" style="width: 1034px;">
 <img alt="" class="wp-image-14306 size-large" height="457" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2023/11/08/b19-1024x457.jpg" width="1024" />
 <p class="wp-caption-text" id="caption-attachment-14306">Figure 18: Successful EdgeNode Deployment</p>
</div> 
<p>To verify that data is coming in from your device, select “<strong>Things</strong>” from the left navigation menu, and click on “<strong>View</strong>”:</p> 
<div class="wp-caption alignnone" id="attachment_14307" style="width: 1034px;">
 <img alt="" class="wp-image-14307 size-large" height="215" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2023/11/08/b20-1024x215.jpg" width="1024" />
 <p class="wp-caption-text" id="caption-attachment-14307">Figure 19: Verify data flow into EdgeNode</p>
</div> 
<p>Select the “<strong>Realtime</strong>” icon on the right side of the screen and the console will poll your device for new values periodically. Your datapoint will be displayed at the bottom left of the pane with the updated value:</p> 
<div class="wp-caption alignnone" id="attachment_14308" style="width: 1034px;">
 <img alt="" class="wp-image-14308 size-large" height="364" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2023/11/08/b21-1024x364.jpg" width="1024" />
 <p class="wp-caption-text" id="caption-attachment-14308">Figure 20: Confirm Data flow in EdgeNode</p>
</div> 
<h4>EasyEdge Workflow</h4> 
<p>EasyEdge also support <strong>workflows</strong> that allows you to build custom transformations of data at the edge, using an intuitive drag-and-drop user interface.</p> 
<p>To create a workflow, click on “<strong>Workflows</strong>” on the left-side navigation menu, and click on “<strong>Add</strong>”:</p> 
<div class="wp-caption alignnone" id="attachment_14309" style="width: 1034px;">
 <img alt="" class="wp-image-14309 size-large" height="335" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2023/11/08/b22-1024x335.jpg" width="1024" />
 <p class="wp-caption-text" id="caption-attachment-14309">Figure 21: Create EdgeNode Workflow</p>
</div> 
<p>Add the following blocks from the menu:</p> 
<ul> 
 <li>Things → S7-PLC</li> 
 <li>Code Blocks → Time-Series Min-Avg-Max</li> 
 <li>Data Models → Single Input (Int) – rename as “testTagMaximum”</li> 
 <li>Data Models → Single Input (Int) – rename as “testTagMinimum”</li> 
</ul> 
<p>Select the “<strong>Time-Series Min-Avg-Max</strong>” block and click on the “<strong>Gear</strong>” (Properties) icon on the top right; set the “<strong>StorePeriod</strong>” property to 1, and enable the “<strong>Export Thing</strong>” selector.</p> 
<p>Proceed to connecting the blocks as in the image below:</p> 
<div class="wp-caption alignnone" id="attachment_14310" style="width: 1009px;">
 <img alt="" class="wp-image-14310 size-full" height="457" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2023/11/08/b23.jpg" width="999" />
 <p class="wp-caption-text" id="caption-attachment-14310">Figure 21: EdgeNode Workflow Configuration</p>
</div> 
<p>Click on the “<strong>Save and deploy</strong>” option from the drop-down next to the “<strong>Save</strong>” icon (top-right navigation menu). Your workflow will be deployed to the Edge Node.</p> 
<p>For more information on the EasyEdge workflows please refer to <a href="https://docs.easyedge.io/easyedge-studio/easyedge-project/edge/workflow/overview.html">Domatica’s documentation</a>.</p> 
<h3>4. Verify data in AWS IoT SiteWise</h3> 
<p>In this step, we will verify that the data from the Siemens S7 PLC is ingested in <a href="https://aws.amazon.com/iot-sitewise/">AWS IoT SiteWise</a> using the data streams. <a href="https://aws.amazon.com/iot-sitewise/">AWS IoT SiteWise</a> automatically creates data streams to receive streams of raw data from your equipment.</p> 
<p>Navigate to the AWS console <strong>(Services→AWS</strong> <strong>IoT</strong> <strong>SiteWise→Build→Data</strong> <strong>streams)</strong>.</p> 
<p>Select the data streams and expand the panel at the bottom of the page to visualize the data.</p> 
<div class="wp-caption alignnone" id="attachment_14311" style="width: 1034px;">
 <img alt="" class="wp-image-14311 size-large" height="506" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2023/11/08/b24-1024x506.jpg" width="1024" />
 <p class="wp-caption-text" id="caption-attachment-14311">Figure 22: AWS IoT SiteWise Data Streams</p>
</div> 
<p>The data is now ingested from the Siemens S7 PLC into AWS SiteWise. From this point, you can map data streams to assets using the AWS SiteWise Console.</p> 
<h2>Conclusion</h2> 
<p>In this blog post, we walked through how to collect data from a Siemens S7 PLC into <a href="https://aws.amazon.com/iot-sitewise/">AWS IoT SiteWise</a> using AWS Partner Domatica EasyEdge software as a data source on a SiteWise Edge gateway. This solution enables you to ingest your industrial data sources into AWS IoT and opens up multiple use-cases for your industrial data in the cloud.</p> 
<p>For more hands-on instructions on how to add EasyEdge from Domatica data source to your <a href="https://aws.amazon.com/iot-sitewise/sitewise-edge/">AWS IoT SiteWise Edge</a> gateway, you can follow the steps in AWS <a href="https://docs.aws.amazon.com/iot-sitewise/latest/userguide/connect-partner-app.html">documentation</a> and the videos below:</p> 
<p><a href="https://www.youtube.com/watch?v=kvJtTAswMlg">Quick Start Guide to EasyEdge with AWS IoT SiteWise</a></p> 
<p><a href="https://www.youtube.com/watch?v=pDQgo3ASfLg">Building Advanced Capabilities in AWS IoT SiteWise Edge with EasyEdge</a></p> 
<h2>About the authors</h2> 
<table style="height: 456px;" width="1222"> 
 <tbody> 
  <tr style="vertical-align: top;"> 
   <td><img alt="" class="alignnone wp-image-14334 size-medium" height="300" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2023/11/08/ProfilePic-214x300.png" width="214" /></td> 
   <td><a href="https://www.linkedin.com/in/oscar-salcedo-a178306">Oscar Salcedo</a> is Specialist Solutions Architect for IoT &amp; Robotics at Amazon Web Services (AWS). He has over 20 years of experience in Smart Manufacturing, Industrial Automation, Building Automation, and IT/OT systems across diverse industries. He leverages the depth and breadth of AWS platform capabilities to architect and develop scalable and innovative solutions, driving measurable business outcomes for Customers.</td> 
  </tr> 
  <tr style="vertical-align: top;"> 
   <td><img alt="" class="alignnone size-medium wp-image-14336" height="300" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2023/11/08/4ff29e69-3d2a-4972-9ec8-8c496ee3d9cc-1-300x300.jpg" width="300" /></td> 
   <td><a href="https://www.linkedin.com/in/seibou-gounteni/">Seibou Gounteni</a> is a Specialist Solutions Architect for IoT &amp; Robotics at Amazon Web Services (AWS). He helps customers architect, develop, operate scalable and highly innovative solutions using the depth and breadth of AWS platform capabilities to deliver measurable business outcomes. Seibou has over 12 years of experience in digital platforms, smart manufacturing, energy management, industrial automation and IT/OT systems across a diverse range of industries.</td> 
  </tr> 
  <tr style="vertical-align: top;"> 
   <td><img alt="" class="alignnone size-full wp-image-14337" height="160" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2023/11/08/intissar.jpeg" width="120" /></td> 
   <td><a href="https://www.linkedin.com/in/intissar-harrabi-984a7aa9">Intissar Harrabi</a> is a Solutions Architect, part of the Canadian Public Sector team at AWS. Intissar is passionate about helping customers to improve their knowledge of Amazon Web Services (AWS) and find solutions to their technical challenges in the cloud. IoT and security are among her topics of interest.</td> 
  </tr> 
 </tbody> 
</table> 
<p></p>
