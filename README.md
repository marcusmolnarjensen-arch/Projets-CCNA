# TP  Configuration, Troubleshooting & Optimisation OSPF (Area 0)

![Cisco IOS](https://img.shields.io/badge/Cisco-IOS-blue?style=flat-square&logo=cisco)
![Protocol](https://img.shields.io/badge/Protocol-OSPFv2-green?style=flat-square)
![Topology](https://img.shields.io/badge/Topology-Triangle-orange?style=flat-square)

---

## 1. Contexte & Architecture

Maquettage d'une interconnexion en triangle de **3 routeurs Cisco (R1, R2, R3)** au sein d'une aire OSPF unique (**Area 0**).

### Objectifs
* Corriger les erreurs d'adressage IP et réétablir la convergence OSPF globale.
* Manipuler le coût OSPF via la bande passante (`bandwidth`) pour forcer le basculement dynamique des routes.
* Inspecter la base de données LSDB (LSA Type 1) et valider le maintien des adjacences (paquets Hello).

---

## 2. Chronologie du Troubleshooting

### Incident 1 : Voisinage OSPF absent entre R2 et R3

**Diagnostic sur R2 :**
```text
R2> show ip ospf neighbor
PlaintextNeighbor ID     Pri   State           Dead Time   Address         Interface
1.1.1.1           0   FULL/  -        00:00:37    10.0.12.1       FastEthernet0/0
Constat : R2 ne forme aucune adjacence avec R3. Seul R1 (1.1.1.1) est reconnu.❌ Incident 2 : Conflit d'adressage IP sur le segment R2-R3Diagnostic des interfaces sur R2 :PlaintextR2# show ip interface brief
R2# show ip ospf interface brief
PlaintextInterface          IP-Address      OK? Method Status                  Protocol
FastEthernet0/0    10.0.12.2       YES NVRAM  up                      up
FastEthernet1/0    10.0.23.2       YES NVRAM  up                      up
Loopback0          2.2.2.2         YES NVRAM  up                      up

Interface    PID   Area   IP Address/Mask    Cost   State   Nbrs F/C
Lo0          1     0      2.2.2.2/32         1      LOOP    0/0
Fa1/0        1     0      10.0.23.2/30       10     P2P     0/0
Fa0/0        1     0      10.0.12.2/30       10     P2P     1/1
Constat : FastEthernet1/0 sur R2 est configurée en 10.0.23.2, créant un conflit avec l'IP réservée pour R3 sur le sous-réseau 10.0.23.0/30.🔧 Action corrective (R2)PlaintextR2(config)# interface FastEthernet1/0
R2(config-if)# ip address 10.0.23.1 255.255.255.252
R2(config-if)# no shutdown
R2(config-if)# end
Vérification du routage (R2)PlaintextR2# traceroute 3.3.3.3
PlaintextType escape sequence to abort.
Tracing the route to 3.3.3.3
VRF info: (vrf in name/id, vrf out name/id)
  1 10.0.12.1 24 msec 20 msec 16 msec
  2 10.0.13.2 44 msec 32 msec *
Analyse : Le lien direct R2-R3 reste inactif. Le flux vers 3.3.3.3 est contraint de transit via R1 (10.0.12.1).❌ Incident 3 : Interface L2/L3 non configurée sur R3Diagnostic des interfaces sur R3 :PlaintextR3> show ip interface brief
PlaintextInterface          IP-Address      OK? Method Status                  Protocol
FastEthernet0/0    10.0.13.2       YES manual up                      up
FastEthernet1/0    unassigned      YES NVRAM  up                      up
Loopback0          3.3.3.3         YES NVRAM  up                      up
Constat : L'interface FastEthernet1/0 de R3 est physiquement up/up mais n'a aucune adresse IP (unassigned).🔧 Action corrective (R3)PlaintextR3> enable
R3# configure terminal
R3(config)# interface FastEthernet1/0
R3(config-if)# ip address 10.0.23.2 255.255.255.252
R3(config-if)# no shutdown
R3(config-if)# exit
R3(config)# router ospf 1
R3(config-router)# network 10.0.23.0 0.0.0.3 area 0
R3(config-router)# end
Notification d'adjacence OSPFPlaintext%OSPF-5-ADJCHG: Process 1, Nbr 2.2.2.2 on FastEthernet1/0 from LOADING to FULL
3. Validation du Routage Dynamique & Basculement (TP 1.2) Calcul de MétriqueLa bande passante du lien direct R1-R3 (Fa1/0) a été restreinte à bandwidth 10000.Coût direct R1 ➔ R3 : $\frac{10^8}{10^7} = 100 + 1 = 101$Coût alternatif via R2 : $10 (R1 \rightarrow R2) + 10 (R2 \rightarrow R3) + 1 = 21$L'algorithme SPF choisit automatiquement le chemin via R2. Test de vérification (R1)PlaintextR1# traceroute 3.3.3.3
PlaintextType escape sequence to abort.
Tracing the route to 3.3.3.3
VRF info: (vrf in name/id, vrf out name/id)
  1 10.0.12.2 60 msec 4 msec 44 msec
  2 10.0.23.2 24 msec 16 msec *
 Résultat : Le trafic contourne le lien dégradé et bascule sur 10.0.12.2 (R2) puis 10.0.23.2 (R3). 4. Inspection LSDB & Debug OSPF (TP 1.3) Router LSA (Type 1) — R1PlaintextR1# show ip ospf database router
Plaintext            OSPF Router with ID (1.1.1.1) (Process ID 1)

            Router Link States (Area 0)

  LS age: 657
  Options: (No TOS-capability, DC)
  LS Type: Router Links
  Link State ID: 1.1.1.1
  Advertising Router: 1.1.1.1
  LS Seq Number: 80000006
  Checksum: 0x3D60
  Length: 84
  Number of Links: 5

      Link connected to: a Stub Network
       (Link ID) Network/subnet number: 1.1.1.1
       (Link Data) Network Mask: 255.255.255.255
        Number of MTID metrics: 0
         TOS 0 Metrics: 1

      Link connected to: another Router (point-to-point)
       (Link ID) Neighboring Router ID: 3.3.3.3
Debug des paquets Keepalive (Hello)PlaintextR1# debug ip ospf hello
Validation de l'émission/réception des paquets Keepalive sur 224.0.0.5 toutes les 10 secondes.PlaintextR1# undebug all
Clôture du processus de debug.


test R1 show ip ospf neighbor
<img width="1047" height="181" alt="image" src="https://github.com/user-attachments/assets/b4106ad1-c485-4aeb-b50a-cf55c77d8a96" />
2.2.2.2 et 3.3.3.3
test R2 show ip ospf neighbor
<img width="1047" height="181" alt="image" src="https://github.com/user-attachments/assets/ca25fc3b-75b1-41b8-ae2d-08c9c047c3ce" />
1.1.1.1 et 3.3.3.3
test R3 show ip ospf neighbor
<img width="1007" height="175" alt="image" src="https://github.com/user-attachments/assets/9f4c4f11-5efe-416a-823a-51d82cebc206" />
1.1.1.1 et 2.2.2.2


