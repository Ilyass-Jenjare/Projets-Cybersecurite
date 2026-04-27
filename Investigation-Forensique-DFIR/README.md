# Déploiement EDR et Investigation Forensique (DFIR) avec Velociraptor et Sysmon

**Rôle :** Analyste SOC / Spécialiste en Réponse aux Incidents (DFIR)

## 1. Contexte du Mandat
Ce projet documente la mise en place d'une architecture de télémétrie avancée et l'exécution d'un cycle complet de réponse aux incidents. L'objectif était de déployer une solution de détection et de réponse pour les points finaux (EDR), de simuler une compromission système par un attaquant, puis de mener une investigation forensique approfondie (Threat Hunting) pour retracer la chaîne d'attaque (Kill Chain).

## 2. Phase 1 : Architecture et Déploiement de la Télémétrie
Pour assurer une visibilité totale sur les actions système, une architecture client-serveur a été mise en place :
* **Serveur de Gestion (Hunt Manager) :** Déploiement de Velociraptor sur un serveur Ubuntu Linux, génération des certificats TLS et configuration de l'interface d'administration.
* **Agent Client (Endpoint) :** Génération et déploiement de l'agent Velociraptor sur un poste Windows 11.
* **Enrichissement des journaux (Sysmon) :** Installation de Microsoft Sysmon avec une configuration avancée (basée sur Sysmon Modular) pour capturer les événements critiques (Création de processus avec lignes de commande - Event ID 1, chargement de DLL, connexions réseau).

## 3. Phase 2 : Émulation de l'Adversaire (Vecteurs d'Attaque)
Une attaque automatisée a été simulée sur le poste client avec des privilèges élevés pour évaluer les capacités de détection de l'EDR. Les actions hostiles comprenaient :
* **Évasion de défense :** Désactivation de la protection en temps réel de Windows Defender via PowerShell (`Set-MpPreference -DisableRealtimeMonitoring $true`).
* **Vol de données :** Création et dissimulation d'un fichier contenant des identifiants exfiltrés (`credentials.txt` dans `%APPDATA%`).
* **Simulation de Command & Control (C2) :** Téléchargement de fichiers malveillants simulés (`update.txt` déposé dans les dossiers temporaires).
* **Anti-Forensique :** Suppression de fichiers système ciblés (`notes.txt`) pour tester la capacité de récupération et d'analyse des traces résiduelles.

## 4. Phase 3 : Investigation et Threat Hunting
L'analyse post-incident a été réalisée exclusivement à distance depuis le serveur Velociraptor, en utilisant des artefacts pré-intégrés et des requêtes sur mesure.

### 4.1. Analyse des processus et de l'évasion
* Utilisation de l'artefact `Elastic.EventLogs.Sysmon` pour interroger les journaux d'événements à distance.
* **Découverte :** Identification de l'exécution de la commande de désactivation de l'antivirus grâce à l'application d'une expression régulière (`Set-MpPreference.*DisableRealtimeMonitoring`) sur la colonne `process.command_line`.

### 4.2. Recherche d'artefacts malveillants (FileFinder)
* Utilisation de l'artefact `windows.search.FileFinder` pour balayer le système de fichiers du client.
* **Découverte :** Localisation exacte des scripts d'attaque initiaux (`eval2.bat`, scripts `.ps1`) et du fichier d'exfiltration `credentials.txt` malgré leur dissimulation dans l'arborescence.

### 4.3. Analyse de l'Anti-Forensique (USN Journal)
* L'attaquant ayant supprimé le fichier `notes.txt` pour effacer ses traces, une recherche standard s'est révélée infructueuse (comportement attendu).
* **Remédiation :** Utilisation de l'artefact `Windows.Forensics.Usn` pour interroger le journal de mise à jour (USN Journal) du système de fichiers NTFS, permettant de prouver l'existence passée des fichiers supprimés et du payload `update.txt` (via le filtre `(?i)^update\.txt$`).

### 4.4. Requêtes VQL (Velociraptor Query Language)
Pour approfondir la collecte de données volatiles, des requêtes VQL personnalisées ont été développées et poussées vers l'agent :
* **Tri du système :** Requête basique d'énumération (`SELECT Hostname, Fqdn, OS... FROM info()`) pour valider l'empreinte du système compromis.
* **Analyse de la Timeline :** Création d'une requête ciblant spécifiquement les répertoires temporaires des utilisateurs pour lister les 50 derniers fichiers modifiés, identifiant ainsi le moment exact du dépôt de la charge utile :
  `SELECT FullPath, Size, Mtime FROM glob(globs="C:\Users\*\AppData\Local\Temp\**") WHERE NOT IsDir ORDER BY Mtime DESC LIMIT 50`
* **Interaction directe :** Exécution de commandes interactives (PowerShell/CMD) depuis le serveur vers le client pour tester la connectivité réseau résiduelle (ping) et navigation dans les ruches de la base de registre pour vérifier les clés de persistance.

## 5. Conclusion
Le déploiement combiné de Velociraptor et Sysmon a permis d'obtenir une visibilité granulaire (Endpoint Observability) indispensable lors d'un incident de sécurité. L'investigation a démontré la capacité à retracer les actions d'un attaquant tentant de brouiller les pistes, en s'appuyant sur l'analyse bas niveau du système de fichiers (MFT/USN) et l'interrogation en temps réel via VQL.