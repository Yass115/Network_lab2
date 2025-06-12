schema avec un switch 3 ordi qui sont relies a 3 routeurs et un de ces 3 routeurs il est relié a un serveur internet 
Il te demmande d'escrire la config pour le routeur pour avoir pc1 qui appartien a vlan 10 pc2 vlan 20 et pc3 a vlan 30, apres il dit que il faut que tout cds ordinateurs doivient avoir acces au service internet , 
ils ne doivient pas communiquer entre eux et il faut choisir le chemin plus rapide qu'il vous indiquera donc.
Parfait, je vais t’aider à construire **l’architecture complète**, puis te donner les **commandes de configuration** pour que ton labo fonctionne parfaitement, selon ces contraintes :

---

### 🎯 **Objectifs à respecter :**

1. **PC1 → VLAN 10**, **PC2 → VLAN 20**, **PC3 → VLAN 30**.
2. Les **PC ne doivent pas pouvoir communiquer entre eux**.
3. Tous les **PC doivent accéder à Internet via un routeur spécifique**.
4. Tu dois **choisir le chemin le plus rapide** selon les consignes du prof (on supposera un chemin via un seul routeur ici).
5. Tu dois faire ça dans **Cisco Packet Tracer**, avec **1 switch, 3 routeurs, 3 PC, 1 serveur Internet**.

---

### 🖥️ **Architecture réseau (vue simplifiée)**

```
             [PC1]         [PC2]         [PC3]
               |             |             |
           VLAN 10       VLAN 20       VLAN 30
               |             |             |
               +-------------+-------------+
                             |
                          [Switch]
                             |
          +--------+---------+---------+
          |        |                   |
      [R1]       [R2]               [R3] (Accès Internet)
                                        |
                                   [Serveur Internet]
```

Supposons que :

* **PC1 (VLAN 10)** va passer par **R1**
* **PC2 (VLAN 20)** va passer par **R2**
* **PC3 (VLAN 30)** va passer par **R3**, qui est aussi connecté à **Internet**

Mais comme **Internet est accessible uniquement via R3**, il faut :

* soit **faire du routage statique** sur R1 et R2 vers R3,
* soit **utiliser un seul routeur (R3)** pour le routage inter-VLAN (plus simple si autorisé par le prof).

---

## ✅ OPTION PRATIQUE (plus simple à configurer et à justifier)

> **Faire du "Router on a Stick" avec un seul routeur (R3)** pour inter-VLAN et accès Internet. On évite la communication entre VLAN avec des ACL.

---

### 🧱 1. **Configuration VLAN sur le Switch**

```bash
enable
conf t

vlan 10
name VLAN10-PC1
exit

vlan 20
name VLAN20-PC2
exit

vlan 30
name VLAN30-PC3
exit

interface range fa0/1 - 3
switchport mode access

interface fa0/1
switchport access vlan 10

interface fa0/2
switchport access vlan 20

interface fa0/3
switchport access vlan 30

interface fa0/24
switchport mode trunk

exit
```

---

### 🔁 2. **Configuration du routeur (R3) pour le routage inter-VLAN**

Sur l’interface connectée au switch (ex: `g0/0`) :

```bash
enable
conf t

interface g0/0.10
encapsulation dot1Q 10
ip address 192.168.10.1 255.255.255.0

interface g0/0.20
encapsulation dot1Q 20
ip address 192.168.20.1 255.255.255.0

interface g0/0.30
encapsulation dot1Q 30
ip address 192.168.30.1 255.255.255.0

interface g0/0
no shutdown

! Interface vers Internet
interface g0/1
ip address 203.0.113.1 255.255.255.0
no shutdown

ip route 0.0.0.0 0.0.0.0 203.0.113.254  ! passerelle vers le serveur Internet
exit
```

---

### 🌍 3. **PCs configuration (manuellement ou via DHCP)**

| PC  | VLAN | IP Address    | Gateway      |
| --- | ---- | ------------- | ------------ |
| PC1 | 10   | 192.168.10.10 | 192.168.10.1 |
| PC2 | 20   | 192.168.20.10 | 192.168.20.1 |
| PC3 | 30   | 192.168.30.10 | 192.168.30.1 |

---

### 🔒 4. **ACL pour empêcher les VLANs de communiquer entre eux (sur R3)**

```bash
ip access-list extended VLAN10-TO-INTERNET
permit ip 192.168.10.0 0.0.0.255 any
deny ip 192.168.10.0 0.0.0.255 192.168.20.0 0.0.0.255
deny ip 192.168.10.0 0.0.0.255 192.168.30.0 0.0.0.255

ip access-list extended VLAN20-TO-INTERNET
permit ip 192.168.20.0 0.0.0.255 any
deny ip 192.168.20.0 0.0.0.255 192.168.10.0 0.0.0.255
deny ip 192.168.20.0 0.0.0.255 192.168.30.0 0.0.0.255

ip access-list extended VLAN30-TO-INTERNET
permit ip 192.168.30.0 0.0.0.255 any
deny ip 192.168.30.0 0.0.0.255 192.168.10.0 0.0.0.255
deny ip 192.168.30.0 0.0.0.255 192.168.20.0 0.0.0.255

interface g0/0.10
ip access-group VLAN10-TO-INTERNET in

interface g0/0.20
ip access-group VLAN20-TO-INTERNET in

interface g0/0.30
ip access-group VLAN30-TO-INTERNET in
```

---

### 🧪 5. **Test de connectivité**

Depuis chaque PC :

```bash
ping 192.168.X.1     ✅ (gateway)
ping 203.0.113.254   ✅ (serveur Internet)
ping autre PC        ❌ (interdiction par ACL)
```

---

# NAT

Parfait ! Faisons du **NAT (Network Address Translation)** sur Cisco Packet Tracer. C’est indispensable quand tu veux que plusieurs PC dans un réseau local (LAN) accèdent à Internet à travers **une seule IP publique** (celle du routeur connecté à Internet ou à ton "serveur").

---

## 🎯 Objectif :

Mettre en place du **NAT statique ou dynamique** pour permettre à tes PC (réseau privé) d'accéder à Internet (réseau public).

---

## 🧱 TOPOLOGIE DE BASE

```
[PC1 - 192.168.1.2]      \
[PC2 - 192.168.1.3] ---> [SW] ---> [ROUTEUR] ---> [SERVER INTERNET - 8.8.8.8]
```

### Sur le routeur :

* Interface LAN : `Gig0/0` → `192.168.1.1/24`
* Interface WAN : `Gig0/1` → `200.0.0.1/30` (simule une IP publique)

---

## ✅ 1. CONFIGURATION DES INTERFACES DU ROUTEUR

```bash
enable
configure terminal

interface gig0/0
ip address 192.168.1.1 255.255.255.0
ip nat inside
no shutdown
exit

interface gig0/1
ip address 200.0.0.1 255.255.255.252
ip nat outside
no shutdown
exit
```

---

## ✅ 2. CRÉER UNE LISTE D’ACCÈS POUR SPÉCIFIER LES IPS LOCALES À TRADUIRE

```bash
access-list 1 permit 192.168.1.0 0.0.0.255
```

---

## ✅ 3. CONFIGURATION DU **NAT DYNAMIQUE** (PAT : un à plusieurs)

```bash
ip nat inside source list 1 interface gig0/1 overload
```

---

## ✅ 4. CONFIGURATION DES PC

Chaque PC doit avoir :

* IP : `192.168.1.x`
* Subnet mask : `255.255.255.0`
* Gateway : `192.168.1.1`
* DNS : `8.8.8.8` (ou IP de ton serveur)

---

## ✅ 5. TESTER

Sur un PC :

* Ouvre **Command Prompt** → `ping 8.8.8.8`
* Tu devrais voir les réponses ✅

---

## ✅ 6. POUR NAT STATIQUE (ex : accès depuis l'extérieur vers un serveur local)

```bash
ip nat inside source static 192.168.1.10 200.0.0.2
```

> Cela signifie que toute demande sur `200.0.0.2` sera redirigée vers le PC local `192.168.1.10`.

---

## 📌 POUR ENREGISTRER

```bash
write memory
`---
