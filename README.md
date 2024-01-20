# fedimint
Eine Anleitung für Installation und Nutzung einer Fedimint, in deutsch.
# Voraussetzungen für die Installation
- Die Docker-Images werden derzeit nur für Intel-Architektur erstellt. Es wird deshalb ein Rechner mit Intel/AMD-Prozessor benötigt. 
- Wenn der Rechner nur für die Fedimint-Prozess eingesetzt wird, reicht scheinbat vorerst eine Festpaltte/SSD von 100 GB dicke aus. Das muss aber in der Praxis noch beobachtet werden. Soll der Bitco
- Betriebssystem muss ein gängiges Linux sein z.B. Ubuntu. 
# Docker-Images
Docker images werden installiert mit dem Kommando `docker pull <image>`

| docker image | Verwendung |
|--|--|
| fedimint/fedimintd:v0.2.1 | der eigentliche Guardian-Prozess |
| fedimintui/guardian-ui:0.2.1 | die Weboberfläche für den Guardian|
| fedimint/gatewayd:v0.2.1 |  der Gateway-Prozess |
| fedimintui/gateway-ui:0.2.1 | die Weboberfläche für das Gateway |






<!--stackedit_data:
eyJoaXN0b3J5IjpbMTc5OTE4NzAxNiwxODU0NDE3ODg0XX0=
-->