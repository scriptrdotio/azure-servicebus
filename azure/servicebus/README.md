# Purpose of this documentation

The following documentation describes the steps that a required to connect to your Azure service bus and to send/read messages to your queues, using the connector, from a scriptr.io script.

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

# 3. Generate an authentication token

Create an instance of the TokenGenerator class using the TokenGenerator() factory method of the tokenGeneratorModule
```
var tokengenerator = new tokengeneratorModule.TokenGenerator();
```
Then ask the token generator to generate an access token to provide access to the targeted resource (a queue or topic of the service bus)
```
var signatureParameters = {       
  resource: "https://your_azure_subdomain.servicebus.windows.net/your_queue_or_topic_name" 
};
var token = tokengenerator.generateSignature(signatureParameters);
```
# 4. Create an instance of the Client class

Create an instance of the generic Client class that handles the http requests and response to/from the service bus REST API. This instance will be passed to instances of the QueueManager and TopicManager classes.
```
 var client = new clientModule.Client(sigParams);
 ```
 
 # 5. Create an instance of a Queue and an instance of a Topic.
 
Instances of the Queue and Topic classes are used to interact with service bus queues and topics (a topic is a queue that can broadcast messages to multiple listeners - subscribers). You will usually obtain instance of these classes respectively from instances of the QueueManager and TopicManager classes. So let's first see how to create this instances:
 
```
// invoke the QueueManager() factory method of the queueManager module, passing the generic client
var queuemanager = new queuemanagerModule.QueueManager({client: client});
// invoke the TopicManager() factory method of the queueManager module, passing the generic client
var topicmanager = new topicmanagerModule.TopicManager({client: client});
```
To obtain a instance of a Queue class, proceed as follows using the instance of QueueManager:
```
// pass the name of the queue as a parameter of getQueue()
var queue = queuemanager.getQueue({queueName: "testqueue"});
```
To obtain a instance of a Topic class, proceed as follows using the instance of TopicManager:
```
// pass the name of the topci as a parameter of getTopic()
var topic = topicmanager.getTopic({topicName: "testtopic"});
```

# 6. Read messages from a queue and send messages to a queue

You can use the queue.readMessage() method to read a message from the queue
```
var msgObject = queue.readMessage(); // this removes the message from the queue
```
You can also do a *non-destructive* read (i.e. keep the message in the queue), using the queue.readLockMessage() method
```
var msgObjectStillInQueue = queue.readLockMessage ();
```
Use the queue.sendMessage() method to push a message to your service bus queue
```
var someMsgString = "a_string_can_be_a_stringified_json";
var result = queue.sendMessage(someMsgString);
```
You can also push a list of messages in one call:
```
var listOfMsgStrings = ["some_string1", "some_string2"];
var listPushResult = queue.sendMessageBatch(listOfMsgStrings);
```
More methods can be found in the [queue script](./azure/servicebus/queue)

# 7. Read messages from a topic and publish messages to a topic

You can use the topic.readMessage(subscriptionName) method to read a message from the topc
```
var msgObject = topic.readMessage("mysubscription"); // this removes the message from the topic
```
You can also do a *non-destructive* read (i.e. keep the message in the topic), using the topic.readLockMessage(subscriptionName) method
```
var msgObjectStillInTopic = topic.readLockMessage ("mysubscription");
```
Use the topic.sendMessage(subscription_name) method to push a message to your service bus queue
```
var someMsgString = "a_string_can_be_a_stringified_json";
var result = topic.sendMessage(someMsgString);
```
You can also push a list of messages in one call:
```
var listOfMsgStrings = ["some_string1", "some_string2"];
var listPushResult = topic.sendMessageBatch(listOfMsgStrings);
```
More methods can be found in the [topic script](./azure/servicebus/topic)
