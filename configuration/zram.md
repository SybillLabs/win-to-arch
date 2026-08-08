<h1 align="center">
    🛠️ Installation et configuration : ZRAM
</h1>

## `> ./zram-installation.sh`

- **Installation de zram-generator**
```bash
sudo pacman -S zram-generator
```

- **Création du fichier de configuration**
```bash
sudo nano /etc/systemd/zram-generator.conf
```
```conf
[zram0]
zram-size = ram / 2
compression-algorithm = zstd
```
> **Explication des paramètres :**
> - `zram-size = ram / 2` : alloue une zone zram équivalente à la moitié de la RAM physique disponible.
> - `compression-algorithm = zstd` : algorithme de compression retenu pour son bon compromis vitesse/taux de compression, plus rapide que `lzo` à ratio de compression comparable.

- **Activation du service**
```bash
sudo systemctl daemon-reload
sudo systemctl start systemd-zram-setup@zram0.service
```
> **Remarque :** pas besoin d'`enable`, `zram-generator` s'active automatiquement à chaque démarrage via `systemd`, dès lors que le fichier de config existe.

## `> ./zram-verification.sh`
- **Commande**
```bash
zramctl
```
> **Résultat attendu :** une ligne affichant le périphérique `/dev/zram0`, sa taille (proche de la moitié de la RAM), l'algorithme utilisé (`zstd`), et son état `[SWAP]`.

- **Vérification que le système reconnaît bien cet espace comme swap actif**
```bash
free -h
```
> **Résultat attendu :** la ligne `Échange` doit afficher une taille non nulle, additionnant zram et la partition swap classique créée à l'installation.

- **Vérification que les deux sources de swap coexistent correctement**
```bash
swapon --show
```
> **Résultat attendu :** deux lignes distinctes, une pour la partition swap classique (`/dev/sda2` ou équivalent) et une pour `/dev/zram0`, chacune avec sa taille propre.
>
> **Testé et confirmé sur cette VM :**
> ```
> NAME       TYPE      SIZE USED PRIO
> /dev/sda2  partition   8G   0B   -1
> /dev/zram0 partition 3,9G   0B  100
> ```
> `zram0` affiche une priorité (`PRIO 100`) largement supérieure à la partition swap classique (`PRIO -1`). Le noyau privilégie donc systématiquement zram (rapide, en RAM compressée) avant de recourir à la partition sur disque, qui ne sert que de filet de sécurité en dernier recours si zram venait à saturer.

---

<p align="center">  
    <i>➡️ Back to :<a href="/install-log.md"> Journal d'installation</a></i>
</p>

<p align="center">  
    <i>➡️ Back to :<a href="/README.md"> README</a></i>
</p>

---

<p align="center">
  <img src="https://capsule-render.vercel.app/api?type=waving&color=0:1a1a2e,100:0d0d0d&fontColor=FF003C&fontSize=50&height=100&width=900&text=%5BEOF%5D&section=footer"/>
</p>