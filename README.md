# Lab_15_MobileSecurity
# LAB-15-Analyse-Dynamique-Android-Inspection-TLS-HTTPS-et-Gestion-du-SSL-Pinning
## Étape 1 — Installer Frida (PC) et démarrer frida-server (Android)
![Step 1](screenshots/lab15_01.png)
![Step 2](screenshots/lab15_02.png)
![Step 3](screenshots/lab15_03.png)
![Step 4](screenshots/lab15_04.png)
![Step 5](screenshots/lab15_05.png)
![Step 6](screenshots/lab15_06.png)
![Step 7](screenshots/lab15_07.png)
![Step 8](screenshots/lab15_08.png)
## Étape 3 — Lancer l’app cible sous Frida
![Step 9](screenshots/lab15_09.png)

## Étape 4 — Script « universel » Java pour bypass SSL pinning
![Step 10](screenshots/lab15_10.png)
## Étape 5 — Variantes et cibles spécifiques

![Step 11](screenshots/lab15_11.png)
![Step 12](screenshots/lab15_12.png)

Après l’analyse dynamique de l’application :contentReference[oaicite:0]{index=0} à l’aide de Frida, une phase de reconnaissance a été réalisée afin d’identifier les mécanismes de protection SSL utilisés. Cette étape est essentielle pour adapter la technique de contournement en fonction de l’implémentation réelle du pinning.

---

##  5.1 Identification des bibliothèques utilisées

L’exécution d’un script de scan des classes Java chargées a permis de détecter plusieurs composants liés à la sécurité réseau :

- Présence de **OkHttp (`com.android.okhttp`)**
- Présence de **CertificatePinner**
- Présence de **X509TrustManager (`javax.net.ssl`)**
- Présence de **TrustManagerImpl (Conscrypt)**

Ces résultats montrent que l’application utilise une **architecture hybride de validation SSL**, combinant plusieurs mécanismes de sécurité.

---

##  5.2 Classification du mécanisme de protection

D’après les résultats obtenus, le système de sécurité appartient à la catégorie :

>  **Double SSL pinning (OkHttp + TrustManager)**

Cela signifie que la validation TLS repose sur plusieurs couches :

- **Couche 1 : OkHttp CertificatePinner**  
  Validation des certificats serveur au niveau réseau HTTP.
  
- **Couche 2 : TrustManager Android (Conscrypt)**  
  Validation des certificats au niveau système.

---

##  5.3 Stratégie de contournement adaptée

Contrairement aux applications utilisant une seule méthode de pinning, cette application nécessite une approche **ciblée et modulaire** :

###  Étape 1 — Bypass OkHttp
Interception de la méthode `CertificatePinner.check()` afin de neutraliser la vérification des certificats SSL.

###  Étape 2 — Bypass TrustManager
Surcharge des méthodes suivantes :

- `checkServerTrusted`
- `checkClientTrusted`

afin d’autoriser tous les certificats sans validation.

---

##  5.4 Importance de l’approche ciblée

L’utilisation d’un script universel (bypass global) peut entraîner :

- Crash de l’application
- Détection par mécanismes anti-instrumentation
- Appel de fonctions natives de protection (ex: `libfoo.so`)
- Fermeture volontaire du processus (anti-tampering)

Ainsi, une approche adaptée au résultat du scan est indispensable pour garantir la stabilité de l’application.

---

##  5.5 Conclusion de l’étape

Cette étape montre que la réussite du bypass SSL ne dépend pas uniquement de l’utilisation de Frida, mais principalement de :

- L’identification correcte des bibliothèques utilisées
- L’adaptation du hook selon l’architecture de l’application
- L’évitement des hooks globaux trop intrusifs
## ## Étape 6 — Cas avancé : Pinning natif (BoringSSL / OpenSSL)

Dans certains cas, le contournement du SSL pinning au niveau Java n’est pas suffisant.  
Si aucune requête n’apparaît dans le proxy (Burp Suite, mitmproxy, etc.), cela signifie que l’application effectue probablement la vérification SSL au niveau natif via des bibliothèques comme **BoringSSL** ou **OpenSSL**.

Dans ce cas, l’approche consiste à analyser et intercepter les fonctions natives responsables de la validation TLS, puis forcer un résultat valide.

### 6.1 Découverte des symboles natifs

La première étape consiste à identifier les fonctions TLS utilisées par l’application :
![Native symbols](screenshots/lab15_13.png)


La première étape consiste à identifier les fonctions TLS utilisées par l’application :
![](https://github.com/user-attachments/assets/654c23fa-ba17-4825-9ffe-806d1b1e360a)

