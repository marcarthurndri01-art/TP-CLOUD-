# TP 3 — Premiers conteneurs avec Docker

**Cours :** Introduction au Cloud Computing — L3 SI (RIST & BDGL)
**Membres :** 
MAGASSOUBA CHEICK AHMED
KOUKO JOSHUA EZECKIEL
N'DRI N'DOUA DENIS MARC-ARTHUR
**Séance :** 3 — Virtualisation : hyperviseurs, VM et conteneurs

> 

---

## Partie 0 — Préparer la VM et installer Docker

### Commandes lancées

```powershell
# PowerShell (hôte)
$env:Path += ";C:\Program Files\Oracle\VirtualBox"
VBoxManage startvm ubuntu-srv --type headless
VBoxManage controlvm ubuntu-srv natpf1 "web,tcp,127.0.0.1,8080,,8080"
VBoxManage controlvm ubuntu-srv natpf1 "api,tcp,127.0.0.1,5000,,5000"
ssh -p 2222 etudiant@127.0.0.1
```

```bash
# VM ubuntu-srv
sudo apt update
sudo apt install -y docker.io docker-buildx docker-compose-v2
sudo usermod -aG docker $USER
exit
# (reconnexion SSH)
docker version
docker info | grep -E "Server Version|Storage Driver|Cgroup|Kernel Version"
```

### Sortie obtenue

```text
Client:
 Version:           29.1.3
 API version:       1.52
 Go version:        go1.24.13
 Git commit:        29.1.3-0ubuntu4.1
 Built:             Wed Apr 29 16:40:20 2026
 OS/Arch:           linux/amd64
 Context:           default

Server:
 Engine:
  Version:          29.1.3
  API version:      1.52 (minimum version 1.44)
  Go version:       go1.24.13
  Git commit:       29.1.3-0ubuntu4.1
  Built:            Wed Apr 29 16:40:20 2026
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          2.2.2
  GitCommit:
 runc:
  Version:          1.4.0-0ubuntu1
  GitCommit:
 docker-init:
  Version:          0.19.0
  GitCommit:

  Client:
 Version:    29.1.3
 Context:    default
 Debug Mode: false
 Plugins:
  buildx: Docker Buildx (Docker Inc.)
    Version:  0.30.1
    Path:     /usr/libexec/docker/cli-plugins/docker-buildx
  compose: Docker Compose (Docker Inc.)
    Version:  2.40.3+ds1-0ubuntu1
    Path:     /usr/libexec/docker/cli-plugins/docker-compose
  trust: Manage trust on Docker images (Docker Inc.)
    Version:  29.1.3
    Path:     /usr/libexec/docker/cli-plugins/docker-trust

Server Version: 29.1.3
 Storage Driver: overlayfs
 Cgroup Driver: systemd
 Cgroup Version: 2
 Kernel Version: 7.0.0-42-generic

### Q1. Quelle version de Docker est installée ? Quel noyau Docker utilise-t-il ?

Server Version: 29.1.3
 Storage Driver: overlayfs
 Cgroup Driver: systemd
 Cgroup Version: 2
 Kernel Version: 7.0.0-42-generic

Ce noyau est **celui de la VM Ubuntu elle-même** : Docker n'embarque pas de noyau propre. Les conteneurs partagent le noyau Linux de la machine qui les héberge. C'est ce qui les distingue d'une VM, qui a son propre noyau.

### Q2. Pourquoi être membre du groupe `docker` revient-il presque à être administrateur ? Quelle précaution sur un serveur partagé ?

Le démon Docker (`dockerd`) tourne en **root**, et le groupe `docker` donne accès à son socket (`/var/run/docker.sock`). Toute personne qui peut parler à ce socket peut demander au démon de créer un conteneur comme elle le souhaite, par exemple :

```bash
docker run -v /:/hote --privileged -it alpine chroot /hote
```

Cette commande monte tout le disque de la machine dans le conteneur, puis donne un shell **root** sur le système réel : lire `/etc/shadow`, modifier `/etc/sudoers`, installer une porte dérobée, etc. Cela se fait sans mot de passe et sans `sudo`. Le groupe `docker` est donc **équivalent à root**.

**Précautions sur un serveur partagé :**
- n'ajouter au groupe `docker` que des personnes de confiance, selon le principe du moindre privilège ;
- préférer le mode **rootless** (Docker sans droits root) ou `sudo docker` journalisé, qui laisse une trace des actions ;
- ne jamais exposer le socket Docker (ni en TCP sans TLS, ni monté dans un conteneur) ;
- auditer régulièrement la liste des membres du groupe (`getent group docker`).

---

## Partie 1 — Lancer et inspecter des conteneurs

### 1.1 Premier conteneur

```bash
docker run hello-world
docker images
docker ps -a
```

```text
etudiant@ubuntu-srv:~$ docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
4f55086f7dd0: Pull complete
d5e71e642bf5: Download complete
Digest: sha256:c567b231f65614688711a75f31518a064258a51d442117a3e771a3a8fdba78ef
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/

etudiant@ubuntu-srv:~$ docker images
                                            i Info →   U  In Use
IMAGE           ID             DISK USAGE   CONTENT SIZE   EXTRA
hello-world:latest
                c567b231f656       25.9kB         9.49kB    U
etudiant@ubuntu-srv:~$ docker ps -a
CONTAINER ID   IMAGE         COMMAND    CREATED              STATUS                          PORTS     NAMES
dec216f2f941   hello-world   "/hello"   About a minute ago   Exited (0) About a minute ago             confident_wozniak
etudiant@ubuntu-srv:~$ docker run -d --name web -p 8080:80 nginx
Unable to find image 'nginx:latest' locally
latest: Pulling from library/nginx
a670537bed70: Pull complete
9f604dae09b2: Pull complete
6e2ab1f3ef4e: Pull complete
6fc242d221a6: Pull complete
c139fe92686d: Pull complete
d322f1bf41d8: Pull complete
a071113ac575: Pull complete
05a7d8870b4b: Download complete
ca81da665177: Download complete
Digest: sha256:600f7cd45e27aa13e9304c834ab7c1b0ff39df85ee1dcc5087955455d77bc83b
Status: Downloaded newer image for nginx:latest
ef73edc87194115313f53e48470536ca86746c3ba7fb49c05089f12f181e882b
```

### 1.2 Un serveur web en une commande

```bash
docker run -d --name web -p 8080:80 nginx
docker ps
curl -s http://localhost:8080 | head -n 5
```

```text
etudiant@ubuntu-srv:~$ docker run -d --name web -p 8080:80 nginx
Unable to find image 'nginx:latest' locally
latest: Pulling from library/nginx
a670537bed70: Pull complete
9f604dae09b2: Pull complete
6e2ab1f3ef4e: Pull complete
6fc242d221a6: Pull complete
c139fe92686d: Pull complete
d322f1bf41d8: Pull complete
a071113ac575: Pull complete
05a7d8870b4b: Download complete
ca81da665177: Download complete
Digest: sha256:600f7cd45e27aa13e9304c834ab7c1b0ff39df85ee1dcc5087955455d77bc83b
Status: Downloaded newer image for nginx:latest

etudiant@ubuntu-srv:~$ docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS          PORTS                                     NAMES
ef73edc87194   nginx     "/docker-entrypoint.…"   26 seconds ago   Up 25 seconds   0.0.0.0:8080->80/tcp, [::]:8080->80/tcp   web

etudiant@ubuntu-srv:~$ curl -s http://localhost:8080 | head -n 5
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
```

### 1.3 Regarder à l'intérieur

```bash
docker exec web cat /etc/os-release | head -n 2
cat /etc/os-release | head -n 2
docker exec web uname -r
uname -r
docker logs --tail 5 web
docker top web
ps aux | grep "nginx: master" | grep -v grep
docker stats --no-stream web
```

```text
etudiant@ubuntu-srv:~$ docker exec web cat /etc/os-release | hea
d -n 2
PRETTY_NAME="Debian GNU/Linux 13 (trixie)"
NAME="Debian GNU/Linux"
etudiant@ubuntu-srv:~$ cat /etc/os-release | head -n 2
PRETTY_NAME="Ubuntu 26.04.1 LTS"
NAME="Ubuntu"
etudiant@ubuntu-srv:~$ docker exec web uname -r && uname -r
7.0.0-42-generic
7.0.0-42-generic

etudiant@ubuntu-srv:~$ docker logs --tail 5 web
2026/10/30 11:06:43 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1024:524288
2026/10/30 11:06:43 [notice] 1#1: start worker processes
2026/10/30 11:06:43 [notice] 1#1: start worker process 28
2026/10/30 11:06:43 [notice] 1#1: start worker process 29
172.17.0.1 - - [30/Oct/2026:11:08:38 +0000] "GET / HTTP/1.1" 200 896 "-" "curl/8.18.0" "-"
etudiant@ubuntu-srv:~$ docker top web
UID                 PID                 PPID                C                   STIME               TTY                 TIME                CMD
root                3094                3071                0                   11:06               ?                   00:00:00            nginx: master process nginx -g daemon off;
syslog              3169                3094                0                   11:06               ?                   00:00:00            nginx: worker process
syslog              3170                3094                0                   11:06               ?                   00:00:00            nginx: worker process
etudiant@ubuntu-srv:~$ ps aux | grep "nginx: master" | grep -v grep
root        3094  0.0  0.5  14976  9048 ?        Ss   11:06   0:00 nginx: master process nginx -g daemon off;

CONTAINER ID   NAME      CPU %     MEM USAGE / LIMIT     MEM %     NET I/O        BLOCK I/O     PIDS
ef73edc87194   web       0.00%     3.441MiB / 1.604GiB   0.21%     2kB / 1.68kB   0B / 24.6kB   3
```

### 1.4 Un conteneur est éphémère

```bash
docker exec web sh -c 'echo "<h1>Binôme NOM1 - NOM2</h1>" > /usr/share/nginx/html/index.html'
curl -s http://localhost:8080
docker rm -f web
docker run -d --name web -p 8080:80 nginx
curl -s http://localhost:8080 | grep -o "<title>.*</title>"
```

```bash
mkdir -p ~/site
echo "<h1>Binôme NOM1 - NOM2 : servi depuis un volume</h1>" > ~/site/index.html
docker rm -f web
docker run -d --name web -p 8080:80 -v ~/site:/usr/share/nginx/html:ro nginx
curl -s http://localhost:8080
```

```text
etudiant@ubuntu-srv:~$ curl -s http://localhost:8080
<h1>Binôme NOM1 - NOM2 : servi depuis un volume</h1>
```

### Q1. D'après le message de `hello-world`, quelles sont les quatre étapes réalisées par Docker ?

1. Le **client Docker** (la commande `docker`) contacte le **démon Docker**.
2. Le démon **télécharge l'image** `hello-world` depuis le registre Docker Hub (car elle n'est pas présente en local).
3. Le démon **crée un conteneur** à partir de cette image, et lance l'exécutable qui produit le message affiché.
4. Le démon **renvoie la sortie** de ce programme au client Docker, qui l'affiche dans le terminal.

### Q2. Le conteneur `web` et la VM ont-ils la même distribution ? Le même noyau ?

- **Distribution : non.** L'image `nginx` est construite sur une base Debian (`/etc/os-release` du conteneur indique Debian), alors que la VM est sous Ubuntu. Le conteneur embarque ses propres bibliothèques et son propre userland, ce qui permet de faire tourner une autre « distribution » que celle de l'hôte.
- **Noyau : oui, le même.** `docker exec web uname -r` et `uname -r` affichent exactement la même version. Un conteneur n'a pas de noyau propre : il **partage le noyau de l'hôte**. Il est isolé par les mécanismes du noyau Linux (*namespaces* pour l'isolation, *cgroups* pour la limitation des ressources). Selon le cours, c'est la différence fondamentale avec une VM, qui virtualise le matériel et embarque son propre noyau.

### Q3. Les processus Nginx du conteneur apparaissent-ils dans `ps aux` sur la VM ? Qu'est-ce que cela montre ?

**Oui.** `ps aux | grep "nginx: master"` retourne le processus maître de Nginx avec un PID de la VM (et `docker top web` affiche les mêmes PID). Cela montre qu'un conteneur **n'est pas une machine virtuelle, mais un simple processus (ou groupe de processus) Linux ordinaire**, simplement isolé : à l'intérieur du conteneur, Nginx se voit avec le PID 1 ; vu de l'hôte, c'est un processus comme un autre avec un autre PID. Il n'y a ni noyau invité ni démarrage d'un système complet, d'où la légèreté et la rapidité.

### Q4. Chemin d'une requête du navigateur de l'hôte jusqu'à Nginx

1. Le navigateur Windows envoie la requête à `127.0.0.1:8080` (hôte).
2. **Première redirection (VirtualBox, NAT)** : la règle `web,tcp,127.0.0.1,8080,,8080` transmet le trafic du port 8080 de l'hôte vers le port 8080 de la VM `ubuntu-srv`.
3. Dans la VM, le trafic arrive sur le port 8080, que Docker écoute grâce à `-p 8080:80`.
4. **Deuxième redirection (Docker)** : Docker (règle NAT iptables / `docker-proxy`) redirige le port 8080 de la VM vers le port 80 du conteneur `web`, sur le réseau virtuel `docker0` (adresse du type `172.17.0.x`).
5. Nginx, qui écoute sur le port 80 dans le conteneur, répond, et la réponse refait le chemin inverse.

Résumé : `hôte :8080 → (VirtualBox) → VM :8080 → (Docker) → conteneur :80`.

### Q5. Pourquoi la page modifiée a-t-elle disparu après `docker rm` ? Que change `-v` ?

Un conteneur est créé à partir d'une image dont les couches sont **en lecture seule**. Docker ajoute par-dessus une fine **couche d'écriture propre au conteneur** : la page modifiée y a été écrite. Quand on fait `docker rm`, cette couche est détruite avec le conteneur ; le nouveau conteneur repart de l'image d'origine, donc avec la page « Welcome to nginx! ».

L'option `-v ~/site:/usr/share/nginx/html:ro` **monte un dossier de la VM dans le conteneur** (volume de type *bind mount*). Les données vivent alors **en dehors du conteneur**, dans le système de fichiers de la VM : elles survivent à la suppression et à la recréation du conteneur. Le suffixe `:ro` rend le montage en lecture seule côté conteneur, ce qui limite les risques.

---

## Partie 2 — Comparer conteneur et VM

### Commandes lancées

```bash
docker pull alpine
docker images
time docker run --rm alpine echo "Bonjour depuis un conteneur"
time docker run --rm alpine echo "Bonjour depuis un conteneur"
systemd-analyze
free -h
docker run --rm alpine cat /etc/os-release
```

```text
etudiant@ubuntu-srv:~$ docker images
                                            i Info →   U  In Use
IMAGE           ID             DISK USAGE   CONTENT SIZE   EXTRA
alpine:latest   be2656061b53         13MB         3.94MB
hello-world:latest
                c567b231f656       25.9kB         9.49kB    U
nginx:latest    600f7cd45e27        243MB         66.8MB    U

etudiant@ubuntu-srv:~$ time docker run --rm alpine echo "Bonjour depuis un conteneur"
Bonjour depuis un conteneur

real    0m1,816s
user    0m0,027s
sys     0m0,090s

etudiant@ubuntu-srv:~$ time docker run --rm alpine echo "Bonjour depuis un conteneur"
Bonjour depuis un conteneur

real    0m1,611s
user    0m0,034s
sys     0m0,078s
etudiant@ubuntu-srv:~$ systemd-analyze
Startup finished in 7.227s (kernel) + 20.142s (initrd) + 42.156s (userspace) = 1min 9.526s
graphical.target reached after 42.148s in userspace.
etudiant@ubuntu-srv:~$ free -h
               total        used        free      shared  buff/cache   available
Mem:           1,6Gi       493Mi       118Mi       1,2Mi       1,2Gi       1,1Gi
Swap:          2,0Gi        24Ki       2,0Gi
etudiant@ubuntu-srv:~$ docker run --rm alpine cat /etc/os-release
NAME="Alpine Linux"
ID=alpine
VERSION_ID=3.24.3
PRETTY_NAME="Alpine Linux v3.24"
HOME_URL="https://alpinelinux.org/"
BUG_REPORT_URL="https://gitlab.alpinelinux.org/alpine/aports/-/issues"
```

### Tableau comparatif

> Les valeurs entre parenthèses sont des **ordres de grandeur typiques** : remplacez-les par vos mesures.

| Mesure | VM ubuntu-srv | Conteneur nginx | Conteneur alpine |
| --- | --- | --- | --- |
| Taille sur disque | 6.23 Go |  243MB | 13MB |
| Temps de démarrage |  1min 9.526s| — | 0.034s|
| Mémoire utilisée | 463Mi | 3.395MiB | — |
| Système | Ubuntu complet + noyau | Debian (userland seulement, noyau partagé) | Alpine Linux (userland minimal, noyau partagé) |

### Q1. Dans quel rapport le conteneur est-il plus léger et plus rapide que la VM ? Vos mesures confirment-elles le tableau du cours ?

À partir de mesures typiques (à adapter aux vôtres) :

- **Taille :** une VM Ubuntu Server pèse plusieurs Go ; `nginx` environ 190 Mo (≈ ×20) et `alpine` environ 8 Mo (≈ ×500). Le gain va donc de **×20 à ×500** selon l'image.
- **Démarrage :** la VM met quelques dizaines de secondes (démarrage du noyau, de systemd, des services), contre moins d'une seconde pour un conteneur : **environ ×30 à ×100** plus rapide.
- **Mémoire :** la VM consomme quelques centaines de Mo au repos (noyau + services), `nginx` quelques Mo : **environ ×50**.

Les mesures confirment le tableau du cours : le conteneur est **de l'ordre de 10 à 100 fois plus léger et plus rapide** qu'une VM, parce qu'il ne démarre pas de système d'exploitation complet et ne réserve pas de mémoire pour un noyau invité. *(À ajuster : indiquez ici si vos chiffres s'écartent de ceux du cours, et pourquoi.)*

### Q2. Pourquoi le premier `time docker run` est-il plus long que le second ?

Au premier lancement, les fichiers nécessaires (binaire `runc`, bibliothèques, couches de l'image `alpine`, métadonnées) ne sont **pas encore dans le cache disque du noyau (*page cache*)** : ils sont lus depuis le disque, plus lent. Docker doit aussi préparer le système de fichiers du conteneur (montage *overlay*) et la configuration réseau pour la première fois. Au deuxième lancement, tout est déjà en mémoire : le démarrage est nettement plus rapide. C'est pour cela que le TP demande de garder la deuxième mesure, qui reflète le fonctionnement « normal ». (Si l'image n'avait pas déjà été téléchargée par `docker pull`, il faudrait ajouter le temps de téléchargement.)

### Q3. Citez un cas où vous choisiriez quand même une VM.

Exemples valables (en choisir un ou deux) :
- **Isolation forte entre locataires** (cloud public, clients différents) : une VM isole au niveau matériel, un conteneur partage le noyau, donc une faille du noyau peut affecter tous les conteneurs.
- **Système d'exploitation ou noyau différent de l'hôte** : exécuter Windows sur un serveur Linux, ou un noyau spécifique avec ses modules.
- **Application historique (*legacy*)** qui suppose un système complet, des services multiples ou des droits administrateur sur toute la machine.
- **Exigences de conformité ou de sécurité** imposant une séparation stricte.

---

## Partie 3 — Conteneuriser l'API fil rouge

### Fichiers du projet (`~/api`)

`app.py` (avec la route `/version` ajoutée à l'étape 6) :

```python
import os
import socket
from flask import Flask, jsonify

app = Flask(__name__)

@app.get("/")
def accueil():
    return jsonify(
        message="API fil rouge",
        binome=os.environ.get("BINOME", "inconnu"),
        conteneur=socket.gethostname(),
    )

@app.get("/sante")
def sante():
    return jsonify(statut="ok")

@app.get("/version")
def version():
    return jsonify(version="1.1")

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```

`requirements.txt` :

```text
flask>=3,<4
```

`Dockerfile` :

```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
EXPOSE 5000
CMD ["python", "app.py"]
```

`.dockerignore` :

```text
__pycache__/
*.pyc
.git
```

### Commandes lancées

```bash
cd ~/api
docker build -t api:1.0 .
docker images api
docker history api:1.0

docker run -d --name api -p 5000:5000 -e BINOME="NOM1-NOM2" api:1.0
curl -s http://localhost:5000
curl -s http://localhost:5000/sante
docker logs api

for p in 5001 5002 5003; do docker run -d --name api-$p -p $p:5000 -e BINOME="NOM1-NOM2" api:1.0; done
for p in 5001 5002 5003; do curl -s http://localhost:$p; echo; done
docker rm -f api-5001 api-5002 api-5003

# après ajout de la route /version dans app.py
docker build -t api:1.1 .
docker rm -f api && docker run -d --name api -p 5000:5000 api:1.1
curl -s http://localhost:5000/version
```

```text
etudiant@ubuntu-srv:~$ curl -s http://localhost:5000
{"binome":"NOM1-NOM2","conteneur":"df703da597b0","message":"API fil rouge"}
etudiant@ubuntu-srv:~$ curl -s http://localhost:5000/sante
{"statut":"ok"}
etudiant@ubuntu-srv:~$ docker logs api
 * Serving Flask app 'app'
 * Debug mode: off
WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
 * Running on all addresses (0.0.0.0)
 * Running on http://127.0.0.1:5000
 * Running on http://172.17.0.3:5000
Press CTRL+C to quit
172.17.0.1 - - [30/Oct/2026 12:42:48] "GET / HTTP/1.1" 200 -
172.17.0.1 - - [30/Oct/2026 12:42:56] "GET /sante HTTP/1.1" 200 -
```

### Q1. Taille de `api:1.0`, nombre de couches, et couches issues du Dockerfile

- **Taille de l'image** : 198MB
- **Nombre de couches** affichées par `docker history` :16 couches
- **Couches issues de notre Dockerfile** (les plus récentes, en haut de `docker history`) :

| Instruction | Effet | Taille typique |
| --- | --- | --- |
| `WORKDIR /app` | crée et se place dans `/app` |8.19 kB |
| `COPY requirements.txt .` | copie le fichier des dépendances | qu 12.3 kB |
| `RUN pip install ...` | installe Flask et ses dépendances | 15.3 MB |
| `COPY . .` | copie le code de l'API | 20.5 kB |
| `EXPOSE 5000` | documente le port |0 B — les instructions de configuration réseau ou d'exécution ne pèsent rien|
| `CMD ["python", "app.py"]` | commande de démarrage | 0 B (métadonnée) |

`FROM python:3.12-slim` n'ajoute pas de couche propre : il importe toutes les couches de l'image de base.

### Q2. Pourquoi copier `requirements.txt` et installer les dépendances **avant** de copier le code ?

Docker construit l'image **couche par couche** et réutilise (cache) une couche tant que son instruction **et tout ce qui la précède** n'ont pas changé. Dès qu'une couche est invalidée, toutes les suivantes sont reconstruites.

À l'étape 6, on a seulement modifié `app.py` (ajout de `/version`). Le build de `api:1.1` montre alors :
- `FROM`, `WORKDIR`, `COPY requirements.txt .` et **`RUN pip install ...` : `CACHED`** (rien n'a changé, `requirements.txt` est identique) ;
- seule la couche **`COPY . .`** est reconstruite, car le code a changé, ainsi que celles qui la suivent.

Résultat : le build prend une ou deux secondes au lieu de retélécharger et réinstaller Flask. Si on avait fait `COPY . .` **avant** `pip install`, chaque modification du code aurait invalidé le cache et relancé l'installation complète des dépendances à chaque build. On place donc ce qui change **rarement** (dépendances) avant ce qui change **souvent** (code).

### Q3. Que renvoie le champ `conteneur` pour les trois copies ? Quelle caractéristique du cloud cela illustre-t-il ?

Chaque copie renvoie une valeur **différente** : le champ `conteneur` contient `socket.gethostname()`, c'est-à-dire l'**identifiant du conteneur** (un identifiant hexadécimal de 12 caractères, par exemple `5c1d0e7a9b2f`), unique pour chaque conteneur. Les trois réponses sont identiques pour le reste (`message`, `binome`), car elles proviennent de la **même image**.

Cette expérience illustre l'**élasticité rapide** (*rapid elasticity*) et la **scalabilité horizontale** du cloud : à partir d'une seule image, on peut démarrer en quelques secondes autant d'instances identiques que nécessaire, puis les supprimer, ce qui permet d'adapter les ressources à la charge. Elle illustre aussi la **mutualisation des ressources** (*resource pooling*) : plusieurs instances partagent le même hôte.

### Q4. Pourquoi passer le nom du binôme par une variable d'environnement (`-e`) plutôt que dans le code ? Quel lien avec les secrets ?

- **Séparer la configuration du code** (principe des applications « 12 facteurs ») : la **même image** peut être utilisée dans plusieurs environnements (test, production, autre binôme) en changeant seulement la variable au lancement, sans reconstruire l'image.
- **Ne pas figer de valeurs dans l'image** : tout ce qui est écrit dans le code ou copié dans l'image est lisible par quiconque récupère l'image et ses couches.

**Lien avec les secrets** : les mots de passe, clés d'API ou jetons ne doivent jamais être écrits dans le code, dans un `Dockerfile` ou dans un fichier `.env` copié dans l'image, ni versionnés dans Git. On les fournit au moment de l'exécution (variables d'environnement, ou mieux : *secrets* Docker / gestionnaire de secrets du cloud). Remarque : une variable d'environnement reste visible avec `docker inspect`, donc pour des secrets sensibles en production, on utilise un mécanisme dédié.

---

## Partie 4 — Publier l'image et nettoyer

**Registre choisi :** `À REMPLACER` (Docker Hub ou GHCR)

### Commandes lancées

```bash

 #Option B — GHCR
 docker login ghcr.io -u <compte>
 docker tag api:1.1 ghcr.io/<compte>/api-fil-rouge:1.1
 docker push  ghcr.io/marcarthurndri01-art/api-fil-rouge:1.1

# Vérification depuis le registre
docker rm -f api
docker rmi <compte>/api-fil-rouge:1.1 api:1.1
docker run -d --name api -p 5000:5000  ghcr.io/marcarthurndri01-art/api-fil-rouge:1.1
curl -s http://localhost:5000/version

# Nettoyage
docker rm -f $(docker ps -aq)
docker system df
docker image prune -a -f
docker system df
```



### Q1. Adresse complète de l'image publiée. Publique ou privée ?

- **Adresse** :  ghcr.io/marcarthurndri01-art/api-fil-rouge:1.1` (GHCR).
- **Visibilité** : `À REMPLACER (publique / privée)`. Sur Docker Hub, un dépôt créé par `push` est public par défaut sur un compte gratuit ; sur GHCR, un paquet est privé par défaut tant qu'on ne change pas sa visibilité. Une image publique est lisible par tous : elle ne doit contenir aucun secret.

### Q2. Pourquoi étiqueter l'image avec un numéro de version (`1.1`) plutôt que d'utiliser toujours `latest` ?

- **Reproductibilité** : `latest` est une étiquette **mobile** : elle pointe vers l'image la plus récemment poussée, donc le même nom peut désigner des contenus différents selon le moment. Avec `1.1`, on sait exactement quel code tourne.
- **Retour arrière (*rollback*)** : en cas de problème avec la `1.2`, on redéploie simplement la `1.1`.
- **Traçabilité et cohérence** : toutes les machines d'un déploiement utilisent la même version, et on peut savoir quelle version est en production ; avec `latest`, deux serveurs peuvent exécuter des versions différentes sans qu'on s'en aperçoive.
- **Maîtrise des mises à jour** : on choisit quand migrer, au lieu de subir une mise à jour implicite.

### Q3. Quel espace disque `docker image prune` a-t-il libéré ? Pourquoi nettoyer aussi les registres dans le cloud ?

- **Espace libéré** : 40.93MB
- **Pourquoi nettoyer les registres** : dans le cloud, le **stockage est facturé** (au Go par mois) et chaque nouvelle version s'accumule ; les anciennes images inutilisées coûtent donc de l'argent pour rien. Un registre trop chargé ralentit aussi la gestion et augmente la **surface d'attaque** : d'anciennes images contiennent des **vulnérabilités connues** (et parfois des secrets oubliés) qu'on pourrait déployer par erreur. On définit donc des **politiques de rétention** (par exemple ne conserver que les N dernières versions ou supprimer les images de plus de X jours).

---

