# **Systemvorraussetzungen**

Sollte zwischen dem Server-Setup und der Installation Zeit vergangen sein, sollte apt nochmal ein Update bekommen:

```sh
sudo apt update
```

**Open Java Developer Kit:**

```sh
sudo apt install default-jdk
```

**Maven**

```sh
sudo apt install maven
```

**Curl**

```sh
sudo apt install curl
```

**git**

```sh
sudo apt install git
```

**Apache Tomcat**

Tomcat bekommt hier keine eigene Benutzergruppe, da VIVO im Tomcat-Verzeichnet lebt und arbeitet.

Der Tomcat Download kann prinzipiell überall hin erfolgen, es ist aber angeraten ihn in /tmp auszuführen.

```sh
cd /tmp
curl -O http://www-eu.apache.org/dist/tomcat/tomcat-9/v9.0.11/bin/apache-tomcat-9.0.11.tar.gz
```

Die Tomcat Installation erfolgt ins opt/tomcat-Verzeichnis:

```sh
sudo mkdir /opt/tomcat
sudo tar xzvf apache-tomcat-9*tar.gz -C /opt/tomcat --strip-components=1
```

Erstellen eines systemd service files für Tomcat:

```sh
sudo nano /etc/systemd/system/tomcat.service
```

mit Folgendem Inhalt:

```text
[Unit]
Description=Apache Tomcat Web Application Container
After=network.target

[Service]
Type=forking

Environment=JAVA_HOME=/usr/lib/jvm/java-1.11.0-openjdk-amd64
Environment=CATALINA_PID=/opt/tomcat/temp/tomcat.pid
Environment=CATALINA_HOME=/opt/tomcat
Environment=CATALINA_BASE=/opt/tomcat
Environment='CATALINA_OPTS=-Xms512M -Xmx1024M -server -XX:+UseParallelGC'
Environment='JAVA_OPTS=-Djava.awt.headless=true -Djava.security.egd=file:/dev/./urandom'

ExecStart=/opt/tomcat/bin/startup.sh
ExecStop=/opt/tomcat/bin/shutdown.sh

UMask=0007
RestartSec=10
Restart=always

[Install]
WantedBy=multi-user.target
```

Vivo empfiehlt `**-XX:MaxPermSize=128m**` in die CATALINA\_OPTS aufzunehmen. Ebenfalls wird von VIVO ein max heap von 512m empfohlen, wir arbeiten aktuell mit 1024M. Bisher gab es keine Performanzeinbußen.

Um UTF-8 Kompatibilität in Tomcat zu aktivieren, die setup.xml öffnen:

```sh
nano /opt/tocat/conf/server.xml
```

und URI Encoding in den Connector-Elementen aktivieren:

```xml
 <Server ...>

  <Service ...>

    <Connector ... URIEncoding="UTF-8"/>

      ...

    </Connector>

  </Service>

</Server> 
```

Tomcat starten / stoppen

```sh
sudo systemctl start tomcat
sudo systemctl stop tomcat
```

# **VIVO 12**

**Download**

Der VIVO Download kann in einen Ordner im /tmp oder in das user-Verzeichnis erfolgen. Auf dem Testserver ist es im User-Verzeichnis für einfache Erreichbarkeit, auf dem Live-Server wird es ins /tmp geladen.

```sh
git clone https://github.com/vivo-project/Vitro.git Vitro -b rel-1.12.2-maint
git clone https://github.com/vivo-project/VIVO.git VIVO -b rel-1.12.2-maint
git clone https://github.com/vivo-project/Vitro-languages.git Vitro-languages -b rel-1.12.2-maint 
git clone https://github.com/vivo-project/VIVO-languages.git VIVO-languages -b rel-1.12.2-maint 
```

Die Vivo Installationsdatein werden nach der Installation nicht mehr gebraucht und können gelöscht werden. Auf dem Testserver liegen die Dateien im home-Verzeichnis für den Fall einer Neuinstallation.

**Installation**

```sh
mvn install -s example-settings.xml
```

Die Installation erstellt 2 Ordner:

*   vivo-home: /usr/local/vivo -&gt; enthält die TripleStores, SPARQL-Queries für den Seitenaufbau und Java-Code,...
*   webapps: /opt/tomcat/webapps/vivo -&gt; enthält die Templates, Bilder, Texte, Frontend-Informationen, ...

Beide Ordner sind in GitLab gespiegelt:

Der Installationsordner im usr-Verzeichnis: https://scm.cms.hu-berlin.de/kotschfl/vivo\_webapps

Der Webapps-Ordner im tomcat-Verzeichnis: [https://scm.cms.hu-berlin.de/kotschfl/vivo\_bua](https://scm.cms.hu-berlin.de/kotschfl/vivo_bua)

(Benennung in den Links ist flasch! - die hier angegebene Zuordnung stimmt)

**Nach der Installation - Webapps**

Das Frontend kann nach der Installation direkt aus dem Gitlab gezogen werden:

1.  den gesamten Inhalt von /opt/tomcat/webapps/vivo löschen

```sh
sudo rm -r /opt/tomcat/webapps/vivo/*
```

2. git im webapps/vivo-Ordner initialisieren

```sh
git init
git commit -m "init" 
git stash
```

3. das Repository vom Gitlab clonen

```sh
git clone https://scm.cms.hu-berlin.de/kotschfl/vivo_bua main
```

Ein git pull sollte theoretisch das selbe Ergebnis erzielen, hat aber bei bisherigen Versuchen heads in einige config-Dateien geschrieben. Die git-clone Methode ist sauber und sichert das Anzeigen des exakten Frontends.

Ein Neustart des Tomcat-Servers ist prinzipiell nicht nötig, wird aber empfohlen.

Um die Frontend-Updates im Browser anzuzeigen, muss der Browser-Cache gelöscht werden.

**Nach der Installation - vivo-home**

Vor der ersten Verwendung Vivo-home muss die runtime.properties angepasst werden. Das kann entweder händisch geschehen oder die properties-Datei kann durch die Datei im vivo-home-Gitlab ersetzt werden.

Für eine manuelle Bearbeitung sind folgende Änderungen notwendig:

```text
Vitro.defaultNamespace = http://vivo.berlin-university-alliance.de/individual/

rootUser.emailAddress = xxx@yyy.zzz

vitro.local.solr.url = http://192.168.10.20:8983/solr/vivocore
```

Die rootUser.emailAddress ist nur für den ersten Login relevant und kann abweichen.
Die IP in der SolR URL muss angepasst werden. 
