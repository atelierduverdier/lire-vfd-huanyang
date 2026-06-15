# Huanyang VFD Parameter Reader / Lecteur de paramètres VFD Huanyang

🇬🇧 A simple Python tool to read **all parameters** from a Huanyang (HuangYang) variable frequency drive (VFD) over RS485, using the proprietary **HYComm** protocol.

🇫🇷 Un outil Python simple pour lire **tous les paramètres** d'un variateur de fréquence (VFD) Huanyang via RS485, en utilisant le protocole propriétaire **HYComm**.

---

## 🇬🇧 English

### Why this tool?

Huanyang VFDs (the common blue/grey 1.5–2.2 kW spindle drives) do **not** use standard Modbus RTU. They use Huanyang's own serial protocol, often called **HYComm**. This means a standard Modbus library (like `minimalmodbus` or `pymodbus`) **will not communicate** with these drives — the VFD simply stays silent.

This script implements the HYComm "Function Read" frames directly, so you can dump every parameter (PD000–PD189) without manually reading them one by one on the tiny VFD display.

It was developed for a [PrintNC](https://wiki.printnc.info) CNC router running LinuxCNC on a Raspberry Pi 5, where the spindle VFD is controlled through the `hy_vfd` component.

### How it works

The HYComm "Function Read" request frame is:

```
[address][0x01][0x03][PD_number][0x00][0x00][CRC16]
```

The response frame is:

```
[address][0x01][length][PD_echo][value...][CRC16]
```

where `length` is the number of bytes after it (PD echo + value):
- `length = 0x03` → 1 byte PD echo + 2 bytes value
- `length = 0x02` → 1 byte PD echo + 1 byte value

The script validates every response with its CRC16 **and** by checking that the echoed parameter number matches the request — this filters out noise and misaligned replies.

### Requirements

```bash
pip3 install pyserial --break-system-packages
```

### Usage

Edit the configuration at the top of the script if needed:

```python
PORT    = '/dev/ttyAMA2'   # Raspberry Pi 5 = ttyAMA2, Pi 4 = ttyAMA3, USB adapter = ttyUSB0
BAUD    = 9600
ADRESSE = 0x01             # VFD Modbus/comm address (PD163)
```

Then run (with LinuxCNC **stopped**, otherwise the serial port is busy):

```bash
python3 lire_vfd.py
```

The script prints all parameters and saves them to `vfd_parametres.txt`.

### Known parameters

The script decodes and labels the common HY-series parameters (frequency, voltage, motor data, communication settings...). Unknown non-zero parameters are shown as raw values. Adjust the `INFOS` dictionary in the script to add or correct labels for your specific model.

### Important notes

- **Stop LinuxCNC first.** While LinuxCNC runs, the `hy_vfd` component holds the serial port and this script cannot open it.
- Serial settings: **9600 baud, 8 data bits, no parity, 1 stop bit** (8N1).
- This is **read-only** — the script does not modify any VFD parameter.
- Parameter numbering and scaling can vary slightly between Huanyang firmware versions. Always cross-check against your VFD's manual.

### License

MIT — free to use, modify and share.

---

## 🇫🇷 Français

### Pourquoi cet outil ?

Les VFD Huanyang (les variateurs de broche bleus/gris courants de 1,5 à 2,2 kW) n'utilisent **pas** le Modbus RTU standard. Ils utilisent le protocole série propriétaire de Huanyang, souvent appelé **HYComm**. Conséquence : une bibliothèque Modbus standard (comme `minimalmodbus` ou `pymodbus`) **ne communiquera pas** avec ces variateurs — le VFD reste muet.

Ce script implémente directement les trames HYComm de lecture, ce qui permet de récupérer tous les paramètres (PD000 à PD189) sans devoir les lire un par un sur le petit écran du VFD.

Il a été développé pour une fraiseuse [PrintNC](https://wiki.printnc.info) sous LinuxCNC sur Raspberry Pi 5, où le VFD de broche est piloté par le composant `hy_vfd`.

### Fonctionnement

La trame de requête HYComm « Function Read » est :

```
[adresse][0x01][0x03][numéro_PD][0x00][0x00][CRC16]
```

La trame de réponse est :

```
[adresse][0x01][longueur][écho_PD][valeur...][CRC16]
```

où `longueur` est le nombre d'octets qui suivent (écho du PD + valeur) :
- `longueur = 0x03` → 1 octet écho PD + 2 octets de valeur
- `longueur = 0x02` → 1 octet écho PD + 1 octet de valeur

Le script valide chaque réponse par son CRC16 **et** en vérifiant que le numéro de paramètre renvoyé correspond bien à la demande — ce qui élimine le bruit et les réponses décalées.

### Prérequis

```bash
pip3 install pyserial --break-system-packages
```

### Utilisation

Adapter la configuration en haut du script si besoin :

```python
PORT    = '/dev/ttyAMA2'   # Raspberry Pi 5 = ttyAMA2, Pi 4 = ttyAMA3, adaptateur USB = ttyUSB0
BAUD    = 9600
ADRESSE = 0x01             # adresse de communication du VFD (PD163)
```

Puis lancer (avec LinuxCNC **arrêté**, sinon le port série est occupé) :

```bash
python3 lire_vfd.py
```

Le script affiche tous les paramètres et les enregistre dans `vfd_parametres.txt`.

### Paramètres connus

Le script décode et étiquette les paramètres courants de la série HY (fréquence, tension, données moteur, réglages de communication...). Les paramètres inconnus non nuls sont affichés en valeur brute. Modifier le dictionnaire `INFOS` dans le script pour ajouter ou corriger les libellés selon ton modèle.

### Remarques importantes

- **Arrêter LinuxCNC d'abord.** Tant que LinuxCNC tourne, le composant `hy_vfd` occupe le port série et le script ne peut pas l'ouvrir.
- Réglages série : **9600 bauds, 8 bits de données, sans parité, 1 bit de stop** (8N1).
- L'outil est en **lecture seule** — il ne modifie aucun paramètre du VFD.
- La numérotation et les facteurs d'échelle des paramètres peuvent varier légèrement selon les versions de firmware Huanyang. Toujours recouper avec le manuel de ton VFD.

### Licence

MIT — libre d'utilisation, de modification et de partage.

---

*Made by [Atelier du Verdier](https://github.com/atelierduverdier) · [Instagram](https://www.instagram.com/atelierduverdier/)*
