Ah, my bad chef, j'avais pas capté que les balises HTML <img ... /> se faisaient bouffer ou foiraient le rendu quand tu copiais.

Voilà la version 100 % brute, avec la syntaxe Standard Markdown ![alt](url) pour les images, sans aucune balise HTML pour que ça ne bug plus au copier-coller dans GitHub :

Plaintext
# 🛠️ TP — Configuration, Troubleshooting & Optimisation OSPF (Area 0)

![Cisco IOS](https://img.shields.io/badge/Cisco-IOS-blue?style=flat-square&logo=cisco)
![Protocol](https://img.shields.io/badge/Protocol-OSPFv2-green?style=flat-square)
![Topology](https://img.shields.io/badge/Topology-Triangle-orange?style=flat-square)

---

## 📌 1. Contexte & Architecture

Maquettage d'une interconnexion en triangle de **3 routeurs Cisco (R1, R2, R3)** au sein d'une aire OSPF unique (**Area 0**).

### 🎯 Objectifs
* Corriger les erreurs d'adressage IP et réétablir la convergence OSPF globale.
* Manipuler le coût OSPF via la bande passante (`bandwidth`) pour forcer le basculement dynamique des routes.
* Inspecter la base de données LSDB (LSA Type 1) et valider le maintien des adjacences (paquets Hello).

---

## 🚨 2. Chronologie du Troubleshooting

### ❌ Incident 1 : Voisinage OSPF absent entre R2 et R3

**Diagnostic sur R2 :**
```text
R2> show ip ospf neighbor
Neighbor ID     Pri   State           Dead Time   Address         Interface
1.1.1.1           0   FULL/  -        00:00:37    10.0.12.1       FastEthernet0/0
⚠️ Constat : R2 ne forme aucune adjacence avec R3. Seul R1 (1.1.1.1) est reconnu.

❌ Incident 2 : Conflit d'adressage IP sur le segment R2-R3
Diagnostic des interfaces sur R2 :

Plaintext
R2# show ip interface brief
R2# show ip ospf interface brief
Plaintext
Interface          IP-Address      OK? Method Status                  Protocol
FastEthernet0/0    10.0.12.2       YES NVRAM  up                      up
FastEthernet1/0    10.0.23.2       YES NVRAM  up                      up
Loopback0          2.2.2.2         YES NVRAM  up                      up

Interface    PID   Area   IP Address/Mask    Cost   State   Nbrs F/C
Lo0          1     0      2.2.2.2/32         1      LOOP    0/0
Fa1/0        1     0      10.0.23.2/30       10     P2P     0/0
Fa0/0        1     0      10.0.12.2/30       10     P2P     1/1
⚠️ Constat : FastEthernet1/0 sur R2 est configurée en 10.0.23.2, créant un conflit avec l'IP réservée pour R3 sur le sous-réseau 10.0.23.0/30.

🔧 Action corrective (R2)
Plaintext
R2(config)# interface FastEthernet1/0
R2(config-if)# ip address 10.0.23.1 255.255.255.252
R2(config-if)# no shutdown
R2(config-if)# end
🔍 Vérification du routage (R2)
Plaintext
R2# traceroute 3.3.3.3
Type escape sequence to abort.
Tracing the route to 3.3.3.3
VRF info: (vrf in name/id, vrf out name/id)
  1 10.0.12.1 24 msec 20 msec 16 msec
  2 10.0.13.2 44 msec 32 msec *
💡 Analyse : Le lien direct R2-R3 reste inactif. Le flux vers 3.3.3.3 est contraint de transiter via R1 (10.0.12.1).

❌ Incident 3 : Interface L2/L3 non configurée sur R3
Diagnostic des interfaces sur R3 :

Plaintext
R3> show ip interface brief
Interface          IP-Address      OK? Method Status                  Protocol
FastEthernet0/0    10.0.13.2       YES manual up                      up
FastEthernet1/0    unassigned      YES NVRAM  up                      up
Loopback0          3.3.3.3         YES NVRAM  up                      up
⚠️ Constat : L'interface FastEthernet1/0 de R3 est physiquement up/up mais n'a aucune adresse IP (unassigned).

🔧 Action corrective (R3)
Plaintext
R3> enable
R3# configure terminal
R3(config)# interface FastEthernet1/0
R3(config-if)# ip address 10.0.23.2 255.255.255.252
R3(config-if)# no shutdown
R3(config-if)# exit
R3(config)# router ospf 1
R3(config-router)# network 10.0.23.0 0.0.0.3 area 0
R3(config-router)# end
✅ Notification d'adjacence OSPF
Plaintext
%OSPF-5-ADJCHG: Process 1, Nbr 2.2.2.2 on FastEthernet1/0 from LOADING to FULL
🤝 3. Validation de la Convergence & Voisinage OSPF
Validation de l'établissement complet de la grille d'adjacence OSPF sur les trois équipements de la topologie.

🌐 Voisins OSPF sur R1 (1.1.1.1)
Voisinages établis avec R2 (2.2.2.2) et R3 (3.3.3.3).

🌐 Voisins OSPF sur R2 (2.2.2.2)
Voisinages établis avec R1 (1.1.1.1) et R3 (3.3.3.3).

🌐 Voisins OSPF sur R3 (3.3.3.3)
Voisinages établis avec R1 (1.1.1.1) et R2 (2.2.2.2).

🧪 4. Validation du Routage Dynamique & Basculement (TP 1.2)
📊 Calcul de Métrique & Analyse Asymétrique
La bande passante du lien direct R1-R3 (Fa1/0) a été restreinte sur R1 via bandwidth 10000.

Coût direct R1 ➔ R3 : (10^8 / 10^7) = 100 + 1 = 101

Coût alternatif via R2 : 10 (R1 -> R2) + 10 (R2 -> R3) + 1 = 21

L'algorithme SPF sur R1 choisit automatiquement d'emprunter le chemin via R2.

🔍 Métrique sur R1 (FastEthernet1/0)
Le coût passe à 100 car la bande passante a été abaissée (bandwidth 10000).

🔍 Métrique sur R2 (FastEthernet1/0)
Le coût reste à sa valeur par défaut de 10 (bandwidth 100000).

🔍 Métrique sur R3 (FastEthernet1/0)
Le coût reste à sa valeur par défaut de 10 (bandwidth 100000).

🚀 Test de Reroutage Dynamique (Traceroute R1 ➔ R3)
Plaintext
R1# traceroute 3.3.3.3
✅ Résultat : Le trafic contourne le lien direct à coût élevé et transite bien par R2 (10.0.12.2) avant d'atteindre R3 (10.0.23.2).

🔍 5. Inspection LSDB & Debug OSPF (TP 1.3)
📋 Inspection de la base de données LSDB (Router LSA Type 1)
Commandes d'extraction complètes de la table LSDB sur R1 (show ip ospf database router) :

Part 1 : LSDB Overview & Router LSA 1.1.1.1
Part 2 : Router LSA 1.1.1.1 (suite) & 2.2.2.2
Part 3 : Router LSA 2.2.2.2 (suite) & 3.3.3.3
Part 4 : Router LSA 3.3.3.3 (fin des liens Point-to-Point & Stub)
🛰️ Debug des paquets Keepalive (Hello)
Capture de l'émission et de la réception des paquets Keepalive OSPF sur l'adresse multicast 224.0.0.5 à intervalle régulier de 10 secondes.

Plaintext
R1# debug ip ospf hello
Plaintext
R1# undebug all
All possible debugging has been turned off
