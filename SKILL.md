---
name: scroll-video-showcase
description: "Transforme une vidéo courte (8-15s) en site vitrine immersif avec arrière-plan animé frame-par-frame synchronisé au scroll. Utilise FFmpeg + Canvas + Lenis pour créer un effet vidéo défilante au scroll. Exemple inclus : Cocolat (chocolat). Actions : créer un site scroll-vidéo, générer sprite sheet, site vitrine animé. Formats : vidéo → sprite sheet JPEG/PNG → Canvas scroll-driven."
argument-hint: "[video-path] [brand-name]"
license: MIT
metadata:
  author: freebuff
  version: "1.0.0"
---

# Scroll Video Showcase

Transforme une vidéo courte (8-15 secondes) en site vitrine immersif où l'arrière-plan défile frame par frame en synchronisation avec le scroll, donnant l'impression de contrôler une vidéo par le défilement.

## Quand utiliser cette skill

- Créer un site vitrine à partir d'une vidéo courte
- Effet "scroll vidéo" — l'arrière-plan change à chaque frame comme une vidéo
- Landing page immersive avec un produit en mouvement
- Narration visuelle scroll-driven (storytelling vidéo)
- Showcase produit avec transitions d'images fluides

## Workflow Principal

### Étape 1 : Extraire les frames de la vidéo

```bash
# Extraire toutes les frames en JPEG (24 fps recommandé)
ffmpeg -i "video.mp4" -vf "fps=24" -q:v 1 frames/frame-%04d.jpg -y
```

**Règles :**
- 24 fps = fluidité parfaite sans être trop lourd
- `-q:v 1` = qualité JPEG maximale (quasi sans perte)
- Compter le nombre de frames : `ls frames/*.jpg | wc -l`

### Étape 2 : Créer la sprite sheet (fusion en une image)

```bash
# Calculer la grille : chercher le plus grand rectangle NxM possible
# Exemple : 192 frames → 12×16 (12 cols × 16 rows)
COLS=12
ROWS=16
# Résolution des frames = 1280×720 (résolution originale)
# Sprite finale = (1280 × COLS) × (720 × ROWS)

ffmpeg -i "video.mp4" -vf "fps=24,scale=1280:-1,tile=${COLS}x${ROWS}" -q:v 1 sprite.jpg -y
```

**Calcul de la grille :**
```
Frames totales = 192
Trouver COLS × ROWS ≥ total_frames
12 × 16 = 192 ✓ (parfait)
```

**Contraintes :**
- **Limite JPEG/PNG :** pas de limite pratique (< 65535 px)
- **Limite GPU mobile :** ~16384×16384 pixels max (WebGL 2.0)
  - Résolution frame = 1280×720
  - Max colonnes = floor(16384 / 1280) = **12**
  - Max lignes = floor(16384 / 720) = **22**
  - → Grille max = **12×22 = 264 frames**
- **Alterner la grille si besoin :** si la largeur dépasse, échanger COLS et ROWS

**Formule universelle :**
```
MAX_DIM = 16384 (limite GPU mobile)
frame_w = 1280, frame_h = 720
max_cols = floor(MAX_DIM / frame_w) = 12
max_rows = floor(MAX_DIM / frame_h) = 22

# Choisir la meilleure grille
if total_frames <= max_rows:
    COLS, ROWS = 1, total_frames
elif total_frames <= max_cols * max_rows:
    for cols from max_cols down to 1:
        if total_frames % cols == 0:
            COLS, ROWS = cols, total_frames / cols
            break
```

### Étape 3 : Créer le site HTML

Le site utilise :
1. **Canvas** pour découper et afficher la sprite frame par frame
2. **Lenis** pour un smooth scroll professionnel
3. **IntersectionObserver** pour les animations de texte
4. **img.decode()** pour pré-décoder l'image avant affichage

#### Structure du code

```javascript
// --- Configuration ---
const COLS = 12;       // Nombre de colonnes dans la sprite
const ROWS = 16;       // Nombre de lignes dans la sprite
const TOTAL = 192;     // Nombre total de frames
const FPS = 24;        // Images par seconde de la vidéo source
const DURATION = 8;    // Durée en secondes

// --- Canvas setup ---
const canvas = document.getElementById('bg-canvas');
const ctx = canvas.getContext('2d');

function resizeCanvas() {
  canvas.width = window.innerWidth;
  canvas.height = window.innerHeight;
}

// --- Frame drawing ---
function drawFrame(idx) {
  const col = idx % COLS;
  const row = Math.floor(idx / COLS);
  const fw = sprite.naturalWidth / COLS;
  const fh = sprite.naturalHeight / ROWS;
  ctx.clearRect(0, 0, canvas.width, canvas.height);
  ctx.drawImage(sprite, col * fw, row * fh, fw, fh, 0, 0, canvas.width, canvas.height);
}

// --- Scroll → Frame mapping ---
function updateFrame(scrollY) {
  const docHeight = document.documentElement.scrollHeight - window.innerHeight;
  const progress = Math.min(Math.max(scrollY / docHeight, 0), 1);
  const idx = Math.round(progress * (TOTAL - 1));
  drawFrame(idx);
}

// --- Sprite loading with pre-decode ---
const sprite = new Image();
sprite.onload = () => {
  sprite.decode().then(() => {
    resizeCanvas();
    updateFrame(0);
    // Masquer le loader
  });
};
sprite.src = './sprite.jpg';

// --- Lenis smooth scroll ---
const lenis = new Lenis({
  duration: 1.2,
  easing: (t) => Math.min(1, 1.001 - Math.pow(2, -10 * t)),
  orientation: 'vertical',
});
lenis.on('scroll', (e) => updateFrame(e.animatedScroll));
function raf(time) { lenis.raf(time); requestAnimationFrame(raf); }
requestAnimationFrame(raf);
```

#### Structure HTML complète

```html
<!DOCTYPE html>
<html lang="fr">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Brand — Scroll Video</title>
  <link rel="preconnect" href="https://fonts.googleapis.com" />
  <link href="https://fonts.googleapis.com/css2?family=Playfair+Display:wght@400;600;700&family=Inter:wght@300;400;500;600&display=swap" rel="stylesheet" />
  <style>
    /* --- Reset --- */
    * { margin: 0; padding: 0; box-sizing: border-box; }
    html { scroll-behavior: auto; }
    body { font-family: 'Inter', sans-serif; overflow-x: hidden; background: #1a0f0a; }

    /* --- Canvas plein écran --- */
    #bg-canvas {
      position: fixed; top: 0; left: 0;
      width: 100vw; height: 100vh;
      object-fit: cover; z-index: 0;
    }

    /* --- Navbar --- */
    .navbar {
      position: fixed; top: 0; left: 0; right: 0; z-index: 100;
      padding: 1.2rem 2rem;
      display: flex; justify-content: space-between; align-items: center;
      transition: all 0.6s cubic-bezier(0.16, 1, 0.3, 1);
      opacity: 0; transform: translateY(-100%);
      background: linear-gradient(180deg, rgba(0,0,0,0.6) 0%, transparent 100%);
    }
    .navbar.visible { opacity: 1; transform: translateY(0); }
    .navbar .logo { font-family: 'Playfair Display', serif; font-size: 1.5rem; color: #fff; }
    .navbar .nav-links { display: flex; gap: 2rem; }
    .navbar .nav-links a { color: rgba(255,255,255,0.7); text-decoration: none; font-size: 0.85rem; font-weight: 500; letter-spacing: 0.05em; text-transform: uppercase; transition: color 0.3s; }
    .navbar .nav-links a:hover { color: #fff; }

    /* --- Sections contenu --- */
    .content { position: relative; z-index: 1; padding-top: 100vh; }
    .section {
      min-height: 100vh;
      display: flex; align-items: center;
      padding: 4rem 6rem;
      color: #fff;
    }
    .section .text-block {
      max-width: 520px;
      opacity: 0; transform: translateY(40px);
      transition: all 0.8s cubic-bezier(0.16, 1, 0.3, 1);
    }
    .section .text-block.visible { opacity: 1; transform: translateY(0); }
    .section .text-block.from-left { transform: translateX(-60px); }
    .section .text-block.from-left.visible { transform: translateX(0); }
    .section .text-block.from-right { transform: translateX(60px); margin-left: auto; }
    .section .text-block.from-right.visible { transform: translateX(0); }
    .section .text-block h2 { font-family: 'Playfair Display', serif; font-size: 2.8rem; font-weight: 600; margin-bottom: 1rem; }
    .section .text-block p { font-size: 1.05rem; line-height: 1.8; color: rgba(255,255,255,0.75); }
    .section .text-block .badge {
      display: inline-block; padding: 0.4rem 1.2rem; border-radius: 50px;
      font-size: 0.75rem; font-weight: 600; letter-spacing: 0.08em;
      text-transform: uppercase; margin-bottom: 1rem;
      background: rgba(255,255,255,0.12); backdrop-filter: blur(8px);
    }

    /* --- Loader --- */
    .loader {
      position: fixed; inset: 0; z-index: 9999;
      display: flex; flex-direction: column;
      justify-content: center; align-items: center;
      background: #1a0f0a; color: #fff;
      transition: opacity 0.8s ease;
    }
    .loader.hidden { opacity: 0; pointer-events: none; }
    .loader .progress-bar {
      width: 200px; height: 2px; background: rgba(255,255,255,0.15);
      border-radius: 2px; overflow: hidden; margin-top: 1rem;
    }
    .loader .progress-bar .fill {
      height: 100%; width: 0%; background: #c9a96e;
      transition: width 0.3s ease;
    }
    .loader .status { font-size: 0.85rem; color: rgba(255,255,255,0.5); margin-top: 0.8rem; }

    /* --- Scroll progress bar --- */
    .scroll-progress {
      position: fixed; top: 0; left: 0; right: 0; z-index: 99;
      height: 2px; background: transparent;
    }
    .scroll-progress .fill {
      height: 100%; width: 0%;
      background: linear-gradient(90deg, #c9a96e, #e8c88a);
      transition: width 0.1s linear;
    }

    /* --- Hamburger mobile --- */
    .hamburger { display: none; flex-direction: column; cursor: pointer; gap: 5px; }
    .hamburger span { width: 24px; height: 2px; background: #fff; transition: all 0.3s; }
    .hamburger.active span:nth-child(1) { transform: rotate(45deg) translate(5px, 5px); }
    .hamburger.active span:nth-child(2) { opacity: 0; }
    .hamburger.active span:nth-child(3) { transform: rotate(-45deg) translate(5px, -5px); }

    /* --- Responsive --- */
    @media (max-width: 768px) {
      .navbar { padding: 1rem 1.2rem; }
      .navbar .nav-links { display: none; position: absolute; top: 100%; left: 0; right: 0; flex-direction: column; background: rgba(0,0,0,0.95); padding: 1rem; gap: 1rem; }
      .navbar .nav-links.open { display: flex; }
      .hamburger { display: flex; }
      .section { padding: 3rem 1.5rem; min-height: 80vh; }
      .section .text-block h2 { font-size: 2rem; }
      .section .text-block p { font-size: 0.95rem; }
    }
  </style>
</head>
<body>
  <canvas id="bg-canvas"></canvas>

  <div class="scroll-progress"><div class="fill" id="progress-fill"></div></div>

  <nav class="navbar" id="navbar">
    <div class="logo">BRAND</div>
    <div class="hamburger" id="hamburger"><span></span><span></span><span></span></div>
    <div class="nav-links" id="navLinks">
      <a href="#histoire">Histoire</a>
      <a href="#savoir">Savoir-faire</a>
      <a href="#produits">Produits</a>
      <a href="#contact">Contact</a>
    </div>
  </nav>

  <div class="content">
    <section class="section" id="histoire">
      <div class="text-block from-left">
        <span class="badge">Notre histoire</span>
        <h2>Un héritage d'excellence</h2>
        <p>Depuis des générations, nous perpétuons un savoir-faire artisanal d'exception.</p>
      </div>
    </section>
    <section class="section" id="savoir">
      <div class="text-block from-right">
        <span class="badge">Savoir-faire</span>
        <h2>L'art de la perfection</h2>
        <p>Chaque détail compte. De la sélection des matières premières à la finition.</p>
      </div>
    </section>
    <section class="section" id="produits">
      <div class="text-block from-left">
        <span class="badge">Nos créations</span>
        <h2>L'excellence à chaque bouchée</h2>
        <p>Découvrez notre gamme de produits d'exception, conçus pour les connaisseurs.</p>
      </div>
    </section>
    <section class="section" id="contact">
      <div class="text-block from-right">
        <span class="badge">Contact</span>
        <h2>Nous rencontrer</h2>
        <p>Retrouvez-nous dans notre boutique ou commandez en ligne.</p>
      </div>
    </section>
  </div>

  <div class="loader" id="loader">
    <div style="font-family: 'Playfair Display', serif; font-size: 1.2rem; letter-spacing: 0.15em; text-transform: uppercase;">BRAND</div>
    <div class="progress-bar"><div class="fill" id="loader-fill"></div></div>
    <div class="status" id="loader-status">Chargement...</div>
  </div>

  <script src="https://unpkg.com/lenis@1.1.20/dist/lenis.min.js"></script>
  <script>
    // --- Configuration (À PERSONNALISER) ---
    const COLS = 12;
    const ROWS = 16;
    const TOTAL = 192;
    const SPRITE_SRC = './sprite.jpg';

    // --- Canvas ---
    const canvas = document.getElementById('bg-canvas');
    const ctx = canvas.getContext('2d');

    function resizeCanvas() {
      canvas.width = window.innerWidth;
      canvas.height = window.innerHeight;
    }
    window.addEventListener('resize', resizeCanvas);
    resizeCanvas();

    // --- Frame drawing (FW, FH initialisées après decode) ---
    let FW, FH;
    function drawFrame(idx) {
      const col = idx % COLS;
      const row = Math.floor(idx / COLS);
      ctx.clearRect(0, 0, canvas.width, canvas.height);
      ctx.drawImage(sprite, col * FW, row * FH, FW, FH, 0, 0, canvas.width, canvas.height);
    }

    let ready = false;
    let currentIdx = 0;

    function updateFrame(scrollY) {
      if (!ready) return;
      const docHeight = document.documentElement.scrollHeight - window.innerHeight;
      const progress = docHeight > 0 ? Math.min(Math.max(scrollY / docHeight, 0), 1) : 0;
      const idx = Math.round(progress * (TOTAL - 1));
      if (idx === currentIdx) return;
      currentIdx = idx;
      drawFrame(idx);
    }

    // --- Sprite loading ---
    const sprite = new Image();
    sprite.onload = () => {
      sprite.decode().then(() => {
        FW = sprite.naturalWidth / COLS;
        FH = sprite.naturalHeight / ROWS;
        ready = true;
        resizeCanvas();
        updateFrame(0);
        document.getElementById('loader').classList.add('hidden');
      }).catch(() => {
        // Fallback si decode échoue
        FW = sprite.naturalWidth / COLS;
        FH = sprite.naturalHeight / ROWS;
        ready = true;
        resizeCanvas();
        updateFrame(0);
        document.getElementById('loader').classList.add('hidden');
      });
    };
    sprite.onerror = () => {
      document.getElementById('loader-status').textContent = 'Erreur de chargement';
      // Fallback : permettre au site de fonctionner sans fond animé
      setTimeout(() => document.getElementById('loader').classList.add('hidden'), 2000);
    };
    sprite.src = SPRITE_SRC;

    // --- Lenis smooth scroll ---
    const lenis = new Lenis({
      duration: 1.2,
      easing: (t) => Math.min(1, 1.001 - Math.pow(2, -10 * t)),
      orientation: 'vertical',
      gestureOrientation: 'vertical',
      smoothWheel: true,
    });
    lenis.on('scroll', (e) => {
      updateFrame(e.animatedScroll);
      document.getElementById('progress-fill').style.width = (e.progress * 100) + '%';
    });
    function raf(time) { lenis.raf(time); requestAnimationFrame(raf); }
    requestAnimationFrame(raf);

    // --- Navbar ---
    const navbar = document.getElementById('navbar');
    const navLinks = document.getElementById('navLinks');
    const hamburger = document.getElementById('hamburger');
    let lastScroll = 0;

    lenis.on('scroll', (e) => {
      if (e.animatedScroll > 100) navbar.classList.add('visible');
      else navbar.classList.remove('visible');
      lastScroll = e.animatedScroll;
    });

    hamburger.addEventListener('click', () => {
      hamburger.classList.toggle('active');
      navLinks.classList.toggle('open');
    });

    // Navbar links → smooth scroll
    navLinks.querySelectorAll('a').forEach(a => {
      a.addEventListener('click', (e) => {
        e.preventDefault();
        const target = document.querySelector(a.getAttribute('href'));
        if (target) {
          lenis.scrollTo(target, { offset: -60 });
          navLinks.classList.remove('open');
          hamburger.classList.remove('active');
        }
      });
    });

    // --- IntersectionObserver for text blocks ---
    const observer = new IntersectionObserver((entries) => {
      entries.forEach(entry => {
        if (entry.isIntersecting) {
          entry.target.classList.add('visible');
        }
      });
    }, { threshold: 0.15 });

    document.querySelectorAll('.text-block').forEach(el => observer.observe(el));
  </script>
</body>
</html>
```

### Étape 4 : Personnaliser

1. **COLS, ROWS, TOTAL** → adapter au nombre de frames et à la grille
2. **SPRITE_SRC** → chemin vers la sprite sheet
3. **Textes et sections** → adapter au produit/brand
4. **Couleurs** → modifier les variables dans le CSS (fond, doré/accent)
5. **Polices** → Google Fonts (Playfair Display + Inter par défaut)

## Few-shot Example : Cocolat

Un exemple complet est disponible dans le dossier `examples/cocolat/` :

```
examples/cocolat/
├── index.html    ← Site complet (26 Ko)
└── sprite.jpg    ← Sprite sheet 192 frames (17 Mo)
```

**Détails de l'exemple :**
- Vidéo source : 8 secondes, 24 fps, 1280×720
- Frames : 192 (12 colonnes × 16 lignes)
- Sprite : 15360×11520 pixels, JPEG qualité max
- 7 sections avec textes animés (histoire, terroir, fabrication, dégustation, prix, contact)
- Couleurs : fond brun (#1a0f0a), accent doré (#c9a96e)
- Navbar avec dégradé, menu hamburger mobile, barre de progression

## Résolution des problèmes courants

| Problème | Cause | Solution |
|---|---|---|
| **Freeze au premier défilement** | JPEG pas encore décodé par le GPU | Ajouter `img.decode().then(() => ready = true)` |
| **Saccades en scroll lent** | Trop peu de frames | Augmenter le framerate (24 fps minimum, 30 fps idéal) |
| **Ghosting / flou** | Interpolation entre frames | Utiliser `Math.round()` au lieu de cross-fade ou lerp |
| **Image trop grande pour le GPU** | Sprite dépasse 16384×16384 | Diviser en 2-4 sprites ou réduire la résolution |
| **Barre marron/transparente** | Frame suivante pas encore disponible | Vérifier que la sprite est chargée avant de dessiner |
| **Mobile : image ne charge pas** | Chemin absolu ou symlink | Copier les fichiers, pas de lien symbolique |
| **Blanc/rien affiché** | Mauvais chemin de sprite | Vérifier le chemin relatif `./sprite.jpg` |

## Anti-patrons à éviter

- ❌ **Cross-fade entre frames** → crée du ghosting. Préférer le hard switch (`Math.round`)
- ❌ **Animer `currentTime` d'une vidéo** → lent et saccadé. Toujours utiliser une sprite sheet + Canvas
- ❌ **Sprite unique > 16384 px** → risque de planter sur mobile. Découper en sous-sprites
- ❌ **Utiliser `ctx.globalAlpha`** → lourd et inutile. `clearRect` + `drawImage` est plus rapide
- ❌ **Oublier `img.decode()`** → freeze au premier défilement sur les grosses images
- ❌ **Sprite JPEG de qualité basse (< -q:v 5)** → artefacts visibles sur fond plein écran

## Prérequis

**FFmpeg** — doit être installé pour extraire les frames et créer la sprite sheet :

```bash
# Vérifier
ffmpeg -version

# Installer (Linux)
sudo apt install ffmpeg

# Installer (macOS)
brew install ffmpeg

# Installer (Windows)
winget install ffmpeg
```

**Navigateur** — le site fonctionne sur Chrome, Firefox, Safari, Edge (version récente).

## Workflows avancés

### Site multi-sprites (pour très haute résolution)

Si la vidéo source est en 4K (3840×2160) ou si la sprite dépasse 16384 px :

```bash
# Créer 4 sprites de 48 frames chacune
ffmpeg -i "video.mp4" -vf "select='between(n,0,47)',setpts=N/24/TB,scale=1280:-1,tile=6x8" -q:v 1 sprite1.jpg -y
ffmpeg -i "video.mp4" -vf "select='between(n,48,95)',setpts=N/24/TB,scale=1280:-1,tile=6x8" -q:v 1 sprite2.jpg -y
ffmpeg -i "video.mp4" -vf "select='between(n,96,143)',setpts=N/24/TB,scale=1280:-1,tile=6x8" -q:v 1 sprite3.jpg -y
ffmpeg -i "video.mp4" -vf "select='between(n,144,191)',setpts=N/24/TB,scale=1280:-1,tile=6x8" -q:v 1 sprite4.jpg -y
```

Puis dans le JS, charger les sprites progressivement et afficher la bonne sprite selon l'index de frame.

### Format WebP pour sprite (30% plus léger)

```bash
# WebP avec qualité 90 (quasi sans perte)
ffmpeg -i "video.mp4" -vf "fps=24,scale=1280:-1,tile=12x16" -c:v libwebp -quality 90 sprite.webp -y
```

**Attention :** WebP a une limite de 16383×16383 pixels. Pour les très grandes sprites, préférer JPEG.

### Vidéo longue (> 15 secondes)

Pour les vidéos longues, réduire le framerate pour garder une sprite de taille raisonnable :

```bash
# 15 fps au lieu de 24 (moitié moins de frames)
ffmpeg -i "video.mp4" -vf "fps=15,scale=1280:-1,tile=12x16" -q:v 1 sprite.jpg -y
```

## Intégration

**Skills liées :** `frontend-design`, `ui-ux-pro-max`, `immersion`
**Outils externes :** FFmpeg, Lenis (smooth scroll), Canvas API
