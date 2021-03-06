---
title: Utilisation d’un client HTML avec Azure Mobile Services | Microsoft Docs
description: Découvrez comment utiliser un client HTML pour Azure Mobile Services.
services: mobile-services
documentationcenter: ''
author: ggailey777
manager: dwrede
editor: ''

ms.service: mobile-services
ms.workload: mobile
ms.tgt_pltfrm: mobile-html
ms.devlang: javascript
ms.topic: article
ms.date: 07/21/2016
ms.author: glenga

---
# Utilisation d'un client HTML/JavaScript pour Azure Mobile Services
[!INCLUDE [mobile-services-selector-client-library](../../includes/mobile-services-selector-client-library.md)]

## Vue d'ensemble
Ce guide décrit le déroulement de scénarios courants dans le cadre de l'utilisation d'un client HTML/JavaScript pour Azure Mobile Services, qui inclut des applications PhoneGap/Cordova et JavaScript Windows Store. Les scénarios traités incluent l'interrogation des données, l'insertion, la mise à jour et la suppression des données, l'authentification des utilisateurs et la gestion des erreurs. Si vous débutez avec Mobile Services, suivez le didacticiel [Démarrage rapide Mobile Services](mobile-services-html-get-started.md). Ces didacticiels de démarrage rapide vous aideront à configurer et à créer votre premier service mobile.

[!INCLUDE [mobile-services-concepts](../../includes/mobile-services-concepts.md)]

## <a name="create-client"></a>Procédure : création du client Mobile Services
La façon dont vous ajoutez une référence au client Mobile Services dépend de votre plateforme d'application :

* Pour une application web, ouvrez le fichier HTML et ajoutez ce qui suit aux références de script de la page :
  
        <script src="http://ajax.aspnetcdn.com/ajax/mobileservices/MobileServices.Web-1.2.7.min.js"></script>
* Pour une application Windows Store écrite en JavaScript/HTML, ajoutez le package **WindowsAzure.MobileServices.WinJS** à votre projet.
* Pour une application PhoneGap ou Cordova, ajoutez le [plug-in Mobile Services](https://github.com/Azure/azure-mobile-services-cordova) à votre projet. Ce plug-in prend en charge les [notifications Push](#push-notifications).

Dans l'éditeur, ouvrez ou créez un fichier JavaScript, ajoutez le code suivant qui définit la variable `MobileServiceClient` et fournissez l'URL et la clé d'application du service mobile dans le constructeur `MobileServiceClient`, dans cet ordre.

    var MobileServiceClient = WindowsAzure.MobileServiceClient;
    var client = new MobileServiceClient('AppUrl', 'AppKey');

Vous devez remplacer l’espace réservé `AppUrl` par l’URL d’application de votre service mobile et `AppKey` par la clé d’application, que vous récupérez du [portail Azure Classic](http://manage.windowsazure.com/).

> [!IMPORTANT]
> La clé d'application est destinée à filtrer la demande aléatoire sur votre service mobile ; elle est distribuée avec l'application. Cette clé n'étant pas chiffrée, elle ne peut pas être considérée comme sécurisée. Pour vraiment sécuriser l'accès aux données de votre service mobile, vous devez authentifier les utilisateurs avant de leur autoriser l'accès. Pour plus d'informations, consultez [Procédure : authentification des utilisateurs](#authentication).
> 
> 

## <a name="querying"></a>Procédure : interrogation des données à partir d'un service mobile
L'ensemble du code qui permet d'accéder aux données de la table Base de données SQL ou de les modifier appelle des fonctions sur l'objet `MobileServiceTable`. Pour obtenir une référence à la table, appelez la fonction `getTable()`sur une instance du `MobileServiceClient`.

    var todoItemTable = client.getTable('todoitem');


### <a name="filtering"></a>Procédure : filtrage des données renvoyées
Le code suivant montre comment filtrer des données en incluant une clause `where` dans une requête. Il renvoie tous les éléments de `todoItemTable` dont le champ complété est égal à `false`. `todoItemTable` est la référence à la table de service mobile que vous avez créée précédemment. La fonction where applique un prédicat de filtrage de ligne à la requête au niveau de la table. Elle accepte comme argument une fonction ou un objet JSON qui définit le filtre de ligne et renvoie une requête qui peut être composée de manière plus approfondie.

    var query = todoItemTable.where({
        complete: false
    }).read().done(function (results) {
        alert(JSON.stringify(results));
    }, function (err) {
        alert("Error: " + err);
    });

En appelant `where` sur l’objet Query et en transmettant un objet comme paramètre, vous indiquez à Mobile Services de renvoyer uniquement les lignes dont la colonne `complete` contient la valeur `false`. Étudiez également l'URI de requête ci-dessous. Vous remarquerez que nous modifions la chaîne de requête elle-même :

    GET /tables/todoitem?$filter=(complete+eq+false) HTTP/1.1

Vous pouvez afficher l'URI de la requête envoyée au service mobile en utilisant un logiciel d'inspection des messages, tel que les outils destinés aux développeurs de navigateurs ou Fiddler.

Cette requête serait normalement convertie approximativement sous forme de requête SQL côté serveur, comme suit :

    SELECT *
    FROM TodoItem
    WHERE ISNULL(complete, 0) = 0

L'objet qui est transmis à la méthode `where` peut avoir un nombre arbitraire de paramètres qui seront interprétés en tant que clauses AND dans la requête. Par exemple, la ligne ci-dessous :

    query.where({
       complete: false,
       assignee: "david",
       difficulty: "medium"
    }).read().done(function (results) {
       alert(JSON.stringify(results));
    }, function (err) {
       alert("Error: " + err);
    });

Serait approximativement convertie (pour la même requête indiquée plus haut) comme suit

    SELECT *
    FROM TodoItem
    WHERE ISNULL(complete, 0) = 0
          AND assignee = 'david'
          AND difficulty = 'medium'

L'instruction `where` et la requête SQL ci-dessus recherchent les éléments incomplets attribués à "david" avec une difficulté "medium".

Il existe néanmoins une autre manière d'écrire la même requête. Comme un appel `.where` sur l'objet Query ajoute une expression `AND` à la clause `WHERE`, nous pourrions écrire ceci en trois lignes :

    query.where({
       complete: false
    });
    query.where({
       assignee: "david"
    });
    query.where({
       difficulty: "medium"
    });

Ou à l'aide de l'API Fluent :

    query.where({
       complete: false
    })
       .where({
       assignee: "david"
    })
       .where({
       difficulty: "medium"
    });

Les deux méthodes sont équivalentes et peuvent être utilisées de manière interchangeable. Tous les appels `where` évoqués jusqu'à maintenant prennent un objet avec quelques paramètres et leur égalité est comparée par rapport aux données de la base de données. Il y a toutefois une autre surcharge pour la méthode de requête, qui prend une fonction au lieu de l'objet. Dans cette fonction, vous pouvez écrire des expressions plus complexes, en utilisant des opérateurs tels que l'inégalité et d'autres opérations relationnelles. Dans ces fonctions, le mot clé `this` est lié à l'objet serveur.

Le corps de la fonction est converti en expression booléenne OData (Open Data Protocol) qui est transmise à un paramètre de chaîne de requête. Il est possible de transmettre une fonction qui ne prend aucun paramètre, comme suit :

    query.where(function () {
       return this.assignee == "david" && (this.difficulty == "medium" || this.difficulty == "low");
    }).read().done(function (results) {
       alert(JSON.stringify(results));
    }, function (err) {
       alert("Error: " + err);
    });


En cas de transmission d'une fonction sans paramètre, les arguments situés après la clause `where` sont liés aux paramètres de fonction dans l'ordre. Les objets extérieurs à la portée de la fonction DOIVENT être transmis en tant que paramètres. La fonction ne peut pas capturer de variables externes. Dans les deux exemples suivants, l'argument "david" est lié au paramètre `name`, et dans le premier exemple, l'argument "medium" est également lié au paramètre `level`. En outre, la fonction doit se composer d'une instruction `return` simple avec une expression prise en charge, comme suit :

     query.where(function (name, level) {
        return this.assignee == name && this.difficulty == level;
     }, "david", "medium").read().done(function (results) {
        alert(JSON.stringify(results));
     }, function (err) {
        alert("Error: " + err);
     });

Aussi, tant que vous suivez les règles, vous pouvez ajouter des filtres plus complexes à vos requêtes de base de données, comme suit :

    query.where(function (name) {
       return this.assignee == name &&
          (this.difficulty == "medium" || this.difficulty == "low");
    }, "david").read().done(function (results) {
       alert(JSON.stringify(results));
    }, function (err) {
       alert("Error: " + err);
    });

Il est possible de combiner `where` avec `orderBy`, `take` et `skip`. Pour plus d'informations, consultez la section suivante.

### <a name="sorting"></a>Procédure : tri des données renvoyées
Le code suivant montre comment trier les données en incluant une fonction `orderBy` ou `orderByDescending` dans la requête. Il renvoie des éléments de `todoItemTable`, triés dans l'ordre croissant par le champ `text`. Par défaut, le serveur renvoie uniquement les 50 premiers éléments.

> [!NOTE]
> Une taille de page pilotée par un serveur est utilisée par défaut pour empêcher le renvoi de tous les éléments. Cela permet d'éviter que les requêtes par défaut associées à des jeux de données volumineux aient un impact négatif sur le service. Vous pouvez augmenter le nombre d'éléments à renvoyer en appelant `take`, en suivant les instructions de la section suivante. `todoItemTable` est la référence à la table de service mobile que vous avez créée précédemment.
> 
> 

    var ascendingSortedTable = todoItemTable.orderBy("text").read().done(function (results) {
       alert(JSON.stringify(results));
    }, function (err) {
       alert("Error: " + err);
    });

    var descendingSortedTable = todoItemTable.orderByDescending("text").read().done(function (results) {
       alert(JSON.stringify(results));
    }, function (err) {
       alert("Error: " + err);
    });

    var descendingSortedTable = todoItemTable.orderBy("text").orderByDescending("text").read().done(function (results) {
       alert(JSON.stringify(results));
    }, function (err) {
       alert("Error: " + err);
    });

### <a name="paging"></a>Procédure : renvoi de données dans les pages
Par défaut, Mobile Services retourne uniquement 50 lignes dans une requête donnée, sauf si le client demande explicitement davantage de données dans la réponse. Le code suivant montre comment implémenter la pagination des données renvoyées à l'aide des clauses `take` et `skip` dans la requête. Lorsqu'elle est exécutée, la requête suivante renvoie les trois premiers éléments de la table.

    var query = todoItemTable.take(3).read().done(function (results) {
       alert(JSON.stringify(results));
    }, function (err) {
       alert("Error: " + err);
    });

La méthode `take(3)` a été convertie en option de requête `$top=3` dans l'URI de requête.

La requête révisée ci-dessous ignore les trois premiers résultats et renvoie les trois résultats suivants. Il s'agit en fait de la deuxième « page » de données, dont la taille est de trois éléments.

    var query = todoItemTable.skip(3).take(3).read().done(function (results) {
       alert(JSON.stringify(results));
    }, function (err) {
       alert("Error: " + err);
    });

Là aussi, vous pouvez afficher l'URI de la requête envoyée au service mobile. La méthode `skip(3)` a été convertie en option de requête `$skip=3` dans l'URI de requête.

Il s'agit d'un scénario simplifié dans lequel les valeurs de pagination codées en dur sont transmises aux fonctions `take` et `skip`. Dans une application réelle, vous pouvez utiliser des requêtes semblables à celles indiquées plus haut avec un contrôle pager ou une interface utilisateur comparable pour permettre aux utilisateurs d'accéder aux pages précédentes et suivantes.

### <a name="selecting"></a>Procédure : sélection de colonnes spécifiques
Vous pouvez indiquer le jeu de propriétés à inclure dans les résultats en ajoutant une clause `select` à la requête. Par exemple, le code suivant renvoie les propriétés `id`, `complete` et `text` de chaque ligne dans la `todoItemTable` :

    var query = todoItemTable.select("id", "complete", "text").read().done(function (results) {
       alert(JSON.stringify(results));
    }, function (err) {
       alert("Error: " + err);
    })

Ici, les paramètres associés à la fonction select sont les noms des colonnes de table à renvoyer.

Toutes les fonctions décrites jusqu'ici étant cumulatives, nous pouvons continuer de les appeler, ce qui aura à chaque fois des répercussions sur la requête. Un exemple supplémentaire :

    query.where({
       complete: false
    })
       .select('id', 'assignee')
       .orderBy('assignee')
       .take(10)
       .read().done(function (results) {
       alert(JSON.stringify(results));
    }, function (err) {
       alert("Error: " + err);

### <a name="lookingup"></a>Procédure : recherche de données par ID
La fonction `lookup` prend uniquement la valeur `id` et renvoie l'objet de la base de données avec cet ID. Les tables de base de données sont créées avec une colonne `id` sous forme d'entier ou de chaîne. Par défaut, la colonne `id` est affichée sous forme de chaîne.

    todoItemTable.lookup("37BBF396-11F0-4B39-85C8-B319C729AF6D").done(function (result) {
       alert(JSON.stringify(result));
    }, function (err) {
       alert("Error: " + err);
    })

## <a name="odata-query"></a>Exécution d'une opération de requête OData
Mobile Services utilise les conventions URI de requête OData pour composer et exécuter des requêtes REST. Toutes les requêtes OData ne peuvent pas être composées à l'aide des fonctions de requête intégrées, en particulier les opérations complexes portant sur les filtres, par exemple la recherche d'une sous-chaîne dans une propriété. Pour ces requêtes complexes, vous pouvez transmettre n'importe quelle chaîne option valide de requête OData à la fonction `read`, comme ceci :

    function refreshTodoItems() {
        todoItemTable.read("$filter=substringof('search_text',text)").then(function(items) {
            var itemElements = $.map(items, createUiForTodoItem);
            $("#todo-items").empty().append(itemElements);
            $("#no-items").toggle(items.length === 0);
        }, handleError);
    }

> [!NOTE]
> Si vous fournissez une chaîne option brute de requête OData dans la fonction `read`, vous ne pouvez pas utiliser également les méthodes du générateur de requêtes dans la même requête. Dans ce cas, vous devez composer toute la requête sous forme de chaîne option OData. Pour plus d'informations sur les options de requête système OData, consultez la page [Référence des options de requête système OData].
> 
> 

## <a name="inserting"></a>Procédure : insertion de données dans un service mobile
Le code suivant montre comment insérer de nouvelles lignes dans une table. Le client demande à ce qu'une ligne de données soit insérée en envoyant une requête POST au service mobile. Le corps de la requête contient les données à insérer, sous forme d'objet JSON.

    todoItemTable.insert({
       text: "New Item",
       complete: false
    })

Cela permet d'insérer des données de l'objet JSON fourni dans la table. Vous pouvez également spécifier une fonction de rappel à appeler à la fin de l'insertion :

    todoItemTable.insert({
       text: "New Item",
       complete: false
    }).done(function (result) {
       alert(JSON.stringify(result));
    }, function (err) {
       alert("Error: " + err);
    });

### Utilisation de valeurs d'ID
Mobile Services prend en charge les valeurs de chaîne personnalisée uniques pour la colonne **id** de la table. Cela permet aux applications d’utiliser des valeurs personnalisées telles que les adresses de messagerie ou des noms d’utilisateur pour l’ID. Par exemple, le code suivant insère un nouvel élément en tant qu'objet JSON, où l'ID unique est une adresse de messagerie :

    todoItemTable.insert({
       id: "myemail@domain.com",
       text: "New Item",
       complete: false
    });

Les ID de chaîne fournissent les avantages suivants :

* Les ID sont générés sans effectuer d’aller-retour vers la base de données.
* Il est plus facile de fusionner des enregistrements de plusieurs tables ou bases de données.
* Les valeurs d’ID peuvent mieux s’intégrer à la logique d’une application.

Quand une valeur d'ID de chaîne n'est pas déjà définie sur un enregistrement inséré, Mobile Services génère une valeur unique pour l'ID. Pour plus d'informations sur la façon de générer vos propres valeurs d'ID, sur le client ou dans un backend .NET, consultez [Procédure : génération de valeurs d'ID uniques](mobile-services-how-to-use-server-scripts.md#generate-guids).

Vous pouvez également utiliser des ID d’entier pour vos tables. Pour pouvoir utiliser un ID d'entier, vous devez créer votre table avec la commande `mobile table create` et l'option `--integerId`. Cette commande s'utilise avec l'interface de ligne de commande (CLI) pour Azure. Pour plus d'informations sur l'utilisation de l'interface de ligne de commande, consultez la page [Interface de ligne de commande pour la gestion des tables Mobile Services](../virtual-machines-command-line-tools.md#Mobile_Tables).

## <a name="modifying"></a>Procédure : modification de données dans un service mobile
Le code suivant montre comment mettre à jour les données d'une table. Le client demande à ce qu'une ligne de données soit mise à jour en envoyant une requête PATCH au service mobile. Le corps de la requête contient les champs spécifiques à mettre à jour, sous forme d'objet JSON. Il met à jour un élément existant dans la table `todoItemTable`.

    todoItemTable.update({
       id: idToUpdate,
       text: newText
    })

Le premier paramètre spécifie l'instance à mettre à jour dans la table, comme spécifié par son ID.

Vous pouvez également spécifier une fonction de rappel à appeler à la fin de la mise à jour :

    todoItemTable.update({
       id: idToUpdate,
       text: newText
    }).done(function (result) {
       alert(JSON.stringify(result));
    }, function (err) {
       alert("Error: " + err);
    });

## <a name="deleting"></a>Procédure : suppression de données dans un service mobile
Le code suivant montre comment supprimer les données d'une table. Le client demande à ce qu'une ligne de données soit supprimée en envoyant une requête DELETE au service mobile. Il supprime un élément existant de la table todoItemTable.

    todoItemTable.del({
       id: idToDelete
    })

Le premier paramètre spécifie l'instance à supprimer dans la table, comme spécifié par son ID.

Vous pouvez également spécifier une fonction de rappel à appeler à la fin de la suppression :

    todoItemTable.del({
       id: idToDelete
    }).done(function () {
       /* Do something */
    }, function (err) {
       alert("Error: " + err);
    });

## <a name="binding"></a>Procédure : affichage des données dans l'interface utilisateur
Cette section montre comment afficher des objets de données renvoyés à l'aide d'éléments d'interface utilisateur. Pour interroger les éléments de la table `todoItemTable` et afficher celle-ci sous forme de liste toute simple, vous pouvez exécuter l'exemple de code suivant. Aucune sélection, ni aucun filtre ou tri n’est effectué.

    var query = todoItemTable;

    query.read().then(function (todoItems) {
       // The space specified by 'placeToInsert' is an unordered list element <ul> ... </ul>
       var listOfItems = document.getElementById('placeToInsert');
       for (var i = 0; i < todoItems.length; i++) {
          var li = document.createElement('li');
          var div = document.createElement('div');
          div.innerText = todoItems[i].text;
          li.appendChild(div);
          listOfItems.appendChild(li);
       }
    }).read().done(function (results) {
       alert(JSON.stringify(results));
    }, function (err) {
       alert("Error: " + err);
    });

Dans une application Windows Store, les résultats d’une requête peuvent servir à créer un objet [WinJS.Binding.List], qui peut être lié comme source de données d’un objet [ListView]. Pour plus d’informations, consultez la page [Liaison de données (applications du Windows Store en JavaScript et HTML)].

## <a name="custom-api"></a>Procédure : appel d’une API personnalisée
Une API personnalisée vous permet de définir des points de terminaison exposant une fonctionnalité de serveur qui ne mappe pas vers une opération d'insertion, de mise à jour, de suppression ou de lecture. En utilisant une API personnalisée, vous pouvez exercer davantage de contrôle sur la messagerie, notamment lire et définir des en-têtes de message HTTP et définir un format de corps de message autre que JSON. Pour obtenir un exemple montrant comment créer une API personnalisée dans votre service mobile, consultez [Procédure : définition d’un point de terminaison dans une API personnalisée](mobile-services-dotnet-backend-define-custom-api.md).

Vous appelez une API personnalisée à partir du client en appelant la méthode [invokeApi](https://github.com/Azure/azure-mobile-services/blob/master/sdk/Javascript/src/MobileServiceClient.js#L337) sur **MobileServiceClient**. Par exemple, la ligne de code suivante envoie une requête POST à l'API **completeAll** sur le service mobile :

    client.invokeApi("completeall", {
        body: null,
        method: "post"
    }).done(function (results) {
        var message = results.result.count + " item(s) marked as complete.";
        alert(message);
        refreshTodoItems();
    }, function(error) {
        alert(error.message);
    });


Pour des exemples plus réalistes et une discussion plus élaborée sur **invokeApi**, consultez la section [API personnalisée dans les Kits de développement logiciel (SDK) clients pour Azure Mobile Services](http://blogs.msdn.com/b/carlosfigueira/archive/2013/06/19/custom-api-in-azure-mobile-services-client-sdks.aspx).

## <a name="authentication"></a>Procédure : authentification des utilisateurs
Mobile Services prend en charge l'authentification et l'autorisation des utilisateurs d'applications via divers fournisseurs d'identité externes : Facebook, Google, Microsoft Account et Twitter. Vous pouvez définir des autorisations sur les tables pour limiter l'accès à certaines opérations aux seuls utilisateurs authentifiés. Vous pouvez également utiliser l’identité des utilisateurs authentifiés pour implémenter des règles d’autorisation dans les scripts serveur. Pour plus d’informations, consultez la page [Prise en main de l’authentification].

> [!NOTE]
> Quand vous utilisez l’authentification dans une application PhoneGap ou Cordova, vous devez également ajouter les plug-ins suivants au projet :
> 
> * https://git-wip-us.apache.org/repos/asf/cordova-plugin-device.git
> * https://git-wip-us.apache.org/repos/asf/cordova-plugin-inappbrowser.git
> 
> 

Deux flux d'authentification sont pris en charge : un *flux serveur* et un *flux client*. Le flux serveur fournit l'authentification la plus simple, car il repose sur l'interface d'authentification Web du fournisseur. Le flux client permet une intégration approfondie avec les fonctionnalités propres aux appareils, telles que l'authentification unique, car il repose sur des Kits de développement logiciel (SDK) propres aux appareils et aux fournisseurs.

### Flux serveur
Pour que Mobile Services gère le processus d'authentification dans votre application Windows Store ou HTML5, vous devez inscrire votre application auprès de votre fournisseur d'identité. Ensuite, dans votre service mobile, vous devez configurer l'ID d'application et le secret fournis par votre fournisseur. Pour plus d’informations, consultez le didacticiel [Ajout de l’authentification à votre application](mobile-services-html-get-started-users.md).

Une fois que vous avez inscrit votre fournisseur d’identité, appelez simplement la [méthode LoginAsync] avec la valeur [MobileServiceAuthenticationProvider] de votre fournisseur. Par exemple, pour vous connecter avec Facebook, utilisez le code suivant.

    client.login("facebook").done(function (results) {
         alert("You are now logged in as: " + results.userId);
    }, function (err) {
         alert("Error: " + err);
    });

Si vous utilisez un fournisseur d'identité différent de Facebook, remplacez la valeur transmise à la méthode `login` ci-dessus par l'une des valeurs suivantes : `microsoftaccount`, `facebook`, `twitter`, `google` ou `windowsazureactivedirectory`.

Dans ce cas, Mobile Services gère le flux d'authentification OAuth 2.0 en affichant la page de connexion du fournisseur sélectionné et en générant un jeton d'authentification Mobile Services après avoir établi une connexion avec le fournisseur d'identité. La fonction [login], lorsqu'elle est utilisée, renvoie un objet JSON (**user**) qui expose l'ID utilisateur et le jeton d'authentification Mobile Services dans les champs **userId** et **authenticationToken**, respectivement. Ce jeton peut être mis en cache et réutilisé jusqu'à ce qu'il arrive à expiration. Pour plus d’informations, consultez [Mise en cache du jeton d’authentification].

### Flux client
Votre application peut également contacter le fournisseur d'identité de manière indépendante, puis fournir le jeton renvoyé à Mobile Services à des fins d'authentification. Le flux client permet de proposer l'authentification unique aux utilisateurs ou de récupérer d'autres données utilisateur auprès du fournisseur d'identité.

#### Exemple de SDK basique Facebook/Google
Cet exemple utilise le SDK client Facebook pour l'authentification :

    client.login(
         "facebook",
         {"access_token": token})
    .done(function (results) {
         alert("You are now logged in as: " + results.userId);
    }, function (err) {
         alert("Error: " + err);
    });

Cet exemple part du principe que le jeton fourni par le Kit de développement logiciel (SDK) propre au fournisseur est stocké dans une variable `token`. Vous ne pouvez pas utiliser Twitter pour l’authentification client pour le moment.

#### Exemple de compte Microsoft basique
L'exemple suivant utilise le Kit de développement logiciel (SDK) Live, qui prend en charge l'authentification unique pour les applications Windows Store à l'aide d'un compte Microsoft :

    WL.login({ scope: "wl.basic"}).then(function (result) {
          client.login(
                "microsoftaccount",
                {"authenticationToken": result.session.authentication_token})
          .done(function(results){
                alert("You are now logged in as: " + results.userId);
          },
          function(error){
                alert("Error: " + err);
          });
    });

Cet exemple simplifié récupère un jeton de Live Connect, qui est fourni à Mobile Services en appelant la fonction [login].

#### Exemple de compte Microsoft complet
L'exemple suivant montre comment utiliser le SDK Live avec des API WinJS pour offrir une expérience d’authentification unique améliorée :

    // Set the mobileClient variable to client variable generated by the tooling.
    var mobileClient = <yourClient>;

    var session = null;
    var login = function () {
        return new WinJS.Promise(function (complete) {
            WL.login({ scope: "wl.basic" }).then(function (result) {
                session = result.session;

                WinJS.Promise.join([
                    WL.api({ path: "me", method: "GET" }),
                    mobileClient.login(result.session.authentication_token)
                ]).done(function (results) {
                    // Build the welcome message from the Microsoft account info.
                    var profile = results[0];
                    var title = "Welcome " + profile.first_name + "!";
                    var message = "You are now logged in as: "
                        + mobileClient.currentUser.userId;
                    var dialog = new Windows.UI.Popups.MessageDialog(message, title);
                    dialog.showAsync().then(function () {
                        // Reload items from the mobile service.
                        refreshTodoItems();
                    }).done(complete);

                }, function (error) {

                });
            }, function (error) {
                session = null;
                var dialog = new Windows.UI.Popups.MessageDialog("You must log in.", "Login Required");
                dialog.showAsync().done(complete);
            });
        });
    }

    var authenticate = function () {
        // Block until sign-in is successful.
        login().then(function () {
            if (session === null) {
                // Authentication failed, try again.
                authenticate();
            }
        });
    }

    // Initialize the Live client.
    WL.init({
        redirect_uri: mobileClient.applicationUrl
    });

    // Start the sign-in process.
    authenticate();

Cela permet d'initialiser le client Live Connect, d'envoyer une nouvelle demande de connexion à Live Connect, d'envoyer le jeton d'authentification renvoyé à Mobile Services, puis d'afficher des informations sur l'utilisateur connecté. L'application ne démarre pas tant que l'authentification n’a pas abouti.
<!--- //this guidance may be bad from an XSS vulnerability standpoint. We need to find better guidance for this

### Caching the authentication token
In some cases, the call to the login method can be avoided after the first time the user authenticates. We can use [sessionStorage] or [localStorage] to cache the current user identity the first time they log in and every subsequent time we check whether we already have the user identity in our cache. If the cache is empty or calls fail (meaning the current login session has expired), we still need to go through the login process.

    // After logging in
    sessionStorage.loggedInUser = JSON.stringify(client.currentUser);

    // Log in
    if (sessionStorage.loggedInUser) {
       client.currentUser = JSON.parse(sessionStorage.loggedInUser);
    } else {
       // Regular login flow
   }

     // Log out
    client.logout();
    sessionStorage.loggedInUser = null;
-->

## <a name="push-notifications"></a>Procédure : inscription aux notifications Push
Quand votre application est une application HTML/JavaScript Apache Cordova ou PhoneGap, la plateforme mobile native vous permet de recevoir des notifications Push sur l'appareil. Le [plug-in Apache Cordova pour Azure Mobile Services](https://github.com/Azure/azure-mobile-services-cordova) permet de s'inscrire aux notifications Push avec Azure Notification Hubs. Le service de notification utilisé dépend de la plateforme d'appareil native sur laquelle le code s'exécute. Pour obtenir un exemple de procédure, consultez [Utiliser Microsoft Azure pour activer les notifications Push vers les applications Cordova](https://github.com/Azure/mobile-services-samples/tree/master/CordovaNotificationsArticle).

> [!NOTE]
> Actuellement, ce plug-in ne prend en charge que les appareils iOS et Android. Pour une solution qui inclut également les appareils Windows, consultez l'article [Push Notifications to PhoneGap Apps using Notification Hubs Integration](http://blogs.msdn.com/b/azuremobile/archive/2014/06/17/push-notifications-to-phonegap-apps-using-notification-hubs-integration.aspx).
> 
> 

## <a name="errors"></a>Procédure : gestion des erreurs
Il existe plusieurs méthodes pour détecter, valider et résoudre les erreurs dans Mobile Services.

Par exemple, il est possible d'utiliser les scripts serveur inscrits dans un service mobile pour effectuer diverses opérations sur les données insérées et mises à jour, qu'il s'agisse de les valider ou de les modifier. Vous pouvez définir et inscrire un script serveur qui valide et modifie des données comme suit :

    function insert(item, user, request) {
       if (item.text.length > 10) {
          request.respond(statusCodes.BAD_REQUEST, { error: "Text cannot exceed 10 characters" });
       } else {
          request.execute();
       }
    }

Ce script côté serveur valide la longueur des données de chaîne envoyées au service mobile et refuse les chaînes trop longues, en l'occurrence, celles qui font plus de 10 caractères.

Comme le service mobile valide les données et envoie des réponses d'erreur côté serveur, vous pouvez mettre à jour votre application HTML de manière à ce qu'elle traite les réponses d'erreur de la validation.

    todoItemTable.insert({
       text: itemText,
       complete: false
    })
       .then(function (results) {
       alert(JSON.stringify(results));
    }, function (error) {
       alert(JSON.parse(error.request.responseText).error);
    });


En outre, vous transmettez le gestionnaire d'erreur comme deuxième argument à chaque accès aux données :

    function handleError(message) {
       if (window.console && window.console.error) {
          window.console.error(message);
       }
    }

    client.getTable("tablename").read()
        .then(function (data) { /* do something */ }, handleError);

## <a name="promises"></a>Procédure : utilisation des promesses
Les promesses permettent de planifier un travail sur une valeur qui n'a pas encore été calculée. Il s'agit d'une abstraction pour la gestion des interactions avec des API asynchrones.

La promesse `done` est exécutée dès que la fonction qui lui a été fournie a réussi ou a généré une erreur. Contrairement à la promesse `then`, la génération d'une erreur non traitée au sein de la fonction est garantie, et à la fin de l'exécution des gestionnaires, cette fonction génère une erreur qui aurait été renvoyée depuis « then » en tant que promesse dans l'état d'erreur. Pour plus d'informations, consultez la page [done].

    promise.done(onComplete, onError);

Comme suit :

    var query = todoItemTable;
    query.read().done(function (results) {
       alert(JSON.stringify(results));
    }, function (err) {
       alert("Error: " + err);
    });

La promesse `then` est identique à la promesse `done`, mais contrairement à la promesse `then`, `done` garantit la génération d'une erreur qui n'est pas traitée au sein de la fonction. Si vous ne fournissez pas de gestionnaire d'erreurs à `then` et que l'opération génère une erreur, aucune exception n'est levée, mais une promesse est renvoyée dans l'état d'erreur. Pour plus d’informations, consultez la page [then].

    promise.then(onComplete, onError).done( /* Your success and error handlers */ );

Comme suit :

    var query = todoItemTable;
    query.read().done(function (results) {
       alert(JSON.stringify(results));
    }, function (err) {
       alert("Error: " + err);
    });

Vous pouvez utiliser les promesses de différentes manières. Vous pouvez chaîner des opérations de promesse en appelant `then` ou `done` sur la promesse renvoyée par la fonction `then` précédente. Utilisez `then` pour une étape intermédiaire de l'opération (par exemple `.then().then()`), et `done` pour l'étape finale de l'opération (par exemple `.then().then().done()`). Vous pouvez chaîner plusieurs fonctions `then`, car `then` renvoie une promesse. Vous ne pouvez pas chaîner plusieurs méthodes `done`, car elles renvoient des éléments non définis. [Découvrez les différences entre then et done].

    todoItemTable.insert({
       text: "foo"
    }).then(function (inserted) {
       inserted.newField = 123;
       return todoItemTable.update(inserted);
    }).done(function (insertedAndUpdated) {
       alert(JSON.stringify(insertedAndUpdated));
    })

## <a name="customizing"></a>Procédure : personnalisation des en-têtes de requête
Vous pouvez envoyer des en-têtes de requête personnalisés à l'aide de la fonction `withFilter`, en lisant et en écrivant les propriétés arbitraires de la requête sur le point d'être envoyée au sein du filtre. Vous pouvez ajouter un en-tête HTTP personnalisé si le script côté serveur en a besoin ou peut être amélioré grâce à lui.

    var client = new WindowsAzure.MobileServiceClient('https://your-app-url', 'your-key')
       .withFilter(function (request, next, callback) {
       request.headers.MyCustomHttpHeader = "Some value";
       next(request, callback);
    });

Outre la personnalisation des en-têtes de requête, les filtres peuvent être utilisés dans bien d'autres opérations, par exemple pour examiner ou modifier les requêtes/réponses, ignorer les appels réseau, envoyer plusieurs appels, etc.

## <a name="hostnames"></a>Procédure : utilisation de CORS (Cross-Origin Resource Sharing, partage des ressources cross-origin)
Pour contrôler les sites web autorisés à interagir avec les requêtes et à en envoyer à votre service mobile, veillez à ajouter le nom d'hôte du site web utilisé pour l'héberger dans la liste approuvée CORS (Cross-Origin Resource Sharing). Pour un service mobile de backend JavaScript, vous pouvez configurer la liste approuvée sous l'onglet Configurer du [portail Azure Classic](https://manage.windowsazure.com). Vous pouvez utiliser des caractères génériques le cas échéant. Par défaut, les nouveaux services mobiles indiquent aux navigateurs d'autoriser l'accès uniquement à partir de `localhost`, et CORS (Cross-Origin Resource Sharing) permet au code JavaScript de s'exécuter dans un navigateur sur un nom d'hôte externe à des fins d'interaction avec votre service mobile. Cette configuration n'est pas requise pour les applications WinJS.

<!-- Anchors. -->
[What is Mobile Services]: #what-is
[Concepts]: #concepts
[How to: Create the Mobile Services client]: #create-client
[How to: Query data from a mobile service]: #querying
[Filter returned data]: #filtering
[Sort returned data]: #sorting
[Return data in pages]: #paging
[Select specific columns]: #selecting
[Look up data by ID]: #lookingup
[How to: Display data in the user interface]: #binding
[How to: Insert data into a mobile service]: #inserting
[How to: Modify data in a mobile service]: #modifying
[How to: Delete data in a mobile service]: #deleting
[How to: Authenticate users]: #authentication
[How to: Handle errors]: #errors
[How to: Use promises]: #promises
[How to: Customize request headers]: #customizing
[How to: Use cross-origin resource sharing]: #hostnames
[Next steps]: #nextsteps
[Execute an OData query operation]: #odata-query



<!-- URLs. -->
[then]: http://msdn.microsoft.com/library/windows/apps/br229728.aspx
[done]: http://msdn.microsoft.com/library/windows/apps/hh701079.aspx
[Découvrez les différences entre then et done]: http://msdn.microsoft.com/library/windows/apps/hh700334.aspx
[how to handle errors in promises]: http://msdn.microsoft.com/library/windows/apps/hh700337.aspx

[sessionStorage]: http://msdn.microsoft.com/library/cc197062(v=vs.85).aspx
[localStorage]: http://msdn.microsoft.com/library/cc197062(v=vs.85).aspx

[ListView]: http://msdn.microsoft.com/library/windows/apps/br211837.aspx
[Liaison de données (applications du Windows Store en JavaScript et HTML)]: http://msdn.microsoft.com/library/windows/apps/hh758311.aspx
[login]: https://github.com/Azure/azure-mobile-services/blob/master/sdk/Javascript/src/MobileServiceClient.js#L301
[ASCII control codes C0 and C1]: http://en.wikipedia.org/wiki/Data_link_escape_character#C1_set
[Référence des options de requête système OData]: http://go.microsoft.com/fwlink/p/?LinkId=444502

<!---HONumber=AcomDC_0727_2016-->