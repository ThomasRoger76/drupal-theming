# Anatomie d'un Thème Drupal

## Arborescence Complète

```
mon_theme/
├── mon_theme.info.yml           # Métadonnées (obligatoire)
├── mon_theme.libraries.yml      # Bibliothèques CSS/JS
├── mon_theme.theme              # Preprocess + hooks PHP
├── mon_theme.breakpoints.yml    # Points de rupture responsives
├── logo.svg                     # Logo (référencé dans .info.yml)
├── screenshot.png               # Capture admin (/admin/appearance)
├── favicon.ico
├── config/
│   ├── install/                 # Config importée à l'activation du thème
│   └── schema/                  # Schéma de validation config
├── css/
│   ├── base/                    # Reset, variables CSS, typographie de base
│   ├── layout/                  # Grille, régions, structure de page
│   ├── components/              # Boutons, cartes, navigation, formulaires
│   └── theme/                   # Couleurs, polices — surcharges finales
├── js/
│   └── mon-comportement.js
├── images/
├── templates/
│   ├── layout/
│   │   └── page.html.twig
│   ├── content/
│   │   ├── node.html.twig
│   │   └── node--article.html.twig
│   ├── block/
│   ├── field/
│   ├── navigation/
│   ├── views/
│   └── misc/
└── src/                         # PHP optionnel (plugins, EventSubscribers)
```

---

## `.info.yml` — Anatomie Complète

```yaml
# mon_theme.info.yml
name: Mon Thème
description: 'Thème custom pour le projet X.'
type: theme
core_version_requirement: ^10 || ^11
package: Custom

# Héritage — LIRE "Choix du Base Theme" ci-dessous
base theme: stable9

# Régions — déclaration obligatoire si tu veux les utiliser
regions:
  page_top: 'Page top'            # Réservé aux messages système
  header: 'En-tête'
  primary_menu: 'Menu principal'
  secondary_menu: 'Menu secondaire'
  breadcrumb: 'Fil d''Ariane'
  highlighted: 'Mis en avant'
  help: 'Aide'
  content: 'Contenu'              # OBLIGATOIRE — région principale
  sidebar_first: 'Barre latérale gauche'
  sidebar_second: 'Barre latérale droite'
  footer_first: 'Pied de page 1'
  footer_second: 'Pied de page 2'
  page_bottom: 'Page bottom'      # Réservé aux scripts footer

# Librairies chargées GLOBALEMENT sur toutes les pages
libraries:
  - mon_theme/global-styling
  - mon_theme/global-scripts

# Remplacer une librairie core ou contrib par la nôtre
libraries-override:
  core/normalize:
    css:
      base:
        assets/vendor/normalize-css/normalize.css: false   # Désactiver un fichier
  system/base:                    # Désactiver entièrement une librairie
    false

# ÉTENDRE une librairie existante sans la remplacer
libraries-extend:
  core/drupal.dialog:
    - mon_theme/dialog-extend     # Ajouter nos styles au dialog Drupal

# Supprimer des CSS core/contrib individuellement
stylesheets-remove:
  - core/assets/vendor/normalize-css/normalize.css
  - '@mon_module/css/module-styles.css'            # @MODULE_MACHINE_NAME pour cibler un module contrib

# Stylesheet pour l'éditeur CKEditor (édition WYSIWYG)
ckeditor_stylesheets:
  - css/theme/ckeditor.css

# Fichiers logo et favicon (chemins relatifs au thème)
logo: logo.svg
favicon: favicon.ico

# PHP minimum du thème (string exacte, pas de ^)
php: '8.1'

# Masquer ce thème de l'UI admin (utile pour les base themes)
hidden: false
```

---

## Choix du Base Theme

| Base Theme | Quand | Ce qu'il fournit |
|-----------|-------|-----------------|
| `stable9` | Thème custom de zéro sur D9-D11 | Templates minimal stables — peu d'opinion CSS |
| `olivero` | Étendre le thème front D10+ | Design moderne, responsive natif |
| `claro` | Thème d'admin custom | UI admin accessible, système de design cohérent |
| `false` | Contrôle total | Aucun template hérité — tu fournis tout |
| `bootstrap5` | Projets Bootstrap 5 (contrib) | Grille BS5, classes utilitaires — nécessite `drupal/bootstrap5` |
| `gin` | Thème d'admin custom moderne (contrib) | UX admin avancée, sidebar navigation |
| ~~`classy`~~ | ❌ Supprimé D10 | Remplacer par `stable9` ou `false` |
| ~~`bartik`~~ | ❌ Supprimé D10 | Était le thème front D8/D9 |

> **Note Bootstrap5 :** si `base theme: bootstrap5`, la dépendance Composer est obligatoire :
> `composer require drupal/bootstrap5`
> et le thème contrib active jQuery par défaut — `core/jquery` est alors acceptable dans les librairies.

**Héritage :** un thème hérite des templates de son base theme. Si `node.html.twig` n'existe pas dans ton thème, Drupal remonte la chaîne vers le base theme.

---

## Workflow de Développement

### Activer le thème et configurer le debug

```bash
# Activer le thème
ddev drush config:set system.theme default mon_theme -y
ddev drush cr

# Activer Twig Debug (jamais en production)
# Copier le fichier services si pas encore fait
cp web/sites/default/default.services.yml web/sites/default/services.yml
```

Éditer `web/sites/default/services.yml` :
```yaml
parameters:
  twig.config:
    debug: true            # Affiche les commentaires de suggestion de template
    auto_reload: true      # Recharge les templates sans drush cr
    cache: false           # Désactive le cache Twig
```

```bash
ddev drush cr   # OBLIGATOIRE après modification de services.yml
```

**Désactiver l'agrégation CSS/JS** (dev uniquement) :
`/admin/config/development/performance` → décocher les deux cases d'agrégation

### Cycle de développement standard

```bash
# 1. Modifier templates/CSS/JS
# 2. Pour CSS/JS : rafraîchir le navigateur (si agrégation désactivée)
# 3. Pour templates Twig : drush cr ou auto_reload si activé
# 4. Pour preprocess/.theme : TOUJOURS drush cr
ddev drush cr

# Débugger les templates actifs (cr = alias cache:rebuild, un seul suffit)
ddev drush cr

# Voir les erreurs PHP du thème
ddev drush watchdog:tail --type=php
```

---

## Troubleshooting

| Problème | Cause probable | Solution |
|----------|---------------|---------|
| Template non pris en compte | Mauvais nom de fichier ou chemin | Activer Twig debug → copier le nom exact du commentaire HTML |
| CSS/JS absent | Librairie non attachée ou chemin incorrect | Vérifier `.libraries.yml`, désactiver l'agrégation |
| Thème n'apparaît pas dans l'UI | YAML invalide dans `.info.yml` | `drush cr`, puis valider la syntaxe YAML |
| Variables absentes dans Twig | Cache des templates pas vidé après preprocess | `drush cr` — les fonctions preprocess sont mises en cache |
| `base theme: classy` erreur D10 | Module supprimé | Remplacer par `stable9` ou `false` |
| `{{ dump(variable) }}` blanc | Twig debug non activé | Vérifier `services.yml` et `drush cr` |
| Région vide malgré bloc assigné | Région non déclarée dans `.info.yml` | Ajouter la région au YAML |

---

## Single Directory Components (SDC) — D10.3+ / D11

Les SDC regroupent template, CSS, JS et schéma dans un dossier par composant :

```
mon_theme/components/
  card/
    card.component.yml   # Schéma des props
    card.html.twig
    card.css
    card.js
```

```yaml
# card.component.yml
$schema: 'https://git.drupalcode.org/project/drupal/-/raw/HEAD/core/assets/schemas/v1/metadata.schema.json'
name: Card
props:
  type: object
  properties:
    title:
      type: string
    image_url:
      type: string
```

```twig
{# Utiliser un SDC dans un template Drupal #}
{% component 'mon_theme:card' with { title: node.label, image_url: image_url } %}
```

Activer : `ddev drush en sdc -y` (module core en D10.3+, stable D11)
