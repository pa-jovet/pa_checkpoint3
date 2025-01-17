### Partie 1 : Gestion des utilisateurs

**Q.1.1.1** Créer l'utilisateur **Lionel Lemarchand** avec les même attribut de société que **Kelly Rhameur**.
![[Exercice 1/Capture d'écran 2025-01-17 092340.png]]

![[Exercice 1/Capture d'écran 2025-01-17 092356.png]]

![[Exercice 1/Capture d'écran 2025-01-17 092405.png]]

![[Exercice 1/Capture d'écran 2025-01-17 092412.png]]
**Q.1.1.2** Créer une OU **DeactivatedUsers** et déplace le compte désactivé de **Kelly Rhameur** dedans.
![[Exercice 1/Pasted image 20250117092921.png]]
**Q.1.1.3** Modifier le groupe de l'OU dans laquelle était **Kelly Rhameur** en conséquence.
![[Exercice 1/Pasted image 20250117093138.png]]

**Q.1.1.4** Créer le dossier Individuel du nouvel utilisateur et archive celui de **Kelly Rhameur** en le suffixant par **-ARCHIVE**.
![[Exercice 1/Pasted image 20250117093331.png]]
### [](https://odyssey.wildcodeschool.com/quests/2934/pages/12385#partie-2--restriction-utilisateurs)Partie 2 : Restriction utilisateurs

**Q.1.2.1** Faire en sorte que l'utilisateur **Gabriel Ghul** ne puisse se connecter que du lundi au vendredi, de 7h à 17h.
![[Exercice 1/Pasted image 20250117093544.png]]
**Q.1.2.2** De même, bloquer sa connexion au seul ordinateur **CLIENT01**.
![[Exercice 1/Pasted image 20250117093639.png]]
**Q.1.2.3** Mettre en place une stratégie de mot de passe pour durcir les comptes des utilisateurs de l'OU **LabUsers**.
On va créer une GPO pour renforcer la stratégie de mots de passe du domaine : 
- Création de la GPO :
	- Server Manager > Tools > `Group Policy Management` ;
	- Dans votre domaine > `Group Policy Objects` > Créer une nouvelle GPO et la nommer selon votre convention ;
	- Editer la GPO > `Computer Configuration > Policies > Security Settings > Account Policies > Password Policy`
	- Paramètres :
		- Enforce password history : `6 passwords remembered`
		- Maximum password age : `180 days`
		- Minimum password age : `30 days`
		- Relax minimum password length limit :` Enabled`
		- Minimum password length : `17 characters`
		- Password must meet complexity requirements : `Enabled`
	- Sortir de l'éditeur ;
- Lier la GPO :
	- Group Policy Management > Group Policy Objects > Sélectionner la GPO
	- Sélectionner l'OU à laquelle vous souhaitez lier la GPO. Pour nous `LabComputers`, toujours puisqu'il s'agit d'une configuration ordinateur, elle s'applique aux clients et non aux comptes utilisateurs ;
	- Clic droit > `Link to an existing GPO` > choisir la GPO.
	- Scope :
		- Links : `LabComputers`
		- Security Filtering : `Authenticated Users` ;
	- Details : GPO Status - `User Configuration settings disabled`, puisqu'il s'agit d'une configuration ordinateur ;
### [](https://odyssey.wildcodeschool.com/quests/2934/pages/12385#partie-3--lecteurs-r%C3%A9seaux)Partie 3 : Lecteurs réseaux

**Q.1.3.1** Créer une GPO **Drive-Mount** qui monte les lecteurs **E:** et **F:** sur les clients.
- Nous devons commencer par partager ces répertoires pour qu'ils soient visibles par les clients :
	- Explorateur de fichiers > Lecteur E: > Clic droit > Properties > Sharing > Advanced Sharing
	- Activer la case `Share this folder`
	- Permissions : Retirer `Everyone` et ajouter `Authenticated Users` avec les droits en lecture
	- Properties > Security > Advanced : ajouter `Authenticated Users` avec les droits en modification ;
	- Le chemin réseau du répertoire partagé est `\\SRVWIN01\DossiersCommuns`
	- Faire la même chose sur le répertoire `F:` pour obtenir son chemin réseau : `\\SRVWIN01\DossiersIndividuels`
- Création de la GPO :
	- Server Manager > Tools > `Group Policy Management` ;
	- Dans votre domaine > `Group Policy Objects` > Créer une nouvelle GPO ;
	- Editer la GPO > `User Configuration > Preferences > Windows Settings > Drive Maps` > Clic droit > New Mapped Drive
	- Paramètres : 
		- Action : Update
		- Location : `\\SRVWIN01\DossiersCommuns`
		- Reconnect : Yes
		- Label as : DossiersCommuns
		- Drive Letter, use : E
		- Hide/Show this drive : `Show this drive`
	- Créer un second lecteur mappé, sur F avec le chemin réseau `\\SRVWIN01\DossiersIndividuels`
- Lier la GPO :
	- Group Policy Management > Group Policy Objects > Sélectionner la GPO
	- Sélectionner l'OU à laquelle vous souhaitez lier la GPO. Pour nous `LabUsers`, toujours puisqu'il s'agit d'une configuration utilisateur, elle s'applique aux comptes utilisateurs et non aux clients ;
	- Clic droit > `Link to an existing GPO` > choisir la GPO. 
	- Scope :
		- Links : `LabUsers`
		- Security Filtering : `Authenticated Users` ;
	- Details : GPO Status - `Computer Configuration settings disabled`, puisqu'il s'agit d'une configuration utilisateur ;