# 🎬 Scroll Video Showcase

Transformez n'importe quelle vidéo courte (8-15s) en **site vitrine immersif** où l'arrière-plan défile frame par frame synchronisé avec le scroll. Un effet "contrôle vidéo par le scroll" professionnel.

> ⚡ **Démo instantanée :** Ouvre [`examples/cocolat/index.html`](examples/cocolat/index.html) dans ton navigateur pour voir le résultat en action !
> *(La sprite fait 17 Mo, le temps de chargement initial est normal)*

[![Voir le site Cocolat](https://img.shields.io/badge/🍫-Voir%20la%20démo%20Cocolat-8B5E3C?style=for-the-badge)](examples/cocolat/index.html)

## ✨ Démo

Le dossier `examples/cocolat/` contient un site complet prêt à l'emploi :
- Une vidéo chocolat de 8 secondes transformée en site vitrine
- 192 frames en 1280×720
- 7 sections avec textes animés
- Navigation fluide, responsive, barre de progression

**Ouvre `examples/cocolat/index.html` dans ton navigateur pour voir le résultat !**

## 🚀 Installation

```bash
npx skills add Fad-cod/scroll-video-showcase
```

## 🎯 Utilisation

Une fois la skill installée, dis simplement à ton agent :

> "Utilise la skill scroll-video-showcase pour créer un site à partir de [ma vidéo]"

L'agent saura exactement quoi faire :
1. Extraire les frames de la vidéo avec FFmpeg
2. Créer une sprite sheet unique
3. Générer le site HTML avec Canvas + Lenis
4. Adapter les textes, couleurs et sections à ton produit

## 🛠️ Comment ça marche

```
Vidéo MP4 (8s, 24fps, 1280×720)
       ↓ FFmpeg tile=12x16
1 Sprite sheet (15360×11520, 192 frames)
       ↓ JavaScript + Canvas
Site vitrine scroll-driven
       ↓ Lenis (smooth scroll)
Défilement fluide et professionnel
```

### Stack technique

| Technologie | Rôle |
|---|---|
| **FFmpeg** | Extraction des frames + création de la sprite sheet |
| **Canvas API** | Découpage et affichage frame par frame |
| **Lenis** | Smooth scroll professionnel |
| **IntersectionObserver** | Animations de texte au scroll |
| **img.decode()** | Pré-décodage GPU pour éviter les freezes |

## 📋 Prérequis

- **FFmpeg** : `sudo apt install ffmpeg` (Linux), `brew install ffmpeg` (macOS)
- Un navigateur moderne (Chrome, Firefox, Safari, Edge)

## ⚡ Résultats garantis

- ✅ 60 fps — animation fluide sans saccade
- ✅ Pas de ghosting — hard switch pur (pas de cross-fade)
- ✅ Pas de freeze au premier défilement — `img.decode()` précharge le GPU
- ✅ Responsive — fonctionne sur mobile et desktop
- ✅ Single file — tout tient dans un dossier transportable

## 📖 Apprendre

Consulte le fichier [SKILL.md](SKILL.md) pour la documentation complète de la méthode, les astuces de performance, les anti-patrons à éviter et les workflows avancés (multi-sprites 4K, WebP, vidéos longues).

## 🌟 Visibilité & Partage

Pour que les gens découvrent et utilisent ta skill :

1. **skills.sh** — Ton repo est automatiquement référencé sur [skills.sh](https://skills.sh) si le `name:` dans le YAML frontmatter est valide (il l'est ✅)
2. **Partage le lien** : `https://github.com/Fad-cod/scroll-video-showcase`
3. **Communautés** : Partage sur Twitter/X avec `@Fad-cod`, sur Reddit (r/webdev, r/javascript, r/css), Discord dev francophones
4. **Tag tes repos** : Ajoute des topics sur GitHub ("skill", "agent-skills", "scroll-animation", "canvas", "ffmpeg", "video-to-site") pour que les gens trouvent la skill par recherche

> 💡 **Astuce :** Une petite vidéo de démonstration (Loom, Streamable, ou GIF) postée sur Twitter/X avec le lien du repo est le meilleur moyen de faire connaître ta création !

## 📄 Licence

MIT — libre d'utiliser, modifier et partager.

---

⭐ **Si tu trouves cette skill utile, n'oublie pas de laisser une star sur GitHub !**
