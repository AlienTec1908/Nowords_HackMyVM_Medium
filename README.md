# Nowords - HackMyVM (Medium)

![Nowords.png](Nowords.png)

## Übersicht

*   **VM:** Nowords
*   **Plattform:** HackMyVM (https://hackmyvm.eu/machines/machine.php?vm=Nowords)
*   **Schwierigkeit:** Medium
*   **Autor der VM:** DarkSpirit
*   **Datum des Writeups:** 13. Oktober 2022
*   **Original-Writeup:** https://alientec1908.github.io/Nowords_HackMyVM_Medium/
*   **Autor:** Ben C.

## Kurzbeschreibung

Das Ziel dieser Challenge war es, Root-Rechte auf der Maschine "Nowords" zu erlangen. Der initiale Zugriff erfolgte durch das Brute-Forcen von FTP-Credentials für den Benutzer `sophie`. Nach dem FTP-Login wurde ein ungewöhnlicher Mechanismus ausgenutzt: Ein in das beschreibbare Verzeichnis `/me/` hochgeladenes Shell-Skript (`script.sh`) konnte durch den Upload einer speziell benannten Bilddatei (`command.jpg`) in Sophies Home-Verzeichnis zur Ausführung gebracht werden. Dies führte zu einer Reverse Shell als Benutzer `sophie`. Die finale Rechteausweitung zu Root gelang durch Ausnutzung der bekannten Polkit-Schwachstelle "PwnKit" (CVE-2021-4034) mittels Metasploit.

## Disclaimer / Wichtiger Hinweis

Die in diesem Writeup beschriebenen Techniken und Werkzeuge dienen ausschließlich zu Bildungszwecken im Rahmen von legalen Capture-The-Flag (CTF)-Wettbewerben und Penetrationstests auf Systemen, für die eine ausdrückliche Genehmigung vorliegt. Die Anwendung dieser Methoden auf Systeme ohne Erlaubnis ist illegal. Der Autor übernimmt keine Verantwortung für missbräuchliche Verwendung der hier geteilten Informationen. Handeln Sie stets ethisch und verantwortungsbewusst.

## Verwendete Tools

*   `arp-scan`
*   `nmap`
*   `gobuster`
*   `hydra`
*   `ftp`
*   `ncftp` (impliziert für `put`)
*   `cp`
*   `cat`
*   `vi` / `nano`
*   `bash`
*   `mv`
*   `nc` (netcat)
*   `msfconsole` (Metasploit Framework)
*   Python3
*   Standard Linux-Befehle (`find`, `id`, `pwd`, `ls`, `export`)

## Lösungsweg (Zusammenfassung)

Der Angriff auf die Maschine "Nowords" gliederte sich in folgende Phasen:

1.  **Reconnaissance & Web/FTP Enumeration:**
    *   IP-Adresse des Ziels (192.168.2.147) mit `arp-scan` identifiziert. Hostname `word.hmv` in `/etc/hosts` eingetragen (obwohl später nicht primär genutzt).
    *   `nmap`-Scan offenbarte Port 21 (FTP, vsftpd 3.0.3) und Port 80 (HTTP, Nginx 1.18.0).
    *   `gobuster` auf Port 80 fand `index.html` und `robots.txt`.
    *   `robots.txt` oder `index.html` enthielt den Hinweis auf den Benutzernamen `sophie`.

2.  **Initial Access (FTP Brute-Force & RCE als `sophie`):**
    *   Mittels `hydra` wurde ein Brute-Force-Angriff auf den FTP-Dienst für den Benutzer `sophie` mit `rockyou.txt` durchgeführt. Das Passwort `natalia` wurde gefunden.
    *   Login per FTP als `sophie:natalia`. Im FTP-Root wurde ein für alle beschreibbares Verzeichnis `/me` gefunden. In `/me` lag die User-Flag (`Icanreadyeah`), die heruntergeladen wurde.
    *   Ein Bash-Reverse-Shell-Skript (`script.sh`) wurde erstellt und in das Verzeichnis `/me` hochgeladen.
    *   Eine Bilddatei (`command.jpg`) wurde erstellt, die als Text den Befehl `/bin/bash /home/me/script.sh` enthielt (vermutlich durch Screenshot und Umbenennung). Diese Datei wurde in das Home-Verzeichnis von `sophie` (`/home/sophie/`) hochgeladen.
    *   Ein (nicht näher spezifizierter) Mechanismus auf dem Zielsystem (wahrscheinlich ein Skript wie `doit.py` aus `/me`) überwachte `/home/sophie`, extrahierte den Text aus `command.jpg` und führte ihn aus.
    *   Dies löste `/home/me/script.sh` aus und eine Reverse Shell als Benutzer `sophie` wurde auf einem Netcat-Listener empfangen.

3.  **Privilege Escalation (von `sophie` zu `root` via Polkit/PwnKit):**
    *   Die erhaltene `sophie`-Shell wurde mittels Metasploit (`exploit/multi/handler`) empfangen und zu einer Meterpreter-Session (`post/multi/manage/shell_to_meterpreter`) aufgerüstet.
    *   Der Exploit `exploit/linux/local/polkit_dbus_auth_bypass` (PwnKit, CVE-2021-4034) wurde ausgewählt, auf die Meterpreter-Session von `sophie` gesetzt und erfolgreich ausgeführt.
    *   Dies führte zu einer neuen Meterpreter-Session mit Root-Rechten.
    *   Eine interaktive Root-Shell wurde geöffnet.
    *   Die Root-Flag (`Ihavenowords`) wurde in `/root/root.txt` gefunden.

## Wichtige Schwachstellen und Konzepte

*   **Schwaches FTP-Passwort:** Das Passwort für `sophie` (`natalia`) konnte durch Brute-Force erraten werden.
*   **Unsicherer Dateiupload- und Ausführungsmechanismus:** Ein benutzerdefinierter Mechanismus erlaubte das Hochladen eines Shell-Skripts und dessen Ausführung durch das Hochladen einer "Trigger"-Bilddatei, die einen Befehl als Text enthielt.
*   **Global beschreibbares Verzeichnis:** Das Verzeichnis `/me` war für alle beschreibbar und wurde genutzt, um das Payload-Skript zu platzieren.
*   **Kernel/System-Schwachstelle (PwnKit / CVE-2021-4034):** Eine bekannte Schwachstelle im Polkit-Dienst ermöglichte die lokale Privilegieneskalation zu Root.

## Flags

*   **User Flag (`/me/user.txt`):** `Icanreadyeah`
*   **Root Flag (`/root/root.txt`):** `Ihavenowords`

## Tags

`HackMyVM`, `Nowords`, `Medium`, `FTP Brute-Force`, `RCE`, `File Upload Exploit`, `Polkit Exploit`, `PwnKit`, `CVE-2021-4034`, `Metasploit`, `Linux`, `Privilege Escalation`, `Nginx`, `vsftpd`
