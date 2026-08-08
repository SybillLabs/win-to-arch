<h1 align="center">
    🛠️ Installation et configuration : Reflector
</h1>

## `> ./reflector-installation.sh`

- **Installation de reflector**
```bash
sudo pacman -S reflector
```

- **Génération d'une liste de miroirs optimisée**
```bash
sudo reflector --country France,Germany,Netherlands --age 12 --protocol https --sort rate --download-timeout 15 --save /etc/pacman.d/mirrorlist
```
> **Explication des options :**
> - `--country France,Germany,Netherlands` : sélectionne les miroirs situés en France, Allemagne ou Pays-Bas, zones réputées pour leur stabilité et leur proximité réseau.
> - `--age 12` : ignore les miroirs non synchronisés depuis plus de 12 heures.
> - `--protocol https` : ne garde que les connexions chiffrées.
> - `--sort rate` : classe les miroirs du plus rapide au plus lent, selon un test de téléchargement réel.
> - `--download-timeout 15` : limite le temps d'attente pour chaque miroir à 15 secondes.  
*Testé avec la valeur par défaut (5 secondes) : timeout massif sur la quasi-totalité des miroirs en environnement virtualisé. 15 secondes résout le problème sans échec significatif.*
> - `--save /etc/pacman.d/mirrorlist` : écrit la liste directement dans le fichier utilisé par `pacman`.

> **Remarque :** si le test de vitesse échoue pour un miroir (timeout), il est conservé en fin de liste plutôt qu'exclu, reflector le classe simplement en dernier plutôt que de le retirer.

- **Vérification du résultat**
```bash
cat /etc/pacman.d/mirrorlist
```
> **Résultat attendu :** une liste de miroirs situés en France, Allemagne ou Pays-Bas, triés du plus rapide au plus lent.

- **Test concret : forcer une synchronisation de la base de paquets pour valider que les miroirs répondent bien**
```bash
sudo pacman -Syy
```
> **Résultat attendu :** rafraîchissement complet des index de paquets, sans erreur de connexion.

## `> ./reflector-maintenance.sh`

Les miroirs évoluent dans le temps : un miroir rapide aujourd'hui peut devenir lent ou se désynchroniser demain. Sur une rolling release, un miroir en retard peut faire échouer un `FullUpdate` en cours de route.

**Choix retenu : rafraîchissement manuel, pas de timer systemd automatisé.** Cohérent avec la philosophie de maintenance active déjà appliquée à `FullUpdate` (voir `maintenance.md`) : contrôle et vérification à chaque étape, plutôt qu'une automatisation silencieuse en tâche de fond.

- **Alias dédié**
```bash
nano ~/.bashrc
```
Ajout à la fin du document :
```bash
alias UpdateMirrors='sudo reflector --country France,Germany,Netherlands --age 12 --protocol https --sort rate --download-timeout 15 --save /etc/pacman.d/mirrorlist'
```
Application de la modification :
```bash
source ~/.bashrc
```
Vérification que l'alias est bien reconnu :
```bash
type UpdateMirrors
```

- **Routine hebdomadaire**, un jour où la machine n'est pas utilisée activement, pour éviter toute interruption de téléchargement pendant le test de vitesse des miroirs :
```bash
UpdateMirrors
FullUpdate
```
> **Remarque :** l'ordre est important. `UpdateMirrors` doit toujours précéder `FullUpdate`, une mise à jour lancée sur une liste de miroirs non rafraîchie retombe sur le risque qu'on cherche justement à éviter.

- **Désactivation du timer reflector par défaut**, pour éviter qu'il tourne en parallèle avec une configuration différente de celle validée manuellement ci-dessus :
```bash
systemctl is-enabled reflector.timer
```
> Si la commande retourne `enabled` :
```bash
sudo systemctl disable --now reflector.timer
```
> **Remarque :** le fichier `/etc/xdg/reflector/reflector.conf` piloté par ce timer utilise par défaut `--sort age` (tri par fraîcheur, pas par vitesse) et `--latest 5` (limite à 5 miroirs sans tenir compte de la vitesse), une configuration différente et moins pertinente que celle testée et retenue ci-dessus. Désactiver le timer plutôt que de le reconfigurer, pour garder une seule source de vérité : l'alias `UpdateMirrors`.

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