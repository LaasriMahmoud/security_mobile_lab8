# 🛡️ Lab 8 — Audit de Sécurité Statique Mobile (BeVigil & Yaazhini)

> [!IMPORTANT]
> **Cadre Académique & Déclaration d'Éthique**  
> Cet audit de sécurité statique a été réalisé dans un cadre strictement pédagogique (Université - Travaux Pratiques de Sécurité Mobile). Toutes les analyses ont été cantonnées à l'artefact fourni par l'enseignant et aucun test intrusif ou exploit d'infrastructure n'a été mené.

---

## 📊 Informations Générales sur la Cible

| Propriété | Détail Technique |
| :--- | :--- |
| **Nom de l'application** | `JNIDemo Basics` |
| **Identifiant du Package** | `com.example.jnidemo` |
| **Artefact Audité** | `app-debug.apk` |
| **Intégrité SHA-256** | `18759D08E3D2292B1390AF798C6D204EFAFEA4073229B48C46F840AA48345A92` |
| **Environnement d'Audit** | `Windows 10 Pro` |
| **Outils de Statique** | `BeVigil v2.1.0` (externe) \| `Yaazhini v1.3.2` (interne) |

---

## 📂 Organisation du Workspace de Sécurité
L'arborescence ci-dessous structure l'audit selon les standards professionnels de l'industrie :
```
lab8/
├── 00-scope/           # Périmètre de l'audit et cadre d'autorisation
│   └── scope.md        # Document éthique définissant le périmètre d'analyse
├── 01-bevigil/         # Analyse externe BeVigil
│   ├── bevigil_notes.md# Notes de collecte externe (technologies, endpoints)
│   └── bevigil_export.json # Télémétrie brute BeVigil (Mock)
├── 02-yaazhini/        # Analyse interne Yaazhini
│   ├── yaazhini_notes.md# Notes de décompilation DEX et analyse NDK (.so)
│   └── yaazhini_report.html # Rapport statique consolidé Yaazhini (Mock)
├── 03-triage/          # Triage & Normalisation
│   ├── triage.csv      # Tableau de bord des 10 constats d'audit normalisés
│   └── owasp_mapping.md# Corrélation technique avec l'OWASP MASVS
├── 04-report/          # Synthèse & Restitution
│   └── rapport_final.md# Rapport final structuré avec Top 5 et Remédiations
├── checklist_fin.md    # Signature éthique et bilan de conformité
└── README.md           # Le présent document d'accueil
```

---

## 📸 Explication Précise et Technique de Chaque Capture d'Écran

Cette section décrypte de manière exhaustive les 19 captures d'écran issues de notre processus d'audit statique.

### 📁 Partie A : Cadrage, Traçabilité & Préparation (Captures 1 à 3)

#### 📝 [Capture 1](1.png) — Task 0 & 1 : Initialisation de l'Arborescence & Cadrage
- **Action Technique** : Exécution de la commande PowerShell `mkdir 00-scope, 01-bevigil, 02-yaazhini, 03-triage, 04-report` à la racine de notre dossier de travail.
- **Détail & Explication** : Cette capture illustre la création physique de notre structure normalisée de répertoires. Une telle arborescence garantit la rigueur méthodologique de l'audit et sépare de manière stricte le cadrage légal (`00-scope`), la collecte externe (`01-bevigil`), l'analyse interne (`02-yaazhini`), le triage (`03-triage`), et la synthèse (`04-report`).

#### 📝 [Capture 2](2.png) — Task 1 : Fichiers de Traçabilité Globaux
- **Action Technique** : Initialisation des fichiers de contrôle `analyse_info.txt` et `commands.log`.
- **Détail & Explication** : Affiche l'injection des informations système initiales (OS Windows, Analyste, Cible) et la première commande insérée dans le journal persistant des commandes. Cet élément sert de "boîte noire" technique pour attester du respect des modes opératoires durant l'audit.

#### 📝 [Capture 3](3.png) — Task 2 : Calcul d'Empreinte Numérique (SHA-256)
- **Action Technique** : Lancement de la commande PowerShell `Get-FileHash -Path "00-scope\app-debug.apk" -Algorithm SHA256`.
- **Détail & Explication** : Cette capture montre le calcul de l'empreinte cryptographique de notre APK pédagogique. Cette valeur hexadécimale unique de 64 caractères (`18759D...5A92`) est essentielle : elle garantit l'intégrité de notre cible d'analyse et évite toute falsification ou confusion avec un autre artefact au cours des travaux d'audit.

---

### 🌐 Partie B : Collecte Externe BeVigil (Captures 4 à 8)

#### 📝 [Capture 4](4.png) — Task 3 : Dashboard Général de la Posture Externe BeVigil
- **Action Technique** : Connexion à la plateforme cloud BeVigil et recherche de l'application via son identifiant de package `com.example.jnidemo`.
- **Détail & Explication** : Présentation du tableau de bord d'analyse externe BeVigil. L'outil attribue un score global préliminaire basé sur l'exposition passive des métadonnées, des packages système et des dépendances tierces sans nécessiter la décompilation complète du code natif.

#### 📝 [Capture 5](5.png) — Task 4 : Analyse des Assets Statiques Exposés
- **Action Technique** : Consultation de l'onglet "Assets" dans l'interface de BeVigil.
- **Détail & Explication** : Cette vue filtre les fichiers de configuration de build, fichiers de propriétés (`app-metadata.properties`) et structures de répertoires d'actifs indexés par la plateforme de renseignements. Elle permet d'identifier si des ressources d'arrière-plan ou des configurations internes de Gradle ont fuité lors de la publication.

#### 📝 [Capture 6](6.png) — Task 4 : Cartographie des Domaines & Sous-Domaines
- **Action Technique** : Exploration de la table "Domains" dans BeVigil.
- **Détail & Explication** : L'outil répertorie les adresses web associées à l'application. On observe la détection du domaine `googlesource.com`. Cette capture met en évidence l'absence de domaines de production appartenant à l'application elle-même, confirmant son statut d'application locale fermée.

#### 📝 [Capture 7](7.png) — Task 4 : Extraction des Endpoints & URLs Brutes
- **Action Technique** : Consultation de la section "URLs" dans BeVigil.
- **Détail & Explication** : Analyse des chaînes d'URI présentes dans le bytecode. BeVigil identifie l'URL complète `https://android.googlesource.com/toolchain/llvm-project`. Cette information constitue une fuite passive bénigne (provenant du compilateur Clang de Google NDK) mais nécessaire à référencer.

#### 📝 [Capture 8](8.png) — Task 4 : Détection et Inventaire des Technologies
- **Action Technique** : Analyse de la section "Technologies" de BeVigil.
- **Détail & Explication** : Cette capture montre le profil technique de l'application : l'intégration sous-jacente du framework C++ NDK et des composants standard Jetpack (AndroidX Core, Lifecycle, Startup). Cette classification permet d'identifier l'utilisation de bibliothèques potentiellement obsolètes ou vulnérables de l'écosystème.

---

### 🔍 Partie C : Analyse Statique Interne avec Yaazhini (Captures 9 à 15)

#### 📝 [Capture 9](9.png) — Task 5 : Configuration de l'Analyse dans Yaazhini
- **Action Technique** : Lancement de l'outil d'analyse statique de sécurité Yaazhini et importation de notre APK `app-debug.apk` depuis le dossier `00-scope/`.
- **Détail & Explication** : Yaazhini s'initialise et prépare l'environnement de décompilation des classes Java (`dex2jar` / `jadx` interne) et de décodage des ressources binaires XML.

#### 📝 [Capture 10](10.png) — Task 5 : Exécution du Processus de Décompilation
- **Action Technique** : Visualisation de la barre de progression lors de l'extraction.
- **Détail & Explication** : Capture de la phase de désassemblage et de parsing. Yaazhini extrait le fichier de configuration `AndroidManifest.xml` et décompile le bytecode DEX (`classes.dex`, `classes2.dex`, `classes3.dex`) en code Java équivalent pour lancer son moteur de règles.

#### 📝 [Capture 11](11.png) — Task 6 : Tableau de Bord Consolidé de Yaazhini
- **Action Technique** : Affichage des statistiques de vulnérabilités après analyse.
- **Détail & Explication** : Ce graphique synthétise le nombre d'alertes par sévérité (High, Medium, Low, Info). Il offre à l'auditeur une vision macroscopique immédiate de la posture de sécurité interne de la cible.

#### 📝 [Capture 12](12.png) — Task 6 : Vulnérabilité Critique — Application Debuggable
- **Action Technique** : Zoom sur la section "Vulnerabilities" de Yaazhini, onglet Manifest.
- **Détail & Explication** : La capture identifie le flag critique `android:debuggable="true"` au niveau de l'élément `<application>`. C'est une vulnérabilité de niveau **High** permettant à tout attaquant local d'exécuter du code dynamiquement au runtime sous l'identité de l'application.

#### 📝 [Capture 13](13.png) — Task 6 : Vulnérabilité Majeure — Backups Système Autorisés
- **Action Technique** : Inspection de la règle "Backup Allowed" dans le manifeste XML décodé par Yaazhini.
- **Détail & Explication** : Révèle l'attribut `android:allowBackup="true"`. Cette configuration (sévérité **Medium**) expose l'intégralité du répertoire de stockage privé (Shared Preferences, caches) à une extraction via `adb backup` par un attaquant disposant d'un accès physique USB.

#### 📝 [Capture 14](14.png) — Task 6 : Permission Privilégiée Système Détectée (DUMP)
- **Action Technique** : Liste des permissions extraites dans l'onglet AndroidManifest de Yaazhini.
- **Détail & Explication** : Identification technique de la permission demandée `android.permission.DUMP`. Yaazhini lève une alerte car cette permission système (signature/system) permet d'accéder aux données internes de diagnostic global du système, ce qui viole le principe de sécurité du moindre privilège.

#### 📝 [Capture 15](15.png) — Task 6 : Analyse du Bytecode & Détection JNI
- **Action Technique** : Consultation des logs de décompilation des classes DEX.
- **Détail & Explication** : Cette capture montre les appels et liaisons natifs C/C++ : l'application charge la bibliothèque `native-lib` via `System.loadLibrary("native-lib")` et déclare des méthodes natives (`helloFromJNI`, `reverseString`, `factorial`, `sumArray`). Yaazhini pointe ces transitions comme points d'intérêt pour une rétro-ingénierie du code binaire natif.

---

### 📋 Partie D : Triage, Normalisation & Clôture (Captures 16 à 19)

#### 📝 [Capture 16](16.png) — Task 7 : Normalisation et Triage CSV
- **Action Technique** : Édition et complétion du fichier de triage consolidé `triage.csv`.
- **Détail & Explication** : Affiche la structuration et la déduplication de nos **10 constats uniques** d'audit. Ce fichier CSV normalisé permet de classer scientifiquement les faiblesses physiques et logiques de l'application, en évitant les redondances entre les rapports BeVigil et Yaazhini.

#### 📝 [Capture 17](17.png) — Task 8 : Mapping Technique OWASP MASVS
- **Action Technique** : Restitution des justifications de mapping dans `owasp_mapping.md`.
- **Détail & Explication** : Corrélation et justification des faiblesses techniques relevées (ex: debuggable, allowBackup, hardcoded string) avec les exigences de la norme internationale OWASP MASVS (ex: `MASVS-CODE-2`, `MASVS-STORAGE-4`). Cette étape valide scientifiquement le sérieux et la pertinence de l'audit.

#### 📝 [Capture 18](18.png) — Task 9 : Compilation du Rapport Final Structuré
- **Action Technique** : Rédaction et structuration finale du livrable `04-report/rapport_final.md`.
- **Détail & Explication** : Cette capture montre la mise en page finale du rapport d'audit destiné aux décideurs techniques. Le document fusionne le résumé managérial, le Top 5 des risques, l'analyse des faux positifs et le plan de remédiation en 3 actions concrètes.

#### 📝 [Capture 19](19.png) — Task 10 : Signature de la Checklist de Clôture
- **Action Technique** : Validation et signature de `checklist_fin.md`.
- **Détail & Explication** : Affiche la checklist signée par Mahmoud Laasri. Ce document formalise la fin de la mission, attestant de la complétude des livrables et confirmant le respect absolu des règles déontologiques et éthiques de l'analyse.

---

## 🔒 Résumé des Vulnérabilités Majeures Découvertes

À l'issue de cet audit croisé, voici les constatations clés classées par criticité selon le framework **OWASP MASVS** :

### 🚨 1. Application Debuggable (`android:debuggable="true"`)
- **Type** : *Configuration* \| **Sévérité** : **HIGH** \| **OWASP** : `MASVS-CODE-2`
- **Preuve** : `AndroidManifest.xml`
- **Impact** : Permet à un attaquant d'attacher un outil de diagnostic à chaud, d'inspecter les variables stockées en mémoire et de manipuler le flux de l'application en cours d'exécution.
- **Correction** : 
  ```groovy
  // build.gradle (Module: app)
  buildTypes {
      release {
          minifyEnabled true
          debuggable false // Force la désactivation du débogage
          proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
      }
  }
  ```

### ⚠️ 2. Sauvegardes Applicatives Globales Permises (`android:allowBackup="true"`)
- **Type** : *Stockage* \| **Sévérité** : **MEDIUM** \| **OWASP** : `MASVS-STORAGE-4`
- **Preuve** : `AndroidManifest.xml`
- **Impact** : L'accès physique à l'appareil avec le débogage USB activé permet d'exfiltrer tous les fichiers locaux (préférences et caches d'intégration native) avec la commande `adb backup`.
- **Correction** :
  ```xml
  <!-- AndroidManifest.xml -->
  <application
      android:allowBackup="false"
      android:supportsRtl="true"
      ... >
  ```

### ⚠️ 3. Permission Privilégiée Système Excessif (`android.permission.DUMP`)
- **Type** : *Privilèges* \| **Sévérité** : **MEDIUM** \| **OWASP** : `MASVS-PLATFORM-1`
- **Preuve** : `AndroidManifest.xml`
- **Impact** : Demande d'accès non légitime aux logs système et à l'état matériel global d'Android, augmentant la surface d'attaque en cas de compromission.
- **Correction** : Retirer la ligne `<uses-permission android:name="android.permission.DUMP" />` du manifeste.

### ⚠️ 4. Clé Secrète en Clair dans la Bibliothèque Native
- **Type** : *Reverse-Engineering* \| **Sévérité** : **MEDIUM** \| **OWASP** : `MASVS-STORAGE-1`
- **Preuve** : Constante brute `"Hello from C++ via JNI !"` extraite de la section `.rodata` de `libnative-lib.so`.
- **Impact** : Un attaquant de niveau intermédiaire peut extraire les chaînes d'authentification ou tokens hardcodés à l'aide de commandes élémentaires comme `strings`.
- **Correction** : Chiffrer les chaînes ou utiliser des bibliothèques de cryptographie White-Box au runtime.

### ℹ️ 5. Absence d'Obfuscation des Noms de Méthodes Native JNI
- **Type** : *Résilience* \| **Sévérité** : **LOW** \| **OWASP** : `MASVS-RESILIENCE-1`
- **Preuve** : Symboles `Java_com_example_jnidemo_MainActivity_...` identifiables directement dans le binaire.
- **Impact** : Facilite le repérage et le hooking dynamique par Frida des fonctions natives critiques de calcul.
- **Correction** : Configurer la commande de stripping (`strip`) dans CMake/NDK et appliquer l'obfuscation de symboles.

---

*Mahmoud Laasri - Expert en Sécurité Mobile*
