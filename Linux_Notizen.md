# Linux Notizen

Dieses Repository enthält meine persönlichen Notizen zur Administration von
Linux. Diese Notizen sind nicht als vollständige Anleitung oder Tutorial 
gedacht, sondern als Referenz und Gedächtnisstütze für mich selbst.

## Inhaltsverzeichnis

* [cron](#cron)
* [Dateien](#dateien)
* [find](#find)
* [GnuPG](#gnupg)
* [Laufwerke](#laufwerke)
* [Locical Volume Manager](#locical-volume-manager)
* [Netzwerk](#netzwerk)
* [Sicherheit](#sicherheit)
* [SSH](#ssh)
* [SWAP](#swap)
* [Ubuntu](#ubuntu)
* [User und groups](#user-und-groups)


## cron

```
1     2     3     4     5    (6)   L
|     |     |     |     |     |    |
|     |     |     |     |     |    +----- Befehl der ausgeführt werden soll
|     |     |     |     |     +------ (Benutzer/Besitzer des Jobs)
|     |     |     |     |
|     |     |     |     +----- Wochentag (0 - 7) (Sonntag ist 0 und 7; oder Namen, siehe unten)
|     |     |     +------- Monat (1 - 12; oder Namen, siehe unten)
|     |     +--------- Tag (1 - 31)
|     +----------- Stunde (0 - 23)
+------------- Minute (0 - 59)
```
*[Zurück zum Inhaltsverzeichnis](#inhaltsverzeichnis)*

## Dateien

## Anzahl der Dateien pro Unterverzeichnis anzeigen
```
$ find . -maxdepth 1 -type d | while read -r dir; do printf "%s:\t" "$dir"; find "$dir" -type f | wc -l; done
```
*[Zurück zum Inhaltsverzeichnis](#inhaltsverzeichnis)*

## find

### Suchen einer Datei nach Namen
```
$ find /path/to/dir -type -f -name file-to-search
```

### Ausführen eines Befehls auf alle Verzeichnisse
```
$ find /path/to/dir -type d -exec <command> {} +
```
#### Beispiel
```
$ find /path/to/dir -type d -exec chmod 755 {} +
```

### Ausführen eines Befehls auf alle Dateien
```
$ find /path/to/dir -type f -exec <command> {} +
```
#### Beispiel
```
$ find /path/to/dir -type d -exec chmod g-x {} +
```
*[Zurück zum Inhaltsverzeichnis](#inhaltsverzeichnis)*

## GnuPG

### Schlüsselpar erstellen
```
$ gpg --full-generate-key
```

### Anzeigen der Schlüssel
#### Öffentliche Schlüssel
```
$ gpg --list-public-keys
```
oder
```
$ gpg --list-public-keys --keyid-format=long
```
#### Privater Schlüssel
```
$ gpg --list-secret-keys
```
oder
```
$ gpg --list-secret-keys --keyid-format=long
```

### Schlüssel exportieren (ASCII)
Die Option ```--armor``` sorgt dafür, dass der Schlüssel im ASCII-Format
exportiert wird. Wenn man die Option weg lässt, erhält man die Schlüssel im
Binärformat. Die Ausgabe im ASCII-Armor-Format benötigt 33 Prozent mehr
Speicherplatz als die im Binärformat, enthält jedoch nur druckbare Zeichen.
Sie eignet sich daher besser zum Übertragen über das Internet, wie etwa bei
E-Mails oder zum Einbinden in HTML-Seiten. Sollen die Daten hingegen
verschlüsselt auf der Festplatte abgelegt werden, ist das Binärformat gerade
bei großen Dateien vorzuziehen.
 
#### Öffentliche Schlüssel
```
$ gpg --output public.pgp --armor --export <Schlüssel ID>
```
#### Privater Schlüssel
```
$ gpg --output public.pgp --armor --export-secret-key <Schlüssel ID>
```

### Schlüssel importieren
1. ```
   $ gpg --import <Schlüsseldatei>
   ```
1. ```
   $ gpg --edit-key <Schlüssel ID>
   ```
1. ```
   > trust
   ```
1. ```
   > save
   ```

### Verschlüsselungen von Dateien
```
$ gpg --encrypt --armor --recipient <id, mail oder 'name'> <Datei>
```

### Entschlüsseln
```
$ gpg --decrypt <Datei>
```

### Ausgabe in Datei schreiben

```
$ gpg --decrypt --output <Ausgabedatei> <cerschlüsselte Datei>
```

### Schlüssel löschen
#### Öffentlicher Schlüssel
```
$ gpg --delete-key <Schlüssel ID>
```
#### Privater Schlüssel
```
$ gpg --delete-secret-key <Schlüssel ID>
```
*[Zurück zum Inhaltsverzeichnis](#inhaltsverzeichnis)*

## Laufwerke

### Laufwerke einbinden
#### Temporär
```
$ sudo mount -type <filesystem> <Quelle> <Verzeichnis> 
```
##### Beispiel
```
$ sudo mount -type ext4 /dev/sdb1 /mnt/data 
```
#### Dauerhaft
1. **```/etc/fstab```**
   ```
    /dev/sdb1       /home/user/disk ext4    defaults        0       0
   ```
1. ```sudo mount -a```
*[Zurück zum Inhaltsverzeichnis](#inhaltsverzeichnis)*


## Locical Volume Manager

### Logical volume einhängen

1. ```
   $ sudo vgsacn --mknodes
   ```
1. ```
   $ sudo vgchange -ay
   ```
1. ```
   $ sudo lvdisplay
   ```
1. ```
   $ sudo mkdir -vp /mnt/mount_point/{root,home}
   ```
1. ```
   $ sudo mount {LV_PATH} /path/to/mount_point
   ```

### Logical volume auf gesamten freien Bereich vergrößern

1. ```
   $ sudo lvextend -l +100%FREE /dev/vg0/lv-var
   ```
1. ```
   $ sudo resize2fs /dev/vg0/lv-var
   ```
*[Zurück zum Inhaltsverzeichnis](#inhaltsverzeichnis)*


## Netzwerk

### Ein bestimmtes network interface ein- und ausschalten
- ```
  $ sudo ip link set enp0s31f6 up
  ```
- ```
  $ sudo ip link set enp0s31f6 down
  ```
*[Zurück zum Inhaltsverzeichnis](#inhaltsverzeichnis)*


## Sicherheit

### Lynis

Mit Lynis kann man ein IT-Sicherheitsaudit durchführen.

#### Installation
```
$ sudo apt install lynis
```

#### Audit durchführen
```
$ sudo lynis audit system --auditor "<dein Name>" --pentest --forensics
```
*[Zurück zum Inhaltsverzeichnis](#inhaltsverzeichnis)*


## SSH

### Schlüsselpaar erzeugen
```
$ ssh-keygen -t ed25519 -a 100
```
oder
```
$ ssh-keygen -t -b 4096 -a 100
```

### Öffentlichen Schlüssel auf Server kopieren
```
ssh-copy-id -i ~/.ssh/id_rsa.pub user@host
```
oder
```
$ cat ~/.ssh/id_rsa.pub | ssh user@host "mkdir -p ~.ssh && cat >> ~/.ssh/authorized_keys"
```

### Passwort des Schlüssels ändern
```
$ ssh-keygen -p
```
```
$ ssh-keygen -p -f ~/.ssh/id_rsa
```
*[Zurück zum Inhaltsverzeichnis](#inhaltsverzeichnis)*


## SWAP

### Label für Swap vergeben
```
$ sudo swaplabel -L swap /dev/sdaX
```

### Zusammenfassung des SWAPs anzeigen
```
$ sudo swapon --show
```
*[Zurück zum Inhaltsverzeichnis](#inhaltsverzeichnis)*


## Ubuntu

### Upgrade der Ubuntu Version

1.  ```
     $ sudo apt update && sudo apt upgrade -y
    ```
1.  ```
    $ sudo apt install ubuntu-release-upgrader-core
    ```
1.  In der Konfigurationsdatei ```normal``` oder ```lts``` festlegen
    ```
    $ sudo vim /etc/update-manager/release-upgrades
    ```
1.  ```
    $ sudo do-release-upgrade 
    ```
*[Zurück zum Inhaltsverzeichnis](#inhaltsverzeichnis)*


## User und groups

### useradd vs adduser
Der Befehl ```useradd``` ist eine niederigere Stufe und auf allen Linux-
Distributionen verfügbar. Er fordert zusätzliche Parameter, um das Konto
vollständig einzurichten. Der Befehl ```adduser``` ist eine höhere Stufe und
nicht auf allen Linux-Distributionen verfügbar. Mit diesem Befehl wird dem
System ein Benutzer mit Standarteinstellungen hinzugefügt.

### User ohne login erstellen
```
$ sudo adduser <user> --shell /usr/sbin/nologin
```

oder

```
$ sudo adduser <user> --disable-login
```

### Login für einen user deaktivieren
```
$ sudo usermod <user> -s /sbin/nologin
```

### Systemuser ohne home -Verzeichnis
```
$ sudo adduser --system --no-create-home <user>
```
*[Zurück zum Inhaltsverzeichnis](#inhaltsverzeichnis)*
