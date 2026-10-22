# Scénario 02 – Test dans powershell

## Objectif
Sysmon detectant tous les processus lancés dans powershell depuis son demarrage

## Action réalisée
1- J'ai lancé powershell
2- J'ai executé une commande ping vers une adresse IP
3- J'ai executé la commande whoami
4- J'ai fait le test de dection reseau avec utilisant Invoke-WebRequest sur un site web

## Événements attendus
Event ID : 1,3,11,22


(+ captures dans screenshots/

demarragedepowershell.png
pingreseaudetecter.png
executionwhoamidanspowershell.png
connectionreseaudetect.png)


## Timeline
1- 22/10/2026 20:34:07 
2- 22/10/2026 20:41:42
3- 22/10/2026 20:42:11
4- 22/10/2026 21:15:46


## Requête / filtre utilisé
Voir Screenshots

## Conclusion
Les journaux de windows peuvent seulement voir le lancement de powershell mais pas les autres processus lancés dans powershell, 
Sysmon ici nous permet de voir en detail tout ce qui a été faite dans powershell
