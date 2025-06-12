# Network_lab2
Il s'agit du deuxième laboratoire du cours de réseaux et systèmes informatique.

Dans les réseaux d'entreprise modernes, la segmentation du trafic est essentielle pour améliorer la sécurité, l'efficacité et la gestion du réseau. Les VLANs (Virtual Local Area Networks) permettent de diviser un réseau physique en plusieurs réseaux logiques, réduisant ainsi la diffusion inutile du trafic et améliorant l'organisation du réseau.

Le trunking, quant à lui, est un mécanisme permettant de transporter plusieurs VLANs sur un même lien physique entre des commutateurs ou entre un commutateur et un routeur. Il repose généralement sur le protocole IEEE 802.1Q, qui insère un tag VLAN dans les trames Ethernet pour identifier leur appartenance à un VLAN spécifique.

L’objectif de ce deuxième laboratoire est de comprendre et de configurer des VLANs et des liens Trunk sur des équipements réseau, en observant leur impact sur la communication et l’isolation des différents segments du réseau. Nous utiliserons un simulateur pour mettre en place un réseau multi-VLAN et assurer l’interconnexion entre eux à l’aide d’un trunking approprié.

## Qu'est ce qu'un VLAN ?
Un VLAN (Virtual Local Area Network) est un réseau local virtuel permettant de segmenter un réseau physique en plusieurs sous-réseaux logiques. Grâce aux VLANs, des machines physiquement connectées à un même commutateur peuvent être séparées en différents groupes logiques, comme si elles étaient sur des réseaux distincts.

L'objectif principal des VLANs est de réduire la diffusion du trafic, améliorer la sécurité et optimiser la gestion du réseau en regroupant les appareils selon leur fonction, plutôt que selon leur emplacement physique.

## Son fonctionnement
Un VLAN fonctionne en attribuant un identifiant unique (VLAN ID) aux trames réseau, ce qui permet aux équipements de savoir à quel VLAN une trame appartient. Il existe deux types d'interfaces sur un commutateur prenant en charge les VLANs :

 1- Ports d'accès (Access Ports)

    * Un port d'accès appartient à un seul VLAN.

    * Il est généralement utilisé pour connecter des hôtes (PC, imprimantes, etc.).

    * Un PC connecté à un port configuré sur un VLAN spécifique ne pourra communiquer qu’avec d’autres appareils du même VLAN.

2- Ports de Trunk (Trunk Ports)

    * Un port trunk transporte plusieurs VLANs sur un même lien physique.

    * Il utilise un protocole de marquage des trames comme IEEE 802.1Q, qui ajoute un tag VLAN à chaque trame pour indiquer son appartenance.

    * Les trunks sont souvent utilisés pour interconnecter plusieurs commutateurs ou pour relier un commutateur à un routeur.




## Premier exercice du laboratoire

![Exercice 1](vlan1.png)
![Exercice 1](vlan11.png)

### Cas 1

![Exercice 1](conf1.png)

Pour ce qui est de la première configuration nous observons 3 switchs, la première et la deuxième communiquent avec du trunking et la deuxième et la troisième communiquent avec une connexion normale (mode access)
donc si l'information rentre dans le premier switch elle continue son chemin jusqu'au 2e switch sans problème (que ce soit pour le Vlan A ou B) car les deux premiers switch sont liés en mode trunk (ce qui veut dire qu'un tag a été ajouté dans la trame pour indiquer s cette dernière est à destination du vlan A ou B) par contre rien ne pourra passer dans le 3e switch car les ports ne font pas partie du meme vlan (donc meme si les 2 derniers switchs sont liés en mode access si les ports ne font pas partie du meme vlan alors aucune information ne sera échangé).
Voici à quoi devrait ressembler la configuration (j'ai ajouté des hosts afin de pouvoir tester le passage de l'information).

### étapes pour avoir cette architecture
Parfait ! L'image montre **trois switches connectés entre eux via des liens en mode trunk**, avec des hôtes répartis dans **deux VLANs : VLAN A et VLAN B**. Voici comment **configurer cette architecture pas à pas dans Cisco Packet Tracer**.

---

## 🧱 **Architecture**

* **VLAN A** : ports à gauche sur chaque switch (par ex. FastEthernet 0/1 à 0/12)
* **VLAN B** : ports à droite (par ex. FastEthernet 0/13 à 0/24)
* Les switches sont connectés entre eux via des **liens en mode trunk**
* Objectif : permettre aux hôtes d’un même VLAN de communiquer **même s’ils sont sur des switches différents**.

---

* VLAN A → VLAN 10
* VLAN B → VLAN 20
* Les ports FastEthernet 0/1 à 0/12 = VLAN A
* Les ports FastEthernet 0/13 à 0/24 = VLAN B
* Les ports GigabitEthernet 0/1 = liaison trunk entre les switches

---

## 🔧 Étapes de configuration

---

### ✅ 1. Sur **chaque switch**

Tu peux refaire ça pour les trois switches (nommés ici `Switch1`, `Switch2`, `Switch3`).

```bash
enable
configure terminal

! Création des VLANs
vlan 10
name VLAN_A
exit

vlan 20
name VLAN_B
exit

! Affectation des ports aux VLANs
interface range fa0/1 - 12
switchport mode access
switchport access vlan 10

interface range fa0/13 - 24
switchport mode access
switchport access vlan 20

! Configuration du port trunk vers les autres switches (ex: Gi0/1)
interface gig0/1
switchport trunk encapsulation dot1q
switchport mode trunk
```

---

### 📡 2. Connexions entre switches

Dans Packet Tracer :

* Connecte `Switch1` → `Switch2` via **GigabitEthernet 0/1**
* Connecte `Switch2` → `Switch3` via **GigabitEthernet 0/1**
* Assure-toi que tu as bien connecté **Gig0/1 ↔ Gi0/1**

---

### 💻 3. Ajouter les PC et tester

Ajoute 6 PC :

| PC  | Switch  | Port   | VLAN | IP           |
| --- | ------- | ------ | ---- | ------------ |
| PC1 | Switch1 | Fa0/1  | A    | 192.168.10.1 |
| PC2 | Switch2 | Fa0/2  | A    | 192.168.10.2 |
| PC3 | Switch3 | Fa0/3  | A    | 192.168.10.3 |
| PC4 | Switch1 | Fa0/13 | B    | 192.168.20.1 |
| PC5 | Switch2 | Fa0/14 | B    | 192.168.20.2 |
| PC6 | Switch3 | Fa0/15 | B    | 192.168.20.3 |

⚠️ Configure les adresses IP manuellement dans chaque PC :

```
IP: 192.168.10.x ou 192.168.20.x
Masque: 255.255.255.0
Passerelle: (laisser vide pour test local)
```

---

### 🔁 4. Test de communication

✅ Tu dois pouvoir :

* `PC1` pinger `PC2` et `PC3` (tous dans VLAN A)
* `PC4` pinger `PC5` et `PC6` (tous dans VLAN B)
* ❌ `PC1` ne doit **pas** pouvoir pinger `PC4` (VLAN différent)

---
**J'ai ajouté tous les fichiers des cas presents dans les exercices pour que vous puissiez tester les configurations mais n'hésitez pas à refaire la configuration**
Pour les autres cas on fait exactement la meme chose mais en modifiant les modes donc il n'y a pas grand interet à toutes les reproduire, sauf les deux dernières qui sont assez interessantes.













## Deuxième exercice du laboratoire
