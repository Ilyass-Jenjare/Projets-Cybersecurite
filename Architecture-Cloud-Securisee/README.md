# Architecture Infonuagique et Sécurité Périmétrique (Microsoft Azure)

**Rôle :** Architecte Cloud et Ingénieur en Sécurité
**Technologies :** Microsoft Azure (WAF, DDoS Protection, Conditional Access), WordPress, IAM

## 1. Contexte du Déploiement
Ce projet documente la conception, le déploiement et la sécurisation complète d'une infrastructure web infonuagique. L'objectif était de moderniser un service web existant (basé sur le CMS WordPress) en le migrant vers le cloud Microsoft Azure, tout en garantissant une haute disponibilité et une résilience totale face aux cyberattaques modernes (Top 10 OWASP, attaques volumétriques, compromission d'identifiants). La stratégie adoptée repose sur le principe de la défense en profondeur (Defense in Depth).

## 2. Hébergement et Infrastructure Initiale
L'application cible, un système de gestion de contenu (WordPress), a été déployée de manière isolée sur l'infrastructure Azure. 
* **Déploiement :** Configuration de l'environnement d'hébergement Azure (App Service / Virtual Machines) optimisé pour les performances de traitement PHP et la gestion de la base de données.
* **Architecture réseau :** Isolation de l'environnement au sein d'un réseau virtuel (VNet) dédié, limitant l'exposition directe des serveurs backend à Internet.

## 3. Gestion des Identités et des Accès (IAM)
Pour contrer les attaques par force brute et le "credential stuffing", la surface d'attaque liée à l'authentification a été drastiquement réduite.

### 3.1. Contrôle d'accès granulaire (RBAC)
* Création et gestion de comptes utilisateurs non-administrateurs selon le principe du moindre privilège.
* Attribution d'attributs personnalisés stricts aux comptes (ex: segmentation par département et rôle métier) pour cloisonner l'accès aux données.

### 3.2. Authentification Multifacteur (MFA)
* Configuration et obligation d'utilisation de l'authentification à double facteur (2FA/MFA) pour l'intégralité des comptes utilisateurs, y compris les comptes à faibles privilèges.
* Validation technique des flux de connexion pour s'assurer qu'aucun contournement du MFA n'est possible via des API ou des points de terminaison alternatifs.

### 3.3. Accès Conditionnel et Géoblocage
* Définition de stratégies d'accès conditionnel basées sur la localisation géographique (Geo-blocking).
* **Règle stricte :** Autorisation exclusive des connexions entrantes (authentification et administration) provenant d'adresses IP localisées au Canada.
* **Validation :** Tests de pénétration simulant des requêtes depuis des régions étrangères (via VPN/Proxies) pour valider le blocage systématique (Drop) des tentatives d'accès non autorisées.

## 4. Sécurité Périmétrique et Atténuation des Menaces
La protection frontale de l'application a été assurée par l'intégration des services de sécurité natifs d'Azure pour filtrer le trafic malveillant avant même qu'il n'atteigne le serveur web.

### 4.1. Pare-feu Applicatif Web (WAF)
* Déploiement d'un Azure Web Application Firewall (intégré via Azure Application Gateway / Front Door).
* **Configuration :** Activation de l'ensemble de règles de base (Core Rule Set) pour bloquer automatiquement les attaques d'injection SQL (SQLi), les scripts intersites (XSS) et les inclusions de fichiers locaux/distants (LFI/RFI).
* **Règles personnalisées :** Création de règles de filtrage spécifiques pour bloquer certains User-Agents malveillants, limiter la taille des requêtes et interdire l'accès à des chemins sensibles (URI). Validation de l'efficacité des règles par l'analyse des journaux de blocage.

### 4.2. Protection contre les attaques DDoS
* Intégration de la protection Azure DDoS pour prémunir l'infrastructure contre les attaques par déni de service distribué (attaques volumétriques et protocolaires).
* **Tests de résilience :** Réalisation de tests de montée en charge contrôlée (Load Testing) pour observer le comportement de l'infrastructure et valider l'activation des métriques d'atténuation DDoS en temps réel.

## 5. Durcissement Applicatif (Hardening) et Dispositifs Anti-Robots
Outre la protection périmétrique, l'application WordPress elle-même a subi un processus d'obfuscation et de durcissement.

### 5.1. Minimisation de l'empreinte numérique (Information Hiding)
* Suppression et modification de tous les éléments d'affichage par défaut (bannières de version WordPress, thèmes par défaut, fichiers `readme.html`).
* Masquage et renommage des pages de connexion critiques (ex: modification de l'URL par défaut `wp-admin` et `wp-login.php`) pour complexifier la phase de reconnaissance (reconnaissance automatisée) des attaquants.

### 5.2. Dispositifs Anti-Automatisation
* Intégration de mécanismes CAPTCHA robustes sur tous les formulaires exposés publiquement (authentification, commentaires, réinitialisation de mot de passe).
* Objectif : Empêcher l'utilisation abusive de l'application par des robots (bots) pour le spam, le scraping de données ou l'épuisement des ressources.

## 6. Conclusion
Ce déploiement démontre la capacité à architecturer une solution infonuagique complète où la sécurité est intégrée à chaque couche du modèle OSI. La combinaison du durcissement de l'application (Hardening), du contrôle d'accès strict (MFA, Geo-blocking) et de la protection périmétrique avancée (WAF, DDoS Protection) permet de garantir la disponibilité du service tout en protégeant efficacement l'intégrité et la confidentialité des données hébergées.