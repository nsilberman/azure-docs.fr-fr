---
title: Consignes et limitations générales de base de données SQL Azure
description: Cette page décrit certaines limitations générales de la base de données SQL Azure, ainsi que les zones d’interopérabilité et de prise en charge.
services: sql-database
documentationcenter: na
author: CarlRabeler
manager: jhubbard
editor: monicar

ms.service: sql-database
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: data-management
ms.date: 09/06/2016
ms.author: carlrab

---
# Consignes et limitations générales de base de données SQL Azure
Cette rubrique fournit des instructions et présente les limitations générales applicables à la base de données Azure SQL. Pour mieux comprendre les notions de quotas, de gestion des ressources et de support, consultez les [ressources supplémentaires](#additional-guidelines) référencées à la fin de cette rubrique.

## Connectivité et authentification
* L’authentification Windows n’est pas prise en charge. Voir [Gestion des bases de données et des connexions dans la base de données Azure SQL](sql-database-manage-logins.md) Toutefois, l’authentification Azure Active Directory est prise en charge avec certaines limitations. Consultez la page [Connexion à la base de données SQL avec l’authentification Azure Active Directory](sql-database-aad-authentication.md).
* La base de données SQL Microsoft Azure prend en charge les versions 7.3 et ultérieures du client de protocole TDS (Tabular Data Stream).
* Seules les connexions TCP/IP sont autorisées.
* Le navigateur de SQL Server 2008 n’est pas pris en charge, car la base de données SQL Microsoft Azure ne possède pas de ports dynamiques, mais uniquement le port 1433.

## Agents/tâches SQL Server
La base de données SQL Microsoft Azure ne prend pas en charge l’Agent SQL Server. Cependant, vous pouvez utiliser des tâches élastiques pour exécuter des tâches sur une ou plusieurs bases de données. Pour en savoir plus sur les tâches élastiques, consultez la rubrique [Tâches élastiques](sql-database-elastic-jobs-overview.md).

## Prise en charge du classement SQL Server
Le classement de base de données par défaut utilisé par la base de données SQL Microsoft Azure est **SQL\_LATIN1\_GENERAL\_CP1\_CI\_AS**, où **LATIN1\_GENERAL** correspond à l’anglais (États-Unis) et **CP1** à la page de code 1252. La propriété **CI** indique que le classement n’est pas sensible à la casse, et **AS** qu’il respecte les accents. Il est impossible de modifier le classement des bases de données V12. Pour plus d'informations sur la façon de définir le classement, consultez [COLLATE (Transact-SQL)](https://msdn.microsoft.com/library/ms184391.aspx).

## Conventions de dénomination
Certains noms d’utilisateur ne sont pas autorisés pour des raisons de sécurité. Vous ne pouvez pas utiliser les noms suivants :

* **admin**
* **administrator**
* **guest**
* **root**
* **sa**

Les noms de tous les nouveaux objets doivent être conformes aux règles des identificateurs SQL Server. Pour plus d'informations, consultez [Identificateurs](https://msdn.microsoft.com/library/ms175874.aspx).

En outre, les noms de connexion et d’utilisateur ne peuvent pas contenir le caractère \\ (l’authentification Windows n’est pas prise en charge).

## Conseils supplémentaires
* Outre les limitations générales décrites dans cet article, la base de données SQL impose des quotas de ressources et des limitations spécifiques selon votre **niveau de service**. Pour obtenir une présentation des niveaux de service, consultez [Niveaux de service de Base de données SQL](sql-database-service-tiers.md).
* Pour connaître les autres limites de la base de données SQL, consultez [Limites de ressources de la base de données SQL Azure](sql-database-resource-limits.md).
* Pour des informations sur la sécurité, consultez [Instructions et limitations de sécurité dans la base de données SQL Azure](sql-database-security-guidelines.md).
* Un autre domaine connexe entoure la compatibilité de la base de données SQL Azure avec les versions locales de SQL Server, comme SQL Server 2014 et SQL Server 2016. La dernière version (V12) de la base de données SQL Azure a introduit de nombreuses améliorations dans ce domaine. Pour plus d'informations, consultez [Nouveautés de la base de données SQL V12](sql-database-v12-whats-new.md).
* Pour plus d'informations sur la disponibilité des pilotes et sur la prise en charge de la base de données SQL, consultez [Bibliothèques de connexions pour SQL Database et SQL Server](sql-database-libraries.md).

<!---HONumber=AcomDC_0907_2016-->