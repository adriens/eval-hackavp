# eval-hackavp

Évaluation des 11 soumissions du **HackAVP 2026**, le hackathon de l'OPT-NC, de la Station N et de la commission Data & IA d'OPEN NC, dédié à l'emploi dans la fonction publique en Nouvelle-Calédonie.

Le dépôt contient une grille de notation commune et une copie de cette grille par évaluateur (humain ou modèle). Toutes les copies ont le même format, ce qui permet de comparer les évaluations avec DuckDB.

## Sources

- **Guide du jury** (source de vérité pour la liste des soumissions et les pistes) : <https://dev.to/adriens/hackavp-guide-devaluation-pour-les-membres-du-jury-12i0>
- **Lancement du hackathon** (défi, fonctions attendues, grille sur 100) : <https://dev.to/adriens/hackavp-premier-hackathon-dedie-a-lemploi-dans-la-fonction-publique-en-ncl-3oj0>

## Fichiers

| Fichier | Rôle |
|---|---|
| `grille_hackavp_notation.csv` | Modèle : les 11 candidats avec les notes vides. **Ne pas noter dedans.** |
| `adriens_grille_hackavp_notation.csv` | Grille d'Adrien SALES (jury), sur 100 ; n° 9 et 10 non notés (conflit d'intérêt) |
| `thomas-avron_grille_hackavp_notation.csv` | Grille de Thomas AVRON (jury), sur 100 ; n° 3 incomplète (13 critères sur 22) |
| `christophe-roger_grille_hackavp_notation.csv` | Grille de Christophe Roger (jury), sur 100 |
| `christophe-roger_justifications.md` | Justification de chacune des 242 notes de Christophe Roger, avec références |
| `michele-barre_grille_hackavp_notation.csv` | Grille de Michèle Barre (jury), sur 100 |
| `marc-rodrigues_grille_hackavp_notation.csv` | Grille de Marc Rodrigues (jury), sur 100 ; sans commentaires |
| `andy-malo_grille_hackavp_notation.csv` | Grille d'Andy Malo (jury), sur 100 |
| `opus-5_5_grille_hackavp_notation.csv` | Évaluation par Claude Opus 5.5 |
| `sonnet-5_5_grille_hackavp_notation.csv` | Évaluation par Claude Sonnet 5.5 |
| `haiku-5_5_grille_hackavp_notation.csv` | Évaluation par Claude Haiku 5.5 |
| `fable-5_grille_hackavp_notation.csv` | Évaluation par Claude Fable 5 |
| `detection_ia_articles.csv` | Part de texte généré par IA dans chaque article, selon ZeroGPT (article entier) et Pangram (extrait), mesurée le 23/10/2026 |
| `methode_detection_ia.md` | Méthode, contrôles et limites de cette mesure |
| `reactions_articles.csv` | Réactions et commentaires de chaque article sur dev.to (API publique, `public_reactions_count` et `comments_count`), avec la date du relevé. Indicateur à part : il ne s'ajoute pas à la note /100 |
| `jury.csv` | Composition du jury : identifiant (préfixe du fichier de grille), nom, type, profil LinkedIn |
| `prix_du_jury.csv` | Projet proposé pour le prix du jury (coup de cœur), qui l'a proposé et pourquoi |
| `vote_prix_du_jury.csv` | Vote de chaque juré humain sur le prix du jury (`Pour`, `Contre`, `Abstention`) |
| `transcripts/` | Sous-titres automatiques français des 11 vidéos (`.vtt`), utilisés par les modèles |
| `rapport_hackavp.qmd` | Rapport Quarto (R, DuckDB, ggplot2) : classements, accord entre évaluateurs, détection IA |
| `Taskfile.yml` | Build du rapport PDF avec Task et xelatex |

Les fichiers s'appellent `<evaluateur>_grille_hackavp_notation.csv`. Dans `<evaluateur>`, les points de version sont remplacés par des `_` : `opus-5_5`, `sonnet-5_5`, `adriens`. Cette convention permet aux requêtes ci-dessous de lire toutes les évaluations d'un coup.

## Le modèle de grille et comment noter

`grille_hackavp_notation.csv` est le fichier d'exemple : la grille sur 100, **vierge**, que chaque évaluateur copie pour noter. Il contient :

- les **11 soumissions**, une par ligne, avec leurs colonnes d'identification déjà remplies : `N°`, `Projet`, `Auteur`, `Profil auteur`, `Piste`, `Article dev.to`, `Vidéo YouTube` ;
- les **22 colonnes de critères**, vides, dont l'en-tête indique le maximum (`(/4)`, `(/8)`…) ;
- `TOTAL /100` et `Commentaires`, vides ;
- `Conflit d'intérêt`, rempli pour les n° 9 et 10 (soumissions de l'organisateur).

On ne note **jamais** dans le modèle lui-même : il sert de point de départ commun, et le rapport l'utilise comme référence pour la liste des projets et l'ordre des critères.

### Ajouter un membre du jury

1. **Copier le modèle** sans écraser une grille existante, en préfixant le fichier par l'identifiant du juré :

   ```sh
   cp --update=none grille_hackavp_notation.csv marc-rodrigues_grille_hackavp_notation.csv
   ```

2. **Déclarer le juré** dans `jury.csv` s'il n'y est pas : la colonne `identifiant` doit être exactement le préfixe du fichier (`marc-rodrigues`).
3. **Noter** dans un tableur (LibreOffice, Excel) ou un éditeur de texte :
   - remplir les 22 critères sans dépasser le maximum de l'en-tête ; les demi-points sont permis (`2.5`, avec un point) ;
   - laisser **vide** un critère non évalué (vide n'est pas zéro) et le signaler dans le commentaire ;
   - `TOTAL /100` = somme des notes renseignées ;
   - justifier les notes dans `Commentaires` en quelques phrases ;
   - ne pas toucher aux colonnes d'identification ni à `Conflit d'intérêt`. L'organisateur ne note pas les n° 9 et 10 ; tous les autres jurés les notent.
4. **Enregistrer en CSV** en gardant le format (UTF-8, séparateur virgule, une ligne d'en-tête et 11 lignes).
5. **Relancer `task`** : le rapport intègre la grille automatiquement et passe le juré de « En attente » à « Reçue ».

Une ligne dont `TOTAL /100` est vide est considérée comme non notée : un juré peut donc rendre une grille partielle.

## Format du CSV

- Encodage UTF-8 **avec BOM**, fins de ligne CRLF, séparateur virgule, décimales avec un point (`0.5`).
- Une ligne d'en-tête, puis une ligne par soumission (11 lignes), 32 colonnes.
- `N°` : numéro de la soumission dans le guide du jury (1 à 11).

| Colonne | Contenu |
|---|---|
| `N°`, `Projet`, `Auteur`, `Profil auteur`, `Piste`, `Article dev.to`, `Vidéo YouTube` | Identification de la soumission |
| `Socle : …` (9 colonnes, /40) | Créativité et esthétique (/4), UX et utilisabilité (/5), accessibilité (/4), adéquation moyens/résultats (/5), standards et Open Data (/6), storytelling (/4), réutilisabilité et essaimage (/4), plaisir de lecture (/4), plaisir de visionnage (/4) |
| `Technicité : …` (4 colonnes, /25) | Pertinence des ressources (/8), intégrabilité (/8), maintenabilité (/5), interaction inédite (/4, bonus) |
| `Valeur RH : …` (5 colonnes, /25) | Qualité du matching, documents générés, restitution, facilité de création, effet sur la décision (/5 chacun) |
| `Track : …` (4 colonnes, /10) | Chaîne de matching (/4), architecture ou frameworks Open Source (/2), déployabilité ou licences (/2), intégrations ou accessibilité hardware (/2), lus selon la piste |
| `TOTAL /100` | Somme des notes renseignées |
| `Commentaires` | Justification de l'évaluateur |
| `Conflit d'intérêt` | Rempli pour les n° 9 et 10 (soumissions de l'organisateur) |

Le maximum de chaque critère figure entre parenthèses dans l'en-tête. Une cellule de note **vide** veut dire « non évalué ». Ce n'est pas un zéro. Le `TOTAL /100` ne fait alors la somme que des critères notés, et le commentaire doit le dire.

**Grille sur 100.** Elle remplace la grille MIAGE sur 20 (19 colonnes, `TOTAL /20`), qui reste dans l'historique git. Les critères *plaisir de lecture* et *plaisir de visionnage* sont nouveaux ; les quatre colonnes Track sont communes aux deux pistes et recouvrent les critères propres à chacune. Toutes les grilles, jury comme modèles, se remplissent sur 100 ; une grille restée sur 20 est ignorée par le rapport et par les requêtes ci-dessous.

Les nouvelles évaluations Sonnet, Fable et Opus reposent sur les 11 articles et leurs transcriptions (archivées dans `transcripts/`, sous-titres automatiques YouTube en français). Sonnet n'a pas évalué la créativité/esthétique (totaux sur 96) ; Fable l'a notée d'après les captures publiques des articles, sauf pour les n° 9 et 11 qui n'en contiennent pas (totaux sur 96 pour ces deux-là) ; Opus a noté les 22 critères partout, l'esthétique d'après les images publiques des articles. Le plaisir de visionnage porte uniquement sur le propos transcrit, pas sur les images, le son ou le montage ; chaque commentaire précise ces limites.

## Pistes

| Piste | Soumissions |
|---|---|
| onPrem souverain | 1, 3, 4, 6, 7, 11 |
| SaaS no limit | 2, 5, 8, 9, 10 |

La n° 5 (*Adopte un Job*) est en **SaaS no limit**, comme le déclare son article. Le guide du jury avait été corrigé en « onPrem » le 22/10/2026, puis la piste SaaS a été confirmée : le guide doit être aligné.

## Points connus

- **Conflit d'intérêt** : les n° 9 et 10 sont des soumissions d'Adrien SALES, l'organisateur. Ils restent en compétition : seul l'organisateur ne les note pas (et ne vote pas sur ces deux projets). Tous les autres évaluateurs, modèles compris, les notent normalement.
- **Transcriptions** : les sous-titres automatiques français des 11 vidéos sont archivés dans `transcripts/`. Ceux des vidéos 1 et 6 (*CV Generator*, *CV FORGE*), d'abord introuvables, ont été récupérés après correction de l'identifiant de la vidéo.
- **Ce que les modèles n'ont pas vu** : ils ont lu les articles (API dev.to) et les transcriptions automatiques des vidéos. Ils n'ont vu ni les images, ni les vidéos elles-mêmes.

## Requêtes DuckDB

### Classements d'un évaluateur

```sql
CREATE VIEW n AS
SELECT "N°" AS num, Projet, Piste, "TOTAL /100" AS total
FROM read_csv('opus-5_5_grille_hackavp_notation.csv')
WHERE "TOTAL /100" IS NOT NULL;

-- Gagnant par piste
SELECT Piste, num, Projet, total FROM n
QUALIFY rank() OVER (PARTITION BY Piste ORDER BY total DESC) = 1;

-- Classement par piste
SELECT Piste, rank() OVER (PARTITION BY Piste ORDER BY total DESC) AS rang, num, Projet, total
FROM n ORDER BY Piste, rang;

-- Vue d'ensemble indicative (pas de classement global officiel)
SELECT rank() OVER (ORDER BY total DESC) AS rang, num, Projet, Piste, total
FROM n ORDER BY rang;
```

### Bench de toutes les évaluations

```sql
CREATE VIEW evals AS
SELECT regexp_extract(filename, '([^/]+)_grille_hackavp_notation\.csv$', 1) AS evaluateur,
       "N°" AS num, Projet, Piste, "TOTAL /100" AS total
FROM read_csv('*_grille_hackavp_notation.csv', filename = true, union_by_name = true)
WHERE "TOTAL /100" IS NOT NULL;   -- écarte les lignes non notées et les grilles restées sur 20

-- Notes côte à côte, une colonne par évaluateur
PIVOT (SELECT evaluateur, num, Projet, Piste, total FROM evals)
ON evaluateur USING first(total) ORDER BY num;

-- Moyenne et dispersion par soumission
SELECT num, any_value(Projet) AS Projet, round(avg(total), 2) AS moyenne,
       round(stddev_samp(total), 2) AS ecart_type, count(*) AS n
FROM evals GROUP BY num ORDER BY moyenne DESC;

-- Classement par rangs (Borda) : insensible à l'échelle de chaque évaluateur
WITH r AS (
  SELECT evaluateur, num, Projet, Piste,
         (count(*) OVER w - rank() OVER (w ORDER BY total DESC)
            - (count(*) OVER (PARTITION BY evaluateur, total) - 1) / 2.0)
           / (count(*) OVER w - 1) AS score   -- 1 = premier, 0 = dernier, rang moyen pour les ex æquo
  FROM evals WINDOW w AS (PARTITION BY evaluateur))
SELECT rank() OVER (ORDER BY avg(score) DESC) AS rang, num, any_value(Projet) AS Projet,
       any_value(Piste) AS Piste, round(100 * avg(score), 1) AS score_borda, count(*) AS n
FROM r GROUP BY num ORDER BY rang;
```

Le motif `*_grille_hackavp_notation.csv` ne prend pas le modèle `grille_hackavp_notation.csv`, qui n'a pas de préfixe. Les lignes non notées (copies vierges, n° 9 et 10 dans la grille de l'organisateur) et les grilles restées sur 20 sont écartées par le filtre sur `TOTAL /100`.

Avec le CLI, lancer `duckdb -no-agent -box < requete.sql`. Si la variable `AI_AGENT` est définie, `-no-agent` est nécessaire pour que les commandes `.print` fonctionnent.

## Détection de contenu généré par IA

La part de texte généré par IA dans chaque article a été mesurée avec deux détecteurs. Le détail est dans `methode_detection_ia.md`, les chiffres dans `detection_ia_articles.csv`.

1. **Préparation** : texte de l'article récupéré par l'API dev.to, puis nettoyé (sans code, intégrations, images, URL ni balises).
2. **ZeroGPT** (gratuit, sans compte) : articles entiers, envoyés en fichier `.txt`. Presque tout sort à 0 % : résultat jugé peu fiable.
3. **Pangram** (compte gratuit, 20 crédits par jour, 1 crédit pour 100 mots) : un extrait de 50 à 100 mots par article, choisi avec une règle fixe (premiers paragraphes de prose, sans titres, listes, emojis ni citations). Quatre articles ressortent entre 46 et 73 % d'IA.
4. **Contrôles** : deux textes témoins écrits par IA (brut et retravaillé) sont reconnus à 100 % par Pangram. Le contrôle sur un texte humain reste à faire.

Ces scores donnent une indication, pas une preuve. Ils ne doivent pas servir, seuls, à pénaliser une équipe.

## Rapport PDF

```sh
task          # vérifie les outils, puis rend rapport_hackavp.pdf
task open     # ouvre le PDF
task clean    # supprime les fichiers générés
```

Il faut Quarto, R (paquets DBI, duckdb, dplyr, tidyr, ggplot2, forcats, stringr, knitr, rmarkdown, scales), xelatex et la police Inter. Le rapport relit à chaque rendu toutes les grilles `*_grille_hackavp_notation.csv` présentes : pour ajouter un membre du jury, il suffit de déposer sa grille et de relancer `task`. Le prix du jury (coup de cœur) se décide par les seuls membres du jury humains : le projet proposé est dans `prix_du_jury.csv` (colonnes `N°`, `Proposé par`, `Motivation`), chaque juré inscrit son vote (`Pour`, `Contre`, `Abstention`) dans `vote_prix_du_jury.csv`, et le prix est attribué à la majorité absolue du jury (4 voix sur 6). Le rapport prend le nom et le type de chaque évaluateur dans `jury.csv`, et y affiche la grille comme « Reçue » ou « En attente ». Pour une grille absente de `jury.csv`, les noms de fichiers commençant par un nom de modèle (`opus`, `sonnet`, `haiku`, `gpt`, etc.) sont classés « Modèle IA », les autres « Membre du jury ».
