# Purpose of this documentation

The following documentation describes the steps that a required to connect to your Azure service bus and to send/read messages to your queues, using the connector from a scriptr.io script.

# 1. Configure the connector

Open the /azure/servicebus/config script and update the following variables with your corresponding values:

- defaultSubscriptionId: your Azure subscription id for the Azure offer (e.g "pay as you go", from portal.azure.com, search for "subscriptions") 
- defaultResourceGroupName: the name of one of your Azure resources group (from portal.azure.com, search for "Resource Group")
- hashKey: one of your Azure shared access keys for the service bus service.

In the **queueConfig** setup code, modify the default "/testqueue" and "/testopic" string to respectively match existing queue name and topic name in your Azure servic bus subscription (check the example below).

```
// replace /testqueue with the name of one of your queues in service bus
queueConfig[defaultNameSpace + "/testqueue"] = { 
    "resourceGroupName": defaultResourceGroupName,
    "apiVersion": defaultApiVersion,
    "subscriptionId": defaultSubscriptionId
};
// replace /testopic with the name of one of your topics in service bus
topicConfig[defaultNameSpace + "/testtopic"] = {
    "resourceGroupName": defaultResourceGroupName,
    "apiVersion": defaultApiVersion,
    "subscriptionId": defaultSubscriptionId
};
```

Once you are done with the configuration, create a new script in the scrptr.io workspace and execute the following steps.

# 2. Require all needed modules

First, you need to import the necessary classes of the connector into the script you are using to interact with service bus.

```
// this module handles the generation of Azure authentication tokens
// based on requested permissions scope
var tokengeneratorModule = require("/azure/authentication/tokengenerator");
// this module allows you to read/write messages from/to queues 
var queuemanagerModule = require("/azure/servicebus/queuemanager");
// this module allows you use your queues as topics (broadcast messgages)
//  and to read/write messages from/to topics 
var topicmanagerModule = require("/azure/servicebus/topicmanager");
// this is a generic client that handles the requests/response to/from Azure's service bus REST API
var clientModule = require("/azure/servicebus/client");
```

# Second step - generate an authentication token

Create an instance of the TokenGenerator class using the TokenGenerator() factory method of the tokenGeneratorModule
```
var tokengenerator = new tokengeneratorModule.TokenGenerator();
```
Then ask the token generator to generate an access token to provide access to the targeted resource (a queue or topic of the service bus)
```
var sigParams = {       
  resource: "https://your_azure_subdomain.servicebus.windows.net/your_queue_or_topic_name" 
};
var token = tokengenerator.generateSignature(sigParams);
```



