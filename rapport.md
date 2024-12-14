# Analyse TCP : Rapport d'analyse réseau

Ce rapport documente l'analyse d'une conversation TCP capturée avec Wireshark. La capture montre l'établissement, le transfert sécurisé et la fermeture d'une connexion TCP.

---

## **Introduction**

La conversation analysée utilise le filtre **`tcp.stream eq9`** et utilise le port source **62164** sur la machine locale (**192.168.1.2**) pour se connecter au port destination **443** (HTTPS) de l’adresse **152.199.20.80**. Le protocole **TLSv1.2** est utilisé au-dessus de TCP pour sécuriser la communication.

---

## **1. Ouverture de la connexion : Three-Way Handshake**

Le processus de **three-way handshake** est réalisé en trois étapes :

![Three-Way Handshake](https://github.com/user-attachments/assets/74ee45d8-0bca-4eb6-81c8-580ec0017a59)

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

## **3. Relation entre numéros de séquence, taille de segment et ACK**

### **Principe général :**
- Le **numéro de séquence (SEQ)** représente le premier octet de données dans un segment TCP.
- Le **numéro d’acquittement (ACK)** correspond au prochain octet attendu par le récepteur.
- La taille du segment (payload) détermine l’incrément des numéros SEQ et ACK.

### **Analyse dans la capture :**
Prenons un exemple d'évolution des numéros dans cette capture :
- **Trame 1110** :
  - **SEQ = 1**, aucune donnée envoyée (taille = 0).
  - **ACK = 1**, le serveur confirme la réception initiale.
- **Trame suivante (Data)** :
  - Si le client envoie **1460 octets** de données (taille du segment), le prochain SEQ sera incrémenté de 1460.
  - Le serveur acquittera avec `ACK = SEQ + taille_du_segment`, ici `ACK = 1461`.

Ce mécanisme assure la fiabilité et l'ordre des données transmises.

---

## **4. Pourquoi TCP est utilisé**

TCP est le protocole sous-jacent à **HTTP/TLS**, car il offre plusieurs avantages adaptés aux besoins de cette application :

1. **Fiabilité** :
   - Assure que les données sont correctement transmises sans pertes grâce aux numéros d’ACK et aux retransmissions.
2. **Transmission ordonnée** :
   - Les segments arrivent dans l’ordre correct, essentiel pour les requêtes/réponses HTTP.
3. **Contrôle de flux et congestion** :
   - TCP ajuste dynamiquement la vitesse de transmission selon la capacité du récepteur et la congestion réseau.

Pour ces raisons, TCP est préféré dans des cas comme les communications HTTPS où la fiabilité et l'ordre des données sont critiques.

---

## **5. Fermeture de la connexion : Four-Way Handshake**

La fermeture d’une connexion TCP suit un processus en quatre étapes, appelé **Four-Way Handshake**. Ce mécanisme permet aux deux parties (client et serveur) de libérer proprement les ressources utilisées pour la session.

![Fermeture TCP](https://github.com/user-attachments/assets/75ccca5c-e945-40e8-8c1d-b97563be9fe4)

### **Étapes observées dans la capture**

1. **Trame 1137 : FIN (client)**  
   - **Source :** 192.168.1.2  
   - **Destination :** 152.199.20.80  
   - **Flag TCP :** FIN, ACK  
   - **Numéro de séquence (SEQ) :** 557  
   - **Numéro d'acquittement (ACK) :** 4998  
   - **Description :**  
     Le client indique qu’il a terminé d’envoyer des données en mettant le flag **FIN**.

2. **Trame 1138 : ACK (serveur)**  
   - **Source :** 152.199.20.80  
   - **Destination :** 192.168.1.2  
   - **Flag TCP :** ACK  
   - **Numéro de séquence (SEQ) :** 4998  
   - **Numéro d'acquittement (ACK) :** 558  
   - **Description :**  
     Le serveur confirme la réception du message FIN du client.

3. **Trame 1139 : FIN (serveur)**  
   - **Source :** 152.199.20.80  
   - **Destination :** 192.168.1.2  
   - **Flag TCP :** FIN, ACK  
   - **Numéro de séquence (SEQ) :** 4998  
   - **Numéro d'acquittement (ACK) :** 558  
   - **Description :**  
     Le serveur signale qu’il a terminé d’envoyer des données en envoyant un message FIN.

4. **Trame 1143 : ACK (client)**  
   - **Source :** 192.168.1.2  
   - **Destination :** 152.199.20.80  
   - **Flag TCP :** ACK  
   - **Numéro de séquence (SEQ) :** 558  
   - **Numéro d'acquittement (ACK) :** 4999  
   - **Description :**  
     Le client acquitte la réception du FIN du serveur, complétant ainsi le processus de fermeture.

---

### **Résumé de la fermeture**

- **Trames impliquées :** 1137, 1138, 1139, 1143.  
- **Protocole TCP :** La connexion est terminée de manière propre et ordonnée. Chaque côté signale la fin de ses transmissions avec un message FIN et reçoit une confirmation (ACK) de l’autre partie.
- **Objectif :** Assurer que toutes les données ont bien été transmises et reçues avant de libérer les ressources réseau.

---

### **Conclusion**
Ce rapport détaille les principaux mécanismes du protocole TCP :  
- L’établissement de la connexion avec le three-way handshake.  
- La transmission fiable des données avec des numéros de séquence et des acquittements.  
- Une fermeture propre grâce au four-way handshake.  

Ces analyses mettent en lumière pourquoi TCP est utilisé pour des communications nécessitant fiabilité, ordre, et contrôle strict des flux, comme dans le cas des protocoles sécurisés tels que HTTPS.

---





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

![trame 1103](https://github.com/user-attachments/assets/e2b7932d-2066-4e84-8bc3-5b1179502ffc)


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

![trame 1104](https://github.com/user-attachments/assets/1ffd73ac-7003-4e25-b062-9d79ec40974e)


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

![trames 1827 a 1832, dns echec](https://github.com/user-attachments/assets/ce506fee-86f7-4efd-9fcd-33a82cdbb0ce)


### **Résumé :**
- **Requêtes :**
  - Domaine demandé : `wpad.home`.  
  - Types d'enregistrements : **A** (IPv4) et **AAAA** (IPv6).  
- **Réponses :**
  - Code de réponse DNS : **No such name**, indiquant que le serveur DNS n’a pas trouvé d’enregistrements correspondant à ce domaine.

---

## **4. Exemple de résolution réussie avec plusieurs adresses**

Les trames **2909** (requête) et **2917** (réponse) illustrent la résolution du domaine `prod.otel.kaizen.nvidia.com`.

![trames 2909 et 2017 reussis](https://github.com/user-attachments/assets/1fe2d0c6-7d2c-47c2-8f21-15df10104405)


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

[lien de la capture général ](https://github.com/mm-elmazani/Analyse-de-traces-perso-sur-la-couche-transport/blob/main/Screenshots%20UDP/conv%20g%C3%A9r%C3%A9nal.png)
