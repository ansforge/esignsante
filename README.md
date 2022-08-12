# eSignSante
## INTRODUCTION
### DESCRIPTION GENERALE

L’outil eSignSante est un outil exposant une API REST de signature et de vérification de signature, développé et maintenu par l’Agence du Numérique en Santé (ANS) pour ses propres besoins.


L’outil eSignSante, implémente les fonctionnalités suivantes :
> Signature de document, avec ou sans génération de preuve, dans les formats suivants :
* signature enveloppée de fichier XML en XMLDSIG-Core 1 ;
* signature enveloppée de fichier XML en XADES-Baseline B ;
* signature enveloppante en XMLDSIG-Core 1 ;
* signature enveloppante en XADES-Baseline B.
* signature PAdES-Baseline B 
> Validation de signature, avec ou sans génération de preuve, dans les formats suivants:
* signature enveloppée de fichier XML en XMLDSIG-Core 1 ;
* signature enveloppée de fichier XML en XADES-Baseline B ;
* signature enveloppante en XMLDSIG-Core 1 ;
* signature enveloppante en XADES-Baseline B.
* signature PAdES-Baseline B

Les preuves générées sont signées en XADES-Baseline B.

## DESCRIPTION DES PRINCIPALES FONCTIONNALITES
### SIGNATURE

Il est possible de signer des documents au formats XMLDsig, XADES et PADES BASELINE B.
Les signatures peuvent être :
* Enveloppante : ce type de signature est utilisable pour tous les types de document. Le contenu du document signé est transformé en base 64, et inséré à l’intérieur de la signature. La signature « enveloppe » le contenu signé.
* Enveloppée : ce type de signature est utilisable uniquement pour les documents XMLs. La signature est insérée dans le contenu signé. Le contenu « enveloppe » la signature. Le contenu signé est donc directement visible (contrairement à la signature enveloppante ou le contenu signé est transformé en base 64).
Les signatures sont renvoyées en base 64.
L’outil offre également la possibilité de protéger l’appel des opérations de signatures via le passage d’un « secret ». 

### VERIFICATION DE SIGNATURE
L’outil peut vérifier des signatures XMLDSig ou XADES et PADES BASELINE B. La vérification s’appuie sur les règles et procédures définies par l’ETSI (https://www.etsi.org/deliver/etsi_ts/102800_102899/102853/01.01.02_60/ts_102853v010102p.pdf).
Seules les règles applicables aux formats XMLDSIG et XADES et PADES BASELINE B sont prises en compte. Il est possible, si on le souhaite, de désactiver des règles par paramétrage.
Les autorités de confiances, utilisées pour vérifier l’origine du certificat de signature, sont déclarées dans le fichier de configuration.
L’outil eSignSante permet également de générer une preuve de vérification de signature. Cette preuve contient l’ensemble du contexte de vérification de la signature (certificat, CRL, configuration de l’outil de signature) et elle est signée en XADES BASELINE B enveloppée.
La vérification de signature renvoi :
* Le résultat de la vérification : valide ou invalide
* La liste des erreurs si la signature est invalide
* Un rapport (le rapport de la librairie DSS), avec le résultat de la vérification de chacune des règles

### IHM
L’outil eSignSante permet d’activer une IHM générée, à partir de la description OpenAPI, par le framework Swagger.
Cette IHM permet de faciliter l’utilisation de l’API exposée.  

### RECHARGEMENT DES CRLs
Pour contrôler la non-révocation des certificats, l’outil recharge les CRLs des autorités de confiances configurées, périodiquement. Par défaut ce rechargement se fait une fois par jour à l’heure d’installation. Mais la périodicité peut être paramétrée.


## SOCLE TECHNIQUE DE L’OUTIL
### Socle pour la signature et la vérification de signature

L’outil a été développé en s’appuyant sur la librairie java DSS (Digital Signature Service), mise en œuvre par la commission européenne (https://ec.europa.eu/cefdigital/DSS/webapp-demo/doc/dss-documentation.html).
Cette librairie permet de générer des signatures au format XAdES, qui est un format de signature définie par l’ETSI (ETSI EN 319 132 partie 1 et 2) et au format PADES (ETSI EN 319 142 partie 1 et 2).  XAdES et PAdEs définissent des profils de signatures en conformité avec la réglementation européenne pour les transactions électroniques. 
Les 4 profils (ou niveau) suivants sont définis :
* XAdES/PAdES Baseline B : ce niveau permet d’être conforme aux règles de signatures définies par la réglementation européenne en terme de format.
* XAdES/PAdES-BASELINE-T : ce niveau ajoute au niveau précédent, un horodatage permettant la non remise en cause de la date de signature du document.
* XAdES/PAdES-BASELINE-LT : ce niveau ajoute au niveau précédent, l’ensemble des données nécessaire à la vérification de signature (certificats, liste de révocations) afin de garantir la possibilité de vérifier la signature dans le futur. 
* XAdES/PAdES-BASELINE-LTA : ce niveau ajoute au précédent un horodatage périodique à des fins d’archivage.

La librairie DSS permet également de vérifier des signatures via un processus conforme aux règles émises par l’ETSI (cf https://www.etsi.org/deliver/etsi_ts/102800_102899/102853/01.01.02_60/ts_102853v010102p.pdf). Lors de la vérification de signature, la librairie DSS génère un rapport contenant le résultat de chacune des règles définies par l’ETSI.

L’utilisation de DSS, permet de déléguer la bonne mise en œuvre des formats et des règles de validation de signature, à une librairie d’une autorité reconnue (la commission européenne).

Pour le moment dans eSignSanté, et compte tenu des besoins de l’ANS, seul le niveau XAdES Baseline B a été mis en œuvre. Les autres niveaux peuvent être rajoutés en cas de besoins

## MODE DE DISTRIBUTION ET DE DEPLOIEMENT
La version actuelle de eSignSante est distribuée sous forme d'une image Docker ou d'un fichier exécutable via une JVM et de 2 fichiers de configuration :
* application.properties : fichier de propriétés permettant de paramétrer l’outil de signature (taille maximale autorisée pour les requêtes, chemin des fichiers de logs, activation du secret pour l’appel des opérations de signatures)
* config.json : fichier de configuration au format JSON, permettant de définir les signatures (clé privée, algorithmes etc.), les vérifications de signatures (les contrôles à effectuer), les preuves (clé privé de signature, algorithme etc.)
Le déploiement est simplifié, il suffit de copier ces 3 fichiers dans un répertoire et de lancer la commande java pour démarrer esignsante. Aucun autre composant n’est utilisé (pas de serveur d’application).

## APPEL DE L’API
L’API est décrite via le fichier OpenAPI suivant :esignsante-ws-api-doc.yaml

La documentation de l’API peut être visualisée en copiant le fichier OpenAPI dans l’éditeur en ligne swagger : https://editor.swagger.io/

L’API a été conçue pour être facile d’utilisation. Par exemple pour signer un fichier, seuls 2 paramètres sont obligatoires :
* Le fichier à signer
* L’identifiant de la configuration à utiliser. Cette configuration est déclarée dans le fichier de config.json et définie :
  ** La clé privée à utiliser pour la signature
  ** Le type de signature : enveloppée ou enveloppante
  ** Les algorithmes à utiliser
