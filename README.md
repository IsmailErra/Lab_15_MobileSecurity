# Lab_15_MobileSecurity
# LAB‑15 – Analyse Dynamique Android – Inspection TLS/HTTPS & Gestion du SSL Pinning

## Étape 1 – Installer Frida (PC) et démarrer frida‑server (Android)
### 1.1 Installer Frida côté PC
```powershell
python -m pip install --upgrade frida frida-tools
```
### Vérification
```powershell
frida --version
python -c "import frida; print(frida.__version__)"
```
![Vérification de l'environnement](../Lab14/screenshots/1_frida_version.png)

### 1.2 Préparer ADB et l’appareil
Sur l’appareil : Options développeur → activer « Débogage USB ». Connectez‑le en USB et acceptez l’empreinte.
```powershell
adb devices
```
![Connexion ADB](../Lab14/screenshots/2_adb_devices.png)

### 1.3 Déployer et lancer frida‑server
```powershell
adb push frida-server /data/local/tmp/
adb shell chmod 755 /data/local/tmp/frida-server
adb shell "/data/local/tmp/frida-server -l 0.0.0.0"
frida-ps -Uai
```
![Démarrage de frida‑server](../Lab14/screenshots/3_frida_server.png)

## Étape 2 – Mettre en place le proxy et le certificat CA
### 2.1 Lancer le proxy sur le PC
- **Burp** : Proxy → Intercept ON/OFF (port ex 127.0.0.1:8080)
- **mitmproxy** : `mitmproxy -p 8080`

### 2.2 Installer la CA proxy sur l’appareil
Ouvrez `http://burp` ou `http://mitm.it` depuis le téléphone, téléchargez le certificat et installez‑le comme « certificat CA utilisateur ».

### 2.3 Rediriger le trafic de l’appareil vers le proxy
```powershell
adb reverse tcp:8080 tcp:8080
```
Vérifiez en naviguant vers un site HTTPS ; le proxy doit voir les requêtes.

## Étape 3 – Lancer l’application cible sous Frida
Identifiez le package :
```powershell
frida-ps -Uai | Select-String -Pattern okhttp,webview,ssl,https
```
Spawning (injection tôt) :
```powershell
frida -U -f <package_name> -l sslpin_bypass_universal.js --no-pause
```
Attache (si l’app est déjà lancée) :
```powershell
frida -U -n "<NomDuProcessus>" -l sslpin_bypass_universal.js
```

## Étape 4 – Script « universel » Java pour bypass SSL pinning
```javascript
// sslpin_bypass_universal.js – contenu complet fourni dans le cours
```
![Universal SSL pinning bypass installé](../Lab14/screenshots/4_frida_bypass.png)

## Étape 5 – Variantes et cibles spécifiques
- **OkHttp only** : patch `CertificatePinner.check()`
- **Conscrypt only** : patch `TrustManagerImpl` methods
- **WebView only** : patch `WebViewClient.onReceivedSslError`

## Étape 6 – Cas avancé : pinning natif (BoringSSL / OpenSSL)
Si aucune requête n’apparaît dans le proxy :
1. Découvrir les symboles natifs :
```powershell
frida-trace -U -i SSL_* -i X509_* <package_name>
```
2. Hook minimal :
```javascript
// sslpin_bypass_native.js – hook SSL_get_verify_result, etc.
```
Lancer les deux scripts :
```powershell
frida -U -f <package_name> -l sslpin_bypass_universal.js -l sslpin_bypass_native.js --no-pause
```

## Étape 7 – Validation et livrables
- Vérifiez dans le proxy que les requêtes HTTPS de l’app sont visibles en clair.
- Capturez les logs Frida contenant les messages `[+] SSL bypass:`.
- Fournissez :
  - Capture de `frida --version` & `frida-ps -Uai`
  - Scripts utilisés (`sslpin_bypass_universal.js`, `sslpin_bypass_native.js`)
  - Capture du proxy montrant une requête HTTPS
  - Journal Frida avec au moins une ligne `SSL bypass`.
