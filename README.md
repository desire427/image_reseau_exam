# Mise en place d'une solution intelligente pour la gestion des services réseaux

## Résumé

Ce rapport présente la solution mise en place pour la gestion des services réseaux (DNS, DHCP, FTP, SSH) ainsi que l'automatisation du déploiement du service de messagerie. Le document détaille les objectifs, l'architecture, les choix techniques, les étapes de configuration, la stratégie d'automatisation et des captures d'écran attestant du fonctionnement.

## Objectifs

- Assurer une gestion centralisée et fiable desservices réseau critiques : **DNS**, **DHCP**, **FTP**, **SSH**.
- Automatiser le déploiement, la configuration et la mise à jour du service de messagerie pour garantir reproductibilité et rapidité.
- Fournir une documentation claire et des artefacts (playbooks, scripts, captures) pour faciliter la maintenance et la montée en charge.

## Périmètre

- Infrastructure ciblée : serveurs Linux (distribution utilisée pour le projet).
- Services : DNS (résolution locale et zones), DHCP (attribution d'adresses), FTP (transferts de fichiers), SSH (administration sécurisée).
- Automatisation : scripts et playbooks (ex : Ansible) pour le déploiement et la configuration du service de messagerie.

## Environnement et outils

- Système d'exploitation : Linux
- Services logiciels (exemples courants utilisés pour ce type de projet) :
  - DNS : BIND9
  - DHCP : ISC DHCP Server
  - FTP : vsftpd ou ProFTPD
  - SSH : OpenSSH
  - Messagerie : Postfix (MTA) + Dovecot (IMAP/POP3) ou solution équivalente
  - Automatisation : Ansible (playbooks), scripts Bash
  - Supervision / logs : journalctl, fichiers de log spécifiques aux services

## Architecture technique

Décrire ici l'architecture déployée :

- Topologie réseau : serveurs physiques/virtuels séparés par rôles (contrôleur DNS/DHCP, serveur applicatif, serveur mail).
- Sécurité réseau : règles de pare-feu (iptables/nftables), restriction d'accès SSH par clés, usage de VLAN si pertinent.
- Haute disponibilité et sauvegardes : sauvegarde des configurations et des zones DNS, snapshots VM.

## Détails de mise en œuvre par service

### DNS

- Rôle : résolution des noms internes, gestion des zones directes et inverses.
- Points de configuration essentiels :
  - Fichiers de zone (/etc/bind/zones/*.db) avec enregistrements A, PTR, MX
  - Fichier de configuration principal (/etc/bind/named.conf / named.conf.options)
  - Sécurisation : ACL, vues si nécessaire, transfert d'annonces vers serveurs secondaires

#### Illustrations — DNS

![Figure DNS 1 - configuration des zones](Capture d’écran du 2026-02-17 11-41-45.png)
*Figure 1 — Extrait du fichier de zone et enregistrements A/MX utilisés pour la résolution locale.*

![Figure DNS 2 - named.conf](Capture d’écran du 2026-02-17 11-42-59.png)
*Figure 2 — Configuration principale de BIND (named.conf / options).*

![Figure DNS 3 - état du service bind9](Capture d’écran du 2026-02-17 11-51-54.png)
*Figure 3 — Vérification du service DNS via systemctl et logs.*

### DHCP

- Rôle : attribution dynamique d'adresses IP et distribution d'options réseau (gateway, DNS).
- Points de configuration :
  - Déclarations de sous-réseaux et pools d'adresses
  - Baux statiques pour équipements critiques (MAC -> IP)
  - Options réseau (routers, domain-name-servers, domain-name)

#### Illustrations — DHCP

![Figure DHCP 1 - configuration dhcpd.conf](Capture d’écran du 2026-03-01 10-57-52.png)
*Figure 4 — Extrait de la configuration DHCP : sous-réseaux et pools.*

![Figure DHCP 2 - baux et tests d'obtention d'adresse](Capture d’écran du 2026-03-01 10-59-28.png)
*Figure 5 — Liste des baux DHCP et test client montrant l'attribution d'une adresse.*

![Figure DHCP 3 - service DHCP actif](Capture d’écran du 2026-03-01 10-52-15.png)
*Figure 6 — État du service DHCP et logs d'initialisation.*

### FTP

- Rôle : échanger des fichiers entre utilisateurs/serveurs sur un canal contrôlé.
- Points de configuration :
  - Chroot des utilisateurs lorsque nécessaire
  - Gestion des permissions et des comptes système
  - Sécurisation : usage de FTPS si requis (certificats TLS)

#### Illustrations — FTP

![Figure FTP 1 - configuration vsftpd](Capture d’écran du 2026-03-01 11-24-37.png)
*Figure 7 — Fichier de configuration du serveur FTP (extraits : chroot, utilisateurs).* 

![Figure FTP 2 - création d'un utilisateur FTP](Capture d’écran du 2026-03-01 11-18-03.png)
*Figure 8 — Commandes et sorties pour la création et le test d'un compte FTP.*

![Figure FTP 3 - test de transfert](Capture d’écran du 2026-03-01 11-21-25.png)
*Figure 9 — Résultat d'un transfert de fichier FTP (PUT/GET).* 

### SSH

- Rôle : administration à distance sécurisée.
- Bonnes pratiques appliquées :
  - Authentification par clés publiques (désactivation de l'authentification par mot de passe pour les comptes admins)
  - Restrictions d'accès par groupe et par adresse IP
  - Journalisation et configuration du timeout

#### Illustrations — SSH

![Figure SSH 1 - clés publiques et configuration SSHD](Capture d’écran du 2026-03-01 11-32-14.png)
*Figure 10 — Déploiement des clés publics et paramètres essentiels dans /etc/ssh/sshd_config.*

![Figure SSH 2 - connexions autorisées et audit](Capture d’écran du 2026-03-01 11-32-26.png)
*Figure 11 — Journalisation des connexions SSH et contrôle d'accès.*

![Figure SSH 3 - session SSH administrateur](Capture d’écran du 2026-03-01 11-32-37.png)
*Figure 12 — Session SSH d'un administrateur montrant les actions de maintenance.*

## Automatisation du déploiement du service de messagerie

L'automatisation permet de déployer de manière reproductible le service de messagerie (installation, configuration, certificats, utilisateurs, tests). Une approche recommandée :

- Outil : Ansible (playbooks idempotents)
- Étapes du playbook :
  1. Installer les paquets requis (Postfix, Dovecot, packages de gestion TLS).
  2. Déployer les fichiers de configuration templatisés (ex : Jinja2) pour Postfix et Dovecot.
  3. Déployer les certificats TLS (Let's Encrypt ou certificats internes).
  4. Créer/mettre à jour les comptes utilisateurs et bases de données si besoin.
  5. Démarrer et vérifier les services (systemctl status, tests de connexion SMTP/IMAP).
  6. Ajouter des vérifications automatisées (test d'envoi local, vérification des logs pour erreurs critiques).

#### Illustrations — Automatisation & Messagerie

![Figure Mail 1 - playbook Ansible exemple](Capture d’écran du 2026-03-01 12-14-50.png)
*Figure 13 — Extrait de playbook Ansible pour l'installation et la configuration de Postfix/Dovecot.*

![Figure Mail 2 - déploiement TLS et certificats](Capture d’écran du 2026-03-01 12-33-15.png)
*Figure 14 — Déploiement des certificats TLS et vérification avec openssl.*

![Figure Mail 3 - tests SMTP/IMAP](Capture d’écran du 2026-03-01 12-12-03.png)
*Figure 15 — Tests d'envoi/réception et connexions IMAP pour valider le service de messagerie.*

### Stratégie de rollback

- Versionner les fichiers de configuration (git) et conserver des sauvegardes avant toute modification.
- Playbooks séparés pour rollback (restauration de fichiers, redémarrage de services).

## Tests et validation

- Tests fonctionnels : résolution DNS, acquisition d'adresse DHCP, connexion FTP, connexion SSH.
- Tests du service de messagerie : envoi/réception locaux, vérification des en-têtes, authentification IMAP/POP3.
- Vérification des logs et du bon démarrage des services.

#### Illustrations — Tests et validation

![Figure Test 1 - vérifications systemctl](Capture d’écran du 2026-03-01 12-43-34.png)
*Figure 16 — Statut global des services après déploiement (systemctl).* 

![Figure Test 2 - logs et résultats de test](Capture d’écran du 2026-03-01 13-01-02.png)
*Figure 17 — Exemples de logs et sorties de tests automatisés (envoi/connexion).* 

## Résultats et captures d'écran

Les captures d'écran réalisées lors du projet sont incluses en annexe et illustrent :

- État des services (systemctl / status)
- Fichiers de configuration ouverts dans un éditeur
- Tests de connexion et sorties de commandes
- Interfaces d'administration, si présentes

### Annexes — Captures d'écran

Les images suivantes ont été rassemblées dans le répertoire du projet et sont jointes à ce rapport :

1. Capture d’écran du 2026-02-17 11-41-45.png
2. Capture d’écran du 2026-03-01 11-32-14.png
3. Capture d’écran du 2026-03-01 11-36-38.png
4. Capture d’écran du 2026-03-01 11-28-36.png
5. Capture d’écran du 2026-03-01 10-57-52.png
6. Capture d’écran du 2026-03-01 12-14-50.png
7. Capture d’écran du 2026-03-01 12-33-15.png
8. Capture d’écran du 2026-03-01 12-12-03.png
9. Capture d’écran du 2026-03-01 10-59-28.png
10. Capture d’écran du 2026-03-01 12-43-34.png
11. Capture d’écran du 2026-03-01 13-01-02.png
12. Capture d’écran du 2026-03-01 12-24-04.png
13. Capture d’écran du 2026-03-01 11-31-59.png
14. Capture d’écran du 2026-03-01 10-52-15.png
15. Capture d’écran du 2026-03-01 11-29-16.png
16. Capture d’écran du 2026-03-01 11-18-03.png
17. Capture d’écran du 2026-03-01 11-24-37.png
18. Capture d’écran du 2026-03-01 11-00-16.png
19. Capture d’écran du 2026-03-01 11-21-25.png
20. Capture d’écran du 2026-03-01 11-27-07.png
21. Capture d’écran du 2026-03-01 11-32-26.png
22. Capture d’écran du 2026-03-01 11-09-40.png
23. Capture d’écran du 2026-03-01 12-03-37.png
24. Capture d’écran du 2026-03-01 11-29-53.png
25. Capture d’écran du 2026-03-01 11-19-08.png
26. Capture d’écran du 2026-03-01 12-59-32.png
27. Capture d’écran du 2026-02-17 11-42-59.png
28. Capture d’écran du 2026-03-01 12-15-38.png
29. Capture d’écran du 2026-02-17 11-51-54.png
30. Capture d’écran du 2026-03-01 11-36-28.png
31. Capture d’écran du 2026-03-01 11-32-37.png
32. Capture d’écran du 2026-03-01 10-55-17.png
33. Capture d’écran du 2026-03-01 12-59-04.png
34. Capture d’écran du 2026-03-01 11-30-58.png
35. Capture d’écran du 2026-03-01 10-56-58.png
36. Capture d’écran du 2026-02-24 19-53-50.png
37. Capture d’écran du 2026-03-01 13-01-13.png
38. Capture d’écran du 2026-03-01 11-14-08.png
39. Capture d’écran du 2026-03-01 10-53-36.png
40. Capture d’écran du 2026-03-01 11-26-04.png
41. Capture d’écran du 2026-03-01 10-56-03.png
42. Capture d’écran du 2026-03-01 12-32-25.png
43. Capture d’écran du 2026-03-01 12-19-49.png
44. Capture d’écran du 2026-03-01 11-22-07.png
45. Capture d’écran du 2026-03-01 12-24-12.png
46. Capture d’écran du 2026-03-01 12-44-13.png
47. Capture d’écran du 2026-03-01 11-25-31.png

> Remarque : les légendes précises des figures peuvent être ajustées selon le contenu exact de chaque capture d'écran. Si vous souhaitez, je peux incruster chaque image directement dans le rapport avec une légende détaillée (ex. "Figure 1 — État du service DNS : ...").

## Conclusion

La solution mise en place répond aux objectifs de gestion centralisée des services réseau et propose une méthode d'automatisation reproductible pour le déploiement du service de messagerie. Les étapes décrites et les artefacts joints (captures, scripts, playbooks) permettent d'assurer maintenance, audit et évolutivité.

---

Si vous le souhaitez, je peux :

- Insérer chaque capture d'écran à l'endroit approprié dans le document avec une légende détaillée.
- Générer un playbook Ansible d'exemple pour le déploiement du service de messagerie.
- Produire un sommaire technique additionnel (extraits de configuration) à partir de vos fichiers de configuration réels.
