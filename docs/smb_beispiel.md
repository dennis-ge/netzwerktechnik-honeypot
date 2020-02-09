# Honeypot SMB Beispiel

Ein Beispiel dafür, wie Sicherheitslücken in einem Honeypot dargestellt werden können, kann anhand einer 2017 bekannt gewordenen Sicherheitslücke, die das Netzprotokoll Server Message Block (SMB) in der Version 1 betrifft, dargestellt werden.

## SMB Sicherheitslücke

Durch das Schadprogramm WannaCry, welches diese Sicherheitslücke ausnutzt, sind 2017 viele Windowssysteme befallen worden.
Es basiert auf EternalBlue. Eternal Blue ist ein von einer der NSA-nahen Hacker Gruppe entdeckter Exploit, der verschiedene Sicherheitslücken, u.A. auch CVE-2017-0143 im Microsoft Server Message Block ausnutzt. Ein Hacker, der diese remote code execution Sicherheitslücke ausgenutzt hat, kann unter Anderem eigenen Code auf dem System ausführen. Alle Windowssysteme, auf denen der entsprechenden Patch MS-17-010 nicht durchgeführt wurde, sind von dieser Sicherheitslücke betroffen. Die Sicherheitslücke wird auch mit MS-17-010 referenziert.
Die NSA nutzte diese Sicherheitslücke im SMB über mehr als 5 Jahre und ließ Microsoft darüber im Dunkeln. Erst nachdem Informationen über EternalBlue geleaked wurden, wurde Microsoft kontaktiert und es konnte das Sicherheitsupdate MS17-010 veröffentlicht werden. Da das Update jedoch nicht sofort auf allen Systemen installiert wurde, ist der Exploit, also die bestehende Möglichkeit die Sicherheitslücke auszunutzen, durch Schadsoftware wie WannaCry ausgenutzt worden.
Wenn ein Computer befallen ist, können z.B. einige Benutzerdateien verschlüsselt werden und der Nutzer durch Bitcoinzahlung dazu erpresst werden, sie entschlüsseln zu lassen. Darüber hinaus versucht das Programm SMB auszunutzen, da dadurch z.B. der Zugriff auf Dateien eines anderen Rechners im Netzwerk ermöglicht wird, und so auf weitere Rechner im Netzwerk überzugreifen und installiert ebenso die Backdoor DoublePulsar.
Double Pulsar wurde ebenfalls von der NSA entwickelt und von einer Hacker Gruppe geleaked. Durch Double Pulsar können Hacker z.B. Malware auf ein infiziertes System nachladen.

Um WannaCry ransomware mit dem Honeypot dionaea zu sammeln, wurde dieser 2017 erweitert, so dass das Windowssystem bereits eine DoublePulsar Backdoor, also einen Zugang mit Umgehung der normalen Sicherung, installiert hat.

## Dionaea Implementierung der Sicherheitslücke

Dionaea besitzt für jeden Service/Protokoll das es anbietet eine Konfigurationsdatei, die unter `dionaea/dist/etc/services` zu finden ist. Bei SMB wäre das dann die `smb.yaml` Datei. Wichtig erstmals ist folgender Ausschnitt aus dieser Datei.

```yaml
  native_os: Windows 7 Professional 7600
   native_lan_manager: Windows 7 Professional 6.1
```

Hier wird festgelegt, welches Betriebssystem der Server besitzen soll. Da es bei Windows 7 die oben angesprochene Sicherheitslücke gab,  wird dies ausgewählt. Wenn man nun mithilfe eines Tools (wir haben [nmap](https://nmap.org/) verwendet) den Port 445 der Virtuellen Maschine scannt, dann bekommt man folgendes Ergebnis:

![Nmap Scan Ergebnis](/assets/nmap_port_scan.png)

Hier sind nun unter *smb.os-discovery* unter anderem die eben konfigurierten Daten zu sehen. Um an weitere Informationen über den SMB Server zu gelangen ist ein weiterer Schritt den Server nach beliebigen SMB Sicherheitslücken zu untersuchen. Um dies durchzuführen gibt es  mehrere Möglichkeiten, wir haben wieder nmap verwendet. Im nachfolgenden Bild ist das Ergebnis zu sehen.

![Nmap Script Scan Ergebnis](/assets/nmap_scripts_port_scan.png)

Der SMB Server wurde hierbei auf vier vergangene Sicherheitslücken überprüft (beginnend mit smb-vuln-ms*).  Die zwei Skripten *smb-vuln-ms08-067* und *smb-vuln-ms17-010*  wurden erfolgreich ausgeführt, die Sicherheitslücken sind also auf dem Server vorhanden, wobei hier der Fokus auf dem zweitem Skript liegt, da das die oben beschriebene Sicherheitslücke ist.  

Der nächste Schritt ist es die Sicherheitslücke nun auszunutzen. Hierzu verwenden wir das [Metasploit Framework](https://www.metasploit.com/), da man hier  Sicherheitslücken auswählen und ausführen kann. Nach dem Download startet der Befehl `msfconsole` die Metasploit Konsole. Danach wird mit dem Befehl `use exploit/windows/smb/ms17_010_eternalblue` auf die Sicherheitslücke zugegriffen. Die Optionen kann man sich mithilfe von `show options` anzeigen lassen. Das Ergebnis sollte gleich wie im nachfolgenden Bild aussehen.

![Metasploit](/assets/metasploit_options.png)

Zu sehen ist, dass die Einstellung *RHOSTS* noch leer ist. Diese füllt man mit  `set rhosts <server-ip>`.  Jetzt kann man die Sicherheitslücke mit dem Befehl `exploit` noch ausnutzen. Danach hat man Zugriff auf den SMB Server. Der Angreifer kann nun alles möglichen auf den Server hochladen.

Da es sich hier aber nun um den SMB Server von Dionaea handelt, richten diese Dateien keine Schaden an. Dionaea fertigt eine Kopie der Dateien an und speichert diese. Was dann mit den Kopien geschieht liegt beim Administrator des Honeypots. Mit Dionaea ist es zum Beispiel möglich die Kopien zu [VirusTotal](https://www.virustotal.com/gui/home/upload) zu schicken, um zu überprüfen, um welche Art von Malware es sich handelt. Außerdem werden alle Vorkommnisse in einer SQLite Datenbank und JSON Datei gespeichert gespeichert.

## Quellen:
- https://de.wikipedia.org/wiki/WannaCry
- https://www.csoonline.com/article/3227906/what-is-wannacry-ransomware-how-does-it-infect-and-who-was-responsible.html
- https://portal.msrc.microsoft.com/en-US/security-guidance/advisory/CVE-2017-0143
- https://www.honeynet.org/2017/05/30/dionaea-honeypot-from-conficker-to-wannacry-sambacry-cve-2017-7494/
- https://www.hackingarticles.in/smb-penetration-testing-port-445/
