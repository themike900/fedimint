# fedimint
Eine Anleitung für Installation und Nutzung einer Fedimint, in deutsch.
# Voraussetzungen für die Installation
- Die Docker-Images werden derzeit nur für Intel-Architektur erstellt. Es wird deshalb ein Rechner mit Intel/AMD-Prozessor benötigt. 
- Wenn der Rechner nur für die Fedimint-Prozess eingesetzt wird, reicht scheinbat vorerst eine HD/SSD von 100 GB dicke aus. Das muss aber in der Praxis noch beobachtet werden. Soll der Bitcoin-Knoten und vielleicht auch ein LND-Knoten mit auf den Rechner, muss die HD/SSD entsprechend viel größer sein.
- Betriebssystem muss ein gängiges Linux sein z.B. Ubuntu. Es sollte keine grafische Oberfläche laufen, nur Kommandozeilen-Server-Mode.

Im Ubuntu muss docker und docker-compose installiert werden mit `apt install docker docker-compose`
# Docker-Images
Für einen reinen Guardian müssen die docker images fedimnitd und guardian-ui installiert wer
Docker images werden installiert mit dem Kommando `docker pull <image>`

| docker image | Verwendung |
|--|--|
| fedimint/fedimintd:v0.2.1 | der eigentliche Guardian-Prozess |
| fedimintui/guardian-ui:0.2.1 | die Weboberfläche für den Guardian|
| fedimint/gatewayd:v0.2.1 |  der Gateway-Prozess |
| fedimintui/gateway-ui:0.2.1 | die Weboberfläche für das Gateway |






<!--stackedit_data:
eyJoaXN0b3J5IjpbMTM5MjQ3Nzc2MSwtNDQ4Mzk3MjYsMTg1ND
QxNzg4NF19
-->