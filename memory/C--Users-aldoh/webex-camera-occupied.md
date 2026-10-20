---
name: webex-camera-occupied
description: "Procédure de dépannage quand Webex affiche \"caméra utilisée par une autre application\" (Logitech C920, Windows 11)"
metadata:
  node_type: memory
  type: project
  originSessionId: 82d0532d-5975-480d-9be2-dfe0efde1079
  modified: 2026-10-20T08:06:35.562Z
---

Incident résolu le 2026-10-20 : Webex (app CiscoCollabHost, v46.9.0.35800) refusait la caméra
Logitech HD Pro Webcam C920 alors que Teams l'utilisait normalement le matin même.

**Cause :** le service Windows FrameServer (« Serveur de trame de la Caméra Windows ») était bloqué
et gardait la caméra. Webex ouvre la caméra via DirectShow (pas Media Foundation) et reçoit
`0x800705AA` (ERROR_NO_SYSTEM_RESOURCES) sur `IMediaControl->Run()`, traduit en `VideoCameraOccupied`.
Aucune application n'apparaissait « en cours » dans le registre CapabilityAccessManager.

**Why:** le message de Webex oriente vers une autre application, alors que le blocage vient du service Windows.

**How to apply — procédure, dans l'ordre :**
1. Voir qui utilise la caméra (PowerShell utilisateur ; `EnCours = True` = la tient maintenant) :
   ```
   $b='HKCU:\Software\Microsoft\Windows\CurrentVersion\CapabilityAccessManager\ConsentStore\webcam'; @(Get-ChildItem $b | ? PSChildName -ne 'NonPackaged') + @(Get-ChildItem "$b\NonPackaged") | % { $p = Get-ItemProperty $_.PSPath; if ($p.LastUsedTimeStart) { [pscustomobject]@{ App = $_.PSChildName -replace '#','\'; Debut = [DateTime]::FromFileTime($p.LastUsedTimeStart); EnCours = ($p.LastUsedTimeStop -eq 0) } } } | Sort-Object Debut -Descending | Format-Table -AutoSize -Wrap
   ```
2. Confirmer dans le journal Webex : `%LOCALAPPDATA%\CiscoSpark\Logs\current_log.txt`, chercher
   `VideoCameraOccupied` ; détail dans `Logs\media\calls\*.log` (`IMediaControl->Run() Failed hr=...`).
3. Quitter Teams et les onglets de visio Edge, quitter puis relancer Webex (icône près de l'horloge → Quitter).
4. **Correctif qui a marché :** PowerShell en administrateur → `Restart-Service FrameServer -Force`.
   Le message « Échec d'arrêt du service » peut apparaître alors que le service s'arrête bien quand même
   (vérifier avec `sc.exe queryex FrameServer` → STOPPED ; il redémarre seul à l'ouverture de la caméra).
   Puis quitter et relancer Webex.
5. Sinon : rebrancher la C920 directement sur l'ordinateur (pas station d'accueil/hub), puis redémarrer le PC.
6. Si seul Webex échoue encore après redémarrage : signaler à l'informatique du canton (`0x800705AA` / `VideoCameraOccupied`).

**Après une session Webex — vérifier que la caméra est libérée (pour Teams ensuite) :**
- Test fiable : ouvrir l'application Caméra de Windows (`start microsoft.windows.camera:`).
  Image visible = caméra libre. Message « utilisée par une autre application » / erreur `0xA00F4243`
  = Webex ne l'a pas rendue → quitter Webex complètement (icône près de l'horloge → Quitter) et refaire le test.
- La commande du registre (point 1) ne suffit pas pour Webex : la session Webex du 2026-10-20 n'y a pas été
  enregistrée (dernière trace Webex : 06.10), sans doute à cause de son accès DirectShow.
- FrameServer à l'état STOPPED (`sc.exe queryex FrameServer`) = aucune application ne passe par lui en ce moment.
- Par précaution : quitter Webex complètement après chaque session avant une réunion Teams.

resmon (onglet Processeur → Descripteurs associés, chercher `046d`) montre surtout `svchost.exe (Camera)`
= FrameServer : il ne permet pas de savoir quelle application est derrière.
