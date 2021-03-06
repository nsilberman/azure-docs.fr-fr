---
title: Principaux concepts du service API Management
description: Learn about APIs, products, roles, groups, and other API Management key concepts.
services: api-management
documentationcenter: 
author: steved0x
manager: erikre
editor: 
ms.assetid: e71da405-835a-48f3-956f-45c1a85698d7
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: hero-article
ms.date: 10/25/2016
ms.author: sdanie
translationtype: Human Translation
ms.sourcegitcommit: 2ea002938d69ad34aff421fa0eb753e449724a8f
ms.openlocfilehash: dcff834c9d4aff82e8319e6c23a74c2e49902a8c


---
# <a name="what-is-api-management"></a>Présentation de Gestion des API
API Management aide les organisations à publier des API pour des développeurs externes, partenaires et internes, afin de libérer le potentiel de leurs données et services. Toutes les entreprises cherchent à étendre leurs opérations en tant que plateforme numérique, en créant de nouveaux canaux, en trouvant de nouveaux clients et en suscitant un engagement supérieur auprès de ceux existants. Gestion des API offre les compétences principales pour assurer un programme d'API réussi à travers l'engagement du développeur, les perspectives commerciales, l'analyse, la sécurité et la protection.

Regardez la vidéo qui suit pour obtenir une vue d’ensemble de la gestion des API Azure et apprendre à l’utiliser pour ajouter de nombreuses fonctionnalités à votre API, et notamment le contrôle d’accès, la limitation du débit, la surveillance, la journalisation des événements et la mise en mémoire cache de réponse, avec un minimum de travail de votre part.

> [!VIDEO https://channel9.msdn.com/Blogs/Windows-Azure/Azure-API-Management-Overview/player]
> 
> 

Pour utiliser Gestion des API, les administrateurs créent des API. Chaque API se compose d'une ou plusieurs opérations et chacune peut être ajoutée à un produit ou plusieurs. Pour utiliser une API, les développeurs s'abonnent à un produit qui contient cette API, puis ils peuvent appeler l'opération de l'API, sujette à toutes les stratégies d'utilisation pouvant être en vigueur.

Cette rubrique offre une présentation des principaux concepts associés à Gestion des API.

> [!NOTE]
> Pour plus d'informations, consultez le livre blanc [Gestion des API basées sur le cloud : maîtrise de la puissance des API](http://j.mp/ms-apim-whitepaper) , disponible en version PDF. Ce livre blanc d’introduction à la gestion des API, publié par CITO Research, présente les opérations suivantes : 
> 
> * Configuration requise des API et défis communs
> * Découplage des API et présentation des façades
> * Motivation des développeurs et exécution rapide
> * Sécurisation de l'accès
> * Analyse et mesures
> * Prise de contrôle et aperçu d’une plate-forme de gestion des API
> * Utilisation du cloud et solutions locales
> * Gestion des API Azure
> 
> 

## <a name="apis"> </a>API et opérations
Les API sont la base d'une instance du service Gestion des API. Chaque API représente un ensemble d'opérations disponibles pour les développeurs. Chaque API contient une référence au service principal qui implémente l'API, et ses opérations effectuent un mappage vers les opérations implémentées par le service principal. Les opérations dans Gestion des API sont hautement configurables. Elles contrôlent le mappage d'URL, les paramètres de requête et de chemin d'accès, le contenu de la demande et de la réponse, et la mise en cache de la réponse de l'opération. La limite de débit, les quotas et les stratégies de restriction de l'adresse IP peuvent également être implémentés au niveau de l'API ou du fonctionnement individuel.

Pour plus d'informations, consultez les pages [Création d'API][Création d'API] et [Ajout d'opérations à une API][Ajout d'opérations à une API].

## <a name="products"> </a> Produits
Les produits sont la façon dont les API sont présentées en surface aux développeurs. Les produits dans Gestion des API possèdent une ou plusieurs API et sont configurés avec un titre, une description et des conditions d'utilisation. Les produits peuvent être **ouverts** ou **protégés**. Les produits protégés doivent faire l’objet d’un abonnement avant de pouvoir être utilisés, alors que les produits ouverts peuvent être utilisés sans abonnement. Lorsqu'un produit est prêt à l'emploi pour les développeurs, il peut être publié. Une fois le produit publié, les développeurs peuvent l’afficher et s’y abonner (dans le cas des produits protégés). L'approbation d'abonnement est configurée au niveau du produit et peut nécessiter l'approbation de l'administrateur ou être automatiquement approuvée.

Les groupes permettent de gérer la visibilité des produits pour les développeurs. Les produits accordent de la visibilité aux groupes. Les développeurs peuvent afficher les produits visibles pour les groupes auxquels ils appartiennent et s'y abonner. 

Pour plus d'informations, consultez la page [Création et publication d'un produit][Création et publication d'un produit] ainsi que la vidéo suivante.

> [!VIDEO https://channel9.msdn.com/Blogs/AzureApiMgmt/Using-Products/player]
> 
> 

## <a name="groups"> </a> Groupes
Les groupes permettent de gérer la visibilité des produits pour les développeurs. Le service Gestion des API possède les groupes système suivants, qui ne sont pas modifiables.

* **Administrateurs** : les administrateurs d’abonnements Azure sont membres de ce groupe. Les administrateurs gèrent les instances du service Gestion des API, créant les API, opérations et produits qui sont utilisés par les développeurs.
* **Développeurs** : les utilisateurs authentifiés du portail des développeurs appartiennent à ce groupe. Les développeurs sont les clients qui génèrent des applications grâce à vos API. Les développeurs bénéficient d'un accès au portail des développeurs et génèrent des applications qui appellent les opérations d'une API.
* **Invités** : les utilisateurs non authentifiés du portail des développeurs, comme les prospects, qui consultent le portail des développeurs d’une instance d’API Management appartiennent à ce groupe. Ils peuvent recevoir certains accès en lecture seule, comme la possibilité d'afficher les API, mais pas de les appeler.

Outre ces groupes système, les administrateurs peuvent créer des groupes personnalisés ou [utiliser des groupes externes dans des locataires Azure Active Directory qui leur sont associés](api-management-howto-aad.md#how-to-add-an-external-azure-active-directory-group). Des groupes externes et personnalisés peuvent être utilisés avec des groupes système offrant une certaine visibilité aux développeurs et un accès aux produits d’API. Vous pourriez, par exemple, créer un groupe personnalisé pour les développeurs affiliés à une organisation partenaire spécifique et leur permettre d’accéder aux API à partir d’un produit contenant uniquement des API pertinentes. Un utilisateur peut être membre de plusieurs groupes.

Pour plus d'informations, consultez la page [Création et utilisation de groupes][Création et utilisation de groupes].

## <a name="developers"> </a> Développeurs
Les développeurs représentent les comptes d'utilisateur dans une instance du service Gestion des API. Ils peuvent être créés ou invités à rejoindre le groupe par les administrateurs, ou ils peuvent s'inscrire au [Portail des développeurs][Portail des développeurs]. Chaque développeur est membre d'un ou plusieurs groupes, et peut s'abonner aux produits qui accordent de la visibilité à ces groupes.

Lorsque des développeurs s'abonnent à un produit, ils reçoivent les clés principale et secondaire pour le produit. Cette clé est utilisée pour les appels dans les API du produit.

Pour plus d’informations, consultez [Création ou invitation de développeurs][Création ou invitation de développeurs] et [Association de groupes aux développeurs][Association de groupes aux développeurs].

## <a name="policies"> </a> Stratégies
Les stratégies sont une fonctionnalité puissante de Gestion des API. Elles permettent à l'éditeur de modifier le comportement de l'API grâce à la configuration. Les stratégies sont un ensemble d'instructions qui sont exécutées dans l'ordre sur demande ou sur réponse d'une API. Les instructions les plus utilisées comprennent la conversion du format XML au format JSON et la limitation du débit d'appels pour restreindre la quantité d'appels entrants d'un développeur. De nombreuses autres stratégies sont disponibles.

Les expressions de stratégie peuvent être utilisées comme valeurs d’attribut ou valeurs de texte dans l’une des stratégies de Gestion des API, sauf si la stratégie le spécifie autrement. Certaines stratégies, telles que les stratégies [Contrôler le flux](https://msdn.microsoft.com/library/azure/dn894085.aspx#choose) et [Définir la variable](https://msdn.microsoft.com/library/azure/dn894085.aspx#set-variable), sont basées sur des expressions de stratégie. Pour plus d’informations, consultez les rubriques [Stratégies avancées](https://msdn.microsoft.com/library/azure/dn894085.aspx#AdvancedPolicies), [Expressions de stratégie](https://msdn.microsoft.com/library/azure/dn910913.aspx) et visionnez la vidéo suivante.

> [!VIDEO https://channel9.msdn.com/Blogs/AzureApiMgmt/Policy-Expressions-in-Azure-API-Management/player]
> 
> 

Pour obtenir la liste complète des stratégies Gestion des API, consultez la page [Référence de stratégie][Référence de stratégie]. Pour plus d'informations sur l'utilisation et la configuration des stratégies, consultez la page [Stratégies Gestion des API][Stratégies Gestion des API]. Pour consulter un didacticiel sur la création d'un produit avec des stratégies de limite de débit et de quota, consultez la page [Création et configuration de paramètres produit avancés][Création et configuration de paramètres produit avancés]. Pour obtenir une démonstration, consultez la vidéo suivante.

> [!VIDEO https://channel9.msdn.com/Blogs/AzureApiMgmt/Rate-Limits-and-Quotas/player]
> 
> 

## <a name="developer-portal"> </a> Portail des développeurs
Le portail des développeurs est l'endroit où les développeurs peuvent en savoir plus sur votre API, afficher et appeler des opérations et s'abonner à des produits. Les prospects peuvent consulter le portail des développeurs, afficher les API et opérations et se connecter. L’URL du portail des développeurs se trouve sur le tableau de bord du portail Azure Classic pour votre instance de service Gestion des API.

Vous pouvez personnaliser l'aspect de votre portail des développeurs en ajoutant du contenu personnalisé, en personnalisant des styles et en ajoutant votre marque.

## <a name="api-management-and-the-api-economy"></a>Gestion et économie des API
Pour en savoir plus sur la gestion des API, regardez la présentation suivante depuis la conférence Microsoft Ignite 2015.

> [!VIDEO https://channel9.msdn.com/Events/Ignite/2015/BRK3708/player]
> 
> 

[API et opérations]: #apis
[Produits]: #products
[Groupes]: #groups
[Développeurs]: #developers
[Stratégies]: #policies
[Portail des développeurs]: #developer-portal

[Création d'API]: api-management-howto-create-apis.md
[Ajout d'opérations à une API]: api-management-howto-add-operations.md
[Création et publication d'un produit]: api-management-howto-add-products.md
[Création et utilisation de groupes]: api-management-howto-create-groups.md
[Association de groupes aux développeurs]: api-management-howto-create-groups.md#associate-group-developer
[Création et configuration de paramètres produit avancés]: api-management-howto-product-with-rules.md
[Création ou invitation de développeurs]: api-management-howto-create-or-invite-developers.md
[Référence de stratégie]: api-management-policy-reference.md
[Stratégies Gestion des API]: api-management-howto-policies.md
[Création d'une instance du service API Management]: api-management-get-started.md#create-service-instance







<!--HONumber=Nov16_HO2-->


