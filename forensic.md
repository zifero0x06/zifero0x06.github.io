# Menez une investigation d’incident numérique forensic

Le ***digital forensic*** couvre un domaine permettant de mener des investigations et des analyses sur des supports numériques ; il permet de comprendre une attaque informatique suite à l'analyse d'un évènement de sécurité.

## Enjeux juridiques

- Usage lors d'enquêtes judiciaires en cas d'implication de supports numériques dans une affaire criminelle (cybercriminalité...)
- Utiliser un bloqueur en écriture lors de la copie ou de l'accès au support compromis
- [Expert judiciaire](https://www.service-public.fr/particuliers/vosdroits/F2161)

On peut distinguer trois finalités à la forensic :
- analyse forensic judiciaire
- analyse forensic en réponse à un incident
- analyse forensic scientifique

L'objectif final étant de préserver l'image de la société tout en maintenant ses capacités opérationnelles.

## Méthodologie

Il s'agit de réaliser une analyse factuelle sans jugement de valeur :
1. Caractériser le contexte et collecter les informations
2. Récupérer les supports numériques à analyser (relevé d'empreinte/hash)
3. Analyser les données collectées :
    -  analyse du dump de mémoire
    -  analyse du disque dur
    -  analyse des fichiers suspects ou malveillants
4. Corrélation, déductions et reporting (indices de compromissions, recommandations)

## Choisir ses outils et préparer son laboratoire d'analyse

- Suite logicielle payante [ENCASE](https://www.opentext.com/products/encase-forensic)
- Matériel [bloqueur en écriture](https://forensics.wiki/write_blockers/)
- Framework open source [The Sleuth Kit](https://www.sleuthkit.org/) incluant la GUI Autopsy
- Analyse de mémoire vive avec [Volatility](https://volatilityfoundation.org/)
- Copie de disque dur et de RAM avec [FTK Imager](https://www.exterro.com/digital-forensics-software/ftk-imager)

Pour préparer son environnement d'analyse, l'utilisation de machines virtuelles est recommandé en montant une VM Windows et une [VM Linux](https://digital-forensics.sans.org/community/downloads) avec leurs outils respectifs.

### Mise en garde

Certaines techniques permettent de *se soustraire aux analyses forensic* en dissimulant les données (stéganographie, falsification de métadonnées, modification d'attributs de fichiers...) voire en les protégeant par chiffrement pour ralentir ou empêcher le travail d'investigation numérique.

Il convient donc autant que possible de **réaliser des recoupements et de croiser les sources** pour étayer, vérifier puis contrôler les renseignements collectés avant de tirer des conclusions.

## Collecte des données

Un objectif : ne pas altérer les données collectées puis stockées sur un support numérique.
Outre les images brutes (dîtes RAW) certains formats "métiers" ont émergé :
- EnCase EWF (Expert Witness Format)
- FTK Smart
- AFF (Advanced Forensic Format) comme formart ouvert

Après utilisation d'un logiciel de *dump* il faudra calculer le l'empreinte ou le *hash* de ce dernier. Ce calcul de condensat (ou hachage) s'établira comme une preuve que le fichier n'a pas été altéré par rapport à ce qu'il était à l'origine.

Ce hash pourra être obtenu par différentes fonctions (MD5, SHA1, SHA512, etc.) et permettra de garantir l'intégrité d'un fichier et, potentiellement, de servir d'*IOC* (*Indicator Of Compromission* ou Indicateur de Compromission).

## Analyse de la mémoire volatile

Un outil comme Volatility permet de déterminer le système d'exploitation dont la mémoire vive a été *dumpée* ainsi que de lister les processus qui tournaient sur cette machine au moment de l'incident. En outre, il permet de lister les connections réseaux ouvertes associées à des processus.

Dans le cas d'une machine Windows, il est par exemple possible de :
1. récupérer le profil de l'image
    `volatility -f memdump.mem imageinfo`
2. récupérer la liste des processus
    `volatility -f memdump.mem --profile=Win7SP2x64 pstree`
3. lister les librairies (DLL) attenantes à un processus (PID)
    `volatility -f memdump.mem --profile=Win7SP2x64 dllist -p 1496`
4. obtenir des informations liées au registres
    `volatility -f memdump.mem --profile=Win7SP2x64 hivelist`
5. collecter des *hashes* NTLM de mots de passe
    `volatility -f memdump.mem --profile=Win7SP2x64 hashdump -y 0x8offset -s 0x8offset`
6. lister les connexions réseaux actives
    `volatility -f memdump.mem --profile=Win7SP2x64 netscan`

Enfin, différents fichiers de mémoire peuvent être exploités :
- celui d'enregistrement de mise en hibernation de la machine : hiberfil.sys
- le fichier de gestion des extensions de mémoire RAM (pagination) : pagefile.sys

### Indices à rechercher

A ce niveau, les indices à rechercher peuvent se présenter sous plusieurs formes :
- des processus avec des noms aléatoires ont été identifiés
- l'emplacement d'un de ces processus n'est pas courant et nécessite vérificatin- d
- des connexions réseau sont effectués par ces proce suspects

D'autres commandes Volatility permettent d'aller davantage en profondeur. En voici quelques unes :
- `psxview` pour afficher des processus cachés
- `malfind` pour tenter de déceler du code malveillant
- `mutantscan` pour lister les *mutual exclusion* (mutext)
- `memdump` et  `procmemdump` pour dumper/extraire un processus précis

## Analyse de l'image de la mémoire non-volatile

L'objet de cette partie de l'analyse est avant tout de connaitre l'architecture du système de fichiers de la machine concernée.
Par exemple, Windows utilise le système NTFS (*New Technology File System*) qui permet en autre :
- de gérer les droits des utilisateurs sur les fichiers
- de dissimuler des données avec l'utilisation de l'ADS (*Alternate Data Stream*)
- de structurer et d'enregistrer les données avec la MFT (*Master File Table*) qui est une base de données relationnelle

Dans un *dump* de disque dur (ou de SSD) réalisé avec FTK Imager, la MFT est extrayable depuis la racine de l'arborescence. Chaque entrée MFT est composée d'un en-tête et de plusieurs attributs.

L'export de la MFT est convertible en *.csv* grâce à l'outil [Mft2Csv](https://github.com/jschicht/Mft2Csv).

Après un travail de mise en forme, il devient alors possible de de bâtir une chronologie des évènements survenus sur la machine et d'en retirer des informations complémentaires :
- tri par date de création
- localisation de fichiers sur la machine
- découverte de l'origine d'exécutables
- utilisation de dossiers temporaire (`/Temporary Internet Files` ou `/tmp`)

## Investigation sous Windows

### Analyse des logs

Le système d'exploitation Windows enregistre énormément d'informations - souvent sous forme de *log* - qui sont accessibles facilement et peuvent être énumérées. Par ici, un [poster aide mémoire](https://cyber-ssct.com/SANS%20Digital%20Forensic%20Poster.pdf) du SANS.
Sous Windows, chaque évènement est répertoriée dans les *event logs* comme les messages d'erreur, d'information et de warning. L'outil Event Viewer permet de visualiser ces *logs*.

En utilisant FTK Imager, le répertoire `%SystemRoot%\System32\Winevt\Logs`contient les logs de Windows 7.

### Analyse des services

L'interface native Windows `services.msc` permet d'accéder à la liste des services qui s'exécutent en arrière plan du système d'exploitation.
Certains *malware* ou utilitaires intrusifs et malveillants peuvent s'y dissimuler et utiliser la discrétion des services Windows pour s'exécuter en silence.

### Analyse des prefetch

Au premier lancement d'un programme, Windows crée un fichier *prefetch* avec l'extension *.pf* pour permettre d'accélérer le chargement des applications. On peut retrouver l'ensemble de ces *prefetch* dans le dossier `C\Windows\Prefetch` et y rechercher des indices d'exécution de logiciels ou programmes indésirables.

### Analyse du registre

Le registre Windows est une base de données hiérarchique qui stocke les paramètres de bas niveau pour le système d'exploitation Microsoft Windows et pour les applications qui choisissent d'utiliser le registre.

Les ruches du registre sont stockées dans le répertoire `%SystemRoot%\System32\config` et correspondent à un groupe logique de clés, sous-clés et valeurs.

Quelques exemples :
- La ruche "HKEY_CURRENT_CONFIG" est associée aux fichiers "System, System.alt, System.log, System.sav"
- La ruche "HKEY_CURRENT_USER" est associées aux fichiers "Ntuser.dat, Ntuser.dat.log"
- La ruche "HKEY_LOCAL_MACHINE\SAM" est associée aux fichiers "Sam, Sam.log, Sam.sav"

Le *dump* des fichiers de registre est réalisable gràce à FTK Imager pour être stockés dans un répertoire pour analyse avec l'outil [RegRipper](https://github.com/keydet89/RegRipper4.0).

### Analyse avec Autopsy

Les étapes à suivre pour l'analyse de disque avec la suite offerte par The Sleuth Kit et sa surcouche graphique appelée Autopsy sont les suivantes :
1. créer une enquête et nommer son *case*
2. ajouter les images à analyser et préciser les modules à utiliser
3. lancer l'analyse automatisée pour *parser* les données
4. identifier les exécutables et fichiers suspects en navguant dans l'arborescence
5. rechercher les fichiers récemment consultés au moment de l'incident
6. produire une *timeline* automatique par nombre d'évènements générés en filtrant les dates
7. générer un rapport au format html

### Pour aller plus loin

De nombreux outils complémentaires existent pour conduire davantange d'investigation :
- [VirusTotal](https://virustotal.com/) pour vérifier que le fichier n'est pas déjà identifié par malveillant
- la commande `file` sous Linux pour déterminer le type du fichier
- la commande `strings` sous Linux pour en extraire les chaines de caractère
- l'analyse d'email et des fichiers archives de mail en *.pst* ou *.ost* est possible avec [Free OST Viewer Tool](https://datahelp.in/ost/viewer.html)
- l'analyse d'un fichier *.pdf* peut se faire avec un script Python appelé [pdfid.py](https://blog.didierstevens.com/programs/pdf-tools/) 
- des utilitaires existent également pour décortiquer les fichiers Microsoft Office et LibreOffice

## Rapport d'investigation

Il s'agit dès lors de communiquer les Indicateurs de Compromission (IoC) pour caractériser l'incident.
Ce rapport permettra de capitaliser sur les analyses effectuées et, ultérieurement, d'identifier plus rapidement les menaces similaires.

Il convient de lister :
- les IoC réseaux faisant référence à une activité réseau malveillante (connexions, noms de domaines, URL...)
- les hash de fichiers compromettants
- les adresses mails et autres éléments d'identification de l'origine de l'incident
- l'impact de l'incident sur la machine (création de service, modification de clé de registre...)

Certains outils permettent de mettre en forme ces IoC et de décrire le fonctionnement des attaques :
- [IoC-Editor](https://www.fireeye.com/services/freeware/ioc-editor.html) stocke et gère les indicateurs de compromission
- le format [STIX](https://stixproject.github.io/) permet aussi de réaliser des fichiers d'IoC
- la matrice [ATT&CK](https://attack.mitre.org/) permet de décrire des modes opératoires d'attaques

Voici ci-dessous un exemple de plan pour structurer le rapport de *forensic* :
1. Page de garde
2. Sommaire
3. Résumé des investigations, rappel du contexte et des conclusions
4. Détail des investigations :
    - phase de collecte et hash (copie d’écran, photos) ;
    - analyse réalisée et mise en exergue des découvertes (copie d’écran, photos)
5. Hypothèse de l’analyse
6. Recommandations
7. Liste des indicateurs de compromission