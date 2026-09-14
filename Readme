RAPPORT TECHNIQUE DE TP : CONFIGURATION, TROUBLESHOOTING ET OPTIMISATION OSPF AREA 01. CONTEXTE ET OBJECTIFSMaquettage : Interconnexion en triangle de 3 routeurs Cisco (R1, R2, R3) dans une aire OSPF unique (Area 0).Objectif : Valider la convergence OSPF, forcer un basculement de route dynamique en modifiant la bande passante (bandwidth), puis documenter les étapes de troubleshooting réseau liées aux erreurs d'adressage et d'interfaces.2. CHRONOLOGIE DU TROUBLESHOOTING ET ANOMALIES RENCONTRÉESIncident 1 : Absence d'adjacence OSPF entre R2 et R3Commande de diagnostic (R2) :PlaintextR2>show ip ospf neighbor
Output obtenu :PlaintextNeighbor ID     Pri   State           Dead Time   Address         Interface
1.1.1.1           0   FULL/  -        00:00:37    10.0.12.1       FastEthernet0/0
Constat : R2 ne voit que R1 (1.1.1.1). Le lien de voisinage OSPF avec R3 est totalement absent.Incident 2 : Conflit d'adresse IP sur le segment R2-R3Commande de diagnostic (R2) :PlaintextR2#show ip interface brief
R2#show ip ospf interface brief
Output obtenu (R2) :PlaintextInterface          IP-Address      OK? Method Status                  Protocol
FastEthernet0/0    10.0.12.2       YES NVRAM  up                      up
FastEthernet1/0    10.0.23.2       YES NVRAM  up                      up
Loopback0          2.2.2.2         YES NVRAM  up                      up

Interface    PID   Area   IP Address/Mask    Cost   State   Nbrs F/C
Lo0          1     0      2.2.2.2/32         1      LOOP    0/0
Fa1/0        1     0      10.0.23.2/30       10     P2P     0/0
Fa0/0        1     0      10.0.12.2/30       10     P2P     1/1
Constat : R2 possède l'IP 10.0.23.2 sur Fa1/0, qui est l'adresse attribuée à R3. Conflit d'adressage IP sur le lien point-à-point 10.0.23.0/30.Action corrective 1 (R2) : Reconfiguration de l'interface en .1.PlaintextR2(config)# interface FastEthernet1/0
R2(config-if)# ip address 10.0.23.1 255.255.255.252
R2(config-if)# no shutdown
R2(config-if)# end
Test de vérification post-correction 1 (R2) :PlaintextR2#show ip ospf neighbor
Résultat : Toujours aucun voisin OSPF sur FastEthernet1/0.Test de routage (R2) :PlaintextR2#traceroute 3.3.3.3
Type escape sequence to abort.
Tracing the route to 3.3.3.3
VRF info: (vrf in name/id, vrf out name/id)
  1 10.0.12.1 24 msec 20 msec 16 msec
  2 10.0.13.2 44 msec 32 msec *
Analyse : Pour joindre R3 (3.3.3.3), R2 doit traverser R1 (10.0.12.1). Le lien direct R2-R3 ne transmet pas le trafic.Incident 3 : Interface L2/L3 non configurée sur R3Commande de diagnostic (R3) :PlaintextR3>show ip interface brief
Output obtenu (R3) :PlaintextInterface          IP-Address      OK? Method Status                  Protocol
FastEthernet0/0    10.0.13.2       YES manual up                      up
FastEthernet1/0    unassigned      YES NVRAM  up                      up
Loopback0          3.3.3.3         YES NVRAM  up                      up
Constat : L'interface FastEthernet1/0 de R3 est à l'état up/up mais aucune adresse IP ne lui est assignée (unassigned).Action corrective 2 (R3) : Assignation de l'adresse 10.0.23.2/30 et activation dans le processus OSPF 1.PlaintextR3>enable
R3#configure terminal
R3(config)#interface FastEthernet1/0
R3(config-if)#ip address 10.0.23.2 255.255.255.252
R3(config-if)#no shutdown
R3(config-if)#exit
R3(config)#router ospf 1
R3(config-router)#network 10.0.23.0 0.0.0.3 area 0
R3(config-router)#end
3. TESTS ET VALIDATION FINALE DU BASCULEMENT DE TRAFIC (TP 1.2)Une fois l'adjacence R2-R3 établie, l'algorithme SPF recalcule la meilleure métrique vers 3.3.3.3.Mécanisme : La bande passante du lien direct R1-R3 (Fa1/0) a été restreinte à bandwidth 10000 (Coût OSPF = $\frac{10^8}{10^7} = 100 + 1 = 101$). Le chemin alternatif via R2 présente un coût total inférieur ($10 + 10 + 1 = 21$).Commande de validation du basculement (R1) :PlaintextR1#traceroute 3.3.3.3
Output validé :PlaintextType escape sequence to abort.
Tracing the route to 3.3.3.3
VRF info: (vrf in name/id, vrf out name/id)
  1 10.0.12.2 60 msec 4 msec 44 msec
  2 10.0.23.2 24 msec 16 msec *
Conclusion du test : Le trafic contourne le lien direct et passe par R2 (10.0.12.2) puis R3 (10.0.23.2). Le reroutage dynamique est validé.4. INSPECTION DE LA BASE DE DONNÉES OSPF / LSDB (TP 1.3)Commande d'inspection (R1) :PlaintextR1#show ip ospf database router
Output de preuve (Router LSA Type 1 - R1) :Plaintext          OSPF Router with ID (1.1.1.1) (Process ID 1)

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
Analyse LSDB : Le Router LSA émis par 1.1.1.1 annonce ses 5 liens locaux dans l'Area 0, incluant son réseau Stub (Loopback 1.1.1.1/32) et ses liaisons point-à-point vers ses voisins.
