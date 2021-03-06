---
title: Prise en charge d’AMQP 1.0 dans les rubriques et files d’attente Service Bus partitionnées | Microsoft Docs
description: Découvrez comment utiliser Advanced Message Queuing Protocol (AMQP) 1.0 dans les rubriques et files d’attente partitionnées Azure Service Bus.
services: service-bus
documentationcenter: .net
author: hillaryc
manager: timlt
editor: ''

ms.service: service-bus
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: multiple
ms.topic: article
ms.date: 07/08/2016
ms.author: hillaryc;sethm

---
# <a name="amqp-1.0-support-for-service-bus-partitioned-queues-and-topics"></a>Prise en charge d’AMQP 1.0 dans les rubriques et files d’attente partitionnées Service Bus
Azure Service Bus prend désormais en charge le protocole Advanced Message Queuing Protocol (**AMQP**) 1.0 pour les **files d’attente et rubriques partitionnées** Service Bus.

**AMQP** est un protocole standard de mise en file d’attente de messages ouvert qui vous permet de développer des applications interplateformes utilisant des langages de programmation différents. Pour obtenir des informations générales sur la prise en charge d’AMQP dans Service Bus, consultez [Prise en charge d’AMQP 1.0 dans Service Bus](service-bus-amqp-overview.md).

Les **files d’attente et rubriques partitionnées**, également appelées *entités partitionnées*, offrent une disponibilité, une fiabilité et un débit supérieurs à ceux des files d’attente et rubriques non partitionnées. Pour plus d’informations sur les entités partitionnées, consultez [Files d’attente et rubriques partitionnées](service-bus-partitioning.md).

L’ajout d’AMQP 1.0 en tant que protocole de communication avec des files d’attente et rubriques partitionnées vous permet de créer des applications interfonctionnelles exploitant la disponibilité, la fiabilité et le débit supérieurs des entités Service Bus partitionnées.

Pour un guide détaillé sur le protocole AMQP 1.0 au niveau des câbles qui explique comment Service Bus implémente et s’appuie sur la spécification technique AMQP OASIS, consultez le [Guide du protocole AMQP 1.0 dans Azure Service Bus et Event Hubs](service-bus-amqp-protocol-guide.md).    

## <a name="use-amqp-with-partitioned-queues"></a>Utilisation d’AMQP avec des files d’attente partitionnées
Les files d’attente sont utiles pour les scénarios qui requièrent le découplage temporel, le nivellement et l’équilibrage des charges et des couplages faibles. Avec une file d’attente, les éditeurs envoient des messages à la file d’attente et les consommateurs reçoivent des messages de la file d’attente dans des situations où un message ne peut être reçu qu’une fois. Un système d’inventaire dans lequel les terminaux de points de vente publient des données dans une file d’attente au lieu de les envoyer directement au système de gestion de stock en est un excellent exemple. Le système de gestion de stock consomme alors ces données à tout moment pour assurer le réapprovisionnement du stock. Cela offre plusieurs avantages. Par exemple, le système de gestion de stock n’a pas à être constamment en ligne. Pour plus d’informations sur les files d’attente Service Bus, consultez [Création d’applications qui utilisent des files d’attente Service Bus](service-bus-create-queues.md). 

Une file d’attente partitionnée peut augmenter la disponibilité, la fiabilité et le débit des applications, car elle est partitionnée entre plusieurs répartiteurs et banques de messages.     

### <a name="create-partitioned-queues"></a>Créer des files d’attente partitionnées
Vous pouvez créer une file d’attente partitionnée à l’aide du [portail Azure Classic][portail Azure Classic] et du Kit de développement logiciel (SDK) Service Bus. Pour créer une file d’attente partitionnée, définissez la propriété [EnablePartitioning](https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.queuedescription.enablepartitioning.aspx) sur **true** dans l’instance [QueueDescription](https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.queuedescription.aspx). Le code suivant montre comment créer une file d’attente partitionnée à l’aide du Kit de développement logiciel Service Bus. 

```
// Create partitioned queue
var nm = NamespaceManager.CreateFromConnectionString(myConnectionString);
var queueDescription = new QueueDescription("myQueue");
queueDescription.EnablePartitioning = true;
nm.CreateQueue(queueDescription);
```

### <a name="send-and-receive-messages-using-amqp"></a>Envoi et réception de messages à l’aide d’AMQP
Vous pouvez envoyer des messages à une file d’attente partitionnée et recevoir des messages de celle-ci à l’aide du protocole AMQP en définissant la propriété [TransportType](https://msdn.microsoft.com/library/azure/microsoft.servicebus.servicebusconnectionstringbuilder.transporttype.aspx) sur [TransportType.Amqp](https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.transporttype.aspx), comme indiqué dans le code suivant.  

```
// Sending and receiving messages to and from a queue
var myConnectionStringBuilder = new ServiceBusConnectionStringBuilder(myConnectionString);
myConnectionStringBuilder.TransportType = TransportType.Amqp;
string amqpConnectionString = myConnectionStringBuilder.ToString();
var queueClient = QueueClient.CreateFromConnectionString(amqpConnectionString, "myQueue");

BrokeredMessage message = new BrokeredMessage("Hello AMQP");
Console.WriteLine("Sending message {0}...", message.MessageId);
queueClient.Send(message);

var receivedMessage = queueClient.Receive();
Console.WriteLine("Received message: {0}", receivedMessage.GetBody<string>());
receivedMessage.Complete();
```

## <a name="use-amqp-with-partitioned-topics"></a>Utiliser AMQP avec des rubriques partitionnées
Le concept de rubriques est semblable à celui de file d’attente, sauf que les rubriques peuvent acheminer une copie du même message vers plusieurs *abonnements*. Dans une rubrique, les serveurs de publication envoient des messages à la rubrique et les consommateurs reçoivent des messages à partir d’abonnements. Dans le scénario de point de vente de système d’inventaire, des terminaux publient des données dans la rubrique. Le système de gestion d’inventaire reçoit alors les messages d’un abonnement. En outre, un système de surveillance peut recevoir le même message à partir d’un autre abonnement. Pour plus d’informations sur les abonnements et les rubriques Service Bus, consultez [Création d’applications qui utilisent des rubriques et des abonnements Service Bus](service-bus-create-topics-subscriptions.md). 

En ce qui concerne les files d’attente, les rubriques partitionnées peuvent augmenter la disponibilité, la fiabilité et le débit des applications, car elles sont réparties sur plusieurs répartiteurs de messages et banques de messagerie. 

### <a name="create-partitioned-topics"></a>Créer des rubriques partitionnées
Vous pouvez créer une rubrique partitionnée à l’aide du [portail Azure Classic][portail Azure Classic] et du Kit de développement logiciel (SDK) Service Bus. Pour créer une rubrique partitionnée, définissez la propriété [EnablePartitioning](https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.topicdescription.enablepartitioning.aspx) sur **true** dans l’instance [TopicDescription](https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.topicdescription.aspx). Le code suivant montre comment créer une rubrique partitionnée à l’aide du Kit de développement logiciel (SDK) Service Bus.

```
// Create partitioned topic
var nm = NamespaceManager.CreateFromConnectionString(myConnectionString);
var topicDescription = new TopicDescription("myTopic");
topicDescription.EnablePartitioning = true;
nm.CreateTopic(topicDescription);

var subscriptionDescription = new SubscriptionDescription("myTopic", "mySubscription");
nm.CreateSubscription(subscriptionDescription);
```

### <a name="send-and-receive-messages-using-amqp"></a>Envoi et réception de messages à l’aide d’AMQP
Vous pouvez envoyer des messages à un abonnement de rubrique partitionné et recevoir des messages de celui-ci à l’aide du protocole AMQP en définissant la propriété [TransportType](https://msdn.microsoft.com/library/azure/microsoft.servicebus.servicebusconnectionstringbuilder.transporttype.aspx) sur [TransportType.Amqp](https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.transporttype.aspx), comme indiqué dans le code suivant.  

```
// Sending and receiving messages to a topic and from a subscription
var myConnectionStringBuilder = new ServiceBusConnectionStringBuilder(myConnectionString);
myConnectionStringBuilder.TransportType = TransportType.Amqp;
string amqpConnectionString = myConnectionStringBuilder.ToString();

var topicClient = TopicClient.CreateFromConnectionString(amqpConnectionString, "myTopic");
BrokeredMessage message = new BrokeredMessage("Hello AMQP");
Console.WriteLine("Sending message {0}...", message.MessageId);
topicClient.Send(message);

var subcriptionClient = SubscriptionClient.CreateFromConnectionString(amqpConnectionString, "myTopic", "mySubscription");
var receivedMessage = subcriptionClient.Receive();
Console.WriteLine("Received message: {0}", receivedMessage.GetBody<string>());
receivedMessage.Complete();
```

## <a name="next-steps"></a>Étapes suivantes
Pour en savoir plus sur le partitionnement d’entités de messagerie et sur AMQP, consultez les informations supplémentaires suivantes.

* [Files d’attente et rubriques partitionnées](service-bus-partitioning.md)
* [OASIS Advanced Message Queuing Protocol (AMQP) Version 1.0](http://docs.oasis-open.org/amqp/core/v1.0/os/amqp-core-complete-v1.0-os.pdf) (Spécification OASIS Advanced Message Queuing Protocol (AMQP) Version 1.0)
* [Prise en charge d’AMQP 1.0 dans Service Bus](service-bus-amqp-overview.md)
* [Guide du protocole AMQP 1.0 dans Azure Service Bus et Event Hubs](service-bus-amqp-protocol-guide.md)
* [Utilisation de l’API Java Message Service (JMS) avec Service Bus et AMQP 1.0](service-bus-java-how-to-use-jms-api-amqp.md)
* [Utilisation d’AMQP 1.0 avec l’API .NET Service Bus](service-bus-dotnet-advanced-message-queuing.md)

[Portail Azure Classic]: http://manage.windowsazure.com



<!--HONumber=Oct16_HO2-->


