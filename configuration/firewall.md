<h1 align="center">
    🛠️ Installation et configuration : Firewall (UFW & Fail2ban)
</h1>

## `> ./ufw-installation.sh`

- **Installation d'ufw sur Arch Linux**
```bash
sudo pacman -S ufw
```

- **Activation et démarrage automatique du service ufw**
```bash
sudo systemctl enable --now ufw
```

- **Définition de la politique par défaut**
```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```
> **Remarque :** Toute connexion **entrante non sollicitée** (initiée depuis l'extérieur vers cette machine) est bloquée par défaut, sauf exceptions explicites ci-dessous. Le trafic sortant reste libre, et les réponses aux connexions initiées depuis cette machine (navigation web, requêtes DNS, etc.) sont automatiquement autorisées par le suivi d'état de connexion d'ufw, aucune restriction pour l'usage Internet courant.

- **Ouverture des ports nécessaires**
```bash
sudo ufw allow 22/tcp
sudo ufw allow from 127.0.0.1 to any port 6443
```
> **Remarque :** Le port 22 (SSH) est ouvert ici au niveau filtrage réseau — le durcissement de la configuration SSH elle-même (authentification par clé uniquement, désactivation du login root) est traité séparément dans `ssh-hardening.md`. Le port 6443 (API K3S) est restreint à `127.0.0.1`, cohérent avec un cluster mono-nœud non exposé au réseau.
>
> **Qu'est-ce que le port 6443 :** c'est le port par défaut du **kube-apiserver**, le composant central de Kubernetes/K3S qui reçoit toutes les commandes (`kubectl get nodes`, création de pods, etc.) et communique avec les autres composants du cluster (kubelet, scheduler, controller-manager). C'est via ce port que le fichier `~/.kube/config` généré précédemment se connecte au cluster (`server: https://127.0.0.1:6443`). Le restreindre à `127.0.0.1` signifie que seule la machine elle-même peut y accéder — aucune machine distante sur le LAN ne peut piloter ce cluster K3S, cohérent avec un usage local de lab, pas un cluster destiné à être administré à distance.

- **Activation des règles de filtrage**
```bash
sudo ufw enable
```
> **Remarque :** `systemctl enable --now ufw` active le *service*, `ufw enable` active les *règles de filtrage* elles-mêmes — les deux commandes sont nécessaires et distinctes.

- **Vérification de la configuration**
```bash
sudo ufw status verbose
```
> **Résultat attendu :** statut `active`, politique par défaut `deny (incoming), allow (outgoing)`, et les règles `22/tcp ALLOW` ainsi que `6443 ALLOW from 127.0.0.1` listées.

### ⚠️ Point de vigilance — Docker et ufw

Docker écrit ses propres règles directement dans iptables au démarrage, indépendamment d'ufw, et ces règles sont appliquées **avant** celles d'ufw dans la chaîne de traitement du noyau. Concrètement : un conteneur lancé avec un port publié (`docker run -p 8080:80 ...`) devient accessible depuis le réseau même si ufw n'a jamais autorisé ce port et affiche une politique `deny incoming`, Docker contourne le filtrage sans erreur ni avertissement visible.

- **Téléchargement et installation d'ufw-docker**
```bash
sudo wget -O /usr/local/bin/ufw-docker https://github.com/chaifeng/ufw-docker/raw/master/ufw-docker
sudo chmod +x /usr/local/bin/ufw-docker
```

- **Application de la configuration**
```bash
sudo ufw-docker install
sudo systemctl restart ufw
```
> **Remarque :** cette étape met en place l'infrastructure de filtrage (chaînes iptables dédiées), mais **n'autorise aucun port automatiquement** — tout reste bloqué par défaut, exactement comme pour ufw lui-même. Chaque conteneur exposé doit être autorisé explicitement, port par port, à l'étape suivante.

- **Autorisation explicite d'un conteneur exposé**
```bash
sudo ufw-docker allow <nom_ou_id_du_conteneur> <port_interne>
```
> **Exemple testé :** pour un conteneur nginx lancé avec `docker run -d -p 8080:80 nginx`, l'autorisation cible le port **interne** du conteneur (80), pas le port publié côté hôte (8080) :
```bash
sudo ufw-docker allow optimistic_sanderson 80
```

- **Vérification**
```bash
sudo ufw-docker status
```
> **Résultat attendu :** affichage du conteneur autorisé avec son port, du type `[ 3] 172.17.0.2 80/tcp ALLOW FWD Anywhere # allow optimistic_sanderson 80/tcp bridge`. Un `status` vide après l'étape `install` seule est normal — aucune règle n'existe tant qu'aucun `allow` n'a été fait.

## `> ./fail2ban-installation.sh`

- **Installation de fail2ban**
```bash
sudo pacman -S fail2ban
```
- **Création du fichier de configuration local**
```bash
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
```
> **Remarque :** Ne jamais éditer `jail.conf` directement, ce fichier est écrasé à chaque mise à jour du paquet. Toute configuration personnalisée doit passer par `jail.local`, qui surcharge `jail.conf` sans être affecté par les mises à jour.

- **Configuration de la surveillance SSH**
> **Prérequis** : cette jail suppose que le service `sshd` est actif et que le port 22 est ouvert dans ufw. Pour la configuration complète de SSH, voir `ssh-hardening.md`.
```bash
sudo nano /etc/fail2ban/jail.local
```
```ini
[sshd]
enabled = true
port = 22
filter = sshd
backend = systemd
maxretry = 5
bantime = 3600
findtime = 600
```
> **Remarque :** EndeavourOS/Arch utilise `journald` par défaut et ne génère pas `/var/log/auth.log` (confirmé par test : fichier absent). `backend = systemd` fait lire les tentatives d'authentification directement depuis `journalctl` plutôt que depuis un fichier de log classique — pas besoin de `logpath` dans ce cas.

- **Activation et démarrage du service**
```bash
sudo systemctl enable --now fail2ban
```

- **Vérification du statut**
```bash
sudo fail2ban-client status sshd
```
> **Résultat attendu :** affichage des compteurs `Currently failed`, `Total failed`, `Currently banned`, `Total banned` — tous à `0` en l'absence de tentative échouée récente.

### 🧪 Test de déclenchement

> **Rappel :** Ce test suppose que l'authentification SSH par mot de passe est encore active. Si le durcissement SSH (`ssh-hardening.md`, `PasswordAuthentication no`) a déjà été appliqué, un mauvais mot de passe ne génère plus de tentative d'échec exploitable par fail2ban — le serveur refuse le mode d'authentification avant même de vérifier un mot de passe. Dans ce cas, déclencher le test avec un nom d'utilisateur invalide ou une clé incorrecte à la place.

- **Simuler des échecs d'authentification SSH volontaires** (depuis une autre machine du LAN, ou en local)
```bash
ssh utilisateur_inexistant@<IP_de_la_VM>
```
> **Répéter l'opération jusqu'à dépasser `maxretry` (5 tentatives dans l'exemple ci-dessus).**

- **Vérifier le bannissement**
```bash
sudo fail2ban-client status sshd
```
> **Résultat attendu :** l'IP source des tentatives apparaît dans `Banned IP list`, et `Currently banned` passe à `1` ou plus.

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