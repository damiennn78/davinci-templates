# DaVinci Resolve Fusion — Référence complète des fichiers `.setting` et templates

> Documentation consolidée pour la création de macros, templates et bundles `.drfx`  
> Sources : documentation officielle Blackmagic Design, VFXPedia, forums We Suck Less, fichiers de ce projet

---

## Table des matières

1. [Vue d'ensemble](#1-vue-densemble)
2. [Structure du fichier `.setting`](#2-structure-du-fichier-setting)
3. [MacroOperator vs GroupOperator](#3-macrooperator-vs-groupoperator)
4. [InstanceInput — exposer des paramètres](#4-instanceinput--exposer-des-paramètres)
5. [InstanceOutput — exposer des sorties](#5-instanceoutput--exposer-des-sorties)
6. [UserControls — contrôles personnalisés](#6-usercontrols--contrôles-personnalisés)
7. [Types de nœuds (Tools)](#7-types-de-nœuds-tools)
8. [Animations : BezierSpline et PolyPath](#8-animations--bezierspline-et-polypath)
9. [Référencer des assets locaux](#9-référencer-des-assets-locaux)
10. [Expressions et liaisons dynamiques](#10-expressions-et-liaisons-dynamiques)
11. [Structure du bundle `.drfx`](#11-structure-du-bundle-drfx)
12. [Icônes de prévisualisation](#12-icônes-de-prévisualisation)
13. [Chemins d'installation](#13-chemins-dinstallation)
14. [Exemples complets annotés](#14-exemples-complets-annotés)

---

## 1. Vue d'ensemble

Un fichier `.setting` est un fichier **texte au format Lua-like** (table imbriquées avec `{}` et `ordered()`) utilisé par Blackmagic Fusion / DaVinci Resolve pour stocker :

- des **compositions Fusion** entières (nœuds, connexions, valeurs, courbes d'animation)
- des **macros** (boîte noire avec inputs/outputs exposés)
- des **groupes éditables** (idem mais nœuds internes accessibles)

Le fichier peut être ouvert dans n'importe quel éditeur de texte (VS Code, Notepad++…). Éviter le Bloc-notes Windows qui gère mal les retours à la ligne Fusion.

**Extension** : `.setting` (sans "s")  
**Encoding** : UTF-8  
**Format** : table Lua imbriquée

---

## 2. Structure du fichier `.setting`

### Squelette minimal

```lua
{
    Tools = ordered() {
        NomDeLaMacro = MacroOperator {          -- ou GroupOperator
            UserControls = ordered() { ... },   -- optionnel : contrôles custom
            Inputs = ordered() { ... },          -- inputs exposés
            Outputs = { ... },                   -- outputs exposés
            ViewInfo = GroupInfo { Pos = { 0, 0 } },
            Tools = ordered() { ... },           -- nœuds internes
        }
    },
    ActiveTool = "NomDeLaMacro"
}
```

### Champs racine

| Champ | Type | Description |
|-------|------|-------------|
| `Tools` | `ordered()` | Table ordonnée de tous les outils à la racine |
| `ActiveTool` | `string` | Nom de l'outil sélectionné à l'ouverture |

### Propriétés communes à tous les nœuds

| Propriété | Type | Description |
|-----------|------|-------------|
| `NameSet` | `bool` | `true` = le nom a été défini manuellement |
| `CtrlWZoom` | `bool` | `false` = contrôle caché dans le viewer Fusion |
| `Inputs` | `table` | Paramètres d'entrée du nœud |
| `ViewInfo` | `OperatorInfo` | Position du nœud dans le flow `{ Pos = { x, y } }` |
| `EnabledRegion` | `TimeRegion` | Région temporelle d'activation du nœud |

---

## 3. MacroOperator vs GroupOperator

| | `MacroOperator` | `GroupOperator` |
|---|---|---|
| Nœuds internes visibles | Non (boîte noire) | Oui (double-clic pour entrer) |
| Modification dans Resolve | Impossible | Possible |
| Usage typique | Distribution finale | Développement / templates éditables |
| Conversion | Remplacer `MacroOperator` par `GroupOperator` dans le fichier texte |

**Dans ce projet** : les templates utilisent `GroupOperator` pour rester éditables.

```lua
-- Changer en boîte noire :
NomDeLaMacro = MacroOperator { ... }

-- Garder éditable :
NomDeLaMacro = GroupOperator { ... }
```

---

## 4. InstanceInput — exposer des paramètres

`InstanceInput` relie un paramètre d'un nœud interne à l'interface externe de la macro/groupe.

### Syntaxe complète

```lua
InputN = InstanceInput {
    SourceOp = "NomDuNoeudInterne",  -- obligatoire : nom exact du nœud
    Source   = "NomDuParametre",     -- obligatoire : nom exact du paramètre Fusion
    Name     = "Libellé affiché",    -- optionnel : surcharge le nom d'affichage
    Default  = 0.5,                  -- optionnel : valeur par défaut
    ControlGroup = 3,                -- optionnel : regroupe des contrôles liés (même numéro = groupé)
    Page     = "Controls",           -- optionnel : nom de l'onglet dans l'inspecteur
    MinScale = 0,                    -- optionnel : minimum du slider
    MaxScale = 1,                    -- optionnel : maximum du slider
}
```

### Propriétés détaillées

| Propriété | Obligatoire | Description |
|-----------|-------------|-------------|
| `SourceOp` | Oui | Nom exact du nœud interne source |
| `Source` | Oui | Nom exact du paramètre (ex : `"StyledText"`, `"Size"`, `"Center"`) |
| `Name` | Non | Nom affiché dans l'inspecteur (surcharge le nom du paramètre) |
| `Default` | Non | Valeur par défaut ; écrase la valeur définie dans le nœud interne |
| `ControlGroup` | Non | Entier : les inputs avec le même numéro sont groupés visuellement (ex : `Font` + `Style`) |
| `Page` | Non | Nom de l'onglet dans lequel le contrôle apparaît |
| `MinScale` | Non | Valeur minimale du slider |
| `MaxScale` | Non | Valeur maximale du slider |

### Paramètres courants des nœuds TextPlus

| `Source` | Description |
|----------|-------------|
| `StyledText` | Contenu textuel |
| `Font` | Famille de police |
| `Style` | Style (Bold, Italic…) |
| `Size` | Taille du texte (0–1, typiquement 0.05–0.15) |
| `CharacterSpacingClone` | Tracking (espacement entre caractères) |
| `Center` | Position XY |
| `Red1` / `Green1` / `Blue1` / `Alpha1` | Couleur du style 1 |
| `Opacity1` | Opacité du style 1 |
| `VerticalJustificationNew` | Justification verticale (1=haut, 2=centre, 3=bas) |
| `HorizontalJustificationNew` | Justification horizontale (1=gauche, 2=centre, 3=droite) |

### Exemple tiré de ce projet (TITRE_AVQN)

```lua
Input1 = InstanceInput {
    SourceOp = "TEXT_1",
    Source = "StyledText",
},
Input4 = InstanceInput {
    SourceOp = "TEXT_1",
    Source = "Size",
    Default = 0.08,
},
Input5 = InstanceInput {
    SourceOp = "TEXT_1",
    Source = "CharacterSpacingClone",
    Name = "Tracking",    -- renomme le paramètre
    Default = 1,
},
-- ControlGroup groupe Font + Style ensemble dans l'UI
Input2 = InstanceInput {
    SourceOp = "TEXT_1",
    Source = "Font",
    ControlGroup = 2,
},
Input3 = InstanceInput {
    SourceOp = "TEXT_1",
    Source = "Style",
    ControlGroup = 2,    -- même groupe que Font
},
```

---

## 5. InstanceOutput — exposer des sorties

Expose une sortie interne vers l'extérieur de la macro/groupe.

```lua
Outputs = {
    MainOutput1 = InstanceOutput {
        SourceOp = "MediaOut1",   -- nœud interne
        Source = "Output",        -- sortie de ce nœud
    },
    Output1 = InstanceOutput {
        SourceOp = "Path1",
        Source = "Heading",       -- sortie de chemin pour animation
    },
}
```

| Propriété | Description |
|-----------|-------------|
| `SourceOp` | Nom du nœud interne dont on expose la sortie |
| `Source` | Nom de la sortie (`"Output"`, `"Heading"`, `"Position"`…) |

**Convention de nommage** :
- `MainOutput1` = sortie vidéo principale (reconnue par Resolve)
- `Output1`, `Output2`… = sorties supplémentaires

---

## 6. UserControls — contrôles personnalisés

`UserControls` permet d'ajouter des contrôles **indépendants des nœuds internes** dans l'inspecteur. Ils sont accessibles via des expressions dans les nœuds internes.

### Structure générale

```lua
UserControls = ordered() {
    nom_variable = {
        LINKS_Name       = "Libellé affiché",
        LINKID_DataType  = "Number",        -- type de donnée
        INPID_InputControl = "SliderControl", -- type de widget
        INP_Default      = 0.5,
        INP_MinScale     = 0,
        INP_MaxScale     = 1,
        ICD_Width        = 1,               -- largeur (0–1)
    },
}
```

> La valeur du contrôle est accessible dans les expressions via `NomDeLaMacro.nom_variable`

### 6.1 Types de données (`LINKID_DataType`)

| Valeur | Description |
|--------|-------------|
| `"Number"` | Valeur numérique (entière ou décimale) |
| `"Text"` | Chaîne de caractères |
| `"Point"` | Coordonnées XY |
| `"Image"` | Image |

### 6.2 Types de contrôles (`INPID_InputControl`)

#### SliderControl

```lua
vitesse = {
    LINKS_Name         = "Vitesse",
    LINKID_DataType    = "Number",
    INPID_InputControl = "SliderControl",
    INP_Default        = 1,
    INP_MinScale       = 0,
    INP_MaxScale       = 10,
    INP_Integer        = false,   -- true = entiers seulement
    ICD_Width          = 1,
}
```

| Propriété | Description |
|-----------|-------------|
| `INP_Default` | Valeur par défaut |
| `INP_MinScale` | Borne inférieure du slider |
| `INP_MaxScale` | Borne supérieure du slider |
| `INP_MinAllowed` | Valeur minimale absolue autorisée |
| `INP_MaxAllowed` | Valeur maximale absolue autorisée |
| `INP_Integer` | `true` = pas entier uniquement |

---

#### CheckboxControl

```lua
afficher_logo = {
    LINKS_Name         = "Afficher logo",
    LINKID_DataType    = "Number",
    INPID_InputControl = "CheckboxControl",
    INP_Default        = 1,     -- 1 = coché, 0 = décoché
    ICD_Width          = 1,
}
```

---

#### ComboControl (liste déroulante)

```lua
position_logo = {
    LINKS_Name         = "Position logo",
    LINKID_DataType    = "Number",
    INPID_InputControl = "ComboControl",
    CC_Items           = {
        "Gauche",    -- index 0
        "Centre",    -- index 1
        "Droite",    -- index 2
    },
    INP_Default        = 0,     -- index de l'item par défaut
    ICD_Width          = 1,
}
```

La valeur retournée est l'**index** (0, 1, 2…) de l'item sélectionné.

---

#### MultiButtonControl

```lua
style_bouton = {
    LINKS_Name         = "Style",
    LINKID_DataType    = "Number",
    INPID_InputControl = "MultiButtonControl",
    INP_Default        = 0,
    MBTNC_AddButton    = "Option A",
    MBTNC_AddButton    = "Option B",
    MBTNC_AddButton    = "Option C",
    MBTNC_ShowName     = true,
    MBTNC_StretchToFit = true,
    ICD_Width          = 1,
}
```

---

#### LabelControl (séparateur / section repliable)

```lua
Section_Texte = {
    LINKS_Name         = "Texte principal",
    LINKID_DataType    = "Number",
    INPID_InputControl = "LabelControl",
    INP_Integer        = false,
    LBLC_DropDownButton = true,   -- true = section repliable (accordéon)
    LBLC_NumInputs     = 3,       -- nombre de contrôles suivants inclus dans la section
    INP_Default        = 1,       -- 1 = déplié par défaut, 0 = replié
    ICD_Width          = 1,
}
```

`LBLC_NumInputs` indique combien de contrôles **suivants** dans `UserControls` sont groupés sous cette section.

---

#### TextEditControl

```lua
mon_texte = {
    LINKS_Name         = "Texte",
    LINKID_DataType    = "Text",
    INPID_InputControl = "TextEditControl",
    TEC_ReadOnly       = false,    -- true = lecture seule
    TEC_Lines          = 3,        -- hauteur en lignes
    INP_Default        = "Texte par défaut",
    ICD_Width          = 1,
}
```

---

#### ColorControl

```lua
couleur_fond = {
    LINKS_Name         = "Couleur fond",
    LINKID_DataType    = "Number",   -- répété 4 fois pour RGBA
    INPID_InputControl = "ColorControl",
    ICD_Width          = 1,
}
```

---

#### RangeControl

```lua
plage_temps = {
    LINKS_Name         = "Plage de temps",
    LINKID_DataType    = "Number",
    INPID_InputControl = "RangeControl",
    INP_Default        = 0,
    INP_MinScale       = 0,
    INP_MaxScale       = 100,
    ICD_Width          = 1,
}
```

---

#### ScrewControl (molette infinie)

```lua
angle = {
    LINKS_Name         = "Angle",
    LINKID_DataType    = "Number",
    INPID_InputControl = "ScrewControl",
    INP_Default        = 0,
    ICD_Width          = 1,
}
```

---

### 6.3 Propriétés de layout communes

| Propriété | Description |
|-----------|-------------|
| `ICD_Width` | Largeur du contrôle (0 à 1, `1` = pleine largeur) |
| `ICS_ControlPage` | Nom de la page/onglet dans l'inspecteur (ex : `"Controls"`) |
| `INP_DoNotifyChanged` | `false` = ne notifie pas les changements |
| `PC_Visible` | `false` = contrôle caché |

---

## 7. Types de nœuds (Tools)

### Nœuds fréquents et leurs paramètres clés

#### Background

```lua
MonBackground = Background {
    NameSet = true,
    Inputs = {
        Width                    = Input { Value = 1920, },
        Height                   = Input { Value = 1080, },
        UseFrameFormatSettings   = Input { Value = 1, },   -- 1 = utilise les réglages du projet
        TopLeftRed               = Input { Value = 0.2, },
        TopLeftGreen             = Input { Value = 0.2, },
        TopLeftBlue              = Input { Value = 0.2, },
        TopLeftAlpha             = Input { Value = 1, },
        ["Gamut.SLogVersion"]    = Input { Value = FuID { "SLog2" }, },
    },
    ViewInfo = OperatorInfo { Pos = { 100, 50 } },
}
```

---

#### TextPlus

```lua
MON_TEXTE = TextPlus {
    NameSet = true,
    Inputs = {
        Width                       = Input { Value = 1920, },
        Height                      = Input { Value = 1080, },
        UseFrameFormatSettings      = Input { Value = 1, },
        StyledText                  = Input { Value = "Mon texte", },
        Font                        = Input { Value = "General Sans", },
        Style                       = Input { Value = "Bold", },
        Size                        = Input { Value = 0.08, },
        VerticalJustificationNew    = Input { Value = 3, },  -- 1=top 2=mid 3=bot
        HorizontalJustificationNew  = Input { Value = 3, },  -- 1=left 2=center 3=right
        Center                      = Input { Value = { 0.5, 0.5 }, },
        Wrap                        = Input { Value = 1, },
        Red1                        = Input { Value = 1, },
        Green1                      = Input { Value = 1, },
        Blue1                       = Input { Value = 1, },
        Opacity1                    = Input { Value = 1, },
        CharacterSpacingClone       = Input { Value = 1, },
        Softness1                   = Input { Value = 1, },
    },
    ViewInfo = OperatorInfo { Pos = { 200, 100 } },
}
```

---

#### Loader (image statique ou séquence)

```lua
MON_IMAGE = Loader {
    Clips = {
        Clip {
            ID           = "Clip1",
            Filename     = "setting:Assets\\monimage.png",  -- chemin relatif au .setting
            FormatID     = "PNGFormat",
            StartFrame   = -1,
            LengthSetManually = true,
            TrimIn       = 0,
            TrimOut      = 0,
            ExtendFirst  = 0,
            ExtendLast   = 0,
            Loop         = 0,
            AspectMode   = 0,
            Depth        = 0,
            TimeCode     = 0,
            GlobalStart  = 0,
            GlobalEnd    = 0,
        }
    },
    NameSet = true,
    Inputs = {
        ["Gamut.SLogVersion"]          = Input { Value = FuID { "SLog2" }, },
        ["Clip1.PNGFormat.PostMultiply"] = Input { Value = 1, },
    },
    ViewInfo = OperatorInfo { Pos = { 50, 80 } },
}
```

---

#### Merge

```lua
Merge1 = Merge {
    Inputs = {
        Background = Input {
            SourceOp = "MonBackground",
            Source   = "Output",
        },
        Foreground = Input {
            SourceOp = "MON_TEXTE",
            Source   = "Output",
        },
        Blend               = Input { Value = 1, },
        PerformDepthMerge   = Input { Value = 0, },
    },
    ViewInfo = OperatorInfo { Pos = { 300, 100 } },
}
```

---

#### Transform

```lua
Transform1 = Transform {
    NameSet = true,
    Inputs = {
        Center = Input { Value = { 0.5, 0.5 }, },
        Size   = Input { Value = 1, },
        Angle  = Input { Value = 0, },
        Input  = Input {
            SourceOp = "Merge1",
            Source   = "Output",
        },
    },
    ViewInfo = OperatorInfo { Pos = { 400, 100 } },
}
```

---

#### Glow

```lua
EFFET_GLOW = Glow {
    NameSet = true,
    Inputs = {
        Filter    = Input { Value = FuID { "Fast Gaussian" }, },
        XGlowSize = Input { Value = 7.9, },
        YGlowSize = Input { Value = 10, },
        Glow      = Input { Value = 0.748, },
        Blend     = Input { Value = 0.2, },
        Input     = Input {
            SourceOp = "MON_TEXTE",
            Source   = "Output",
        },
    },
    ViewInfo = OperatorInfo { Pos = { 250, 100 } },
}
```

---

#### MediaOut (sortie vers la timeline)

```lua
MediaOut1 = MediaOut {
    CtrlWZoom = false,
    Inputs = {
        Index = Input { Value = "0", },    -- "0" = sortie principale
        Input = Input {
            SourceOp = "Merge1",
            Source   = "Output",
        },
    },
    ViewInfo = OperatorInfo { Pos = { 500, 100 } },
}
```

---

### Syntaxe d'une connexion entre nœuds

```lua
-- Connexion simple : SourceOp -> Source
NomDuParametre = Input {
    SourceOp = "NomDuNoeudSource",
    Source   = "Output",   -- ou "Position", "Heading", "Value"...
}

-- Valeur fixe :
NomDuParametre = Input { Value = 42, }

-- Valeur point (XY) :
Center = Input { Value = { 0.5, 0.5 }, }

-- Valeur FuID (identifiant Fusion) :
Filter = Input { Value = FuID { "Fast Gaussian" }, }

-- Expression :
Blend = Input { Expression = "MonGroupe.ma_variable", }
```

---

## 8. Animations : BezierSpline et PolyPath

### BezierSpline — courbe d'animation sur un paramètre numérique

```lua
MonAnimSpline = BezierSpline {
    SplineColor = { Red = 255, Green = 0, Blue = 255 },  -- couleur dans le spline editor
    CtrlWZoom   = false,
    KeyFrames   = {
        [0]  = { 0, RH = { 10, 0.333 }, Flags = { Linear = true, LockedY = true } },
        [30] = { 1, LH = { 20, 0.667 }, Flags = { Linear = true, LockedY = true } },
    }
}
```

| Champ KeyFrame | Description |
|----------------|-------------|
| `[frame]` | Numéro de frame |
| `{ valeur, ... }` | Valeur à cette frame |
| `RH` | Handle droit `{ frame, valeur }` |
| `LH` | Handle gauche `{ frame, valeur }` |
| `Flags.Linear` | Interpolation linéaire |
| `Flags.LockedY` | Bloque le handle verticalement |

Connexion du spline vers un paramètre :

```lua
Displacement = Input {
    SourceOp = "MonAnimSpline",
    Source   = "Value",
}
```

---

### PolyPath — chemin d'animation pour un déplacement XY

```lua
MonChemin = PolyPath {
    DrawMode = "InsertAndModify",
    CtrlWZoom = false,
    Inputs = {
        Displacement = Input {
            SourceOp = "MonAnimSpline",   -- avancement sur le chemin (0→1)
            Source   = "Value",
        },
        PolyLine = Input {
            Value = Polyline {
                Points = {
                    { Linear = true, LockY = true, X = -0.5, Y = 0, RX = 0.1, RY = 0 },
                    { Linear = true, LockY = true, X =  0.0, Y = 0, LX = -0.1, LY = 0 },
                }
            },
        }
    },
}
```

Connexion du chemin vers un paramètre :

```lua
Center = Input {
    SourceOp = "MonChemin",
    Source   = "Position",   -- ou "Heading" pour l'angle
}
```

---

### EnabledRegion — activer un nœud sur une plage temporelle

```lua
MonNoeud = Transform {
    EnabledRegion = TimeRegion { { Start = 14, End = 112, FrameLength = 1 } },
    Inputs = { ... },
}
```

---

## 9. Référencer des assets locaux

Pour inclure des images, logos ou autres fichiers dans un template, les placer dans le dossier `Assets/` à côté du `.setting`.

**Structure attendue :**

```
Edit/
  Titles/
    MonTemplate.setting
    MonTemplate.png        ← icône de prévisualisation (optionnel)
    Assets/
      fond.png
      logo.png
      police.ttf
```

**Référence dans le fichier `.setting` :**

```lua
Filename = "setting:Assets\\fond.png"
--         ^^^^^^^^^
--         Chemin relatif au dossier contenant le .setting
```

| Préfixe | Description |
|---------|-------------|
| `setting:` | Relatif au dossier du fichier `.setting` |
| Chemin absolu | Déconseillé (non portable) |

---

## 10. Expressions et liaisons dynamiques

Les expressions permettent de lier la valeur d'un paramètre à un `UserControl` ou à un autre paramètre.

### Syntaxe

```lua
NomParametre = Input { Expression = "NomDeLaMacro.nom_variable", }
```

### Exemples

```lua
-- Lier la visibilité d'un nœud à une checkbox UserControl
Blend = Input { Expression = "INTERTITREmodif.afficher_logo", }

-- Positionner un élément selon un ComboControl (gauche/droite)
Center = Input {
    Expression = "Point(iif(INTERTITREmodif.position_logo < 0.5, -1.872, 1.872), 4.588)",
}

-- Référencer un paramètre d'un autre nœud
Size = Input { Expression = "TEXT_1.Size * 0.5", }
```

### Fonctions d'expression utiles

| Fonction | Description |
|----------|-------------|
| `iif(condition, valTrue, valFalse)` | Ternaire |
| `Point(x, y)` | Crée une valeur XY |
| `time` | Frame actuelle |
| `comp:GetAttrs().COMPN_CurrentTime` | Temps de composition |

---

## 11. Structure du bundle `.drfx`

Un `.drfx` est simplement un **fichier ZIP renommé** qui mime la structure des dossiers Fusion.

### Arborescence complète

```
MonBundle.drfx  (= MonBundle.zip renommé)
├── Edit/
│   ├── Titles/
│   │   ├── MonTitre.setting
│   │   ├── MonTitre.png          ← icône 104×58px
│   │   └── Assets/
│   │       ├── fond.png
│   │       └── logo.png
│   ├── Effects/
│   │   └── MonEffet.setting
│   ├── Generators/
│   │   └── MonGenerateur.setting
│   └── Transitions/
│       └── MaTransition.setting
└── Fusion/
    ├── Templates/
    │   └── MonTemplate.setting
    └── Macros/
        └── MonMacro.setting
```

### Créer un `.drfx`

1. Organiser les fichiers dans la structure ci-dessus
2. Sélectionner le(s) dossier(s) racine (`Edit/` et/ou `Fusion/`)
3. Compresser en ZIP (clic droit → Envoyer vers → Dossier compressé)
4. Renommer l'extension `.zip` → `.drfx`

**Ou en ligne de commande PowerShell :**

```powershell
# Depuis le dossier contenant Edit/ et/ou Fusion/
Compress-Archive -Path "Edit", "Fusion" -DestinationPath "MonBundle.zip"
Rename-Item "MonBundle.zip" "MonBundle.drfx"
```

### Installation d'un `.drfx`

- Double-clic sur le fichier → DaVinci Resolve s'ouvre avec un dialogue d'installation
- Ou glisser-déposer dans la page Fusion de Resolve

> **Attention** : le `.drfx` n'est pas décompressé à l'installation. Supprimer le `.drfx` supprime tous les templates qu'il contient.

---

## 12. Icônes de prévisualisation

| Propriété | Valeur |
|-----------|--------|
| Format | PNG |
| Résolution recommandée | 104 × 58 px |
| Nommage | Même nom que le `.setting` avec extension `.png` |
| Emplacement | Même dossier que le `.setting` |

Exemple :

```
Titles/
  MonTitre.setting
  MonTitre.png     ← 104×58px, affiché dans la bibliothèque d'effets
```

---

## 13. Chemins d'installation

### Windows

| Type | Chemin |
|------|--------|
| Macros utilisateur | `%APPDATA%\Blackmagic Design\DaVinci Resolve\Fusion\Macros\` |
| Templates Edit/Titles | `%APPDATA%\Blackmagic Design\DaVinci Resolve\Fusion\Templates\Edit\Titles\` |
| Templates Edit/Effects | `%APPDATA%\Blackmagic Design\DaVinci Resolve\Fusion\Templates\Edit\Effects\` |
| Templates Edit/Transitions | `%APPDATA%\Blackmagic Design\DaVinci Resolve\Fusion\Templates\Edit\Transitions\` |
| Templates Edit/Generators | `%APPDATA%\Blackmagic Design\DaVinci Resolve\Fusion\Templates\Edit\Generators\` |

### macOS

| Type | Chemin |
|------|--------|
| Macros utilisateur | `~/Library/Application Support/Blackmagic Design/DaVinci Resolve/Fusion/Macros/` |
| Templates | `~/Library/Application Support/Blackmagic Design/DaVinci Resolve/Fusion/Templates/Edit/Titles/` |

---

## 14. Exemples complets annotés

### Exemple 1 : Template minimaliste avec texte et checkbox

```lua
{
    Tools = ordered() {
        -- GroupOperator = éditable dans Resolve
        MonTemplate = GroupOperator {

            -- Contrôles personnalisés dans l'inspecteur
            UserControls = ordered() {
                afficher_sous_titre = {
                    LINKS_Name         = "Afficher sous-titre",
                    LINKID_DataType    = "Number",
                    INPID_InputControl = "CheckboxControl",
                    INP_Default        = 1,
                    ICD_Width          = 1,
                },
            },

            -- Paramètres exposés à l'utilisateur
            Inputs = ordered() {
                Input1 = InstanceInput {
                    SourceOp = "TITRE",
                    Source   = "StyledText",
                },
                Input2 = InstanceInput {
                    SourceOp = "TITRE",
                    Source   = "Font",
                    ControlGroup = 2,
                },
                Input3 = InstanceInput {
                    SourceOp = "TITRE",
                    Source   = "Style",
                    ControlGroup = 2,
                },
                Input4 = InstanceInput {
                    SourceOp = "TITRE",
                    Source   = "Size",
                    Default  = 0.1,
                },
            },

            -- Sortie principale
            Outputs = {
                MainOutput1 = InstanceOutput {
                    SourceOp = "MediaOut1",
                    Source   = "Output",
                },
            },

            ViewInfo = GroupInfo { Pos = { 0, 0 } },

            -- Nœuds internes
            Tools = ordered() {
                FOND = Background {
                    NameSet = true,
                    Inputs = {
                        Width                  = Input { Value = 1920, },
                        Height                 = Input { Value = 1080, },
                        UseFrameFormatSettings = Input { Value = 1, },
                        TopLeftAlpha           = Input { Value = 0, },
                    },
                    ViewInfo = OperatorInfo { Pos = { 100, 50 } },
                },
                TITRE = TextPlus {
                    NameSet = true,
                    Inputs = {
                        Width                      = Input { Value = 1920, },
                        Height                     = Input { Value = 1080, },
                        UseFrameFormatSettings     = Input { Value = 1, },
                        StyledText                 = Input { Value = "Mon titre", },
                        Font                       = Input { Value = "Open Sans", },
                        Style                      = Input { Value = "Bold", },
                        Size                       = Input { Value = 0.1, },
                        VerticalJustificationNew   = Input { Value = 2, },
                        HorizontalJustificationNew = Input { Value = 2, },
                    },
                    ViewInfo = OperatorInfo { Pos = { 200, 50 } },
                },
                Merge1 = Merge {
                    Inputs = {
                        Background = Input { SourceOp = "FOND",  Source = "Output", },
                        Foreground = Input { SourceOp = "TITRE", Source = "Output", },
                        PerformDepthMerge = Input { Value = 0, },
                    },
                    ViewInfo = OperatorInfo { Pos = { 350, 50 } },
                },
                MediaOut1 = MediaOut {
                    CtrlWZoom = false,
                    Inputs = {
                        Index = Input { Value = "0", },
                        Input = Input { SourceOp = "Merge1", Source = "Output", },
                    },
                    ViewInfo = OperatorInfo { Pos = { 500, 50 } },
                },
            },
        }
    },
    ActiveTool = "MonTemplate"
}
```

---

### Exemple 2 : Template avec asset local et ComboControl (tiré de ce projet)

Voir `TITRE_TEST/Edit/Titles/ManuTest.setting` :

- `FOND_IMG` : charge `setting:Assets\fond.png` (relatif au `.setting`)
- `LOGO_IMG` : charge `setting:Assets\logo.png`, avec `Blend` lié au UserControl `afficher_logo`
- `Transform1` : position du logo liée au UserControl `position_logo` via expression `iif(...)`
- `UserControls` : `CheckboxControl` + `ComboControl` exposés dans l'inspecteur

---

### Exemple 3 : Animation avec PolyPath + BezierSpline (tiré de ce projet)

Voir `TITRE_AVQN/Edit/Titles/TITRE_AVQN.setting` :

```lua
Path1 = PolyPath {
    DrawMode = "InsertAndModify",
    CtrlWZoom = false,
    Inputs = {
        -- Avancement animé sur le chemin
        Displacement = Input {
            SourceOp = "Path1Displacement",
            Source   = "Value",
        },
        -- Définition du chemin (de X=-0.537 à X=0)
        PolyLine = Input {
            Value = Polyline {
                Points = {
                    { Linear = true, LockY = true, X = -0.537, Y = 0, RX = 0.179, RY = 0 },
                    { Linear = true, LockY = true, X =  0.000, Y = 0, LX = -0.179, LY = 0 },
                }
            },
        }
    },
},
-- Courbe d'animation : frames 16→22, valeur 0→1
Path1Displacement = BezierSpline {
    SplineColor = { Red = 255, Green = 0, Blue = 255 },
    CtrlWZoom   = false,
    KeyFrames   = {
        [16] = { 0, RH = { 18, 0.333 }, Flags = { Linear = true, LockedY = true } },
        [22] = { 1, LH = { 20, 0.667 }, Flags = { Linear = true, LockedY = true } },
    }
},
```

Le texte se déplace le long du chemin entre les frames 16 et 22.

---

## Récapitulatif rapide

```
.setting
├── Tools = ordered()
│   └── NomMacro = GroupOperator / MacroOperator
│       ├── UserControls = ordered()       ← contrôles UI personnalisés
│       │   └── ma_var = { LINKS_Name, LINKID_DataType, INPID_InputControl, INP_Default, ... }
│       ├── Inputs = ordered()             ← paramètres exposés depuis les nœuds internes
│       │   └── InputN = InstanceInput { SourceOp, Source, Name, Default, ControlGroup, Page }
│       ├── Outputs = {}                   ← sorties exposées
│       │   └── MainOutput1 = InstanceOutput { SourceOp, Source }
│       ├── ViewInfo = GroupInfo { Pos = { 0, 0 } }
│       └── Tools = ordered()             ← nœuds internes
│           ├── Background { Inputs = { Width, Height, TopLeftRed/Green/Blue/Alpha } }
│           ├── TextPlus { Inputs = { StyledText, Font, Style, Size, ... } }
│           ├── Loader { Clips = { Clip { Filename = "setting:Assets\\..." } } }
│           ├── Merge { Inputs = { Background, Foreground, Blend } }
│           ├── Transform { Inputs = { Center, Size, Angle } }
│           ├── Glow { Inputs = { Filter, XGlowSize, Glow, Blend } }
│           ├── PolyPath { Inputs = { Displacement, PolyLine } }
│           ├── BezierSpline { KeyFrames = { [frame] = { value } } }
│           └── MediaOut { Inputs = { Index = "0", Input } }
└── ActiveTool = "NomMacro"
```

---

## Sources

- [Build new Fusion tools with Macros and Templates — VFXstudy](https://vfxstudy.com/tutorials/macros-templates/)
- [Fusion Templates with included Media and custom Icons — VFXstudy](https://vfxstudy.com/tutorials/templates-assets-bundles/)
- [Using Fusion Template Bundles — VFXPedia / Fusion 18 Manual](https://www.steakunderwater.com/VFXPedia/__man/Fusion18-6/Fusion18_Manual_files/part237.htm)
- [Blackmagic Fusion: Macros — Bryan Ray](https://bryanray.name/2017/05/19/blackmagic-fusion-macros/)
- [Blackmagic Forum — Grouping of macro inputs](https://www.steakunderwater.com/wesuckless/viewtopic.php?t=4654)
- [Custom UserControls Guide Needed — We Suck Less](https://www.steakunderwater.com/wesuckless/viewtopic.php?t=4823)
- [Fusion QuickTip #001: Add and Edit Custom Controls — Noah Hähnel](https://noahhaehnel.com/blog/fusion-quicktip-001/)
- [pysion — Python library to create Fusion comps](https://github.com/brunocbreis/pysion)
- [Fusion 9 User Manual — Blackmagic Design](https://documents.blackmagicdesign.com/UserManuals/Fusion9_Manual.pdf)
- [Fusion 8 Scripting Guide — Blackmagic Design](https://documents.blackmagicdesign.com/UserManuals/Fusion8_Scripting_Guide.pdf)
- [Fusion Fuse SDK — Blackmagic Design](https://documents.blackmagicdesign.com/UserManuals/Fusion_Fuse_SDK.pdf)
- Fichiers de ce projet : `TITRE_AVQN.setting`, `ManuTest.setting`
