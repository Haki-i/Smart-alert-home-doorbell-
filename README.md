# Smart-alert-home-doorbell (notification + lumière)-

# **Objectif du projet**

L’objectif est d’informer un malentendant qu’une personne appuie sur sa sonnerie. Nous cherchons à recevoir le signal radio de 433MHz d’une sonnette de maison avec le récepteur MX-RM-5V et notre Nodemcu. Celle-ci fait alors clignoter des lumières Hue en utilisant des requêtes HTTP et nous envoie une notification sur l’application IFTTT.

# I- Conception électronique

## a) Réception du signal radio

La sonnette extérieure émet un signal à une fréquence radio de 433MHz qui est réceptionné par un boîtier intérieur qui diffuse un son.

Ce signal inclut un code unique qui sert d'identifiant pour la sonnette. Ce code permet au récepteur de distinguer différents émetteurs sur la même fréquence.

Pour réceptionner à notre tour ce signal, nous utiliserons le module MX-RM-5V. Lorsque celui-ci  captera un signal issu du bouton de la sonnette, il sera en mesure d’extraire le code inclus dans ce signal. Si le code correspond à la sonnette souhaitée, notre Nodemcu réalisera alors une action spécifique.

Il est essentiel que notre sonnette utilise un système simple sans protocole de communication utilisant du cryptage et d'autres techniques de sécurité. Sur un système simple, les signaux peuvent être facilement interceptés par notre module.

## b) Branchements et antenne

Nous branchons notre récepteur au 5V, Ground et à un pin digital de notre Nodemcu.

Le module étant plutôt précaire, la distance de réception est assez faible. Nous pouvons lui souder une antenne avec du fil de cuivre d’une taille correspondant à un quart de la longueur d'onde.

Nous utilisons la formule : lambda = C/f

Nous souderons donc une antenne d’une longueur de 17.3cm.

Remarque : Si le fil de cuivre est verni, il faut veiller à le retirer avec du papier de verre.

![moduleRadio](https://github.com/Haki-i/Smart-alert-home-doorbell-/assets/137703849/6e3d09d0-6311-4b19-b2e9-22630bb3ca3d)

# II- Conception Informatique

## a) Notifications sur téléphone

Lorsque notre module radio recevra le code de la sonnette souhaité, notre Nodemcu communiquera avec le broker MQTT Adafruit. Nous utiliserons l’application IFTTT pour que cette interaction permette d’automatiser d’autres actions comme l’envoi d’une notification.

### Broker Adafruit IO

Pour utiliser le broker Adafruit, nous devons dans un premier temps créer un nouveau *feed*. Ce *feed* fonctionne de la même manière qu'un *topic* dans le protocole MQTT, des appareils peuvent y publier des messages ou s'abonner.

Dans notre programme, nous utilisons la bibliothèque *Adafruit_MQTT*, nous indiquons les informations de connexion à Adafruit (Username, clé d'authentification...) ainsi que le nom du *feed* créé.
Si la sonnette est pressée, la Nodemcu publie à notre *feed* la valeur 1. Après quelques secondes, elle publie la valeur 0.

### Création d’un Applet IFTTT

IFTTT (If This Then That) est une application qui, comme son nom l'indique, permet d'effectuer des actions en réponse à un événement. Elle utilise des API pour permettre à des dispositifs de communiquer entre eux.

En créant un nouvel Applet, nous allons lui associer notre compte Adafruit et plus précisément le *feed* qui recevra les données issues de la Nodemcu. L'événement déclencheur d'une action est la réception de la valeur 1 sur ce *feed* (If This).
Si cet événement se produit, alors une notification apparaît "Une personne est en train de sonner" (Then That).

## b) Clignotement lumière

Lorsqu’une personne sonne, nous voulons également que nos lumières connectées clignotent. Pour cela, nous pouvons également utiliser l'application IFTTT de la même manière. Cependant, afin de diversifier ce projet et d'obtenir un meilleur contrôle, nous utiliserons cette fois-ci l'API de Philips Hue.
Notre Nodemcu pourra interagir avec nos LED et communiquer avec notre pont Hue en envoyant des requêtes HTTP à cette API sur notre réseau local.

[https://developers.meethue.com/develop/get-started-2/](https://developers.meethue.com/develop/get-started-2/)

### Accéder à l’interface de test

Nous devons dans un premier temps nous rendre sur un navigateur et accéder à l’interface de test suivante : *https://bridgeipaddress/debug/clip.html*

L’adresse IP locale de notre pont peut se trouver sur l’application Hue.

### Création d’un nouvel utilisateur

Après avoir appuyé sur notre pont (par mesure de sécurité), nous lançons cette requête :

| URL | /api |
| --- | --- |
| Body | {"devicetype":"my_hue_app#Hakii"} |
| Method | POST |

Cette commande dit de créer une nouvelle ressource dans /api (où se trouvent les noms d'utilisateur) avec les propriétés suivantes. Nous obtenons un nom d’utilisateur aléatoire (tel que 1028d66426293e821ecfd9ef1a0731df)

### Contrôle des lumières

Pour allumer ou éteindre une lumière, nous envoyons :

| URL | https://<bridgeipaddress>/api/UserName/lights/lightid/state |
| --- | --- |
| Body | {"on":false} |
| Method | PUT |

Nous nous adressons à l'objet « state » de lightid et lui disons de modifier la valeur « on » à l'intérieur en false.

Pour modifier la couleur d’une lumière, nous envoyons : 

| URL | https://<bridgeipaddress>/api/UserName/lights/lightid/state |
| --- | --- |
| Body | {"on":true, "sat":254, "bri":254,"hue":10000} |
| Method | PUT |

Nous nous assurons que la lumière est allumée en définissant la ressource « on » sur true. Nous nous assurons également que la saturation (intensité) des couleurs et la luminosité sont à leur maximum en réglant les ressources « sat » et « bri » à 254. Enfin, nous disons au système de définir la « teinte » (une mesure de couleur) à 10 000 points (la teinte va de 0 à 65 535).

Une fois que nous avons compris comment contrôler nos LED par requête  HTTP sur l’interface de test, nous pouvons faire la même chose avec notre Nodemcu et la bibliothèque *ESP8266HTTPClient*

En conclusion, chaque fois que le module radio reçoit le signal d’intérêt, nous envoyons sur notre *feed* Adafruit la valeur 1 ce qui active une notification sur l’application IFTTT et nous envoyons une requête HTPP pour faire clignoter nos lumières.

# Rendu final
