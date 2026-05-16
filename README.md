# REvSoc — Retrospective Evaluation of Soccer Players

A multidimensional player rating system for outfield players and goalkeepers across the **Big 5 European leagues**, built entirely in R using publicly available FBRef data. Ratings are produced for four consecutive seasons: **2021–22, 2022–23, 2023–24, and 2024–25**.

**This project is currently halted due to Opta stopping their supply of data to Football Reference and the archiving of worldfootballR**

---

## Overview

REvSoc constructs statistically grounded, position-aware performance ratings for every player in the Premier League, La Liga, Serie A, Bundesliga, and Ligue 1 who meets a minimum minutes threshold. The system separates goalkeepers from outfield players and, within outfield players, decomposes performance into three distinct dimensions before combining them into an overall rating validated against real-world outcomes.

---

## Data Source

All player and team statistics are pulled from **FBRef** via the [`worldfootballR`](https://jaseziv.github.io/worldfootballR/) R package. Stat types collected include:

- `standard`, `shooting`, `passing`, `passing_types`
- `gca`, `defense`, `possession`, `playing_time`, `misc`
- `keepers`, `keepers_adv`
- League table standings (for validation)

A minimum playing time filter of **900 minutes** is applied throughout to exclude small-sample players.

---

## Goalkeeper Rating

### Components

The goalkeeper rating is split into two sub-ratings:

| Sub-rating | Key metrics |
|---|---|
| **Conventional Keeper Rating (CKR)** | Saves, Goals Against (GA), Save %, PSxG overperformance |
| **Sweeper Keeper Rating (SKR)** | Distribution accuracy, launch rates, defensive actions outside the box, average distance of defensive actions, cross-stopping % |

Both components are normalised relative to their league averages and combined into a **Total Goalkeeper Rating (GKR)**.

### Weighting via PCA

Principal Component Analysis (PCA) is run on all scaled goalkeeper metrics. The variance explained by **Dim 1** (36.1%) and **Dim 2** (16.7%) is used to derive relative weights:

- Dim 1 weight: **~0.7**
- Dim 2 weight: **~0.3**

Variable contributions to each principal component determine which metrics carry the most influence in the final rating.

### Validation

The GKR is validated against three independent outcome metrics using Spearman correlations:

1. Goals Against (GA)
2. PSxG–GA per 90
3. Save Percentage

Strong alignment with GA confirms outcome relevance; moderate correlations with the shot-stopping metrics confirm validity across both conventional and sweeper dimensions.

---

## Outfield Player Rating

Outfield players are rated across three independent dimensions, each possession-adjusted, before being combined into an overall score.

### 1. Defensive Player Rating (DPR)

Built from defensive and miscellaneous FBRef stat types. Adjusted for team possession — defenders playing for lower-possession teams face more defensive work, so DPR is scaled by `(100 - avg_team_possession)` to produce a **Possession-Adjusted DPR**.

Feature selection uses the **Boruta** algorithm to identify which defensive metrics genuinely contribute to the rating, rather than relying on arbitrary metric choices.

### 2. Offensive Creation Rating (OCR)

Captures chance creation and progressive ball-carrying, including: assists, xA, SCA, GCA, key passes, progressive passes into the final third, penalty area passes, and crosses. Adjusted by **opponent possession** (to reflect the difficulty of creating against high-possession sides) and normalised to the league average.

### 3. Goalscorer Rating (GSR)

Focuses on finishing quality and goal threat:

- Goals per xG (over/underperformance vs. expected)
- Shot-on-target %
- Goals per shot
- xG per shot
- Progressive carries received

Validated against goals per 90 (Spearman ρ ≈ 0.65) and against xG per 90 (ρ ≈ 0.41), confirming the rating captures both volume and efficiency.

---

## Overall Player Rating & Clustering

After computing DPR, OCR, and GSR, all three are min-max normalised (`_std` suffix) and joined into a single player master table. Players are then:

1. **Clustered** using k-means (k determined by the elbow method and silhouette width via PAM) — typically two clusters emerging from the DPR/OCR/GSR space.
2. **PCA run within each cluster** to derive cluster-specific metric weights.
3. **Overall Rating** computed as a weighted sum of the three standardised sub-ratings, with weights derived from the within-cluster PCA.

This approach means the relative importance of defending, creating, and scoring is allowed to vary naturally by player profile, rather than being fixed a priori.

---

## Team Ratings

Individual player ratings are aggregated to a **Weighted Team Rating** by computing a minutes-weighted average of Overall Ratings for each squad. This is validated against final league table position using Spearman correlation across all five leagues.

---

## Tech Stack

| Tool | Role |
|---|---|
| R | All analysis |
| `worldfootballR` | FBRef data ingestion |
| `tidyverse` | Data wrangling |
| `FactoMineR` / `factoextra` | PCA |
| `Boruta` | Feature selection |
| `cluster` | k-means / PAM clustering |
| `ggplot2` / `ggrepel` | Visualisation |

---

## Seasons Covered

| Season | Data year |
|---|---|
| 2021–22 | `season_end_year = 2022` |
| 2022–23 | `season_end_year = 2023` |
| 2023–24 | `season_end_year = 2024` |
| 2024–25 | `season_end_year = 2025` |

The same methodology is applied consistently across all four seasons, enabling retrospective comparison.

---

## Author

**Seshadhri S** — *2025*
