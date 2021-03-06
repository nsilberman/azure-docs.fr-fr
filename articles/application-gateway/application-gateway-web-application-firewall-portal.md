---
title: "Création d’une passerelle Application Gateway avec le pare-feu d’applications web à l’aide du portail | Microsoft Docs"
description: "Apprenez à créer une passerelle Application Gateway avec le pare-feu d’applications web à l’aide du portail"
services: application-gateway
documentationcenter: na
author: georgewallace
manager: carmonm
editor: tysonn
tags: azure-resource-manager
ms.assetid: b561a210-ed99-4ab4-be06-b49215e3255a
ms.service: application-gateway
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 11/16/2016
ms.author: gwallace
translationtype: Human Translation
ms.sourcegitcommit: ee8cfffdbf054b4251ed269745f6b9ee5a5e6c64
ms.openlocfilehash: 216b3890cde6f41c33aa34f23d7e103322d9b502


---
# <a name="create-an-application-gateway-with-web-application-firewall-by-using-the-portal"></a>Création d’une passerelle d’application avec le pare-feu d’applications web à l’aide du portail

> [!div class="op_single_selector"]
> * [Portail Azure](application-gateway-web-application-firewall-portal.md)
> * [Commandes PowerShell pour Azure Resource Manager](application-gateway-web-application-firewall-powershell.md)
> 
> 

Le pare-feu d’applications web (WAF, Web Application Firewall) d’Azure Application Gateway protège les applications web des attaques basées sur le web courantes comme l’injection de code SQL, les attaques de script de site à site et les piratages de session. L’application web protège contre la plupart des 10 plus courantes vulnérabilités web de l’OWASP.

La passerelle Azure Application Gateway est un équilibreur de charge de couche 7. Elle assure l’exécution des requêtes HTTP de basculement et de routage des performances entre serveurs locaux ou dans le cloud.
L’application offre de nombreuses fonctionnalités Application Delivery Controller (ADC), notamment : équilibrage de charge HTTP, affinité de session basée sur les cookies, déchargement SSL (Secure Sockets Layer), sondes d’intégrité personnalisées, prise en charge de plusieurs sites, etc.
Pour obtenir une liste complète des fonctionnalités prises en charge, consultez [Vue d’ensemble d’Application Gateway](application-gateway-introduction.md)

## <a name="scenarios"></a>Scénarios

Dans cet article, il existe deux scénarios :

Dans le premier scénario, vous allez apprendre à [ajouter des pare-feu d’applications web à une passerelle d’application existante](#add-web-application-firewall-to-an-existing-application-gateway).

Dans le second scénario, vous allez apprendre à [créer une passerelle d’application avec le pare-feu d’applications web](#create-an-application-gateway-with-web-application-firewall)

![Exemple de scénario][scenario]

> [!NOTE]
> La configuration supplémentaire de la passerelle Application Gateway, y compris les sondes d’intégrité personnalisées, les adresses de pool principal et les règles supplémentaires sont configurées après avoir configuré la passerelle Application Gateway et non lors du déploiement initial.
> 
> 

## <a name="before-you-begin"></a>Avant de commencer

La passerelle Application Gateway Azure requiert son propre sous-réseau. Lorsque vous créez un réseau virtuel, assurez-vous que vous laissez suffisamment d’espace d’adresse pour disposer de plusieurs sous-réseaux. Une fois que vous avez déployé une passerelle Application Gateway sur un sous-réseau, seules les passerelles Application Gateway supplémentaires peuvent être ajoutées au sous-réseau.

## <a name="add-web-application-firewall-to-an-existing-application-gateway"></a>ajouter des pare-feu d’applications web à une passerelle d’application existante

Ce scénario met à jour une passerelle d’application existante afin de prendre en charge des pare-feu d’applications web en mode de prévention.

### <a name="step-1"></a>Étape 1 :

Accédez au portail Azure et sélectionnez une passerelle Application Gateway existante.

![Création d’une passerelle Application Gateway][1]

### <a name="step-2"></a>Étape 2 :

Cliquez sur **Configuration** et mettez à jour les paramètres de la passerelle d’application. Lorsque vous avez terminé, cliquez sur **Enregistrer**

Les paramètres permettant de mettre à jour une passerelle d’application existante afin de prendre en charge des pare-feu d’applications web sont les suivants :

* **Niveau** - Le niveau sélectionné doit être **WAF** pour prendre en charge les pare-feu d’applications web
* **Taille de la référence (SKU)** - Taille de la passerelle Application Gateway avec un pare-feu d’applications web. Les options disponibles sont **Moyen** et **Grand**.
* **État du pare-feu** - Ce paramètre désactive ou active la pare-feu d’applications web.
* **Mode pare-feu** - Ce paramètre spécifique comment le pare-feu d’applications web traite le trafic malveillant. Le mode de **Détection** consigne uniquement les événements, tandis que le mode de **Prévention** consigne les événements et bloque le trafic malveillant.

![panneau montrant les paramètres de base][2]

> [!NOTE]
> Pour afficher les journaux de pare-feu d’applications web, les diagnostics doivent être activés et ApplicationGatewayFirewallLog doit être sélectionné. Vous pouvez choisir un nombre d’instances de 1 à des fins de test. Il est important de savoir que n’importe quel nombre d’instances inférieur à 2 n’est pas couvert par le contrat SLA et n’est donc pas recommandé. Les petites passerelles ne sont pas disponibles lorsque vous utilisez des pare-feu d’applications web.
> 
> 

## <a name="create-an-application-gateway-with-web-application-firewall"></a>créer une passerelle d’application avec le pare-feu d’applications web

Ce scénario va :

* créer une passerelle Application Gateway moyenne avec deux instances ;
* créer un réseau virtuel nommé AdatumAppGatewayVNET avec un bloc CIDR réservé de 10.0.0.0/16 ;
* créer un sous-réseau appelé Appgatewaysubnet qui utilise 10.0.0.0/28 comme bloc CIDR ;
* configurer un certificat pour le déchargement SSL.

### <a name="step-1"></a>Étape 1

Accédez au portail Azure, cliquez sur **Nouveau** > **Mise en réseau** > **Application Gateway**

![Création d’une passerelle Application Gateway][1-1]

### <a name="step-2"></a>Étape 2 :

Remplissez ensuite les informations de base relatives à la passerelle Application Gateway. Veillez à choisir le niveau **WAF** . Lorsque vous avez terminé, cliquez sur **OK**

Les informations nécessaires pour les paramètres de base sont les suivantes :

* **Nom** -Nom de la passerelle Application Gateway.
* **Niveau** - Le niveau de la passerelle Application Gateway. Les options disponibles sont **Standard** et **WAF**. Le pare-feu d’applications web est uniquement disponible au niveau WAF.
* **Taille de la référence (SKU)** - Taille de la passerelle Application Gateway. Les options disponibles sont **Moyen** et **Grand**.
* **Nombre d’instances** - Nombre d’instances. Cette valeur doit être un nombre compris entre **2** et **10**.
* **Groupe de ressources** - Groupe de ressources destiné à contenir la passerelle Application Gateway. Il peut s’agir d’un groupe de ressources existant ou nouveau.
* **Emplacement** - Région de la passerelle Application Gateway. Il s’agit du même emplacement que celui du groupe de ressources. *Cette notion est importante, dans la mesure où le réseau virtuel et l’adresse IP publique doivent se trouver au même endroit que la passerelle*.

![panneau montrant les paramètres de base][2-2]

> [!NOTE]
> Vous pouvez choisir un nombre d’instances de 1 à des fins de test. Il est important de savoir que n’importe quel nombre d’instances inférieur à 2 n’est pas couvert par le contrat SLA et n’est donc pas recommandé. Les petites passerelles ne sont pas prises en charge dans les scénarios de pare-feu d’applications web.
> 
> 

### <a name="step-3"></a>Étape 3

Une fois les paramètres de base définis, l’étape suivante consiste à définir le réseau virtuel à utiliser. Le réseau virtuel héberge l’application pour laquelle la passerelle Application Gateway effectue l’équilibrage de charge.

Cliquez sur **Choisir un réseau virtuel** pour configurer le réseau virtuel.

![panneau montrant les paramètres de la passerelle Application Gateway][3]

### <a name="step-4"></a>Étape 4

Dans le panneau **Choisir un réseau virtuel** , cliquez sur **Créer nouveau**

Bien que cela ne soit pas expliqué dans ce scénario, un réseau virtuel peut être sélectionné à ce stade.  Si un réseau virtuel existant est utilisé, il est important de savoir que le réseau virtuel a besoin d’un sous-réseau vide ou d’un sous-réseau contenant uniquement des ressources Application Gateway pour pouvoir être utilisé.

![panneau Choisir un réseau virtuel][4]

### <a name="step-5"></a>Étape 5

Remplissez les informations de réseau dans le panneau **Créer un réseau virtuel** comme décrit dans la description du précédent [scénario](#scenario) .

![panneau Créer un réseau virtuel avec informations saisies][5]

### <a name="step-6"></a>Étape 6

Une fois le réseau virtuel créé, l’étape suivante consiste à définir l’adresse IP frontale de la passerelle Application Gateway. À ce stade, le choix se résume à une adresse IP privée ou publique pour le serveur frontal. Le choix dépend si l’application est accessible sur Internet ou en interne uniquement. Ce scénario suppose l’utilisation d’une adresse IP publique. Pour choisir une adresse IP privée, il est possible de cliquer sur le bouton **Privé** . Une adresse IP affectée automatiquement est choisie, ou vous pouvez cocher la case **Choisir une adresse IP privée spécifique** pour en saisir une manuellement.

Cliquez sur **Choisir une adresse IP publique**. Si une adresse IP publique existante est disponible, elle peut être choisie à ce stade Dans ce scénario, vous créez une nouvelle adresse IP publique. Cliquez sur **Create new**

![Panneau Choisir une adresse IP publique][6]

### <a name="step-7"></a>Étape 7

Attribuez ensuite un nom convivial à l’adresse IP publique et cliquez sur **OK**

![Panneau Créer une adresse IP publique][7]

### <a name="step-8"></a>Étape 8

Ensuite, vous configurez l’écouteur.  Si **http** est utilisé, il n’y a rien d’autre à configurer. Vous pouvez alors cliquer sur **OK**. Une configuration supplémentaire est requise pour utiliser **https**.

Un certificat est requis pour utiliser **https**. La clé privée du certificat est nécessaire de sorte qu’un export .pfx du certificat soit requis, ainsi que le mot de passer du fichier.

Cliquez sur **HTTPS**, puis sur l’icône de **dossier** situé à côté de la zone de texte **Charger un certificat PFX**.
Accédez au fichier de certificat .pfx sur votre système de fichiers. Une fois sélectionné, donnez un nom convivial au certificat et tapez le mot de passe du fichier .pfx.

Une fois terminé, cliquez sur **OK** pour passer en revue les paramètres de la passerelle Application Gateway.

![section Configuration de l’écouteur du panneau Paramètres][8]

### <a name="step-9"></a>Étape 9

Configurez les paramètres **WAF** spécifiques.

* **État du pare-feu** - Ce paramètre active ou désactive WAF.
* **Mode pare-feu** - Ce paramètre détermine les actions effectuées par WAF concernant le trafic malveillant. Si le mode de **détection** est choisi, le trafic est uniquement consigné.  Si le mode de **prévention** est choisi, le trafic est consigné et bloqué avec une erreur d’autorisation 403.

![Paramètres du pare-feu d’applications web][9]

### <a name="step-10"></a>Étape 10

Consultez la page Résumé et cliquez sur **OK**.  La passerelle Application Gateway est maintenant mise en file d’attente et créée.

### <a name="step-11"></a>Étape 11

Une fois la passerelle Application Gateway créée, accédez à celle-ci dans le portail pour poursuivre sa configuration.

![Vue des ressources de la passerelle Application Gateway][10]

Ces étapes permettent de créer une passerelle Application Gateway de base avec les paramètres par défaut pour l’écouteur, le pool principal, les paramètres http principaux et les règles. Vous pouvez modifier ces paramètres en fonction de votre déploiement une fois l’approvisionnement réussi

## <a name="next-steps"></a>Étapes suivantes

Apprenez à configurer la journalisation des diagnostics, à consigner les événements détectés ou bloqués par le pare-feu d’applications web en consultant la rubrique [Diagnostics de la passerelle Application Gateway](application-gateway-diagnostics.md)

Apprenez à créer des sondes d’intégrité personnalisées en vous rendant sur [Créer une sonde d’intégrité personnalisée](application-gateway-create-probe-portal.md)

Découvrez comment configurer le déchargement SSL et éviter à vos serveurs web le déchiffrement SSL coûteux en vous rendant sur [Configurer le déchargement SSL](application-gateway-ssl-portal.md)

<!--Image references-->
[1]: ./media/application-gateway-web-application-firewall-portal/figure1.png
[2]: ./media/application-gateway-web-application-firewall-portal/figure2.png
[1-1]: ./media/application-gateway-web-application-firewall-portal/figure1-1.png
[2-2]: ./media/application-gateway-web-application-firewall-portal/figure2-2.png
[3]: ./media/application-gateway-web-application-firewall-portal/figure3.png
[4]: ./media/application-gateway-web-application-firewall-portal/figure4.png
[5]: ./media/application-gateway-web-application-firewall-portal/figure5.png
[6]: ./media/application-gateway-web-application-firewall-portal/figure6.png
[7]: ./media/application-gateway-web-application-firewall-portal/figure7.png
[8]: ./media/application-gateway-web-application-firewall-portal/figure8.png
[9]: ./media/application-gateway-web-application-firewall-portal/figure9.png
[10]: ./media/application-gateway-web-application-firewall-portal/figure10.png
[scenario]: ./media/application-gateway-web-application-firewall-portal/scenario.png



<!--HONumber=Nov16_HO3-->


