# TP 2 : Deux machines virtuelles en réseau avec VirtualBox

**Cours :** Introduction au Cloud Computing, L3 SI
**Membres :** 
MAGASSOUBA CHEICK AHMED
KOUKO JOSHUA EZECKIEL
N'DRI N'DOUA DENIS MARC-ARTHUR
*Dans les sorties, l'utilisateur des VM est `test` (au lieu de `etudiant` dans l'énoncé).*

---

# Partie 1 : Créer les VM par clonage

## 1.1 Cloner l'image et créer le réseau

**Étape 1 : éteindre `ubuntu-srv` et vérifier qu'aucune VM ne tourne**

```powershell
PS> $env:Path += ";C:\Program Files\Oracle\VirtualBox"
PS> VBoxManage controlvm ubuntu-srv acpipowerbutton
VBoxManage.exe: error: Machine 'ubuntu-srv' is not currently running.
PS> VBoxManage list runningvms
```
*(L'erreur est attendue : `ubuntu-srv` était déjà éteinte. `list runningvms` est vide, donc aucune VM n'est démarrée.)*

**Étape 2 : cloner `web` (chronométré) et `client`, puis 1 vCPU et 1 Go de RAM**

```powershell
PS> Measure-Command { VBoxManage clonevm ubuntu-srv --name web --register } | Select-Object TotalSeconds
0%...10%...20%...30%...40%...50%...60%...70%...80%...90%...100%

TotalSeconds
------------
   19,393816

PS> VBoxManage modifyvm web    --cpus 1 --memory 1024
PS> Measure-Command { VBoxManage clonevm ubuntu-srv --name client --register } | Select-Object TotalSeconds
0%...10%...20%...30%...40%...50%...60%...70%...80%...90%...100%

TotalSeconds
------------
  18,0781209

PS> VBoxManage modifyvm client --cpus 1 --memory 1024
```

**Étape 3 : réseau NAT `tp-cloud` sans DHCP et ses trois redirections de port**

```powershell
PS> VBoxManage natnetwork add --netname tp-cloud --network "10.10.0.0/24" --enable --dhcp off
PS> VBoxManage natnetwork modify --netname tp-cloud --port-forward-4 "ssh - web:tcp:[127.0.0.1]:2222:[10.10.0.10]:22"
PS> VBoxManage natnetwork modify --netname tp-cloud --port-forward-4 "ssh - client:tcp:[127.0.0.1]:2223:[10.10.0.20]:22"
PS> VBoxManage natnetwork modify --netname tp-cloud --port-forward-4 "http - web:tcp:[127.0.0.1]:8080:[10.10.0.10]:80"
PS> VBoxManage natnetwork list
NAT Networks:

Name:         tp-cloud
Enabled:      Yes
Network:      10.10.0.0/24
Gateway:      10.10.0.1
DHCP Server:  No
IPv6:         No
IPv6 Prefix:  fd17:625c:f037:2::/64
IPv6 Default: No
Port-forwarding (ipv4)
        http - web:tcp:[127.0.0.1]:8080:[10.10.0.10]:80
        ssh - client:tcp:[127.0.0.1]:2223:[10.10.0.20]:22
        ssh - web:tcp:[127.0.0.1]:2222:[10.10.0.10]:22
loopback mappings (ipv4)
        127.0.0.1=2

1 network found
```

**Étape 4 : brancher la carte réseau 1 des clones sur `tp-cloud`**

```powershell
PS> VBoxManage modifyvm web    --nic1 natnetwork --nat-network1 tp-cloud
PS> VBoxManage modifyvm client --nic1 natnetwork --nat-network1 tp-cloud
PS> VBoxManage showvminfo web | Select-String "Memory size|Number of CPUs|NIC 1"

Memory size:                 1024MB
Number of CPUs:              1
NIC 1:                       MAC: 080027172F60, Attachment:
NAT Network 'tp-cloud', Cable connected: on, Trace: off (file:
none), Type: 82540EM, Reported speed: 0 Mbps, Boot priority:
0, Promisc Policy: deny, Bandwidth group: none
```

## 1.2 Personnaliser chaque clone

Les étapes 5 à 7 (nom d'hôte, `machine-id`, clés SSH, fichier netplan, redémarrage) ont été faites sur la console de chaque VM. Leur résultat est visible à l'étape 8 : nom d'hôte `web` / `client`, adresse `10.10.0.20/24` sur `client`, et empreintes SSH différentes.

**Étape 8 : connexion SSH aux deux VM**

```powershell
PS> ssh-keygen -R "[127.0.0.1]:2222"
# Host [127.0.0.1]:2222 found: line 3
# Host [127.0.0.1]:2222 found: line 4
# Host [127.0.0.1]:2222 found: line 5
C:\Users\Cinem/.ssh/known_hosts updated.
Original contents retained as C:\Users\Cinem/.ssh/known_hosts.old
PS> ssh-keygen -R "[127.0.0.1]:2223"
Host [127.0.0.1]:2223 not found in C:\\Users\\Cinem/.ssh/known_hosts

PS> ssh -p 2222 test@127.0.0.1 "hostnamectl; lsb_release -a; uname -r"
The authenticity of host '[127.0.0.1]:2222 ([127.0.0.1]:2222)' can't be established.
ED25519 key fingerprint is SHA256:IQhgG8cq5At7KbCd9QKaieCblZ3RhyTn+n+P0w1qHFk.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '[127.0.0.1]:2222' (ED25519) to the list of known hosts.
test@127.0.0.1's password:
 Static hostname: web
       Icon name: computer-vm
         Chassis: vm 🖴
      Machine ID: fbeee49c9dba42d795dd2619005c4db1
         Boot ID: 8bea7946349b45afae7ddc3216b44b76
  Virtualization: oracle
Operating System: Ubuntu 26.04.1 LTS
          Kernel: Linux 7.0.0-42-generic
    Architecture: x86-64
 Hardware Vendor: innotek GmbH
  Hardware Model: VirtualBox
Hardware Version: 1.2
Firmware Version: VirtualBox
   Firmware Date: Fri 2006-12-01
    Firmware Age: 19y 10month 4w 1d
Distributor ID: Ubuntu
Description:    Ubuntu 26.04.1 LTS
Release:        26.04
Codename:       resolute
7.0.0-42-generic

PS> ssh -p 2223 test@127.0.0.1 "hostname; ip -4 -br addr"
The authenticity of host '[127.0.0.1]:2223 ([127.0.0.1]:2223)' can't be established.
ED25519 key fingerprint is SHA256:brXPUnTgbit+g1ew+SLfXFfS8k+8fqQJx8FEJeZVhBA.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '[127.0.0.1]:2223' (ED25519) to the list of known hosts.
test@127.0.0.1's password:
Permission denied, please try again.
test@127.0.0.1's password:
client
lo               UNKNOWN        127.0.0.1/8
enp0s3           UP             10.10.0.20/24
```

## Réponses aux questions de la partie 1

**Q1. Durée du clonage de `web`.**
Le clonage de `web` a pris **19,39 s**. L'installation d'Ubuntu au TP précédent avait pris bien plus longtemps. Pour un vrai serveur physique, il faut compter des jours, voire des semaines (commande, livraison, installation). Avec le clonage, on a une machine prête en quelques secondes.

**Q2. Pourquoi changer le nom d'hôte, le `machine-id` et les clés SSH ?**
Un clone est une copie exacte de la VM d'origine. Il faut donc lui donner sa propre identité :
- le **nom d'hôte** sert à reconnaître la machine sur le réseau ;
- le **`machine-id`** est son numéro d'identification unique ;
- les **clés SSH** sont la carte d'identité du serveur.

Si 50 serveurs ont les mêmes clés SSH, il suffit à un pirate de voler la clé d'un seul pour se faire passer pour n'importe lequel des 50.

**Q3. Version, noyau, vCPU, RAM, et équivalent cloud.**
- Ubuntu **26.04.1 LTS**
- Noyau **7.0.0-42-generic**
- **1 vCPU**
- **1 Go de RAM** (1024 Mo configurés, 956 Mi visibles dans la VM)

`ubuntu-srv` joue le rôle de l'**image de départ**, comme une **AMI** sur AWS : on en lance autant d'instances qu'on veut.

---

# Partie 2 : Relier les VM en réseau

**Étape 1 : connectivité depuis `client`**

```powershell
PS> ssh -p 2223 test@127.0.0.1
```
```console
test@client:~$ ping -c 4 10.10.0.10
PING 10.10.0.10 (10.10.0.10) 56(84) bytes of data.
64 bytes from 10.10.0.10: icmp_seq=1 ttl=64 time=4.32 ms
64 bytes from 10.10.0.10: icmp_seq=2 ttl=64 time=6.29 ms
64 bytes from 10.10.0.10: icmp_seq=3 ttl=64 time=1.96 ms
64 bytes from 10.10.0.10: icmp_seq=4 ttl=64 time=2.34 ms

--- 10.10.0.10 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3016ms
rtt min/avg/max/mdev = 1.961/3.730/6.293/1.730 ms
test@client:~$ ping -c 2 10.10.0.1
PING 10.10.0.1 (10.10.0.1) 56(84) bytes of data.
64 bytes from 10.10.0.1: icmp_seq=1 ttl=64 time=1.63 ms
64 bytes from 10.10.0.1: icmp_seq=2 ttl=64 time=0.833 ms

--- 10.10.0.1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1001ms
rtt min/avg/max/mdev = 0.833/1.231/1.630/0.398 ms
test@client:~$ ping -c 2 1.1.1.1
PING 1.1.1.1 (1.1.1.1) 56(84) bytes of data.
64 bytes from 1.1.1.1: icmp_seq=1 ttl=64 time=15.9 ms
64 bytes from 1.1.1.1: icmp_seq=2 ttl=64 time=16.3 ms

--- 1.1.1.1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1001ms
rtt min/avg/max/mdev = 15.944/16.140/16.337/0.196 ms
test@client:~$ ip route
default via 10.10.0.1 dev enp0s3 proto static
10.10.0.0/24 dev enp0s3 proto kernel scope link src 10.10.0.20
test@client:~$ resolvectl status | grep "DNS Servers"
       DNS Servers: 10.10.0.1 1.1.1.1
```

**Étape 2 : nom de `web` dans `/etc/hosts`**

```console
test@client:~$ echo "10.10.0.10 web" | sudo tee -a /etc/hosts
[sudo: authenticate] Password:
10.10.0.10 web
test@client:~$ ping -c 2 web
PING web (10.10.0.10) 56(84) bytes of data.
64 bytes from web (10.10.0.10): icmp_seq=1 ttl=64 time=5.13 ms
64 bytes from web (10.10.0.10): icmp_seq=2 ttl=64 time=5.19 ms

--- web ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 5.129/5.161/5.194/0.032 ms
test@client:~$ exit
logout
Connection to 127.0.0.1 closed.
```

**Étape 3 : ping depuis Windows**

```powershell
PS> ping -n 2 10.10.0.1

Envoi d'une requête 'Ping'  10.10.0.1 avec 32 octets de données :
Délai d'attente de la demande dépassé.
Délai d'attente de la demande dépassé.

Statistiques Ping pour 10.10.0.1:
    Paquets : envoyés = 2, reçus = 0, perdus = 2 (perte 100%),
```

## Réponses aux questions de la partie 2

**Q1. Même sous-réseau ?**
Oui. `web` est en 10.10.0.10 et `client` en 10.10.0.20, avec le masque /24. Cela veut dire que les trois premiers nombres (**10.10.0**) doivent être identiques pour être dans le même réseau, ce qui est le cas.

**Q2. Passerelle par défaut de `client`.**
C'est **10.10.0.1**. C'est la porte de sortie du réseau : tout ce qui part vers Internet passe par elle. Dans le cloud, elle correspond à l'**Internet Gateway / NAT Gateway** d'un VPC (AWS), ou à la route vers Internet d'un VNet (Azure).

**Q3. Temps de réponse moyen `client` vers `web`.**
**3,73 ms** en moyenne. C'est très rapide car les deux VM sont sur le même ordinateur : il n'y a pas de vrai câble ni de vrai routeur, tout se passe en mémoire.

**Q4. Ping depuis Windows, et SSH.**
Non, le ping échoue (**100 % de perte**, testé vers 10.10.0.1). Le réseau des VM est privé et Windows ne le voit pas. Le SSH marche quand même grâce à la **redirection de port** : quand on se connecte à `127.0.0.1:2222`, VirtualBox envoie la connexion vers `web` sur le port 22.

---

# Partie 3 : Publier une page web et la protéger

## 3.1 Installer Nginx

**Étape 1 : installation et vérification sur `web`**

```console
test@web:~$ sudo apt update && sudo apt install -y nginx
[sudo: authenticate] Password:
Atteint : 1 http://ci.archive.ubuntu.com/ubuntu resolute InRelease
Réception de : 2 http://ci.archive.ubuntu.com/ubuntu resolute-updates InRelease [137 kB]
Réception de : 3 http://security.ubuntu.com/ubuntu resolute-security InRelease [137 kB]
Réception de : 4 http://ci.archive.ubuntu.com/ubuntu resolute-backports InRelease [137 kB]
Réception de : 5 http://security.ubuntu.com/ubuntu resolute-security/main amd64 Components [46,7 kB]
Réception de : 6 http://ci.archive.ubuntu.com/ubuntu resolute-updates/main amd64 Packages [803 kB]
Réception de : 7 http://security.ubuntu.com/ubuntu resolute-security/universe amd64 Components [55,5 kB]
Réception de : 8 http://ci.archive.ubuntu.com/ubuntu resolute-updates/main amd64 Components [110 kB]
Réception de : 9 http://ci.archive.ubuntu.com/ubuntu resolute-updates/universe amd64 Components [201 kB]
Réception de : 10 http://ci.archive.ubuntu.com/ubuntu resolute-backports/universe amd64 Components [1 056 B]
1 629 ko réceptionnés en 7s (222 ko/s)

40 paquets peuvent être mis à jour. Exécutez « apt list --upgradable » pour les voir.
Les paquets suivants ont été installés automatiquement et ne sont plus nécessaires :
  linux-headers-7.0.0-30
  linux-headers-7.0.0-30-generic
  linux-image-unsigned-7.0.0-30-generic
  linux-main-modules-zfs-7.0.0-30-generic
  linux-modules-7.0.0-30-generic
  linux-tools-7.0.0-30
  linux-tools-7.0.0-30-generic
Veuillez utiliser « sudo apt autoremove » pour les supprimer.

Installation de :
  nginx

Installation de dépendances :
  nginx-common

Paquets suggérés :
  fcgiwrap  nginx-doc  ssl-cert

Sommaire :
  Mise à niveau de : 0. Installation de : 2, Supprimé : 0. Non mis à jour : 40
Taille du téléchargement : 656 ko
  Espace nécessaire : 1 864 ko / 5 674 Mo disponible

Réception de : 1 http://ci.archive.ubuntu.com/ubuntu resolute-updates/main amd64 nginx-common all 1.28.3-2ubuntu1.12 [38,2 kB]
Réception de : 2 http://ci.archive.ubuntu.com/ubuntu resolute-updates/main amd64 nginx amd64 1.28.3-2ubuntu1.12 [618 kB]
656 ko réceptionnés en 2s (288 ko/s)
Préconfiguration des paquets...
Sélection du paquet nginx-common précédemment désélectionné.
(Lecture de la base de données... 139564 fichiers et répertoires déjà installés.)
Préparation du dépaquetage de .../nginx-common_1.28.3-2ubuntu1.12_all.deb ...
Dépaquetage de nginx-common (1.28.3-2ubuntu1.12) ...
Sélection du paquet nginx précédemment désélectionné.
Préparation du dépaquetage de .../nginx_1.28.3-2ubuntu1.12_amd64.deb ...
Dépaquetage de nginx (1.28.3-2ubuntu1.12) ...
Paramétrage de nginx-common (1.28.3-2ubuntu1.12) ...
Created symlink '/etc/systemd/system/multi-user.target.wants/nginx.service' → '/usr/lib/systemd/system/nginx.service'.
Paramétrage de nginx (1.28.3-2ubuntu1.12) ...
 * Upgrading binary nginx                                [ OK ]
Traitement des actions différées (« triggers ») pour man-db (2.13.1-1build1) ...
Traitement des actions différées (« triggers ») pour ufw (0.36.2-9build1) ...
Scanning processes...
Scanning linux images...

Running kernel seems to be up-to-date.

No services need to be restarted.

No containers need to be restarted.

No user sessions are running outdated binaries.

No VM guests are running outdated hypervisor (qemu) binaries on
 this host.
test@web:~$ systemctl status nginx --no-pager
● nginx.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/usr/lib/systemd/system/nginx.service; enabled; preset: enabled)
     Active: active (running) since Fri 2026-10-30 07:59:36 UTC; 37s ago
 Invocation: 8fba6680a3c84ba0aa5ca09dc63a9ab3
       Docs: man:nginx(8)
    Process: 2162 ExecStartPre=/usr/sbin/nginx -t -q -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
    Process: 2164 ExecStart=/usr/sbin/nginx -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
   Main PID: 2192 (nginx)
      Tasks: 2 (limit: 1059)
     Memory: 2.3M (peak: 5.1M)
        CPU: 249ms
     CGroup: /system.slice/nginx.service
             ├─2192 "nginx: master process /usr/sbin/nginx -g d…
             └─2194 "nginx: worker process"

oct. 30 07:59:35 web systemd[1]: Starting nginx.service - A …...
oct. 30 07:59:36 web systemd[1]: Started nginx.service - A h…er.
Hint: Some lines were ellipsized, use -l to show in full.
test@web:~$ echo "<h1>Binôme NOM1 - NOM2 : ma première VM en réseau</h1>" | sudo tee /var/www/html/index.html
<h1>Binôme NOM1 - NOM2 : ma première VM en réseau</h1>
test@web:~$ exit
logout
Connection to 127.0.0.1 closed.
```

**Étape 2 : page affichée depuis `client`**

```powershell
PS> ssh -p 2223 test@127.0.0.1 "curl -s http://web"
test@127.0.0.1's password:
<h1>Binôme NOM1 - NOM2 : ma première VM en réseau</h1>
```

## 3.2 Fermer puis ouvrir le port 80 avec ufw

**Étapes 3 et 5 : pare-feu sur `web`** *(session root ouverte avec `sudo su`)*

```console
root@web:/home/test# ufw allow OpenSSH
Rules updated
Rules updated (v6)
root@web:/home/test# ufw enable
Command may disrupt existing ssh connections. Proceed with operation (y|n)? y
Firewall is active and enabled on system startup
root@web:/home/test# ufw status verbose
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), disabled (routed)
New profiles: skip

To                         Action      From
--                         ------      ----
22/tcp (OpenSSH)           ALLOW IN    Anywhere
22/tcp (OpenSSH (v6))      ALLOW IN    Anywhere (v6)

root@web:/home/test# ufw allow 'Nginx HTTP'
Rule added
Rule added (v6)
root@web:/home/test# ufw status numbered
Status: active

     To                         Action      From
     --                         ------      ----
[ 1] OpenSSH                    ALLOW IN    Anywhere
[ 2] Nginx HTTP                 ALLOW IN    Anywhere
[ 3] OpenSSH (v6)               ALLOW IN    Anywhere (v6)
[ 4] Nginx HTTP (v6)            ALLOW IN    Anywhere (v6)

root@web:/home/test# ss -tlnp
State   Recv-Q  Send-Q   Local Address:Port   Peer Address:Port Process
LISTEN  0       4096     127.0.0.53%lo:53          0.0.0.0:*     users:(("systemd-resolve",pid=565,fd=17))
LISTEN  0       4096        127.0.0.54:53          0.0.0.0:*     users:(("systemd-resolve",pid=565,fd=19))
LISTEN  0       511            0.0.0.0:80          0.0.0.0:*     users:(("nginx",pid=2194,fd=5),("nginx",pid=2192,fd=5))
LISTEN  0       4096           0.0.0.0:22          0.0.0.0:*     users:(("sshd",pid=1190,fd=3),("systemd",pid=1,fd=199))
LISTEN  0       511               [::]:80             [::]:*     users:(("nginx",pid=2194,fd=6),("nginx",pid=2192,fd=6))
LISTEN  0       4096              [::]:22             [::]:*     users:(("sshd",pid=1190,fd=4),("systemd",pid=1,fd=200))
```

**Étape 4 (test avec le port 80 fermé) et test final après ouverture du port 80 :** 

```powershell
PS> ssh -p 2223 test@127.0.0.1 "curl -m 5 http://web; echo code retour: `$?"
(à compléter)
PS> ssh -p 2223 test@127.0.0.1 "curl -s http://web"
(à compléter)
```

## Réponses aux questions de la partie 3

**Q1. `curl` avec le port 80 fermé.**
`curl` attend 5 secondes puis abandonne avec **`Connection timed out`** et le code retour **28** . Le navigateur reste bloqué car le pare-feu **ne répond rien du tout** : il ignore la demande, donc le navigateur attend longtemps avant d'abandonner, au lieu d'afficher tout de suite une erreur.

**Q2. Port et adresses d'écoute de Nginx.**
Nginx écoute sur le **port 80**, sur `0.0.0.0` (toutes les adresses IPv4 de la machine) et sur `[::]` (toutes les adresses IPv6), d'après `ss -tlnp`.

**Q3. ufw et groupe de sécurité du cloud.**
Ressemblances :
1. Les deux **bloquent tout par défaut** et on ouvre seulement les ports nécessaires (ici 22 et 80).
2. Les deux se modifient facilement, **sans toucher à l'application** : on ouvre ou ferme un port en une règle.

Différence : le groupe de sécurité est **en dehors de la VM** (géré par le fournisseur), alors qu'ufw est **dans la VM** : quelqu'un qui devient root dans la VM peut le désactiver.

**Q4. Qui gère quoi ?**
- **VirtualBox** a fourni le matériel virtuel (CPU, RAM, disque), le réseau NAT et l'hyperviseur.
- **Nous** avons géré le système (nom, clés SSH, adresse IP), le pare-feu ufw, le serveur Nginx, la page web et les données.

---

# Partie 4 : Mesurer et nettoyer

**Étape 1 : ressources vues depuis `web`**

```powershell
PS> ssh -p 2222 test@127.0.0.1 "free -h; df -h /; nproc"
test@127.0.0.1's password:
               total        used        free      shared  buff/cache   available
Mem:           956Mi       372Mi       189Mi       1,0Mi       540Mi       583Mi
Swap:          2,0Gi        16Ki       2,0Gi
Filesystem                         Size  Used Avail Use% Mounted on
/dev/mapper/ubuntu--vg-ubuntu--lv   12G  5,4G  5,3G  51% /
1
```

**Étape 2 : tailles des dossiers de VM sur l'hôte**

```powershell
PS> Get-ChildItem "$HOME\VirtualBox VMs" -Directory | ForEach-Object { "{0} : {1:N2} Go" -f $_.Name,
>> ((Get-ChildItem $_.FullName -Recurse -File | Measure-Object Length -Sum).Sum / 1GB) }
client : 5,98 Go
ubuntu-srv : 5,96 Go
web : 5,97 Go
```

**Étape 3 : arrêt et suppression des VM** 

```powershell
PS> VBoxManage controlvm web acpipowerbutton
PS> VBoxManage controlvm client acpipowerbutton
PS> VBoxManage list runningvms
PS> VBoxManage startvm web --type headless
PS> VBoxManage controlvm web poweroff
PS> VBoxManage unregistervm web --delete
PS> VBoxManage unregistervm client --delete
PS> VBoxManage natnetwork remove --netname tp-cloud
PS> VBoxManage list vms
(à compléter : seule ubuntu-srv doit rester)
```

## Réponses aux questions de la partie 4

**Q1. RAM vue par `web` et RAM consommée sur l'hôte.**
`web` voit environ 1 Go (**956 Mi**), comme configuré. Sur l'hôte, VirtualBox ne prend la mémoire qu'au fur et à mesure que la VM l'utilise : une VM qui ne fait presque rien consomme donc moins que son 1 Go .

**Q2. Place réellement prise par le disque d'un clone.**
Le disque a une taille maximale, mais le fichier ne grossit qu'avec ce qu'on écrit dedans. Sur l'hôte, `web` occupe seulement **5,97 Go**. Chaque clone est une vraie copie complète (environ 6 Go chacun), ce qui explique aussi les 19 secondes de clonage.

**Q3. `acpipowerbutton`, `poweroff`, `unregistervm --delete`.**
- **`acpipowerbutton`** : éteint la VM proprement, comme un appui sur le bouton d'alimentation.
- **`poweroff`** : coupe d'un coup, comme si on débranchait la prise (risque de perdre des données).
- **`unregistervm --delete`** : supprime la VM et ses fichiers définitivement.

**Q4. VM arrêtée dans le cloud : encore facturée ?**
Le calcul (CPU et RAM) n'est plus facturé, mais le **disque** et l'**adresse IP publique** continuent de l'être tant qu'ils existent. Pour ne plus rien payer, il faut supprimer l'instance, son disque et son IP.
