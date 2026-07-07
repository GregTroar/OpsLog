# OpsLog

Un logiciel de log radioamateur moderne et rapide pour Windows — saisie façon
Log4OM, CAT en temps réel pour **OmniRig**, **FlexRadio/SmartSDR** natif,
**Icom CI-V** natif (USB **et** à distance par internet, en remplacement de
RS-BA1) et **TCI** (SunSDR / Expert Electronics), cluster DX avec alertes de
spots, suivi des diplômes, cartes, log de concours, gestion des QSL et un
concepteur de cartes QSL. Construit avec **Wails v2** (backend Go + frontend
React/TypeScript), **100 % Go** (sans CGO) : SQLite pour la configuration,
**MySQL partagé** optionnel pour le journal afin que plusieurs opérateurs
partagent un même log. Entièrement thémable et bilingue (anglais / français).

Développé par **F4BPO**.

---

## Compilation / développement

- **Dev :** `wails dev` (rechargement à chaud Vite ; méthodes Go accessibles sur http://localhost:34115).
- **Build :** `wails build` (utiliser la version wails v2.11 du projet — `~/go/bin/wails.exe`).
- **Régénérer les bindings Go↔TS** après modification des méthodes `App`
  exportées : `wails generate module`.
- **Release :** `.vscode/release.ps1` (Ctrl+Maj+P → *Tasks: Run Task* →
  *Release OpsLog*) — incrémente la version, pousse le source sur Gitea, compile
  l'exe et le publie sur les releases Gitea + GitHub.

---

## Journalisation

- **Bandeau de saisie façon Log4OM :** indicatif, RST émis/reçu, nom/QTH/locator,
  bande/mode, fréquence TX/RX (split), heure de début/fin, commentaire/note. Le
  **drapeau** de l'entité contactée est affiché en grand à côté des champs RST.
- **Recherche d'indicatif** (QRZ.com / HamQTH) avec photo, pré-remplissage du
  nom/QTH/locator et onglet QRZ.com.
- **Résolution DXCC hors ligne** depuis `cty.dat` (pays, zones CQ/ITU,
  continent), avec gestion des `/MM` `/AM` `/B` (balise) et des changements de
  district (`/8`, `/W6`), plus les dérogations DXpédition de ClubLog par dates.
- **QSO récents**, matrice **déjà contactés** (par créneau bande/mode),
  re-résolution en masse depuis cty/QRZ/ClubLog, envoi en masse vers les services
  QSL.
- **Constructeur de filtres QSO avancé** (champ / opérateur / valeur, ET / OU,
  préréglages enregistrés) avec **export ADIF** des lignes filtrées ou
  sélectionnées.
- **Recherche de doublons** (Outils) — regroupe les QSO par même indicatif +
  bande + mode (optionnellement même jour / minute) et permet de choisir lesquels
  supprimer.
- **Conforme ADIF 3.1.7** en import/export : dictionnaire complet des champs,
  30 colonnes promues, éditeur générique de « champs supplémentaires » et modes
  d'export standard/complet.
- **Profils :** chaque réglage est par profil ; chaque profil peut pointer son
  journal vers le fichier SQLite local ou une base **MySQL partagée**
  (multi-opérateur).

## Cartes & antenne

- **Vue principale = deux volets configurables** (par profil, Réglages → Général
  → *Vue principale*) : carte grand-cercle, carte locator (rue), la grille du
  cluster, la grille des déjà-contactés, les QSO récents, les **commandes
  FlexRadio**, la **console Icom** ou le panneau **Net control**.
- **Carte grand-cercle** avec distance & azimut trajet court/long, fonds de carte
  sélectionnables (Light / Voyager / Street / Satellite, tous sans clé et
  légendés) et le(s) **lobe(s) du faisceau d'antenne** tracés depuis l'azimut du
  rotor.
- **Compas de rotor** (azimutal équidistant, clic pour tourner) piloté par
  PstRotator.
- **Support Ultrabeam** (Normal / inversé 180° / bidirectionnel) : la direction
  rayonnée est en vert et le **boom mécanique** en gris, à la fois sur le compas
  et sur la carte, pour toujours savoir où pointe l'antenne.

## Cluster DX

- Plusieurs serveurs de cluster avec reconnexion auto, un maître pour les
  commandes.
- **Barre latérale de filtres** (recherche d'indicatif, masquer-déjà-contactés,
  grouper les doublons, bande / mode / statut / source) partagée entre l'onglet
  Cluster et le volet cluster de la vue principale, avec bascule affichage/masquage.
- **Statut** par spot (nouveau / nouvelle bande / nouveau créneau / contacté),
  clic pour accorder la radio, et une **bandmap** multi-bandes (bandes façon
  panadapter).
- Les spots **POTA** sont étiquetés avec leur référence de parc (via
  `api.pota.app`).
- **Alertes de spots** (façon Log4OM) : règles sur indicatif / pays / bande /
  mode / spotter, avec notification sonore, visuelle et e-mail (Outils →
  *Gestion des alertes*).

## Contrôle CAT

Quatre backends natifs (Réglages → CAT), chacun avec reconnexion auto et une
connexion rapide non bloquante — une radio éteinte ne fige jamais l'application :

- **OmniRig** (Rig 1/2, changement à chaud) — fonctionne avec toute radio
  supportée par OmniRig.
- **FlexRadio (SmartSDR)** via l'API TCP de la radio — fréquence / mode / split
  de la slice en temps réel, découverte UDP, et **spots panadapter** (les spots
  du cluster poussés sur l'écran Flex, clic → remplir l'indicatif).
- **Icom CI-V** — natif, via le port **USB** de la radio *ou* par internet via le
  **serveur LAN intégré** de la radio (voir *Icom à distance* ci-dessous). Ni
  RS-BA1 ni Remote Utility nécessaires.
- **TCI** (WebSocket) — SunSDR / ExpertSDR2 et tout serveur compatible TCI :
  fréquence / mode / PTT / split, plus spots panorama optionnels.

Le mode est lu depuis la radio ; le sous-mode numérique (FT4 vs FT8) est déduit
de la fréquence. Les **antennes RX/TX Flex par bande** sont configurables et
appliquées automatiquement au changement de bande.

### Onglet de commande FlexRadio (façon SmartSDR)

Affiché uniquement quand le backend CAT est une FlexRadio :

- **Émission :** puissance RF, puissance d'accord, TUNE, MOX, processeur de
  parole (NOR/DX/DX+), VOX (+ niveau + délai), moniteur (+ niveau), gain micro.
- **Réception (slice active) :** mode/seuil AGC, niveau audio, NB / NR / ANF.
- **Coupleur d'antenne (ATU) :** accord / bypass / mémoires.
- **Amplificateur :** PowerGenius XL operate/standby + défaut.
- **Mesures en direct** via le flux UDP VITA-49 : S-mètre (unités S), puissance
  directe (W), ROS, ALC, température PA, tension, plus les mesures de l'ampli.

### Onglet de commande Icom

Affiché quand le backend CAT est Icom (USB ou réseau). Une console complète façon
RS-BA1 :

- **Double afficheur VFO** (MAIN / SUB) avec la grande fréquence tabulaire, le
  badge de mode, la bande et l'offset RIT/ΔTX, et une **rangée de boutons de
  mode** (SSB / CW / RTTY / PSK / AM / FM).
- **Scope de spectre + waterfall** (panadapter) : ON/OFF, CTR/FIX, double-clic
  pour accorder, et boutons **◀ ⊙ ▶** pour centrer le scope sur la fréquence
  actuelle (±50 kHz) et le décaler à gauche/droite.
- **Mesures en direct** toujours visibles : S-mètre (clic → remplir le RST),
  puissance en watts, ROS.
- **DSP réception :** gain AF / RF, squelch, AGC, préampli, atténuateur, filtre
  (FIL1/2/3), NB, NR, ANF et — **en CW seulement** — l'**APF** (filtre de pic
  audio).
- **Passe-bande / notch :** Twin PBT (intérieur / extérieur), notch manuel +
  position.
- **Émission :** puissance RF, MOX, TUNE, **split avec offset automatique**
  (+5 kHz en SSB, +1 kHz en CW), et moniteur. **En phonie seulement** : gain
  micro, processeur de parole, VOX (+ gain + anti-VOX). Les commandes qui ne
  s'appliquent pas au mode courant sont masquées automatiquement.
- **Bandes & antenne :** boutons de bande en un clic et sélection ANT1/ANT2.
- **Clarificateurs :** RIT et ΔTX avec accord molette / ± (Ctrl+←/→ décale le
  RIT).
- Boutons **Marche / Arrêt** (manuels par choix — l'application ne réveille
  jamais la radio à la connexion).
- La **manipulation CW** peut passer par le keyer intégré de la radio (voir
  *Keyers* ci-dessous).

### Icom à distance (par internet, sans RS-BA1)

OpsLog parle directement le protocole réseau intégré de l'IC-7610 — il
**remplace à la fois l'Icom Remote Utility et RS-BA1**. Saisissez l'IP de la
radio, le nom/mot de passe Network User1 et l'adresse CI-V, et toute la console
Icom fonctionne sur le LAN/internet : login + token (renouvelé automatiquement),
tunnel CI-V, retransmission côté réception pour une liaison très stable même
avec le panadapter en flux, et Marche/Arrêt manuel. (L'audio est hors périmètre
— utilisez la radio en USB + une liaison voix comme Mumble.)

## Keyers & audio

- **Keyer CW** avec macros et macros sur touches F. Le moteur du keyer est
  sélectionnable : **WinKeyer** (K1EL WK1/2/3 sur port COM), **Icom** (le keyer
  intégré de la radio via CI-V — sans matériel supplémentaire, fonctionne aussi à
  distance) ou **TCI**.
- **Keyer vocal numérique** (DVK) : enregistrer les messages vocaux F1–F6 et les
  émettre.
- **Enregistrement audio des QSO :** capture continue en tampon glissant ; au
  *Log QSO* le contact est sauvegardé dans un WAV par QSO
  (`INDICATIF_AAAAMMJJ_HHMMSS.wav`) ; mixe RX + micro.

## Amplis & commutateurs

- Amplificateur **PowerGenius XL** (4O3A) — TCP direct : operate/standby,
  sélecteur de mode ventilateur et affichage des défauts.
- Commutateur d'antenne **Antenna Genius** (4O3A) via TCP/GSCP — un widget de
  commutation A/B ancré.

## QSL & diplômes

- **Moteur de diplômes :** définitions intégrées + personnalisées (partagées
  **globalement** entre profils) — DXCC, WAS / WAZ / WAC, WPX,
  IOTA / POTA / SOTA / WWFF, **DDFM**, contacté/confirmé/validé par bande & mode,
  règles OU et affectation manuelle de références, détection de référence en
  direct à la saisie de l'indicatif, **import de listes de références** pour les
  totaux/noms, et un **Rescan** qui relit le journal (récupère les nouvelles
  confirmations LoTW/QRZ).
- **Services QSL :** ClubLog (upload ADIF par lots), LoTW, QRZ.com, eQSL —
  upload et **téléchargement des confirmations** (qui rafraîchit automatiquement
  les stats de diplômes).
- **Concepteur de cartes QSL** (voir ci-dessous).
- **eQSL par e-mail :** clic droit sur un QSO → *Envoyer l'eQSL par e-mail* via
  le compte SMTP configuré. (Outlook/Hotmail désactivent le SMTP basic-auth —
  utilisez Gmail avec un mot de passe d'application, ou un mot de passe
  d'application Microsoft.)

## Log de concours

- **Onglet Contest :** choisissez un concours (liste ADIF `CONTEST_ID` intégrée)
  et un échange (numéro de série courant ou échange fixe). OpsLog remplit
  automatiquement `CONTEST_ID` et les numéros émis/reçus (`STX` / `SRX`), impose
  un début/fin de fenêtre, signale les doublons et tient un tableau de score en
  direct.

## Statut opérateur en direct (événements spéciaux)

Pour un indicatif d'événement spécial multi-op sur un journal MySQL partagé (ex.
**TM74TFR**) : Réglages → Général → *Publier le statut opérateur en direct*.
Chaque instance OpsLog envoie un battement de cœur de son activité (indicatif de
l'opérateur, bande, fréquence, mode) dans une table `live_status` toutes les
~15 s. Un petit rendu PHP
([`docs/livestatus/tm74-status.php`](docs/livestatus/tm74-status.php)) sur votre
propre serveur web lit cette table et produit une page/image en direct que vous
pouvez intégrer sur la bio **QRZ.com** de la station
(`<img src="…/tm74-status.php?img=1">`). OpsLog écrit seulement dans la base —
ce n'est pas un serveur web.

## Net control

- **Log de net dirigé** (Outils → Net) : un roster global (`nets.json`) plus une
  session active en mémoire — pointez les stations présentes, puis loguez tout le
  net d'un coup en utilisant la fréquence CAT.

## Apparence & langue

- **Thèmes :** quatre thèmes complets (Clair chaud, Sombre chaud, Graphite
  sombre, Contraste élevé) plus **Auto** (suit la préférence clair/sombre du
  système), sélectionnables dans Réglages → Général. Chaque panneau et chaque
  table AG-Grid suit le thème.
- **Bilingue :** interface complète **anglais / français**, avec un choix de
  drapeau au premier lancement et un sélecteur dans Réglages → Général.

## Sécurité

- **Coffre à secrets :** chiffrement optionnel par phrase de passe des mots de
  passe stockés (AES-GCM + PBKDF2). Les valeurs chiffrées sont portables ; une
  seule invite de déverrouillage au lancement les déchiffre pour la session.

## Intégrations (sortantes)

- **Émetteurs UDP :** pousser la fréquence actuelle vers **PstRotator**, les
  infos radio au format **N1MM `RadioInfo`**, ou un **enregistrement ADIF à
  chaque QSO logué** — pour que les outils externes (contrôle de rotor,
  applications numériques, autres logiciels de log) restent synchronisés.

## Divers

- **Démarrage automatique :** lancer des programmes externes (WSJT-X, JTAlert,
  contrôle de rotor…) au démarrage d'OpsLog, en sautant ceux déjà lancés.
- **Sauvegarde :** sauvegarde optionnelle base + ADIF à la fermeture.
- **Vérification de mise à jour** au démarrage avec un toast (désactivable).
- **Télémétrie d'usage anonyme** (un battement de cœur quotidien : ID
  d'installation aléatoire + version + OS — aucune donnée d'indicatif ou de QSO ;
  désactivable dans les Préférences).

---

## Concepteur de cartes QSL

Outils → *Concepteur de cartes QSL…* transforme quelques photos en une carte
eQSL soignée :

1. Choisissez 1 à 6 photos (jpeg/png). OpsLog les analyse hors ligne (grille de
   détail/luminance) et propose **3 designs** — indicatif dans la zone la plus
   calme de la meilleure photo, nom de l'opérateur, zones CQ/ITU + ligne locator,
   drapeau du pays, les autres photos en inserts encadrés, et un encart de
   confirmation par QSO.
2. Choisissez une proposition et affinez-la : cliquez un élément pour le
   sélectionner, glissez pour déplacer, changez la police / le préréglage de
   style (gel or, gel argent, contour blanc classique, script, plat) et les
   réglages par préréglage dans le panneau de droite.
3. Enregistrez le modèle (les photos sont copiées dans
   `data/qsl/templates/<id>/`, les originaux peuvent donc être déplacés). Un
   modèle peut être le défaut par profil.

Envoi : clic droit sur un QSO → *Envoyer l'eQSL par e-mail*. La carte est rendue
avec les données de ce QSO, matricée en JPEG ≤ 800 Ko, archivée dans
`data/qsl/outbox/` et envoyée via le compte SMTP configuré à l'adresse trouvée
par la recherche QRZ/HamQTH. En cas de succès le QSO est marqué `EQSL_SENT=Y`
(ADIF). Les modèles de sujet/corps de l'e-mail sont dans le concepteur (variables
`{CALL} {DATE} {BAND} {MODE} {MYCALL}`).

Polices : Archivo Black, Lilita One, Baloo 2, Oswald, Great Vibes, Allura
(toutes OFL, embarquées — licences dans `internal/qslcard/assets/fonts/`) ;
Cooper Black est proposée si MS Office l'a installée. Drapeaux : flag-icons
(MIT), embarqués pour les entités DXCC couramment contactées.

---

## Données & stockage

- La **config** (réglages, profils, radios/antennes, nœuds de cluster, cache de
  recherche, listes de diplômes, modèles QSL) réside toujours dans le fichier
  SQLite local sous `data/` — instantané même quand le journal est sur un MySQL
  lointain.
- Le **journal** (QSO) réside là où pointe le profil actif : le fichier SQLite
  local ou une base **MySQL** partagée par profil.

---

*An English version of this document is available in [README.md](README.md).*
