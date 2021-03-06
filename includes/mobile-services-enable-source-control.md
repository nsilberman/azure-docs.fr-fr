
1. Connectez-vous au [portail Azure Classic](https://manage.windowsazure.com/), cliquez sur **Mobiles Services**, cliquez sur votre service mobile, puis sur l’onglet **Tableau de bord**.
2. (Facultatif) Si vous avez déjà défini les informations d’identification du contrôle de code source Mobile Services ou Web Sites pour votre abonnement Azure, vous pouvez passer directement à l’étape 4. Sinon, cliquez sur **Définir le contrôle de code source** sous **Aperçu rapide** et cliquez sur **Oui** pour confirmer.
   
    ![Définition du contrôle de code source](./media/mobile-services-enable-source-control/mobile-setup-source-control.png)
3. Fournissez un **Nom d'utilisateur**, un **Nouveau mot de passe**, confirmez le mot de passe, puis cliquez sur le bouton de vérification.
   
    Le référentiel Git est créé dans votre service mobile. Notez les informations d’identification que vous venez de fournir, car elles vont seront utiles pour accéder à ce référentiel et à d’autres référentiels Mobile Services de votre abonnement.
4. Cliquez sur l'onglet **Configurer** et examinez les champs **Contrôle de code source**.
   
    ![Configuration du contrôle de code source](./media/mobile-services-enable-source-control/mobile-source-control-configure.png)
   
    L'URL du référentiel Git s'affiche. Vous l'utiliserez pour cloner le référentiel sur votre ordinateur local.

Avec le contrôle de code source activé dans votre service mobile, vous pouvez utiliser Git pour cloner le référentiel sur votre ordinateur local.

<!---HONumber=AcomDC_1203_2015-->