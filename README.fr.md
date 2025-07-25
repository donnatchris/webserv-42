# PROJET WEBSERV POUR 42
Par **chdonnat** (Christophe Donnat, 42 Perpignan – France)

Avec mes co-équipiers : **Olivier Thorel** et **Lucas Matkowski**, également de 42 Perpignan – France.

<p align="center">
  <img src="https://github.com/donnatchris/webserv/blob/main/screenshots/index.png" width="30%" height="200px" />
  <img src="https://github.com/donnatchris/webserv/blob/main/screenshots/webserv.png" width="30%" height="200px" />
  <img src="https://github.com/donnatchris/webserv/blob/main/screenshots/siege_results.png" width="30%" height="200px" />
</p>


## BUT DU PROJET :

**Webserv** est un projet d'équipe (3 membres) qui consiste à créer un serveur HTTP entièrement fonctionnel en C++, conforme à la spécification HTTP/1.1 (RFC 2616).
Le serveur doit être capable de :
- Parser et appliquer plusieurs fichiers de configuration
- Gérer les méthodes HTTP : GET, POST, et DELETE
- Servir des fichiers statiques et générer des pages d'autoindex
- Exécuter des scripts CGI
- Supporter plusieurs serveurs virtuels
- Gérer plusieurs connexions simultanées en utilisant `poll()`

L'objectif est d'acquérir une compréhension approfondie du fonctionnement interne des serveurs web, et d'implémenter la programmation de sockets de bas niveau et le multiplexage d'E/S en C++98.


## PARTIE BONUS :

Les fonctionnalités bonus suivantes ont été implémentées :
- Gestion des sessions basée sur les cookies
- Support de multiples CGI (dans notre projet : Python et PHP)

Ces bonus démontrent une maîtrise plus approfondie des mécanismes HTTP et reflètent le fonctionnement des applications web du monde réel.


## CE DONT NOUS SOMMES FIERS :

Comme nous avons vraiment apprécié travailler sur le projet, nous sommes allés plus loin et avons implémenté :
- Compatibilité totale avec **macOS** et **Linux**
- Support de l'upload de fichiers via `multipart/form-data` (limité à un fichier à la fois), permettant l'envoi de n'importe quel type de fichier (PNG, PDF, MP3, etc.)
- Communication "chunked" (par blocs de 4096 octets), y compris pour les uploads, afin d'optimiser l'utilisation de la mémoire
- Un point d'entrée unique : la classe `Server` communique avec `ProcessRequest` exclusivement via sa méthode `process()`, gérant de manière transparente les données entrantes et les réponses sortantes
- Un **site web de test complet et amusant sur le thème des zombies** pour démontrer et valider toutes les fonctionnalités
- Résultats des tests de charge (load testing) avec Siege : 100% de succès sur plus de 250 000 transactions en 1 minute.


## QUELQUES COMMANDES UTILES :

Compiler le programme et supprimer les fichiers .o :

	make && make clean

 ---

Exécuter le programme avec le fichier de configuration par défaut :

(lancer le programme sans argument utilise le fichier de configuration `config/default.conf`)

	./webserv webserv.conf

Avec ce fichier de configuration, vous pouvez maintenant tester un site web que nous avons créé pour tester l'ensemble du projet :

Dans votre navigateur préféré, tapez :

	http://localhost:8000

 ---

 Exécuter le programme avec un autre fichier de configuration

	./webserv config/

Avec ce fichier de configuration, vous pouvez maintenant tester 3 serveurs virtuels, chacun avec son propre site de test.

  Dans votre navigateur préféré, tapez :

	http://localhost:8000

 	http://localhost:80808

  	http://localhost:8888

## TEST DE CHARGE AVEC SIEGE

Le test de charge ("load testing") est une méthode utilisée pour évaluer les performances d'un serveur web sous un trafic élevé.
Il simule de nombreux clients effectuant des requêtes simultanément pour vérifier la stabilité, le temps de réponse et le débit du serveur.
Siege est un outil en ligne de commande populaire pour effectuer de tels tests.
Il est gratuit et disponible sur la plupart des systèmes — vous pouvez l'installer avec un gestionnaire de paquets comme `brew install siege` (macOS) ou `sudo apt install siege` (Linux).

Une commande de base pour lancer un test de charge sur notre serveur serait (dans un autre terminal pendant que le serveur tourne) :

```bash
siege -v http://localhost:8888
```

**Attention** : Assurez-vous de tester un serveur qui ne contient pas de liens externes (par exemple, Google Fonts ou des ressources CDN).
Siege suivra ces liens et enverra des requêtes à des serveurs tiers, ce qui est non seulement inapproprié (et potentiellement illégal), mais provoquera également des transactions échouées sans rapport avec votre propre serveur.

> Pour notre Webserv, lors de l'utilisation du fichier de configuration `config/webserv.conf`, veillez à ne tester que le serveur sur le port 8888, car celui sur le port 8000 peut inclure des liens externes vers les serveurs de Google (pour les polices).

**Nous avons mis Webserv à l'épreuve avec un trafic intense — et il a tenu la charge. Voici les résultats :**
```
Lifting the server siege...
Transactions:		   262650    hits
Availability:		      100.00 %
Elapsed time:		       60.98 secs
Data transferred:	      399.02 MB
Response time:		        5.77 ms
Transaction rate:	     4307.15 trans/sec
Throughput:		        6.54 MB/sec
Concurrency:		       24.86
Successful transactions:   262650
Failed transactions:	        0
Longest transaction:	       40.00 ms
Shortest transaction:	        0.00 ms
```


## ARCHITECTURE :

- `config/` — Contient des exemples de fichiers de configuration utilisés pour tester le projet.
- `documentation_fr/` — Documentation et notes techniques en français.
- `include/` — Tous les fichiers d'en-tête (.hpp) du projet.
- `screenshots` - Captures d'écran du projet.
- `src/` — Code source du serveur, organisé par responsabilité :
  - `config/` — Classes pour le parsing et le stockage des données de configuration.
  - `http/` — Classes pour le parsing des requêtes HTTP et la construction des réponses HTTP.
  - `process/` — Classes pour le traitement des données client et la gestion de la logique de communication.
  - `server/` — Classes pour la gestion des sockets, des connexions et de la boucle principale du serveur.
- `www/` — Contient les fichiers statiques et les sites de test (HTML, CSS, images et scripts CGI).
- `Makefile` — Système de build avec les règles suivantes : `make`, `bonus`, `clean`, `fclean`, `re`.
- `README.md` — Présentation du projet et instructions d'utilisation clés.


## CLASSES ET RESPONSABILITÉS

Webserv est structuré autour de classes modulaires à responsabilité unique qui collaborent pour traiter efficacement les requêtes HTTP :

- **`Server`**
  Le point d'entrée du serveur. Il ouvre les sockets d'écoute, gère les connexions et pilote la boucle d'événements principale à l'aide d'un mécanisme de multiplexage (`poll`, `epoll`, etc.). Il interagit avec chaque client via la méthode `ProcessRequest::process()`.

- **`PollManager`**
  Un wrapper autour de l'API de multiplexage au niveau de l'OS (`poll`, `kqueue`, etc.). Il abstrait l'enregistrement des descripteurs de fichiers, la disponibilité des événements (POLLIN/POLLOUT) et la distribution des événements, garantissant des E/S non bloquantes efficaces.

- **`Connection`**
  Représente une connexion client. Elle stocke le socket client, suit son état de lecture/écriture et délègue le traitement des données à une instance de `ProcessRequest`.

- **`ProcessRequest`**
  Au cœur du traitement des requêtes. Il expose une interface unique `process()` utilisée par le serveur à la fois pour la réception et l'envoi de données. Il orchestre le parsing, la répartition logique, l'exécution CGI et la préparation de la réponse.

- **`RequestParser`**
  Transforme les données brutes en un objet `HttpRequest` valide. Il gère les en-têtes, le "chunked transfer encoding", le `Content-Length` et les entrées invalides avec élégance.

- **`HttpRequest`**
  Contient une requête HTTP entièrement parsée : méthode, URI, en-têtes et corps optionnel. Utilisé par `ProcessRequest` pour prendre des décisions de routage et de logique.

- **`HttpResponse`**
  Construit la réponse HTTP ligne par ligne : statut, en-têtes et corps. Il prend en charge les types de contenu, la longueur du contenu, la redirection et le formatage des erreurs.

- **`ResponseBuilder`**
  (Hérité / aide) Aide à la construction des réponses HTTP avant d'être remplacé ou intégré dans `ProcessRequest` pour le formatage final.

- **`CGIHandler`**
  Exécute les scripts CGI (par exemple, Python, PHP). Il configure les variables d'environnement, gère les pipes et la redirection des E/S, et lit la sortie du script pour construire une réponse HTTP valide.

- **`File`**
  Gère l'accès aux fichiers sur le disque. Valide les permissions, détermine les types MIME et gère les lectures/écritures "chunked" pour un service de fichiers efficace.

- **`ConfigParser`**
  Lit et tokenise le fichier `.conf`. Il extrait les blocs (`server`, `location`) et les directives (`listen`, `root`, etc.) dans une structure interne.

- **`ConfigValidator`**
  Vérifie la configuration parsée pour sa correction sémantique : adresses IP valides, ports, doublons, directives manquantes et intégrité structurelle.

- **`ServerConfig` / `Location`**
  Représentent la configuration hiérarchique du serveur. `ServerConfig` contient les paramètres globaux et par serveur ; `Location` définit les surcharges et les comportements spécifiques à un chemin.

- **`HttpErrorException`**
  Une exception personnalisée utilisée pour signaler les erreurs au niveau HTTP pendant le traitement des requêtes (par exemple, 404 Not Found, 405 Method Not Allowed), permettant une gestion propre des erreurs et la génération de réponses.

---

## DOCUMENTATION

Contrairement à la plupart de mes projets, vous trouverez une **documentation et des notes techniques en français** dans le répertoire `documentation_fr/`.

Ce répertoire contient les fichiers suivants, rédigés pendant notre phase de recherche avant de nous lancer dans l'implémentation :

- `CGI.fr.md` — Une explication rapide de ce qu'est le CGI et de son fonctionnement
- `fichiers_de_config.fr.md` — Une analyse de ce qui doit être géré dans les fichiers de configuration pour un serveur conforme
- `fonctions_doc` — Un répertoire contenant des fichiers avec des listes et des explications de toutes les fonctions C autorisées dans le projet
- `HTTP1.1.fr.md` — Une version résumée et accessible de la RFC 2616, qui définit le protocole HTTP/1.1
- `webserv_doc.fr.md` — Un aperçu complet de tous les concepts clés requis pour le projet - Vous devriez absolument commencer par ici si vous êtes sur le point de réaliser ce projet.


Vous trouverez ci-dessous un résumé des principaux concepts que vous devez comprendre pour travailler sur le projet. La suite de ce document constitue un résumé de ces fichiers.

---

### Un serveur Web en C++

#### Définition

Un serveur web est un programme qui :

- Écoute les requêtes HTTP des clients (comme les navigateurs)
- Parse ces requêtes
- Recherche la ressource demandée (fichier, script, etc.)
- Et renvoie une réponse HTTP structurée, généralement du HTML ou un autre contenu.

> En bref, un serveur web est un traducteur entre un client qui pose une question et un système de fichiers ou un environnement d'exécution qui fournit la réponse.

#### Fichier de Configuration

Le programme Webserv doit être lancé avec le chemin d'un fichier de configuration en argument. C'est un **fichier texte brut**, généralement avec l'extension `.conf`, qui décrit **comment le serveur web doit se comporter**, et doit être parsé à l'exécution.
Le programme doit **lire ce fichier** et **configurer dynamiquement le serveur** selon son contenu.

> Si le programme est lancé sans argument, il doit utiliser un fichier de configuration par défaut situé à un chemin connu codé en dur (par exemple : `"./default.conf"` ou `"/etc/webserv.conf"`).

Il n'y a pas de grammaire strictement imposée pour les fichiers de configuration, mais dans Webserv, nous suivons principalement le style des **fichiers de configuration NGINX**, car NGINX est un serveur web léger et hautement configurable. Ceux-ci sont structurés avec :

1.  **Blocs** (par ex., `server`, `location`)
2.  **Directives** (par ex., `listen`, `root`, `index`) se terminant par `;`
3.  Une **structure hiérarchique logique**

***→ Les fichiers de configuration sont expliqués en détail dans le fichier `fichiers_de_config.fr.md`***

#### 1. Écoute

Le serveur ouvre un **socket TCP** sur un **port** (généralement 80 ou 8080) pour attendre les connexions entrantes.

#### 2. Acceptation

Le serveur accepte une connexion et reçoit une requête HTTP, par exemple :

	GET /index.html HTTP/1.1
	Host: localhost


#### 3. Parsing

Le serveur analyse la requête :

- Méthode (`GET`, `POST`, etc.)
- Chemin (`/index.html`)
- En-têtes (`Host`, `Content-Type`, etc.)

#### 4. Réponse

Le serveur construit une réponse HTTP incluant :

- Une **ligne de statut** (`HTTP/1.1 200 OK`)
- Des **en-têtes** (`Content-Type`, `Content-Length`, etc.)
- Un **corps** (souvent du HTML, du JSON ou une image)

#### 5. Gestion de Plusieurs Clients

Il doit être capable de gérer **plusieurs connexions simultanées** sans bloquer.

#### 6. Exécution de Scripts (CGI)

Si la requête cible un script (par ex., `.py`, `.php`), le serveur doit :

- Créer un processus enfant avec `fork()`
- Exécuter le script avec `execve()`
- Capturer sa sortie
- Envoyer cette sortie comme réponse

#### Résumé de Webserv

En résumé, le programme Webserv doit être capable de :

- Lire et appliquer un fichier de configuration
- Écouter sur des sockets
- Comprendre le protocole HTTP (requêtes/réponses)
- Ouvrir et lire des fichiers
- Exécuter des scripts CGI
- Gérer les erreurs (404, 500, etc.)
- Supporter plusieurs serveurs configurables

---

### Un serveur Web en C++

#### Définition

Un serveur web est un programme qui :

- Écoute les requêtes HTTP des clients (comme les navigateurs)
- Parse ces requêtes
- Recherche la ressource demandée (fichier, script, etc.)
- Et renvoie une réponse HTTP structurée, généralement du HTML ou un autre contenu.

> En bref, un serveur web est un traducteur entre un client qui pose une question et un système de fichiers ou un environnement d'exécution qui fournit la réponse.

#### Fichier de Configuration

Le programme Webserv doit être lancé avec le chemin d'un fichier de configuration en argument. C'est un **fichier texte brut**, généralement avec l'extension `.conf`, qui décrit **comment le serveur web doit se comporter**, et doit être parsé à l'exécution.
Le programme doit **lire ce fichier** et **configurer dynamiquement le serveur** selon son contenu.

> Si le programme est lancé sans argument, il doit utiliser un fichier de configuration par défaut situé à un chemin connu codé en dur (par exemple : `"./default.conf"` ou `"/etc/webserv.conf"`).

Il n'y a pas de grammaire strictement imposée pour les fichiers de configuration, mais dans Webserv, nous suivons principalement le style des **fichiers de configuration NGINX**, car NGINX est un serveur web léger et hautement configurable. Ceux-ci sont structurés avec :

1.  **Blocs** (par ex., `server`, `location`)
2.  **Directives** (par ex., `listen`, `root`, `index`) se terminant par `;`
3.  Une **structure hiérarchique logique**

***→ Les fichiers de configuration sont expliqués en détail dans le fichier `fichiers_de_config.fr.md`***

#### 1. Écoute

Le serveur ouvre un **socket TCP** sur un **port** (généralement 80 ou 8080) pour attendre les connexions entrantes.

#### 2. Acceptation

Le serveur accepte une connexion et reçoit une requête HTTP, par exemple :

	GET /index.html HTTP/1.1
	Host: localhost


#### 3. Parsing

Le serveur analyse la requête :

- Méthode (`GET`, `POST`, etc.)
- Chemin (`/index.html`)
- En-têtes (`Host`, `Content-Type`, etc.)

#### 4. Réponse

Le serveur construit une réponse HTTP incluant :

- Une **ligne de statut** (`HTTP/1.1 200 OK`)
- Des **en-têtes** (`Content-Type`, `Content-Length`, etc.)
- Un **corps** (souvent du HTML, du JSON ou une image)

#### 5. Gestion de Plusieurs Clients

Il doit être capable de gérer **plusieurs connexions simultanées** sans bloquer.

#### 6. Exécution de Scripts (CGI)

Si la requête cible un script (par ex., `.py`, `.php`), le serveur doit :

- Créer un processus enfant avec `fork()`
- Exécuter le script avec `execve()`
- Capturer sa sortie
- Envoyer cette sortie comme réponse

#### Résumé de Webserv

En résumé, le programme Webserv doit être capable de :

- Lire et appliquer un fichier de configuration
- Écouter sur des sockets
- Comprendre le protocole HTTP (requêtes/réponses)
- Ouvrir et lire des fichiers
- Exécuter des scripts CGI
- Gérer les erreurs (404, 500, etc.)
- Supporter plusieurs serveurs configurables

---

### Quelques Définitions de Base

#### Client

> Un **client** est un programme (souvent un **navigateur web**) qui envoie une **requête HTTP** à un serveur web pour demander une **ressource** (page HTML, image, fichier, script, etc.).

Dans le contexte de Webserv :

- Le **client** se connecte au serveur Webserv via le réseau
- Il envoie une requête
- Il attend une réponse HTTP

#### Requête HTTP

> Une **requête HTTP** est un **message textuel** envoyé par un client à un serveur pour demander ou envoyer des données, en suivant une structure spécifique.

Elle commence par une **ligne de requête** comme :

	GET /index.html HTTP/1.1


Viennent ensuite une série d'**en-têtes** avec des informations supplémentaires (comme le nom de l'hôte ou les préférences de langue : `Host`, `User-Agent`, etc.), suivis d'une ligne vide, et parfois d'un **corps** (surtout dans les requêtes `POST`).

Le serveur lit et interprète la requête, puis renvoie une **réponse HTTP structurée**.

***→ Les requêtes HTTP sont expliquées en détail dans le fichier `HTTP1.1.fr.md`***

#### HTML (HyperText Markup Language)

> **HTML** est un **langage de balisage** utilisé pour structurer et afficher du contenu sur les pages web.

Un fichier HTML est une **ressource** que le serveur peut livrer au client.

Exemple simple :

```html
<html>
  <head><title>Ma page</title></head>
  <body><h1>Bonjour !</h1></body>
</html>
```

#### HTML (HyperText Markup Language)

Le serveur Webserv ne parse pas le HTML — **il l'envoie simplement tel quel**.
C'est le navigateur qui est responsable de son interprétation.


#### Socket

Un **socket** est une **interface logicielle** (un morceau de code à l'intérieur d'un programme) qui permet la communication sur un réseau en utilisant des protocoles comme TCP ou UDP.

Dans Webserv :

- Le serveur crée un socket avec `socket()`
- Le lie à une adresse et un port avec `bind()`
- Écoute les connexions entrantes avec `listen()`
- Accepte les connexions avec `accept()`

En C/C++, un socket est représenté par un **descripteur de fichier** (un `int`), ce qui permet d'utiliser des fonctions standard comme `read()` et `write()` pour la communication réseau.


#### Port

Un **port** est un **numéro logique** utilisé pour identifier une **application spécifique** sur une machine.

- Le **Port 80** est le **port par défaut pour HTTP**
- Le **Port 443** est utilisé pour **HTTPS**
- Le **Port 8080** est souvent utilisé comme **alternative au port 80**, notamment :
  - lorsque vous n'avez pas les privilèges root
  - pour les serveurs de développement/test locaux

> Le serveur Webserv écoute sur un port (par ex., `8080`) pour accepter les connexions HTTP.


#### TCP (Transmission Control Protocol)

**TCP** est un **protocole de transport fiable** utilisé pour transmettre des données entre deux machines sur un réseau, en s'assurant que les données :

- arrivent dans le **bon ordre**
- ne sont **pas perdues**
- ne sont **pas dupliquées**

Contrairement à d'autres protocoles comme UDP (qui est rapide mais non fiable), TCP établit une **connexion stable** (une session) entre deux points d'extrémité avant de transmettre des données.

HTTP est construit sur TCP, ce qui signifie :

- Une requête HTTP est envoyée **à travers un flux TCP**
- Webserv utilise un **socket TCP** pour recevoir les requêtes
- Les clients peuvent compter sur une **transmission complète et ordonnée** des messages

> Grâce à TCP, le serveur peut lire une requête HTTP **en toute sécurité**, sans avoir à gérer manuellement l'ordre ou l'intégrité des paquets.


#### UDP (User Datagram Protocol)

**UDP** est un **protocole de transport rapide mais non fiable**, utilisé pour envoyer de **petits paquets de données** sans établir de connexion préalable.

Contrairement à TCP, UDP ne **garantit pas** :

- Que les paquets arrivent **dans le bon ordre**
- Que les paquets arrivent **tout court** (la perte de paquets est possible)
- Que les paquets ne sont **pas dupliqués**

UDP est donc :

- **Plus léger**
- **Plus rapide**
- Mais aussi **moins fiable**

Chaque message envoyé est appelé un **datagramme**, indépendant des autres.
UDP n'offre **aucune vérification d'erreur ni retransmission**.

Il est souvent utilisé pour :

- Les jeux en ligne
- Le streaming vidéo en temps réel
- Les requêtes DNS
- Les applications qui tolèrent une certaine perte de données

> En bref, **UDP échange la fiabilité contre la vitesse**, ce qui le rend adapté aux communications où **la réactivité compte plus que la perfection**.


#### CGI (Common Gateway Interface)

**CGI** est une **interface standard** qui permet à un serveur web d'**exécuter un programme externe** (comme un script PHP ou Python) et de **retourner sa sortie comme une réponse HTTP**.

Webserv doit :

- Détecter qu'un fichier est un **script CGI**
- Appeler `fork()` pour créer un processus enfant
- Utiliser `execve()` pour exécuter le script
- Lire la sortie du script (via un pipe)
- Renvoyer cette sortie comme réponse HTTP

***→ Le CGI est expliqué en détail dans le fichier `CGI.fr.md`***

### Fonctions Autorisées

Le projet Webserv autorise l'utilisation de tout ce qui est compatible avec **C++98**, sans Boost ni autres bibliothèques externes.
Le sujet liste de nombreuses fonctions de niveau système, qui peuvent être regroupées en catégories :

#### Gestion et Exécution de Processus

> Ces fonctions permettent au serveur de **lancer des programmes externes (comme les scripts CGI)**, de **gérer les processus enfants** et de **rediriger leurs entrées/sorties**.

| Fonction | Objectif Principal |
| :--- | :--- |
| `execve` | Exécute un programme externe (par ex., un script CGI) |
| `fork` | Crée un processus enfant (utilisé pour le CGI) |
| `waitpid` | Attend la fin d'un processus enfant |
| `kill` | Envoie un signal à un processus |
| `signal` | Définit un gestionnaire personnalisé pour un signal (par ex., `SIGINT`) |
| `errno` | Variable globale contenant le dernier code d'erreur système |
| `strerror` | Renvoie un message lisible pour la valeur `errno` actuelle |


#### Redirection et Duplication de Descripteurs de Fichiers

> Utilisé pour **rediriger l'entrée/sortie standard**, notamment pour l'exécution de CGI et la gestion des pipes.

| Fonction | Objectif Principal |
| :--- | :--- |
| `dup` | Duplique un descripteur de fichier |
| `dup2` | Duplique un descripteur vers une cible spécifique |
| `pipe` | Crée une paire de descripteurs connectés pour l'IPC |


#### Réseau — **Sockets**

> Ce sont des fonctions essentielles pour construire un **serveur web TCP**, accepter des connexions et envoyer/recevoir des requêtes.

| Fonction | Objectif Principal |
| :--- | :--- |
| `socket` | Crée un socket TCP |
| `bind` | Lie un socket à une adresse IP et un port |
| `listen` | Met le socket en mode écoute |
| `accept` | Accepte une connexion entrante |
| `connect` | Initie une connexion à un socket distant |
| `recv` | Reçoit des données d'un socket |
| `send` | Envoie des données via un socket |
| `setsockopt` | Configure des options sur un socket |
| `getsockname` | Récupère l'adresse locale d'un socket |
| `socketpair` | Crée une paire de sockets connectés (par ex., pipe CGI bidirectionnel) |


#### Multiplexage d'E/S — **Gérer Plusieurs Connexions Simultanément**

> Ces interfaces permettent au serveur de **surveiller plusieurs sockets** pour la disponibilité en lecture/écriture sans utiliser de threads.

| Interface | Fonctions Associées |
| :--- | :--- |
| `select` | Méthode portable, limitée à 1024 descripteurs de fichiers |
| `poll` | Plus flexible que `select`, supporte plus de fds |
| `epoll` | Pour Linux : `epoll_create`, `epoll_ctl`, `epoll_wait` |
| `kqueue` | Pour macOS : `kqueue`, `kevent` |


#### Résolution de Noms et Configuration Réseau

> Ces fonctions aident à **traduire les noms d'hôtes en IP**, à configurer les protocoles et à gérer les ports.

| Fonction | Objectif Principal |
| :--- | :--- |
| `getaddrinfo` | Résout un nom d'hôte (par ex., `localhost` → adresse IP) |
| `freeaddrinfo` | Libère la mémoire allouée par `getaddrinfo` |
| `getprotobyname` | Récupère le numéro d'un protocole (par ex., `"tcp"`) |
| `htons`, `htonl` | Convertit les entiers en ordre d'octets réseau |
| `ntohs`, `ntohl` | Convertit les entiers **depuis** l'ordre d'octets réseau |


#### Accès aux Fichiers et au Système de Fichiers

> Utilisé pour **servir des fichiers HTML, des images, etc.**, vérifier l'existence et les permissions, et parcourir les répertoires.

| Fonction | Objectif Principal |
| :--- | :--- |
| `access` | Vérifie si un fichier existe ou a les bonnes permissions |
| `stat` | Récupère les métadonnées d'un fichier |
| `open`, `read`, `write`, `close` | Ouvre, lit, écrit et ferme un fichier |
| `chdir` | Change le répertoire de travail courant (par ex., pour le CGI) |
| `opendir`, `readdir`, `closedir` | Ouvre, lit et ferme des répertoires |

---

### Cycle de Vie d'une Connexion HTTP dans Webserv

#### 1. **Initialisation du Serveur**

- `socket()` → crée un socket d'écoute (`SOCK_STREAM`)
- `setsockopt()` → active l'option `SO_REUSEADDR`
- `bind()` → lie le socket à un `host:port` spécifique
- `listen()` → met le socket en mode écoute
- `epoll_create()` / `poll()` → met en place le mécanisme de boucle d'événements


#### 2. **Attente de Connexions (Événement `EPOLLIN` ou `POLLIN`)**

- `accept()` → accepte une connexion entrante
- `epoll_ctl(ADD)` → ajoute le nouveau socket client à l'ensemble poll/epoll
- le socket client est mis en **mode non bloquant**


#### 3. **Réception de la Requête HTTP**

- `recv()` → lit la requête depuis le socket client
- `parse()` → analyse :
  - la méthode HTTP (GET, POST…)
  - l'URI / chemin
  - les en-têtes (Host, Content-Length…)
  - le corps optionnel (par ex., pour POST)


#### 4. **Traitement de la Requête**

- **Fichier statique :**
  - `access()` → vérifie si le fichier existe et si l'accès est autorisé
  - `stat()` → récupère la taille/type du fichier
  - `open()` + `read()` → lit le fichier demandé
  - construit une **réponse HTTP complète**

- **Script CGI :**
  - `fork()` → crée un processus enfant
  - `pipe()` / `socketpair()` → met en place la communication avec le CGI
  - `dup2()` → redirige `stdin` et `stdout`
  - `execve()` → exécute le script
  - `waitpid()` → attend que le CGI se termine
  - `read()` → lit la sortie du CGI
  - parse et **injecte la sortie dans la réponse HTTP**


#### 5. **Envoi de la Réponse**

- `send()` → envoie la réponse HTTP (en-têtes + corps)
- (optionnel : passer de `EPOLLIN` à `EPOLLOUT` en mode non bloquant)


#### 6. **Fermeture de la Connexion**

- `close()` → ferme le socket client (ou `epoll_ctl(DEL)`)
- ou le **keep-alive** est maintenu en fonction de l'en-tête `Connection: keep-alive` et de la configuration du timeout


#### Résumé du Traitement d'une Requête

```text
le client se connecte → accept() → recv() → parse()
          ↓
  [fichier statique]      [script CGI]
       ↓                      ↓
access/stat/open     fork() → execve()
       ↓                      ↓
     read()                 pipe/read
       ↓                      ↓
 réponse HTTP             réponse HTTP
          ↓
        send()
          ↓
       close()
```

---

### Multiplexage

#### Définition du Multiplexage (alias **Multiplexage d'E/S**)

> Le **multiplexage** fait référence à la capacité d'un programme à **surveiller plusieurs sources d'entrée/sortie simultanément** sans bloquer, et à réagir **dès que l'une d'entre elles devient active** (par ex., un socket prêt pour la lecture ou l'écriture).

Au lieu de faire :

```cpp
read(socket1);
read(socket2);
read(socket3);
```

### Multiplexage

#### ...et de bloquer sur chaque socket ?

Au lieu de bloquer sur chaque socket un par un, on demande à l'OS :

> “Préviens-moi dès que **l'un d'entre eux** a quelque chose à dire.”

---

### Le Multiplexage dans Webserv

Le serveur doit :

- Écouter **plusieurs clients simultanément**
- **Éviter de bloquer** sur des sockets inactifs
- **Réagir immédiatement** lorsqu'un client devient prêt

Le multiplexage permet de faire tout cela avec un **seul thread/processus**, de manière **non bloquante** et **efficace**.

> ⚠️ Le sujet interdit explicitement d'utiliser `fork()` ou des threads pour gérer plusieurs connexions :
>
> *"Vous ne pouvez pas utiliser fork pour autre chose que le CGI (comme PHP ou Python, etc.)."*


### Mécanismes de Multiplexage Courants

Le sujet autorise le choix d'**un seul et unique** mécanisme de multiplexage.
Vous ne devez **jamais appeler `read()` ou `write()`** à moins que le mécanisme choisi n'ait confirmé que le descripteur de fichier est prêt.

| Mécanisme | Systèmes Supportés | Niveau |
| :--- | :--- | :--- |
| `select()` | POSIX (Linux, macOS) | Basique |
| `poll()` | POSIX (Linux, macOS) | Intermédiaire |
| `epoll()` | Linux uniquement | Avancé |
| `kqueue()` | BSD/macOS uniquement | Avancé |


#### 1. `select()`

Cette fonction utilise des **ensembles statiques (`fd_set`)** et des macros comme `FD_SET`.

**Avantages :**
- Très simple à comprendre
- Universel (supporté partout)

**Inconvénients :**
- Limité à **1024 descripteurs de fichiers** (`FD_SETSIZE`)
- Mauvaises performances avec de nombreux sockets (scan linéaire)
- Syntaxe verbeuse et rigide


#### 2. `poll()`

Cette fonction utilise un **tableau dynamique de structures `pollfd`** — un bon choix portable.

**Avantages :**
- Pas de limite de 1024 FD
- Interface plus propre que `select()`
- Disponible sur tous les systèmes POSIX

**Inconvénients :**
- Utilise toujours un **scan linéaire**
- Nécessite de reconstruire le tableau à chaque itération


#### 3. `epoll()` (Linux uniquement)

Une interface **pilotée par les événements** conçue pour de hautes performances.

**Avantages :**
- Très rapide : O(1) pour `add`, `remove`, et `wait`
- Idéal pour des **milliers de connexions concurrentes**
- Ne retourne que les **FDs prêts**

**Inconvénients :**
- Spécifique à Linux
- API légèrement plus complexe (`epoll_create`, `epoll_ctl`, `epoll_wait`)
- Non portable


#### 4. `kqueue()` (macOS / BSD)

Équivalent de `epoll()` pour BSD/macOS.

**Avantages :**
- Hautes performances (similaires à `epoll`)
- Peut aussi surveiller des **fichiers, signaux et timers**

**Inconvénients :**
- Spécifique à macOS/BSD
- Syntaxe plus verbeuse (`kqueue`, `kevent`, etc.)


### Contraintes de Multiplexage pour Webserv

Le serveur **Webserv** doit :

- Fonctionner en **mode non bloquant**
- Utiliser un **unique mécanisme de multiplexage** (`poll()`, `select()`, `epoll()`, ou `kqueue()`)
- L'utiliser pour gérer **toutes les E/S**, y compris :
  - Le **socket d'écoute**
  - Les **sockets clients**
  - Les **pipes de communication CGI**


### Règles Obligatoires

- Utiliser **une boucle principale**
- Un seul appel à `poll()` (ou équivalent) par itération
- Cet appel doit surveiller :
  - Les **sockets prêts à lire** (`POLLIN`)
  - Les **sockets prêts à écrire** (`POLLOUT`)
- Vous ne devez **pas appeler `read()`/`recv()` ou `write()`/`send()`** à moins que le mécanisme de polling ne confirme la disponibilité
- Vous ne devez **pas vous fier aux valeurs de `errno`** (par ex., `EAGAIN`) pour gérer la disponibilité — cela doit être évité en utilisant `poll()` ou un équivalent


### Exception : Lecture du Fichier de Configuration

La lecture du fichier `.conf` **peut être faite en mode bloquant**.
Ces règles **s'appliquent uniquement aux opérations d'E/S sur les sockets et les pipes**.


### Notes pour `select()`

Si vous choisissez d'utiliser `select()`, les macros suivantes sont **requises** pour gérer les descripteurs de fichiers surveillés :

- `FD_ZERO` — Efface un ensemble de descripteurs
- `FD_SET`  — Ajoute un descripteur à l'ensemble
- `FD_CLR`  — Retire un descripteur de l'ensemble
- `FD_ISSET` — Vérifie si un descripteur est prêt (lecture/écriture)

> Ces macros sont **essentielles avec `select()`**,
> mais elles ne sont **pas utilisées avec `poll()` ou `epoll()`**.

---

### Telnet et NGINX

`telnet` et `nginx` sont des **outils en ligne de commande (CLI)** qui peuvent vous aider à comprendre et à tester Webserv.
Ils sont utiles pour :

- Tester manuellement des **requêtes HTTP brutes**, en combinant `telnet` et `nginx` pour apprendre comment une requête/réponse HTTP est structurée
- Voir comment le serveur réagit à des requêtes mal formées ou minimales
- Tester votre propre serveur Webserv en envoyant des requêtes HTTP via `telnet`

---

#### Telnet

> `telnet` est un **client réseau en ligne de commande** qui vous permet d'**ouvrir une connexion TCP** vers un hôte et un port donnés.
> C'est très utile pour **envoyer manuellement des requêtes HTTP** et observer la **réponse HTTP brute** du serveur.

Lancez `telnet` comme ceci :

```bash
telnet <hôte> <port>

# par exemple :
telnet localhost 8080    # port par défaut sur macOS
telnet localhost 80      # port par défaut sur Linux
```

#### Telnet — Taper une Requête HTTP Brute

Une fois connecté, `telnet` vous donne une invite où vous pouvez taper une **requête HTTP brute ligne par ligne**, en terminant par une **ligne vide** (appuyez deux fois sur Entrée) pour signaler la fin de la requête.

Exemple d'une requête HTTP brute dans `telnet` :

	GET / HTTP/1.1
	Host: localhost


Si `nginx` (ou Webserv) est à l'écoute sur le port donné, il recevra la requête et **renverra une réponse HTTP**, qui apparaîtra directement dans le terminal.


#### NGINX

`nginx` est un **serveur HTTP haute performance et largement utilisé**, souvent déployé dans des environnements de production.
Il sert de référence utile pour comprendre **comment un serveur HTTP du monde réel gère les requêtes**.

Dans le contexte de **Webserv**, `nginx` est utile pour :

- Comparer le **comportement de votre serveur** avec NGINX (même lors de l'évaluation — NGINX est la référence)
- Observer les **réponses aux cas d'erreur** (par ex., `404 Not Found`, `405 Method Not Allowed`)
- Étudier la **structure d'un fichier de configuration** (`nginx.conf`)
- L'utiliser comme **modèle** pour écrire votre propre fichier `.conf` pour Webserv


##### Démarrer `nginx`

```bash
brew services start nginx    # sur macOS (port par défaut : 8080)
sudo systemctl start nginx   # sur Linux (port par défaut : 80)
```

Vérifier si nginx est en cours d'exécution :

	lsof -i :8080    # sur macOS
	lsof -i :80      # on Linux
