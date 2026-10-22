# Scénario 01  –  connexion echouée

## Objectif
Detecter le signalement d'un echec d'ouverture de session d'un compte

## Action réalisée
- j'ai essayé d'entrer le bon mot de passe
- Voir dans screenshots/Testdeconnexionreussimdp.png

## Événements attendus
Event ID = 4624

## Timeline
22/10/2026 20:24:00

## Requête / filtre utilisé
Winlogon.exe ou service.exe

Nom du processus: C:\Windows\system32\svchost.exe

