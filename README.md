# One Piece RPG Desktop

Application desktop native (Windows, macOS, Linux) qui encapsule le jeu navigateur **[One Piece RPG](https://one-piece-rpg.fr/)** dans une fenêtre dédiée. Bâti avec **Tauri 2** + **Leptos**.

> Tu lances un .exe / .app / .AppImage, tu te connectes une fois, tu joues. Plus besoin d'ouvrir un onglet dans ton navigateur ni de réauthentifier à chaque session.

---

## Pourquoi ?

- **Connexion persistante** — les cookies sont stockés dans le dossier app de l'OS, plus jamais besoin de se reconnecter.
- **Fenêtre dédiée** — pas de barre d'onglets, pas d'extensions qui ralentissent.
- **Léger** — webview natif (~13 Mo binaire, ~4 Mo par installeur) plutôt qu'Electron (>100 Mo).
- **Multi-plateforme** — un seul codebase Rust → installateurs Windows / macOS / Linux générés par CI.

---

## Téléchargement

Va dans la section **[Releases](../../releases)** et prends l'installateur correspondant à ton OS :

### Installateurs (recommandé)

| OS | Fichier | Notes |
|---|---|---|
| Windows | `.msi` ou `.exe` (NSIS) | Installe l'app + WebView2 si manquant |
| macOS Apple Silicon (M1/M2/M3/M4) | `.dmg` `aarch64` | Glisser dans Applications |
| macOS Intel | `.dmg` `x64` | Glisser dans Applications |
| Debian / Ubuntu / Mint | `.deb` | `sudo apt install ./fichier.deb` |
| Fedora / openSUSE | `.rpm` | `sudo dnf install ./fichier.rpm` |
| Autre Linux | `.AppImage` | `chmod +x` puis double-clic |

### Binaires portables (no-install)

Pour ceux qui préfèrent un fichier unique à lancer sans installation :

| OS | Fichier | Prérequis |
|---|---|---|
| Windows | `onepiecerpg-lejeu-windows-x64-portable.exe` | **WebView2** (préinstallé sur Win 10/11 ≥ 2022) |
| macOS Apple Silicon | `onepiecerpg-lejeu-macos-arm64-portable` | `chmod +x`, autoriser dans Réglages > Sécurité |
| macOS Intel | `onepiecerpg-lejeu-macos-x64-portable` | idem |
| Linux x64 | `onepiecerpg-lejeu-linux-x64-portable` | `libwebkit2gtk-4.1` installé (Ubuntu 22.04+, Fedora 37+, Arch) |

---

## Stack technique

| Couche | Techno |
|---|---|
| Backend natif | **Rust** + Tauri 2 (wry / webkit2gtk / WKWebView / WebView2) |
| Frontend (Leptos) | **Rust → WASM** via Trunk (actuellement non utilisé pour la fenêtre principale, prêt pour panneaux secondaires) |
| Build & bundling | `cargo tauri build` + GitHub Actions matrix (3 OS) via `tauri-action` |

L'app pointe directement le webview natif sur `https://one-piece-rpg.fr/` — aucun proxy, aucune modification de la page, aucun accès au site backend autre que HTTPS standard.

---

## Build local

Prérequis :
- **Rust stable** (`rustup default stable`)
- Cible WASM : `rustup target add wasm32-unknown-unknown`
- **Trunk** : `cargo install trunk --locked`
- **Tauri CLI** : `cargo install tauri-cli --version "^2" --locked`
- Linux uniquement : `libwebkit2gtk-4.1-dev`, `librsvg2-dev`, `libappindicator3-dev`

```bash
git clone https://github.com/MotherSphere/one-piece-rpg-desktop.git
cd one-piece-rpg-desktop

# Mode dev (hot reload)
cargo tauri dev

# Build release optimisé
cargo tauri build
```

Sortie : `target/release/<binaire>` + bundles dans `target/release/bundle/`.

---

## Publier une nouvelle version

Le workflow CI build et publie une **draft release** sur GitHub à chaque push de tag `v*` :

```bash
git tag v0.1.0
git push origin v0.1.0
```

Va ensuite dans `Actions` → vérifie que le run est vert → édite la draft release créée → publie.

Le workflow construit :
- macOS arm64 + x64 (`.dmg`, `.app`)
- Windows x64 (`.msi`, `.exe`)
- Linux x64 (`.deb`, `.rpm`, `.AppImage`)

---

## Notes Wayland (Linux)

Le runtime patche automatiquement `WEBKIT_DISABLE_DMABUF_RENDERER=1` et `WEBKIT_DISABLE_COMPOSITING_MODE=1` au démarrage pour contourner les crashs connus de webkit2gtk sous Hyprland / sway et autres compositeurs Wayland tilings. Aucune action utilisateur requise.

---

## Avertissement

Application **non-officielle**, **fan-made**, sans aucune affiliation avec les développeurs de [one-piece-rpg.fr](https://one-piece-rpg.fr/). Aucune donnée n'est collectée par cette app — les cookies et la session restent locaux à ta machine.

L'univers *One Piece* est la propriété d'Eiichiro Oda / Shueisha / Toei Animation.
