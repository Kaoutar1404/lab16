# Android SSL Pinning Bypass avec Objection

## Introduction

Ce guide explique comment contourner le SSL Pinning sur Android avec Objection et Frida afin d’intercepter le trafic HTTPS d’une application mobile dans Burp Suite ou mitmproxy.

Objectifs :
- Installer Objection et Frida
- Démarrer frida-server
- Configurer un proxy HTTPS
- Désactiver le SSL Pinning
- Intercepter les requêtes HTTPS Android

---

# Étape 1 — Installer Objection et Frida

## Installation recommandée avec pipx

bash
pip install --user pipx


bash
pipx ensurepath


bash
pipx install objection


---

## Installation classique avec pip

bash
pip install --upgrade objection frida frida-tools


---

## Vérification des versions

bash
objection --version


bash
frida --version


bash
python -c "import frida; print(frida.__version__)"


---

## Remarque Windows

Si la commande objection n’est pas reconnue :

Ajouter au PATH :

text
%USERPROFILE%\AppData\Roaming\Python\Python311\Scripts


---

# Étape 2 — Préparer Android et démarrer frida-server

## Activer le Débogage USB

Sur Android :

text
Paramètres
→ Options développeur
→ Activer "Débogage USB"


---

## Vérifier la connexion ADB

bash
adb devices


Résultat attendu :

bash
device


---

## Vérifier l’architecture CPU

bash
adb shell getprop ro.product.cpu.abi


Exemples :
- arm64-v8a
- armeabi-v7a
- x86_64

---

## Télécharger frida-server

Télécharger la version correspondant exactement à :

bash
frida --version


Depuis :

text
https://github.com/frida/frida/releases


---

## Déployer frida-server

bash
adb push frida-server /data/local/tmp/


---

## Donner les permissions

bash
adb shell chmod 755 /data/local/tmp/frida-server


---

## Lancer frida-server

bash
adb shell "/data/local/tmp/frida-server -l 0.0.0.0"


---

## Redirection des ports (optionnel)

bash
adb forward tcp:27042 tcp:27042


bash
adb forward tcp:27043 tcp:27043


---

## Vérifier Frida

bash
frida-ps -Uai


Résultat attendu :
- Liste des applications Android visibles

---

# Étape 3 — Configurer le proxy HTTPS

## Lancer Burp Suite

Dans Burp :

text
Proxy → Intercept


Adresse exemple :

text
192.168.X.Y:8080


---

## Lancer mitmproxy

bash
mitmproxy -p 8080


---

# Étape 4 — Configurer le proxy sur Android

## Proxy Wi-Fi Android

Configurer :
- Adresse IP du PC
- Port 8080

---

## Installer le certificat CA

### Burp Suite

Depuis le téléphone :

text
http://burp


---

### mitmproxy

Depuis le téléphone :

text
http://mitm.it


Choisir :
- Android Certificate

---

## Installer comme certificat CA utilisateur

Android :

text
Installer certificat
→ Certificat CA


---

# Étape 5 — Vérifier le proxy

Depuis le téléphone :
- Ouvrir un site HTTPS
- Vérifier que Burp ou mitmproxy voit les requêtes

Remarque :
- Certaines applications continueront à bloquer à cause du SSL Pinning tant qu’Objection n’est pas lancé.

---

# Étape 6 — Identifier le package Android

## Rechercher les applications

powershell
frida-ps -Uai | Select-String -Pattern https,ssl,okhttp


---

## Exemple de package

text
com.example.app


---

# Étape 7 — Désactiver le SSL Pinning avec Objection

## Méthode 1 — Spawn (recommandée)

Injection dès le démarrage :

bash
objection -g com.example.app explore --startup-command "android sslpinning disable"


---

## Méthode 2 — Attach

### Ouvrir l’application normalement

Puis :

bash
objection -g com.example.app explore


Dans la console Objection :

bash
android sslpinning disable


---

# Étape 8 — Résultat attendu

Dans la console Objection :
- Hooks SSL installés
- Messages de bypass

Dans Burp/mitmproxy :
- Requêtes HTTPS visibles
- Headers visibles
- Réponses visibles

Dans l’application :
- Plus d’erreurs SSL
- Plus d’échec de certificat

---

# Étape 9 — Commandes utiles Objection

## Aide SSL Pinning

bash
help android sslpinning


---

## Recherche de classes liées au pinning

bash
android hooking search classes pin


---

## Recherche de méthodes SSL

bash
android hooking search methods ssl


---

# Étape 10 — Validation

## Générer du trafic

Tester :
- Login
- API
- Navigation
- Requêtes réseau

---

## Vérifier Burp Suite

Les requêtes HTTPS doivent apparaître :
- URL
- Méthodes HTTP
- Headers
- Corps des requêtes/réponses

---

## Vérifier Objection

Observer :
- Logs des hooks SSL
- Messages de bypass

---

# Dépannage

## Objection ne détecte pas l’appareil

Vérifier :

bash
adb devices


Puis :

bash
frida-ps -Uai


---

## Version incompatible

Les versions doivent correspondre :

bash
frida --version


=

text
frida-server


---

## L’application crash avec spawn

Essayer attach :

bash
objection -g com.example.app explore


Puis :

bash
android sslpinning disable


---

## Rien n’apparaît dans Burp

Vérifier :
- Proxy Android
- Certificat CA installé
- Burp actif
- Même réseau Wi-Fi
- SSL Pinning désactivé

---

# Livrables

## Captures demandées

- objection --version
- frida --version
- frida-ps -Uai
- Console Objection
- Requêtes HTTPS dans Burp

---

## Commandes utilisées

### Spawn

bash
objection -g com.example.app explore --startup-command "android sslpinning disable"


---

### Attach

bash
objection -g com.example.app explore


Puis :

bash
android sslpinning disable


---

# Structure du projet

text
/project/
 ├── screenshots/
 │    ├── objection_console.png
 │    ├── burp_requests.png
 │    └── frida_ps.png
 │
 ├── logs/
 │    └── objection_logs.txt
 │
 └── README.md


---

# Outils utilisés

| Outil | Description |
|---|---|
| Objection | Framework mobile basé sur Frida |
| Frida | Instrumentation dynamique Android |
| frida-server | Service Frida Android |
| Burp Suite | Proxy HTTPS |
| mitmproxy | Proxy HTTPS |
| ADB | Android Debug Bridge |
| OkHttp | Client HTTP Android |

---

# Conclusion

Cette méthode permet :
- De contourner le SSL Pinning Android
- D’intercepter le trafic HTTPS
- D’analyser les API mobiles
- De comprendre le fonctionnement d’Objection et Frida
- De réaliser des tests de sécurité mobile
