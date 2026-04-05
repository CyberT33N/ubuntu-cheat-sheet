# Cheat-Sheet: Kubuntu + NVIDIA + Wayland korrekt einrichten

## Ziel
Auf **Kubuntu** mit **NVIDIA** eine funktionierende **Plasma-Wayland-Session** aktivieren.

Wichtige Einordnung:
- Auf **Kubuntu** ist für dich **SDDM** relevant, nicht GDM.
- Deshalb ist [`/lib/udev/rules.d/61-gdm.rules`](/lib/udev/rules.d/61-gdm.rules) **für diesen Fall normalerweise nicht der richtige Hebel**.
- Im Login-Screen solltest du später **Plasma (Wayland)** wählen, **nicht** `Ubuntu on Wayland` oder `Ubuntu (Wayland)`.

---

## 1) Wayland-Paket installieren
Zuerst sicherstellen, dass die Plasma-Wayland-Session überhaupt installiert ist:

```bash
sudo apt update
sudo apt install plasma-workspace-wayland
```

Optional prüfen:

```bash
apt policy plasma-workspace-wayland plasma-workspace
```

---

## 2) GRUB-Konfiguration bearbeiten
Die relevante Datei ist:
- [`/etc/default/grub`](/etc/default/grub)

Zum Bearbeiten:

```bash
sudo gedit /etc/default/grub
```

---

## 3) In GRUB `nomodeset` entfernen und `nvidia-drm.modeset=1` setzen
In [`/etc/default/grub`](/etc/default/grub) die Zeile mit `GRUB_CMDLINE_LINUX_DEFAULT` prüfen.

### Falsch
Wenn dort `nomodeset` drinsteht, muss es raus.

Beispiel falsch:

```bash
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash nomodeset"
```

### Richtig
`nomodeset` **nicht** verwenden.

Stattdessen `nvidia-drm.modeset=1` hinzufügen.

Beispiel richtig:

```bash
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash nvidia-drm.modeset=1"
```

### Merksatz
- **`nomodeset` darf nicht drin sein**
- **`nvidia-drm.modeset=1` soll drin sein**

---

## 4) GRUB aktualisieren
Nach der Änderung an [`/etc/default/grub`](/etc/default/grub):

```bash
sudo update-grub
```

---

## 5) Reboot durchführen
Da du den Kernel-Startparameter über GRUB geändert hast, jetzt neu starten:

```bash
sudo reboot
```

Wichtig:
- **Nach GRUB-Änderungen reicht Logout nicht**
- **Hier ist Reboot erforderlich**

---

## 6) Danach im Login-Screen die richtige Session wählen
Nach dem Neustart im SDDM-Login-Screen **nicht** diese Einträge wählen:
- `Ubuntu on Wayland`
- `Ubuntu (Wayland)`

Stattdessen die **KDE-Session** wählen, also sinngemäß:
- `Plasma (Wayland)`

Wenn nur Ubuntu-Wayland-Einträge sichtbar sind und **kein** `Plasma (Wayland)`, dann ist die Plasma-Wayland-Session noch nicht korrekt vorhanden oder wird nicht korrekt angeboten.

---

## 7) Optional prüfen, ob der NVIDIA-Modeset-Parameter aktiv ist
Nach dem Reboot optional kontrollieren:

```bash
sudo cat /sys/module/nvidia_drm/parameters/modeset
```

Erwartet:

```bash
Y
```

---

## Kompaktversion für dein Cheat-Sheet

```bash
# 1) Wayland-Paket installieren
sudo apt update
sudo apt install plasma-workspace-wayland

# 2) GRUB bearbeiten
sudo gedit /etc/default/grub

# 3) In /etc/default/grub sicherstellen:
#    - KEIN nomodeset
#    - nvidia-drm.modeset=1 ist gesetzt
# Beispiel:
# GRUB_CMDLINE_LINUX_DEFAULT="quiet splash nvidia-drm.modeset=1"

# 4) GRUB aktualisieren
sudo update-grub

# 5) Neustarten
sudo reboot

# 6) Im Login-Screen Plasma (Wayland) wählen
```
