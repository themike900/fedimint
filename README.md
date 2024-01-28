# Fedimint

Eine Anleitung für Installation und Nutzung einer Fedimint, in deutsch.

Diskussionen hierzu in der Telegram-Gruppe <https://t.me/fedimintgerman>.

Inhalt

* [Voraussetzungen für die Installation](#voraussetzungen)

* [Docker-Images](#docker)

* [Docker-Images](#docker-images)

# Voraussetzungen für die Installation <a name="voraussetzungen"></a>

Hier wird die Installation eines Fedimint-Guardians auf einem Debian-Rechner beschrieben, der direkt im Internet hängt. Falls ein Rechner verwendet wird, der hinter einem Internet-Router läuft, müssen noch zusätzliche Dinge (Port-Weiterleidung, DynDNS) beachtet werden, die hier nicht beschrieben werden.

Es werden auf dem Rechner folgende Anwendungen installiert:

- docker und docker-compose (für Installation, Konfiguration und Start der wichtigsten Programme)
- fedimintd (der eigentliche Fedimint-Guardian-Prozess)
- guardian-ui (die Web-Bedienoberfläche für den Guardian-Prozess)
- bitcoind (ein pruned BitcoinCore für den Guardian)
- nginx (ein Reverse-Proxy-Server zur Verteilung der Webaufrufe und die Nutzung der TLS-Zertifikate)
- certbot (damit werden die TLS-Zertifikate erstellt und verwaltet)
- ufw (die Firewall zum Schutz aller nicht benötigten Ports)

Wenn der Rechner nur für die Fedimint-Prozess eingesetzt wird, reicht scheinbar vorerst eine HD/SSD von 150 GB dicke aus. Das muss aber in der Praxis noch beobachtet werden. Betriebssystem muss ein gängiges Linux sein z.B. Debian oder Ubuntu. Es sollte keine grafische Oberfläche laufen, nur Kommandozeilen-Server-Mode.

Die Docker-Images werden von den Devs derzeit nur für Intel-Architektur bereitgestellt. Es wird deshalb ein Rechner mit Intel/AMD-Prozessor benötigt, kein RaspberryPi.

Für  die folgenden Absätze wird vorausgesetzt, dass der Ausführende mit einem Nutzer  in einer Linux-Konsole angemeldet ist der nicht der root-User ist, und aber sudo-Rechte hat.

## Docker-Images

Zunächst muss docker installiert werden

```bash
sudo apt install docker docker-compose
```

Für einen reinen Guardian müssen die docker images `fedimnitd` und `guardian-ui` und `bitcond` installiert werden. 

So werden die Images aus dem Internet geholt:

```bash
sudo docker pull fedimint/fedimintd:v0.2.1
sudo docker pull fedimintui/guardian-ui:0.2.1
sudo docker pull btcpayserver/bitcoin:26.0
```

Die spannende Stelle ist jetzt die docker images als docker container zu starten. Dabei müssen eine ganze Reihe von richtig gesetzten Parametern übergeben werden. Das geht elegant mit dem Kommando `docker-compose`.
`docker-compose` ruft immer eine Datei `docker-compose.yaml` auf, in der alle für den Start der Containers nötige Parameter stehen. Vor jedem ersten Aufruf eine docker-compose.yaml mit docker-compose sollte nochmal genau geprüft werden, ob nicht vielleicht einzelne Parameter an IP-Adresse und Domain-Name angepasst werden müssen.

Hier sind die Starts der Container in zwei Datei aufgeteilt, eine docker-compose.yaml für den Start des Bitcoin-Knotens und eine docker-compos.yaml für den Start der beiden Fedimint-Container.

Zuerst muss bitcoind gestartet werden. Die Konfiguration sieht einen pruned-Bitcoin-Knoten vor bei dem die Blockchain auf die jüngsten 40GB beschränkt ist. Trotzdem muss sich dieser Konten einmal alle Blöcke der Blockchain holen und verifizieren. Das dauert im günstigsten Fall ca. 12 Stunden. Erst wenn bitcoind synchron ist, kann der Fedimint-Guardian erfolgreich gestartet werden.

```bash
cd ~
mkdir bitcoind
cd bitcoind
wget https://raw.githubusercontent.com/themike900/fedimint/main/bitcoind-mainnet/docker-compose.yaml
sudo docker-compose up -d
```

Hier im Ordner `docker-compose-guardian` liegt die `docker-compose.yaml`, mit der ein Guardian im Bitcoin-Mainnet gestartet wird. Dieses Datei muss vorher jeweils an die lokalen Bedingungen angepasst werden . Ist bitcoind synchron geht es so weiter:

```bash
cd ~
mkdir fedimitd
cd fedimitd
wget https://raw.githubusercontent.com/themike900/fedimint/main/fedimint-mainnet/docker-compose.yaml
sudo docker-compose up -d
```

Wenn alles geklappt hat, dann läuft jetzt der Guardian, aber noch ohne TLS. Das kann mit dem Aufruf der Guardian-UI geprüft werden. Die sollte jetzt unter http://x.x.x.x:3000 erreichbar sein (x.x.x.x für die IP-Adresse des Servers.)

# Eigene Domain in nginx

Für die Verwendung von TLS-Zertifikaten braucht es erst einen eigenen Domainnamen für den Server. Der kann bei einem beliebigen Registrar gekauft werden. Nach dem Kauf müssen DNS-Records angepasst oder angelegt werden. Prinzipiell werden die Records am Ende so aussehen:

```dns-zone-file
@ IN A x.x.x.x
www IN CNAME beispiel.de
fmd IN CNAME beispiel.de
fmdui IN CNAME beispiel.de
```

Dabei ist `x.x.x.x` die IP-Adresse des Servers, `beispiel.de` der gekaufte Domainname, `fmd` die Subdomain der Guardian-API und `fmdui` die Subdomain der Web-UI des Guardians. Die Subdomains können natürlich auch an eigene Ideen angepasst werden.

Die dazu passende Konfigurationsdatei des nginx ist:

```nginx
server {
    listen 80;
    server_name beispiel.de;
    location / {
        root /var/www/html;
    }
}
server {
    listen 80;
    server_name fmdui.beispiel.de;
    location / {
        proxy_pass http://127.0.0.1:3000;
        include proxy_params;
    }
}
server {
    listen 80;
    server_name fmd.beispiel.de;
    location / {
        proxy_pass http://127.0.0.1:8173;
        include proxy_params;
    }
}
```

Diese Datei muss in `/etc/nginx/sites-available` angelegt werden (z.B. als beispiel.de), und dann darauf ein symbolic link in `/etc/nginx/sites-enabled` angelegt werden, mit 

```bash
ln -s /ect/nginx/sites-availabe/beispiel.de /etc/nginx/sites-enabled/beispiel.de
```

So übernimmt nginx die Änderungen:

```bash
sudo systemctl restart nginx
```

Jetzt kann die Guardian-WebUI mit http://fmdui.beispiel.de aufgerufene werden. Für die Verbindung zu anderen Guardians gilt jetzt http://fmd.beispiel.de und mit http://beispiel.de wird die nginx-Standardseite angezeigt.

Damit wissen wir, dass der Zugriff über nginx funktioniert.

# TLS-Zertifikate

Die TLS-Zertifikate sollen mit certbot installiert werden. certbot aus dem Debian-Repository ist veraltet. Es muss certbot aus dem snap-Repository verwendet werden. Dazu muss zunächst snap installiert werden:

```bash
sudo apt install snap
sudo snap install core
```

Dabei wird snap installiert, dann noch das nötige core-Modul geladen. Damit ist snap bereit.

Jetzt kann der aktuelle certbot installiert werden:

```bash
snap install --classic certbot
ln -s /snap/bin/certbot /usr/bin/certbot
```

Jetzt ist certbot installiert und kann aufgerufen werden.

Da wir oben bereits den nginx für die drei Domains vorbereitet haben, ist der Aufruf von cerbot jetzt ganz einfach:

```bash
certbot --nginx
```

certbot findet alle drei Domain-Name. Die Frage zu welchen er ein Zertifikat installieren soll muss nur mit ENTER bestätigt werden, dann werden Zertifikate für alle drei Domain-Namen erstellt und installiert. Auch nginx wird gleich neu gestartet. Damit ist das Port 443 für alle drei Namen freigeschaltet. Das Port 80 wird umdefiniert, damit Aufrufe dorthin auf Port 443 umgeleitet werden. Und certbot wird in systemctl so eingerichtet, dass es die Zertifikate selbständig von Ablauf aktualisiert. Die Zertifikate sind jetzt fertig installiert.

!!! Noch passt die Konfiguration nicht ganz. Ohne TLS über Port 80 haben die Zugriffe funktioniert, mit TLS über Port 443 derzeit noch nicht. Fehlersuche ist angesagt.

# Härtung

Jetzt können alle Ports gesperrt werden, die nicht mehr gebraucht werden. Es werden nur noch gebraucht:

- 22 für SSH
- 80 für certbot
- 443 für Fedimint-Guardian-Verbindung
- 443 für Web-UI
- 443 für certbot
- 39388 für bitcoind

Fortsetzung folgt :)
