## Einloggen in den Raspberry Pi 
Die Anmeldung erfolgt mit dem Benutzer _pi_ und der IP-Adresse: `ssh pi@192.168.24.xxx`, wobei _xxx_ für eine der vorher vergebenen IP-Adressen steht. Es folgt die Passwortabfrage für den Benutzer.
# Statische IP-Adresse
Im ersten Schritt sollten das Standard-Gateway und die Adresse des DNS-Servers ermittelt werden. Dafür kann der Befehl `ip r` verwendet werden. Als Ergebnis erhält man in diesem Fall die Adresse _192.168.24.0/24_ sowohl für das Standard-Gateway als auch für den DNS-Server.
Der NetworkManager wird als Standard-Controller für die Netzwerkkonfiguration verwendet. 
Mit dem Befehl `sudo nmcli -p connection show` erhält man eine Liste der Verfügbaren Netzwerkschnittstellen. 
Der Type _ethernet_ ist – lautet dann _Wired connection 1_.
Es folgen drei Befehle, in denen die neue IP-Adresse, die Standard-Gateway-Adresse und die DNS-Server-Adresse aktualisiert werden:
```
sudo nmcli c mod “Wired connection 1“ ipv4.addresses 192.168.24.181/24 ipv4.method manual
sudo nmcli con mod “Wired connection 1“ ipv4.gateway 192.168.24.0/24
sudo nmcli con mod “Wired connection 1“ ipv4.dns 192.168.24.0/24
```
Nach der Änderung sollte ein Neustart vorgenommen werden (`sudo reboot`), um die Änderungen wirksam zu machen.
# Zwei lokale Benutzer
Der Benutzer „willi“ soll ein normaler Benutzer ohne Administratorrechte sein. 
Der Benutzer „fernzugriff“ soll ein Benutzer für den Zugriff von außen mittels SSH mit sudo-Rechten sein. 
## Benutzer „willi“ und „fernzugriff“
Für das Anlegen der beiden Benutzer sollte mit sudo-Rechten gearbeitet werden, aktiviert durch: `sudo -i`

Als nächstes werden die beiden Benutzer mit einem Home-Verzeichnis angelegt. Dafür wird der Befehl `useradd -m {USERNAME}` verwendet. Im Anschluss daran erfolgt mit `passwd {USERNAME} {PASSWORD}` eine Passwortvergabe.

## sudo-Rechte für „fernzugriff“ 
Für die Vergabe von sudo-Rechten muss die sudoers-Datei bearbeitet werden. Durch den Befehl `sudo visudo` kann man diese Datei editieren. Am Ende der Datei muss die folgende Zeile hinzugefügt werden: `fernzugriff ALL=(ALL) NOPASSWD:ALL`.

## SSH-Dienst für „fernzugriff“
Um für den Benutzer „fernzugriff“ einen SSH-Dienst zur Administration einrichten zu können, sollte zunächst geschaut werden, dass der SSH-Dienst bereits aktiviert ist und läuft: `sudo systemctl status ssh`

Sollte der SSH-Dienst nicht installiert sein, so muss dieser noch installiert und aktiviert werden:
```
sudo apt update
sudo apt install openssh-server
sudo systemctl enable ssh
sudo systemctl start ssh
```
## SSH-Zugriff konfigurieren 
Als nächstes muss der SSH-Zugriff für den Benutzer „fernzugriff“ konfiguriert werden. Dafür muss die SSH-Konfigurationsdatei bearbeitet werden: `sudo nano /etc/ssh/sshd_config`. 
Um den SSH-Zugriff einzig auf den Benutzer „fernzugriff“ zu beschränken, muss die Zeile `AllowUsers fernzugriff` entweder der Datei hinzugefügt oder aber, wenn schon vorhanden, für den Benutzer „fernzugriff“ geändert werden.

Mit dem Befehl `sudo systemctl restart ssh` sollte anschließend der SSH-Dienst neu gestartet werden, um die Änderungen zu übernehmen.
