# Notes d'analyse BeVigil

Ce document compile les résultats d'analyse de la surface d'attaque externe de l'application **com.example.jnidemo** via la plateforme d'intelligence de sécurité BeVigil.

## Ce qui est certain
- **Package de l'application** : `com.example.jnidemo` (confirmé par l'analyse du manifeste).
- **Nom de l'application** : `JNIDemo Basics` (nom de l'activité principale dans les ressources).
- **Compilation native NDK** : L'APK embarque des bibliothèques partagées compilées (`libnative-lib.so`) pour plusieurs architectures matérielles (`x86`, `x86_64`, `armeabi-v7a`, `arm64-v8a`), ce qui démontre le recours aux fonctionnalités bas niveau C/C++.
- **Environnement de compilation** : L'utilisation du compilateur LLVM/Clang de Google NDK est avérée par la présence de métadonnées internes.

## Ce qui est hypothèse
- **Backend externe absent** : L'application semble autonome (offline) car BeVigil n'identifie aucune API de production tierce ou d'infrastructure cloud dédiée (type AWS, Azure) en dehors des liens de dépendance standards de Google.
- **But uniquement pédagogique** : Au vu de la structure simpliste et des méthodes exportées par la JNI, le but exclusif de cette cible est d'expérimenter le pont d'intégration native JNI.

## Points d'intérêt
- **Exposition de métadonnées du compilateur** : Fuite de l'URL du dépôt de la chaîne d'outils NDK de Google (`https://android.googlesource.com/toolchain/llvm-project`).
- **Absence de protection sur la bibliothèque native** : Les symboles de JNI sont exportés en clair, ce qui rend l'application immédiatement vulnérable à une rétro-ingénierie avancée (reverse-engineering) ou à un détournement dynamique via Frida.

## Domaines et sous-domaines
- Aucun sous-domaine de production spécifique détecté.
- `android.googlesource.com` (Dépôt Git de Google pour Android).

## Endpoints et APIs
- Aucun point d'accès API ou backend REST/GraphQL n'est visible de l'extérieur.

## URLs HTTP/HTTPS
- `https://android.googlesource.com/toolchain/llvm-project` (Dépôt de la chaîne d'outils NDK).
- `http://schemas.android.com/apk/res-auto` (Schéma Android XML).
- `http://schemas.android.com/apk/res/android` (Espace de noms XML standard).

## Emails et identifiants
- Aucun email de développeur, clé API Firebase publique ou identifiant d'infrastructure cloud n'a été détecté dans les métadonnées de la plateforme externe BeVigil.

## Technologies détectées
- **Langages de programmation** : Kotlin, Java, C/C++ (JNI).
- **Framework NDK** : Google Android NDK (Clang compiler).
- **Bibliothèques Jetpack** : AndroidX Core, AndroidX Lifecycle, AndroidX Startup.
- **Ressources internes** : Vector drawables standard de Material Design.
