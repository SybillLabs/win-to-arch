<p align="center">
    <img src="https://capsule-render.vercel.app/api?type=blur&height=300&color=&color=FF003C&text=WINDOWS%20TO%20ARCH-LINUX&fontColor=FF003C&textBg=false&stroke=121212&strokeWidth=5&fontAlign=50&fontAlignY=50&fontSize=50&reversal=false" alt="HEADER" />
</p>

### `> ./tech_stack.sh`
<p align="left">
    <img src="https://img.shields.io/badge/VIRTUALBOX-white?style=for-the-badge&logo=virtualbox&color=%232F61B4"/>
    <img src="https://img.shields.io/badge/ARCH%20LINUX-white?style=for-the-badge&logo=archlinux&logoColor=white&color=%231793D1"/>
    <img src="https://img.shields.io/badge/ENDEAVOUROS-white?style=for-the-badge&logo=endeavouros&logoColor=white&color=%237F7FFF"/>
    <img src="https://img.shields.io/badge/BASH-white?style=for-the-badge&logo=gnubash&logoColor=white&color=%234EAA25"/>
    <img src="https://img.shields.io/badge/DOCKER-white?style=for-the-badge&logo=docker&logoColor=white&color=%232496ED"/>
    <img src="https://img.shields.io/badge/K3S-white?style=for-the-badge&logo=k3s&logoColor=black&color=%23FFC61C"/>
</p>
<p align="left">
    <img src="https://img.shields.io/badge/BTRFS-white?style=for-the-badge&color=%23E63946"/>
    <img src="https://img.shields.io/badge/UFW-white?style=for-the-badge&color=%23792EE5"/>
    <img src="https://img.shields.io/badge/SSH-white?style=for-the-badge&logo=openssh&logoColor=white&color=%23000000"/>
    <img src="https://img.shields.io/badge/FAIL2BAN-white?style=for-the-badge&color=%23F94144"/>
    <img src="https://img.shields.io/badge/SECURITY%20HARDENING-white?style=for-the-badge&color=%23000000"/>
    <img src="https://img.shields.io/badge/SYSTEMD-white?style=for-the-badge&logo=systemd&logoColor=white&color=%23001428"/>
</p>

## `> ./specifications.sh`

### 📓 Cahier des charges
- Migration d'un poste Windows 11 vers une distribution Arch-based.  
- Contrainte non négociable : liberté totale de configuration, aucune distribution basée `Debian/Ubuntu`, jugées trop rigides malgré leur stabilité reconnue.  
- Nécessité d'une maintenance active du système, car la distribution retenue est une rolling release.
- Cohabitation sur le même hôte physique de la conteneurisation (DevOps) et de la virtualisation (Admin Sys/Réseaux) pour différents usages.
- Personnalisation de l'environnement de bureau et autres éléments à la charge de l'utilisateur.

### 🔎 Méthodologie de travail
- **Projets en deux phases** : d'abord sur une machine virtuelle, pour tester et valider chaque installation et configuration sans risque, puis sur le poste physique une fois que la VM a validé l'ensemble des étapes.
- **[Journal d'installation et de configuration](/install-log.md)** complet, étape par étape, inclut un chapitre dédié aux différences rencontrées entre la VM et le poste physique (hostname, Secure Boot, partitionnement, réseau).

**Suivi d'avancement** — une étape n'est cochée qu'une fois testée et validée en conditions réelles, pas seulement documentée :

| Étape                    | VM | PC |
|--------------------------|----|----|
| EndeavourOS | ✅ | ⬜ |
| Terminal (Kitty) | ✅ | ⬜ |
| Navigateur (Google Chrome + OperaGX) | ✅ | ⬜ |
| Mirrors (Reflector) | ✅ | ⬜ |
| ZRAM (zram-generator) | ✅ | ⬜ |
| Maintenance (FullUpdate + CheckDKMS + Clean) | ⬜ | ⬜ |
| Snapshots (Timeshift + Snapper + Grub-BTRFS) | ⬜ | ⬜ |
| SSH Installation | ⬜ | ⬜ |
| Firewall (UFW + Fail2Ban) | ⬜ | ⬜ |
| Docker | ⬜ | ⬜ |
| Kubernetes (K3S) | ⬜ | ⬜ |
| VS Code | ⬜ | ⬜ |
| VirtualBox | ⬜ | ⬜ |
| Nmap | ⬜ | ⬜ |
| Wireshark | ⬜ | ⬜ |
| Git | ⬜ | ⬜ |

> **⚠️ Aucun passage à la phase PC sans validation intégrale de la colonne VM.**

### 🖥️ Application et logiciel retenue
| Logiciel | Source | Remarque |
|---|---|---|
| VS Code | AUR (visual-studio-code-bin) | — |
| Docker | dépôts officiels (docker, docker-compose) | activer et ajouter l'utilisateur au groupe docker |
| Kubernetes | dépôts officiels + AUR | — |
| K3S | AUR (k3s-bin) | - |
| VirtualBox | dépôts officiels (virtualbox, virtualbox-host-dkms) | risque de rupture DKMS après mise à jour noyau — surveiller après maintenance |
| Wireshark | dépôts officiels (wireshark-qt) | ajouter l'utilisateur au groupe wireshark pour capture sans root |
| Nmap | dépôts officiels (nmap) | — |
| SSH | dépôts officiels (openssh) | — |
| Git | dépôts officiels (git) | — |
| Firefox | dépôt officiels (firefox) | — |

## `> ./architecture.sh`

### 📁 Structure du dépôt

```
win-to-arch/
├── README.md                       # ce document
├── install-log.md                  # journal d'installation et de configuration, étape par étape
└── configuration/
    ├── terminal-config.md          # Kitty + Starship + fastfetch
    ├── maintenance.md              # alias FullUpdate / CheckDKMS / Clean
    ├── mirrors.md                  # reflector, sélection des miroirs pacman
    ├── snapshots.md                # timeshift + snapper + grub-btrfs
    ├── zram.md                     # zram-generator, gestion mémoire
    ├── firewall.md                 # ufw, fail2ban
    ├── ssh-hardening.md            # durcissement sshd_config, clé SSH
    └── docker-k3s.md               # cohabitation Docker/k3s, k3s désactivé au boot
└── assets/                         # captures d'écran, logos, images...
```

> J'ai fait le choix de le faire sans script d'automatisation parce que EndeavourOS est une rolling release, et que la maintenance active est un prérequis. L'automatisation complète d'une rolling release est un non-sens : il faut pouvoir intervenir manuellement sur les mises à jour, vérifier les modules DKMS, vérifier les snapshots, etc.

### 🛠️ Décisions d'architecture

Chaque choix ci-dessous a été tranché avec un compromis explicite. Rien n'est laissé à l'implicite.
- **Secure Boot → désactivé.** Compromis accepté : perte de la vérification de signature au boot (surface théorique pour rootkit bootkit-level) contre suppression de la contrainte de signature MOK sur les modules DKMS à chaque update noyau. Retenu car le scénario d'attaque écarté suppose une compromission root déjà acquise.
- **sudo → retenu face à doas.** doas documenté comme alternative valide (surface de code ~10x réduite, config minimaliste) mais écarté : pas de gain justifié pour un usage personnel hors contexte haute sécurité, au prix d'une rupture de compatibilité avec l'écosystème DevOps standard.
- **k3s → désactivé au démarrage.** Cohabitation avec Docker sur le même hôte : risque de chevauchement de plages IP entre le bridge Docker et le CNI de k3s. Activation manuelle à la demande, jamais en tâche de fond.
- **VirtualBox → aucune virtualisation imbriquée, à aucune phase.** En phase VM, VirtualBox est installé et son module DKMS vérifié (`CheckDKMS` = `installed`), mais aucune VM n'est démarrée à l'intérieur de la VM EndeavourOS — l'imbrication est écartée pour des raisons de fiabilité et d'isolation, sans bénéfice réel puisque ce n'est jamais l'usage final. Les labs VirtualBox (AD/DNS/DHCP/mail) ne démarrent qu'en phase 2, sous EndeavourOS en hôte direct sur la machine physique.

---

<p align="center">  
    <i>↪️ Back to the hub :<a href="https://github.com/SybillLabs/sysnet-practice-labs.git"> sysnet-practice-labs</a></i> | <i>📍 From <a href="https://github.com/SybillLabs">SybillLabs</a></i>
</p>

---

<p align="center">
  <img src="https://capsule-render.vercel.app/api?type=waving&color=0:1a1a2e,100:0d0d0d&fontColor=FF003C&fontSize=50&height=100&width=900&text=%5BEOF%5D&section=footer"/>
</p>