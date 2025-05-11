# Dance - HackMyVM (Hard)

![Dance.png](Dance.png)

## Übersicht

*   **VM:** Dance
*   **Plattform:** [HackMyVM](https://hackmyvm.eu/machines/machine.php?vm=Dance)
*   **Schwierigkeit:** Hard
*   **Autor der VM:** DarkSpirit
*   **Datum des Writeups:** 20. Oktober 2022
*   **Original-Writeup:** https://alientec1908.github.io/Dance_HackMyVM_Hard/
*   **Autor:** Ben C.

## Kurzbeschreibung

Ziel dieser "Hard"-Challenge war es, Root-Zugriff auf der Maschine "Dance" zu erlangen. Der Weg begann mit der Entdeckung eines offenen FTP-Dienstes mit anonymem Zugriff und eines Webservers. Eine Local File Inclusion (LFI)-Schwachstelle auf dem Webserver (nginx) wurde ausgenutzt, um ein ZIP-Archiv des `/var`-Verzeichnisses herunterzuladen. Darin befand sich eine `config.php` mit Klartext-Zugangsdaten für die Benutzer `aria` und `alba`. Mit `aria`s Credentials wurde ein SSH-Zugang erlangt. Das Passwort von `alba` wurde dann genutzt, um via `su` zu diesem Benutzer zu wechseln. Eine unsichere `sudo`-Regel erlaubte es `alba`, das Programm `espeak` als Root auszuführen. Durch geschickte Nutzung der `espeak`-Optionen konnte dies als Local File Inclusion missbraucht werden, um die Root-Flag auszulesen.

## Disclaimer / Wichtiger Hinweis

Die in diesem Writeup beschriebenen Techniken und Werkzeuge dienen ausschließlich zu Bildungszwecken im Rahmen von legalen Capture-The-Flag (CTF)-Wettbewerben und Penetrationstests auf Systemen, für die eine ausdrückliche Genehmigung vorliegt. Die Anwendung dieser Methoden auf Systeme ohne Erlaubnis ist illegal. Der Autor übernimmt keine Verantwortung für missbräuchliche Verwendung der hier geteilten Informationen. Handeln Sie stets ethisch und verantwortungsbewusst.

## Verwendete Tools

*   `arp-scan`
*   `nmap`
*   `wget`
*   `unzip`
*   `ssh`
*   `su`
*   `sudo`
*   `espeak`
*   Standard Linux-Befehle (`cd`, `cat`, `ls`)

## Lösungsweg (Zusammenfassung)

Der Angriff auf die Maschine "Dance" gliederte sich in folgende Phasen:

1.  **Reconnaissance & Enumeration:**
    *   IP-Findung mittels `arp-scan` (Ziel: `192.168.2.158`).
    *   Umfassender `nmap`-Scan identifizierte offene Ports:
        *   Port 21 (FTP): vsftpd 3.0.3, **anonymer Login erlaubt**.
        *   Port 22 (SSH): OpenSSH 8.4p1 (Debian).
        *   Port 80 (HTTP): nginx 1.18.0.

2.  **Web LFI Exploitation & Credential Discovery:**
    *   Eine Local File Inclusion (LFI) / Path Traversal Schwachstelle wurde auf dem Webserver unter `/music/` identifiziert.
    *   Mittels `wget` wurde die Schwachstelle ausgenutzt, um das Verzeichnis `/var` als ZIP-Datei (`var.zip`) herunterzuladen: `wget 'http://192.168.2.158/music/?getAlbum&parent=../../&album=var' -O var.zip`.
    *   Nach dem Entpacken von `var.zip` wurde in der Struktur (vermutlich `var/www/html/music/`) die Datei `config.php` gefunden.
    *   Die `config.php` enthielt Klartext-Zugangsdaten, u.a. für `aria:seraphim` und `alba:thehostof`.

3.  **Initial Access (SSH als aria):**
    *   Mit den gefundenen Zugangsdaten wurde eine SSH-Verbindung als Benutzer `aria` zum Zielsystem hergestellt: `ssh aria@dance` (Passwort: `seraphim`).
    *   Die User-Flag wurde im Home-Verzeichnis von `aria` gefunden und ausgelesen.

4.  **Privilege Escalation (von aria zu alba):**
    *   Mit dem aus der `config.php` bekannten Passwort für `alba` (`thehostof`) wurde mittels `su -s /bin/bash alba` zum Benutzer `alba` gewechselt.

5.  **Privilege Escalation (von alba zu root via Sudo/Espeak LFI):**
    *   Der Befehl `sudo -l` als `alba` zeigte, dass der Benutzer `/usr/bin/espeak` als `root` ohne Passwort ausführen darf: `(root) NOPASSWD: /usr/bin/espeak`.
    *   `espeak` hat eine Option `-f <dateiname>`, um Text aus einer Datei zu lesen. Da es als Root ausgeführt wird, konnte dies zum Lesen beliebiger Dateien genutzt werden.
    *   Die Root-Flag wurde durch folgenden Befehl ausgelesen: `sudo /usr/bin/espeak -f /root/root.txt -q -X`. Die Option `-q` unterdrückte die Sprachausgabe und `-X` zeigte Phonem-Mnemonics, wodurch der Inhalt der Datei (`deadcandance`) in der Terminalausgabe erschien.

## Wichtige Schwachstellen und Konzepte

*   **Anonymer FTP-Zugriff:** Obwohl im Lösungsweg nicht direkt ausgenutzt, stellte dies einen initialen Informationspunkt dar.
*   **Local File Inclusion (LFI) / Path Traversal (Web):** Eine Schwachstelle in einer Webanwendung (`/music/`) ermöglichte das Herunterladen beliebiger Verzeichnisse (wie `/var`) durch Manipulation von URL-Parametern (`parent`, `album`).
*   **Hardcoded Credentials in Konfigurationsdatei:** Klartext-Passwörter in `config.php` (gefunden durch LFI) ermöglichten den direkten Zugriff als `aria` und `alba`.
*   **Unsichere `sudo`-Regel (LFI über `espeak`):** Die Erlaubnis für den Benutzer `alba`, `espeak` als Root auszuführen, in Kombination mit der `-f` Option von `espeak`, führte zu einer Local File Inclusion, die das Auslesen der Root-Flag ermöglichte.
*   **Passwort-Wiederverwendung / Klartext-Speicherung:** Die Entdeckung von Passwörtern für mehrere Benutzer an einer Stelle erleichterte die laterale Bewegung und Eskalation.

## Flags

*   **User Flag (`/home/aria/user.txt`):** `godisadj`
*   **Root Flag (`/root/root.txt`):** `deadcandance`

## Tags

`HackMyVM`, `Dance`, `Hard`, `LFI`, `Path Traversal`, `Web`, `nginx`, `FTP`, `SSH`, `sudo`, `espeak`, `Privilege Escalation`, `Hardcoded Credentials`, `Linux`
