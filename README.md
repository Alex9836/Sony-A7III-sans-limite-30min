# Sony A7 III : supprimer la limite d'enregistrement vidéo de 30 minutes (Windows)

Tutoriel pas à pas pour retirer la coupure automatique à **29 min 50 s** en vidéo sur le **Sony A7 III (ILCE-7M3)**, avec l'outil open source [Sony-PMCA-RE](https://github.com/ma1co/Sony-PMCA-RE) de ma1co.

✅ **Testé et fonctionnel** sur un A7 III européen, **firmware 4.04** (dernière version à ce jour), sous **Windows 11**, en novembre 2026.
Résultat : la limite passe de `0h 29m 50s` à `13h 01m 00s`. En pratique, seules la batterie, la carte ou la chaleur arrêtent l'enregistrement.

---

## ⚠️ Avertissement

Ce tutoriel est fourni **« tel quel », sans aucune garantie**. C'est une modification **non officielle**, non prise en charge par Sony, qui peut **annuler la garantie**. L'outil utilisé est présenté par ses auteurs comme expérimental.

**Vous faites cette manipulation à vos risques et périls.** L'auteur de ce tutoriel décline toute responsabilité en cas de dommage ou de problème (appareil, ordinateur, données, garantie…).

> 🔋 Le principal risque technique : **couper l'alimentation ou débrancher le câble pendant que l'appareil est en mode service** (étapes 6 à 8). Batterie chargée à 100 %, et on ne touche à rien pendant ces étapes.

La modification est **réversible** : on peut remettre la limite de la même façon (voir [Revenir en arrière](#-revenir-en-arrière)).

---

## Sommaire

1. [Pourquoi cette limite existe](#pourquoi-cette-limite-existe)
2. [Ce qu'il vous faut](#-ce-quil-vous-faut)
3. [Étape 1 – Installer Python](#étape-1--installer-python)
4. [Étape 2 – Télécharger Sony-PMCA-RE (version récente)](#étape-2--télécharger-sony-pmca-re-version-récente)
5. [Étape 3 – Installer les dépendances](#étape-3--installer-les-dépendances)
6. [Étape 4 – Préparer l'appareil](#étape-4--préparer-lappareil)
7. [Étape 5 – Premier pilote avec Zadig (mode normal)](#étape-5--premier-pilote-avec-zadig-mode-normal)
8. [Étape 6 – Passer l'appareil en mode service](#étape-6--passer-lappareil-en-mode-service)
9. [Étape 7 – Deuxième pilote avec Zadig (mode service)](#étape-7--deuxième-pilote-avec-zadig-mode-service)
10. [Étape 8 – Supprimer la limite](#étape-8--supprimer-la-limite)
11. [Étape 9 – Remettre les pilotes Windows d'origine](#étape-9--remettre-les-pilotes-windows-dorigine)
12. [Étape 10 – Tester](#étape-10--tester)
13. [Dépannage](#-dépannage)
14. [Revenir en arrière](#-revenir-en-arrière)
15. [Conseils après la modification](#-conseils-après-la-modification)
16. [FAQ](#-faq)

---

## Pourquoi cette limite existe

Sony a imposé cette limite pour que ses appareils photo ne soient pas classés comme « caméscopes » dans l'Union européenne, une catégorie plus taxée. La règle a disparu en 2019, et les modèles plus récents (A7 IV, A6400…) n'ont plus de limite, mais **Sony n'a jamais retiré la limite de l'A7 III par une mise à jour**.

L'ancien hack (applis PlayMemories) ne marche pas sur l'A7 III. Sony-PMCA-RE passe par le **mode service** de l'appareil pour modifier ce réglage interne directement.

---

## 🧰 Ce qu'il vous faut

| Élément | Détail |
|---|---|
| Appareil | Sony A7 III (ILCE-7M3) |
| Batterie | **Chargée à 100 %** |
| Câble | Câble USB d'origine ou de bonne qualité, branché **directement sur le PC** (pas de hub) |
| PC | Windows 10 ou 11 |
| Logiciels | Python 3, Sony-PMCA-RE (code source récent), Zadig |
| Temps | 20 à 30 minutes |

Pour connaître votre version de firmware : **Menu > Réglages (mallette) > page 7 > Version**.

### 📥 Les fichiers exacts à télécharger

| Fichier | Lien direct | Source officielle |
|---|---|---|
| **Python 3.14.10** (Windows 64 bits) | [python-3.14.10-amd64.exe](https://www.python.org/ftp/python/3.14.10/python-3.14.10-amd64.exe) | [python.org](https://www.python.org/downloads/windows/) |
| **Sony-PMCA-RE** (code source, branche master) | [Sony-PMCA-RE-master.zip](https://github.com/ma1co/Sony-PMCA-RE/archive/refs/heads/master.zip) | [github.com/ma1co/Sony-PMCA-RE](https://github.com/ma1co/Sony-PMCA-RE) |
| **Zadig 2.8** (recommandé) | [zadig-2.8.exe](https://github.com/pbatard/libwdi/releases/download/v1.5.0/zadig-2.8.exe) | [libwdi v1.5.0](https://github.com/pbatard/libwdi/releases/tag/v1.5.0) |
| **Zadig 2.7** (alternative) | [zadig-2.7.exe](https://github.com/pbatard/libwdi/releases/download/v1.4.1/zadig-2.7.exe) | [libwdi v1.4.1](https://github.com/pbatard/libwdi/releases/tag/v1.4.1) |

> Zadig 2.7 ou 2.8 sont recommandés plutôt que la dernière version (2.9, sur [zadig.akeo.ie](https://zadig.akeo.ie)) : plusieurs utilisateurs signalent que les versions récentes provoquent le message `No tweaks available`. Python 3.11 ([python-3.11.9-amd64.exe](https://www.python.org/ftp/python/3.11.9/python-3.11.9-amd64.exe)) fonctionne aussi.

> 💡 **Important :** n'utilisez **pas** l'exécutable `pmca-console-v0.18-win.exe` de la page Releases. Cette version (avril 2022) est trop ancienne : en mode service, la commande `tweak` répond `Error: Unknown command`. Il faut le **code source récent**, lancé avec Python (étapes 1 à 3).

---

## Étape 1 – Installer Python

1. Téléchargez **[python-3.14.10-amd64.exe](https://www.python.org/ftp/python/3.14.10/python-3.14.10-amd64.exe)** (version testée).
2. Lancez l'installateur.
3. ⚠️ **Cochez « Add python.exe to PATH »** en bas de la première fenêtre.
4. Cliquez sur **Install Now**.
5. À la fin, si le bouton **« Disable path length limit »** apparaît, cliquez dessus, puis sur **Close**.

---

## Étape 2 – Télécharger Sony-PMCA-RE (version récente)

1. Téléchargez le code source directement depuis la branche principale :
   **https://github.com/ma1co/Sony-PMCA-RE/archive/refs/heads/master.zip**
2. Créez un dossier **`C:\pmca`**.
3. Faites un clic droit sur le zip > **Extraire tout…** > destination **`C:\pmca`**.
4. Vérifiez que vous avez un dossier **`C:\pmca\Sony-PMCA-RE-master`** qui contient **directement** `pmca-console.py` et `requirements.txt`.
   S'il y a un dossier en double (`Sony-PMCA-RE-master\Sony-PMCA-RE-master`), utilisez le second.

---

## Étape 3 – Installer les dépendances

1. Ouvrez le dossier `C:\pmca\Sony-PMCA-RE-master` dans l'Explorateur.
2. Cliquez dans la **barre d'adresse**, tapez `cmd` et appuyez sur **Entrée**. Une fenêtre noire (invite de commande) s'ouvre dans ce dossier.
3. Tapez :
   ```
   pip install -r requirements.txt
   ```
4. Attendez la fin (1 à 2 minutes). La dernière ligne doit commencer par **`Successfully installed`**.
   Le message `[notice] A new release of pip is available` peut être ignoré.

**Gardez cette fenêtre ouverte**, on s'en resservira.

---

## Étape 4 – Préparer l'appareil

1. Sur l'A7 III : **Menu > Réglages (mallette) > Connexion USB > Stockage de masse**.
2. Allumez l'appareil.
3. Branchez-le au PC avec le câble USB.

---

## Étape 5 – Premier pilote avec Zadig (mode normal)

Zadig remplace temporairement le pilote USB Windows de l'appareil par un pilote **libusb**, nécessaire pour que l'outil puisse dialoguer avec lui. **Ça ne modifie rien dans l'appareil**, uniquement sur le PC.

1. Téléchargez **[zadig-2.8.exe](https://github.com/pbatard/libwdi/releases/download/v1.5.0/zadig-2.8.exe)** (ou [zadig-2.7.exe](https://github.com/pbatard/libwdi/releases/download/v1.4.1/zadig-2.7.exe)).
   C'est un `.exe` qui se lance directement, sans installation. Dans Zadig, le pilote libusb-win32 proposé doit être en version **1.2.6.0**.
2. Lancez Zadig et acceptez la demande d'autorisation Windows.
3. Cliquez sur **Options > List All Devices**.
4. Dans la liste déroulante, sélectionnez **ILCE-7M3**.
   Vérifiez :
   - **Driver** (à gauche) : `USBSTOR` (pilote Windows actuel)
   - **USB ID** : `054C` `0C02`
5. À droite de la flèche, choisissez **libusb-win32**.
6. Cliquez sur **Replace Driver**.
7. Un avertissement « You are about to modify a system driver » apparaît : c'est normal, cliquez sur **Oui**. Seul l'appareil photo est concerné, vos clés USB et disques externes ne sont pas touchés.
8. Attendez le message de réussite.

---

## Étape 6 – Passer l'appareil en mode service

Dans la fenêtre noire (dossier `C:\pmca\Sony-PMCA-RE-master`), tapez :

```
python pmca-console.py serviceshell
```

Des avertissements jaunes `SyntaxWarning: ... invalid escape sequence` peuvent s'afficher au début : ils sont **sans importance** (code ancien lancé avec un Python récent).

Vous devez obtenir :

```
Sony DSC is a camera in mass storage mode

Switching to service mode

Waiting for camera to switch...
Found a camera in service mode
Error: Only libusb drivers are supported for service mode
```

✅ **C'est normal.** L'appareil est bien passé en mode service ; il manque juste un pilote pour ce nouveau mode.

🚫 **À partir d'ici, ne débranchez pas et n'éteignez pas l'appareil.**

---

## Étape 7 – Deuxième pilote avec Zadig (mode service)

1. Revenez dans Zadig (**List All Devices** toujours coché).
2. Dans la liste déroulante, sélectionnez le **nouveau** périphérique :
   - Nom : **Sony USB Device**
   - **USB ID** : `054C` `0336`
   - **Driver** : `(NONE)`
3. Choisissez **libusb-win32**.
4. Cliquez sur **Install Driver** (le bouton s'appelle ainsi car il n'y a pas encore de pilote).
5. Attendez le message de réussite.

---

## Étape 8 – Supprimer la limite

1. Dans la fenêtre noire, relancez :
   ```
   python pmca-console.py serviceshell
   ```
2. Vous devez arriver sur :
   ```
   Found a camera in service mode
   Authenticating
   Starting service shell...

   Welcome to platform shell.
   Type `help` for the list of supported commands.
   Type `exit` to quit.
   >
   ```
   > Si vous voyez **`Welcome to USB service shell`** au lieu de `platform shell`, vous utilisez l'ancienne version de l'outil : reprenez à l'étape 2.

3. À l'invite `>`, tapez :
   ```
   tweak
   ```
4. Le menu s'affiche :
   ```
   1: [ ] Disable video recording limit
          0h 29m 50s

   2: [ ] Unlock all languages
          18 / 35 languages activated

   3: [X] Enable PAL / NTSC selector & warning
          Enabled

   4: [X] Enable USB app installer
          Enabled

   Enter number of tweak to toggle (0 to apply):
   ```
   > Si vous obtenez `No tweaks available`, voir [Dépannage](#no-tweaks-available).

5. Tapez **`1`** puis **Entrée**. Le menu se réaffiche avec :
   ```
   1: [X] Disable video recording limit
          13h 01m 00s
   ```
   Vérifiez que **seule la ligne 1 a changé**. Ne touchez pas aux lignes 3 et 4.
6. Tapez **`0`** puis **Entrée** pour **appliquer**. ⚠️ Sans cette étape, rien n'est enregistré.
7. Quand l'invite `>` revient, tapez **`exit`** puis **Entrée**, et attendez **`Done`**.
8. Débranchez le câble, éteignez l'appareil et rallumez-le.

🎉 La limite est supprimée.

---

## Étape 9 – Remettre les pilotes Windows d'origine

Tant que le pilote libusb est en place, l'appareil affiche **« Vérifier le périphérique connecté »** quand vous le branchez, et le transfert de fichiers ne fonctionne pas. Il faut remettre le pilote Windows :

1. Rebranchez l'appareil allumé.
2. Clic droit sur le bouton **Démarrer** > **Gestionnaire de périphériques**.
3. Dépliez la catégorie **libusb-win32 devices**.
4. Clic droit sur **ILCE-7M3** > **Désinstaller l'appareil**.
5. Cochez **« Tenter de supprimer le pilote de ce périphérique »** si l'option apparaît, puis **Désinstaller**.
6. Faites de même pour **Sony USB Device** s'il apparaît.
7. Débranchez, attendez quelques secondes, rebranchez : Windows réinstalle son pilote habituel. La catégorie `libusb-win32 devices` doit avoir disparu.

---

## Étape 10 – Tester

1. Batterie pleine, au moins **25 Go libres** sur la carte (en 4K 100M).
2. Lancez un enregistrement et laissez-le tourner **au-delà de 30 minutes**.
3. Si le compteur continue après 29:50, c'est gagné ✅

---

## 🛠 Dépannage

### `Error: Unknown command` après `tweak`
Vous utilisez l'exécutable **v0.18** de la page Releases (le shell affiche `Welcome to USB service shell`). Tapez `exit`, puis suivez les étapes 1 à 3 pour utiliser le code source récent avec Python. Les pilotes Zadig déjà installés restent valables.

### `No tweaks available`
L'outil n'arrive pas à lire les réglages internes de l'appareil. Solutions, dans l'ordre :
1. Tapez `exit`, **éteignez puis rallumez l'appareil** (toujours branché), et relancez `python pmca-console.py serviceshell` puis `tweak`. Ça a suffi dans notre cas au deuxième essai.
2. Si ça persiste et que vous avez utilisé Zadig 2.9, réinstallez les pilotes avec **[zadig-2.8.exe](https://github.com/pbatard/libwdi/releases/download/v1.5.0/zadig-2.8.exe)** ou **[zadig-2.7.exe](https://github.com/pbatard/libwdi/releases/download/v1.4.1/zadig-2.7.exe)**. Plusieurs utilisateurs signalent que les versions récentes de Zadig (pilote libusb-win32 1.4.x) provoquent ce message, contrairement à la version 1.2.6 installée par Zadig 2.7/2.8.
   Refaites l'étape 5 (ILCE-7M3) et l'étape 7 (Sony USB Device) avec **Replace Driver**.

### `Error: Only libusb drivers are supported for service mode`
- Au **premier** lancement : normal, passez à l'étape 7.
- Au **deuxième** lancement : le pilote du mode service n'est pas installé. Vérifiez dans Zadig que vous avez bien sélectionné **Sony USB Device (054C 0336)** et choisi **libusb-win32** (pas WinUSB).

### L'appareil n'est pas détecté
- Vérifiez que l'appareil est en **Connexion USB : Stockage de masse**.
- Essayez un autre port USB (directement sur la carte mère, pas un hub ou un port en façade).
- Éteignez/rallumez l'appareil et relancez la commande.

### `'python' n'est pas reconnu...`
La case **« Add python.exe to PATH »** n'a pas été cochée. Relancez l'installateur Python > **Modify** ou réinstallez en cochant la case, puis rouvrez une nouvelle fenêtre `cmd`.

### Alerte bleue Windows SmartScreen au lancement de Zadig
Cliquez sur **Informations complémentaires** > **Exécuter quand même**.

---

## ↩️ Revenir en arrière

Pour **remettre la limite de 30 minutes** :
1. Refaites les étapes 4 à 8, mais à l'étape 8, tapez `1` pour **décocher** la ligne (`[ ]`), puis `0` pour appliquer.
2. Refaites l'étape 9 pour remettre les pilotes Windows.

Pour **désinstaller les outils** : supprimez le dossier `C:\pmca`, et désinstallez Python depuis *Paramètres > Applications* si vous n'en avez plus besoin. (Gardez-les si vous pensez devoir refaire la manip un jour.)

---

## 💡 Conseils après la modification

### Température : régler « Tmp HORS tens. aut. » sur « Élevé »
Sans la limite des 30 minutes, c'est la chaleur qui risque de couper l'enregistrement.
**Menu > Réglages (mallette) > page 2 (Réglage2) > Tmp HORS tens. aut. > Élevé**

- C'est un réglage **officiel Sony**. L'appareil garde sa protection contre la surchauffe : s'il devient vraiment trop chaud, il coupe et s'éteint tout seul, sans dommage.
- Avec « Élevé », le boîtier peut devenir **chaud au toucher** : préférez un trépied pour les longues prises.
- Ouvrez l'écran et **décollez-le du boîtier**, évitez le plein soleil.
- Une **batterie factice sur secteur** (type NP-FZ100) évite la chauffe de la batterie et permet des prises de plusieurs heures.

### Espace sur la carte
Le 4K à 100 Mbit/s occupe environ **45 Go par heure**. Utilisez une carte **V30 (U3) minimum** dans le **slot 1** (le seul UHS-II).

### PAL ou NTSC ?
La modification fonctionne dans les deux modes, et changer de mode **ne remet pas la limite**.
- **PAL** (Europe) : 4K 25p, 1080p 50p / 100p. Pas de scintillement sous l'éclairage à 50 Hz.
- **NTSC** : 4K 24p / 30p, 1080p 60p / 120p. Message « Exécution en NTSC » à chaque démarrage (normal). Le 4K 30p est recadré d'environ 1,2×.

---

## ❓ FAQ

**Est-ce que ça marche avec le firmware 4.04 ?**
Oui, testé et confirmé (novembre 2026). D'autres utilisateurs l'ont aussi confirmé dans [l'issue #724](https://github.com/ma1co/Sony-PMCA-RE/issues/724).

**Une mise à jour du firmware Sony va-t-elle remettre la limite ?**
C'est possible : une mise à jour réécrit le système et peut réinitialiser ce réglage, voire bloquer la méthode. **Évitez de mettre à jour** sauf besoin réel (les dernières versions n'apportent que de la « stabilité »). Si vous mettez à jour, vérifiez d'abord dans les issues de Sony-PMCA-RE que la nouvelle version est compatible, puis refaites la manip.

**Formater la carte ou changer de mode PAL/NTSC efface-t-il la modification ?**
Non. Le réglage est stocké dans la mémoire interne de l'appareil, pas sur la carte. En revanche, une **réinitialisation complète** de l'appareil (menu *Réinitialiser/Initialiser*) pourrait l'effacer.

**Est-ce que ça change la qualité d'image, l'autofocus ou autre chose ?**
Non. Seule la durée maximale d'enregistrement est modifiée.

**Et sur Mac ou Linux ?**
L'outil fonctionne aussi, sans Zadig (pas besoin de pilote). Sous Linux : `pip install -r requirements.txt`, puis `sudo python3 pmca-console.py serviceshell` et `tweak`. Ce tutoriel détaille uniquement Windows.

**Et sur les autres boîtiers Sony ?**
Sony-PMCA-RE prend en charge d'autres modèles (A7R III, A6400…), mais ce tutoriel n'a été testé que sur l'A7 III. Les modèles récents à firmware signé (A7 IV, A7S III, A1…) ne sont **pas** compatibles avec cette méthode, et la plupart n'ont de toute façon plus de limite.

---

## Crédits

- **[ma1co](https://github.com/ma1co)** pour [Sony-PMCA-RE](https://github.com/ma1co/Sony-PMCA-RE) : tout le travail de rétro-ingénierie vient de lui.
- **[Zadig](https://zadig.akeo.ie)** par Pete Batard (libwdi).

Ce tutoriel ne redistribue aucun fichier : téléchargez toujours les outils depuis leurs sources officielles ci-dessus.

Ce projet n'est affilié ni à Sony, ni aux auteurs de Sony-PMCA-RE ou de Zadig. « Sony » et « Alpha » sont des marques de Sony Group Corporation, citées uniquement pour identifier l'appareil concerné.

---

> ⚠️ Tutoriel fourni « tel quel », sans garantie. À vos risques et périls.

---

*Si ce tuto vous a aidé, laissez une ⭐. Une erreur ou un retour d'expérience sur un autre firmware ? Ouvrez une issue.*
