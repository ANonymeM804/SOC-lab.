# Scénario 01  –  connexion echouée

## Objectif
Detecter le signalement d'un echec d'ouverture de session d'un compte

## Action réalisée
- J'ai redemarrer ma machine virtuel
- ensuite j'ai essayé d'entrer un mauvais mot de passe
- Voir dans screenshots/Testdeconnexionechec.png

## Événements attendus
Event ID = 4625
 
## Timeline
22/10/2026 20:23:48 

## Requête / filtre utilisé
Winlogon.exe ou service.exe

Nom du processus: C:\Windows\system32\svchost.exe


