# Aufgabe 1: SSH Setup

Hier könnte Ihre Werbung stehen.

(Der SSH Zugriff war erfolgreich. `*** System restart required ***`)

# Aufgabe 2: SSH Login mit Schlüsselpaar

## Bearbeitung

1.	**Auf dem Client-PC einen SSH-Key erstellen.**
Das geht mit dem Kommando `ssh-keygen -t ed25519`. Der Parameter `-t` (für "type") legt dabei den Typ von Schlüsselalgorithmus fest der erzeugt werden soll (hier `ed25519` weil Standardempfehlung). Es wurde nach einem Dateinamen bzw Dateipfad und einem Passphrase gefragt. Zugunsten der Aufgabe habe ich diese leer gelassen, schliesslich soll die Anmeldung OHNE Password sein.
2.	**Schlüssel in den Server kopieren.**
Unter Unix-ähnlichen Systemen (Mac und Linux) kann man einfach per Kommando `ssh-copy-id [benutzer]@[server]` den neuen SSH-Key in den Server rüberkopieren. Unter Windows ging das etwas sperriger, führte aber zum selben Ergebnis.  Erstmal musste ich den Inhalt des neuen Public-Key mit `Get-Content $env:USERPROFILE\.ssh\id_ed25519.pub` anzeigen lassen und kopieren. Anschliessend in den SSH Server mit Passwort einloggen und mit `mkdir -p ~\.ssh` sicherheitshalber den Pfad erstellen, falls noch nicht vorhanden (dafür der Parameter `-p`). Danach Rechte für den Pfad erstellen (`chmod 700 ~\.ssh`) und mit einem Editor der Wahl die Datei `authorized_keys` erstellen, in welche der vorhin kopierte Inhalt eingefügt wird. Zum Schluss nochmal Rechte auf die Datei vergeben (`chmod 600 ~\.ssh\authorized_keys`).
3.	**Testen, ob der Schlüssel funktioniert.**
Hierfür habe ich einfach `ssh [benutzer]@[server]` eingegeben. Es wurde keine weitere Authentifizierung gefragt und ich wurde direkt in den Server eingeloggt.

## Warum sind SSH-Keypairs sicherer als eine einfache Passwort-Authentifizierung?
SSH-Schlüssel sind sicherer, weil der private Schlüssel an einen bestimmten Rechner (bzw. eine bestimmte Schlüsseldatei) gebunden ist und nie an den Server übertragen wird. Passwörter hingegen können erraten, wiederverwendet, abgefangen oder weitergegeben werden und sind nicht an ein bestimmtes Gerät gebunden.

# Aufgabe 3: SSH Basics und Nutzerberechtigungen

1.	`mkdir 1 2`
2.	`scp file [benutzer]@[server]:~/1`
3.	`cd ../2`, `nano script.sh`, `chmod +x ./script.sh`, `./script.sh`
4.	`chmod 751 ./script.sh`

	Die gesetzen Berechtigungen waren beim Erstellen der Datei waren zuvor `-rw-rw-r--`, danach `-rwxr-x--x`. Der erste Strich heißt lediglich, dass es sich hierbei um eine einfache Datei handelt. Wären weitere Unterpfade enthalten wären diese stattdessen mit `d` markiert. Dann kommen drei Blöcke der Form `rwx`, wobei `r` für "Read" (Lesen), `w` für "Write" (Schreiben) und `x` für "Execute" (Ausführen) steht. Der erste Block zeigt die Rechte des Nutzers welcher die Datei besitzt. Das ist auch der erste Nutzername der bei `ls -l` auftaucht. Der zweite Block zeigt die Rechte der Nutzergruppe welche die Datei besitzt. Das ist auch der zweite Name der bei `ls -l` auftaucht. Jeder Nutrzer Der dritte Block zeigt die Rechte der "Anderen", also derjenigen, welche die Datei nicht besitzten. `rwxr-x--x` heißt also: Der Besitzer darf alles, andere Gruppenmitglieder darf nur lesen und ausführen (`r-x`), der Rest darf nur ausführen (`--x`).

# Aufgabe 4

Eingabe

```sh
$ wget -O FPO2020.pdf https://www.uni-trier.de/fileadmin/studium/STUDIENVERLAUFSINFOS/ORDNUNGEN/PruefungsO/1-BA/FPO_BA/FPO_BA_Informatik/FPO_BA_Informatik_1F/FPO_BA_Informatik_1F_2020/FPO_BSc_1F-HF-NF_Informatik_2020-07-27.pdf
```

Ausgabe

```text
--2026-06-13 19:41:48--  https://www.uni-trier.de/fileadmin/studium/STUDIENVERLAUFSINFOS/ORDNUNGEN/PruefungsO/1-BA/FPO_BA/FPO_BA_Informatik/FPO_BA_Informatik_1F/FPO_BA_Informatik_1F_2020/FPO_BSc_1F-HF-NF_Informatik_2020-07-27.pdf
Resolving www.uni-trier.de (www.uni-trier.de)... 136.199.189.15
Connecting to www.uni-trier.de (www.uni-trier.de)|136.199.189.15|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 107056 (105K) [application/pdf]
Saving to: ‘FPO2020.pdf’

FPO2020.pdf                                        100%[==============================================================================================================>] 104.55K  --.-KB/s    in 0.01s   

2026-06-13 19:41:48 (7.47 MB/s) - ‘FPO2020.pdf’ saved [107056/107056]
```

# Aufgabe 5

Eingabe:

```sh
ssh -L 8080:localhost:3000 benutzer@server
```

Dabei bedeutet `-L`, dass ein lokaler Port auf meinem Rechner an einen Port auf dem entfernten Server weitergeleitet wird. Der lokale Port 8080 wird dabei mit dem Port 3000 des Servers verbunden.

Nach dem Aufbau der SSH-Verbindung konnte ich im Browser die Adresse `http://localhost:8080` aufrufen. Die Anfragen wurden dabei nicht an einen Webserver auf meinem eigenen Rechner gesendet, sondern durch den SSH-Tunnel verschlüsselt an den Server weitergeleitet. Auf dem Server wurden die Anfragen an den lokal laufenden Webserver auf Port 3000 übergeben. Die Antworten des Webservers wurden anschließend über denselben SSH-Tunnel zurück an meinen Browser übertragen.

Der Datenfluss ist somit:

Browser → localhost:8080 → SSH-Tunnel → Server (localhost:3000) → Webserver

und für die Antwort:

Webserver → Server (localhost:3000) → SSH-Tunnel → localhost:8080 → Browser

Durch die lokale Portweiterleitung konnte auf einen Webserver zugegriffen werden, der nur lokal auf dem Server erreichbar ist und von außen keine direkten Verbindungen akzeptiert.