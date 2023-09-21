When working with Azure IoT Edge, you have the advantage of utilizing Azure's built-in features for module-to-module communication. The Edge Hub is an Azure IoT Edge runtime module that manages communication between devices and the cloud, as well as between modules on the same device.

When using Azure IoT Edge modules, you do not need to set up MQTT directly as the communication between modules is handled by the Edge Hub using the built-in MQTT broker. Here's how you do it:

1. **Module-to-Module Communication**

   Each module can send messages to the Edge Hub, and then the Edge Hub can route these messages to other modules or to the cloud. This routing is configured using routing rules. 

2. **Create a Route**

   In the Azure portal, you can define a route that delivers messages from one module to another.

   For example, let's say you have two modules, `Module1` and `Module2`. `Module1` sends a message, and you want `Module2` to receive this message. You would go to your IoT Edge device in the Azure portal, and under `Routes`, add a new route with the following settings:

   - Name: `route1` (or any name you prefer)
   - Value: `FROM /messages/modules/Module1/outputs/* INTO BrokeredEndpoint("/modules/Module2/inputs/input1")`

   This route takes all messages outputted by `Module1` and delivers them to an input named `input1` on `Module2`.

3. **Use SDKs to Send/Receive Messages**

   In your modules, you would use one of the Azure IoT device SDKs (like C#, Node.js, Python, Java, or C) to send messages from `Module1` and receive messages in `Module2`.

   - `Module1` would send a message like this (using C# as an example):

     ```csharp
     var message = new Message(Encoding.ASCII.GetBytes("Hello, Module2"));
     await moduleClient.SendEventAsync("output1", message);
     ```

   - `Module2` would receive the message like this (using C# as an example):

     ```csharp
     await moduleClient.SetInputMessageHandlerAsync("input1", PipeMessage, iotHubModuleClient);
     ```

     Where `PipeMessage` is a method that processes incoming messages.

This process abstracts away the lower-level details of the MQTT protocol, making it easier to send and receive messages between modules. The Edge Hub takes care of the lower-level protocol details.