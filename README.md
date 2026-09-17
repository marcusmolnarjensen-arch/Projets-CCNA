Rapport Technique Global — Lab CCNA : OSPFv2, Hardening, L2 Security, ACL & NAT/PATSociété / École : Satom IT & Learning Solutions / Geneva Institute of TechnologyAuteur : Marcus Jensen MolnarEnvironnement : PNetLab / GNS3 — Images : Cisco IOSv (routeurs), IOSvL2 (switches Catalyst)📘 Module 1 — Concepts de l'OSPF à zone uniqueObjectifs & PrérequisComprehension du protocole IGP à état de liens (Dijkstra/SPF), des états d'adjacence (Down → Init → 2-Way → ExStart → Exchange → Loading → Full) et du calcul de coût ($\text{Coût} = \frac{\text{Bande passante de référence}}{\text{Bande passante interface}}$).Prérequis : 3 routeurs Cisco interconnectés en triangle, notation wildcard mask.TP 1.1 — Découverte du voisinage OSPF et des états d'adjacenceArchitecture & Plan d'Adressage IPÉquipementInterfaceAdresse IP / MasqueRéseau AssociéR1FastEthernet0/010.0.12.1/3010.0.12.0/30 (Lien R1-R2)R1FastEthernet1/010.0.13.1/3010.0.13.0/30 (Lien R1-R3)R1Loopback01.1.1.1/32Router IDR2FastEthernet0/010.0.12.2/3010.0.12.0/30 (Lien R1-R2)R2FastEthernet1/010.0.23.2/3010.0.23.0/30 (Lien R2-R3)R2Loopback02.2.2.2/32Router IDR3FastEthernet0/010.0.13.3/3010.0.13.0/30 (Lien R1-R3)R3FastEthernet1/010.0.23.3/3010.0.23.0/30 (Lien R2-R3)R3Loopback03.3.3.3/32Router IDConfiguration CLI (Exemple R1)PlaintextR1(config)# interface loopback 0
R1(config-if)# ip address 1.1.1.1 255.255.255.255
R1(config-if)# exit
R1(config)# router ospf 1
R1(config-router)# router-id 1.1.1.1
R1(config-router)# network 10.0.12.0 0.0.0.3 area 0
R1(config-router)# network 10.0.13.0 0.0.0.3 area 0
R1(config-router)# network 1.1.1.1 0.0.0.0 area 0
R1(config-router)# exit
R1# debug ip ospf adj
Rôle des Commandes KeyCommandeRôle Techniqueinterface loopback 0Interface virtuelle toujours UP, garantissant la stabilité du Router ID.router-id 1.1.1.1Force l'identifiant OSPF du routeur de manière explicite.network 10.0.12.0 0.0.0.3 area 0Active OSPF en Zone 0 sur l'interface correspondant au wildcard mask.debug ip ospf adjTrace en temps réel les transitions d'états d'adjacence OSPF.Preuves de ValidationPlaintextR1# show ip ospf neighbor
Neighbor ID     Pri   State           Dead Time   Address         Interface
2.2.2.2           1   FULL/  -        00:00:35    10.0.12.2       FastEthernet0/0
3.3.3.3           1   FULL/  -        00:00:31    10.0.13.3       FastEthernet1/0

R2# show ip ospf neighbor
Neighbor ID 1.1.1.1 et 3.3.3.3 validés en état FULL.
R3# show ip ospf neighbor
Neighbor ID 1.1.1.1 et 2.2.2.2 validés en état FULL.
TP 1.2 — Calcul et manipulation de la métrique OSPF (Coût)Configuration du Coût et RéférencePlaintextR1(config)# interface FastEthernet1/0
R1(config-if)# bandwidth 10000
R1(config-if)# exit
R1(config)# router ospf 1
R1(config-router)# auto-cost reference-bandwidth 1000
Rôle des Commandes KeyCommandeRôle Techniquebandwidth 10000Modifie la bande passante logique (10 Mbps) pour impacter le calcul de coût OSPF.auto-cost reference-bandwidth 1000Réajuste la référence à 1 Gbps pour différencier les liens FastEthernet/Gigabit.Preuves de ValidationPlaintextR1# show ip ospf interface FastEthernet1/0 | include Cost
Cost: 100 (Bande passante à 10000 Kbit)

R2# show ip ospf interface FastEthernet0/0 | include Cost
Cost: 10 (Bande passante à 100000 Kbit par défaut)

R1# traceroute 3.3.3.3
Type escape sequence to abort.
1 10.0.12.2 16 ms 20 ms 16 ms
2 10.0.23.3 24 ms 22 ms 25 ms
Analyse : Le flux contourne le lien direct R1-R3 (Coût 100) et transit par R2 (Coût total $10 + 10 = 20$).TP 1.3 — Analyse des paquets OSPF (LSDB & Wireshark)Commandes d'InspectionPlaintextR1# debug ip ospf hello
R1# show ip ospf database
R1# show ip ospf database router
Validation de la RecettePaquets Hello : Fréquence observée de 10s (Dead interval à 40s sur réseau Broadcast Ethernet).Captures Wireshark :Wireshark Fa0/0 (R1-R2) : Échanges Hello, DBD, LSR, LSU, LSAck validés.Wireshark Fa1/0 (R1-R3) : Validation du maintien des paquets Keepalive OSPF.📗 Module 2 — Configuration OSPFv2 à zone unique & HardeningNote d'Architecture : Basculement de la topologie PNetLab sur un Backbone centralisé via Switch Catalyst L2 (SW1) reliant R1, R2 et R3 sur le sous-réseau 10.0.0.0/24.Topologie & Plan d'Adressage BackboneRéseau Inter-Routeurs : 10.0.0.0/24R1 (Primary DR) : Loopback 1.1.1.1/32 | Priority 255R2 (BDR) : Loopback 2.2.2.2/32 | Priority 100R3 (DROTHER) : Loopback 3.3.3.3/32 | Priority 1Configurations Intégrales des Routeurs & SwitchRouteur R1 (Primary DR)Plaintexthostname R1
ip domain-name satom.local
username SATOMIT privilege 15 secret admin123
enable secret admin123

interface Loopback0
 ip address 1.1.1.1 255.255.255.255
 exit

interface FastEthernet0/0
 ip address 10.0.0.1 255.255.255.0
 ip ospf priority 255
 ip ospf message-digest-key 1 md5 CCNA-Satom2026!
 ip ospf authentication message-digest
 duplex full
 no shutdown
 exit

router ospf 1
 router-id 1.1.1.1
 network 10.0.0.0 0.0.0.255 area 0
 network 1.1.1.0 0.0.0.255 area 0
 default-information originate
 exit

crypto key generate rsa modulus 2048
ip ssh version 2

line vty 0 4
 transport input ssh
 login local
 exec-timeout 5 0
 exit
Routeur R2 (BDR)Plaintexthostname R2
ip domain-name satom.local
username SATOMIT privilege 15 secret admin123
enable secret admin123

interface Loopback0
 ip address 2.2.2.2 255.255.255.255
 exit

interface FastEthernet0/0
 ip address 10.0.0.2 255.255.255.0
 ip ospf priority 100
 ip ospf message-digest-key 1 md5 CCNA-Satom2026!
 ip ospf authentication message-digest
 duplex full
 no shutdown
 exit

router ospf 1
 router-id 2.2.2.2
 network 10.0.0.0 0.0.0.255 area 0
 network 2.2.2.0 0.0.0.255 area 0
 exit

crypto key generate rsa modulus 2048
ip ssh version 2

line vty 0 4
 transport input ssh
 login local
 exec-timeout 5 0
 exit
Routeur R3 (DROTHER)Plaintexthostname R3
ip domain-name satom.local
username SATOMIT privilege 15 secret admin123
enable secret admin123

interface Loopback0
 ip address 3.3.3.3 255.255.255.255
 exit

interface FastEthernet0/0
 ip address 10.0.0.3 255.255.255.0
 ip ospf priority 1
 ip ospf message-digest-key 1 md5 CCNA-Satom2026!
 ip ospf authentication message-digest
 duplex full
 no shutdown
 exit

router ospf 1
 router-id 3.3.3.3
 network 10.0.0.0 0.0.0.255 area 0
 network 3.3.3.3 0.0.0.0 area 0
 exit

crypto key generate rsa modulus 2048
ip ssh version 2

line vty 0 4
 transport input ssh
 login local
 exec-timeout 5 0
 exit
Switch SW1 (Commutateur d'Accès Backbone)Plaintexthostname SW1
ip domain-name satom.local
enable secret admin123

interface range FastEthernet1/0 - 15
 switchport mode access
 speed 100
 duplex full
 no shutdown
 exit

line vty 0 4
 password admin123
 login
 exit
TP 2.1 à 2.3 — Avancés OSPF & Preuves de Recette1. Interfaces Passives & Default Route InjectionPlaintextR1(config-router)# passive-interface FastEthernet0/1
R1(config)# ip route 0.0.0.0 0.0.0.0 203.0.113.254
R1(config-router)# default-information originate
passive-interface : Empêche l'envoi de paquets Hello sur les LAN utilisateurs tout en annonçant le réseau.default-information originate : Injecte la route 0.0.0.0/0 (LSA Type 5 External) à travers tout le domaine OSPF.2. Authentification MD5 OSPFPlaintextR1(config-if)# ip ospf message-digest-key 1 md5 CCNA-Satom2026!
R1(config-if)# ip ospf authentication message-digest
Chiffre les paquets Hello et mises à jour LSA pour contrer l'injection rogue.3. Preuves CLIPlaintextR1# show ip ospf neighbor
Neighbor ID     Pri   State           Dead Time   Address         Interface
2.2.2.2         100   FULL/BDR        00:00:18    10.0.0.2        FastEthernet0/0
3.3.3.3           1   FULL/DROTHER    00:00:16    10.0.0.3        FastEthernet0/0

R1# show ip route ospf
O        2.2.2.2 [110/2] via 10.0.0.2, 00:10:20, FastEthernet0/0
O        3.3.3.3 [110/2] via 10.0.0.3, 00:10:30, FastEthernet0/0

R3# ping 1.1.1.1 source loopback0
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 1.1.1.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 16/20/24 ms

R1# show ip ssh
SSH Enabled - version 2.0
Authentication timeout: 120 secs; Authentication retries: 3
📕 Module 3 — Concepts de sécurité réseau & L2 HardeningModification d'infrastructure : En raison de limitations logicielles sur les modules routeurs NM-16ESW, le composant c3745 a été remplacé par un Switch dédié Cisco Catalyst IOSvL2 pour exécuter Port Security et DHCP Snooping sans restriction d'IOS.TP 3.1 — Durcissement Administratif (AAA local + SSHv2)Configuration R1 / SW1PlaintextR1(config)# username SATOMIT privilege 15 secret Adm1n#2026
R1(config)# ip domain-name satom.local
R1(config)# crypto key generate rsa modulus 2048
R1(config)# ip ssh version 2
R1(config)# line vty 0 4
R1(config-line)# transport input ssh
R1(config-line)# login local
R1(config-line)# exec-timeout 5 0
TP 3.2 — Port Security sur Switch d'Accès (SW1)ConfigurationPlaintextSW1(config)# interface FastEthernet0/1
SW1(config-if)# switchport mode access
SW1(config-if)# switchport port-security
SW1(config-if)# switchport port-security maximum 1
SW1(config-if)# switchport port-security mac-address sticky
SW1(config-if)# switchport port-security violation shutdown
Procédure de Recovery en cas de Violation (err-disabled)PlaintextSW1# show interfaces FastEthernet0/1 status
Port      Name               Status       Vlan       Duplex  Speed Type
Fa0/1                        err-disabled 1          full    100   10/100BaseTX

SW1(config)# interface FastEthernet0/1
SW1(config-if)# shutdown
SW1(config-if)# no shutdown
TP 3.3 — Mitigation DHCP Rogue via DHCP SnoopingConfiguration SW1PlaintextSW1(config)# ip dhcp snooping
SW1(config)# ip dhcp snooping vlan 10
SW1(config)# interface FastEthernet0/24
SW1(config-if)# ip dhcp snooping trust
SW1(config)# interface range FastEthernet0/1 - 2
SW1(config-if-range)# ip dhcp snooping limit rate 10
Ports Non-Fiables (Untrusted) : Bloquent les réponses DHCP OFFER et ACK non autorisées.Port Fiable (Trusted - Fa0/24) : Autorise le relais du serveur DHCP d'entreprise.📙 Module 4 — Concepts de liste de contrôle d'accès (ACL)TP 4.1 — Calculs de Wildcard MasksCible / Sous-réseauMasque Sous-RéseauWildcard Mask Calculé192.168.10.0/24255.255.255.00.0.0.255192.168.20.0/26255.255.255.1920.0.0.6310.0.0.0/16255.255.0.00.0.255.255Hôte unique 192.168.30.5255.255.255.2550.0.0.0 (ou mot-clé host)Tout le trafic (any)0.0.0.0255.255.255.255 (ou mot-clé any)TP 4.2 — Ordre d'Évaluation & Masquage ImpliciteAnalyse de PannePlaintext! ACL erronée (Bloque TOUT le trafic par le deny any implicite)
access-list 10 deny 192.168.20.0 0.0.0.255

! ACL Corrigée
access-list 10 deny 192.168.20.0 0.0.0.255
access-list 10 permit any
Principe : Traitement séquentiel ("First Match"). La ligne permit any est obligatoire pour neutraliser le deny ip any any implicite situé en fin d'ACL.TP 4.3 — Choix de Placement : Standard vs ÉtendueACL Standard (1-99 / Nommée) : Filtre uniquement sur l'adresse source. Règle : Placer au plus près de la destination.ACL Étendue (100-199 / Nommée) : Filtre sur source, destination, protocole et ports. Règle : Placer au plus près de la source.📙 Module 5 — Configuration ACL IPv4, NAT/PAT & Traces Debug CLITP 5.1 & 5.2 — Déploiement des ACLs & PAT Overload sur R1Configuration R1Plaintextip access-list standard ADMIN-ONLY
 permit host 192.168.1.10
 exit

line vty 0 4
 access-class ADMIN-ONLY in
 exec-timeout 5 0
 login local
 transport input ssh
 exit

ip access-list standard TOUT-INTERNE
 permit 192.168.0.0 0.0.255.255
 exit

ip access-list extended INVITES-WEB-ONLY
 permit tcp 192.168.99.0 0.0.0.255 any eq www
 permit tcp 192.168.99.0 0.0.0.255 any eq 443
 permit udp 192.168.99.0 0.0.0.255 any eq domain
 deny ip 192.168.99.0 0.0.0.255 any
 exit

interface FastEthernet1/0
 ip access-group INVITES-WEB-ONLY in
 ip nat outside
 exit

interface FastEthernet0/0
 duplex full
 ip nat inside
 exit

ip nat inside source list TOUT-INTERNE interface FastEthernet1/0 overload
Dépannage, Recette Réelle & Analyse des Traces Logs1. Resolution du Duplex Mismatch (Fa0/0)Plaintext*Sep 17 12:34:43.855: %CDP-4-DUPLEX_MISMATCH: duplex mismatch discovered on FastEthernet0/0 (not half duplex), with SW1.satom.local Ethernet0/1 (half duplex).
Correction appliquée : Passage explicite de FastEthernet0/0 en duplex full pour correspondre à la configuration globale des ports de SW1.2. Reconfiguration Dynamique du NAT (PAT Overload)Plaintext*Sep 17 12:35:54.175: ipnat_remove_static_cfg: id 1, flag A
*Sep 17 12:35:54.207: ipnat_remove_dynamic_cfg: id 1, flag 9, range 0
*Sep 17 12:35:54.211: ipnat_add_dynamic_cfg_common: id 1, flag 11, range 0
*Sep 17 12:35:54.211: id 1, flags 0, domain 0, lookup 0, aclnum 0, aclname TOUT-INTERNE, mapname idb 0x68EB0230
Analyse Trace : Validation CLI du démontage du NAT statique obsolète au profit de la translation dynamique PAT sur FastEthernet1/0 avec l'ACL TOUT-INTERNE.3. Traitement du Control Plane OSPF sous Debug PacketPlaintext*Sep 17 12:35:53.643: IP: s=10.0.0.2 (FastEthernet0/0), d=224.0.0.5, len 124, input feature, proto=89, Common Flow Table(5)
*Sep 17 12:35:53.647: IP: s=10.0.0.2 (FastEthernet0/0), d=224.0.0.5, len 124, rcvd 0, proto=89
Analyse Trace : Interception et validation des paquets Hello OSPF (Protocole IP 89, Adresse Multicast AllSPFRouters 224.0.0.5) émis par le voisin R2 (10.0.0.2), confirmant le non-blocage des flux de contrôle par les règles NAT/ACL.1.Copier le rapport complet:Export du document.Récupère le texte Markdown ci-dessus pour le coller directement dans ton rapport d'évaluation HackMD ou Word.2.Valider la NVRAM:Sauvegarde Lab.Exécute write memory sur R1, R2, R3 et SW1 pour figer la maquette PNetLab.
