# LogicAppWorkshop

# Agenda

- Intro to Logic Apps and Azure Integration Services
- Development - Local and Portal
- Connectors and Workflows
- CI/CD
- Monitoring
- Networking
- Performance
- Security/Identity
- Roadmap

# Pre reqs
See [this link](https://learn.microsoft.com/en-us/azure/logic-apps/create-single-tenant-workflows-visual-studio-code)

1)	For the storage emulator, the doc above says to download and install Azurite (storage emulator). It’s easiest to do this as a Visual Code extension (which is mentioned in the link)
2)	Enable the public preview of Application Insights, details can be found at https://techcommunity.microsoft.com/t5/integrations-on-azure-blog/application-insights-enhancements-for-azure-logic-apps-standard/ba-p/3758909
3)	Enable the public preview of the new Visual Mapper, details can be found at https://techcommunity.microsoft.com/t5/integrations-on-azure-blog/announcement-azure-logic-apps-new-data-mapper-for-visual-studio/ba-p/3777138
4)	An Azure Subscription, with permission to add/edit/delete Azure resources including Logic Apps Standard, Application Insights, Functions, Storage etc.
5)	Access to Azure DevOps to edit/view pipelines etc.
6)	Postman client, details can be found at https://www.postman.com/downloads/
7)	Internet access, with ability to install software

# Challenge 1 - Logic Apps Development

Topics Covered:
- Portal Development
- VS Code Development
- Expressions
- Deploy to Azure


Challenge Details:

1) Create a Logic App in the Azure Portal
    - Create a stateful worflow with an HTTP request (the trigger is called "Request"), using  this [sample payload](Challenge%201/order_request.json) as the request body (click on the HTTP trigger and "Use sample payload to generate schema")
    - Add a response HTTP action and return the the request payload from the workflow (add the "Respone" action)
    - Test using Postman - to do this, navigate to the workflow being developed, go the the oviewview and copy the "Workflow URL". Load postman and create a new request using the local URL and pass in the [sample payload] in the order.json file. Make sure to set the Content-Type header to be "application/json"(Challenge%201/order_request.json)
    - View Run History, looking at the inputs and outputs for the triggers and actions
    - Explore mini map, zooming in/out and triggers/connectors
3) Create a Logic App in VS Code
    - Create a local project in VS Code by navigating to the Azure Account extension, which should be visible as the Azure icon. Navigate to the Azure extension, create a new project, then a stateful workflow
    - Using the [sample payload](Challenge%201/order_request.json) create the same workflow as above
    - Ensure the local storage emulator Azurite is running - press CTRL + SHIFT + P, then Azurite:Start
    - Test locally using Postman - to do this,  select "Run without Debugging" from the Run menu in VS Code, right click on the workflow in the project view and select "Overview" where the local url can be viewed. Load postman and create a new request using the local URL and pass in the [sample payload](Challenge%201/order_request.json). Make sure to set the Content-Type header to be "application/json"
    - Explore project structure - multiple workflows can be created within a single Logic App, allowing more complex workflows to be broken down so they can be re-used and tested independently
    - Enable schema validation for the HTTP request by clicking the request, going to Settings, Data Handling and select Schema Validation
    - Enable Azure Connectors and use the Office 365 Connector to send an email to yourself if the workflow is successful
4) Explore Expressions. Update the request message to add a new JSON property called "fullName". Use the Compose action and Concat expression to concatenate forenames and surname to the new field.
5) Deploy from VS Code to Azure. Test deploying directly from VS Code to Azure by selecting "Deploy to Logic App" from the Azure extension.
6) Export Logic App from Portal to VS Code
    - If the Logic Apps has been created in the portal, it can be exported to a project that can be loaded into VS Code
    - Through Azure Portal use "Download App Content" to download the project to a local folder
    - Within VS Code select Open Folder and navigate to the downloaded project folder
    - Run the project locally and test using Postman.
7) Local/remote config
    - Review connections.json
    - Review local.settings.json.



## Optional Extras

Now we'd like you to call an API from a Logic App and then transform the result that comes back. For this optional challenge we will use the Star Wars API: https://swapi.dev/

To succeed in this challenge we would like you to call the people API and return a message from a Star Wars character everytime the Logic App is triggered. Please provide whatever information about the character you like, but we will need to know 4 attributes about the character. The characters must be chosen at random. Good luck!


# Challenge 2 - Development Practice

Topics Covered:
- Connector types - Builtin and Azure Managed
- Using child workflows
- Workflow types - Stateful and Stateless
- Configuration

Challenge Scenario:

Contoso Retail receive orders  in JSON format from a service bus topic which must then be enriched by calling an internal API, returning details of stock levels. The original order message and stock message need to be mapped into a new payload to be sent to the Contoso order fulfillment API. The message must also be sent to another internal Service Bus topic.

1) Service Bus
    - Create a Azure Service Bus instance in the Standard tier, then create a single topic called "OrderTopic" and subscription "OrderTopicSubsription"
    - Also create a topic called "fulfilOrderTopic"
    - Make a note of the Connection String, available under "Shared Access Policies". You may need to create a Shared Access Policy by clicking Add and selecting Send and Listen. Once created, copy the Primary Connection String for future use. Note: in a production scenario, access policies would be set at the queue or topic level, or would use Azure AD authentication using RBAC.
2)  Create a Stateful Logic App workflow in VS Code to trigger from the Service Bus topic, using the In App Service Bus trigger to receive the message
3) To trigger the workflow there are three ways to push a message to the topic:
    - Use Service Bus Explorer (available at https://github.com/paolosalvatori/ServiceBusExplorer) to send the message
    - Use Service Bus Explorer available in the Azure Portal. Navigate to the Service Bus instance in the Azure Portal, then to the topic and click Service Bus Explorer. Here, select a content type of application/json and paste in the [sample payload](Challenge%201/order_request.json)
    - Create a new workflow within the Logic App triggered by HTTP, then add a Service Bus action to send a message to a topic, then use the payload from the HTTP action as the input to the Service Bus action. 
4) Run the Logic App locally within VS Code (note: if you see an error saying the port is in-use, open a cmd promot and enter the following command; taskkill /im func.exe /f)
5) Review the run history for the workflow to verify the workflow was triggered
6) We now need to call the Stock API check that there is enough stock to fulfill the order. Create an HTTP action, with an HTTP GET to https://davephelpsapim.azure-api.net/contoso/order, passing the product productDetails.Id to the stock serice in query string paramter called "productId"
7) Add a Compose action to create a new message containing the request content and the stock level returned from the stock API (don't worry about what the fields are called)
8) Finally, add an action to call the Order API passing in the new message to https://davephelpsapim.azure-api.net/contoso/order. As there is an HTTP body, use an HTTP POST and Add two headers, "Content-Type" with a value of "application/json" and "Ocp-Apim-Subscription-Key" with a security key that will be provided to you
9) Rename the actions to be something more meaningful, such as "Order API" and "Get Stock"
10) Test the API to make sure the APIs are being called correclty and the new message is in the correct format
11) Finally, create a new action to send the new message to the new topic called "fulfilOrderTopic"
12) Test the workflow again to ensure the Service Bus action was successful sending a message to the "fulfilOrderTopic" topic 


# Challenge 3 - Creating a CI/CD pipeline

Contoso retail want to automate the process of deploying their logic apps. As part of this challenge you will need to set up a CI/CD process for Azure DevOps and store the code in a repository.

Note: the CI/CD process for Azure Logic Apps is very similar to that of Azure Functions so should be familiar to anyone who has done this before.

1) Create a repo for your code either in GitHub or Azure Devops Repos (Azure DevOps repos is more straightforward but use GitHub if you are familiar with it)
2) Initialise a local repo where your Logic Apps solution is stored, then synchronise with the remote repo
3) Create two YAML based pipelines in Azure DevOps to deploy your Logic App, one for the Logic Apps resource (infra) and one for the Workflows (code). Tip: create a folder called "infra" to store the infra pipeline so it can be triggered when there are any changes.
3) Use variables in pipelines for API keys and URLs

Tips:
1) The task is Azure DevOps is called "AzureFunctionApp@1" and the appType is "workflowApp"
2) Review https://learn.microsoft.com/en-us/azure/logic-apps/set-up-devops-deployment-single-tenant-azure-logic-apps?tabs=azure-devops#release-to-azure for details

For a full CI/CD process, review the following complete solution:
https://github.com/jordanbean-msft/logic-sb-integration

