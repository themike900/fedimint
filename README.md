# Fedimint

Eine Anleitung für Installation und Nutzung einer Fedimint, in deutsch.

# Voraussetzungen für die Installation

Hier wird die Installation eines Fedimint-Guardians auf eine Debian-Rechner beschrieben, der direkt im Internet hängt. Falls ein Rechner verwendet wird, der hinter einem Internet-Router läuft, müssen noch zusätzliche Dinge (Port-Weiterleidung, DynDNS) beachtet werden, die hier nicht beschrieben werden.

Es werden auf dem Rechner folgende Anwendungen installiert:

- docker und docker-compose (für Installation, Konfiguration und Start der wichtigsten Programme)

- fedimintd (der eigentliche Fedimint-Guardian-Prozess)

- guardian-ui (die Web-Bedienoberfläche für den Guardian-Prozess)

- bitcoind (ein pruned BitcoinCore für den Guardian)

- nginx (eine Reverse-Proxy-Server zur Verteilung der Webaufrufe und die Nutzung der TLS-Zertifikate)

- certbot (damit werden die TLS-Zertifikate erstellt und verwaltet)

- ufw (die Firewall zum Schutz aller nicht benötigten Ports)

Wenn der Rechner nur für die Fedimint-Prozess eingesetzt wird, reicht scheinbar vorerst eine HD/SSD von 100 GB dicke aus. Das muss aber in der Praxis noch beobachtet werden. Soll der Bitcoin-Knoten und vielleicht auch ein LND-Knoten mit auf den Rechner, muss die HD/SSD wahrscheinlich viel größer sein.

Betriebssystem muss ein gängiges Linux sein z.B. Ubuntu. Es sollte keine grafische Oberfläche laufen, nur Kommandozeilen-Server-Mode.

Im Ubuntu muss docker und docker-compose installiert werden. 

Wird der Rechner für eine Mainnet-Installation direkt ins Internet gehängt, braucht es zusätzliche Voraussetzungen. Ein revese-proxy mit TLS-Zertifikaten (`caddy` oder `nginx`) und eine Firewall (`ufw`) sollten unbedingt installiert werden, damit der Rechner geschützt ist und die Kommunikation sicher ist. Wenn `nginx` der reverse-proxy wird, ist `certbot` ein gute Wahl für die Beschaffung und Aktualisierung der TLS-Zertifikate.

Für  die folgenden Absätze wird vorausgesetzt, dass der Ausführende mit einem Nutzer  in einer Linux-Konsole angemeldet ist der nicht der root-User ist, und aber sudo-Rechte hat.

# Docker-Images

Zunächst muss docker installiert werden

```bash
sudo apt install docker docker-compose
```

Die Docker-Images werden von den Devs derzeit nur für Intel-Architektur bereitgestellt. Es wird deshalb ein Rechner mit Intel/AMD-Prozessor benötigt.

Für einen reinen Guardian müssen die docker images `fedimnitd` und `guardian-ui` und `bitcond` installiert werden. 

So werden die Images aus dem Internet geholt:

```bash
sudo docker pull fedimint/fedimintd:v0.2.1
sudo docker pull fedimintui/guardian-ui:0.2.1
sudo docker pull btcpayserver/bitcoin:26.0
```

Die spannende Stelle ist jetzt die docker images als docker container zu starten. Dabei müssen eine ganze Reihe von richtig gesetzten Parametern übergeben werden. Das geht elegant mit dem Kommando `docker-compose`.
`docker-compose` ruft immer eine Datei `docker-compose.yaml` auf, in der alle für den Start der Containers nötige Parameter stehen.

Hier sind die Starts der Container in zwei Datei aufgeteilt, eine docker-compose.yaml für den Start des Bitcoin-Knotens und eine docker-compos.yaml für den Start der beiden Fedimint-Container.

Zuerst muss bitcoind gestartet werden. Die Konfiguration sieht einen pruned-Bitcoin-Knoten vor bei dem die Blockchain auf die jüngsten 40GB beschränkt ist. Trotzdem muss sich dieser Konten einmal die ganze Blockchain holen und verifizieren. Das dauert im günstigsten Fall ca. 12 Stunden. Erst wenn bitcoind synchron ist, kann der Fedimint-Guardian erfolgreich gestartet werden.

```bash
cd ~
mkdir bitcoind
cd bitcoind
wget https://raw.githubusercontent.com/themike900/fedimint/main/mainnet-no-tls/docker-compose.yaml
docker-compose up -d
```

Hier im Ordner `docker-compose-guardian` liegt die `docker-compose.yaml`, mit der ein Guardian im Bitcoin-Mainnet gestartet wird. Dieses Datei muss vorher jeweils an die lokalen Bedingungen angepasst werden . Ist bitcoind synchron geht es so weiter:

```bash
cd ~
mkdir fedimitd
cd fedimitd
wget https://raw.githubusercontent.com/themike900/fedimint/main/mainnet-no-tls/docker-compose.yaml
docker-compose up -d
```

Wenn alles geklappt hat, dann läuft jetzt der Guardian, aber noch ohne TLS. Das kann mit dem Aufruf der Guardian-UI geprüft werden. Die sollte jetzt unter http://x.x.x.x:3000 erreichbar sein (x.x.x.x für die IP.Adresse des Servers.)

# Schritt für Schritt (mainnet)

Für einen Debian Server sind für die Mutinynet-Testumgebung folgende Schritte auszuführen:

```bash
sudo docker pull fedimint/fedimintd:v0.2.1
sudo docker pull fedimintui/guardian-ui:0.2.1
cd ~
mkdir fedimint
cd ~/fedimint
wget https://github.com/themike900/fedimint/raw/main/docker-compose-mutinynet/docker-compose.yaml
```

# Eigene Domain für TLS

Für die Verwendung von TLS-Zertifikaten braucht es erst einen eigenen Domainnamen für den Server. Der kann bei einem beliebigen Registrar gekauft werden. Nach dem Kauf müssen DNS-Records angepasst oder angelegt werden. Prizipell wird es am Ende so aussehen:

```dns-zone-file
@ IN A x.x.x.x
www IN CNAME beispiel.de
fmd IN CNAME beispiel.de
fmdui IN CNAME beispiel.de
```

Dabei ist x.x.x.x die IP-Adresse des Servers, beispiel.de der gekaufte Domainname, fmd die Subdomain der Guardian-API und fmdui die Subdomain der Web-UI des Guardians.

Die dazu passende Konfigurationsdatei des nginx ist

```nginx

```
