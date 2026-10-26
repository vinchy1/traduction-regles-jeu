# traduction-regles-jeu — skill Claude Code

> **Dépôt archivé.** Cette skill fait désormais partie du plugin Claude Code [**regles-jeu**](https://github.com/vinchy1/regles-jeu) (skills `rechercher` et `traduire`), avec tout son historique. Installation : `/plugin marketplace add vinchy1/regles-jeu`, puis `/plugin install regles-jeu@regles-jeu`.

Traduit en français un livret de règles de jeu de société au format PDF, depuis **n'importe quelle langue**
(anglais, allemand, espagnol, italien, japonais…), **en gardant exactement la mise en page** : polices,
illustrations, icônes dans le texte, encadrés, cartes reproduites, positions. Le résultat est un `<nom>_FR.pdf`
prêt à imprimer, rangé à côté de l'original.

Il suffit de déposer le PDF dans un dossier et de demander à Claude Code « traduis-moi ce livret ». La skill fait
le reste : préparation, traduction, construction du PDF, contrôles, relecture, rangement.

Testée sur des livrets de jeux de plis et de cartes : Trick to the Future, Numberwang±, Fright Factor,
Dr. Science, Tricklings / Vazerneiros, Merge Take, Pack Tricks, The Good Farmer, Poker Avoces et Fagioli (anglais ;
Fagioli est un jeu japonais), This is from that customer over there (japonais), et quatre livrets **scannés** : Shinobi Blade (japonais),
After You Betobeto-san, The Final Bosses' Blues et Man Eating House (anglais).

---

## Sommaire
- [Ce qu'elle fait](#ce-quelle-fait)
- [Installation](#installation)
- [Utilisation](#utilisation)
- [Organisation des dossiers](#organisation-des-dossiers)
- [Déroulement d'une traduction](#déroulement-dune-traduction)
- [Livrets scannés](#livrets-scannés)
- [Règles sur une page web](#règles-sur-une-page-web)
- [Contrôles automatiques et relecture](#contrôles-automatiques-et-relecture)
- [Coût en jetons et modèles](#coût-en-jetons-et-modèles)
- [Corriger une traduction déjà faite](#corriger-une-traduction-déjà-faite)
- [Contenu du dépôt](#contenu-du-dépôt)
- [Limites connues](#limites-connues)
- [Remarques](#remarques)

---

## Ce qu'elle fait
- **Détecte la langue** du livret (pas besoin de la préciser) et l'annonce ; ne pose une question que si le
  livret est multilingue ou déjà en partie en français.
- **Retire le texte d'origine** du PDF sans toucher aux images ni aux dessins vectoriels (traits de
  soulignement compris), et sans décaler le texte voisin ni faire ressortir des images cachées (pièges de
  certains livrets InDesign, protégés automatiquement).
- **Pose le texte français au même endroit**, avec des polices libres proches de l'originale (les polices
  embarquées n'ont presque jamais tous les accents), et réduit légèrement la taille si le français est plus long.
- **Trouve seule la mise en page** : colonnes (jusqu'à 4 par page), étapes numérotées posées sur des pastilles,
  paragraphes qui reprennent sous une image.
- **Replace les icônes intégrées au texte** (symboles de couleur, étoiles, lettres cerclées…) au bon endroit
  dans la phrase française.
- **Traduit aussi le texte des cartes** reproduites dans le livret.
- **Respecte la typographie française** : « guillemets », espaces insécables avant ; : ! ?, décimales à virgule,
  pas de coupure dans « Bleu 19 », « joueur A », « page 3 ».
- **Garde un vocabulaire cohérent d'un jeu à l'autre** grâce à un glossaire (pli, entamer, fournir, atout,
  couleur demandée…).
- **Accepte les livrets scannés** (pages-images) : lecture automatique du texte, puis texte de l'image masqué et
  remplacé par le français, ou traduction placée à côté de chaque page.
- **Contrôle son propre travail** : texte oublié, chevauchements, débordements, fautes, termes du glossaire non
  respectés, paragraphes anormalement courts.
- **Fait aussi la version anglaise** si on la demande (utile pour un livret japonais) : la même spec contient
  les deux textes, et la livraison produit `<nom>_FR.pdf` et `<nom>_EN.pdf`.
- **Range tout** : livrets dans un dossier par jeu, outils allégés à part, index des jeux traduits.

Préférences de traduction appliquées :
- le titre du jeu et les noms de variantes ne sont jamais traduits ;
- les noms de cartes et de personnages restent dans la langue d'origine, avec le français entre parenthèses dans
  les titres (« The Bride (La Fiancée) ») ; pour un alphabet non latin, le nom imprimé en lettres latines, sinon
  le nom anglais officiel, sinon une romanisation ;
- les compromis sont toujours signalés (police réduite, phrase condensée, texte d'illustration non traduit).

---

## Installation
1. **Copier la skill** : le dossier `traduction-regles-jeu` dans `~/.claude/skills/`
   (Windows : `C:\Users\<vous>\.claude\skills\`).
2. **Installer Python 3 et les bibliothèques** :
   ```
   pip install pymupdf fonttools pyspellchecker
   ```
   - `pymupdf` : lecture et écriture des PDF ; `fonttools` : polices ; `pyspellchecker` : dictionnaire français
     de la relecture (facultatif : sans lui, seul le contrôle d'orthographe est sauté).
   - Lecture automatique des livrets scannés (Windows uniquement, moteur de lecture intégré à Windows) :
     ```
     pip install winrt-runtime winrt-Windows.Media.Ocr winrt-Windows.Globalization winrt-Windows.Graphics.Imaging winrt-Windows.Storage.Streams winrt-Windows.Foundation winrt-Windows.Foundation.Collections
     ```
     Chaque langue à lire doit être installée dans Windows (Paramètres → Heure et langue → Langue → ajouter la
     langue avec la reconnaissance optique). Le japonais n'y est pas par défaut ; le plus simple est d'installer
     seulement son module de lecture, dans un terminal administrateur :
     ```
     dism /Online /Add-Capability /CapabilityName:Language.OCR~~~ja-JP~0.0.1.0
     ```
     (`Add-WindowsCapability` échoue sous PowerShell 7 avec « Classe non enregistrée »).
3. **Installer les agents relecteurs** (facultatif mais conseillé) : copier les fichiers de `agents/` dans
   `~/.claude/agents/`, puis redémarrer Claude Code. La relecture est alors faite sur un modèle moins coûteux
   (voir [Coût en jetons et modèles](#coût-en-jetons-et-modèles)). Sans eux, la skill lance un agent ordinaire
   sur le même modèle moins coûteux : le résultat est le même, seule la configuration change.
4. **Créer le dossier de dépôt** `A traduire` dans votre ludothèque.

Facultatif : un `CLAUDE.md` à la racine de la ludothèque peut noter le chemin exact de Python ou un autre nom
de dossier de dépôt. Les chemins propres à votre machine vont là, pas dans la skill.

---

## Utilisation
Déposer le PDF dans `A traduire/`, ouvrir Claude Code dans ce dossier et demander, par exemple :

> Traduis-moi ce livret de règles : MonJeu_rules.pdf

ou simplement « traduis la règle » s'il n'y a qu'un PDF. On peut aussi donner le chemin d'un PDF ou d'un dossier
situé ailleurs.

**Plusieurs PDF dans le dossier** : Claude les liste (en regroupant ceux d'un même jeu : règles et extension,
livret et aide de jeu), indique le coût estimé (environ 150 000 à 200 000 jetons par livret) et demande s'il faut
tout traduire ou un seul livret ; il ne commence rien avant la réponse. « Tout » : les livrets sont traduits un
par un, avec un court bilan après chacun ; un livret qui pose un vrai problème (police introuvable, scan
illisible…) est mis de côté et signalé, et les autres continuent.

Pour chaque livret, Claude annonce la langue, le nombre de pages et le type
de livret (à texte ou scanné), pose une question seulement si c'est nécessaire (par exemple le rendu d'un livret
scanné), traduit, vérifie, puis range. À la fin, il résume : fichier produit, choix de vocabulaire, compromis.

**Livret seulement consultable en ligne** (lien Google Drive sans téléchargement) : donner le lien à Claude. Le
blocage du téléchargement est un choix du propriétaire : Claude ne le contourne pas. Il cherche un PDF
téléchargeable (site de l'éditeur, BoardGameGeek) ; sinon, il vous propose de demander le fichier au
propriétaire, ou de lui fournir vos propres captures des pages, qu'il assemble en PDF et traite comme un livret
scanné (`references/en-ligne.md`).

**Jeu japonais** : Claude cherche toujours d'abord la fiche du jeu sur le Tokyo Game Market (gamemarket.jp)
et le manuel de l'auteur (titre officiel, noms en lettres latines, errata). Si le livret est lui-même une
traduction (règle anglaise d'un jeu japonais, aide de jeu de fan), il la compare à l'original, signale les écarts
et traduit en suivant l'original (`references/source.md`). Pour les autres langues, il le propose.

Pour corriger une traduction existante : « dans Merge Take, remplace "la plus forte" par "de plus haute
valeur" » (voir [plus bas](#corriger-une-traduction-déjà-faite)).

---

## Organisation des dossiers
```
<Ludothèque>/
├── A traduire/                  ← déposer ici les PDF à traduire
├── Merge Take/                  ← un dossier par jeu, créé à la livraison :
│   ├── MergeTake_EN.pdf             l'original
│   └── MergeTake_FR.pdf             la traduction
├── Pack Tricks/
│   └── …
└── _outils/
    ├── LISEZMOI.md              ← index : une ligne par jeu traduit
    ├── Merge Take/              ← outils du jeu, gardés pour les corrections futures :
    │   ├── spec.py                  la traduction et ses réglages
    │   ├── NOTES.md                 langue, éditeur, polices, vocabulaire, choix, compromis
    │   ├── ff/                      polices de remplacement, allégées
    │   └── build.py, …              scripts copiés de la skill
    └── …
```
- **Les dossiers de jeu ne contiennent que les livrets.** Rien n'y est écrasé sans votre accord.
- **Nom du dossier de jeu** : le titre imprimé en lettres latines (titre anglais officiel ou romanisation pour un
  titre japonais), nettoyé pour Windows : virgules, points d'exclamation et `\ / : * ? " < > |` sont retirés
  (« After You, Betobeto-san » → « After You Betobeto-san », « Dune: Imperium » → « Dune Imperium ») ; les noms
  réservés de Windows (`CON`, `NUL`…) reçoivent « (jeu) ». Si le dossier du jeu existe déjà, il est réutilisé tel
  quel.
- **Le dossier d'outils reste léger** (de l'ordre de 0,5 à 1,5 Mo par jeu) : à la livraison, les fichiers de travail
  sont supprimés et les polices réduites aux seuls caractères du français.
- **L'index `LISEZMOI.md`** reste court même avec des centaines de jeux (une ligne : Jeu | Langue | Éditeur |
  Type | Polices | Date). Claude y cherche par mot-clé (éditeur, police) au lieu de le lire en entier, et ne
  lit le `NOTES.md` que des jeux proches : un nouveau livret du même éditeur réutilise ses polices et son
  vocabulaire.
- **Nom du fichier traduit** : la mention de langue est retirée et `_FR` ajouté à la fin
  (« MergeTake_EN.pdf » → « MergeTake_FR.pdf », « Shinobi Blade.pdf » → « Shinobi Blade_FR.pdf »,
  « Man Eating House ENG Rules.pdf » → « Man Eating House Rules_FR.pdf »). Au milieu du nom, seules les
  abréviations en capitales (EN, ENG, JP…) et les noms de langue en toutes lettres (English, Japanese…) sont
  retirés, pour ne pas toucher un mot du titre (« Take It Easy »).
- **Livret nommé en japonais, chinois…** : il est d'abord renommé en lettres latines, titre anglais officiel
  de préférence et autres mots traduits en anglais (« 夜連_説明書.pdf » → « Who follows me_Rules.pdf »,
  « コスモショーファー_JP_Rule_v1.2.pdf » → « Cosmo Chauffeur_JP_Rule_v1.2.pdf »), ce qui donne un `_FR.pdf`
  lisible partout. L'ancien nom est noté dans `NOTES.md`.
- **Retraduction** d'un livret déjà rangé dans `<Jeu>/` : les outils vont dans `_outils/<Jeu>/` et rien n'est
  déplacé.
- Si le PDF n'est ni dans `A traduire/` ni dans un dossier de jeu, les outils sont créés à côté du PDF et rien
  n'est déplacé.

---

## Déroulement d'une traduction
1. **Préparer** (`preparer.py`) : nettoie le nom du jeu, crée `_outils/<Jeu>/`, détecte un livret scanné, copie
   les scripts utiles et un modèle de `spec.py` (chemin du livret déjà rempli), fait une planche d'aperçu des
   pages, l'inventaire du texte (`blocks.txt`, icônes comprises), extrait les polices et signale les accents qui
   leur manquent, devine la langue, lit le texte des scans, et liste les jeux déjà traduits qui utilisent les
   mêmes polices.
2. **Choisir les polices** (`polices.py`) : réutiliser celles d'un jeu proche, sinon trouver des polices libres
   de même allure (comparaison visuelle et de largeur), placées dans `ff/`. Les correspondances déjà trouvées
   sont dans `references/polices.md`.
3. **Traduire** : Claude écrit `spec.py`, qui associe chaque bloc de texte d'origine à sa traduction française.
   Exemple :
   ```python
   F(0, P("b18", "Cartes Slime (numérotées) : 46<br/>Remarque : en principe, chaque combinaison…"))
   #    page ↑   ↑ bloc 18 de l'inventaire     ↑ français (<b>, <i>, <br/>, {} = icône suivante)
   ```
   La mise en page (colonnes, pastilles, insécables) est trouvée par le moteur ; le spec ne contient des
   réglages (`x1=`, `fix=`, `COLUMNS`) que pour corriger un cas raté. Pour dessiner sous le français (élargir un
   bandeau de titre trop court, remplacer un drapeau), le spec peut fournir une fonction `AVANT_TEXTE`, appelée
   entre l'effacement de l'original et la pose du français.
4. **Construire** (`build.py`) : efface le texte d'origine, pose le français et les icônes, et écrit un journal :
   police réduite, débordement, icône non replacée, **texte non traduit**, **à vérifier** (voir plus bas).
5. **Relire** : texte et mise en page, par deux agents relecteurs ; Claude vérifie chaque point signalé avant de
   corriger, reconstruit, et ne regarde que les pages modifiées.
6. **Livrer** (`livrer.py`) : Claude écrit `NOTES.md`, puis le script déplace l'original dans `<Jeu>/`, met à
   jour le chemin dans `spec.py`, allège le dossier d'outils (`ranger.py`), reconstruit le PDF final, le copie en
   `<nom>_FR.pdf` (refuse d'écraser un PDF existant sans accord) et ajoute ou met à jour la ligne de l'index.

---

## Livrets scannés
Un livret dont les pages sont des images (pas de texte sélectionnable) est détecté automatiquement. Trois rendus
au choix :
- **Recouvrir** (conseillé pour une image numérique propre) : chaque zone de texte de l'image est masquée par la
  couleur du fond mesurée autour (ou, pour des encadrés semi-transparents, la couleur dominante de la zone), et le
  français est posé dedans, la police étant réduite au besoin pour tenir.
- **Côte à côte** : chaque page d'origine reste intacte, sa traduction complète est placée à droite.
- **Recomposer** (conseillé pour un scan papier plié ou froissé) : le livret est réécrit à neuf, même découpage
  et mêmes couleurs, sans l'image du scan (`references/recomposer.md`, exemple dans `exemples/recomposer/`).

Le texte de l'image est lu automatiquement (`ocr.py`, moteur de Windows, résolution adaptée à celle du scan ;
japonais choisi seul quand le livret en contient) et regroupé en paragraphes
avec leur position (`zones.py`), ce qui évite de recopier le texte à la main. Sans la langue installée dans
Windows (souvent le cas du japonais), Claude lit les pages en images, ce qui coûte plus cher.

Pour le rendu « recouvrir », la skill sait aussi :
- poser les symboles ①❶◆★ avec une police de Windows (Segoe UI Symbol, copiée d'office) quand celle du texte
  ne les a pas ;
- découper une icône de l'original (`icone.py`) pour la remettre dans la phrase française ;
- refaire les titres sur bandeau ou sur pastille, en élargissant le bandeau si le français est plus long ;
- garder à leur taille les zones d'une seule ligne (interligne et marge resserrés d'office) ;
- poser une étiquette verticale (`rot`) ;
- sur un livret dont les lettres sont des tracés vectoriels (ni texte ni image), supprimer les lettres au lieu
  de poser un aplat (`VECTEUR = True`) : le fond d'origine reste intact sous le français.

Livrets à texte : les puces japonaises (●, 〇, ❶, ①…), les étiquettes penchées (`INCLINEES`) et les titres
aux lettres doublées d'un contour (`CONTOURS`) sont gérés par `build.py` (voir `references/pieges.md`).

Détails : `references/scan.md`.

---

## Règles sur une page web
Quand les règles ne sont publiées que sur une page web (Google Sites, Notion, site de l'éditeur), le PDF fait
par la skill [recherche-regles-jeu](https://github.com/vinchy1/recherche-regles-jeu) n'est pas recouvert : la
page elle-même est traduite (`page_web.py` liste ses morceaux de texte dans l'ordre d'affichage, puis remet le
français à leur place), et remise en PDF avec le même script (`page_pdf.py --garder` garde la page et ses
images). Le livret français a exactement la présentation du site, sans police réduite ni texte condensé.
Le texte dessiné dans les images reste dans la langue d'origine. Détails : `references/page-web.md`.

---

## Contrôles automatiques et relecture
**Pendant la construction** (`build.py`, `scan.py`, `controle.py`) :
- **TEXTE NON TRADUIT** : toute ligne d'origine ni remplacée ni volontairement ignorée (`IGNORE`) est listée.
- **À VÉRIFIER** : français qui chevauche une image, un autre texte ou un reste de texte d'origine, ou qui sort
  de la page ; sur un scan, ligne d'origine qui dépasse d'un masque.
- **DÉBORDE / police réduite** : le français ne tient pas à la taille d'origine.
- **flux protégé / réécriture du flux annulée** : avant d'effacer le texte d'origine, `build.py` réécrit
  certaines pages (livrets InDesign japonais) pour éviter que des lignes voisines se décalent ou disparaissent,
  ou que des images cachées réapparaissent ; si cela fait bouger le moindre mot, la réécriture est annulée et
  la page est à regarder de près.

**Relecture du texte** (`relecture.py`) : écrit `texte_fr.txt` (original et français, paragraphe par paragraphe)
et signale les mots inconnus du dictionnaire, les fautes de typographie, l'anglais resté, les termes du glossaire
non respectés (« trump » sans « atout ») et les paragraphes anormalement courts ou longs (signe d'un oubli).
Avec `LANGUE=en`, elle relit la version anglaise (`texte_en.txt`, orthographe anglaise).

**Deux agents relecteurs** (`agents/`), lancés en parallèle :
- **relecteur-texte** lit tout le texte et rend les contresens, oublis, incohérences de vocabulaire et fautes,
  et dit pour chaque alerte automatique si elle est vraie ou fausse ;
- **relecteur-visuel** compare chaque page à l'original, agrandit les endroits signalés et rend les défauts
  localisés (page et coordonnées).

Les agents ne modifient rien : leurs rapports sont des propositions, que le modèle principal vérifie avant de
corriger.

---

## Coût en jetons et modèles
Une traduction consomme beaucoup de jetons, surtout à cause des images : compter environ 150 000 à 200 000
jetons pour un livret de quelques pages, davantage pour un scan ou un long livret. La skill limite ce coût de plusieurs
façons :
- **les scripts font le travail mécanique** : préparation, inventaire, construction, contrôles, rangement ;
- **les images ne servent qu'à bon escient** : planches à faible résolution, agrandissements seulement sur ce qui
  est signalé, texte lu dans des fichiers plutôt que sur des images ;
- **la relecture est confiée à un modèle moins coûteux.** Le modèle que vous avez choisi dans Claude Code dirige
  le travail : il traduit, écrit le spec et prend les décisions, car c'est ce qui fait la qualité. Les deux
  relectures, longues et gourmandes en images, tournent sur **Sonnet** dans des agents séparés : leurs images et
  leur lecture du texte complet n'alourdissent pas la conversation principale, qui ne reçoit qu'un rapport de
  quelques lignes. À titre d'exemple, sur un livret de 2 pages en 4 colonnes, chaque relecteur a pris environ
  25 secondes.
- Les tâches courtes ne sont **pas** déléguées : démarrer un agent a un coût fixe qui dépasserait le gain.

Pour changer le modèle des relecteurs, modifier la ligne `model:` en tête des fichiers de `agents/` (par
exemple `haiku`, moins cher mais moins sûr pour juger une traduction).

---

## Corriger une traduction déjà faite
Demander la correction à Claude (« dans <Jeu>, page 2, remplace … par … »). Il lit `_outils/<Jeu>/NOTES.md`,
modifie `spec.py`, reconstruit, regarde la page, puis remplace `<nom>_FR.pdf` **après votre accord**
(`livrer.py --remplacer`). Le dossier d'outils de chaque jeu est conservé pour cela.

---

## Contenu du dépôt
- `SKILL.md` — la méthode suivie par Claude et les préférences de traduction
- `agents/` — `relecteur-texte.md` et `relecteur-visuel.md` (Sonnet), à copier dans `~/.claude/agents/`
- `references/`
  - `dossiers.md` : organisation des dossiers, noms de jeux, index et notes ;
  - `langues.md` : langue de départ, livrets multilingues, longueur du français selon la langue, texte vertical ;
  - `scan.md` : livrets scannés (lecture automatique, zones, rendus) ;
  - `recomposer.md` : livret recomposé à neuf, sans l'image du scan ;
  - `page-web.md` : règles publiées sur une page web (page traduite puis remise en PDF ; page bilingue,
    version anglaise) ;
  - `en-ligne.md` : livret seulement consultable en ligne (Google Drive) : où chercher le PDF, captures de
    l'utilisateur ;
  - `source.md` : vérifier une règle traduite contre l'original (Tokyo Game Market pour les jeux japonais) ;
  - `glossaire.md` : vocabulaire de jeu commun à tous les jeux, avec équivalences par langue ;
  - `polices.md` : polices de remplacement déjà trouvées pour des polices de livrets ;
  - `pieges.md` : cas difficiles rencontrés et leurs solutions (texte en double, calques cachés, icônes,
    mini-cartes, soulignements, marges de paragraphe, lignes décalées ou images cachées qui ressortent à
    l'effacement…).
- `scripts/`
  - `preparer.py` : démarre un livret (voir plus haut) ;
  - `build.py` : moteur des livrets à texte ; `scan.py` : moteur des livrets scannés ; `commun.py` : fonctions
    communes ; `controle.py` : contrôles automatiques de mise en page ;
  - `spec_modele.py` / `spec_scan_modele.py` : modèles de `spec.py`, avec la description de toutes les options ;
  - `dump.py` : inventaire du texte (`blocks.txt`) ; `polices.py` : extraction, contrôle des accents,
    comparaison et largeur des polices ; `ocr.py` : lecture du texte des scans ;
  - `grille.py` / `bandes.py` : mesures de position sur les scans ;
  - `icone.py` : découpe une icône d'un scan pour la remettre dans le texte français ;
  - `sheet.py` / `side.py` : planche d'aperçu et images côte à côte original / français ;
  - `relecture.py` : relecture du texte seul ;
  - `page_web.py` : texte d'une page web à traduire, et page française reconstruite ;
  - `livrer.py` : livraison (version anglaise comprise ; `--pdf` pour une traduction faite sans spec) ;
    `ranger.py` : allègement du dossier d'outils.
- `exemples/recomposer/` : `noir.py` et `textes.py`, moteur et texte d'un livret recomposé à neuf (CMYK), à copier
  et adapter.

---

## Limites connues
- **Texte vertical** (japonais) : le français est posé à l'horizontale dans la même zone si la place le permet,
  sinon c'est signalé.
- **Japonais, chinois, coréen** : le français prend beaucoup plus de place (souvent 1,5 à 2 fois en largeur) ; la
  traduction est plus concise et la police plus réduite, ce qui est signalé.
- **Texte dessiné dans les illustrations** (logos, onomatopées) : laissé tel quel, et signalé.
- **Lecture automatique des scans** : Windows uniquement ; ailleurs, Claude lit les pages en images (plus cher).
- **Livrets très longs** : la traduction se fait page par page dans une même session, ce qui reste coûteux ;
  mieux vaut le savoir avant de commencer un livret de 40 pages.

---

## Remarques
- **Usage personnel** : ne diffusez les livrets traduits qu'avec l'accord de l'éditeur du jeu.
- Skill initiée par [notoriousFabb](https://github.com/notoriousFabb/traduction-regles-jeu), optimisée et complétée par
  [vinchy1](https://github.com/vinchy1/traduction-regles-jeu) ; cette version par vinchy1 y ajoute la traduction depuis
  toutes les langues, les livrets scannés, l'organisation en dossiers, les contrôles automatiques, la relecture du texte et les agents relecteurs.
- Ce dépôt évolue désormais de façon indépendante. Le dépôt d'origine reste noté comme source facultative
  (`git fetch notoriousfabb`) : ses nouveautés ne sont reprises qu'à la demande.
