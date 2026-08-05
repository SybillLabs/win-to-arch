<h1 align="center">
    🛠️ Guide d'installation : SSHD
</h1>

## `> ./sshd-installation.sh`

- **Vérification de la présence d'OpenSSH (client et serveur)**
```bash
pacman -Qs openssh
```
> **Résultat attendu :** une ligne `local/openssh <version>` doit apparaître. Si la commande ne retourne rien, le paquet n'est pas installé — passer à l'étape suivante. S'il est déjà présent, l'installation peut être ignorée.

- **Installation d'OpenSSH (si absent)**
```bash
sudo pacman -S openssh
```
> **Remarque :** un seul paquet fournit à la fois le client (`ssh`) et le serveur (`sshd`).

- **Vérification du statut du service sshd**
```bash
sudo systemctl status sshd
```
> **Résultat attendu :** si `inactive (dead)`, passer à l'étape suivante. Si déjà `active (running)`, cette étape peut être ignorée.

- **Activation et démarrage automatique du service sshd**
```bash
sudo systemctl enable --now sshd
```

- **Vérification finale**
```bash
sudo systemctl status sshd
ssh -V
```
> **Résultat attendu :** statut `active (running)`, et la commande `ssh -V` affiche la version du client OpenSSH installé.

## `> ./sshd-hardening.sh`

> **Prérequis impératif :** générer et tester une connexion par clé **avant** de désactiver l'authentification par mot de passe. Inverser cet ordre bloque tout accès SSH à la machine.

- **Génération d'une paire de clés SSH (sur la VM)**
```bash
ssh-keygen -t ed25519 -C "$USER@$(hostname)"
```
> **Remarque :** `ed25519` est l'algorithme recommandé actuellement (plus court et plus robuste que RSA à taille de clé équivalente). Laisser le chemin par défaut (`~/.ssh/id_ed25519`) et définir une passphrase lors de la génération.

- **Ajout de la clé publique aux clés autorisées**
```bash
cat ~/.ssh/id_ed25519.pub >> ~/.ssh/authorized_keys
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```
> **Remarque :** les permissions strictes sur `~/.ssh` et `authorized_keys` sont obligatoires — `sshd` refuse silencieusement l'authentification par clé si les droits sont trop permissifs.

- **Test de connexion par clé, avant toute modification de `sshd_config`**
```bash
ssh localhost
```
> **Résultat attendu :** connexion réussie sans demande de mot de passe (uniquement la passphrase de la clé, si définie). Ne pas continuer tant que ce test n'est pas concluant.

- **Modification de la configuration SSH**
```bash
sudo nano /etc/ssh/sshd_config
```
```ini
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
```

- **Redémarrage du service pour appliquer la configuration**
```bash
sudo systemctl restart sshd
```

- **Vérification post-durcissement**
```bash
ssh localhost
```
> **Résultat attendu :** connexion toujours fonctionnelle par clé. Dans un second terminal (sans fermer la session active), tenter une connexion avec un utilisateur/mot de passe pour confirmer le refus :
```bash
ssh -o PubkeyAuthentication=no localhost
```
> **Résultat attendu :** la connexion doit être refusée — confirmation que `PasswordAuthentication no` est bien appliqué. Conserver la session active du premier terminal ouverte jusqu'à validation complète, au cas où un rollback de `sshd_config` serait nécessaire.

---

<p align="center">  
    <i>➡️ Back to :<a href="/install_guide.md"> Guide d'installation</a></i>
</p>

<p align="center">  
    <i>➡️ Back to :<a href="/README.md"> README</a></i>
</p>

---

<p align="center">
  <img src="https://capsule-render.vercel.app/api?type=waving&color=0:1a1a2e,100:0d0d0d&fontColor=FF003C&fontSize=50&height=100&width=900&text=%5BEOF%5D&section=footer"/>
</p>