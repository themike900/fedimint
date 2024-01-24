# Fedimint
Eine Anleitung für Installation und Nutzung einer Fedimint, in deutsch.
# Voraussetzungen für die Installation
- Die Docker-Images werden von den Devs derzeit nur für Intel-Architektur bereitgestellt. Es wird deshalb ein Rechner mit Intel/AMD-Prozessor benötigt. 
- Wenn der Rechner nur für die Fedimint-Prozess eingesetzt wird, reicht scheinbar vorerst eine HD/SSD von 100 GB dicke aus. Das muss aber in der Praxis noch beobachtet werden. Soll der Bitcoin-Knoten und vielleicht auch ein LND-Knoten mit auf den Rechner, muss die HD/SSD wahrscheinlich viel größer sein.
- Betriebssystem muss ein gängiges Linux sein z.B. Ubuntu. Es sollte keine grafische Oberfläche laufen, nur Kommandozeilen-Server-Mode.
- Im Ubuntu muss docker und docker-compose installiert werden. 
Wird der Rechner für eine Mainnet-Installation direkt ins Internet gehängt, braucht es zusätzliche Voraussetzungen. Ein revese-proxy mit TLS-Zertifikaten (`caddy` oder `nginx`) und eine Firewall (`ufw`) sollten unbedingt installiert werden, damit der Rechner geschützt ist und die Kommunikation sicher ist. Wenn `nginx` der reverse-proxy wird, ist `certbot` ein gute Wahl für die Beschaffung und Aktualisierung der TLS-Zertifikate.
# Docker-Images
- Für einen reinen Guardian müssen die docker images `fedimnitd` und `guardian-ui` installiert werden. Es es muss die API-Schnittstelle eines BitcoinCore im lokalen Netzt oder auf dem gleichen Rechner erreichbar sein.
- Ein Gateway braucht die docker images `gatewayd` und `gateway-ui` und muss eine Lightning-Node im lokalen Netz oder auf dem gleichen Rechner erreichen können. 

Docker images werden installiert mit dem Kommando `docker pull <image>`

| docker image | Verwendung |
|--|--|
| fedimint/fedimintd:v0.2.1 | der eigentliche Guardian-Prozess |
| fedimintui/guardian-ui:0.2.1 | die Weboberfläche für den Guardian|
| fedimint/gatewayd:v0.2.1 |  der Gateway-Prozess |
| fedimintui/gateway-ui:0.2.1 | die Weboberfläche für das Gateway |

Die spannende Stelle ist jetzt die docker images als docker container zu starten. Dabei müssen eine ganze Reihe von richtig gesetzten Parametern übergeben werden. Das geht elegant mit dem Kommando `docker-compose`.
`docker-compose` ruft eine Datei `docker-compose.yaml` auf, in der alle für den Start der Containers nötige Parameter stehen.

Hier im Ordner `docker-compose-mutinynet` liegt die `docker-compose.yaml` mit der 4 Guradians mit 4 Guardian-UIs gestartet werden, die zum Testen nicht mit dem Bitcoin-Mainnet verbunden sind, sondern mit dem Mutinynet, dass die Fedimint-Entwickler auch verwenden. Das hat auch den Vorteil, dass kein eigener Bitcoin-Core installiert werden muss.

Hier im Ordner `docker-compose-guardian` wird die `docker-compose.yaml` liegen, mit der ein einziger Guardian im Bitcoin-Mainnet gestartet wird. Dieses Datei muss vorher jeweils an die lokalen Bedingungen angepasst werden (wie wird die BitcoinCore API erreicht). Die Datei ist noch nicht erstellt.

# Schritt für Schritt
## Mutiny-Testnetz
Für einen Ubuntu Server sind für die Mutinynet-Testumgebung folgende Schritte auszuführen:

    sudo apt install docker docker-compose
    sudo docker pull fedimint/fedimintd:v0.2.1
    sudo docker pull fedimintui/guardian-ui:0.2.1 
    cd ~
    mkdir fedimint
    cd fedimint
    wget https://github.com/themike900/fedimint/raw/main/docker-compose-mutinynet/docker-compose.yaml
    sudo docker-compse up -d







<!--stackedit_data:
eyJoaXN0b3J5IjpbLTIwNTY5ODUwMTEsNjI0MTM0NTk0LDc1Nj
I1NzE0NiwyMzE2MTE2NzksLTExNDA1OTk3NzAsLTQ0ODM5NzI2
LDE4NTQ0MTc4ODRdfQ==
-->
