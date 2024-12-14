# Analyse TCP : Rapport d'analyse réseau

Ce rapport documente l'analyse d'une conversation TCP capturée avec Wireshark. La capture montre l'établissement, le transfert sécurisé et la fermeture d'une connexion TCP.

---

## **Introduction**

La conversation analysée utilise le port source **62164** sur la machine locale (**192.168.1.2**) pour se connecter au port destination **443** (HTTPS) de l’adresse **152.199.20.80**. Le protocole **TLSv1.2** est utilisé au-dessus de TCP pour sécuriser la communication.

---

## **1. Ouverture de la connexion : Three-Way Handshake**

Le processus de **three-way handshake** est réalisé en trois étapes :

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
- **Ouverture de connexion :** Réalisée via le three-way handshake.  
- **Transfert sécurisé :** Réalisé via TLSv1.2, protégeant les données échangées.  
- **Transition TLS :** La session sécurisée est terminée avec un message d’alerte TLS (`Close Notify`).  
- **Fermeture de connexion :** Effectuée proprement via le four-way termination.
