# Notes d'analyse Yaazhini

Ce rapport présente les indicateurs d'exposition internes identifiés dans les fichiers de bytecode DEX et dans le binaire natif (`libnative-lib.so`) après décompilation et désassemblage avec l'outil d'analyse statique Yaazhini.

## Éléments identifiés

### Élément 1: Application en mode Débogage activé (Debuggable)
- **Localisation** : `AndroidManifest.xml` (Attribut `android:debuggable="true"` au niveau de la balise `<application>`)
- **Description** : L'indicateur de débogage est activé. Cet APK est un build de débogage (`app-debug.apk`), ce qui laisse le port JDWP (Java Debug Wire Protocol) ouvert à l'exécution.
- **Impact potentiel** : Un attaquant physique ou un malware s'exécutant sur le même appareil peut attacher un débogueur (comme `jdb`) au processus de l'application. Cela permet d'exécuter du code arbitraire, d'accéder aux données en mémoire et d'altérer la logique de l'application en cours d'exécution.
- **Remédiation suggérée** : S'assurer que le flag `android:debuggable` est explicitement réglé sur `false` dans le fichier `AndroidManifest.xml` pour toutes les versions de production. Dans Gradle, configurer `debuggable false` dans le bloc `buildTypes.release`.

### Élément 2: Sauvegarde automatique des données utilisateur activée (AllowBackup)
- **Localisation** : `AndroidManifest.xml` (Attribut `android:allowBackup="true"`)
- **Description** : Le mécanisme de backup automatique d'Android est autorisé pour cette application.
- **Impact potentiel** : N'importe quel utilisateur ou attaquant ayant accès à l'appareil avec le débogage USB activé peut utiliser la commande `adb backup` pour copier l'intégralité du répertoire privé de l'application (fichiers de configuration, bases de données privées, etc.) sur son ordinateur, sans avoir besoin d'être Root.
- **Remédiation suggérée** : Désactiver explicitement les sauvegardes en modifiant l'attribut dans le manifeste : `android:allowBackup="false"`.

### Élément 3: Demande de permission système hautement privilégiée (DUMP)
- **Localisation** : `AndroidManifest.xml` (Déclaration `<uses-permission android:name="android.permission.DUMP" />`)
- **Description** : L'application demande la permission privilégiée `android.permission.DUMP`.
- **Impact potentiel** : La permission `DUMP` est une permission de niveau système (signature/system). Elle permet à l'application de lire des informations de diagnostic système sensibles de bas niveau (état de la mémoire, logs système globaux, états de services système). L'attribution de permissions aussi puissantes élargit considérablement la surface d'attaque et peut être exploitée pour collecter des données sur d'autres applications.
- **Remédiation suggérée** : Retirer la déclaration de cette permission système car elle n'est pas nécessaire pour les fonctionnalités basiques de JNI de cette application pédagogique.

### Élément 4: Exposition et fuite de la logique interne dans la bibliothèque native
- **Localisation** : `lib/arm64-v8a/libnative-lib.so` (et autres architectures, fonctions de transition JNI en clair)
- **Description** : La bibliothèque native `libnative-lib.so` expose des symboles JNI critiques et des signatures de fonctions en texte clair : `Java_com_example_jnidemo_MainActivity_helloFromJNI`, `Java_com_example_jnidemo_MainActivity_reverseString`, `Java_com_example_jnidemo_MainActivity_factorial`, et `Java_com_example_jnidemo_MainActivity_sumArray`. De plus, elle n'est pas stripée de ses métadonnées de debug.
- **Impact potentiel** : Des outils d'analyse de binaires comme Ghidra ou IDA Pro peuvent facilement décompiler cette bibliothèque pour cartographier et analyser toute la logique de calcul C/C++. L'absence d'obfuscation de ces points d'entrée JNI permet à un analyste ou à un attaquant de comprendre l'implémentation et de créer facilement des scripts Frida pour hooker ou modifier les valeurs au runtime.
- **Remédiation suggérée** : Utiliser un compilateur intégrant l'obfuscation native (par exemple, OLLVM) et appliquer la commande `strip` sur les binaires `.so` compilés pour éliminer tous les symboles de débogage inutiles avant l'empaquetage de l'APK.

### Élément 5: Constante textuelle sensible en clair dans le code natif (Secret exposable)
- **Localisation** : `libnative-lib.so` (Section `.rodata` de la bibliothèque native)
- **Description** : Une constante de message est stockée en texte clair : `[MASQUÉ]` (valeur brute : "Hello from C++ via JNI !").
- **Impact potentiel** : Les constantes de texte codées en dur dans le code natif ne sont pas sécurisées. N'importe quel outil de lecture de chaînes (comme la commande standard `strings`) peut en extraire le contenu directement. Si ce message contenait une clé secrète, un token d'accès API ou une clé cryptographique, elle serait instantanément compromise.
- **Remédiation suggérée** : Ne jamais stocker de secrets ou de tokens en clair dans le code natif ou Java. Utiliser des mécanismes de dérivation dynamiques (White-Box Cryptography), ou charger les secrets depuis un backend sécurisé au runtime après authentification forte.
