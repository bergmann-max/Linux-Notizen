# Linux Notizen

> **Hinweis** \
> Dieses Repository enthält meine persönlichen Notizen zur Administration von
> Linux. Diese Notizen sind nicht als vollständige Anleitung oder Tutorial
> gedacht, sondern als Referenz und Gedächtnisstütze für mich selbst.

## Inhaltsverzeichnis

- [cron](#cron)
- [Dateien](#dateien)
- [find](#find)
- [GnuPG](#gnupg)
- [Laufwerke](#laufwerke)
- [Locical Volume Manager](#locical-volume-manager)
- [Netzwerk](#netzwerk)
- [Sicherheit](#sicherheit)
- [SSH](#ssh)
- [SSL](#ssl)
- [SWAP](#swap)
- [top](#top)
- [Ubuntu](#ubuntu)
- [User und groups](#user-und-groups)

## cron

### Syntax

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

### Anzeigen der crontab aller user

```
$ for user in $(getent passwd | cut -f1 -d:); do echo $user; sudo crontab -u $user -l; done
```

_[⬆️ Zurück zum Inhaltsverzeichnis](#inhaltsverzeichnis)_

## Dateien

## Anzahl der Dateien pro Unterverzeichnis anzeigen

```
$ find . -maxdepth 1 -type d | while read -r dir; do printf "%s:\t" "$dir"; find "$dir" -type f | wc -l; done
```

_[⬆️ Zurück zum Inhaltsverzeichnis](#inhaltsverzeichnis)_

## find

### Suchen einer Datei nach Namen

```
$ find /path/to/dir -type f -name file-to-search
```

### Ausführen eines Befehls auf alle Verzeichnisse

```
$ find /path/to/dir -type d -exec [BEFEHL] {} +
```

#### Beispiel

```
$ find /path/to/dir -type d -exec chmod 755 {} +
```

### Ausführen eines Befehls auf alle Dateien

```
$ find /path/to/dir -type f -exec [BEFEHL] {} +
```

#### Beispiel

```
$ find /path/to/dir -type d -exec chmod g-x {} +
```

_[⬆️ Zurück zum Inhaltsverzeichnis](#inhaltsverzeichnis)_

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

Die Option `--armor` sorgt dafür, dass der Schlüssel im ASCII-Format
exportiert wird. Wenn man die Option weg lässt, erhält man die Schlüssel im
Binärformat. Die Ausgabe im ASCII-Armor-Format benötigt 33 Prozent mehr
Speicherplatz als die im Binärformat, enthält jedoch nur druckbare Zeichen.
Sie eignet sich daher besser zum Übertragen über das Internet, wie etwa bei
E-Mails oder zum Einbinden in HTML-Seiten. Sollen die Daten hingegen
verschlüsselt auf der Festplatte abgelegt werden, ist das Binärformat gerade
bei großen Dateien vorzuziehen.

#### Öffentliche Schlüssel

```
$ gpg --output public.pgp --armor --export [SCHLÜSSEL ID]
```

#### Privater Schlüssel

```
$ gpg --output public.pgp --armor --export-secret-key [SCHLÜSSEL ID]]
```

### Schlüssel importieren

1. ```
   $ gpg --import [SCHLÜSSELDATEI]
   ```
1. ```
   $ gpg --edit-key [SCHLÜSSEL ID]]
   ```
1. ```
   > trust
   ```
1. ```
   > save
   ```

### Verschlüsselungen von Dateien

```
$ gpg --encrypt --armor --recipient [ID | MAIL | 'NAME'] [Datei]
```

### Entschlüsseln

```
$ gpg --decrypt [DATEI]
```

### Ausgabe in Datei schreiben

```
$ gpg --decrypt --output [AUSGABEDATEI] [VERSCHLÜSSELTE DATEI]
```

### Schlüssel löschen

#### Öffentlicher Schlüssel

```
$ gpg --delete-key [SCHLÜSSEL ID]
```

#### Privater Schlüssel

```
$ gpg --delete-secret-key [SCHLÜSSEL ID]
```

_[⬆️ Zurück zum Inhaltsverzeichnis](#inhaltsverzeichnis)_

## Laufwerke

### Laufwerke einbinden

#### Temporär

```
$ sudo mount -type [FILESYSTEM] [QUELLE] [VERZEICHNIS]
```

##### Beispiel

```
$ sudo mount -type ext4 /dev/sdb1 /mnt/data
```

#### Dauerhaft

1. _/etc/fstab_
   ```
    /dev/sdb1       /home/user/disk ext4    defaults        0       0
   ```
1. `sudo mount -a`

_[⬆️ Zurück zum Inhaltsverzeichnis](#inhaltsverzeichnis)_

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

_[⬆️ Zurück zum Inhaltsverzeichnis](#inhaltsverzeichnis)_

### Samba Share mounten

1. ```
   $ sudo apt install keyutils cifs-utils
   ```
1. Es ist zwar möglich, die Anmeldedaten für Samba Share in die Datei
   `/etc/fstab` einzutragen, aber es ist besser, die Zugangsdaten für das
   Samba Share in einer separaten Datei (z. B. `/home/user1/.smb` oder
   `/root/.smb`) zu speichern und die Zugriffsrechte auf das Notwendigste zu
   beschränken.
   ```
   user=user1
   password=P4zzw0rd
   domain=myDomain
   ```
1. _/etc/fstab_
   ```
   //windows/server  /path/to/mountponit     cifs    uid=0,credentials=/root.smb,iocharset=utf8,vers=3.0,noperm 0 0
   ```
1. ```
      $ sudo mount -a
   ```
   _[⬆️ Zurück zum Inhaltsverzeichnis](#inhaltsverzeichnis)_

## Netzwerk

### Netwrok Interface Einschalten, Ausschalten und Neustarten

```
$ sudo systemctl [start|stop|restart|status] systemd-networkd.service
```

### Ein bestimmtes network interface ein- und ausschalten

- ```
  $ sudo ip link set enp0s31f6 up
  ```
- ```
  $ sudo ip link set enp0s31f6 down
  ```

_[⬆️ Zurück zum Inhaltsverzeichnis](#inhaltsverzeichnis)_

## Sicherheit

### Zu untersuchen bei (Verdacht auf) Sicherheitsvorfall

- Systemprotokolle
  - `/var/log/auth.log`
  - `/var/log/syslog`
  - `/var/log/kern.log`
  - `/var/log/dmesg`
- Dateisystem
  - `/etc/fstab`
  - `/tmp`
  - `/home`
  - `/var`
  - `/root`
- Benutzerkonten
  - `/etc/passwd`
  - `/etc/group`
  - `/etc/sudoers`
- Software
  - `/usr/bin`
  - `/usr/lib`
  - `/etc/apt`
- Firewall
  - `/etc/iptables`

### Lynis

Mit Lynis kann man ein IT-Sicherheitsaudit durchführen.

#### Installation

```
$ sudo apt install lynis
```

#### Audit durchführen

```
$ sudo lynis audit system --auditor "[DEIN NAME]" --pentest --forensics
```

_[⬆️ Zurück zum Inhaltsverzeichnis](#inhaltsverzeichnis)_

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

_[⬆️ Zurück zum Inhaltsverzeichnis](#inhaltsverzeichnis)_

## SSL

### pfx zu pem konvertieren

```
$ openssl pkcs12 -in file.pfx -out file.pem -nodes
```

_[⬆️ Zurück zum Inhaltsverzeichnis](#inhaltsverzeichnis)_

## SWAP

### Label für Swap vergeben

```
$ sudo swaplabel -L swap /dev/sdaX
```

### Zusammenfassung des SWAPs anzeigen

```
$ sudo swapon --show
```

_[⬆️ Zurück zum Inhaltsverzeichnis](#inhaltsverzeichnis)_

## top

### Ein Signal an einen Prozess senden (KILL ist default, 9 für hartnäckige Fälle)

`< K >`

### Nur Prozesse eines user anzeigen

`< U >`

### Sortierung nach Speicherverbrauch

`< SHIFT >` + `< M >`

### Sortierung nach CPU-Last

`< SHIFT >` + `< P >`

### Aktuelle Optionsauswahl in eine Konfigurationsdatei (`~/.toprc`) schreiben

`< SHIFT >` + `< w >`

_[⬆️ Zurück zum Inhaltsverzeichnis](#inhaltsverzeichnis)_

## Ubuntu

### Upgrade der Ubuntu Version

1.  ```
     $ sudo apt update && sudo apt upgrade -y
    ```
1.  ```
    $ sudo apt install ubuntu-release-upgrader-core
    ```
1.  In der Konfigurationsdatei `normal` oder `lts` festlegen
    ```
    $ sudo vim /etc/update-manager/release-upgrades
    ```
1.  ```
    $ sudo do-release-upgrade
    ```

_[⬆️ Zurück zum Inhaltsverzeichnis](#inhaltsverzeichnis)_

## User und groups

### Zeige alle alle angemdeldeten user

```
$ who
```

oder

```
$ w
```

### Passwortstatus für ein Konto prüfen

```
$ sudo passwd -S [USER]
```

### useradd vs adduser

Der Befehl `useradd` ist eine niederigere Stufe und auf allen Linux-
Distributionen verfügbar. Er fordert zusätzliche Parameter, um das Konto
vollständig einzurichten. Der Befehl `adduser` ist eine höhere Stufe und
nicht auf allen Linux-Distributionen verfügbar. Mit diesem Befehl wird dem
System ein Benutzer mit Standarteinstellungen hinzugefügt.

### user einer group hinzufügen z. B. sudo

```
$ sudo usermod -aG sudo [USER]
```

### user ohne Login erstellen

```
$ sudo adduser [USER] --shell /usr/sbin/nologin
```

oder

```
$ sudo adduser [USER] --disable-login
```

### Login für einen user deaktivieren

```
$ sudo usermod [USER] -s /sbin/nologin
```

### Systemuser ohne home -Verzeichnis

```
$ sudo adduser --system --no-create-home [USER]
```

### home-Verzeichnis eines user ändern

```
$ sudo usermod -d /NewHome/user [USER]
```

### user löschen

```
$ sudo usermod -d /NewHome/user [USER]
```

### user mit home-Verzeichnis löschen

```
$ sudo deluser --remove-home [USER]
```

### Alle Dateien eines user löschen

```
$ sudo deluser --remove-all-files [USER]
```

### Lock Account

```
$ sudo passwd -l [USER]
```

### Unlock Account

```
$ sudo passwd -u [USER]
```

### Zeige alle User des Systems
```
$ awk -F':' '{ print $1}' /etc/passwd
```

### Zeige alle Gruppen des Systems
```
$ awk -F: '{print $1}' /etc/group
```

_[⬆️ Zurück zum Inhaltsverzeichnis](#inhaltsverzeichnis)_
