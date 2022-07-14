# multiboot-win-usb

How to Multiboot Windows2Go USB Drive (2 Identische Win2GO Systeme, z.B. in unterschiedlichen Domänen)

Vorwort:
Da der USB-Stick Eine GUID Partitions tabelle (GPT) benötigt ist er nicht im Legacy BIOS verwendbar. Es wird UEFI benötigt!
Mainboards die UEFI unterstützen sind zwar meistens abwärtskompatibel, Mainbords die BIOS verwenden sind aber nicht aufwärts kompatibel.
Was benötigt wird:
 - Leerer USB Stick mit mind. 64GB (besser 128 oder 256, damit Platz für Updates und persönliche Dateien sind) im Idealfall USB 3.0 oder besser. (alle Daten auf dem Stick werden gelöscht)
 - Windows ISO Installationsimage
 - Ein "Quell"-PC mit Windows


1. Kommandozeile mit Diskpart öffnen.
 - Windowstaste drücken und "cmd" suchen. Rechtsklick und als Administrator ausführen.
 - In der kommandozeile "diskpart" eingeben
 - In Diskpart "list disk"
 - Herausfinden welche Disk der USBstick ist
 - "sel disk x" (x= nummer des USB Sticks)
 - Mit "list part" partitionen anzeigen
 - Mit "sel part x" vorhandene partitionen auswählen
 - Mit "del part" partition löschen
- Solange wiederholen bis keine Partitionen mehr existieren.
 - "clean" eingeben um den USB Stick zu bereinigen
 - "convert gpt" eingeben um die Partitionstabelle zu erstellen
 - "create part primary size=xxxxx" um eine Primäre partition zu erstellen. Dies wird die Partition auf der das erste Windows installiert ist. Bei "size" die größe in MB angeben (z.B. size=50000 für 50gb speicherplatz)
 - "format fs=ntfs quick label=windowsA" Die partition in NTFS Formatieren. Label is die Datenträger bezeichnung. (So heißt dann auch das C-Drive des windows2go) 

 - "create part primary size=xxxxx" um eine Primäre partition zu erstellen. Dies wird die Partition auf der das zweite Windows installiert ist.
 - "format fs=ntfs quick label=windowsB" Die partition in NTFS Formatieren. Label is die Datenträger bezeichnung.

 - "create part primary size=200" Eine 200mb große partition. Dies wird später die Systempartition mit dem Bootloader
 - "format fs=FAT32 quick label=efi" Formatieren des EFI Datenträgers in FAT32
Dies ist vorerst der letzte Schritt in DISKPART. Mit "exit" kann diskpart beendet werden.



2. Windowsdateien auf die NTFS Datenträger kopieren
 - Mit Doppelklick auf die ISO des windowsinstallers die ISO als Virtuelles Laufwerk mounten.
 - Kommandozeile mit Administratorrechten öffnen (cmd->rechtsklick)
 - "dism /Get-WimInfo /WimFile:X:\sources\install.wim" X ersetzen mit dem laufwerksbuchstaben der Windows Install ISO

Es werden aller installations typen der ISO gelistet.
Die Nummer bei "Index" von der Windows version die man haben möchte Merken

 - "dism /Apply-Image /ImageFile:X:\sources\install.wim /Index:5 /ApplyDir:Y:\"
mit diesem befehl werden die Windows Daten auf das zuvor erstellte NTFS Laufwerk kopiert. X ist dabei das ISO Laufwerk, Y ist das Ziel NTFS Laufwerk des USB Sticks.
 - Dieser Vorgang kann eine Weile dauern. Nach ein paar Minuten Wartezeit erscheint ein Ladebalken 
 - Den befehl für die zweite Windows installation wiederholen. Y ist dann natürlich das zweite NTFS Laufwerk des USB-Sticks



Nachdem die Dateien Kopiert wurden muss nun der Bootloader Konfiguriert werden.
 - In CMD:
"Y:\Windows\System32\bcdboot Y:\Windows /f UEFI /s Z:"
Wobei Y Die USB NTFS partion ist und Z die USB FAT32 partition (EFI) (Achtung 2 positionen für Y)
 - Diesen Befehl für die zweite USB NTFS partition wiederholen. Z bleibt die gleiche EFI Partition des USB STICKS nur Y ändert sich entsprechend der zweiten NTFS Partition
 - Nun benötigt man noch einmal DISKPART (in cmd DISKPART eingeben)



3. In DISKPART die EFI Partition als EFI Systempartition schreiben.
 - "list disk"
 - "sel disk X" (X= USB stick)
 - "list part"
 - "sel part X" (EFI Partition 200mb größe)
 - "set id=c12a7328-f81f-11d2-ba4b-00a0c93ec93b"
Dieser befehl setzt der Partitionsart der EFI Partition auf EFI SYSTEM
(Siehe https://docs.microsoft.com/de-de/windows-server/administration/windows-commands/set-id)

4. Der USB Stick ist nun fertig. Man kann nun den Computer ausschalten und in das UEFI starten. (Mainboard webseite suchen wie man bei dem Hersteller ins SETUP kommt, Meistens irgendeine F-Taste oder ENTF)
 - Bootreihenfolge so ändern, dass von dem USB-Stick gebootet wird (UEFI! NICHT Legacy)
 - Speichern und Neu starten
Der PC sollte nun vom Stick starten und ein WINDOWS BOOTMENÜ anzeigen. Die beiden Windows-Versionen sind an der Volume-bezeichnung zu unterscheiden.
Nach dem einrichten des Windows (erster Start) kann die Bezeichnung noch geändert werden:
 - CMD mit Adminrechten ausführen
 - "bcdedit /set {current} description "Neuer Anzeigename""
Die anführungszeichen bei Neuer Anzeigename nicht vergessen
