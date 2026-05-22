# Lab_15_MobileSecurity
# LAB-15-Analyse-Dynamique-Android-Inspection-TLS-HTTPS-et-Gestion-du-SSL-Pinning
## Étape 1 — Installer Frida (PC) et démarrer frida-server (Android)
![](https://github.com/user-attachments/assets/047e7016-2c28-4cce-995b-5ce8860024bc)
![](https://github.com/user-attachments/assets/7c230d07-e269-43ef-84dd-5ce5112472b4)
![](https://github.com/user-attachments/assets/a67a9857-ef39-4c57-ad5b-8e0e62390f3b)
![](https://github.com/user-attachments/assets/60ce9960-4666-49c8-8390-4440e81a0f1e)
![](https://github.com/user-attachments/assets/e74acdd8-be0a-4d0f-9312-85fbf5dc30d8)
![](https://github.com/user-attachments/assets/e7edca9a-ccb3-48a1-b221-f58db4ee763d)
![](https://github.com/user-attachments/assets/23760fff-ec33-4857-886e-026d5cf2bc77)
![](https://github.com/user-attachments/assets/3d08cd6c-6583-44b1-bf5d-90282d681ccb)
## Étape 3 — Lancer l’app cible sous Frida
![](https://github.com/user-attachments/assets/a9934350-cce2-4b8d-a512-d166f2528b76)

## Étape 4 — Script « universel » Java pour bypass SSL pinning
![](https://github.com/user-attachments/assets/5887a2bb-f8ff-488e-ae4c-6e1b54856292)
## Étape 5 — Variantes et cibles spécifiques

![](https://github.com/user-attachments/assets/5ba24d3c-df83-4269-9fea-4712e195ad15)
![](https://github.com/user-attachments/assets/b98ee819-9881-4715-b055-d43bf6a2c9ef)

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
![](https://github.com/user-attachments/assets/654c23fa-ba17-4825-9ffe-806d1b1e360a)

