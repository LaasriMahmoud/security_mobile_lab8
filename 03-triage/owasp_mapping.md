# Mapping OWASP MASVS

Ce document justifie et met en corrélation les constatations clés de sécurité de notre audit avec les exigences du standard de sécurité de référence de l'industrie : **OWASP MASVS** (Mobile Application Security Verification Standard).

---

## FIND-001 : Application en mode Débogage activé (Debuggable)
- **Catégorie OWASP** : **MASVS-CODE** (Qualité et configuration du code)
- **Référence spécifique** : **MASVS-CODE-2** ("L'application est configurée pour le déploiement en production sans que le mode débogage soit activé.")
- **Justification** : L'activation de `android:debuggable="true"` permet aux attaquants locaux d'attacher un débogueur JDWP au processus en cours d'exécution. Cela enfreint directement l'exigence de non-déboguabilité en production, rendant possible l'extraction de secrets en mémoire et la falsification de la logique.

---

## FIND-002 : Sauvegarde de l'application activée (AllowBackup)
- **Catégorie OWASP** : **MASVS-STORAGE** (Stockage des données locales)
- **Référence spécifique** : **MASVS-STORAGE-4** ("L'application empêche la sauvegarde de données sensibles en dehors du conteneur de l'application.")
- **Justification** : Autoriser les sauvegardes via `android:allowBackup="true"` expose toutes les données stockées dans le dossier privé de l'application (comme les préférences partagées, les bases de données et les fichiers locaux). Les attaquants physiques peuvent utiliser la commande `adb backup` pour exfiltrer ces informations locales.

---

## FIND-003 : Demande de permission système hautement privilégiée (DUMP)
- **Catégorie OWASP** : **MASVS-PLATFORM** (Interaction avec la plateforme)
- **Référence spécifique** : **MASVS-PLATFORM-1** ("L'application respecte le principe du moindre privilège lors de la demande d'accès aux API et fonctionnalités système.")
- **Justification** : Demander la permission système hautement privilégiée `android.permission.DUMP` alors que les fonctionnalités de l'application ne le requièrent pas est une violation directe du moindre privilège de la plateforme Android. Cela étend inutilement la surface d'attaque en cas de compromission.

---

## FIND-004 : Exposition de secrets dans la bibliothèque native (helloFromJNI)
- **Catégorie OWASP** : **MASVS-STORAGE** (Stockage de données sensibles)
- **Référence spécifique** : **MASVS-STORAGE-1** ("L'application ne stocke pas de données sensibles en texte clair ou de manière non sécurisée.")
- **Justification** : Bien que codée dans le code natif C++ compilé (`libnative-lib.so`), la chaîne de caractères statique `"Hello from C++ via JNI !"` reste présente en clair dans la section `.rodata` du binaire. Elle est facilement identifiable via l'outil `strings`. Le stockage codé en dur de secrets ou de données sensibles viole le stockage sécurisé.

---

## FIND-005 : Absence d'obfuscation de la bibliothèque native
- **Catégorie OWASP** : **MASVS-RESILIENCE** (Résilience aux attaques inverses)
- **Référence spécifique** : **MASVS-RESILIENCE-1** ("L'application intègre des protections pour rendre la rétro-ingénierie difficile.")
- **Justification** : La bibliothèque native `.so` conserve tous ses noms de méthodes JNI et ses symboles d'origine en texte clair. L'absence de stripping des symboles de debug ou d'obfuscation de symboles rend le binaire hautement vulnérable à l'analyse par décompilateur et enfreint la résilience du code natif.
