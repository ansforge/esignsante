# eSignSante
# Installation de ESignSante à partir de l'image Docker

## Prérequis

- Docker
- Fichier de configuration au format json.
- Fichier application.properties (recommandé)

## Fichier de configuration
### Mode de fonctionnement
La configuration de ESignSante et des fonctionnalités présentes après le démarrage de l'application, est définie dans un unique fichier externe au format Json, config.json.
L'objet Json qui définit la configuration est constitué de 5 listes d'objets (la casse est respectée) :
- signature : liste des configurations de signature.
- proof : liste des configurations de preuve de validation de signature.
- signatureVerification : liste des configurations de validation de signature.
- certificateVerification : liste des configurations de validation de certificat.
- ca : liste des certificats au formats pem  qui constituent le bundle des autorités de confiance et le bundle des listes de révocation de certificats.
Voici donc à quoi ressemblerait le squelette du fichier de configuration :
```json
{
"signature": [
		{
			...
		},
		...
	],
"proof": [
		{
			...
		},
		...
	],
	"signatureVerification": [
		{
			...
		},
		...
	],
	"certificateVerification": [
		{
			...
		},
		 ...
	],
	 "ca": [
		{
			...
		},
		...
	]
}
```
Remarque : L'application pourrait être démarrée avec une configuration « vide », par contre le contrôle sur les objets et les champs est strict, et un fichier de configuration malformé empêchera le démarrage de l'application. Une fois l'application démarrée, les modifications apportées au fichier de configuration seront prises en compte si elles sont légitimes, sinon les configurations en mémoire seront retenues.

### Configuration de signature
L'objet configuration de signature contient les éléments suivants :
- idSignConf : ID unique de configuration de signature.
- secret : le ou les secrets pour accéder à la configuration, la valeur du secret ici sera le Hash généré dans “/secrets”, plusieurs secrets doivent être séparés par un espace blanc.
- idProofConf : ID de la configuration de la preuve associée à cette configuration de signature.
- description: description de la configuration
- certificate : certificat de signature au format pem (base 64), sur une ligne pour respecter le format Json.
- privateKey : clé privée au format PKCS#8 (commence avec -----BEGIN PRIVATE KEY----- et fini avec -----END PRIVATE KEY-----) qui correspond au certificat.
- canonicalisationAlgorithm : algorithme.
- digestAlgorithm : digest algorithme.
- signaturePackaging : type de signature, ENVELOPING ou ENVELOPED.
- signId : Id de la balise <ds:Signature>
- signValueId : Id de la balise <ds:SignatureValue>
- objectId : Id de la balise <ds:Object> et URI de la balise <ds:Reference> précédée d'un caractère #.
Les propriétés signId, signValueId et objectId ne sont prises en comptes que par les opérations de signatures XmlDsig. En effet pour les signatures xades ces champs sont générés.
Voici un exemple d'un objet de configuration de signature :
```json
"signature": [
	{
		"idSignConf": "1",
		"secret": "$2a$12$0BhlieeeeDqJ4Z.lQO41/vfclnB4H3p3YtNquCRQ./1cfO $2a$12$wKbipqCY5PRjxfC0fgTXJ.hZhVXTlb84Zbm5LU0ygrdxPNJbo86M2",
		"idProofConf": "1",
		"description": "Scheduling Test.",	
		"certificate": "-----BEGIN CERTIFICATE-----MIIIiDC … VtpQ==-----END CERTIFICATE-----",
		"privateKey": "-----BEGIN PRIVATE KEY-----MIIEv … zcipw==-----END PRIVATE KEY-----",
		"canonicalisationAlgorithm": "http://www.w3.org/2001/10/xml-exc-c14n#",
		"digestAlgorithm": "SHA512",
		"signaturePackaging": "ENVELOPING",
		"signId":"Sig_Id_SIG",
		"signValueId":"smth",
		"objectId":"Id_SignedDocument"
	}
]
```
### Configuration de preuve de validation
L'objet configuration de preuve contient les éléments suivants :
- idProofConf: ID unique de configuration de preuve de validation de signature.
- description
- certificate : certificat de signature de preuve au format pem, sur une ligne pour respecter le format Json. 
- privateKey : clé privée au format PKCS#8 (commence avec -----BEGIN PRIVATE KEY----- et fini avec -----END PRIVATE KEY-----) qui correspond au certificat.
- canonicalisationAlgorithm : canonicalisation algorithm.
- digestAlgorithm : digest algorithm.
- signaturePackaging : type de signature, ENVELOPING ou ENVELOPED.
Voici un exemple :
```json
"proof": [
	{
		"idProofConf": "1",
		"description": "Scheduling Test.",	
		"certificate": "-----BEGIN CERTIFICATE-----\nMIIIiDC … VtpQ==\n-----END CERTIFICATE-----",
		"privateKey": "-----BEGIN PRIVATE KEY-----\nMIIEv … zcipw==\n-----END PRIVATE KEY-----",
		"canonicalisationAlgorithm": "http://www.w3.org/2001/10/xml-exc-c14n#",
		"digestAlgorithm": "SHA512",
		"signaturePackaging": "ENVELOPING",
	}
]
```
### Configuration de validation de signature
L'objet configuration de validation de signature contient les éléments suivants :
- idVerifSign: ID unique de configuration de validation de signature.
- description
- metadata : liste de métadonnées à inclure dans le rapport de validation ESignSante
- rules : règles ESignSante de validation de signature.
Voici un exemple :
```json
"signatureVerification": [
	{
		"idVerifSign": "1",
		"description": "",
		"metadata": "DATE_SIGNATURE,DN_CERTIFICAT,RAPPORT_DIAGNOSTIQUE,DOCUMENT_ORIGINAL_NON_SIGNE,RAPPORT_DSS",
		"rules":	"TrustedCertificat,FormatSignature,SignatureCertificatValide,ExistenceBaliseSigningTime,ExistenceDuCertificatDeSignature,ExpirationCertificat,RevocationCertificat,SignatureNonVide,SignatureIntacte,DocumentIntact"
	}
]
```
Valeurs possibles des règles et métadonnées :
```properties
# Liste des règles de validation, d'une signature ; valeurs possibles :
SignatureCertificatValide
ExistenceBaliseSigningTime
ExistenceDuCertificatDeSignature
ExpirationCertificat
FormatSignature
RevocationCertificat 
SignatureNonVide
TrustedCertificat
SignatureIntacte
DocumentIntact
```
```properties
# Types de metadata retournées lors d'une validation de signature ; valeurs possibles :
DATE_SIGNATURE
DN_CERTIFICAT
RAPPORT_DIAGNOSTIQUE
DOCUMENT_ORIGINAL_NON_SIGNE
RAPPORT_DSS
```
### Configuration de validation de certificat
L'objet configuration de validation de certificat contient les éléments suivants :
- idVerifCert: ID unique de configuration de validation de certificat.
- description
- metadata : liste de métadonnées à inclure dans le rapport de validation ESignSante
- rules : règles ESignSante de validation de certificat.
Voici un exemple :
```json
{
	"idVerifSign": "1",
	"description": "",
	"metadata": "DN_CERTIFICAT,RAPPORT_DIAGNOSTIQUE,RAPPORT_DSS",
	"rules": "ExpirationCertificat,RevocationCertificat,SignatureCertificatValide,TrustedCertificat"
}
```

Valeurs possibles des règles et métadonnées :
```properties
# Liste des règles de validation, rules, d'une signature ; valeurs possibles :
SignatureCertificatValide
ExpirationCertificat
RevocationCertificat 
SignatureNonVide
TrustedCertificat
```
```properties
# Types de metadata retournées lors d'une validation de signature ; valeurs possibles :
DN_CERTIFICAT
RAPPORT_DIAGNOSTIQUE
RAPPORT_DSS
```
### Configuration des autorités de confiance et la liste de révocation
Un objet « ca » contient les éléments suivants :
- certificate: certificat de confiance au format pem, sur une ligne pour respecter le format Json. Ce certificat fera partie du bundle de CAs de confiance.
- crl : url (http ou ldap) de téléchargement de la CRL.
Voici un exemple :
```json
"ca": [
	{
	  "certificate": "-----BEGIN CERTIFICATE-----\n MIIHb ... 3cyxN\n -----END CERTIFICATE-----",
	  "crl": "ldap://annuaire-igc.esante.gouv.fr/cn=TEST%20AC%20IGC-SANTE%20ELEMENTAIRE%20ORGANISATIONS,ou=TEST%20AC%20RACINE%20IGC-SANTE%20ELEMENTAIRE,ou=IGC-SANTE%20TEST,ou=0002%20187512751,o=ASIP-SANTE,c=FR?certificaterevocationlist;binary?base?objectClass=pkiCA"
	},
]
```
Note : pour trouver l'url des CRLs associées à un certificat il suffit de consulter les [points de distribution](https://www.digicert.com/kb/util/utility-test-ocsp-and-crl-access-from-a-server.htm) de la CRL en ouvrant le certificat.

## Commandes utiles de manipulation des fichiers de certificats et de clés privées

### Commandes OpenSSL

Pour extraire la clé privée d’un fichier p12, sans mot de passe et au format PKCS#8, la commande openssl est la suivante :
```
openssl pkcs12 -in <votre fichier p12> -nocerts -nodes -out <fichier de sortie>
```

Pour passer un fichier d'une clée privée, d'un format PKCS#1 vers un format PKCS#8 la commande openssl est la suivante :
```
openssl pkey -in <fichier de clé pem en PKCS#1> -out <fichier de clé de sortie pem en PKCS#8>
```


## Installation de l'Image Docker

### Installation 
A partir du TAR
```shell script
docker load --input [path-to]image.tar
```
Via docker pull
docker pull docker.pkg.github.com/ansforge/esignsante/esignsante:2.4.0.3-release

`docker images ls` pour vérifier que l'image est bien chargée.

Pour faciliter la tâche de démarrage du conteneur, il est recommandé d'avoir le fichier `config.json` et `application.properties` dans un répertoire accessible sur notre hôte. 
Le chemin vers ce répertoire sera `[path-to-conf-files-parent-dir]`.

Exemple du fichier `application.properties`:
```properties
spring.servlet.multipart.max-file-size=200MB
spring.servlet.multipart.max-request-size=2MB
config.secret=enable
server.servlet.context-path=/esignsante/v1
// config.crl.scheduling=[chron job pour rechargement des crl] (optionnel)
server.tomcat.accesslog.enabled=true
server.tomcat.accesslog.directory=/var/esignsante/logs
config.crl.scheduling=0 0 3 * * ?
```

Pour démarrer le conteneur il suffit de lancer :
```shell script
docker run -v [path-to-conf-files-parent-dir]:/var/esignsante -p 8080:8080 -e JAVA_TOOL_OPTIONS="-Xms1g -Xmx1g -Dspring.config.location=/var/esignsante/application.properties -Dspring.profiles.active=swagger -Dhttp.proxyHost=[http-proxy-ip] -Dhttps.proxyHost=[https-proxy-ip]  -Dhttp.proxyPort=[http-proxy-port] -Dhttps.proxyPort=[https-proxy-port]" -d [image-id] --ws.conf=var/esignsante/conf.json
```
Options docker:
* -v : Se compose de deux champs dans notre cas, séparés par des deux points (:).
    - le premier champ est le chemin absolu d'accès au répertoire sur la machine hôte.
    - Le deuxième champ est le chemin absolue où le répertoire est monté dans le conteneur. Ici on a choisi `/var/esignsante`.
* -p : Pour lier un port du conteneur à un port de son choix sur la machine hôte. Ici c'est tout simplement `8080:8080` (hôte:conteneur)
* -d : Detached, le conteneur tourne en tâche de fond. Ne pas renseigner pour dérouler les logs de démarrage.

Options jvm :
* `-Dspring.config.location` : chemin vers le fichier application.properties *sur le conteneur* (optionnel - ici /var/esignsante/application.properties)
* `-Dspring.profiles.active` : égale à `swagger` pour activer l'interface IHM de swagger (désactivé par défaut)
* Proxy: `-Dhttp.proxyHost=`... `-Dhttps.proxyHost=`...  `-Dhttp.proxyPort=`... `-Dhttps.proxyPort=`... (optionnel)

Arguments java :
* `--ws.conf` : chemin vers le fichier de configuration *sur le conteneur* (obligatoire - ici /var/esignsante/conf.json)

L'application est maintenant accessible sur http://localhost:8080/esignsante/v1/swagger-ui.html#/

Pour afficher les conteneurs :
```shell script
docker ps -all
```
Pour afficher les logs :
```shell script
docker container logs [container-name]
```
Pour arrêter le conteneur :
```shell script
docker stop [container-name]
```
