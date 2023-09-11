# _talend-ESB-runtime-configuration-SSL_

## Objectifs
Mettre en place la configuration SSL dans Talend ESB Runtime :
- Afin de sécuriser les services en les exposant avec un certificat SSL.
- Pour garantir la sécurité lors de la consommation de services protégés par un certificat SSL.

## Outils et Versions
- Talend Open Studio for ESB Version: 8.0.1
- Java Version 11

## SSL/TLS 
> SSL a été remplacé par TLS (Transport Layer Security), une version plus récente et plus sécurisée du protocole, mais le terme "SSL" est souvent utilisé de manière interchangeable pour se référer à cette technologie de sécurité. SSL/TLS est largement utilisé pour sécuriser les connexions HTTPS (HTTP sécurisé) sur le web, ainsi que pour sécuriser d'autres protocoles de communication tels que SMTP (email), IMAP (email), et plus encore.

## Activer le cryptage SSL dans Talend Runtime
### Lancer l’ESB 
dans le répertoire que vous venez d'extracter le Talend Open Studio, aller vers _<Runtime_ESBSE>/container/bin_ et exécuter **trun.bash** (sur windows) et **./trun** si vous êtes sur Linux ou mac. La fenêtre suivante devrait s’afficher
![Lancer l’ESB.](/image/runtime-ESBSE.PNG "Lancer l’ESB")

Lancer les deux commandes suivantes dans l'invite commandes
```
feature:install http
feature:install webconsole
```
![Installation Features.](/image/runtime-cmd-1.PNG "Installation Features")

Redemarrer le runtime

Ouvre les liens suivantes dans le navigateur 
- http://localhost:8040/services
- https://localhost:9001/services 

![Installation Features.](/image/runtime-cmd-2.PNG "Installation Features")

` Pour le Talend administration : https://help.talend.com/r/fr-FR/8.0/installation-guide-windows/enabling-ssl-encryption-in-talend-runtime `

## Configurer le SSL/TLS sur runtime Talend
> Authentification et autorisation à l’aide de certificats pour les services externes
Il est très courant de créer des services de données qui consomment des données provenant d’autres services dans Talend. Dans l’exemple suivant, vous allez convertir le certificat SSL du service que vous souhaitez appeler vers Java KeyStore (JKS).
Vous pouvez obtenir le certificat SSL de deux façons:
- Il peut être fourni par le fournisseur WebService
- Vous pouvez télécharger le certificat SSL en appelant le service web dans un navigateur (en essayant de lire le WSDL dans un navigateur, par exemple).

1 - Télécharger le certificat SSL 
![Télécharger le certificat SSL](/image/certificat-ssl-1.PNG "Télécharger le certificat SSL")

### Convertir des certificats SSL en Java KeyStore
2 - Créer le fichier JKS avec la commande keytool
```
keytool -import -alias <nom de l'alias> -file <fichier certificat .cer> -keystore <fichier certificat .jks> -storepass changeit -keypass changeit -noprompt
```
Vérification du certificat sur jks:
```
keytool -list -v -keystore <fichier certificat .jks> -alias <nom de l'alias>
```
![Télécharger le certificat SSL](/image/certificat-ssl-2.PNG "Télécharger le certificat SSL")

### Déploiement de votre service sécurisé dans Talend Runtime
3 - Ajouter le fichier JKS généré dans le répertoire _<Runtime_ESBSE>/container/etc/keystores_
4 - Créer un fichier org.apache.cxf.http.conduits-<nom_service>.cfg  dans le répertoire _<Runtime_ESBSE>/container/etc_ suivant le model du fichier org.apache.cxf.http.conduits-common.cfg
```
url = <URL>
tlsClientParameters.disableCNCheck = truetlsClientParameters.trustManagers.keyStore.type = JKS
tlsClientParameters.trustManagers.keyStore.password = <Mot de passe du fichier JKS>
tlsClientParameters.trustManagers.keyStore.file = ./etc/keystores/<Nom du fichier JKS>.jks
tlsClientParameters.keyManagers.keyStore.type = JKS
tlsClientParameters.keyManagers.keyStore.password = <Mot de passe du fichier JKS>
tlsClientParameters.keyManagers.keyStore.file = ./etc/keystores/<Nom du fichier JKS>.jks
```
5 - Redémarrez Talend Runtime pour appliquer les modifications  