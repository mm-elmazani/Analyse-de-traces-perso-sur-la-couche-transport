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
## **3. Fermeture de la connexion : Four-Way Handshake**

La fermeture d’une connexion TCP suit un processus en quatre étapes, appelé **Four-Way Handshake**. Ce mécanisme permet aux deux parties (client et serveur) de libérer proprement les ressources utilisées pour la session.

### **Étapes observées dans la capture**

1. **Trame 1123 : FIN (client)**  
   - **Source :** 192.168.1.2  
   - **Destination :** 152.199.20.80  
   - **Flag TCP :** FIN, ACK  
   - **Numéro de séquence (SEQ) :** 5001  
   - **Numéro d'acquittement (ACK) :** 4001  
   - **Description :**  
     Le client indique qu’il a terminé d’envoyer des données en mettant le flag **FIN**.

2. **Trame 1124 : ACK (serveur)**  
   - **Source :** 152.199.20.80  
   - **Destination :** 192.168.1.2  
   - **Flag TCP :** ACK  
   - **Numéro de séquence (SEQ) :** 4001  
   - **Numéro d'acquittement (ACK) :** 5002  
   - **Description :**  
     Le serveur confirme la réception du message FIN du client.

3. **Trame 1137 : FIN (serveur)**  
   - **Source :** 152.199.20.80  
   - **Destination :** 192.168.1.2  
   - **Flag TCP :** FIN, ACK  
   - **Numéro de séquence (SEQ) :** 4001  
   - **Numéro d'acquittement (ACK) :** 5002  
   - **Description :**  
     Le serveur signale qu’il a terminé d’envoyer des données en envoyant un message FIN.

4. **Trame 1138 : ACK (client)**  
   - **Source :** 192.168.1.2  
   - **Destination :** 152.199.20.80  
   - **Flag TCP :** ACK  
   - **Numéro de séquence (SEQ) :** 5002  
   - **Numéro d'acquittement (ACK) :** 4002  
   - **Description :**  
     Le client acquitte la réception du FIN du serveur, complétant ainsi le processus de fermeture.

---

### **Résumé de la fermeture**

- **Trames impliquées :** 1123, 1124, 1137, 1138.  
- **Protocole TCP :** La connexion est terminée de manière propre et ordonnée. Chaque côté signale la fin de ses transmissions avec un message FIN et reçoit une confirmation (ACK) de l’autre partie.
- **Objectif :** Assurer que toutes les données ont bien été transmises et reçues avant de libérer les ressources réseau.




## **4. Relation entre numéros de séquence, taille de segment et ACK**

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

## **5. Pourquoi TCP est utilisé**

TCP est le protocole sous-jacent à **HTTP/TLS**, car il offre plusieurs avantages adaptés aux besoins de cette application :

1. **Fiabilité** :
   - Assure que les données sont correctement transmises sans pertes grâce aux numéros d’ACK et aux retransmissions.
2. **Transmission ordonnée** :
   - Les segments arrivent dans l’ordre correct, essentiel pour les requêtes/réponses HTTP.
3. **Contrôle de flux et congestion** :
   - TCP ajuste dynamiquement la vitesse de transmission selon la capacité du récepteur et la congestion réseau.

Pour ces raisons, TCP est préféré dans des cas comme les communications HTTPS où la fiabilité et l'ordre des données sont critiques.

---

### **Conclusion**
Les modifications ont enrichi le rapport en répondant aux attentes :  
- Une explication claire de la relation **SEQ-ACK-segment size**.
- Une justification détaillée du choix de TCP comme protocole de transport.

