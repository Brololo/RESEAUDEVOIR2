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

La commande "ip a" affiche désormais : inet 10.2.1.3/24

L'ip à donc bien changé.

Puis la commande "ping 10.2.1.3" permet de ping mon réseau Host-Only.

Entre 37 et 80 ms.

Faites un scan nmap du réseau host-only : nmap 10.2.1.3

Déterminer l'adresse IP des machines trouvées : Nmap scan report for 10.2.1.3

Nmap done: 256 IP addresses (2 hosts up)

Les 2 hosts sont mon pc et ma carte réseau host-only.

Les IP sont dans l'ordre : 10.33.3.144 et 10.2.1.3

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
users:(("master",pid=1085,fd=3))
| `tcp` | `LISTEN` | `0` | `100` | `[::1]:25` | `[::]:*` |
users:(("master",pid=1316,fd=14))
| `tcp` | `LISTEN` | `0` | `128` | `[::]:22` | `[::]:*` |
users:(("master",pid=1085,fd=4))

Déterminer quel programme écoute sur chacun de ces ports :

Le programme `dhclient` écoute sur le port `*:68`
Le programme `master` écoute sur le port `[::1]:25` et `127.0.0.1:25`
Le programme `sshd` écoute sur le port `*:22` et `[::]:22`

II. Notion de ports

1. SSH

Déterminer sur quelle(s) IP(s) et sur quel(s) port(s) le serveur SSH écoute actuellement.

Le programme `sshd` écoute sur le port `*:22` et `[::]:22`
