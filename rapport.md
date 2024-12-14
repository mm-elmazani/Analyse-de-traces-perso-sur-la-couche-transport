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




# **Analyse UDP : Rapport d'analyse réseau**

Ce rapport documente l'analyse d'une conversation UDP capturée avec Wireshark. La capture montre des échanges DNS, protocole utilisé pour la résolution de noms de domaine, via UDP sur le port 53.

---

## **Introduction**

Le **Domain Name System (DNS)** est un protocole essentiel qui permet la traduction de noms de domaine (ex. : `www.google.com`) en adresses IP (ex. : `192.168.1.1`) ou la récupération d’autres informations associées à ces domaines. DNS utilise principalement **UDP sur le port 53**, mais peut basculer sur **TCP** en cas de besoins spécifiques (ex. : réponses volumineuses).

Dans cette analyse, nous étudions plusieurs requêtes et réponses DNS, couvrant différents scénarios :  
- Requêtes réussies avec enregistrements **A**, **AAAA**, **CNAME**, et **SOA**.  
- Cas d’échecs avec le code de réponse **No such name**.  

---

## **1. Analyse d'une requête DNS**

### **Exemple analysé : Trame 1103**

![Trame 1103 - Requête DNS](https://github.com/user-attachments/assets/e2b7932d-2066-4e84-8bc3-5b1179502ffc)

**Contexte de la trame :**  
Une requête DNS de type **AAAA** est envoyée par le client pour obtenir l’adresse IPv6 associée au domaine `ops.gx.nvidia.com`.

#### **Détails UDP**
- **Source :** `fe80::f02b:a5e6:a096:ac2e` (adresse IPv6 du client).  
- **Destination :** `fe80::46d4:54ff:fef6:ble3` (serveur DNS).  
- **Port source :** Dynamique (attribué par le client).  
- **Port destination :** 53 (standard DNS).  
- **Longueur UDP :** 97 octets.  
- **Checksum :** Calculé pour vérifier l’intégrité.

#### **Détails DNS**
- **Transaction ID :** `0x4200` (identifiant unique pour associer la requête à la réponse correspondante).  
- **Flags DNS :**  
  - `0x0100` : Requête standard, récursion non demandée.  
- **Questions DNS :**  
  - Domaine : `ops.gx.nvidia.com`.  
  - Type d’enregistrement : **AAAA** (IPv6).

---

## **2. Analyse d'une réponse DNS**

### **Exemple analysé : Trame 1104**

![Trame 1104 - Réponse DNS](https://github.com/user-attachments/assets/1ffd73ac-7003-4e25-b062-9d79ec40974e)

**Contexte de la trame :**  
Le serveur DNS répond à la requête pour `ops.gx.nvidia.com` avec des enregistrements **CNAME**.

#### **Détails UDP**
- **Source :** `fe80::46d4:54ff:fef6:ble3` (serveur DNS).  
- **Destination :** `fe80::f02b:a5e6:a096:ac2e` (client).  
- **Port source :** 53.  
- **Port destination :** Dynamique.  
- **Longueur UDP :** 179 octets.  

#### **Détails DNS**
- **Transaction ID :** `0x4200` (correspond à la requête précédente).  
- **Flags DNS :**  
  - `0x8180` : Réponse standard, récursion disponible.  
- **Réponses DNS :**  
  - **CNAME :** `ops.gx.nvidia.com` → alias de `cs1137.wpc.ea55a.phicdn.net`.  
  - **CNAME final :** `cs1137261584.wpc.phicdn.net`.  

---

## **3. Analyse des requêtes et réponses échouées**

### **Exemple analysé : Trames 1827 à 1832**

![Trames 1827 à 1832 - DNS échoué](https://github.com/user-attachments/assets/ce506fee-86f7-4efd-9fcd-33a82cdbb0ce)

**Contexte des trames :**  
Le client tente de résoudre le domaine `wpad.home` via des requêtes DNS, mais reçoit des réponses indiquant qu’aucune correspondance n’existe.

#### **Détails des requêtes**
- Domaine demandé : `wpad.home`.  
- Types d’enregistrement :  
  - **A** (IPv4).  
  - **AAAA** (IPv6).  

#### **Détails des réponses**
- **Code de réponse :** **No such name** (nom de domaine inexistant).  
- **Interprétation :**  
  Le serveur DNS confirme qu’il ne peut pas résoudre ce domaine.

---

## **4. Exemple de résolution réussie avec plusieurs adresses**

### **Exemple analysé : Trames 2909 (requête) et 2917 (réponse)**

![Trames 2909 et 2917 - Résolution réussie](https://github.com/user-attachments/assets/1fe2d0c6-7d2c-47c2-8f21-15df10104405)

**Contexte des trames :**  
Une requête est envoyée pour le domaine `prod.otel.kaizen.nvidia.com`. La réponse contient plusieurs enregistrements.

#### **Réponses DNS**
- **Domaine :** `prod.otel.kaizen.nvidia.com`.  
- **Types d’enregistrements retournés :**  
  - **A (IPv4)** :  
    - Adresses : `3.71.226.131`, `63.176.90.252`, et `63.176.236.234`.  
  - **SOA (Start of Authority)** :  
    - Informations sur le serveur DNS principal.

---

## **5. Relation entre DNS et UDP**

### **Pourquoi UDP est utilisé pour DNS ?**
1. **Rapidité :** UDP est sans connexion, permettant des requêtes et réponses rapides sans la surcharge d’une session TCP.  
2. **Efficacité :** La majorité des échanges DNS sont courts (une question, une réponse), ce qui ne nécessite pas les mécanismes de fiabilité de TCP.  
3. **Fallback vers TCP :**  
   Si une réponse dépasse la taille limite d’un datagramme UDP (512 octets dans certains cas), DNS utilise TCP pour garantir la livraison.

### **Avantages de l’utilisation d’UDP pour DNS :**
- Optimise la rapidité pour les résolutions courantes.  
- Réduit la surcharge réseau par rapport à TCP.  

---

## **Conclusion**

Cette analyse met en lumière le rôle fondamental de DNS dans la résolution de noms de domaine. L’utilisation d’**UDP sur le port 53** permet des échanges rapides et adaptés à la majorité des cas.

Cependant, le protocole DNS reste flexible, capable de basculer sur **TCP** pour gérer des cas exceptionnels tels que des réponses volumineuses. Les exemples analysés montrent également la diversité des enregistrements retournés (**A**, **AAAA**, **CNAME**, **SOA**), ainsi que la gestion des échecs dans les résolutions.

[lien vers la capture générale](https://github.com/mm-elmazani/Analyse-de-traces-perso-sur-la-couche-transport/blob/main/Screenshots%20UDP/conv%20g%C3%A9r%C3%A9nal.png)
