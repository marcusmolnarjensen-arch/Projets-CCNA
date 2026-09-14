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
