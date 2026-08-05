<h1 align="center">
    🛠️ Guide d'installation : étape par étape
</h1>

## `> ./vm-vs-pc.sh`

Chaque chapitre de ce guide est exécuté à l'identique sur la VM de validation et sur le poste physique : mêmes commandes, même ordre. Certains points, en revanche, diffèrent nécessairement selon l'environnement d'exécution. Ils sont listés ici plutôt que répétés à chaque chapitre concerné.

| Point | VM (validation) | PC (physique) |
|---|---|---|
| Hostname | `sybill_labs-wintoarch` | `cypher_nyx` |
| Secure Boot | Désactivé dans les réglages VirtualBox | Désactivé dans le firmware UEFI réel |
| Partitionnement | Disque virtuel, données non critiques | Disque réel, sauvegarde préalable requise |
| Réseau | Mode Bridged (nécessaire pour les tests de pare-feu/fail2ban depuis une autre machine) | Connexion réseau réelle du poste (Wi-Fi/Ethernet) |
| CPU (Coeurs) | 4 CPU | 10 CPU |
| CPU (Threads) | 0 Threads | 16 Threads |
| RAM | 8 Go | 32 Go |
| Disque | 60 Go virtuel | 500 Go réel |

> Toute divergence rencontrée en dehors de ce tableau, propre à un chapitre précis, est documentée directement dans la section concernée du guide plutôt qu'ici.

## `> ./endeavouros-installation.sh`

ISO récupérée depuis [`endeavouros.com/latest-release`](https://endeavouros.com/latest-release/).

**Environnement de bureau** : 
- KDE Plasma, **retenu** pour sa légèreté et son niveau de personnalisation, le reste du guide reste applicable avec un autre DE supporté par EndeavourOS, seule cette étape en dépend.
- XFCE, installé en parallèle pour tester plusieurs environnements de bureau sur la même machine, et pour disposer d'une alternative légère en cas de problème avec KDE Plasma.

**Choix faits pendant l'installation :**
- Mise à jour des miroirs via `reflector-simple` dès le menu d'installation, avant même de lancer l'installeur, cohérent avec la démarche appliquée ensuite en post-install (`mirrors-installation.sh`).
- Installeur en mode `Online`, pour disposer du choix complet des environnements de bureau.
- Partitionnement : `Effacer le disque`, système de fichiers racine en **Btrfs** (prérequis pour `snapshots-installation.sh`, voir ce chapitre), swap dimensionné à un minimum de 6 Go sans hibernation, identique sur VM et PC.
- Chargeur de démarrage : GRUB.
- Mots de passe root et utilisateur volontairement distincts, le mot de passe utilisateur sert de base à `sudo` (voir `Décisions d'architecture` dans le README).
- Aucune sélection de paquets additionnels à l'installation, le reste de la stack est géré explicitement chapitre par chapitre dans ce guide, pas au moment de l'installeur.

Une fois l'installation terminée : extinction de la VM, retrait de l'ISO, snapshot VirtualBox de l'état post-install avant de poursuivre.

> **Remarque :** Une fois l'installation terminée, il est possible de montrer le **Welcome** avec la commande `eos-welcome --enable` ou la commande `eos-welcome --once`.

## `> ./terminal-installation.sh`

## `> ./navigator-installation.sh`

## `> ./mirrors-installation.sh`

## `> ./zram-installation.sh`

## `> ./maintenance.sh`

## `> ./snapshots-installation.sh`

## `> ./ssh-installation.sh`

## `> ./firewall-installation.sh`

## `> ./docker-k3s-installation.sh`

## `> ./vscode-installation.sh`

## `> ./virtualbox-installation.sh`

## `> ./nmap-installation.sh`

## `> ./wireshark-installation.sh`

## `> ./git-installation.sh`

## `> ./personalization.sh`

---

<p align="center">  
    <i>➡️ Back to :<a href="/README.md"> README</a></i>
</p>

---

<p align="center">
  <img src="https://capsule-render.vercel.app/api?type=waving&color=0:1a1a2e,100:0d0d0d&fontColor=FF003C&fontSize=50&height=100&width=900&text=%5BEOF%5D&section=footer"/>
</p>