test R1 show ip ospf neighbor :

<img width="1047" height="181" alt="image" src="https://github.com/user-attachments/assets/b4106ad1-c485-4aeb-b50a-cf55c77d8a96" />

2.2.2.2 et 3.3.3.3

test R2 show ip ospf neighbor :

<img width="1047" height="181" alt="image" src="https://github.com/user-attachments/assets/ca25fc3b-75b1-41b8-ae2d-08c9c047c3ce" />

1.1.1.1 et 3.3.3.3

test R3 show ip ospf neighbor :

<img width="1007" height="175" alt="image" src="https://github.com/user-attachments/assets/9f4c4f11-5efe-416a-823a-51d82cebc206" />

1.1.1.1 et 2.2.2.2



R1 test show ip ospf interface FastEthernet1/0 | include Cost :

<img width="902" height="73" alt="image" src="https://github.com/user-attachments/assets/d19cee6a-248a-4117-b28e-18bd5bc540e5" />

100 car le bandwidth est a 1000 

R2 test show ip ospf interface FastEthernet1/0 | include Cost :

<img width="878" height="71" alt="image" src="https://github.com/user-attachments/assets/aa109af9-9e8e-4421-881e-9b271a707270" />

10 car le bandwidth est a 100

R3 test show ip ospf interface FastEthernet1/0 | include Cost :

<img width="868" height="76" alt="image" src="https://github.com/user-attachments/assets/0f40e483-5d2b-47dc-a576-f844f7ec7e4d" />

10 car le bandwidth est a 100 



R1 traceroute vers R3 :

<img width="521" height="145" alt="image" src="https://github.com/user-attachments/assets/a820a170-f0f5-471e-ad77-be4b39d85c10" />

il passe bien par R2



R1 test show ip ospf database router :

<img width="695" height="802" alt="image" src="https://github.com/user-attachments/assets/78969ec0-762f-4799-b98e-d4bf1065c13e" />

<img width="697" height="817" alt="image" src="https://github.com/user-attachments/assets/b3717b48-f871-4e09-8adb-10f07d1eb1cf" />

<img width="687" height="818" alt="image" src="https://github.com/user-attachments/assets/75c3d989-51f0-4729-b28e-53f712fdf9de" />

<img width="673" height="641" alt="image" src="https://github.com/user-attachments/assets/1404c6da-aaab-4b5f-bdbf-de306c47cf1f" />





R1 test debug ip ospf hello :

<img width="1112" height="492" alt="image" src="https://github.com/user-attachments/assets/8c5f1c8a-62fb-4436-aefb-cf15fed8d255" /> 

