---
title: "Prise en main des représentations d’appareils | Microsoft Docs"
description: "Ce didacticiel vous montre comment utiliser les représentations d’appareils"
services: iot-hub
documentationcenter: node
author: fsautomata
manager: timlt
editor: 
ms.assetid: f7e23b6e-bfde-4fba-a6ec-dbb0f0e005f4
ms.service: iot-hub
ms.devlang: node
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 09/13/2016
ms.author: elioda
translationtype: Human Translation
ms.sourcegitcommit: 00746fa67292fa6858980e364c88921d60b29460
ms.openlocfilehash: 4b90b2529fa6d763ed3ce9c3b4da07afeb860f0e


---
# <a name="tutorial-get-started-with-device-twins"></a>Didacticiel : Prise en main des représentations d’appareil
[!INCLUDE [iot-hub-selector-twin-get-started](../../includes/iot-hub-selector-twin-get-started.md)]

À la fin de ce didacticiel, vous disposerez d’applications console Node.js et .NET :

* **AddTagsAndQuery.sln**, application .NET destinée à être exécutée à partir du serveur principal, qui ajoute des balises et interroge des représentations d’appareil.
* **TwinSimulatedDevice.js**, application Node.js qui simule un appareil se connectant à votre IoT Hub avec l’identité d’appareil créée précédemment, et signale son état de connectivité.

> [!NOTE]
> L’article [Kits de développement logiciel (SDK) Azure IoT][lnk-hub-sdks] fournit des informations sur les Kits de développement logiciel (SDK) IoT que vous pouvez utiliser pour générer des applications d’appareils et de backend.
> 
> 

Pour réaliser ce didacticiel, vous avez besoin des éléments suivants :

* Microsoft Visual Studio 2015.
* Node.js version 0.10.x ou ultérieure.
* Un compte Azure actif. (Si vous ne possédez pas de compte, vous pouvez créer un [compte gratuit][lnk-free-trial] en quelques minutes.)

[!INCLUDE [iot-hub-get-started-create-hub](../../includes/iot-hub-get-started-create-hub.md)]

[!INCLUDE [iot-hub-get-started-create-device-identity](../../includes/iot-hub-get-started-create-device-identity.md)]

## <a name="create-the-service-app"></a>Créer l’application de service
Dans cette section, vous créez une application console Node.js qui ajoute des métadonnées d’emplacement à la représentation d’appareil associée à **myDeviceId**. Elle interroge ensuite les représentations d’appareils stockées dans l’IoT hub en sélectionnant les appareils situés aux États-Unis, puis ceux qui signalent une connexion cellulaire.

1. Dans Visual Studio, ajoutez un projet Visual C# Bureau classique Windows à la solution actuelle en utilisant le modèle de projet **Application Console** . Nommez le projet **AddTagsAndQuery**.
   
    ![Nouveau projet Visual C# Bureau classique Windows][img-createapp]
2. Dans l’Explorateur de solutions, cliquez avec le bouton droit sur le projet **AddTagsAndQuery**, puis cliquez sur **Gérer les packages NuGet**.
3. Dans la fenêtre **Gestionnaire de package Nuget**, cliquez sur **Parcourir**, puis recherchez **microsoft.azure.devices**. Cliquez ensuite sur **Installer** pour installer le package **Microsoft.Azure.Devices**, puis acceptez les conditions d’utilisation. Cette procédure lance le téléchargement, l’installation et ajoute une référence au package Nuget [kit de développement logiciel (SDK) de service Microsoft Azure IoT][lnk-nuget-service-sdk] et ses dépendances.
   
    ![Fenêtre du gestionnaire de package Nuget][img-servicenuget]
4. Ajoutez les instructions `using` suivantes en haut du fichier **Program.cs** :
   
        using Microsoft.Azure.Devices;
5. Ajoutez les champs suivants à la classe **Program** . Remplacez la valeur d’espace réservé par la chaîne de connexion pour le IoT Hub créé dans la section précédente.
   
        static RegistryManager registryManager;
        static string connectionString = "{iot hub connection string}";
6. Ajoutez la méthode suivante à la classe **Program** :
   
        public static async Task AddTagsAndQuery()
        {
            var twin = await registryManager.GetTwinAsync("myDeviceId");
            var patch =
                @"{
                    tags: {
                        location: {
                            region: 'US',
                            plant: 'Redmond43'
                        }
                    }
                }";
            await registryManager.UpdateTwinAsync(twin.DeviceId, patch, twin.ETag);
   
            var query = registryManager.CreateQuery("SELECT * FROM devices WHERE tags.location.plant = 'Redmond43'", 100);
            var twinsInRedmond43 = await query.GetNextAsTwinAsync();
            Console.WriteLine("Devices in Redmond43: {0}", string.Join(", ", twinsInRedmond43.Select(t => t.DeviceId)));
   
            query = registryManager.CreateQuery("SELECT * FROM devices WHERE tags.location.plant = 'Redmond43' AND properties.reported.connectivity.type = 'cellular'", 100);
            var twinsInRedmond43UsingCellular = await query.GetNextAsTwinAsync();
            Console.WriteLine("Devices in Redmond43 using cellular network: {0}", string.Join(", ", twinsInRedmond43UsingCellular.Select(t => t.DeviceId)));
        }
   
    La classe **RegistryManager** expose toutes les méthodes requises pour interagir avec des représentations d’appareil à partir du service. Le code précédent initialise l’objet **registryManager**, récupère la représentation d’appareils de **myDeviceId**, puis met à jour ses balises avec les informations d’emplacement souhaitées.
   
    Après la mise à jour, il exécute deux requêtes : la première sélectionne uniquement les représentations des appareils situés dans l’usine **Redmond43**et la seconde affine la requête pour sélectionner uniquement les appareils qui sont également connectés via un réseau cellulaire.
   
    Notez que le code précédent, quand il crée l’objet **query**, spécifie un nombre maximal de documents retournés. L’objet **query** contient une propriété booléenne **HasMoreResults** permettant d’appeler les méthodes **GetNextAsTwinAsync** plusieurs fois afin de récupérer tous les résultats. Une méthode appelée **GetNextAsJson** est disponible pour les résultats qui ne sont pas des représentations d’appareil, par exemple, les résultats de requêtes d’agrégation.
7. Enfin, ajoutez les lignes suivantes à la méthode **Main** :
   
        registryManager = RegistryManager.CreateFromConnectionString(connectionString);
        AddTagsAndQuery().Wait();
        Console.WriteLine("Press Enter to exit.");
        Console.ReadLine();
8. Exécutez cette application, et vous devriez voir un appareil dans les résultats de la requête demandant tous les appareils situés à **Redmond43**, et aucun pour la requête limitant les résultats aux appareils utilisant un réseau cellulaire.
   
    ![Résultats de requête dans la fenêtre][img-addtagapp]

Dans la section suivante, vous allez créer une application d’appareil qui signale les informations de connectivité et modifie le résultat de la requête de la section précédente.

## <a name="create-the-device-app"></a>Créer l’application d’appareil
Dans cette section, vous créez une application console Node.js qui se connecte à votre hub en tant que **myDeviceId**, puis met à jour ses propriétés signalées afin qu’elles contiennent les informations indiquant qu’elle est connectée par le biais d’un réseau cellulaire.

1. Créez un dossier vide nommé **reportconnectivity**. Dans le dossier **reportconnectivity**, créez un fichier package.json en utilisant la commande suivante à l’invite de commandes. Acceptez toutes les valeurs par défaut :
   
    ```
    npm init
    ```
2. À l’invite de commandes, dans le dossier **reportconnectivity**, exécutez la commande suivante pour installer les packages **azure-iot-device** et **azure-iot-device-mqtt** :
   
    ```
    npm install azure-iot-device azure-iot-device-mqtt --save
    ```
3. À l’aide d’un éditeur de texte, créez un fichier **ReportConnectivity.js** dans le dossier **reportconnectivity**.
4. Ajoutez le code suivant au fichier **ReportConnectivity.js**, puis remplacez l’espace réservé **{device connection string}** par la chaîne de connexion que vous avez copiée lors de la création de l’identité d’appareil **myDeviceId** :
   
        'use strict';
        var Client = require('azure-iot-device').Client;
        var Protocol = require('azure-iot-device-mqtt').Mqtt;
   
        var connectionString = '{device connection string}';
        var client = Client.fromConnectionString(connectionString, Protocol);
   
        client.open(function(err) {
        if (err) {
            console.error('could not open IotHub client');
        }  else {
            console.log('client opened');
   
            client.getTwin(function(err, twin) {
            if (err) {
                console.error('could not get twin');
            } else {
                var patch = {
                    connectivity: {
                        type: 'cellular'
                    }
                };
   
                twin.properties.reported.update(patch, function(err) {
                    if (err) {
                        console.error('could not update twin');
                    } else {
                        console.log('twin state reported');
                        process.exit();
                    }
                });
            }
            });
        }
        });
   
    L’objet **Client** expose toutes les méthodes requises pour interagir avec des représentations d’appareil à partir de l’appareil. Le code précédent, après avoir initialisé l’objet **Client**, récupère la représentation d’objets de **myDeviceId**, puis met à jour sa propriété signalée avec les informations de connectivité.
5. Exécuter l’application d’appareil
   
        node ReportConnectivity.js
   
    Vous devriez voir le message `twin state reported`.
6. À présent que l’appareil a signalé ses informations de connectivité, il doit apparaître dans les deux requêtes. Exécutez l’application .NET **AddTagsAndQuery** pour exécuter des requêtes à nouveau. Cette fois, **myDeviceId** doit apparaître dans les résultats des deux requêtes.
   
    ![][img-addtagapp2]

## <a name="next-steps"></a>Étapes suivantes
Dans ce didacticiel, vous avez configuré un nouveau IoT Hub dans le portail Azure, puis créé une identité d’appareil dans le registre des identités de l’IoT Hub. Vous avez ajouté des métadonnées d’appareil en tant que balises à partir d’une application backend et écrit une application d’appareil simulé pour signaler des informations de connectivité d’appareil dans la représentation d’appareils. Vous avez également appris à interroger ces informations à l’aide du langage de requête IoT Hub similaire à celui de SQL.

Utilisez les ressources suivantes :

* Pour savoir comment envoyer la télémétrie d’appareils, voir le didacticiel [Prise en main d’IoT Hub][lnk-iothub-getstarted].
* Pour savoir comment configurer des appareils à l’aide des propriétés de représentation d’appareils souhaitées, voir le didacticiel [Utiliser des propriétés souhaitées pour configurer des appareils][lnk-twin-how-to-configure].
* Pour savoir comment contrôler des appareils de façon interactive (par exemple en mettant en marche un ventilateur à partir d’une application contrôlée par l’utilisateur), voir le didacticiel [Utiliser des méthodes directes][lnk-methods-tutorial].

<!-- images -->
[img-servicenuget]: media/iot-hub-csharp-node-twin-getstarted/servicesdknuget.png
[img-createapp]: media/iot-hub-csharp-node-twin-getstarted/createnetapp.png
[img-addtagapp]: media/iot-hub-csharp-node-twin-getstarted/addtagapp.png
[img-addtagapp2]: media/iot-hub-csharp-node-twin-getstarted/addtagapp2.png

<!-- links -->
[lnk-hub-sdks]: iot-hub-devguide-sdks.md
[lnk-free-trial]: http://azure.microsoft.com/pricing/free-trial/
[lnk-nuget-service-sdk]: https://www.nuget.org/packages/Microsoft.Azure.Devices/

[lnk-d2c]: iot-hub-devguide-messaging.md#device-to-cloud-messages
[lnk-methods]: iot-hub-devguide-direct-methods.md
[lnk-twins]: iot-hub-devguide-device-twins.md
[lnk-query]: iot-hub-devguide-query-language.md
[lnk-identity]: iot-hub-devguide-identity-registry.md

[lnk-iothub-getstarted]: iot-hub-node-node-getstarted.md
[lnk-methods-tutorial]: iot-hub-node-node-direct-methods.md
[lnk-twin-how-to-configure]: iot-hub-csharp-node-twin-how-to-configure.md

[lnk-dev-setup]: https://github.com/Azure/azure-iot-sdks/blob/master/doc/get_started/node-devbox-setup.md




<!--HONumber=Nov16_HO5-->


