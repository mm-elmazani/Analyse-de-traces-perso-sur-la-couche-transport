# Analyse TCP : Rapport d'analyse réseau

Ce rapport documente l'analyse d'une conversation TCP capturée avec Wireshark. La capture montre l'établissement, le transfert sécurisé et la fermeture d'une connexion TCP. 

## **Introduction**
La conversation analysée utilise le port source **62164** sur la machine locale (**192.168.1.2**) pour se connecter au port destination **443** (HTTPS) de l’adresse **152.199.20.80**. Le protocole TLSv1.2 est utilisé au-dessus de TCP pour sécuriser la communication.

## **1. Ouverture de la connexion : Three-Way Handshake**

Le processus de **three-way handshake** est illustré par les trois premières trames :

### **Trame 1107 (SYN)**
- **Source :** 192.168.1.2  
- **Destination :** 152.199.20.80  
- **Flag TCP :** SYN  
- **Numéro de séquence (SEQ) :** 0  
- **Taille de fenêtre :** 65535 octets  
- **Options TCP :**  
  - **MSS=1460** (Maximum Segment Size)  
  - **SACK_PERM** (Selective Acknowledgment Permitted)

### **Trame 1108 (SYN-ACK)**
- **Source :** 152.199.20.80  
- **Destination :** 192.168.1.2  
- **Flag TCP :** SYN-ACK  
- **Numéro de séquence (SEQ) :** 0  
- **Numéro d'acquittement (ACK) :** 1

### **Trame 1109 (ACK)**
- **Source :** 192.168.1.2  
- **Destination :** 152.199.20.80  
- **Flag TCP :** ACK  
- **Numéro de séquence (SEQ) :** 1  
- **Numéro d'acquittement (ACK) :** 1

L'ouverture de la connexion est maintenant complète, et les deux parties peuvent échanger des données.

---

## **2. Transmission de données avec TLS**

Le protocole utilisé au-dessus de TCP est **TLSv1.2**. Voici les étapes importantes du handshake TLS :

### **Trame 1110 : Client Hello**
- **Description :** Le client initie le handshake TLS.  
- **Informations clés :**
  - SNI (Server Name Indication) : `ops-gx.nvidia.com`  
  - Algorithmes de chiffrement supportés.

### **Trame 1122 : Server Hello**
- **Description :** Le serveur répond avec les paramètres TLS (choix des algorithmes, certificat).  
- **Certificat :** Certificat valide émis par DigiCert.

### **Échange de clés et données sécurisées**
- Les trames suivantes montrent :
  - Échange des clés pour sécuriser la connexion.  
  - Envoi de données chiffrées sous forme d'Application Data.

Le transfert de données se poursuit avec plusieurs trames TLSv1.2 contenant des données chiffrées. Ces données sont sécurisées et ne peuvent pas être analysées sans les clés de déchiffrement.

---

## **3. Fermeture de connexion : Four-Way Termination**

La fermeture de la connexion est propre, comme le montrent les trames suivantes :

### **Trame 1141 (FIN, ACK)**
- **Source :** 192.168.1.2  
- **Destination :** 152.199.20.80  
- **Flag TCP :** FIN, ACK  
- **Numéro de séquence (SEQ) :** 557  
- **Numéro d'acquittement (ACK) :** 4998

### **Trame 1142 (ACK)**
- **Source :** 152.199.20.80  
- **Destination :** 192.168.1.2  
- **Flag TCP :** ACK  
- **Numéro d'acquittement (ACK) :** 557  

### **Trame 1142 (FIN, ACK)**
- **Source :** 152.199.20.80  
- **Destination :** 192.168.1.2  
- **Flag TCP :** FIN, ACK  
- **Numéro de séquence (SEQ) :** 4998  
- **Numéro d'acquittement (ACK) :** 557  

### **Trame 1143 (ACK)**
- **Source :** 192.168.1.2  
- **Destination :** 152.199.20.80  
- **Flag TCP :** ACK  
- **Numéro d'acquittement (ACK) :** 4999  

---

## **4. Analyse des numéros de séquence et d'acquittement (SEQ et ACK)**

Les numéros de séquence (`SEQ`) et d'acquittement (`ACK`) évoluent comme suit :
- Chaque `SEQ` indique le premier octet d'un segment.  
- Chaque `ACK` confirme la réception de tous les octets précédents.  

---

## **5. Analyse des options TCP et de la taille de fenêtre**

- **Options TCP :**
  - **MSS=1460** : Taille maximale des segments TCP, adaptée au MTU Ethernet (1500).  
  - **SACK_PERM** : Permet les retransmissions sélectives en cas de perte de paquet.

- **Taille de fenêtre :**
  - Initialement, la taille est élevée (**65535 octets**), indiquant une bonne capacité de traitement des segments.

---

## **6. Protocole utilisé au-dessus de TCP**

Le protocole utilisé au-dessus de TCP est **TLSv1.2**, un protocole sécurisé pour les communications HTTPS. Ce choix garantit :
- La confidentialité des données.  
- L'intégrité des communications.  

Le serveur utilise un certificat SSL/TLS émis par DigiCert pour s'authentifier auprès du client.

---

## **Conclusion**

Cette analyse montre une conversation TCP complète :
- **Ouverture de connexion :** Réalisée via le three-way handshake.  
- **Transfert sécurisé :** Réalisé via TLSv1.2, protégeant les données échangées.  
- **Fermeture de connexion :** Effectuée proprement via le four-way termination.  

Les options TCP et l'évolution des numéros de séquence et d'acquittement respectent les spécifications du protocole TCP.

---

## **Screenshots**
1. **Three-Way Handshake :** Trames 1107, 1108, et 1109.  
2. **Handshake TLS :** Trames 1110 (Client Hello) et 1122 (Server Hello).  
3. **Fermeture de connexion :** Trames 1141 à 1143.


