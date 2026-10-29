# Simulation-d-un-reseau-avec-spanning-tree

Étape 1 : Conception et définition des structures de données
La première étape consiste à créer des structures de données pour représenter les équipements du réseau local : sta-
tions et switchs. Pour cela, vous aurez tout d’abord besoin de créer des types permettant de représenter des adresses
Ethernet et IPv4. L’adresse IP ne servira ici qu’à identifier les stations dans le réseau puisqu’il n’est pas demandé de
modéliser l’algorithme de routage IP.

Une station sera représentée par les informations suivantes :
— adresse MAC (Ethernet)
— adresse IP

Un switch sera représenté par les informations suivantes :
— adresse MAC (Ethernet)
— nombre de ports
— priorité (pour le protocole STP)
— table de commutation

Enfin, il faudra définir une structure permettant de représenter un réseau local interconnectant plusieurs équipe-
ments.
Pour toutes ces structures et types de données, créez l’ensemble des fonctions permettant de les visualiser correcte-
ment (affichage de l’adresse IP en notation décimale pointée, de l’adresse MAC en hexadécimal, table d’association
d’un switch, etc.).


Étape 2 : Stockage d’une architecture dans un fichier de configuration
Afin de pouvoir créer rapidement des architectures utilisant vos structures de données, un réseau local sera encodé
dans un fichier de configuration dont le format est imposé.

Il s’agit d’un fichier texte contenant une première ligne d’en-tête comportant deux nombres séparés par un espace :
le nombre d’équipements dans le réseau, suivi du nombre de liens interconnectant ces équipements. Ensuite, les
lignes suivantes correspondront à chaque équipement réseau décrits par les informations nécessaires pour les créer
(adresses MAC et IP pour les stations, adresse MAC, nombre de ports et priorité pour les switchs). Le type d’équipe-
ment sera donné par un entier en début de ligne (1 pour une station, 2 pour un switch).

Les switchs seront toujours regroupés en premier dans le fichier, suivis des stations.
Enfin, les lignes suivantes correspondent à chaque lien d’interconnexion, représenté par les numéros des deux équi-
pements interconnectés par ce lien ainsi que le coût (poids) de ce lien.
Rappel : le poids des liens est dépendant du débit. Le protocole STP les définit de la façon suivante :
— 10Mb/s : poids = 100
— 100Mb/s : poids = 19
1
— 1Gb/s : poids = 4

Un exemple est donné ci-dessous :
4 3
2;01:45:23:a6:f7:ab;8;1024
1;54:d6:a6:82:c5:23;130.79.80.21
1;c8:69:72:5e:43:af;130.79.80.27
1;77:ac:d6:82:12:23;130.79.80.42
0;1;4
0;2;19
0;3;4

Ce fichier de configuration correspond au réseau suivant :
sw1
st1 st2
st3
4
19
4
avec :
0. switch sw1
— adresse MAC : 01 :45 :23 :a6 :f7 :ab
— nombre de ports : 8
— priorité : 1024
1. station st1
— adresse IP : 130.79.80.21
— adresse MAC : 54 :d6 :a6 :82 :c5 :23
2. station st2
— adresse IP : 130.79.80.27
— adresse MAC : c8 :69 :72 :5e :43 :af
3. station st3
— adresse IP : 130.79.80.42
— adresse MAC : 77 :ac :d6 :82 :12 :23


Étape 3 : Commutation de trames Ethernet
Afin de simuler un échange de trame Ethernet dans un réseau local, il faut tout d’abord créer une structure de don-
nées permettant de représenter une telle trame. Afin d’être très proche de la réalité, il est demandé que la structure
corresponde fidèlement à l’architecture réelle d’une trame Ethernet même si seuls les champs d’adresse (et éventuel-
lement de type) seront utilisés dans ce projet.

Rappel - le format d’une trame Ethernet est le suivant :
préambule SFD @destination @source type données bourrage FCS
7 octets 1 octet 6 octets 6 octets 2 octets 0 à 1500 octets 0 à 46 octets 4 octets
DATA

Chaque champ occupe un nombre entier d’octets :
— préambule : synchronisation 7 octets 10101010 (début peut être perdu)
— SFD (Start of Frame Delimiter) : début de l’info utile 1 octet 10101011
— adresse destination : 6 octets
— adresse source : 6 octets
— type : 2 octets - indique quel protocole est concerné par le message.
Ex : 0x0800 (IPv4), 0x0806 (ARP), 0x86DD (IPv6)…
— DATA (données) : 46 à 1500 octets (y compris bourrage/padding éventuel)
2
— FCS (Frame Check Sequence) : 4 octets - code polynomial détecteur d’erreurs
Prévoir une fonction d’affichage d’une trame en mode ”utilisateur”, avec les informations clairement présentées
(source, destination, type, etc.) et aussi en hexadécimale (contenu ”brut” des octets dans la trame).


Étape 4 : Spanning Tree Protocol
Cette étape consiste à implanter le protocole STP au sein d’un réseau local avec échange des BPDU entre les switchs.
On chargera un réseau à partir d’un fichier de configuration. Toutes les tables d’associations seront vides au départ.
Tous les ports seront dans un état inconnu. Le protocole STP permettra de configurer automatiquement tous les ports
de tous les switchs du réseau local par échange de BPDU jusqu’à convergence.
Rappel : un port de chaque switch est élu ”port racine”, certains sont bloqués pour couper les cycles et les autres sont
des ports dits ”désignés” (ouvert à la transmission d’information)
