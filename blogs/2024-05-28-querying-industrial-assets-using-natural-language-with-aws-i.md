---
title: "Querying industrial assets using natural language with AWS IoT SiteWise and Agents for Amazon Bedrock"
url: "https://aws.amazon.com/blogs/iot/querying-industrial-assets-using-natural-language-with-aws-iot-sitewise-and-agents-for-amazon-bedrock/"
date: "Tue, 28 May 2024 14:47:50 +0000"
author: "Gabriel Verreault"
feed_url: "https://aws.amazon.com/blogs/iot/tag/aws-iot-sitewise/feed/"
---
<h2>Introduction</h2> 
<p>Generative AI-powered chatbots are driving productivity gains across industries by providing instant access to information from diverse data sources, accelerating decision-making, and reducing response times. In fast-paced industrial environments, process engineers, reliability experts, and maintenance personnel require quick access to accurate, real-time operational data to make informed decisions and maintain optimal performance. However, querying complex and often siloed industrial systems like SCADA, historians, and Internet of Things (IoT) platforms can be challenging and time-consuming, especially for those without specialized knowledge of how the operational data is organized and accessed.</p> 
<p>Generative AI-powered chatbots provide natural language interfaces to access real-time asset information from disparate operational and corporate data sources. By simplifying data retrieval through conversational interactions, generative AI enables operators to spend less time gathering data and more time optimizing industrial productivity. These user-friendly chatbots empower personnel across roles with valuable operational insights, streamlining access to critical information scattered throughout operational and corporate sources.</p> 
<p>Implementing chatbots in industrial settings requires a tool to assist a large language model (LLM) in navigating structured and unstructured data from industrial data stores to retrieve relevant information. This is where generative AI-powered agents come into play. Agents are AI systems that use an LLM to understand a problem, create a plan to solve it, and execute that plan by calling APIs, databases, or other resources. Agents act as an interface between users and complex data systems, enabling users to ask questions in natural language without needing to know the underlying data representations. For example, shop floor personnel could ask about a pump’s peak revolutions per minute (RPM) in the last hour without knowing how that data is organized. Since LLMs cannot perform complex calculations directly, agents orchestrate offloading those operations to industrial systems designed for efficient data processing. This allows end users to get natural language responses while leveraging existing data platforms behind the scenes.</p> 
<p>In this blog post, we will guide developers through the process of creating a conversational agent on <a href="https://aws.amazon.com/bedrock/" rel="noopener" target="_blank">Amazon Bedrock</a> that interacts with <a href="https://aws.amazon.com/iot-sitewise/" rel="noopener" target="_blank">AWS IoT SiteWise</a>, a service for collecting, storing, organizing, and monitoring industrial equipment data at scale. By leveraging AWS IoT SiteWise’s industrial data modeling and processing capabilities, chatbot developers can efficiently deliver a powerful solution to enable users across roles to access critical operational data using natural language.</p> 
<h2>Solution Overview</h2> 
<p>By leveraging <a href="https://aws.amazon.com/bedrock/agents/" rel="noopener" target="_blank">Agents for Amazon Bedrock</a>, we will build an agent that decomposes user requests into queries for AWS IoT SiteWise. This allows accessing operational data using natural language, without knowing query syntax or data storage. For example, a user can simply ask “What is the current RPM value for Turbine 1?” without using specific tools or writing code. The agent utilizes the contextualization layer in AWS IoT SiteWise for intuitive representations of industrial resources. See <a href="https://docs.aws.amazon.com/iot-sitewise/latest/userguide/how-sitewise-works.html" rel="noopener" target="_blank">How AWS IoT SiteWise works</a>&nbsp;for details on resource modeling.</p> 
<p><img alt="system architecture" class="wp-image-15301 size-full" height="449" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2024/05/20/IOTB-758-arch.png" width="1036" /></p> 
<p>From a chatbot interface, the user asks a natural language question that requires access to industrial asset data. The agent uses the <a href="https://swagger.io/specification/" rel="noopener" target="_blank">OpenAPI specification</a> (item 1) to orchestrate a plan for retrieving relevant data. It leverages an <a href="https://docs.aws.amazon.com/bedrock/latest/userguide/agents-action-create.html" rel="noopener" target="_blank">action group</a> defining queries the agent can perform (item 2), handled by an <a href="https://aws.amazon.com/lambda/" rel="noopener" target="_blank">AWS Lambda</a> function that uses the AWS IoT SiteWise <code><a href="https://docs.aws.amazon.com/iot-sitewise/latest/APIReference/API_ExecuteQuery.html" rel="noopener" target="_blank">ExecuteQuery API</a></code> (item 3). The agent may invoke multiple actions to execute the LLM’s plan until obtaining necessary data, e.g., querying property names, selecting the matching name, then querying recent measurements. Once provided the requested operational data, the model composes an answer to the original question (item 4).</p> 
<h2>Building the Agent</h2> 
<h3>Pre-requisites</h3> 
<ol> 
 <li>This solution leverages <a href="https://docs.aws.amazon.com/bedrock/latest/userguide/agents.html" rel="noopener" target="_blank">Agents for Amazon Bedrock</a>. See <a href="https://docs.aws.amazon.com/bedrock/latest/userguide/agents-supported.html" rel="noopener" target="_blank">Supported regions and models</a> for a current list of supported regions and foundation models. To enable access to Anthropic Claude models, you will need to enable <a href="https://docs.aws.amazon.com/bedrock/latest/userguide/model-access.html" rel="noopener" target="_blank">Model access</a>&nbsp;in Amazon Bedrock. The agent described in this blog was designed and tested for Claude 3 Haiku.</li> 
 <li>The agent uses the SiteWise SQL engine, which requires that AWS IoT SiteWise and <a href="https://aws.amazon.com/iot-twinmaker/" rel="noopener" target="_blank">AWS IoT TwinMaker</a> are integrated. Please follow&nbsp;<a href="https://docs.aws.amazon.com/iot-sitewise/latest/userguide/integrate-tm.html" rel="noopener" target="_blank">these steps</a>&nbsp;to create an AWS IoT TwinMaker workspace for AWS IoT SiteWise’s <code>ExecuteQuery</code> API.</li> 
 <li>The source code for this agent is available on <a href="https://github.com/aws-samples/aws-iot-sitewise-conversational-agent" rel="noopener" target="_blank">GitHub</a>.</li> 
</ol> 
<p>To clone the repository, run the following command:</p> 
<p><code>git clone&nbsp;https://github.com/aws-samples/aws-iot-sitewise-conversational-agent</code></p> 
<h3>Step 1: Deploy AWS IoT SiteWise assets</h3> 
<p>In this agent, AWS IoT SiteWise manages data storage, modeling, and aggregation, while Amazon Bedrock orchestrates multi-step actions to retrieve user-requested information. To begin, you will need real or simulated industrial assets streaming data into AWS IoT SiteWise. Follow the instructions on <a href="https://docs.aws.amazon.com/iot-sitewise/latest/userguide/getting-started.html" rel="noopener" target="_blank">Getting started with AWS IoT SiteWise</a> to ingest and model your industrial data, or use the <a href="https://docs.aws.amazon.com/iot-sitewise/latest/userguide/getting-started-demo.html" rel="noopener" target="_blank">AWS IoT SiteWise demo</a> to launch a simulated wind farm with four turbines. Note that the instructions on <strong>step 3</strong> and the sample questions in <strong>step 4</strong> were prepared for the simulated wind farm and, if using your own assets, you will have to prepare your own agent instructions and test questions.</p> 
<h3>Step 2: Define the action group</h3> 
<p>Before creating an agent in Amazon Bedrock, you need to define the action group: the actions that the agent can perform. This action group will specify the individual queries the agent can make to AWS IoT SiteWise while gathering required data. An action group requires:</p> 
<ul> 
 <li>An OpenAPI schema to define the API operations that the agent can invoke</li> 
 <li>A Lambda function that will take the API operations as inputs</li> 
</ul> 
<h3>Step 2.1: Design the OpenAPI specification</h3> 
<p>This solution provides API operations with defined paths that describe actions the agent can execute to retrieve data from recent operations. For example, the <code>GET /measurements/{AssetName}/{PropertyName}</code>&nbsp;path takes <code>AssetName</code>&nbsp;and <code>PropertyName</code>&nbsp;as parameters. Note the detailed description&nbsp;that informs the agent when and how to call the actions. Developers can add relevant paths to the <a href="https://github.com/aws-samples/aws-iot-sitewise-conversational-agent/blob/main/openapischema/iot_sitewise_agent_openapi_schema.json" rel="noopener" target="_blank">schema</a> to include actions (queries) relevant to their use cases.</p> 
<pre><code class="lang-json">  "paths": {
    "/measurements/{AssetName}/{PropertyName}": {
      "get": {
        "summary": "Get the latest measurement",
        "description": "Based on provided asset name and property name, return the latest measurement available",
        "operationId": "getLatestMeasurement",
        "parameters": [
          {
            "name": "AssetName",
            "in": "path",
            "description": "Asset Name",
            "required": true,
            "schema": {
              "type": "string"
            }
          },
          {
            "name": "PropertyName",
            "in": "path",
            "description": "Property Name",
            "required": true,
            "schema": {
              "type": "string"
            }
          }
        ]</code></pre> 
<p>Upload the <code>openapischema/iot_sitewise_agent_openapi_schema.json</code>&nbsp;file with the OpenAPI specification to Amazon S3. Copy the bucket and path because we are going to need that in <strong>step 3</strong>.</p> 
<h3>Step 2.2: Deploy the AWS Lambda function</h3> 
<p>The agent’s action group will be defined by an AWS Lambda function. The repository comes with a template to automatically deploy a serverless application built with the <a href="https://aws.amazon.com/serverless/sam/" rel="noopener" target="_blank">Serverless Application Model (SAM)</a>. To build and deploy, clone the <a href="https://github.com/aws-samples/aws-iot-sitewise-conversational-agent" rel="noopener" target="_blank">GitHub repository</a> and run the following commands from the main directory, where the <code>template.yaml</code> file is stored.</p> 
<div class="hide-language"> 
 <pre><code class="lang-bash">sam build --use-container
sam deploy --guided</code></pre> 
</div> 
<p>Follow the instructions from the prompt to complete the deployment.</p> 
<p>The <code>lambda_handler</code>&nbsp;function will read the API path from the invocation, and will call one of the following functions depending on the request. See the example below for the action defined for the <code>/measurements/{AssetName}/{PropertyName}</code>&nbsp;path, which calls the <code>get_latest_value</code>&nbsp;function where we use the SiteWise ExecuteQuery API to select the most recent observations for a user defined property. Notice that actions can be defined to return successful and unsuccessful HTTP status codes, and that the agent can use the error code to continue the conversation and prompt the user for clarification.</p> 
<pre><code class="lang-python">def lambda_handler(event, context):
    responses = []
    try:
        api_path = event['apiPath']
        logger.info(f'API Path: {api_path}')
        body = ""
        
        if api_path == "/measurements/{AssetName}/{PropertyName}":
            asset_name = _get_named_parameter(event, "AssetName")
            property_name = _get_named_parameter(event, "PropertyName")
            try:
                body = get_latest_value(sw_client, asset_name, property_name)
            except ValueError as e:
                return {
                    'statusCode': 404,
                    'body': json.dumps({'error': str(e)})
                }</code></pre> 
<p>Developers interested in expanding this agent can create new methods in the Lambda function to make their queries to the IoT SiteWise ExecuteQuery API, and map those methods to new paths. The ExecuteQuery API allows developers to run complex calculations with current and historical data, which can include aggregates, value filtering, and metadata filtering.</p> 
<h3>Step 3: Build the agent with Agents for Amazon Bedrock</h3> 
<p>Go to the Amazon Bedrock console, click on <code>Agents</code>&nbsp;under <code>Orchestration</code>, and then click on <code><a href="https://docs.aws.amazon.com/bedrock/latest/userguide/agents-create.html" rel="noopener" target="_blank">Create Agent</a></code>.&nbsp;Give your agent a meaningful name (e.g., <code>industrial-agent</code>) and select a model (e.g., Anthropic – Claude 3 Haiku).</p> 
<p>The most important part in the agent definition are the agent instructions, which will inform the agent of what it should do and how it should interact with users. Some best practices for agent instructions include:</p> 
<ul> 
 <li>Clearly defining purpose and capabilities upfront.</li> 
 <li>Specifying tone and formality level.</li> 
 <li>Instructing how to handle ambiguous or incomplete queries (e.g., ask for clarification).</li> 
 <li>Guiding how to gracefully handle out-of-scope queries.</li> 
 <li>Mentioning any specific domain knowledge or context to consider.</li> 
</ul> 
<p>If you deployed the wind turbines simulation from AWS IoT SiteWise in <strong>step 1</strong>, we recommend the following instructions. Remember that agent instructions are <strong>not optional</strong>.</p> 
<blockquote>
 <p>You are an industrial agent that helps operators get the most recent measurement available from their wind turbines. You are going to give responses in human-readable form, which means spelling out dates. Use clear, concise language in your responses, and ask for clarification if the query is ambiguous or incomplete. If no clear instruction is provided, ask for the name of the asset and the name of the property whose measurement we want to retrieve.&nbsp;If a query falls outside your scope, politely inform the user</p>
</blockquote> 
<p>Under <code>Action Groups</code>, select the Lambda function you created in <strong>step 3</strong>, and browse or enter the S3 URL that points to the API schema from <strong>step 2.1</strong>. Alternatively, you can directly enter the text from the API schema on the Bedrock console.</p> 
<p>Go to <code>Review and create</code>.</p> 
<h3>Step 4: Test the agent</h3> 
<p>The Amazon Bedrock console allows users to test agents in a conversational setting, view the thought process behind each interaction, and utilize <a href="https://docs.aws.amazon.com/bedrock/latest/userguide/advanced-prompts.html" rel="noopener" target="_blank"><code>Advanced prompts</code></a> to modify the pre-processing and orchestration templates automatically generated in the previous step.</p> 
<p>In the Amazon Bedrock console, select the agent and click on the <code>Test</code>&nbsp;button. A chat window will pop up.</p> 
<p>Try the agent to ask questions such as:</p> 
<ul> 
 <li>What wind turbine assets are available?</li> 
 <li>What are the properties for Turbine 1?</li> 
 <li>What is the current value for RPM?</li> 
</ul> 
<p>Notice that the agent can reason through the data from SiteWise and the chat history, often understanding the asset or property without being given the exact name. For instance, it recognizes <code>Turbine 1</code> as <code>Demo Turbine Asset 1</code> and <code>RPM</code> as <code>RotationsPerMinute</code>. To accomplish this, the agent orchestrates a plan: list available assets, list properties, and query based on the asset and property names stored in SiteWise, even if they do not match the user’s query verbatim.</p> 
<p><img alt="Q&amp;A interaction testing the agent" class="wp-image-15302 size-full" height="809" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2024/05/20/IOTB-758-response.png" style="margin: 10px 0px 10px 0px;" width="551" /></p> 
<p>The response given by the agent can always be tuned. By clicking the <code>Show trace</code> button, you can analyze the decision-making process and understand the agent’s reasoning. Additionally, you can modify the agent’s behavior by using <a href="https://docs.aws.amazon.com/bedrock/latest/userguide/advanced-prompts.html" rel="noopener" target="_blank">Advanced prompts</a> to edit the pre-processing, orchestration, or post-processing steps.</p> 
<p>Once confident in your agent’s performance, create an alias on the Amazon Bedrock console to deploy a draft version. Under <code>Agent details</code>, click <code>Create alias</code> to publish a new version. This allows chatbot applications to programmatically invoke the agent using <code><a href="https://docs.aws.amazon.com/bedrock/latest/APIReference/API_agent-runtime_InvokeAgent.html" rel="noopener" target="_blank">InvokeAgent</a></code>&nbsp;in the AWS SDK.</p> 
<p><img alt="Create an alias console" class=" wp-image-15304" height="473" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2024/05/20/IOTB-758-SCR-20240520-qdma-2.png" width="486" /></p> 
<h2>Conclusion</h2> 
<p>The generative AI agent discussed in this blog enables industrial companies to develop chatbots that can interact with operational data from their industrial assets. By leveraging AWS IoT SiteWise data connectors and models, the agent facilitates the consumption of operational data, integrating generative AI with Industrial IoT workloads. This industrial chatbot can be used alongside specialized agents or knowledge bases containing corporate information, machine data, and O&amp;M manuals. This integration provides the language model with relevant information to assist users in making critical business decisions through a single, user-friendly interface.</p> 
<h3>Call to action</h3> 
<p>Once your agent is ready, the next step is to build a user interface for your industrial chatbot. Visit this <a href="https://github.com/aws-samples/bedrock-claude-chat" rel="noopener" target="_blank">GitHub repository</a> to learn the components of a generative AI-powered chatbot and to explore sample code.</p> 
<h2>About the Authors</h2> 
<div class="blog-author-box" style="border: 1px solid #d5dbdb; padding: 15px;"> 
 <p class="samvas_headshot1.jpg"><img alt="gabemv-headshot" class="wp-image-15308 size-full alignleft" height="125" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2024/05/21/gabemv-headshot.jpg" width="110" /></p> 
 <h3 class="lb-h4">Gabriel Verreault</h3> 
 <p>Gabriel is a Senior Manufacturing Partner Solutions Architect at AWS. Gabriel works with global AWS partners to define, build, and evangelize solutions around Smart Manufacturing, OT, Sustainability and AI/ML. Prior to joining AWS, Gabriel worked with OSIsoft and AVEVA and has expertise in industrial data platforms, predictive maintenance, and how to combine AI/ML with industrial workloads.</p> 
</div> 
<div class="blog-author-box" style="border: 1px solid #d5dbdb; padding: 15px;"> 
 <p class="samvas_headshot1.jpg"><img alt="feliplp-headshot" class="wp-image-15306 size-full alignleft" height="125" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2024/05/21/feliplp-headshot.jpg" width="113" /></p> 
 <h3 class="lb-h4">Felipe Lopez</h3> 
 <p>Felipe is a Senior AI/ML Specialist Solutions Architect at AWS. Prior to joining AWS, Felipe worked with GE Digital and SLB, where he focused on modeling and optimization products for industrial applications.</p> 
</div> 
<div class="blog-author-box" style="border: 1px solid #d5dbdb; padding: 15px;"> 
 <p class="samvas_headshot1.jpg"><img alt="" class="alignleft wp-image-15307" height="125" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2024/05/21/avikg-headshot.jpg" width="94" /></p> 
 <h3 class="lb-h4">Avik Ghosh</h3> 
 <p>Avik is a Senior Product Manager on the AWS Industrial IoT team, focusing on the AWS IoT SiteWise service. With over 18 years of experience in technology innovation and product delivery, he specializes in Industrial IoT, MES, Historian, and large-scale Industry 4.0 solutions. Avik contributes to the conceptualization, research, definition, and validation of Amazon IoT service offerings.</p> 
</div>
