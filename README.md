<center>
 <img src="ERPNext-servidor-cloud.jpg">
</center>

# So installieren Sie ERPNext in Ubuntu 20.04
### A Step by Step Guide
Die Installation von ERPNext kann nervig sein, besonders wenn Sie gerade erst anfangen. In diesem Artikel werde ich Schritt für Schritt vorgehen, um unser neu installiertes Betriebssystem Ubuntu 20.04 zu konfigurieren, um eine Umgebung einzurichten und ERPNext zu installieren. 
## Voraussetzungen
### Software Anforderungen
* Updated Ubuntu 20.04
* A user with sudo privileges
* Python 3.6+
* Node.js 12
* Redis 5
* MariaDB 10.3.x / Postgres 9.5.x
* yarn 1.12+
* pip 20+
* wkhtmltopdf (version 0.12.5 with patched qt)
* cron
* NGINX
### Hardware Anforderungen
* 4GB RAM
* 40GB Hard Disk
 
In unseren ersten Schritten stellen wir sicher, dass unser System auf dem neuesten Stand ist, indem wir die folgenden Befehle ausführen:
```
sudo apt update
sudo apt -y upgrade
```

Es wird empfohlen, Ihr System bei jedem Upgrade neu zu starten:
```
sudo reboot
```
#### Schritt 1: Installieren Sie Python Tools & wkhtmltopdf
```
sudo apt -y install vim libffi-dev python3-pip python3-dev python3-testresources libssl-dev wkhtmltopdf
```
#### Schritt 2: Installieren Sie Curl, Redis und Node.js
```
sudo apt install curl

sudo curl --silent --location https://deb.nodesource.com/setup_14.x | sudo bash -

curl -sL https://dl.yarnpkg.com/debian/pubkey.gpg | gpg --dearmor | sudo tee /usr/share/keyrings/yarnkey.gpg >/dev/null

echo "deb [signed-by=/usr/share/keyrings/yarnkey.gpg] https://dl.yarnpkg.com/debian stable main" | sudo tee /etc/apt/sources.list.d/yarn.list

sudo apt-get update && sudo apt-get install yarn

sudo apt -y install gcc g++ make nodejs redis-server
```
#### Schritt 3: Installieren Sie den Nginx-Webserver und den MariaDB-Datenbankserver
```
sudo apt -y install nginx
sudo apt install mariadb-server
```
Authentifizierungs-Plugin ändern.
```
sudo mysql -u root
USE mysql;
UPDATE user SET plugin='mysql_native_password' WHERE User='root';
UPDATE user SET authentication_string=password('your_password') WHERE user='root';
FLUSH PRIVILEGES;
EXIT;
```
Stellen Sie sicher, dass Sie die folgenden Einstellungen für mysqld und den mysql-Client wie angegeben haben. Ich habe eine Datei in einem [Github-Repo](https://github.com/SafdariAlireza/ERPNext_mariadb_conf) abgelegt, damit Sie die gesamte Datei kopieren und ersetzen können.
```
sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
sudo systemctl restart mariadb
```

#### Schritt 4: Installieren Sie Bench und ERPNext
Eine Bench ist ein Tool zum Installieren und Verwalten von ERPNext auf Ihrem Ubuntu-System. Wir erstellen einen Benutzer, der das ERPNext-System ausführt, und konfigurieren dann das System.
```
sudo useradd -m -s /bin/bash erpnext
sudo passwd erpnext
sudo usermod -aG sudo erpnext
```
Jetzt ist es an der Zeit, Ihren PATH zu aktualisieren.
```
sudo su - erpnext
tee -a ~/.bashrc<<EOF
PATH=\$PATH:~/.local/bin/
EOF
source ~/.bashrc
```

Als nächstes müssen Sie ein Verzeichnis für das ERPNext-Setup erstellen und erpnext-Benutzern Lese- und Schreibberechtigungen für das Verzeichnis erteilen:
```
sudo mkdir /opt/bench
sudo chown -R erpnext /opt/bench
```
Wechseln Sie als Nächstes zum erpnext-Benutzer und installieren Sie die Anwendung:
```
sudo su - erpnext
cd /opt/bench
```
Frappe-bench und Git installieren
```

sudo apt install git

sudo pip3 install frappe-bench
```

Der nächste Schritt besteht darin, das Bench-Verzeichnis mit installiertem Frappe-Framework zu initialisieren. Stellen Sie sicher, dass Sie sich noch im Verzeichnis /opt/bench befinden:
```
bench init erpnext
```
Erstellen Sie eine neue Frappe-Site.
```
cd erpnext
bench new-site erp.testSite.com 
```
#### Schritt 5: Holen Sie sich die ERPNext-Anwendung von GitHub.
Laden Sie die ERPNext aus dem Frappe-Github-Repo herunter. Wir werden Version 13 verwenden. Sie können jede Version verwenden, die Sie möchten.
```
bench get-app branch version-13 erpnext https://github.com/frappe/erpnext
```
#### Schritt 6: Installieren Sie die ERPNext-App auf unserer Website
```
bench --site erp.testSite.com install-app erpnext
```
#### Schritt 7: Starten Sie ERPNext und schließen Sie die Installation ab
```
bench start
```
Navigieren Sie zur IP-Adresse Ihrer Installation und der Portnummer, die nach dem Ausführen auf dem Terminal angezeigt wird. Verwenden Sie im Fall einer lokalen Instanz 127.0.0.1:8000

Um vom Dev-Mode zu Production-Mode zu wechseln, führen Sie folgende Befehl aus:
```
sudo bench setup production $USER
sudo supervisorctl restart all
```
