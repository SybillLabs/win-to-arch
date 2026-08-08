<h1 align="center">
    🛠️ Installation et configuration : Kitty, Starship & Fastfetch
</h1>

## `> ./kitty-installation.sh`

- **Installation de Kitty**
```bash
sudo pacman -S kitty
```

- **Installation de la police (Nerd Font, requise pour les icônes du prompt et de fastfetch)**
```bash
sudo pacman -S ttf-jetbrains-mono-nerd
```

- **Sélection du thème via le kitten intégré**
```bash
kitty +kitten themes
```
> **Remarque :** ouvre un sélecteur interactif dans le terminal, recherche possible par nom (exemple : `cyberpunk`), aperçu en direct avant validation. Le thème choisi est automatiquement écrit dans `~/.config/kitty/current-theme.conf` et référencé depuis `kitty.conf` via `include current-theme.conf`.

- **Ajustements manuels après sélection du thème**
```bash
nano ~/.config/kitty/kitty.conf
```
- **Contenu du fichier `kitty.conf`**
```conf
# BEGIN_KITTY_THEME
# Cyberpunk
include current-theme.conf
# END_KITTY_THEME

font_family                 JetBrainsMono Nerd Font
font_size                   12.0

background_opacity          0.85
window_padding_width        8

cursor_shape                beam
cursor_beam_interval        0.5

enable_audio_bell           no
confirm_os_window_close     0
```
> **Remarque :** police et opacité ajoutées par-dessus le thème de base, pour rester cohérent avec le reste des conventions esthétiques du dépôt.

## `> ./starship-installation.sh`

- **Installation de Starship (prompt)**
```bash
sudo pacman -S starship
```

- **Configuration de Starship**
```bash
starship preset nerd-font-symbols -o ~/.config/starship.toml
nano ~/.config/starship.toml
```
- **Contenu du fichier `starship.toml`**
```toml
format = """
$os\
$username\
$hostname\
$directory\
$git_branch\
$git_status\
$character"""

[os]
style = "fg:#fcee09"
disabled = false

[os.symbols]
EndeavourOS = "🐧 "

[username]
style_user = "fg:#fcee09"
style_root = "fg:#fcee09"
format = "[$user]($style)"
show_always = true
disabled = false

[hostname]
ssh_only = false
style = "fg:#fcee09"
format = "[@$hostname ]($style)"
disabled = false

[directory]
style = "fg:#00eaff"
format = "[$path]($style) "
truncation_length = 3

[git_branch]
symbol = ""
style = "fg:#00eaff"
format = "[$symbol$branch]($style) "

[git_status]
style = "fg:#00eaff"
format = "[$all_status]($style) "

[character]
success_symbol = "[➜](bold #00eaff)"
error_symbol = "[➜](bold #ff005c)"
```
- **Activation de Starship au démarrage du shell**
```bash
nano ~/.bashrc
```
- **Ajout de la ligne suivante à la fin du fichier**
```bash
eval "$(starship init bash)"
```

## `> ./fastfetch-installation.sh`

- **Installation de fastfetch**
```bash
sudo pacman -S fastfetch
```
- **Activation de fastfetch au démarrage du shell**
```bash
nano ~/.bashrc
```
- **Ajout de la ligne suivante à la fin du fichier**
```bash
fastfetch
```

## `> ./terminal-verification.sh`

```bash
kitty --version
starship --version
fastfetch --version
```
> **Résultat attendu :** ouverture d'un nouveau terminal Kitty affichant le thème cyberpunk sélectionné, le logo EndeavourOS via fastfetch, et le prompt Starship personnalisé.

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