---
title: "Comment acheter un nom de domaine personnalisé dans Azure App Service Web Apps"
description: "Découvrez comment acheter un nom de domaine personnalisé avec une application web dans Azure App Service."
services: app-service\web
documentationcenter: 
author: rmcmurray
manager: erikre
editor: 
ms.assetid: 70fb0e6e-8727-4cca-ba82-98a4d21586ff
ms.service: app-service-web
ms.workload: web
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 11/01/2016
ms.author: robmcm
translationtype: Human Translation
ms.sourcegitcommit: 2ea002938d69ad34aff421fa0eb753e449724a8f
ms.openlocfilehash: 9c5c0b40d796130d93111545c93bedf86c374fd9


---
# <a name="buy-and-configure-a-custom-domain-name-in-azure-app-service"></a>Acheter et configurer un nom de domaine personnalisé dans Azure App Service
[!INCLUDE [web-selector](../../includes/websites-custom-domain-selector.md)]

Lorsque vous créez une application web, Azure l'attribue à un sous-domaine de azurewebsites.net. Par exemple, si votre application web se nomme **contoso**, l’URL est **contoso.azurewebsites.net**. Azure attribue également une adresse IP virtuelle.

Pour une application web de production, vous souhaitez probablement afficher un nom de domaine personnalisé pour vos utilisateurs. Cet article explique comment acheter et configurer un domaine personnalisé avec [App Service Web Apps](http://go.microsoft.com/fwlink/?LinkId=529714). 

[!INCLUDE [introfooter](../../includes/custom-dns-web-site-intro-notes.md)]

## <a name="overview"></a>Vue d'ensemble
Si vous n’avez pas de nom de domaine pour votre application web, vous pouvez facilement en acheter un sur le [portail Azure](https://portal.azure.com/). Pendant le processus d'achat, vous pouvez choisir de mapper automatiquement les enregistrements DNS du domaine racine et WWW à votre application web. Vous pouvez également gérer votre domaine directement dans le portail Azure.

Procédez comme suit pour acheter des noms de domaine et les attribuer à votre application web.

1. Dans votre navigateur, ouvrez le [portail Azure](https://portal.azure.com/).
2. Dans l’onglet **Web Apps**, cliquez sur le nom de votre application web, puis sélectionnez **Paramètres** et **Domaines personnalisés**.
   
    ![](./media/custom-dns-web-site-buydomains-web-app/dncmntask-cname-6.png)
3. Dans le panneau **Domaines personnalisés**, cliquez sur **Acheter des domaines**.
   
    ![](./media/custom-dns-web-site-buydomains-web-app/dncmntask-cname-buydomains-1.png)
4. Dans le panneau **Acheter des domaines**, utilisez la zone de texte pour taper le nom de domaine que vous voulez acheter et appuyez sur Entrée. Les domaines disponibles proposés s'affichent simplement en-dessous de la zone de texte. Sélectionnez le domaine que vous souhaitez acheter. Vous pouvez choisir d'acheter plusieurs domaines à la fois. 
   
   ![](./media/custom-dns-web-site-buydomains-web-app/dncmntask-cname-buydomains-2.png)
5. Cliquez sur **Informations de contact** et remplissez le formulaire d’informations de contact du domaine.
   
   ![](./media/custom-dns-web-site-buydomains-web-app/dncmntask-cname-buydomains-3.png)
   
   > [!NOTE]
   > Il est très important de remplir tous les champs obligatoires aussi précisément que possible, notamment l'adresse de messagerie. Si vous achetez un domaine sans « Protection de confidentialité », vous devrez peut-être vérifier votre adresse de messagerie avant l'activation du domaine. Dans certains cas, l'inexactitude des informations de contact fournies peuvent empêcher l'achat des domaines. 
   > 
   > 
6. À présent, vous pouvez choisir l'une des options suivantes :
   
    a) « Renouveler automatiquement » votre domaine chaque année
   
    b) « Protection de confidentialité », incluse GRATUITEMENT dans le prix d’achat (à l’exception des domaines de premier niveau dont le registre ne prend en charge la confidentialité, par exemple .co.in,. co.uk, etc.)  
   
    c) « Attribuer des noms d'hôte par défaut » pour le domaine racine et WWW dans l'application web actuelle. 
   
   ![](./media/custom-dns-web-site-buydomains-web-app/dncmntask-cname-buydomains-2.5.png)
   
   > [!NOTE]
   > L'option C configure automatiquement les liaisons DNS et de nom d'hôte.  De cette façon, votre application web est accessible à l'aide du domaine personnalisé dès que l'achat est terminé (sous réserve des délais de propagation DNS dans certains cas). Si votre application web se trouve derrière Azure Traffic Manager, l'option permettant d'attribuer un domaine racine n'est pas affichée, car les enregistrements A ne fonctionnent pas avec Traffic Manager. Vous pouvez toujours attribuer les domaines/sous-domaines que vous avez achetés pour une application web à une autre, et vice versa. Consultez l'étape 8 pour plus d'informations. 
   > 
   > 
7. Cliquez sur **Sélectionner** dans le panneau **Acheter des domaines**. Les informations d’achat s’affichent dans le panneau **Confirmation d’achat**. Si vous acceptez les conditions juridiques et cliquez sur **Acheter**, votre commande est envoyée et vous pouvez surveiller le processus d’achat dans **Notification**. L'achat de domaine peut prendre plusieurs minutes. 
   
   ![](./media/custom-dns-web-site-buydomains-web-app/dncmntask-cname-buydomains-4.png)
   
   ![](./media/custom-dns-web-site-buydomains-web-app/dncmntask-cname-buydomains-5.png)
8. Si vous avez correctement commandé un domaine, vous pouvez gérer le domaine et l'attribuer à votre application web. Cliquez sur **…**, à droite de votre domaine. Vous pouvez **Annuler l’achat** ou **Gérer le domaine**. Cliquez sur **Gérer le domaine**. Vous pouvez lier un **sous-domaine** à votre application web dans le panneau **Gérer le domaine**. Si vous souhaitez lier un **sous-domaine** à une autre application web, effectuez cette étape dans le contexte de l’application web correspondante. À partir de là, vous pouvez choisir d'attribuer le domaine au point de terminaison Traffic Manager (si l'application web se trouve derrière TM) en sélectionnant simplement le nom Traffic Manager dans le menu déroulant. De cette manière, le domaine/sous-domaine est automatiquement attribué à toutes les applications web situées derrière ce point de terminaison Traffic Manager. 
   
    ![](./media/custom-dns-web-site-buydomains-web-app/dncmntask-cname-buydomains-6.png)
   
   > [!NOTE]
   > Vous pouvez « Annuler l'achat » dans les 5 jours pour obtenir un remboursement intégral. Au bout de 5 jours, vous ne pouvez plus « Annuler l'achat », mais vous pouvez « Supprimer » le domaine. La suppression du domaine le retire de votre abonnement sans remboursement, et le domaine devient disponible. 
   > 
   > 

Une fois la configuration terminée, le nom de domaine personnalisé est répertorié dans la section **Liaisons de nom d’hôte** de votre application web.

À ce stade, vous devez pouvoir entrer le nom de domaine personnalisé dans votre navigateur et vérifier que vous êtes bien dirigé vers votre application web.

## <a name="what-happens-to-the-custom-domain-you-bought"></a>Que se passe-t-il pour le domaine personnalisé que vous avez acheté ?
Le domaine personnalisé que vous avez acheté dans le panneau **Domaines personnalisés et SSL** est lié à l’abonnement Azure. En tant que ressource Azure, ce domaine personnalisé est distinct et indépendant de l’application App Service pour laquelle vous avez initialement acheté le domaine. Cela signifie que :

* Dans le portail Azure, vous pouvez utiliser le domaine personnalisé que vous avez acheté pour plusieurs applications App Service et pas uniquement pour l’application pour laquelle vous avez initialement acheté le domaine personnalisé. 
* Vous pouvez gérer tous les domaines personnalisés que vous avez achetés dans l’abonnement Azure en accédant au panneau **Domaines personnalisés et SSL** de *n’importe quelle* application App Service dans cet abonnement.
* Vous pouvez attribuer n’importe quelle application App Service du même abonnement Azure à un sous-domaine inclus dans ce domaine personnalisé.
* Si vous décidez de supprimer une application App Service, vous pouvez choisir de ne pas supprimer le domaine personnalisé auquel elle est liée si vous souhaitez conserver celui-ci pour l’utiliser avec d’autres applications.

## <a name="if-you-cant-see-the-custom-domain-you-bought"></a>Si le domaine personnalisé que vous avez acheté ne s’affiche pas
Si vous avez acheté le domaine personnalisé à partir du panneau **Domaines personnalisés et SSL**, mais s’il ne s’affiche pas sous **Domaines gérés**, vérifiez les éléments suivants :

* La création du domaine personnalisé n’est peut-être pas terminée. Vérifiez la cloche de notification en haut du portail Azure pour connaître sa progression.
* La création du domaine personnalisé peut avoir échoué pour une raison quelconque. Vérifiez la cloche de notification en haut du portail Azure pour connaître sa progression.
* La création du domaine personnalisé est peut-être réussie, mais le panneau n’a peut-être pas été actualisé. Essayez de rouvrir le panneau **Domaines personnalisés et SSL**.
* Vous avez peut-être supprimé le domaine personnalisé à un moment donné. Vérifiez les journaux d’audit en cliquant sur **Paramètres** > **Journaux d’audit** dans le panneau principal de votre application. 
* Le panneau **Domaines personnalisés et SSL** que vous consultez appartient peut-être à une application créée dans un autre abonnement Azure. Basculez vers une autre application dans un autre abonnement et vérifiez le panneau **Domaines personnalisés et SSL**.  
    Dans le portail, vous ne pourrez pas voir ni gérer les domaines personnalisés créés dans un abonnement Azure différent de celui lié à l’application. Toutefois, si vous cliquez sur **Gestion avancée** dans le panneau **Gérer le domaine** du domaine, vous êtes redirigé vers le site web du fournisseur de domaine, où vous pouvez   [configurer manuellement votre domaine personnalisé comme n’importe quel domaine personnalisé externe](web-sites-custom-domain-name.md) 
   pour les applications créées dans un autre abonnement Azure. 




<!--HONumber=Nov16_HO3-->


