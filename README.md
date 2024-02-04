# Fedimint

Eine Anleitung für Installation und Nutzung einer Fedimint, in deutsch.

Diskussionen hierzu in der Telegram-Gruppe <https://t.me/fedimintgerman>.

Erstellt von [Telegram: Contact @mikeninehundred](https://t.me/mikeninehundred)

## Inhalt

* [Voraussetzungen für die Installation](#voraussetzungen-für-die-installation)

* [Docker-Images](#docker-images)

* [Eigene Domain in Apache Webserver](#eigene-domain-in-apache-webserver)

* [TLS-Zertifikate](#tls-zertifikate)

* [Härtung](#härtung)

## Voraussetzungen für die Installation

Hier wird die Installation eines Fedimint-Guardians auf einem Debian-Rechner beschrieben, der direkt im Internet hängt. Falls ein Rechner verwendet wird, der hinter einem Internet-Router läuft, müssen noch zusätzliche Dinge (Port-Weiterleidung, DynDNS) beachtet werden, die hier nicht beschrieben werden.

Es werden auf dem Rechner folgende Anwendungen installiert:

- docker und docker-compose (für Installation, Konfiguration und Start der wichtigsten Programme)
- fedimintd (der eigentliche Fedimint-Guardian-Prozess)
- guardian-ui (die Web-Bedienoberfläche für den Guardian-Prozess)
- bitcoind (ein pruned BitcoinCore für den Guardian)
- Apache (ein Reverse-Proxy-Server zur Verteilung der Webaufrufe und die Nutzung der TLS-Zertifikate)
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

## Eigene Domain in Apache Webserver

### Domain-Beschaffung

Für die Verwendung von TLS-Zertifikaten braucht es erst einen eigenen Domainnamen für den Server. Der kann bei einem beliebigen Registrar gekauft werden. Nach dem Kauf müssen DNS-Records angepasst oder angelegt werden. Prinzipiell werden die Records am Ende so aussehen:

```dns-zone-file
@ IN A x.x.x.x
www IN CNAME beispiel.de
fmd IN CNAME beispiel.de
fmdui IN CNAME beispiel.de
```

Dabei ist `x.x.x.x` die IP-Adresse des Servers, `beispiel.de` der gekaufte Domainname, `fmd` die Subdomain der Guardian-API und `fmdui` die Subdomain der Web-UI des Guardians. Die Subdomains können natürlich auch an eigene Ideen angepasst werden.

### Apache-Konfiguration

Die TLS-Zertifikate werden später in einem Apache-Webserver installiert, der als Reverse-Proxy konfiguriert wird. Damit wird die TLS-Installation einfach und der Apache fungiert auch noch als Schutz und Puffer vor den beiden Fedimint-Komponenten. Die sind damit also nicht mehr direkt mit dem Internet verbunden.

Die Installation des Apache ist einfach

```bash
sudo apt install apache2
sudo a2enmod proxy proxy_http proxy_wstunnel
```

Dazu müssen die passenden Konfigurationsdatei des Apache in `/etc/apache2/sites-available` angelegt werden. `beispiel.de.conf` :

```apacheconf
<VirtualHost *:80>
	ServerName beispiel.de:80

	ServerAdmin webmaster@localhost
	DocumentRoot /var/www/html

	ErrorLog ${APACHE_LOG_DIR}/error.log
	CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>

```

und `fedimint.beispiel.de.conf`:

```apacheconf
<VirtualHost *:80>
    ServerName fmdui.beispiel.de
    ProxyPass / http://x.x.x.x:3000/ nocanon
    ProxyPassReverse / http://x.x.x.x/
    ProxyPreserveHost On

    ErrorLog ${APACHE_LOG_DIR}/fmdui-error.log
    CustomLog ${APACHE_LOG_DIR}/fmdui-access.log combined
</VirtualHost>

<VirtualHost *:80>
    ServerName fmd.beispiel.de
    ProxyPass / ws://x.x.x.x:8174/ nocanon
    ProxyPassReverse / ws://x.x.x.x/
    ProxyWebsocketFallbackToProxyHttp On

    ErrorLog ${APACHE_LOG_DIR}/fmd-error.log
    CustomLog ${APACHE_LOG_DIR}/fmd-access.log combined
</VirtualHost>
```

Diese Datei muss in `/etc/apache2/sites-available` angelegt werden (z.B. als beispiel.de), und dann aktiviert mit:

```bash
sudo a2ensites beispiel.de fedimint.beispiel.de
```

So übernimmt Apache die Änderungen:

```bash
sudo systemctl restart apache2
```

Jetzt kann die Guardian-WebUI mit http://fmdui.beispiel.de aufgerufene werden. Für die Verbindung zu anderen Guardians gilt jetzt http://fmd.beispiel.de und mit http://beispiel.de wird die Apache-Standardseite angezeigt.

Damit wissen wir, dass der Zugriff über Apache funktioniert, aber noch ohne TLS-Zertifikate.

## TLS-Zertifikate

Die TLS-Zertifikate sollen mit `certbot` installiert werden. `certbot` aus dem Debian-Repository ist veraltet. Es muss `certbot` aus dem snap-Repository verwendet werden. Dazu muss zunächst `snap` installiert werden: (auf Ubuntu ist `snap` möglicherweise schon drauf)

```bash
sudo apt install snap
sudo snap install core
```

So wird `snap` installiert, und noch das nötige core-Modul geladen. Damit ist `snap` bereit.

Jetzt kann der aktuelle `certbot` installiert werden:

```bash
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot
```

Jetzt ist `certbot` installiert und kann aufgerufen werden.

Da wir oben bereits den `Apache` für die drei Domains vorbereitet haben, ist der Aufruf von `cerbot` jetzt ganz einfach:

```bash
sudo certbot --apache2
```

`certbot` findet alle drei Domain-Name. Die Frage zu welchen er ein Zertifikat installieren soll muss nur mit `ENTER` bestätigt werden, dann werden Zertifikate für alle drei Domain-Namen erstellt und installiert. Auch der `Apache` wird gleich neu gestartet. Damit ist das Port 443 für alle drei Namen freigeschaltet. Das Port 80 wird umdefiniert, damit Aufrufe dorthin auf Port 443 umgeleitet werden. Und `certbot` wird in `systemctl` so eingerichtet, dass es die Zertifikate selbständig vor Ablauf aktualisiert. Die Zertifikate sind jetzt fertig installiert.

## Härtung

### Schritt 1 ssh-key einrichten

Annahme: Der Zugriff auf den Linux-Rechner erfolgt per PuTTY von einem Windows-PC aus. Dann muss mit PuTTYgen ein Keypair erstellt werden und Public- und Private-Key jeweils in Dateien sicher gespeichert werden. (!!! Besonders die Private-Key-Datei dürft ihr nicht verlieren, sie wird am Ende die einzige Möglichkeit sein an den Rechner ran zu kommen).

Der gesamte Textblock in PuTTYgen im oberen Fenster muss auf dem Debian-Server in die Datei ~/.ssh/authorized_keys als eine Zeile eingetragen werden. Unbedingt prüfen, ob diese Datei dem User gehört und Datei sowie Verzeicnis nur vom User gelesen und geändert werden können. Die Rechte sind wichtig, sonst funktioniert der sshkey nicht.

Danach muss Pageant gestartet werden und der eben erstellte Private-Key in ihn geladen werden. Dann können PuTTY und auch SuperPuTTY ohne Kennwortabfrage auf die Linux-Kommandozeile kommen.

Wenn das klappt, kann die Kennwortabfrage abgeschaltet werden.

Dazu in `/etc/ssh/sshd_config` den Wert `PasswordAuthentication no` setzen.

Falls nicht schon geschehen, kann das root-login auch deaktiviert werden. 

Dazu in `/etc/ssh/sshd_config` den Wert `PermitRootLogin no` setzen, dann

```bash
sudo systemctl restart ssh
```

### Schritt 2 ufw Firewall einrichten

So wird die Firewall `ufw` aktiviert, wenn sie nicht bereits aktiv ist. Die Grundeinstellung ist, dass alle Port geschlossen sind und die Ports, die benötigt werden per Eingabe geöffnet werden müssen. Vorsicht: das Port 22 muss auf jeden fall geöffnet werden, bevor die Firewall aktiviert wird, ansonsten habt ihr euch ausgeschlossen. Der Remote-Zugriff auf die Konsole findet immer per SSH auf Port 22 statt. 

```bash
sudo ufw allow 22/tcp
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw allow 39388/tcp
sudo ufw enable
```
