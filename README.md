# Module 1

# Validations & Preuves OSPF (TP 1.1 - 1.3)

## Topologie PNetLab
![Topologie PNetLab OSPF](https://github.com/user-attachments/assets/0fca844d-1f34-403c-9909-192e8962f8b1)

---

## 1. Voisinage OSPF (`show ip ospf neighbor`)

### R1
![R1 show ip ospf neighbor](https://github.com/user-attachments/assets/b4106ad1-c485-4aeb-b50a-cf55c77d8a96)
* Voisins 2.2.2.2 et 3.3.3.3 validés.

### R2
![R2 show ip ospf neighbor](https://github.com/user-attachments/assets/ca25fc3b-75b1-41b8-ae2d-08c9c047c3ce)
* Voisins 1.1.1.1 et 3.3.3.3 validés.

### R3
![R3 show ip ospf neighbor](https://github.com/user-attachments/assets/9f4c4f11-5efe-416a-823a-51d82cebc206)
* Voisins 1.1.1.1 et 2.2.2.2 validés.

---

## 2. Métrique des Interfaces (`show ip ospf interface FastEthernet1/0 | include Cost`)

### R1
![R1 Cost](https://github.com/user-attachments/assets/d19cee6a-248a-4117-b28e-18bd5bc540e5)
* Coût à 100 car le bandwidth est à 10000.

### R2
![R2 Cost](https://github.com/user-attachments/assets/aa109af9-9e8e-4421-881e-9b271a707270)
* Coût à 10 car le bandwidth est à 100000 (par défaut).

### R3
![R3 Cost](https://github.com/user-attachments/assets/0f40e483-5d2b-47dc-a576-f844f7ec7e4d)
* Coût à 10 car le bandwidth est à 100000 (par défaut).

---

## 3. Reroutage Dynamique (`traceroute 3.3.3.3` depuis R1)

![Traceroute R1 vers R3](https://github.com/user-attachments/assets/a820a170-f0f5-471e-ad77-be4b39d85c10)
* Le flux passe bien par R2 (`10.0.12.2`) pour joindre R3.

---

## 4. Inspection LSDB (`show ip ospf database router` sur R1)

![LSDB Part 1](https://github.com/user-attachments/assets/78969ec0-762f-4799-b98e-d4bf1065c13e)
![LSDB Part 2](https://github.com/user-attachments/assets/b3717b48-f871-4e09-8adb-10f07d1eb1cf)
![LSDB Part 3](https://github.com/user-attachments/assets/75c3d989-51f0-4729-b28e-53f712fdf9de)
![LSDB Part 4](https://github.com/user-attachments/assets/1404c6da-aaab-4b5f-bdbf-de306c47cf1f)

---

## 5. Paquets Hello & Analyse Trame (`debug ip ospf hello` & Wireshark sur R1)

![Debug Hello OSPF](https://github.com/user-attachments/assets/8c5f1c8a-62fb-4436-aefb-cf15fed8d255)
* Validation des paquets Keepalive et échanges Hello OSPF.

### Captures Wireshark
* **Wireshark Fa0/0 (Lien R1-R2) :**
![Wireshark Fa0/0](https://github.com/user-attachments/assets/82e821b6-bdbc-4054-93f1-c176cb311b38)

* **Wireshark Fa1/0 (Lien R1-R3) :**
![Wireshark Fa1/0](https://github.com/user-attachments/assets/e3cf0c5f-c005-45ef-9ef4-7e71cfb52d22)

# Module 2

DR et BDR (puis Drother) avec priorités
<img width="887" height="282" alt="image" src="https://github.com/user-attachments/assets/311ecd10-d795-44dc-bf80-25f8b9d2da3a" />


passives, route par défaut OSPF et MD5
R1 
<img width="460" height="132" alt="image" src="https://github.com/user-attachments/assets/b2049115-ee11-41de-884f-75f378211027" />
<img width="953" height="750" alt="image" src="https://github.com/user-attachments/assets/1a804f06-318a-442c-9d25-6902ce3d027b" />
<img width="892" height="610" alt="image" src="https://github.com/user-attachments/assets/1d656fd2-fa77-4605-8131-632a8adeb45c" />


PASSAGE SUR PNETLAB EN LOCAL C'EST PLUS OU MOINS LA MEME CHOSE 
donc pas d'image avant module 4 (c'etait surtout du copier coller de pnetlab hebergé)

Documentation Technique — Lab CCNA OSPF & Hardening (R1, R2, R3, SW1)

1. Architecture et Plan d'Adressage
Topologie : 3 Routeurs Cisco c7200 interconnectés via un switch Catalyst (module NM-16ESW).

Sous-réseau Backbone : 10.0.0.0/24

Loopbacks de management / OSPF :

R1 : 1.1.1.1/32 (Priorité OSPF : 255 - DR)

R2 : 2.2.2.2/32 (Priorité OSPF : 100 - BDR)

R3 : 3.3.3.3/32 (Priorité OSPF : 1 - DROTHER)

2. Configuration des Routeurs (R1, R2, R3)
R1 (Primary DR)
Cisco CLI
hostname R1
ip domain-name satom.local
username SATOMIT privilege 15 secret admin123
enable secret admin123

interface Loopback0
 ip address 1.1.1.1 255.255.255.255

interface FastEthernet0/0
 ip address 10.0.0.1 255.255.255.0
 ip ospf priority 255
 ip ospf message-digest-key 1 md5 CCNA-Satom2026!
 ip ospf authentication message-digest
 duplex full
 no shutdown

router ospf 1
 router-id 1.1.1.1
 network 10.0.0.0 0.0.0.255 area 0
 network 1.1.1.0 0.0.0.255 area 0

crypto key generate rsa modulus 2048
ip ssh version 2

line vty 0 4
 transport input ssh
 login local
 exec-timeout 5 0
R2 (BDR)
Cisco CLI
hostname R2
ip domain-name satom.local
username SATOMIT privilege 15 secret admin123
enable secret admin123

interface Loopback0
 ip address 2.2.2.2 255.255.255.255

interface FastEthernet0/0
 ip address 10.0.0.2 255.255.255.0
 ip ospf priority 100
 ip ospf message-digest-key 1 md5 CCNA-Satom2026!
 ip ospf authentication message-digest
 duplex full
 no shutdown

router ospf 1
 router-id 2.2.2.2
 network 10.0.0.0 0.0.0.255 area 0
 network 2.2.2.0 0.0.0.255 area 0

crypto key generate rsa modulus 2048
ip ssh version 2

line vty 0 4
 transport input ssh
 login local
 exec-timeout 5 0
R3 (DROTHER)
Cisco CLI
hostname R3
ip domain-name satom.local
username SATOMIT privilege 15 secret admin123
enable secret admin123

interface Loopback0
 ip address 3.3.3.3 255.255.255.255

interface FastEthernet0/0
 ip address 10.0.0.3 255.255.255.0
 ip ospf priority 1
 ip ospf message-digest-key 1 md5 CCNA-Satom2026!
 ip ospf authentication message-digest
 duplex full
 no shutdown

router ospf 1
 router-id 3.3.3.3
 network 10.0.0.0 0.0.0.255 area 0
 network 3.3.3.3 0.0.0.0 area 0

crypto key generate rsa modulus 2048
ip ssh version 2

line vty 0 4
 transport input ssh
 login local
 exec-timeout 5 0
3. Configuration du Switch (SW1)
Cisco CLI
hostname SW1
ip domain-name satom.local
enable secret admin123

interface range FastEthernet1/0 - 15
 switchport mode access
 speed 100
 duplex full
 no shutdown

line vty 0 4
 password admin123
 login
4. Preuves de Tests et Vérifications (Rapport)
État des Voisins OSPF
Cisco CLI
R1# show ip ospf neighbor

Neighbor ID     Pri   State           Dead Time   Address         Interface
2.2.2.2         100   FULL/BDR        00:00:18    10.0.0.2        FastEthernet0/0
3.3.3.3           1   FULL/DROTHER    00:00:16    10.0.0.3        FastEthernet0/0
Table de Routage OSPF
Cisco CLI
R1# show ip route ospf

      2.0.0.0/32 is subnetted, 1 subnets
O       2.2.2.2 [110/2] via 10.0.0.2, 00:10:20, FastEthernet0/0
      3.0.0.0/32 is subnetted, 1 subnets
O       3.3.3.3 [110/2] via 10.0.0.3, 00:10:30, FastEthernet0/0
Validation de la Connectivité (Ping Étendu)
Cisco CLI
R3# ping 1.1.1.1 source loopback0
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 1.1.1.1, timeout is 2 seconds:
Packet sent with a source address of 3.3.3.3
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 16/20/24 ms
Vérification du Service SSHv2
Cisco CLI
R1# show ip ssh
SSH Enabled - version 2.0
Authentication timeout: 120 secs; Authentication retries: 3
pour le module 3.2 et 3.3 je me suis retrouvé dans une impasse a cause du routeur donc j'ai supprime le 3745 et j'ai mis un vrai switch
