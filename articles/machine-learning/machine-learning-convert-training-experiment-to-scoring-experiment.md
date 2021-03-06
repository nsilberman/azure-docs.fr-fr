---
title: Conversion d’une expérience d’apprentissage Machine Learning en expérience de prédiction | Microsoft Docs
description: Découvrez comment convertir une expérience d’apprentissage Machine Learning, utilisée pour l’apprentissage du modèle d’analyse prédictive, en expérience de notation pouvant être publiée en tant que service web.
services: machine-learning
documentationcenter: ''
author: garyericson
manager: jhubbard
editor: cgronlun

ms.service: machine-learning
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/19/2016
ms.author: garye

---
# Conversion d’une expérience d’apprentissage Machine Learning en expérience prédictive
Microsoft Azure Machine Learning vous permet de générer, tester et déployer des solutions d’analyse prédictive.

Une fois que vous avez créé une *expérience de formation*, effectué l’itération sur celle-ci pour former le modèle d’analyse prédictive et que vous êtes prêt à l’utiliser pour noter les nouvelles données, vous devez préparer et rationaliser votre expérience à des fins de notation. Vous pouvez ensuite déployer cette *expérience prédictive* en tant que service web Microsoft Azure, afin que les utilisateurs puissent envoyer des données à votre modèle et recevoir les prédictions de ce dernier.

En la convertissant en expérience prédictive, vous préparez votre modèle formé à être déployé en tant que service web. Les utilisateurs du service web envoient des données d’entrée à votre modèle, qui leur renvoie les résultats de sa prédiction. Par conséquent, lorsque vous convertissez l’expérience en expérience prédictive, vous devez tenir compte du mode d’utilisation de votre modèle par les autres utilisateurs.

[!INCLUDE [machine-learning-free-trial](../../includes/machine-learning-free-trial.md)]

Le processus de conversion d’une expérience d’apprentissage en expérience prédictive comprend trois étapes :

1. Enregistrez le modèle d’apprentissage automatique que vous avez formé, puis remplacez l’algorithme d’apprentissage automatique et les modules [Train Model][train-model] par le modèle formé enregistré.
2. Découpez l’expérience pour ne garder que les modules nécessaires au calcul des notations. Une expérience d’apprentissage inclut un certain nombre de modules nécessaires pour l’apprentissage, mais qui ne sont plus requis une fois le modèle formé et prêt à servir lors du calcul des notations.
3. Définissez à quel niveau de votre expérience vous voulez accepter les données de l’utilisateur du service web et choisissez le type des données qui seront renvoyées.

## Définir le bouton de service web
Après avoir mené votre expérience (bouton **EXÉCUTER** au bas de la zone de dessin d’expérimentation), le bouton **Configurer le Service web** (sélectionnez l’option **Service web prédictif**) effectue pour vous les trois étapes de conversion de votre expérience de formation en prévision d’une expérience pour vous :

1. Il enregistre votre modèle en tant que module dans la section **Modèles formés** de la palette du module (située à gauche de la zone de dessin de l’expérimentation), puis remplace l’algorithme d’apprentissage automatique et les modules [Train Model][train-model] par le module formé enregistré.
2. Il supprime les modules qui ne sont pas nécessaires. Dans notre exemple, cela inclut le module [Split Data][split], le deuxième module [Score Model][score-model] et le module [Evaluate Model][evaluate-model].
3. Il crée les modèles d’entrée et de sortie du service web et les ajoute aux emplacements par défaut prévus dans votre expérience.

Par exemple, l’expérience suivante effectue l’apprentissage d’un modèle d’arborescence de décision augmenté incluant deux classes, au moyen des données de recensement :

![Expérience d'apprentissage][figure1]

Les modules de cette expérience effectuent quatre fonctions de base :

![Fonctions du module][figure2]

Lorsque vous convertissez cette expérience d’apprentissage en expérience prédictive, certains de ces modules ne sont plus nécessaires ou présentent un objectif différent :

* **Data** : les données de cet exemple de jeu de données ne sont pas utilisées pour le calcul de la notation ; l’utilisateur du service web fournit les données à noter. Toutefois, les métadonnées de ce jeu de données, comme les types de données, sont utilisées par le modèle formé. Vous devez donc conserver le jeu de données dans l’expérience prédictive, afin qu’il puisse fournir ces métadonnées.
* **Prep** : selon les données qui seront soumises pour le calcul de la notation, il se peut que ces modules soient requis ou non pour le traitement des données entrantes.
  
    Par exemple, l’exemple de jeu de données indiqué ici peut présenter des valeurs manquantes ; il inclut des colonnes qui ne sont pas nécessaires pour former le modèle. Par conséquent, un module [Clean Missing Data][clean-missing-data] a été inclus pour gérer les valeurs manquantes et un module [Select Columns in Dataset][select-columns] a été ajouté pour exclure ces colonnes supplémentaires du flux de données. Si vous savez qu’aucune donnée ne manque parmi les données qui seront soumises à des fins de calcul de la notation via le service web, vous pouvez retirer le module [Clean Missing Data][clean-missing-data]. Toutefois, étant donné qu’il permet de définir l’ensemble de fonctionnalités qui sont notées, le module [Select Columns in Dataset][select-columns] doit être conservé.
* **Train** : une fois le modèle formé, vous pouvez l'enregistrer en tant que module unique de modèle formé. Vous devez ensuite remplacer ces modules individuels par le modèle formé enregistré.
* **Score** : dans cet exemple, le module Split est utilisé pour diviser le flux de données en un ensemble de données de test, d’une part, et un ensemble de données d’apprentissage, d’autre part. Dans l’expérience prédictive, ce module n’est pas nécessaire et peut être supprimé. De même, le deuxième module [Score Model][score-model] et le module [Evaluate Model][evaluate-model] sont utilisés pour comparer les résultats à partir des données de test. Ils ne sont donc pas nécessaires à l’expérience prédictive. Le module [Score Model][score-model] restant est cependant requis pour renvoyer le résultat de la notation par le biais du service web.

Voici comment notre exemple apparaît une fois que vous avez cliqué sur **Définir un service Web** :

![Expérience prédictive convertie][figure3]

Cela peut être suffisant pour préparer votre expérience à être déployée en tant que service web. Toutefois, certaines tâches supplémentaires, spécifiques à votre expérience, seront peut-être à prévoir.

### Ajuster des modules d’entrée et de sortie
Dans votre expérience d’apprentissage, vous avez utilisé un ensemble de données d’apprentissage, puis effectué un certain traitement pour obtenir les données au format requis par l’algorithme ML. Si les données que vous vous attendez à recevoir via le service web n’ont pas besoin d’être soumises à ce traitement, vous pouvez déplacer le module **Web service input** vers un autre nœud dans votre expérience.

Par exemple, le bouton **Définir un service Web** place par défaut le module **Web service input** en haut de votre flux de données, comme indiqué dans la figure ci-dessus. Toutefois, si les données d’entrée n’ont pas besoin d’être soumises à ce traitement, vous pouvez placer manuellement le module **Web service input** après les modules de traitement des données :

![Déplacement de l’entrée du service web][figure4]

Les données d’entrée fournies via le service web accéderont directement au module Score Model, sans être soumises à un traitement préalable.

De même, par défaut, **Définir un service Web** place le module Web service output en bas de votre flux de données. Dans cet exemple, le service web renvoie à l’utilisateur la sortie du module [Score Model][score-model], qui inclut le vecteur de données d’entrée complet, ainsi que les résultats de la notation.

Toutefois, si vous souhaitez renvoyer d’autres données (par exemple, les résultats de la notation sans le vecteur de données d’entrée complet), vous pouvez insérer un module [Select Columns in Dataset][select-columns] pour exclure toutes les colonnes, sauf celle des résultats de la notation. Ensuite, déplacez le module **Web service output** vers la sortie du module [Select Columns in Dataset][select-columns] :

![Déplacement de la sortie du service web][figure5]

### Ajouter ou supprimer des modules de traitement des données supplémentaires
Si votre expérience inclut un plus grand nombre de modules que nécessaire pour le calcul de la notation, vous pouvez en supprimer. Par exemple, étant donné que nous avons déplacé le module **Web service input** vers un point situé après les modules de traitement des données, nous pouvons supprimer le module [Clean Missing Data][clean-missing-data] de l’expérience prédictive.

Notre expérience de notation ressemble à présent à ce qui suit :

![Suppression du module supplémentaire][figure6]

### Ajouter des paramètres de service web facultatifs
Dans certains cas, vous voudrez permettre à l’utilisateur de votre service web de modifier le comportement des modules en cas d’accès au service. Vous pouvez le faire grâce aux *paramètres de service Web*.

Un exemple courant consiste à configurer le module [Import Data][import-data], afin que l’utilisateur du service web publié puisse spécifier une autre source de données lors de l’accès au service web. Il est également possible de configurer le module [Exporter les données][export-data] de façon à spécifier une destination différente.

Vous pouvez définir des paramètres de service web et les associer à un ou plusieurs paramètres de module, en spécifiant s’ils sont obligatoires ou facultatifs. L’utilisateur du service web peut ensuite fournir des valeurs pour ces paramètres lors de l’accès au service ; les actions de module seront modifiées en conséquence.

Pour en savoir plus sur les paramètres de service web, consultez la page [Utilisation des paramètres de service web de Microsoft Azure Machine Learning][webserviceparameters].

[webserviceparameters]: machine-learning-web-service-parameters.md


## Déploiement de l’expérience prédictive sous la forme d’un service web
Maintenant que l’expérience prédictive est correctement préparée, vous pouvez la publier en tant que service web Azure. En accédant au service web, les utilisateurs peuvent envoyer des données à votre modèle, qui renvoie alors ses prédictions.

Pour en savoir plus sur le processus de déploiement complet, consultez la page [Déploiement d’un service web Machine Learning Azure][deploy]

[deploy]: machine-learning-publish-a-machine-learning-web-service.md


<!-- Images -->
[figure1]: ./media/machine-learning-convert-training-experiment-to-scoring-experiment/figure1.png
[figure2]: ./media/machine-learning-convert-training-experiment-to-scoring-experiment/figure2.png
[figure3]: ./media/machine-learning-convert-training-experiment-to-scoring-experiment/figure3.png
[figure4]: ./media/machine-learning-convert-training-experiment-to-scoring-experiment/figure4.png
[figure5]: ./media/machine-learning-convert-training-experiment-to-scoring-experiment/figure5.png
[figure6]: ./media/machine-learning-convert-training-experiment-to-scoring-experiment/figure6.png


<!-- Module References -->
[clean-missing-data]: https://msdn.microsoft.com/library/azure/d2c5ca2f-7323-41a3-9b7e-da917c99f0c4/
[evaluate-model]: https://msdn.microsoft.com/library/azure/927d65ac-3b50-4694-9903-20f6c1672089/
[select-columns]: https://msdn.microsoft.com/library/azure/1ec722fa-b623-4e26-a44e-a50c6d726223/
[import-data]: https://msdn.microsoft.com/library/azure/4e1b0fe6-aded-4b3f-a36f-39b8862b9004/
[score-model]: https://msdn.microsoft.com/library/azure/401b4f92-e724-4d5a-be81-d5b0ff9bdb33/
[split]: https://msdn.microsoft.com/library/azure/70530644-c97a-4ab6-85f7-88bf30a8be5f/
[train-model]: https://msdn.microsoft.com/library/azure/5cc7053e-aa30-450d-96c0-dae4be720977/
[export-data]: https://msdn.microsoft.com/library/azure/7a391181-b6a7-4ad4-b82d-e419c0d6522c/

<!---HONumber=AcomDC_0914_2016-->