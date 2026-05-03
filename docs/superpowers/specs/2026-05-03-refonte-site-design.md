# Refonte du site Hexa-AI — Design

**Date :** 2026-05-03
**Périmètre :** refonte complète du site corporate `www.hexa-ai.fr` (one-pager) + ajout des pages légales et fichiers SEO/IA.
**Statut :** implémenté.

## 1. Objectifs

Remplacer l'ancien site one-pager (basé sur Tailwind CDN, contenu daté) par une version :
- esthétique « industriel-moderne, dashboard épuré » conforme au brief client
- alignée sur le positionnement actuel : automatisme industriel, supervision, modernisation, intervention « là où les autres s'arrêtent »
- techniquement saine (sans dépendance lourde, performante mobile)
- optimisée SEO local (Lyon / Brignais) et AI (LLMs)
- légalement conforme (mentions légales, RGPD, CGV)

## 2. Décisions de style

| Item | Choix |
|---|---|
| Palette fond | `#0a0f14` (très sombre) |
| Surface cartes | `#11181f` / `#14202a` |
| Bordures | `#1d2832` / `#2a3a47` |
| Accent principal | `#00d4aa` (cyan/vert électrique) |
| Texte principal | `#e6edf3` |
| Texte secondaire | `#9aa8b3` |
| Typographie | Inter unique (400 / 500 / 600 / 700 / 800) — Google Fonts |
| Hero background | Grille technique 32px, fade radial vers le centre, parallax léger au scroll, 6 nœuds cyan pulsants |
| Cartes Références | Avec photo en haut (16:9, hover scale 1.05) |
| Logo | `img/Hexa-AI dark.png` (préexistant) |
| Favicon | Récupéré depuis `edge.hexa-ai.fr` (hexagone orange multi-tailles) |

## 3. Architecture des fichiers

Static GitHub Pages (CNAME `www.hexa-ai.fr`). Aucun build step. Aucune dépendance JS externe sauf Google Fonts.

```
/
├── index.html              ← page principale (one-pager, ~1450 lignes inline CSS+JS)
├── mentions-legales.html   ← page légale standalone
├── confidentialite.html    ← politique RGPD standalone
├── favicon.ico             ← multi-tailles
├── robots.txt              ← autorise 15 crawlers IA explicitement
├── sitemap.xml             ← 3 URLs (home + 2 légales)
├── llms.txt                ← résumé markdown structuré pour LLMs
├── CNAME                   ← www.hexa-ai.fr (préexistant)
├── img/                    ← assets visuels (5 photos référence + logo + box produit + og-image)
└── docs/
    └── CGV-v2.1.pdf        ← Conditions Générales de Vente
```

**Choix de virer Tailwind CDN** : remplacé par CSS custom inline (~700 lignes). Gains : -3MB, suppression du JIT compiler client, meilleure cohérence visuelle, conforme au brief « pas de dépendances externes sauf Google Fonts ».

## 4. Sections de la page principale

Dans l'ordre vertical :

1. **Nav sticky** — logo gauche, liens (Solutions / Réalisations / Notre Produit / Contact-CTA) droite. Mobile : burger plein écran. Background opaque quand menu ouvert (correction du bug de transparence). Active link avec souligné cyan via Intersection Observer.

2. **Hero** — `min-height: 88vh`, eyebrow `Hexa-AI · Lyon — Brignais (69)`, H1 « Vous n'avez pas d'automaticien en interne ? » + accent cyan « C'est exactement pour ça qu'on existe », sous-titre, 2 CTA (primary cyan + ghost outline), indicateur scroll en bas. Cascade d'apparition rise+fade au load.

3. **Situations** (4 cas) — grille auto-fit `minmax(260px, 1fr)`, cartes avec numéro `CAS 0X` en cyan + titre + description. Hover : translateY + barre cyan top qui s'étend.

4. **Domaines** (3 piliers) — Automatisme & Intégration · Supervision & Connectivité · Modernisation. Cartes avec icône SVG cyan (56x56 dans rond), hover : border cyan + bg surface-hi.

5. **Produit HAI-P200** — layout 2 colonnes (texte gauche / visuel droite). Visuel = photo réelle `Box_Monitoring_HAI_P200.webp` sur fond grille + radial cyan, drop-shadow. 3 badges pills (installation, 4G+VPN, prix). Bouton vers `edge.hexa-ai.fr`. Background différent (`bg-surface`) pour distinguer la section.

6. **Plateformes & écosystème technologique** — bande sociale-proof avec 12 wordmarks texte (CODESYS, WAGO, Beckhoff, SIEMENS, Schneider Electric, Isma Controlli, Johnson Controls, Ignition, PC-Vue, Node-RED, Grafana, Tailscale). Inter 700, blanc 55% opacité, hover 100%. Pas de liens (cf. arbitrage : éviter les fuites d'attention).

7. **Réalisations** (5 références) — grille 6 colonnes avec cartes en `span 2`. Centrage des orphelins via `:nth-child(3n+1):nth-last-child(2)` (cartes 4-5 centrées). Cartes : photo 16:9, label `CAS 0X — secteur`, titre client, description avec partenaires en gras, tags cyan. Cas 04 (Scorp-IO) avec lien externe sortant + icône ↗. Cas 03 (IMAGEAU) avec sous-titre `(Groupe SAUR)` discret.

   **Références retirées (autorisations en attente)** : TotalEnergies (CAS 01) et Nemera (CAS 07).

8. **Contact** — H2 + lead + phrase d'engagement (« on répond dans la journée, pas de commercial »). CTA primary vers Google Calendar. 4 blocs info : email, téléphone, bureau Brignais, siège Vourles. Bordure top.

9. **Footer** — top : newsletter (gauche) + liens externes hexa-ai.fr/codesys.hexa-ai.fr/edge.hexa-ai.fr + LinkedIn (droite). Bottom : copyright + liens légaux (Mentions légales / Confidentialité / CGV).

10. **Bouton retour-en-haut** — fixed bottom-right, apparaît après 600px de scroll. Refactoré de `<a href="#">` vers `<button>` + JS `scrollTo` pour ne pas polluer l'URL.

## 5. Newsletter — intégration Odoo

Choix retenu : **mode `no-cors`** (Option B sur 3 proposées).

- POST vers `https://edge.hexa-ai.fr/website_mass_mailing/subscribe`
- Format JSON-RPC, `list_id: 2` (Newsletter Hexa-AI Edge)
- Content-Type `text/plain` (forcé par no-cors, accepté par Odoo)
- Validation email regex côté client avant envoi
- Affichage optimiste « Merci ! Vous recevrez un e-mail de confirmation » après envoi (la réponse Odoo n'est pas lisible en no-cors)
- L'utilisateur reçoit (ou non) un mail de confirmation Odoo → auto-correction implicite si l'email était invalide

## 6. SEO local (Lyon / Brignais)

- Title : `Hexa-AI — Automatisme industriel à Lyon · Brignais (69)`
- Description : explicite « Lyon (Brignais) ... Auvergne-Rhône-Alpes & France entière »
- JSON-LD `ProfessionalService` (et non plus `Organization`) avec :
  - `address` Brignais (signal local pack Google)
  - `additionalProperty` Siège social Vourles
  - `geo` coordonnées GPS (45.6695, 4.7547)
  - `addressRegion` Auvergne-Rhône-Alpes
  - `areaServed` : Lyon, Auvergne-Rhône-Alpes, France
  - `knowsAbout` : 18 keywords métier (CODESYS, Wago, Modbus, OPC-UA, GTB...)
- OpenGraph + Twitter Card complets, `og:image` 1200×630 généré depuis le hero
- `<meta name="theme-color">` `#0a0f14` (barre d'adresse mobile)
- `<link rel="canonical">`
- `<meta name="robots" content="index, follow, max-image-preview:large, max-snippet:-1">`

## 7. AI SEO (référencement par les LLMs)

- **`/robots.txt`** : 15 crawlers IA explicitement autorisés (GPTBot, ChatGPT-User, OAI-SearchBot, ClaudeBot, anthropic-ai, Claude-Web, PerplexityBot, Perplexity-User, Google-Extended, Applebot-Extended, CCBot, cohere-ai, Meta-ExternalAgent, Bytespider, DuckAssistBot)
- **`/llms.txt`** : standard émergent llmstxt.org. Markdown structuré : identité, domaines, produit, plateformes, cas clients, contact.
- **`/sitemap.xml`** : 3 URLs avec `<image:image>` pour l'og-image
- Contenu déjà aligné « LLM-friendly » : chiffres précis (450 luminaires DALI, 590 €), faits structurés, partenaires nommés.

## 8. Pages légales

### `mentions-legales.html` (loi LCEN 2004-575)
SASU au capital de 10 000 €, RCS Lyon B 900 340 910, SIRET 900 340 910 00018, TVA FR27 900 340 910, APE 6201Z, Julien Talbourdet directeur de publication, GitHub Pages hébergeur.

### `confidentialite.html` (RGPD)
- Aucun cookie, aucun analytics sur `www.hexa-ai.fr`
- Seule donnée collectée : email newsletter via Odoo Online (Belgique, UE)
- Sous-traitants déclarés : GitHub Pages (US), Odoo Online (UE), Google Calendar/Fonts (US)
- Transferts US encadrés DPF + CCT
- Conservation 3 ans (standard)
- Droits RGPD complets, contact `contact@hexa-ai.fr`, recours CNIL

Les 2 pages utilisent un mini-design system cohérent (mêmes tokens couleur, Inter, layout container 820px) sans dupliquer la complexité du index.

## 9. Performance & accessibilité

| Item | Mesure |
|---|---|
| Poids images référence | ~830 KB (réduit de 6 MB initial via `Image.LANCZOS` + JPEG q=78) |
| OG image | 58 KB (1200×630) |
| Dépendances externes | Google Fonts uniquement (Inter) |
| `prefers-reduced-motion` | toutes les animations désactivées |
| `noscript` fallback | menu visible sans JS, reveal désactivé |
| Print stylesheet | inversion fond → blanc, masque nav/footer/decorations |
| Sémantique HTML | `<header>`, `<main>` implicite, `<section>`, `<article>`, `<nav>`, `<footer>` |
| Aria | `aria-label`, `aria-expanded`, `aria-controls`, `aria-live` newsletter, `aria-labelledby` platforms |
| Focus visible | global `:focus-visible` outline cyan |

## 10. Pas de scope

Volontairement non inclus :
- Multilinguisme (EN viendra plus tard si besoin Wikipedia EN / corpus US des LLMs)
- Blog / actualités (pas de besoin éditorial actuel)
- Page « Équipe » détaillée (la société est encore mono-personne)
- Analytics (volontairement absent pour minimiser RGPD ; à ajouter quand besoin de mesures)
- Système de design partagé entre `index.html` et pages légales (mineur, pas justifié pour 3 pages)
- Mentions légales rédigées avec un avocat (générique RGPD, à valider en cas de doute)

## 11. Évolutions probables

- Ajout des références TotalEnergies et Nemera (en attente d'autorisations clients)
- Ajout de logos officiels SVG sur le wall plateformes (actuellement wordmarks texte)
- Versions traduites (EN minimum) pour le SEO international + LLMs
- Ajout d'un blog / cas clients détaillés en sous-pages quand le volume le justifiera
- Mise à jour `lastmod` sitemap + `llms.txt` à chaque évolution majeure du contenu

## 12. Déploiement

- Hébergement : GitHub Pages, branche `main`, fichier `CNAME` → `www.hexa-ai.fr`
- Aucun build, push direct = déploiement
- TTL DNS et propagation Pages : ~1 min après push
