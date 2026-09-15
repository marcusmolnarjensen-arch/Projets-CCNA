Documentation Technique — OSPFv2 (Modules 1 & 2)
Topologie de Réseau
Module 1 — Validation de la Connectivité et Comportement OSPF
1. Adjacences et Voisinage (show ip ospf neighbor)
Vérification des adjacences OSPF à l'état FULL sur l'ensemble des routeurs du domaine.

R1 :

Adjacences établies avec R2 (2.2.2.2) et R3 (3.3.3.3).

R2 :

Adjacences établies avec R1 (1.1.1.1) et R3 (3.3.3.3).

R3 :

Adjacences établies avec R1 (1.1.1.1) et R2 (2.2.2.2).

2. Coût des Interfaces (show ip ospf interface FastEthernet1/0 | include Cost)
R1 :

Coût ajusté à 100 en raison de la bande passante modifiée à 10 000 Kbit/s.

R2 :

Coût par défaut à 10 avec une bande passante FastEthernet (100 000 Kbit/s).

R3 :

Coût par défaut à 10 avec une bande passante FastEthernet (100 000 Kbit/s).

3. Reroutage Dynamique (traceroute 3.3.3.3 depuis R1)
Validation du calcul SPF : le trafic depuis R1 transite par R2 (10.0.12.2) pour atteindre R3 au lieu d'emprunter le lien direct à coût élevé.

4. Inspection de la LSDB (show ip ospf database router sur R1)
Base de données LSA Router (Partie 1) :

Base de données LSA Router (Partie 2) :

Base de données LSA Router (Partie 3) :

Base de données LSA Router (Partie 4) :

5. Analyse Trame & Captures Wireshark
Debug des paquets Hello (debug ip ospf hello) :

Wireshark Fa0/0 (Lien R1-R2) :

Wireshark Fa1/0 (Lien R1-R3) :

Module 2 — Optimisation & Sécurisation OSPFv2
1. Élection DR/BDR & Priorités (show ip ospf neighbor)
Validation de la hiérarchie DR/BDR sur le segment multi-accès :

R1 (1.1.1.1) : Élu DR (Priorité OSPF 255).

R2 (2.2.2.2) : Élu BDR (Priorité OSPF 100).

R3 (3.3.3.3) : Rôle DROTHER (Priorité OSPF 0).

2. Interfaces Passives — TP 2.1 (show ip protocols)
Validation du paramètre passive-interface FastEthernet1/0 sur R1.

Blocage de l'émission/réception des paquets Hello vers le LAN utilisateur, interdisant la formation d'adjacence non sollicitée tout en garantissant la propagation du sous-réseau dans la LSDB.

3. Propagation de Route par Défaut — TP 2.2 (show ip route ospf)
Injection centralisée de la route statique 0.0.0.0/0 depuis R1 via la commande default-information originate always.

Confirmation de la réception sur R2 et R3 sous la forme d'une route externe O*E2 avec mise à jour automatique du Gateway of last resort vers 10.0.0.1.

4. Authentification MD5 du Backbone — TP 2.3 (show ip ospf interface)
Durcissement de la liaison backbone FastEthernet0/0 via la clé MD5 CCNA-Satom2026!.

Validation par le statut Message digest authentication enabled et maintien des adjacences de voisinage à l'état FULL.
