<h1 align="center">
    🛠️ Installation et configuration : Docker & K3S
</h1>

## `> ./docker-installation.sh`

- **Installation de Docker sur Arch Linux**
```bash
sudo pacman -Syu docker
```
- **Activation et démarrage automatique du service Docker**
```bash
sudo systemctl enable --now docker
```
- **Ajout de l'utilisateur au groupe Docker pour exécuter les commandes sans sudo**
```bash
sudo usermod -aG docker $USER
```
> **Remarque :** Après avoir ajouté l'utilisateur au groupe Docker, il est nécessaire de se déconnecter et de se reconnecter pour que les changements prennent effet.
- **Vérification de l'installation de Docker**
```bash
groups $USER            # pour s'assurer que l'utilisateur est bien dans le groupe docker
docker --version
docker run hello-world
```
- **Installation de docker-compose**
```bash
sudo pacman -Syu docker-compose
```
- **Vérification de l'installation de docker-compose**
```bash
docker-compose --version
```

## `> ./k3s-installation.sh`

- **Installation de K3S sur Arch Linux (AUR, absent des dépôts officiels)**
```bash
yay -S k3s-bin
```
- **Installation de kubectl (dépôts officiels)**
```bash
sudo pacman -Syu kubectl
```
- **Activation du service K3S**
```bash
sudo systemctl enable --now k3s
```
> **Remarque :** Laisser quelques secondes à K3S pour terminer son initialisation (génération des certificats et du kubeconfig interne) avant de passer à l'étape suivante. Vérifiable via `sudo systemctl status k3s` — attendre `Active: active (running)` avant de continuer.
- **Configuration de kubectl pour cibler le cluster K3S**
```bash
mkdir -p ~/.kube
sudo k3s kubectl config view --raw | tee ~/.kube/config
chmod 600 ~/.kube/config
```
> **Remarque :** Ne pas utiliser `sudo` devant `mkdir` — cela créerait le dossier `~/.kube` avec les droits root, empêchant ensuite l'écriture du fichier de configuration par l'utilisateur courant.
- **Vérification de l'installation et du bon fonctionnement de K3S**
```bash
k3s --version
kubectl get nodes
```
> **Résultat attendu :** le nœud local doit apparaître avec le statut `Ready`. Si le kubeconfig généré à l'étape précédente contient des champs `null` (`clusters: null`, `contexts: null`), c'est le signe que K3S n'avait pas terminé son démarrage — relancer la commande de configuration après avoir confirmé le statut `active (running)`.
- **Désactivation du service K3S au démarrage pour éviter tout conflit réseau avec Docker**
```bash
sudo systemctl disable k3s
```
> **Remarque :** K3S reste installé mais inactif par défaut. Activation manuelle ponctuelle à la demande, au moment des labs concernés :
```bash
sudo systemctl start k3s
```

### ⚠️ Sécurité — kubeconfig

Le fichier `~/.kube/config` généré à l'étape de configuration contient un certificat client et une **clé privée** (`client-key-data`), encodés en Base64 — pas chiffrés. N'importe qui en possession de ce fichier peut le décoder instantanément et s'authentifier auprès du cluster comme s'il était l'utilisateur légitime.

**Ne jamais coller le contenu de ce fichier en clair** : dans une conversation, un ticket, une issue GitHub, un README, ou tout autre support versionné ou partagé — même dans le cadre d'un lab local non exposé. C'est un réflexe à construire dès l'apprentissage, avant de manipuler un cluster réel où l'exposition aurait un impact concret. Sur ce lab précis (`server: https://127.0.0.1:6443`, cluster local non exposé au réseau), le risque immédiat est nul, mais l'habitude prise maintenant est celle qui compte en production.

Si un extrait doit être partagé pour du debug, ne montrer que la structure (présence ou non de `clusters:`, `contexts:`, `current-context:`) en supprimant ou masquant tout champ se terminant par `-data`.

## `> ./docker-k3s-cohabitation.sh`

- **Démarrer k3s manuellement si inactif**
```bash
sudo systemctl status k3s
sudo systemctl start k3s
```
- **Lister les interfaces réseaux créées par Docker et K3S**
```bash
ip addr show
```

- **Isoler spécifiquement les bridges Docker et Flannel (K3S)**
```bash
ip addr show docker0
ip addr show cni0
```

> **Vérification :** les plages d'adresses IP affichées pour `docker0` (Docker, par défaut `172.17.0.0/16`) et `cni0` (K3S/Flannel, par défaut `10.42.0.0/24`) ne doivent pas se chevaucher.
>
> **Pourquoi `cni0` et pas `flannel.1` :** Flannel crée deux interfaces distinctes. `cni0` est le bridge local qui attribue une IP à chaque pod **sur ce nœud** et gère le trafic pod-à-pod local — c'est l'équivalent direct de `docker0` côté Docker, donc la seule comparaison pertinente pour détecter un chevauchement d'adressage. `flannel.1` est une interface VXLAN dédiée au trafic **inter-nœuds** (encapsulation des paquets entre pods situés sur des machines différentes) ; sur un cluster mono-nœud comme celui de ce lab, elle existe par défaut mais ne route rien de concret, puisqu'il n'y a pas de second nœud vers lequel communiquer. Elle n'entre donc pas dans le périmètre du test de non-conflit.

- **Test fonctionnel réel — un conteneur Docker et un pod k3s doivent tous deux avoir accès réseau sortant**
```bash
docker run --rm alpine ping -c 3 8.8.8.8

kubectl run test-pod --image=alpine --restart=Never -- ping -c 3 8.8.8.8
kubectl logs test-pod
kubectl delete pod test-pod
```
> **Résultat attendu :** les deux commandes `ping` doivent réussir (3 paquets transmis et reçus dans les logs), confirmant que chaque environnement dispose d'un accès réseau sortant fonctionnel et isolé.

- **Nettoyage — repasser k3s en inactif après le test, cohérent avec la désactivation au boot déjà en place**
```bash
sudo systemctl disable k3s
sudo systemctl stop k3s
sudo systemctl status k3s
```

> **Remarque :** `systemctl stop k3s` n'interrompt pas toujours les processus `containerd-shim` associés aux pods actifs au moment de l'arrêt — un comportement connu du service. Si `ps aux | grep containerd-shim` affiche encore des process après l'arrêt, les terminer manuellement avec `sudo pkill -9 containerd-shim`.

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