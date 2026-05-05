---
title: "Empowering operations: A scalable remote asset health monitoring solution"
url: "https://aws.amazon.com/blogs/iot/empowering-operations-a-scalable-remote-asset-health-monitoring-solution-the-internet-of-things-aws-official-blog/"
date: "Mon, 27 Jun 2022 15:28:41 +0000"
author: "Yuri Chamarelli"
feed_url: "https://aws.amazon.com/blogs/iot/tag/aws-iot-sitewise/feed/"
---
<p>Looking for ways to monitor the health of industrial remote assets and create a centralized dashboard? Perhaps your remote industrial assets are connected or ready to connect to AWS, but you haven’t decided on how to present the data to operators and maintenance teams. Industrial remote assets are many and from different technological eras. Industrial customers need ways to equally visualize and monitor the status of those assets. Operation teams can then rely on a comprehensive dashboard and alerts to activate maintenance teams or remote actions, improving uptime and efficiency. In this blog post, you will learn how to use the data ingested to AWS IoT Core to enable field teams with a centralized Grafana dashboard and alerts table.</p> 
<h2>Overview</h2> 
<p>This blog will work with a simulated dataset, the dataset will represent 10 remote pumping stations. You will build the end-to-end solution and running deployment scripts from the AWS Command Line Interface (AWS CLI). The goal of this post is to show the step-by-step process of building a remote asset monitoring solution in AWS. We have built an architecture which is scalable, completely serverless, and composed of an IoT data ingestion service. We are using AWS IoT Core for ingestion and simulation, AWS IoT SiteWise for asset modeling, and Amazon Managed Grafana for a centralized dashboard and alerts. In addition, you will interact with Amazon Simple Notification Service (Amazon SNS), where alarms assigned to different subscribers.</p> 
<h2>Scalable Architecture for Remote Asset health monitoring solution</h2> 
<div class="wp-caption alignnone" id="attachment_8126" style="width: 2020px;">
 <img alt="" class=" wp-image-8126" height="531" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/06/13/remote_asset_monitoring-rev03.jpg" width="2010" />
 <p class="wp-caption-text" id="caption-attachment-8126"><span style="font-size: 16px;">This is the scalable architecture you will build in this blog. (Note: the Amazon Elastic Compute Cloud (Amazon EC2) instance is a simulation environment only)</span></p>
</div> 
<h2>Prerequisites</h2> 
<p>An AWS account is required to setup and execute the steps in this blog, AWS service will be configured, and you must have the necessary Identity and Access Management (IAM) permission to do the following:</p> 
<ul> 
 <li>Setup of an Amazon EC2 instance through Cloud9 environment for the IoT Device Simulator. The Amazon EC2 instance must have a role associated, which allows administrator access to the following services: 
  <ul> 
   <li>AWS IoT Core</li> 
   <li>AWS IoT SiteWise</li> 
   <li>AWS Managed Grafana</li> 
   <li>Amazon SNS</li> 
  </ul> </li> 
 <li>Access to IAM to create a role (or pre-created role).</li> 
 <li>An email box for the notification subscription.</li> 
 <li>Work from a region which supports all the services pre-listed, we recommend <strong>us-east-1</strong>.</li> 
</ul> 
<h2>Getting the environment ready</h2> 
<h3>Create AWS Cloud9 environment</h3> 
<ol> 
 <li>Log into the <a href="https://us-east-1.console.aws.amazon.com/cloud9/home/product">AWS Cloud9 console.</a></li> 
 <li>Click <strong>Create environment.</strong></li> 
 <li>Enter the name <strong>IoT Simulator</strong> and click <strong>next</strong>.</li> 
 <li>Select Create a new <strong>EC2 instance (direct access).</strong></li> 
 <li>Select Instance type <strong>t3.small.</strong></li> 
 <li>Select <strong>Amazon Linux 2</strong> and click Next step.</li> 
 <li>Click<strong> Create environment</strong>. The instance will start and the AWS Cloud9 environment will be ready for work. <strong>Note</strong>: You will execute commands from the AWS Cloud9 environment. Make sure the role attached to your Cloud9 provisioned instance has rights to execute AWS CLI commands for all services utilized in this blog. You can verify it from the Amazon EC2 console (go to &nbsp;<a href="https://us-east-1.console.aws.amazon.com/cloud9/home/product">AWS Cloud9</a> → <strong>your environments</strong> → <strong>select IoT Simulator</strong> → <strong>View details</strong> → Click on<strong>&nbsp;instances</strong> in EC2. From EC2 select the AWS Cloud9 instance →&nbsp;<strong> Actions</strong> →&nbsp; <strong>Security</strong> → and <strong>Modify IAM role</strong>. At this point, you can check which is assigned to the instance, select a pre-created role or create a new one).</li> 
 <li>This step will clone the scripts and sample files we created for this blog post. From the terminal run the following commands:<br /> <code>git clone https://github.com/aws-samples/aws-iot-remote-asset-health-monitoring.git<br /> cd aws-iot-remote-asset-health-monitoring</code><br /> <code>chmod +x bootstrap.sh</code><br /> <code>./bootstrap.sh</code></li> 
</ol> 
<h3>Start Simulator for AWS IoT Core ingestion</h3> 
<p>We have prepared a script to create the IoT Thing with the required resources for the simulation.</p> 
<ol> 
 <li>Now execute the following commands:<br /> <code>python create_thing.py</code><br /> <code>nohup ./start.sh &gt; iotconnection.log 2&gt;&amp;1 &amp;</code></li> 
 <li>Navigate to<a href="https://us-east-1.console.aws.amazon.com/iot/home?region=us-east-1#"> AWS IoT Core</a> → <strong>Test</strong> → <strong>MQTT test client</strong></li> 
 <li>Got to Subscribe to a topic, type <strong># </strong>and click Subscribe. You will see messages arriving under Topic as below:<br /> The payload represents 10 pumping stations with different locations and anomalies. You will use part of the simulated values to build assets and dashboards.<img alt="" class="aligncenter wp-image-8245" height="981" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/06/20/image-16.png" width="3000" /></li> 
</ol> 
<h2>Ingesting data into AWS IoT SiteWise</h2> 
<h3>Creating IAM role for SiteWise ingestion rules</h3> 
<ol> 
 <li>Log into <a href="https://us-east-1.console.aws.amazon.com/iamv2/home#/home">IAM (Identy Access managment)</a> → <strong>Roles</strong> and click <strong>Create Role</strong></li> 
 <li>Select <strong>Custom trust policy</strong> and paste the below JSON snippet, click Next. <pre><code class="lang-json">{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Service": "iot.amazonaws.com"
            },
            "Action": "sts:AssumeRole"
        }
    ]
}</code></pre> </li> 
 <li>In Permissions search for <strong>SiteWise</strong> and select the policy <strong>AWSIoTSiteWiseFullAccess</strong> and click next.</li> 
 <li>In Role Details, name the role with the following name “ <strong>iot_to_sitewise</strong>” and <strong>Create a Role</strong>.</li> 
 <li>Click on <strong>View Role </strong></li> 
 <li>Copy and save the <strong>Role ARN</strong>, as you will need it to run the rule creation script.</li> 
</ol> 
<h3>Creating Rules to publish to AWS IoT SiteWise</h3> 
<p>You can find more on how to create IoT rules in <a href="https://docs.aws.amazon.com/iot/latest/developerguide/iot-create-rule.html">AWS IoT Core Documentation</a> and how to configure Actions to publish to AWS IoT SiteWise at <a href="https://docs.aws.amazon.com/iot-sitewise/latest/userguide/iot-rules.html">AWS IoT SiteWise Documentation</a>. In this blog, you will use the AWS CLI commands to create the rules from your AWS Cloud9 environment. We have prepared a python script which will create the rules using the previously created <strong>IAM role</strong>.</p> 
<ol> 
 <li>Go to your AWS Cloud9 environment, from the <strong>~/environment/aws-iot-remote-asset-health-monitoring</strong>&nbsp;directory. Now run the following command:<br /> <code>python create_iotrules.py -r <strong>&lt;the Role Arn here&gt;</strong></code><br /> This script creates the IoT rules to ingest the simulation data into AWS IoT SiteWise. (<strong>Note</strong>: The policy used for this demonstration is over permissive and shouldn’t be used in a production environment, for more information on scoping down policies check this <a href="https://docs.aws.amazon.com/iot/latest/developerguide/audit-chk-iot-policy-permissive.html">AWS IoT policies overly permissive</a>)</li> 
 <li>Now navigate to <a href="https://us-east-1.console.aws.amazon.com/iot/home?region=us-east-1#/home">AWS IoT Core</a> → <strong>Message Routing</strong>→ <strong>Rules</strong><br /> You will see the rules at the console as <strong>Active </strong>as below.<br /> <img alt="" class="alignnone wp-image-8138 size-full" height="664" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/06/13/image-1-1.png" width="1304" /></li> 
 <li>Now navigate to <a href="https://us-east-1.console.aws.amazon.com/iotsitewise/home?region=us-east-1#/">AWS IoT SiteWise</a>→ <strong>Data streams<br /> </strong>AWS IoT SiteWise Data Streams is where ingested data, which is not yet assigned to an asset is automatically stored. For more information on Data Streams follow this <a href="https://docs.aws.amazon.com/iot-sitewise/latest/userguide/data-streams.html">link</a>. Now verify the data arriving in AWS IoT SiteWise and being update periodically, you can also filter through all <strong>Alias prefix</strong>, use the <strong>/pumpingstation/{n}</strong> and <strong>All data streams</strong> to observe the data.<br /> <img alt="" class="alignnone wp-image-8139 size-full" height="905" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/06/13/image-2-2.png" width="1708" /></li> 
</ol> 
<h3></h3> 
<h3>Asset model and Assets in AWS IoT SiteWise</h3> 
<h3>Creating Asset Model</h3> 
<ol> 
 <li>Navigate to <a href="https://us-east-1.console.aws.amazon.com/iotsitewise/home?region=us-east-1#/">AWS IoT SiteWise</a>→ <strong>Models</strong> and click on <strong>Create model </strong></li> 
 <li>Configure your model as following, note that it is important to respect the <strong>syntax</strong> (including upper case and lower case, we recommend coping and pasting through this section) as that will influence on the auto asset generation. 
  <ul> 
   <li><strong>Name</strong>: PumpingStation;</li> 
   <li><strong>Description</strong>: This is the digital model representation of all pumping station</li> 
   <li><strong>Measurement definitions</strong>: 
    <ul> 
     <li>name: Temperature; Unit:F; Data type: Integer</li> 
     <li>name: Humidity; Unit:%; Data type: Integer</li> 
     <li>name: Pressure; Unit: PSI; Data type: Integer</li> 
     <li>name: Vibration; Unit: Hz; Data type: Integer</li> 
     <li>name: Flow; Unit: m3/s; Data type: Integer</li> 
     <li>name: rpm; Unit: rpm; Data type: Integer</li> 
     <li>name: Voltage; Unit: V; Data type: Integer</li> 
     <li>name: Amperage; Unit: A; Data type: Integer</li> 
     <li>name: Fan; Unit: on/off; Data type: Boolean</li> 
     <li>name: Location; Unit: State; Data type: String</li> 
    </ul> </li> 
   <li>Finish by clicking on <strong>Create model</strong>, you will have your mode available on the list.</li> 
  </ul> </li> 
 <li>Now your model will show as follows, copy the <strong>model ID</strong> and save it for the next steps, as we will need as an input for the creation scripts.<br /> <img alt="" class="alignnone wp-image-8141 size-full" height="270" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/06/13/image-3-2.png" width="1672" /><br /> <img alt="" class="alignnone wp-image-8140 size-full" height="543" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/06/13/image-4-2.png" width="1673" /></li> 
</ol> 
<h3>Creating Assets</h3> 
<ol> 
 <li>The next step is creating Assets from the model PumpingStation, for this step you will be running a python script which will automatically create 10 pumping station and associate the available Data stream to its match asset measurement. For more information on how to create Assets from Asset model refer to the <a href="https://docs.aws.amazon.com/iot-sitewise/latest/userguide/create-assets.html">AWS IoT SiteWise Documentation</a>.</li> 
 <li>Navigate back to the <strong>AWS Cloud9 environment</strong>.</li> 
 <li>From the <strong>~/environment/aws-iot-remote-asset-health-monitoring</strong> directory, execute the following commands:<br /> From your AWS Cloud9 terminal execute the following command:<br /> <code>python create_iotsitewise_assets.py -i <strong>&lt;your model Id here&gt;</strong></code><br /> The Script will run for about 5 minutes. You can watch the responses from the AWS CLI commands.</li> 
 <li>Go to <a href="https://us-east-1.console.aws.amazon.com/iotsitewise/home?region=us-east-1#/">AWS IoT SiteWise</a>→ <strong>Assets</strong> and confirm that all pumping stations were created and are active as below.<br /> <img alt="" class="alignnone wp-image-8142 size-full" height="527" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/06/14/image-5-2.png" width="1360" /></li> 
 <li>Select one of the <strong>Pumping Stations</strong> and go to <strong>measurements</strong>. Now confirm that your data is arriving in AWS IoT SiteWise and being updated periodically.<br /> <img alt="" class="alignnone wp-image-8143 size-full" height="725" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/06/14/image-6-2.png" width="1715" /></li> 
</ol> 
<h2>Creating Dashboards in Amazon Managed Grafana</h2> 
<h3>Creating Amazon SNS notification</h3> 
<p>Before we work with Amazon Managed Grafana, we recommend that you create and configure the notification channel. Amazon Managed Grafana can directly send notifications to Amazon SNS given the<strong> ARN</strong> for the Amazon SNS Topic.</p> 
<ol> 
 <li>Navigate to <a href="https://us-east-1.console.aws.amazon.com/sns/v3/home?region=us-east-1#/mobile/push-notifications">Amazon SNS (amazon.com)</a> →<strong> Topics</strong> and click on <strong>Create Topic</strong><br /> Select <strong>Standard</strong>, Name it : <strong>All_Pumping_Stations</strong></li> 
 <li>Go to Access policy, and Select <strong>Everyone in both options</strong>, publish and subscribe,&nbsp;and click Create.(Note: an over-permissive policy is not recommended beyond this simulation environment)</li> 
 <li>Navigate to your newly create topic <strong>All_Pumping_Stations</strong> and click on Create subscription.</li> 
 <li>Select <strong>Email</strong>, under Endpoint <strong>add your test email address</strong>, and click create subscription.</li> 
 <li>The Subscription will show as <strong>Pending confirmation Status</strong>, within a minute you will receive a confirmation link in your test email, after accepting it the subscription is ready. (shown as below).</li> 
 <li>Copy the <strong>ARN</strong>&nbsp;for the topic and save it for later on your clipboard.<br /> <img alt="" class="alignnone wp-image-8145 size-full" height="662" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/06/14/image-7.png" width="942" /></li> 
</ol> 
<h3>Creating Amazon Managed Grafana workspace</h3> 
<ol> 
 <li> 
  <ol> 
   <li>Navigate to <a href="https://us-east-1.console.aws.amazon.com/grafana/">AWS Grafana Console</a> , click on <strong>Create workspace</strong></li> 
   <li>Name: <strong>all_pumping_stations</strong>, Click next</li> 
   <li>Select <strong>AWS Single Sign-On </strong>(If AWS Single Sign-On is not enabled, you must enable it, follow this <a href="https://docs.aws.amazon.com/singlesignon/latest/userguide/step1.html">link</a> for instructions. Amazon Managed Grafana also supports SAML authentication, you can find more information <a href="https://docs.aws.amazon.com/grafana/latest/userguide/authentication-in-AMG-SAML.html">here</a>)</li> 
   <li>Select <strong>Service managed</strong>, Click next</li> 
   <li>Select <strong>Current account</strong></li> 
   <li>Under Data sources select <strong>AWS IoT SiteWise</strong></li> 
   <li>Under Notification channels, select <strong>Amazon&nbsp;SNS</strong>, Click Next</li> 
   <li>Click <strong>Create Workspace</strong></li> 
   <li>Once your workspace is <strong>ready</strong> status under Authentication, you must assign a new user to your workspace. <strong>Click on Assign new user or group</strong>.</li> 
   <li>If you have enable AWS Single Sign-On (SSO), you will then see a user or group, select it and click <strong>Assign User</strong>, then Select the same user or group, and go to <strong>Actions and make Admin</strong><br /> Navigate back to <a href="https://us-east-1.console.aws.amazon.com/grafana/home?region=us-east-1#/">AWS Grafana Console</a> → <strong>All workspaces</strong> → <strong>all_pumping_stations</strong>, look for the Grafana workspace URL and click on it.</li> 
   <li>A new tab will open and you see the login page as below. Log in and you are ready for the next step.<br /> <img alt="" class="alignnone wp-image-8146 size-full" height="595" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/06/14/image-8.png" width="747" /></li> 
  </ol> </li> 
</ol> 
<h3>Configuring Amazon Managed Grafana notification channel</h3> 
<ol> 
 <li>In your <strong>Grafana workspace</strong> Navigate to <strong>Alerting</strong>→<strong> Notification channels</strong>.</li> 
 <li>Click on Add Channel. 
  <ul> 
   <li>Name : <strong>all_pumping_stations</strong></li> 
   <li>Type : <strong>AWS SNS</strong></li> 
   <li>Topic : &lt;<strong>Paste the topic ARN from the previous section</strong>&gt;. If you need to locate it again, Navigate to <a href="https://us-east-1.console.aws.amazon.com/sns/v3/home?region=us-east-1#/dashboard">Simple Notification Service</a>→ <strong>Topics</strong>→<strong> all_pumping_stations</strong>, the ARN is located under details.</li> 
   <li>Auth Provider: <strong>Workspace IAM Role</strong></li> 
   <li>Message Body Format:<strong> Text</strong></li> 
   <li>Under <strong>Notification setting</strong>, select Default</li> 
   <li>Click <strong>Save</strong>. (Optionally, you can also click on the test to make sure your notification channel is correctly setup)</li> 
  </ul> </li> 
</ol> 
<h3>Configuring Grafana Data sources</h3> 
<ol> 
 <li>In your <strong>Grafana workspace</strong> Navigate to <strong>Configuration</strong>→ <strong>Data Sources</strong></li> 
 <li>Click on <strong>Add data Source</strong>.</li> 
 <li>Search for <strong>AWS IoT SiteWise</strong> and click on it<br /> <img alt="" class="alignnone wp-image-8148 size-full" height="316" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/06/14/image-10.png" width="1484" /></li> 
 <li>Under Connection Details keep the default except for <strong>Default Region</strong>. Select the region where you built your SiteWise assets.</li> 
 <li>Click <strong>Save &amp; test<br /> </strong></li> 
</ol> 
<h3>Creating Dashboards in Amazon Managed Grafana</h3> 
<p>For the monitoring dashboards and alerts we have created a python script which will automatically deploy one dashboard for each pumping station.</p> 
<ol> 
 <li>Navigate to <a href="https://us-east-1.console.aws.amazon.com/grafana/home?region=us-east-1#/workspaces">AWS Grafana Console</a>→ <strong>Workspaces</strong>→ <strong>all_pumping_stations</strong></li> 
 <li>Look for the Grafana workspace <strong>URL</strong> and copy the ID before .grafana, as shown below.<br /> <img alt="" class="alignnone wp-image-8147 size-full" height="53" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/06/14/image-9.png" width="388" /></li> 
 <li>Go to your <strong>AWS Cloud9 environment</strong>, from the <strong>~/environment/aws-iot-remote-asset-health-monitoring</strong> directory, run the following command:<br /> <code>python create_grafana_dashboards.py -i <strong>&lt;your workspace ID here&gt;</strong> -r <strong>&lt;your Model ID here&gt;</strong></code></li> 
 <li>After the script is finished, navigate to the <strong>Grafana Workspace</strong>→ <strong>Dashboards</strong>→ <strong>Browse</strong>, confirm that all dashboards have been successfully created and the simulation data is being ingested as shown below.<br /> <img alt="" class="alignnone wp-image-8149 size-full" height="988" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/06/14/image-11.png" width="1471" /></li> 
 <li>Navigate to <strong>pumpingstation9/Status/Time_series</strong> dashboard and confirm the temperature is high, and the alert is <strong>active</strong>. For this simulation data set, the Pumping Station 9 presents the anomalous temperature and triggers an alert to the notification channel every 2 minutes. Optionally navigate to any other pumping station dashboard and compare them.<br /> <img alt="" class="alignnone wp-image-8150 size-full" height="1277" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/06/14/image-12.png" width="2534" /></li> 
 <li>Now navigate to <strong>Alerting</strong>→ <strong>Alert rules</strong>, and check that all the other alerts are healthy.<br /> <img alt="" class="alignnone wp-image-8151 size-full" height="723" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/06/14/image-14.png" width="1473" /></li> 
</ol> 
<h2>Conclusion</h2> 
<p>In this blog, you learned how to use AWS IoT SiteWise to collect and organize data from remote industrial assets. And with Amazon Managed Grafana operators can notified about key issues, with alarms and alerts delivered to them when equipment behaves anomalously or operates outside of expected operating limits. With the solution described in this blog, you can implement a reliable and scalable field-to-cloud Industrial IoT (IIoT) solution on AWS for use cases such as asset remote asset monitoring. Also, the architecture example provides further connectivity with other AWS services, allowing integration with automation services and data sources.</p> 
<h2>Clean Up</h2> 
<p>Be sure to clean up the work in this blog to avoid charges. Delete the following resources when finished in this order:</p> 
<ol> 
 <li><strong>AWS Manage Grafana</strong> → Workspace, and delete the workspace created for the work.</li> 
 <li><strong>Simple Notification Service</strong> → Topic, and delete the topic created for the work.</li> 
 <li><strong>AWS IoT SiteWise</strong> → Assets and delete Assets.</li> 
 <li><strong>AWS IoT SiteWise</strong> → Data Streams, and delete all data streams related to the work.</li> 
 <li><strong>AWS IoT SiteWise</strong> → Model, and delete the Model created for the work.</li> 
 <li><strong>AWS IoT</strong> → Message routing → Rules, and Delete rules created for the role.</li> 
 <li><strong>AWS Cloud9</strong> → Your environments and delete the environment.</li> 
</ol> 
<h2>About the Authors</h2> 
<p><a href="https://www.linkedin.com/in/yuri-chamarelli-49130956/">Yuri Chamarelli</a><img alt="" class="alignleft wp-image-8360 size-thumbnail" height="150" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/06/27/yuriresized-150x150.png" width="150" /> is an Amazon Web Services Solution Architect (AWS) based out of the United States. As an IoT specialist, he focuses on helping customers build with AWS IoT and accomplish their business outcomes. Yuri is a Controls engineer with over 10 years of experience in IT/OT systems and has helped several customers with Industrial transformation and Industrial automation projects throughout many industries.</p> 
<p>&nbsp;</p> 
<p>&nbsp;</p> 
<p>&nbsp;</p> 
<p style="text-align: left;"><img alt="" class="wp-image-8361 alignleft" height="154" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/06/27/seibouresized.jpg" width="148" /><a href="https://www.linkedin.com/in/seibou-gounteni?miniProfileUrn=urn%3Ali%3Afs_miniProfile%3AACoAAAx6DLoBDPrnCVaDjaoA97kqZng7GI1pugM&amp;lipi=urn%3Ali%3Apage%3Ad_flagship3_search_srp_all%3Bnj9QyHYrRYG8xAI4qjeq6g%3D%3D">Seibou Gounteni</a> is a Specialist Solutions Architect for IoT at Amazon Web Services (AWS). He helps customers architect, develop, operate scalable and highly innovative solutions using the depth and breadth of AWS platform capabilities to deliver measurable business outcomes. Seibou is an instrumentation engineer with over 10 years of experience in digital platforms, smart manufacturing, energy management, industrial automation and IT/OT systems across a diverse range of industries.</p>
