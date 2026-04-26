# Lab 4 — Analyse statique d'un APK Android

> **Cours** : Sécurité des applications mobiles  
> **Environnement** : Mobexler VM  
> **Date** : 26 avril 2026  
> **APK analysé** : OWASP UnCrackable Level 1

---

## Objectifs

- Comprendre la structure interne d'un APK
- Analyser l'`AndroidManifest.xml` pour identifier les permissions et composants exposés
- Explorer le code source décompilé avec **JADX GUI**
- Convertir des fichiers DEX en JAR avec **dex2jar** et les analyser avec **JD-GUI**
- Identifier des vulnérabilités courantes (secrets en clair, logs sensibles, mauvaise cryptographie)
- Produire un rapport d'audit professionnel

---

## APK analysé

| Champ | Valeur |
|---|---|
| Nom | `UnCrackable-Level1.apk` |
| Source | [OWASP MASTG GitHub](https://github.com/OWASP/mastg/tree/master/Crackmes/Android/Level_01) |
| Version | 1.0 |
| Taille | 66 KB |
| SHA-256 | `1da8bf57d266109f9a07c01bf7111a1975ce01f190b9d914bcd3ae3dbef96f21` |
| Package | `owasp.mstg.uncrackable1` |
| minSdk | 19 (Android 4.4) |
| targetSdk | 28 (Android 9) |

---

## Outils utilisés

| Outil | Usage |
|---|---|
| **JADX GUI** | Décompilation de l'APK, analyse du manifeste et du code source |
| **dex2jar** | Conversion du bytecode DEX en fichier JAR |
| **JD-GUI** | Analyse du JAR et comparaison avec JADX |
| **Terminal (zsh)** | Extraction, vérification SHA-256, wget |


<img width="954" height="592" alt="Capture d&#39;écran 2026-04-26 153046" src="https://github.com/user-attachments/assets/707d6fea-a691-438a-b8a9-373cb8f6d088" />

---


## Structure de l'APK

```
UnCrackable-Level1.apk
├── AndroidManifest.xml       (1 648 octets)
├── classes.dex               (5 528 octets)
├── META-INF/
│   ├── CERT.RSA
│   ├── CERT.SF
│   └── MANIFEST.MF
├── res/
│   ├── layout/activity_main.xml
│   ├── menu/menu_main.xml
│   └── mipmap-*/ic_launcher.png  (5 densités)
└── resources.arsc
```
<img width="797" height="402" alt="Capture d&#39;écran 2026-04-26 152801" src="https://github.com/user-attachments/assets/3aae1d90-7041-4c92-9afe-e34d8c92c7bb" />

---

## Vulnérabilités identifiées

| # | Vulnérabilité | Sévérité | Localisation |
|---|---|---|---|
| 1 | Clé AES hardcodée dans le code source | 🔴 Élevée | `sg.vantagepoint.uncrackable1.a` |
| 2 | Secret chiffré hardcodé (Base64) | 🔴 Élevée | `sg.vantagepoint.uncrackable1.a` |
| 3 | AES/ECB sans vecteur d'initialisation | 🔴 Élevée | `sg.vantagepoint.a.a` |
| 4 | `android:allowBackup="true"` | 🔴 Élevée | `AndroidManifest.xml` |
| 5 | Root detection bypassable | 🟡 Moyenne | `sg.vantagepoint.a.c` |
| 6 | Debug detection bypassable | 🟡 Moyenne | `sg.vantagepoint.a.b` |
| 7 | Log de debug en production (`CodeCheck`) | 🟢 Faible | `sg.vantagepoint.uncrackable1.a` |

---

## Task 4 — Recherche de chaînes sensibles

Recherches effectuées via **JADX GUI → Text Search (Ctrl+Shift+F)** :

| Terme cherché | Résultats | Sévérité |
|---|---|---|
| `key` | 4 — `SecretKeySpec`, `AES/ECB/PKCS7Padding`, `test-keys` | 🔴 Élevée |
| `secret` | 4 — `SecretKeySpec` + `"This is the correct secret."` | 🔴 Élevée |
| `debug` | 1 — `"App is debuggable!"` dans `MainActivity` | 🟡 Moyenne |
| `CodeCheck` | 1 — `Log.d("CodeCheck", "AES error:")` | 🟢 Faible |
| `5UJiFctbmgb...` | 1 — clé AES + secret Base64 hardcodés | 🔴 Élevée |
| `http` | 0 — aucune URL en clair | — |
| `su` | Chemins root hardcodés dans la classe `c` | 🟡 Moyenne |

---
<img width="1263" height="498" alt="Capture d&#39;écran 2026-04-26 153645" src="https://github.com/user-attachments/assets/e546a7b8-96ea-44c5-b45e-69909558fd3d" />

<img width="1282" height="502" alt="Capture d&#39;écran 2026-04-26 153657" src="https://github.com/user-attachments/assets/591a5b8f-4915-45db-89e6-0ceb7e464cff" />

<img width="1255" height="484" alt="Capture d&#39;écran 2026-04-26 153727" src="https://github.com/user-attachments/assets/9ef7f47e-2913-40eb-9c27-3e5a7561419d" />

<img width="1273" height="356" alt="Capture d&#39;écran 2026-04-26 154024" src="https://github.com/user-attachments/assets/f5261fbb-adc3-4dda-b201-35c890282e80" />

<img width="1786" height="320" alt="Capture d&#39;écran 2026-04-26 154101" src="https://github.com/user-attachments/assets/dc4b9ba2-b3ad-485d-a098-83c39fc79646" />




---
 Le rapport ://docs.google.com/document/d/1y_3LI9VktJ8JY1gWn_YpDQ3kuWpZZ5_j/edit?usp=drive_link&ouid=115915294887889410221&rtpof=true&sd=true

## Comparaison JADX GUI vs JD-GUI

<img width="1508" height="402" alt="Capture d&#39;écran 2026-04-26 154810" src="https://github.com/user-attachments/assets/470a0491-4e48-48c8-a327-8eb6dada3052" />

<img width="1157" height="392" alt="Capture d&#39;écran 2026-04-26 154842" src="https://github.com/user-attachments/assets/78918b02-9cc2-4a72-8820-281a170bc669" />

<img width="954" height="592" alt="Capture d&#39;écran 2026-04-26 153046" src="https://github.com/user-attachments/assets/5c8bba08-98db-403f-8968-62d247142afc" />


| Aspect | JADX GUI | JD-GUI |
|---|---|---|
| Format d'entrée | APK directement | JAR (dex2jar requis) |
| Navigation | Structure Android complète | Packages Java uniquement |
| Noms de paramètres | `bArr`, `bArr2` | `paramArrayOfbyte1`, `paramArrayOfbyte2` |
| Accès ressources | `AndroidManifest.xml`, `strings.xml` | Aucun accès |
| Cas d'usage | Analyse complète APK | Vérification secondaire JAR |

**Conclusion** : JADX GUI est l'outil recommandé pour l'analyse statique d'APK Android.

---


## Commandes utilisées

# Téléchargement de l'APK
wget https://raw.githubusercontent.com/OWASP/owasp-mastg/master/Crackmes/Android/Level_01/UnCrackable-Level1.apk

<img width="831" height="304" alt="Capture d&#39;écran 2026-04-26 152707" src="https://github.com/user-attachments/assets/37deff92-1d8b-4d1c-9dc3-77f94b973ab8" />

# Vérification de l'intégrité
sha256sum UnCrackable-Level1.apk
file UnCrackable-Level1.apk

<img width="806" height="196" alt="Capture d&#39;écran 2026-04-26 152746" src="https://github.com/user-attachments/assets/0476ab0a-b92b-405b-87a2-c4ef7ca22afd" />

unzip -l UnCrackable-Level1.apk | head -20

<img width="797" height="402" alt="Capture d&#39;écran 2026-04-26 152801" src="https://github.com/user-attachments/assets/4bdd8721-226f-4f70-85d5-4c2495a60428" />

# Lancement de JADX GUI
jadx-gui UnCrackable-Level1.apk &

<img width="1781" height="563" alt="Capture d&#39;écran 2026-04-26 152903" src="https://github.com/user-attachments/assets/41a091e3-1883-456c-95bc-9418bdc7a86d" />

# Extraction du DEX
unzip -j UnCrackable-Level1.apk "classes.dex" -d dex_out/

<img width="717" height="264" alt="Capture d&#39;écran 2026-04-26 154423" src="https://github.com/user-attachments/assets/5cda04d4-ee22-4397-8534-7abb971460b2" />

# Conversion DEX → JAR
d2j-dex2jar dex_out/classes.dex -o jar_out/app.jar

<img width="897" height="446" alt="Capture d&#39;écran 2026-04-26 154541" src="https://github.com/user-attachments/assets/fedc83f9-b518-4e2a-88d9-a0b0d4f8c847" />

# Lancement de JD-GUI
java -jar jd-gui.jar jar_out/app.jar &

<img width="859" height="72" alt="Capture d&#39;écran 2026-04-26 154749" src="https://github.com/user-attachments/assets/e2849c1d-5cab-4604-b7a3-2c8e4c6afb0e" />



## Avertissement éthique

> Ce lab est réalisé dans un cadre strictement pédagogique.  
> L'APK analysé est un challenge officiel OWASP conçu à des fins éducatives.  
> Aucune exploitation réelle n'a été effectuée.
