# Rapport d'Audit de Sécurité et Optimisation d'Architecture

**Rôle :** Auditeur en Sécurité Logicielle
**Contexte :** Mandat d'accompagnement pour MediQuebec Solutions (MQS)

## 1. Mise en situation et Objectifs
Ce projet porte sur l'audit complet de la plateforme de télémédecine « MaSantéConnectée ». Suite à un incident de sécurité ayant exposé les données de plus de 3 000 patients via une API de test, ce mandat visait à restructurer la posture de sécurité de l'entreprise pour assurer la conformité à la Loi 25 et préparer une introduction en bourse (IPO).

## 2. Analyse et Stratégie de Sécurité
L'audit a couvert l'ensemble de l'écosystème technique, incluant les systèmes legacy (Java/Oracle), les microservices (Node.js/Python) et les applications mobiles (React Native).

### 2.1. Classification des Données et Conformité
Une matrice de classification a été élaborée pour 8 types de données critiques (données biométriques, numéros RAMQ, dossiers médicaux, informations financières, etc.). 
* **Objectif :** Garantir la confidentialité et l'intégrité selon les exigences de la Loi 25 et de la PIPEDA.
* **Impact :** Définition des niveaux de protection requis pour chaque actif informationnel.

### 2.2. Modélisation des Menaces (STRIDE)
Application de la méthodologie STRIDE sur le flux critique de la consultation vidéo entre patient et médecin.
* **Menaces identifiées :** Usurpation d'identité (Spoofing), altération des prescriptions (Tampering) et divulgation d'informations sensibles.
* **Contre-mesures :** Mise en place d'une authentification mutuelle et chiffrement de bout en bout.

### 2.3. Architecture Cible
Proposition d'une transition d'une architecture monolithique vers une architecture de microservices sécurisée :
* **Composants clés :** Implémentation d'une API Gateway pour centraliser les politiques de sécurité et d'un Service Mesh pour sécuriser les communications inter-services (mTLS).
* **Bénéfices :** Segmentation réseau, visibilité accrue et réduction de la surface d'attaque.

## 3. Gestion des Vulnérabilités (Analyse Technique)
L'audit a permis d'identifier et de documenter 10 vulnérabilités critiques avec un plan de remédiation priorisé par score CVSSv3.1.

### Points saillants des vulnérabilités traitées :
* **Injection SQL (Score 9.8) :** Identification d'une faille dans le module de recherche patient permettant l'exfiltration de la base de données. *Remédiation : Utilisation de requêtes paramétrées.*
* **Dépendances Obsolètes - Log4Shell (Score 10.0) :** Présence de bibliothèques vulnérables permettant l'exécution de code à distance (RCE). *Remédiation : Mise à jour immédiate et intégration de scans SCA.*
* **Stockage JWT Non Sécurisé (Score 7.5) :** Jeton d'authentification stocké en clair sur mobile. *Remédiation : Utilisation d'Android Keystore / iOS Keychain.*
* **Contrôle d'Accès Brisé - IDOR (Score 8.1) :** Possibilité d'accéder aux documents d'autres patients par incrémentation d'ID. *Remédiation : Validation systématique de l'autorisation côté serveur.*
* **Mauvaises configurations Cloud :** Conteneurs Azure Blob Storage accessibles publiquement. *Remédiation : Désactivation de l'accès anonyme et utilisation de Managed Identities.*

## 4. Gestion des Risques et Feuille de Route
Établissement d'un registre de risques majeurs incluant :
* Évaluation de la probabilité et de l'impact.
* Définition du risque résiduel après mesures d'atténuation.
* Planification stratégique (Roadmap) sur 18 mois pour atteindre un niveau de maturité de sécurité requis pour l'IPO.

## 5. Conclusion
L'audit a démontré que la sécurité doit être intégrée dès la conception (Security by Design). La mise en œuvre des recommandations techniques et organisationnelles permet non seulement de protéger les renseignements personnels des patients, mais aussi de limiter les risques juridiques et financiers liés aux nouvelles réglementations québécoises.