### Module 1

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

### Module 2
