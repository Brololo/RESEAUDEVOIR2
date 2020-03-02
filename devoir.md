I. Création et utilisation simples d'une VM CentOS

4. Configuration réseau d'une machine CentOS

| Name | IP            | MAC                 | Fonction |
| ---- | ------------- | ------------------- | -------- |
| `lo` | `127.0.0.1/8` | `00:00:00:00:00:00` | Lookback |

| Name     | IP             | MAC                 | Fonction                   |
| -------- | -------------- | ------------------- | -------------------------- |
| `enp0s3` | `10.0.2.15/24` | `08.00.27.84.08.64` | Carte NAT (accès internet) |

| Name     | IP            | MAC                 | Fonction          |
| -------- | ------------- | ------------------- | ----------------- |
| `enp0s8` | `10.2.1.2/24` | `08.00.27.25.24:c5` | Host-only Adapter |

Changer la configuration de la carte réseau Host-Only:

Changer vers une autre IP statique : 1) cd /etc/sysconfig/network-scripts 2) sudo nano ifcfg-enp0s8 3) IPADDR --> 10.2.1.3

commande "ifdown esp0s8" puis "ifup esp0s8"

La commande "ip a" affiche désormais : inet 10.2.1.3/24

L'ip à donc bien changé.

Puis la commande "ping 10.2.1.3" permet de ping mon réseau Host-Only.

Entre 37 et 80 ms.

Faites un scan nmap du réseau host-only : nmap -sP 10.2.1.0/24

Déterminer l'adresse IP des machines trouvées : Nmap scan report for 10.2.1.3

Nmap done: 256 IP addresses (2 hosts up)

Les 2 hosts sont ma vm et ma carte réseau host-only.

Les IP sont dans l'ordre : 10.2.1.1 et 10.2.1.3

Je fais un ipconfig /all pour trouver l'adresse MAC de ma carte réseau host-only.

Les 2 adresses MAC correspondent bien : 0A:00:27:00:00:0D

Lister les connexions actives en TCP (-t), UDP (-u) : J'utilise la commande "ss -tu" et les résultats affichés sont :

| Nedit | State | Recv-Q | Send-Q | Local Address:Port | Peer Address:Port |
| ----- | ----- | ------ | ------ | ------------------ | ----------------- |


Il n'y à aucun résultat.

Liste les ports en écoute (-l), en TCP (-t) et UDP (-u) : J'utilise la commande "ss -ltu" et les résultats affichés sont :

| Nedit | State    | Recv-Q | Send-Q | Local Address:Port | Peer Address:Port |
| ----- | -------- | ------ | ------ | ------------------ | ----------------- |
| `udp` | `UNCONN` | `0`    | `0`    | `*:bootpc`         | `*:*`             |
| `tcp` | `LISTEN` | `0`    | `100`  | `127.0.0.1:smtp`   | `*:*`             |
| `tcp` | `LISTEN` | `0`    | `128`  | `*:ssh`            | `*:*`             |
| `tcp` | `LISTEN` | `0`    | `100`  | `[::1]:smtp`       | `[::]:*`          |
| `tcp` | `LISTEN` | `0`    | `128`  | `[::]:ssh`         | `[::]:*`          |

J'ai 5 résultats.

Liste les ports en écoute, en TCP et UDP, sans traduire les numéros de protocole (-n) : J'utilise la commande "ss -ltun" et les résultats affichés sont :

| Nedit | State    | Recv-Q | Send-Q | Local Address:Port | Peer Address:Port |
| ----- | -------- | ------ | ------ | ------------------ | ----------------- |
| `udp` | `UNCONN` | `0`    | `0`    | `*:68`             | `*:*`             |
| `tcp` | `LISTEN` | `0`    | `100`  | `127.0.0.1:25`     | `*:*`             |
| `tcp` | `LISTEN` | `0`    | `128`  | `*:22`             | `*:*`             |
| `tcp` | `LISTEN` | `0`    | `100`  | `[::1]:25`         | `[::]:*`          |
| `tcp` | `LISTEN` | `0`    | `128`  | `[::]:22`          | `[::]:*`          |

J'ai 5 résultats.

Idem, mais affiche en plus le processus (le "programme", ou "logiciel", ou "application") qui est en jeu dans cette connexion (-p)

J'utilise la commande "ss -ltunp" et les résultats affichés sont :

| Nedit | State    | Recv-Q | Send-Q | Local Address:Port | Peer Address:Port |
| ----- | -------- | ------ | ------ | ------------------ | ----------------- |
| `udp` | `UNCONN` | `0`    | `0`    | `*:68`             | `*:*`             |

users:(("dhclient",pid=861,fd=6))
| `tcp` | `LISTEN` | `0` | `100` | `127.0.0.1:25` | `*:*` |
users:(("master",pid=1316,fd=13))
| `tcp` | `LISTEN` | `0` | `128` | `*:22` | `*:*` |
users:(("sshd",pid=1085,fd=3))
| `tcp` | `LISTEN` | `0` | `100` | `[::1]:25` | `[::]:*` |
users:(("master",pid=1316,fd=14))
| `tcp` | `LISTEN` | `0` | `128` | `[::]:22` | `[::]:*` |
users:(("sshd",pid=1085,fd=4))

Déterminer quel programme écoute sur chacun de ces ports :

Le programme `dhclient` écoute sur le port `*:68`
Le programme `master` écoute sur le port `[::1]:25` et `127.0.0.1:25`
Le programme `sshd` écoute sur le port `*:22` et `[::]:22`

II. Notion de ports

1. SSH

Déterminer sur quelle(s) IP(s) et sur quel(s) port(s) le serveur SSH écoute actuellement.

Le programme `sshd` écoute sur le port `*:22` et `[::]:22`

il faut modifier le fichier /etc/ssh/sshd_config:

On rentre dans se fichier et on modifie le paramètre "#Port 22" en "Port 1025".

sudo systemctl restart sshd

🌞 Vérifier que le service SSH écoute sur le nouveau port choisi:

J'utilise la commande "ss -ltunp" et le résultat affiché intéressant est :

| `tcp` | `LISTEN` | `0` | `128` | `*:1025` | `*:*` |
users:(("sshd",pid=1373,fd=3))
...
| `tcp` | `LISTEN` | `0` | `128` | `[::]:1025` | `[::]:*` |
users:(("sshd",pid=1373,fd=4))

On voit que le port est passé de 22 à 1025.

De mon pc je fais la commande "ssh root@10.2.1.3 -p 1025" mais rien ne se passe.

🌞 Ouvrir le port correspondant du firewall:

sudo firewall-cmd --add-port=1025/tcp --permanent pour autoriser les connexions sur le port TCP 1025

sudo firewall-cmd --reload permet aux modifications effectuées de prendre effet

puis vérifier la bonne connexion:

De mon pc je fais la commande "ssh root@10.2.1.3 -p 1025": (-p permet de préciser le port)

Et je réussi à me connecter à ma vm : on m'affiche : Warning: Permanently added '[10.2.1.3]:1025' (ECDSA) to the list of known hosts.

Sur ma vm je fais les commandes "firewall-cmd --add-port=1024/tcp --permanent"
"firewall-cmd --reload" pour ouvrir le port 1024

Sur mon pc je fais les commandes ".\nc64.exe 10.2.1.3 1024" quand je suis dans le dossier contenant netcat64

La connexion est établie et les messages s'échanges correctement.

dans un troisième terminal sur la VM

utiliser ss pour visualiser la connexion netcat en cours:

j'ai connecté une seconde vm avec l'ip 10.2.1.3

Une fois la connexion établie j'utilise la commande "ss-l"

🌞 Inversez les rôles, le PC est le serveur netcat, la VM est le client.

Sur mon pc je vais dans mon dossier contenant netcat. Puis j'utilise la commande "./nc64.exe -l -p 5555" pour écouter sur le port 5555, mon pc est donc maintenant le serveur netcat.

Sur un autre powershell je fais ipconfig pour savoir l'ip de mon pc.

Sur ma vm je fais la commande "nc <ip du pc> 5555" pour connecter ma machine à mon serveur netcat.

Les messages s'échangent bien.

3. Wireshark

🌞 Essayez de mettre en évidence

le fait que le contenu des messages envoyés avec netcat est visible dans Wireshark
les trois messages d'établissement de la connexion

peu importe qui est client et qui est serveur une fois la connexion établie
mais au moment de l'établissement de la connexion, il y a trois messages particuliers qui sont échangés
ces trois messages ont pour source le client
![screen1](capture2réseau.jpg)
la connexion qui s'établie entre mon pc (serveur) et la vm (client)
![screen2](capture1réseau.jpg)
On peut voir le message envoyé
![screen3](capture3réseau.jpg)
la connexion qui s'établie entre mon pc (client) et la vm (serveur)
![screen4](capture4réseau.jpg)
On peut voir le message envoyé (c'est le bonsoir en fin de ligne)

N'AYANT PAS D'ENTREE ETHERNET SUR MON PC LA SUITE DE SE DEVOIR EST TIRE DE CELUI DE MARC TEXIER.

III. Routage statique

### 2. Configuration du routage

#### A. PC1

Ping de PC2(10.2.2.1) depuis PC1:

```bash
C:\Windows\system32>ping 10.2.2.1

Envoi d’une requête 'Ping'  10.2.2.1 avec 32 octets de données :
Réponse de 10.2.2.1 : octets=32 temps=3 ms TTL=127
Réponse de 10.2.2.1 : octets=32 temps=1 ms TTL=127
Réponse de 10.2.2.1 : octets=32 temps=1 ms TTL=127
Réponse de 10.2.2.1 : octets=32 temps=2 ms TTL=127

Statistiques Ping pour 10.2.2.1:
    Paquets : envoyés = 4, reçus = 4, perdus = 0 (perte 0%),
Durée approximative des boucles en millisecondes :
    Minimum = 1ms, Maximum = 3ms, Moyenne = 1ms
```

#### B. PC2

Ping de PC1(10.2.1.1) depuis PC2 :

```bash
C:\WINDOWS\system32>ping 10.2.1.1

Envoi d’une requête 'Ping'  10.2.1.1 avec 32 octets de données :
Réponse de 10.2.1.1 : octets=32 temps=1 ms TTL=127
Réponse de 10.2.1.1 : octets=32 temps=2 ms TTL=127

Statistiques Ping pour 10.2.1.1:
    Paquets : envoyés = 2, reçus = 2, perdus = 0 (perte 0%),
Durée approximative des boucles en millisecondes :
    Minimum = 1ms, Maximum = 2ms, Moyenne = 1ms
```

#### C. VM1

Commande utiliser pour ajouter une route :

```bash
ip route add 10.2.2.0/24 via 10.2.1.1 dev enp0s8
```

Ping PC2(10.2.2.1) depuis VM1 :

```bash
[centos@localhost ~]$ ping 10.2.2.1
PING 10.2.2.1 (10.2.2.1) 56(84) bytes of data.
64 bytes from 10.2.2.1: icmp_seq=1 ttl=126 time=1.88 ms
64 bytes from 10.2.2.1: icmp_seq=2 ttl=126 time=1.98 ms
64 bytes from 10.2.2.1: icmp_seq=3 ttl=126 time=2.78 ms
^C
--- 10.2.2.1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2018ms
rtt min/avg/max/mdev = 1.889/2.222/2.789/0.404 ms
```

#### D. VM2

Ping PC1 depuis VM2:

```bash
[centos@localhost ~]$ ping 10.2.1.1
PING 10.2.1.1 (10.2.1.1) 56(84) bytes of data.
64 bytes from 10.2.1.1: icmp_seq=1 ttl=126 time=2.01 ms
64 bytes from 10.2.1.1: icmp_seq=2 ttl=126 time=2.56 ms
64 bytes from 10.2.1.1: icmp_seq=3 ttl=126 time=3.22 ms
64 bytes from 10.2.1.1: icmp_seq=4 ttl=126 time=2.84 ms
64 bytes from 10.2.1.1: icmp_seq=5 ttl=126 time=2.57 ms
^C
--- 10.2.1.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4009ms
rtt min/avg/max/mdev = 2.015/2.644/3.229/0.398 ms
```

#### E. El gran final

Ping VM1(10.2.1.2) depuis VM2:

```bash
[centos@localhost ~]$ ping 10.2.1.2
PING 10.2.1.2 (10.2.1.2) 56(84) bytes of data.
64 bytes from 10.2.1.2: icmp_seq=1 ttl=62 time=1.90 ms
64 bytes from 10.2.1.2: icmp_seq=2 ttl=62 time=2.37 ms
64 bytes from 10.2.1.2: icmp_seq=3 ttl=62 time=2.78 ms
64 bytes from 10.2.1.2: icmp_seq=4 ttl=62 time=3.02 ms
64 bytes from 10.2.1.2: icmp_seq=5 ttl=62 time=2.26 ms
^C
--- 10.2.1.2 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4011ms
rtt min/avg/max/mdev = 1.906/2.469/3.023/0.397 ms
```

Ping VM2 depuis VM1:

```bash
[centos@localhost ~]$ ping 10.2.2.2
PING 10.2.2.2 (10.2.2.2) 56(84) bytes of data.
64 bytes from 10.2.2.2: icmp_seq=1 ttl=62 time=1.85 ms
64 bytes from 10.2.2.2: icmp_seq=2 ttl=62 time=3.16 ms
64 bytes from 10.2.2.2: icmp_seq=3 ttl=62 time=3.54 ms
^C
--- 10.2.2.2 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2007ms
rtt min/avg/max/mdev = 1.853/2.855/3.544/0.724 ms
```

### 3. Configuration des noms de domaine

VM1 ping VM2 avec nom dommaine:

```bash
[centos@localhost ~]$ ping vm2.tp2.b1
PING vm2.tp2.b1 (10.2.2.2) 56(84) bytes of data.
64 bytes from vm2.tp2.b1 (10.2.2.2): icmp_seq=1 ttl=62 time=3.89 ms
64 bytes from vm2.tp2.b1 (10.2.2.2): icmp_seq=2 ttl=62 time=3.01 ms
64 bytes from vm2.tp2.b1 (10.2.2.2): icmp_seq=3 ttl=62 time=2.47 ms
^C
--- vm2.tp2.b1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2004ms
rtt min/avg/max/mdev = 2.477/3.127/3.894/0.584 ms
```

Traceroute de la VM2 depuis VM1 :

```bash
[centos@localhost ~]$ traceroute vm2.tp2.b1
traceroute to vm2.tp2.b1 (10.2.2.2), 30 hops max, 60 byte packets
 1  gateway (10.0.2.2)  0.191 ms  0.101 ms  0.070 ms
 2  gateway (10.0.2.2)  2.520 ms  2.408 ms  2.307 ms
```
