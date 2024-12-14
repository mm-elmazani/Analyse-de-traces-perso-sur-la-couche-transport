# Analyse TCP : Rapport d'analyse réseau

Ce rapport documente l'analyse d'une conversation TCP capturée avec Wireshark. La capture montre l'établissement, le transfert sécurisé et la fermeture d'une connexion TCP.

---

## **Introduction**

La conversation analysée utilise le filtre **`tcp.stream eq9`** et utilise le port source **62164** sur la machine locale (**192.168.1.2**) pour se connecter au port destination **443** (HTTPS) de l’adresse **152.199.20.80**. Le protocole **TLSv1.2** est utilisé au-dessus de TCP pour sécuriser la communication.

---

## **1. Ouverture de la connexion : Three-Way Handshake**

Le processus de **three-way handshake** est réalisé en trois étapes :

![three way handshake tcp stream eq9](https://github.com/user-attachments/assets/74ee45d8-0bca-4eb6-81c8-580ec0017a59)

### **Trame 1107 (SYN)**
- **Source :** 192.168.1.2  
- **Destination :** 152.199.20.80  
- **Flag TCP :** SYN  
- **Numéro de séquence (SEQ) :** 0  
- **Taille de fenêtre (Window Size) :** 65535 octets  
- **Options TCP :**  
  - **MSS=1460** (Maximum Segment Size) : Taille maximale des segments TCP autorisée par le client.  
  - **SACK_PERM** (Selective Acknowledgment Permitted) : Permet les acquittements sélectifs en cas de perte de segment.

### **Trame 1109 (SYN-ACK)**
- **Source :** 152.199.20.80  
- **Destination :** 192.168.1.2  
- **Flag TCP :** SYN, ACK  
- **Numéro de séquence (SEQ) :** 0  
- **Numéro d'acquittement (ACK) :** 1  
- **Taille de fenêtre (Window Size) :** 65535 octets  
- **Options TCP :**  
  - **MSS=1452** (Maximum Segment Size) : Taille de segment légèrement inférieure, adaptée au serveur.  
  - **SACK_PERM** (Selective Acknowledgment Permitted).

### **Trame 1110 (ACK)**
- **Source :** 192.168.1.2  
- **Destination :** 152.199.20.80  
- **Flag TCP :** ACK  
- **Numéro de séquence (SEQ) :** 1  
- **Numéro d'acquittement (ACK) :** 1  
- **Taille de fenêtre (Window Size) :** 65535 octets  

**Résumé :**  
L'ouverture de la connexion est complète après ces trois trames. Le client et le serveur sont maintenant prêts à échanger des données.

---

## **2. Transmission de données avec TLS**

Le protocole utilisé au-dessus de TCP est **TLSv1.2**, permettant une communication sécurisée. Les principales étapes du handshake TLS sont observées dans la capture :

![Handshake TLS](https://github.com/user-attachments/assets/f3999c06-a0dd-4494-a7ce-272108774b21)

### **Trame 1110 : Client Hello**
- **Description :** Le client initie le handshake TLS.  
- **Informations clés :**  
  - **SNI (Server Name Indication) :** `ops-gx.nvidia.com` (le nom de domaine demandé).  
  - **Algorithmes de chiffrement supportés.**

### **Trame 1122 : Server Hello**
- **Description :** Le serveur répond avec les paramètres TLS, incluant :  
  - Choix des algorithmes de chiffrement (ex. : AES).  
  - Certificat émis par **DigiCert** pour authentifier le serveur.

### **Échange de clés et données sécurisées**
- Les trames suivantes montrent :  
  - L’échange des clés nécessaires pour chiffrer la session.  
  - Le début de l’envoi de données chiffrées sous forme d’Application Data.

---

## **3. Analyse des trames 1123 à 1137 : Transition TLS vers fermeture**

![Transition TLS vers fermeture](https://github.com/user-attachments/assets/d91440e1-6329-45a3-8933-ca176c01882d)


Entre les trames **1123** et **1137**, on observe deux événements importants :

### **Échange de données chiffrées (Application Data)**  
- Les trames **1123 à 1135** montrent un échange actif de **données chiffrées** via TLSv1.2.  
- Ces trames contiennent des données applicatives échangées entre le client et le serveur.  
- Les contenus précis ne sont pas visibles, car le protocole TLS chiffre les messages pour garantir leur confidentialité.  
- **Évolution des numéros SEQ et ACK** :
  - Chaque trame montre l’évolution des numéros de séquence (`SEQ`) et d’acquittement (`ACK`), confirmant une transmission fluide sans pertes apparentes.

### **Trames 1136 et 1137 : Préparation à la fermeture**
- **Trame 1136 : ACK**  
  - Le client acquitte la réception des données envoyées par le serveur (`ACK=4998`).  
- **Trame 1137 : Encrypted Alert**  
  - Le serveur envoie une alerte chiffrée via TLS.  
  - Il s’agit probablement d’une alerte de type **Close Notify**, utilisée pour indiquer la fin normale de la session TLS.

Ces trames marquent la fin de la communication via TLS et la transition vers la fermeture de la connexion TCP.

---

## **4. Fermeture de connexion : Four-Way Termination**

![Fermeture de connexion](https://github.com/user-attachments/assets/e2ee7c4d-945c-41e3-b524-9087f1bacfc9)

### **Trame 1138 (FIN, ACK)**
- **Source :** 192.168.1.2  
- **Destination :** 152.199.20.80  
- **Flags TCP :** FIN, ACK  
- **Numéro de séquence (SEQ) :** 588  
- **Numéro d'acquittement (ACK) :** 4998  

### **Trame 1139 (ACK)**
- **Source :** 152.199.20.80  
- **Destination :** 192.168.1.2  
- **Flags TCP :** ACK  
- **Numéro de séquence (SEQ) :** 4998  
- **Numéro d'acquittement (ACK) :** 589  

### **Trame 1140 (FIN, ACK)**
- **Source :** 152.199.20.80  
- **Destination :** 192.168.1.2  
- **Flags TCP :** FIN, ACK  
- **Numéro de séquence (SEQ) :** 4998  
- **Numéro d'acquittement (ACK) :** 589  

### **Trame 1143 (ACK)**
- **Source :** 192.168.1.2  
- **Destination :** 152.199.20.80  
- **Flags TCP :** ACK  
- **Numéro de séquence (SEQ) :** 589  
- **Numéro d'acquittement (ACK) :** 4999  

**Résumé :**  
La connexion est proprement terminée après ces quatre trames.

---

## **Conclusion**

Cette analyse montre une conversation TCP complète :
- [**Ouverture de connexion :**](https://github.com/mm-elmazani/Analyse-de-traces-perso-sur-la-couche-transport/blob/main/Screenshots/three%20way%20handshake%20tcp.stream%20eq9.png) Réalisée via le three-way handshake.  
- [**Transfert sécurisé :**](https://github.com/mm-elmazani/Analyse-de-traces-perso-sur-la-couche-transport/blob/main/Screenshots/Handshake%20TLS.png) Réalisé via TLSv1.2, protégeant les données échangées.  
- [**Transition TLS :**](https://github.com/mm-elmazani/Analyse-de-traces-perso-sur-la-couche-transport/blob/main/Screenshots/Transition%20TLS%20vers%20fermeture.png)La session sécurisée est terminée avec un message d’alerte TLS (`Close Notify`).   
- [**Fermeture de connexion :**](https://github.com/mm-elmazani/Analyse-de-traces-perso-sur-la-couche-transport/blob/main/Screenshots/Fermeture%20de%20connexion.png) Effectuée proprement via le four-way termination.




# Analyse UDP : Rapport d'analyse réseau

Ce rapport documente l'analyse d'une conversation UDP capturée avec Wireshark. La capture montre des échanges DNS, protocole utilisé pour la résolution de noms de domaine, via UDP sur le port 53.

---

## **Introduction**

Le **Domain Name System (DNS)** est un protocole utilisé pour traduire des noms de domaine en adresses IP ou pour récupérer d'autres informations liées à ces domaines. Il fonctionne généralement sur le port **53** en utilisant le protocole **UDP** pour des échanges rapides et légers.

Dans cette analyse, nous observons des requêtes et réponses DNS :
- Requêtes pour des domaines comme `ops.gx.nvidia.com`, `wpad.home`, et `prod.otel.kaizen.nvidia.com`.
- Réponses incluant différents types d'enregistrements : **CNAME**, **A**, **AAAA**, et **SOA**.

---

## **1. Analyse d'une requête DNS**

Prenons la **trame 1103** comme exemple d'une requête DNS.


### **Informations générales**
- **Protocole :** DNS via UDP.
- **Source :** `fe80::f02b:a5e6:a096:ac2e` (client).  
- **Destination :** `fe80::46d4:54ff:fef6:ble3` (serveur DNS).  
- **Info :** Requête DNS de type **AAAA** pour le domaine `ops.gx.nvidia.com`.

### **Analyse des champs UDP**
- **Port source :** Dynamique (choisi par le client).  
- **Port destination :** 53 (standard DNS).  
- **Longueur :** 97 octets (taille totale du datagramme UDP).  
- **Checksum :** Inclus pour vérifier l'intégrité des données.

### **Analyse des champs DNS**
- **Transaction ID :** `0x4200` (identifiant unique pour associer cette requête à sa réponse).  
- **Flags DNS :**  
  - `0x0100` : Indique une requête standard (pas une réponse).  
  - La récursion n'est pas demandée.  
- **Question DNS :**  
  - Domaine demandé : `ops.gx.nvidia.com`.  
  - Type d'enregistrement : **AAAA** (adresse IPv6).  

---

## **2. Analyse d'une réponse DNS**

Prenons la **trame 1104** comme exemple d'une réponse DNS.

### **Informations générales**
- **Protocole :** DNS via UDP.
- **Source :** `fe80::46d4:54ff:fef6:ble3` (serveur DNS).  
- **Destination :** `fe80::f02b:a5e6:a096:ac2e` (client).  
- **Info :** Réponse DNS pour `ops.gx.nvidia.com` avec des enregistrements **CNAME**.

### **Analyse des champs UDP**
- **Port source :** 53 (serveur DNS).  
- **Port destination :** Dynamique (attribué par le client).  
- **Longueur :** 179 octets (taille totale du datagramme UDP).  
- **Checksum :** Inclus.

### **Analyse des champs DNS**
- **Transaction ID :** `0x4200` (correspondant à la requête dans la trame 1103).  
- **Flags DNS :**  
  - `0x8180` : Indique qu’il s’agit d’une réponse.  
  - La récursion est disponible.  
- **Réponse DNS :**  
  - **CNAME** : Le domaine `ops.gx.nvidia.com` est un alias vers `cs1137.wpc.ea55a.phicdn.net`.  
  - **CNAME final :** `cs1137261584.wpc.phicdn.net`.  

---

## **3. Analyse des requêtes et réponses échouées**

Les trames **1827 à 1832** montrent des requêtes DNS pour le domaine `wpad.home` qui échouent.

### **Résumé :**
- **Requêtes :**
  - Domaine demandé : `wpad.home`.  
  - Types d'enregistrements : **A** (IPv4) et **AAAA** (IPv6).  
- **Réponses :**
  - Code de réponse DNS : **No such name**, indiquant que le serveur DNS n’a pas trouvé d’enregistrements correspondant à ce domaine.

---

## **4. Exemple de résolution réussie avec plusieurs adresses**

Les trames **2909** (requête) et **2917** (réponse) illustrent la résolution du domaine `prod.otel.kaizen.nvidia.com`.

### **Réponse DNS :**
- Domaine demandé : `prod.otel.kaizen.nvidia.com`.
- Types d'enregistrements retournés :
  - **A (IPv4)** :  
    - 3 adresses IPv4 : `3.71.226.131`, `63.176.90.252`, et `63.176.236.234`.
  - **SOA (Start of Authority)** :  
    - Informations sur le serveur DNS principal pour le domaine.

---

## **5. Relation entre DNS et UDP**

### **Pourquoi UDP est utilisé pour DNS ?**
- **Rapidité :** UDP est sans connexion, donc les requêtes/réponses sont envoyées rapidement sans établir une session préalable, contrairement à TCP.  
- **Légereté :** Les échanges DNS sont généralement courts (une question, une réponse), donc les fonctionnalités de fiabilité de TCP ne sont pas nécessaires.  
- **Cas particulier :** Si la réponse DNS dépasse la taille limite d’un datagramme UDP (512 octets dans certains cas), DNS peut basculer vers TCP.

### **Avantages de l’utilisation d’UDP pour DNS :**
- Permet des résolutions rapides pour une grande majorité des requêtes.  
- Diminue la surcharge réseau par rapport à TCP.  

---

## **Conclusion**

Cette analyse met en évidence le fonctionnement typique d’un échange DNS sur UDP :
- Une requête est envoyée par un client vers un serveur DNS sur le port **53**.
- Le serveur DNS répond avec les informations demandées ou un message d'erreur si le domaine est introuvable.
- Les différents types d’enregistrements DNS (CNAME, A, AAAA, SOA) montrent la diversité des données que DNS peut fournir.

En utilisant **UDP**, DNS privilégie la rapidité et la simplicité pour répondre à la majorité des besoins de résolution de noms.
