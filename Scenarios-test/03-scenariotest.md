# Scénario 03 – Gestion des comptes et création de fichier

## Objectif
Comprendre quelles traces laissent la création, la modification et la suppression de comptes utilisateurs locaux (journal Security),
 puis la création d'un fichier (Sysmon 11), et apprendre à relier ces événements entre eux pour reconstituer l'activité d'un même opérateur.

## Action réalisée
Toutes les actions ont été faites le 23/10/2026, avec le compte `batoc` (session administrateur) :

1. Création du compte local `testuser` à 12:31:51
2. Ajout de `testuser` au groupe local **Administrateurs** (12:31:51)
3. Création du compte local `testsupp` à 12:33:54
4. Suppression du compte `testsupp` à 12:37:55
5. Création du fichier `scenariotest.ps1` dans `C:\Users\batoc\Download\` à 12:54:18

## Événements attendus

| Action                                                                                 | Sour     | Event ID |
|--                      -                                                               |---       |- --      |
| Création d'un compte                                                                   | Security | 4720     |
| Ajout à un groupe local                                                                | Security | 4732     |
| Suppression d'un compte                                                                | Security | 4726     |
| Processus ayant exécuté la commande (si ligne de commande : `net.exe` puis `net1.exe`) | Sysmon   | 1        |
| Création du fichier `.ps1`                                                             | Sysmon   | 11       |

(+ captures dans screenshots/

creationuser.png
creationuser2.png
ajoutgrpadmin.png
suppressionuser2.png
creationfichier.png)

## Événements retrouvés

| Heure    | Source   | Event ID | Ce que l'événement dit                                                              | Capture              |
|---       |---       |---       |---                                                                                  |---                   |
| 12:31:51 | Security | 4720     | Compte `testuser` créé par `batoc`                                                  | creationuser.png     |
| 12:31:51 | Security | 4732     | Membre ajouté au groupe `Administrateurs` (le compte est identifié par son **SID**) | ajoutgrpadmin.png    |
| 12:33:54 | Security | 4720     | Compte `testsupp` créé par `batoc`                                                  | creationuser2.png    |
| 12:37:55 | Security | 4726     | Compte `testsupp` supprimé par `batoc`                                              | suppressionuser2.png |
| 12:54:18 | Sysmon   | 11       | Fichier `C:\Users\batoc\Download\scenariotest.ps1` créé                             | creationfichier.png  |

### Identifiants des comptes (à relever dans les événements)

| Compte     | SID             | Créé (4720) | Autre événement                        |
|---         |---              |---          |---                                     |
| `testuser` | S-1-5-21-…-10xx | 12:31:51    | Ajout au groupe Administrateurs (4732) |
| `testsupp` | S-1-5-21-…-10xx | 12:33:54    | Suppression (4726) à 12:37:55          |



## Corrélation : Action → Événement → Contexte

**1. Qui a fait l'action ? (Security 4720, 4732, 4726)**
Dans chacun de ces événements, le bloc *Subject* indique le compte qui a **agi** (`batoc`) et le bloc *Target* le compte **concerné** (`testuser`, `testsupp`).


**2. Retrouver la session qui a tout déclenché (`SubjectLogonId`)**
Le champ `SubjectLogonId` des 4720, 4732 et 4726 permet de remonter au 4624 (la connexion) de la session de `batoc`. 
Le même identifiant existe dans le champ `LogonId` des événements Sysmon 1. 
Si la valeur est identique partout, toutes les actions viennent **d'une seule session**, donc d'un seul opérateur. 

**3. Comprendre comment l'action a été faite (Sysmon 1)**
Security dit **ce qui** s'est passé. Sysmon 1 dit **comment** : si les comptes ont été créés en ligne de commande, 
on y voit `net.exe` puis `net1.exe` (lancé en interne par `net.exe`), avec la commande complète et le processus parent
. l'interface graphique a été utilisée, seul `mmc.exe` apparaît, sans détail : le 4720 reste alors la seule source fiable.

**4. Relier le fichier au processus qui l'a créé (Sysmon 11 → Sysmon 1)**
Le champ `Image` du Sysmon 11 donne le processus qui a écrit `scenariotest.ps1`. Son `ProcessGuid` correspond à celui du Sysmon 1 de ce processus,
 ce qui permet de retrouver sa ligne de commande et son parent.

**5. Lire l'enchaînement comme un analyste**
Dans l'ordre : création d'un compte, création d'un second compte,
               élévation de privilèges du premier, suppression du second, 
               puis dépôt d'un script. Chaque événement pris seul est banal. 


## Requête / filtre utilisé

Event Viewer, journal **Sécurité** (Filtrer le journal actuel > XML) :

```xml
<QueryList><Query Id="0" Path="Security"><Select Path="Security">
  *[System[(EventID=4720 or EventID=4732 or EventID=4726)]]
</Select></Query></QueryList>
```

Event Viewer, journal **Sysmon**, filtrer sur l'ID 11 puis rechercher `scenariotest`.

PowerShell, pour afficher les comptes dans la plage horaire du scénario :

```powershell
Get-WinEvent -FilterHashtable @{
  LogName='Security'; Id=4720,4732,4726
  StartTime=[datetime]'2026-10-23 12:30'; EndTime=[datetime]'2026-10-23 13:00'
} | Select-Object TimeCreated, Id, Message | Format-List
```

Voir aussi les screenshots.


## Conclusion
Les événements de gestion de comptes viennent du journal **Security** et non de Sysmon : Sysmon surveille l'activité système, pas l'authentification ni les comptes.
 Pris isolément, chaque événement est ordinaire ; c'est leur **corrélation** (SID, `SubjectLogonId`, `ProcessGuid`, ordre et rythme des actions) qui reconstitue l'histoire.
 Security dit **ce qui** s'est passé et **qui** l'a fait, Sysmon dit **comment**. 
Cette logique Action → Événement → Contexte sera la base de la centralisation des logs et de la détection dans le projet 2.
