# Rapport d'analyse de sécurité mobile

## A. Informations générales
- **Date** : 2026-05-22
- **Analyste** : Mahmoud Laasri
- **Cible** : com.example.jnidemo (JNIDemo Basics)
- **Version/Hash** : 1.0 / SHA-256 : `18759D08E3D2292B1390AF798C6D204EFAFEA4073229B48C46F840AA48345A92`
- **Outils utilisés** : BeVigil v2.1.0, Yaazhini v1.3.2, Python Custom Binary Inspector

## B. Résumé exécutif
L'analyse de sécurité statique externe (BeVigil) et interne (Yaazhini) de l'application *com.example.jnidemo* a identifié 10 constats uniques. Bien que l'application soit principalement un démonstrateur de pont d'appels natifs (JNI), elle présente un niveau de risque **Moyen à Élevé** en raison de l'activation par défaut du mode débogage (`debuggable="true"`), permettant l'injection de code dynamique, combinée à l'autorisation des sauvegardes système en clair et à l'absence de stripping/obfuscation de la bibliothèque C++ (`libnative-lib.so`).

## C. Top 5 constats

### 1. Application compilée en mode débogage actif (FIND-001)
- **Sévérité** : **High**
- **Preuve** : `AndroidManifest.xml` -> `<application android:debuggable="true">`
- **Impact** : Permet à un attaquant physique ou un malware d'attacher un débogueur local au processus de l'application via le protocole JDWP, d'analyser la mémoire vive au runtime et de détourner le flux d'exécution.
- **Remédiation** : Modifier la configuration de build dans Gradle pour forcer `debuggable false` sur le type de build release.
- **Référence OWASP** : MASVS-CODE-2

### 2. Sauvegarde système globale autorisée (FIND-002)
- **Sévérité** : **Medium**
- **Preuve** : `AndroidManifest.xml` -> `<application android:allowBackup="true">`
- **Impact** : Un attaquant ayant un accès USB de débogage peut exfiltrer l'intégralité du répertoire privé de l'application en exécutant la commande `adb backup`, compromettant ainsi la confidentialité des données stockées localement.
- **Remédiation** : Désactiver explicitement les sauvegardes dans le manifeste avec `android:allowBackup="false"`.
- **Référence OWASP** : MASVS-STORAGE-4

### 3. Permission système hautement privilégiée non justifiée (FIND-003)
- **Sévérité** : **Medium**
- **Preuve** : `AndroidManifest.xml` -> `<uses-permission android:name="android.permission.DUMP" />`
- **Impact** : L'obtention de la permission privilège `DUMP` donne à l'application l'accès aux journaux système confidentiels et à l'état interne de la mémoire du système d'exploitation, étendant inutilement la surface d'attaque en cas de compromission.
- **Remédiation** : Supprimer purement et simplement la déclaration de cette permission privilégiée dans `AndroidManifest.xml`.
- **Référence OWASP** : MASVS-PLATFORM-1

### 4. Exposition de constantes en texte clair dans le binaire natif (FIND-004)
- **Sévérité** : **Medium**
- **Preuve** : `libnative-lib.so` -> Section `.rodata` contenant `[MASQUÉ]` ("Hello from C++ via JNI !")
- **Impact** : L'exposition de chaînes brutes dans la bibliothèque native permet d'extraire facilement des clés secrètes ou des tokens par simple lecture statique à l'aide de la commande `strings`.
- **Remédiation** : Ne jamais stocker de secrets ou de données sensibles en clair dans le code natif. Mettre en place un stockage sécurisé de plateforme ou des méthodes de chiffrement White-Box.
- **Référence OWASP** : MASVS-STORAGE-1

### 5. Absence de stripping et d'obfuscation de la bibliothèque native (FIND-005)
- **Sévérité** : **Low**
- **Preuve** : `libnative-lib.so` -> Présence des symboles JNI originaux en clair (e.g. `Java_com_example_jnidemo_MainActivity_helloFromJNI`)
- **Impact** : Facilite grandement le reverse engineering du code natif par des outils comme Ghidra ou IDA Pro, et permet l'écriture rapide de scripts d'instrumentation dynamique Frida pour détourner la logique de sécurité.
- **Remédiation** : Stripper les symboles de débogage inutiles via la commande `strip` lors de la compilation native et obfusquer les fonctions internes du code C/C++.
- **Référence OWASP** : MASVS-RESILIENCE-1

## D. Faux positifs notables
Un signal a été relevé par les outils concernant l'URL `https://android.googlesource.com/toolchain/llvm-project` découverte dans les binaires `.so`. Après analyse approfondie, il s'agit d'un **Faux Positif**. Cette URL est une chaîne de métadonnées injectée de manière standard par le compilateur Clang de Google NDK pour référencer la version de la chaîne d'outils de compilation. Elle ne présente aucun risque de fuite de données de production ou de serveur backend propre à l'application.

## E. Recommandations prioritaires
1. **Désactiver le mode debug et les sauvegardes** : Configurer la version release de l'application dans Gradle pour désactiver le mode debug (`debuggable false`) et configurer `android:allowBackup="false"` dans `AndroidManifest.xml`.
2. **Retirer les privilèges système excessifs** : Supprimer la permission non nécessaire `android.permission.DUMP` du fichier de configuration du manifeste.
3. **Sécuriser la bibliothèque native** : Compiler la bibliothèque native `.so` en appliquant les optimisations de release, en strippant les symboles de débogage inutiles (`strip`) et en masquant la logique interne sensible.

## F. Annexes
- [Export Télémétrie BeVigil](file:///C:/Users/Mahmoud/Desktop/dev_mobile/sécurité/lab8/01-bevigil/bevigil_export.json)
- [Notes détaillées BeVigil](file:///C:/Users/Mahmoud/Desktop/dev_mobile/sécurité/lab8/01-bevigil/bevigil_notes.md)
- [Rapport Statique Yaazhini (Simulé)](file:///C:/Users/Mahmoud/Desktop/dev_mobile/sécurité/lab8/02-yaazhini/yaazhini_report.html)
- [Notes détaillées Yaazhini](file:///C:/Users/Mahmoud/Desktop/dev_mobile/sécurité/lab8/02-yaazhini/yaazhini_notes.md)
- [Tableau de Triage CSV](file:///C:/Users/Mahmoud/Desktop/dev_mobile/sécurité/lab8/03-triage/triage.csv)
- [Mapping OWASP MASVS](file:///C:/Users/Mahmoud/Desktop/dev_mobile/sécurité/lab8/03-triage/owasp_mapping.md)
