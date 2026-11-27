# London Breakout NQ · range 4h-9h ET, entrée 9h30-11h

🇬🇧 [English version](./README.en.md)

Indicateur TradingView (Pine Script v6) pour le **NASDAQ (NQ / NAS100) en M1**. Il trace le range de la session de Londres (04:00-09:00 heure de New York), attend la **première cassure** pendant l'ouverture US (09:30-11:00), affiche l'entrée, le SL et le TP, puis **backteste automatiquement** la stratégie sur l'historique visible.

**Un trade par jour maximum.**

---

## Sommaire

1. [Installation](#1-installation)
2. [La stratégie en 4 étapes](#2-la-stratégie-en-4-étapes)
3. [Lire le graphique](#3-lire-le-graphique)
4. [Paramètres](#4-paramètres)
5. [Panneau « Aujourd'hui »](#5-panneau--aujourdhui-)
6. [Tableau de statistiques](#6-tableau-de-statistiques)
7. [Grille « SL fixe »](#7-grille--sl-fixe-)
8. [Alertes](#8-alertes)
9. [Limites connues](#9-limites-connues)

---

## 1. Installation

1. Ouvre [`London_Breakout_NQ.pine`](./London_Breakout_NQ.pine), clique sur **Copy raw file** (icône copier en haut à droite du code).
2. Sur TradingView : **Pine Editor** (bas de l'écran) → **Ouvrir** → **Nouvel indicateur**.
3. Supprime le code par défaut, colle le script, **Enregistrer**, puis **Ajouter au graphique**.
4. Mets le graphique sur **NAS100 / NQ en 1 minute**.

Les heures sont toujours en **heure de New York (ET)**, quel que soit le fuseau de ton graphique :

| Session | Heure ET | Heure de Londres | Heure de Paris |
|---|---|---|---|
| Range London | 04:00-09:00 | 09:00-14:00 | 10:00-15:00 |
| Fenêtre d'entrée | 09:30-11:00 | 14:30-16:00 | 15:30-17:00 |
| Sortie forcée | 16:00 | 21:00 | 22:00 |

*(Hors des semaines de décalage d'heure d'été entre l'Europe et les États-Unis.)*

---

## 2. La stratégie en 4 étapes

**① Le range (04:00-09:00 ET)**
La boîte bleue suit le plus haut et le plus bas de la session. À 09:00, le range est figé : deux traits noirs prolongent le **High** et le **Low**.

**② L'ouverture (09:30 ET)**
- Si le prix ouvre **déjà hors du range** → le jour est sauté (réglable : entrer quand même à l'ouverture).
- Sinon → la fenêtre d'entrée s'ouvre (fond orange) jusqu'à 11:00.

**③ La cassure**
- Le prix touche le **High** → **ACHAT** au niveau du High.
- Le prix touche le **Low** → **VENTE** au niveau du Low.
- **SL** : sous le plus bas (achat) ou au-dessus du plus haut (vente) de la bougie qui casse, + buffer optionnel.
- **TP** : 2R par défaut.
- Si les deux côtés cassent dans la même bougie → jour sauté (ambigu).
- Pas de cassure avant 11:00 → pas de trade.

**④ La sortie**
- SL, TP, ou **sortie forcée à 16:00 ET** si rien n'est touché.
- Option : TP1 partiel, puis SL à breakeven et reste au TP2.

**Exemple réel (NAS100 M1, 27 nov. 2026)**
- Range : High **26 829.47** · Low **26 550.67** (279 pts, 0.29 ATR jour).
- 09:30 ET : le prix ouvre dans le range → fenêtre ouverte.
- La première bougie M1 casse le Low → **VENTE à 26 550.67**.
- SL au-dessus de la bougie de cassure : **26 561.30** (≈ 10.6 pts de risque).
- TP 2R : **26 529.42** → touché quelques minutes plus tard : **+2R**.

---

## 3. Lire le graphique

| Élément | Signification |
|---|---|
| Boîte bleue « SESSION » | Range London 04:00-09:00 ET |
| Pointillés au milieu de la boîte | Milieu du range |
| Traits noirs | High et Low du range, prolongés jusqu'à 16:00 (ou 11:00) |
| Boîte grise en tirets « SAUTÉ » | Range rejeté par le filtre de taille |
| Fond orange | Fenêtre d'entrée 09:30-11:00 |
| Zone verte / rouge | TP / SL du trade en cours |
| « SL → BE » | TP1 touché, stop remonté à l'entrée |
| Étiquette verte / rouge | Résultat du jour en R : `+2R`, `-1R`, `(temps)` = sortie à 16:00, `(TP1+BE)` |

---

## 4. Paramètres

### Sessions
| Paramètre | Défaut | Rôle |
|---|---|---|
| Fuseau des sessions | America/New_York | Ne pas changer pour le NQ |
| Range London (ET) | 0400-0900 | |
| Fenêtre d'entrée (ET) | 0930-1100 | Aucune nouvelle entrée en dehors |
| Gestion (sortie forcée à la fin) | 0930-1600 | Le trade est clôturé au prix du marché à la fin |

### Trade
| Paramètre | Défaut | Rôle |
|---|---|---|
| R:R du TP | 2.0 | Distance du TP en multiple du risque (TP2 si TP1 actif) |
| Buffer SL (points) | 0 | Points ajoutés au-delà de la bougie de cassure. NAS100 : 1 à 3 pts pour absorber le spread |
| Prix déjà hors range à 9h30 | Sauter le jour | Ou : entrer au prix d'ouverture de 09:30 |
| Bougie d'entrée refermée dans le range = perte | oui | **S'applique en M1** (et dès que l'intrabar est inactif) : si la bougie de cassure clôture de nouveau dans le range, le trade compte comme perte (hypothèse prudente, l'ordre des prix dans la bougie étant inconnu) |
| Sens | Les deux | Achats seulement / ventes seulement. Si le premier côté cassé est exclu, pas de trade ce jour-là |

### TP1 / BE
| Paramètre | Défaut | Rôle |
|---|---|---|
| TP1 partiel + SL à BE | non | Non = TP unique. Oui = TP1, stop à l'entrée, reste au TP2 |
| TP1 (R) | 1.0 | |
| Part fermée au TP1 (%) | 50 | |

### Filtre range
| Paramètre | Défaut | Rôle |
|---|---|---|
| Filtre taille du range | Aucun | **Points** ou **× ATR jour** (ATR 14 de la veille) : saute les ranges trop petits ou trop grands (jours de news) |
| Min / Max | 0 / 1.0 | Exemple en × ATR : min 0.15, max 0.60 |

### Résolution
| Paramètre | Défaut | Rôle |
|---|---|---|
| Simuler en intrabar | oui | Sur un graphique **supérieur à M1** (M5, M15…), rejoue chaque bougie minute par minute pour l'ordre réel cassure → SL → TP. **Sans effet en M1** (le tableau affiche alors « Bougie 1 ») |
| TF intrabar | 1 | |
| SL sous la bougie | Bougie du graphique | Avec intrabar : SL sous la bougie du graphique (ex. M5) ou sous la bougie 1 minute qui casse |

### Affichage, Box session, Période
- Panneau « Aujourd'hui », tableau de stats, grille SL fixe, labels de résultat, fond de la fenêtre : affichables ou non, positions réglables.
- Couleurs de la boîte, milieu en pointillés, étiquette (texte « SESSION » modifiable), épaisseur des traits, prolongation jusqu'à 11:00 ou 16:00.
- **Limiter les stats à une période** : compare deux timeframes ou deux réglages sur exactement les mêmes jours.

---

## 5. Panneau « Aujourd'hui »

| Ligne | Contenu |
|---|---|
| High / Low | Niveaux du range du jour |
| Taille | Taille en points et en ATR jour |
| Prix vs High / Low | Distance du prix actuel aux deux niveaux |
| État | `Range en cours` → `Range prêt · attente 9h30` → `Fenêtre ouverte · attente cassure` → `EN TRADE ACHAT/VENTE` (+ R en direct) → `Terminé : +2R` ; ou le motif du saut (`Sauté : déjà hors range à 9h30`, `Pas de cassure aujourd'hui`…) |

---

## 6. Tableau de statistiques

| Ligne | Signification |
|---|---|
| Mode | Résolution utilisée (`Bougie 1` en M1, `Intrabar 1 / SL…` au-dessus) |
| Jours avec range / Trades | Jours analysés / trades pris |
| Sautés (hors range 9h30) / (filtre range) | Jours écartés et pourquoi |
| Pas de cassure | Jours sans cassure avant 11:00 |
| Ambigus / sens exclu | Deux côtés cassés dans la même bougie / côté filtré |
| Win rate (R > 0) | En vert s'il dépasse le **WR mini rentable** (100 / (1 + R:R), soit 33.3 % à 2R) |
| R moyen / trade, Total R | Espérance et résultat cumulé |
| Achats / Ventes : n / R moy | Détail par sens |
| Sorties temps / TP1+BE | Trades sortis à 16:00 / au breakeven |
| Pire série de pertes | Plus longue série de trades perdants |
| SL moyen (pts) / SL moyen / range | Taille moyenne du stop, en points et en % du range |
| Jours sans intrabar | Jours où les données 1 minute manquaient (simulation sur la bougie du graphique) |

---

## 7. Grille « SL fixe »

Deuxième tableau : la **même entrée**, mais avec un **SL fixe = k × taille du range** au lieu du SL sous la bougie, et un TP à 2R ou 3R.

| k | TP 2R | TP 3R |
|---|---|---|
| 0.10 · 0.20 · 0.35 · 0.50 | R moyen · win rate · nombre de trades | idem |

Sert à vérifier si un stop plus large (ou plus serré) que la bougie de cassure aurait mieux marché sur ton historique.

---

## 8. Alertes

Crée **une seule alerte** sur l'indicateur avec la condition **« N'importe quel appel de la fonction alert() »**.

| Moment | Message |
|---|---|
| 09:00 ET, range validé | `LDN BO NAS100 \| range prêt  H 26829.47  L 26550.67` |
| Pendant la fenêtre, en temps réel | `LDN BO NAS100 : cassure du LOW London 26550.67` |
| Entrée (clôture de bougie) | `LDN BO NAS100 VENTE  entrée 26550.67  SL 26561.30  TP 26529.42` |

---

## 9. Limites connues

- Stats **hors spread et slippage**. L'entrée est supposée remplie **exactement** au High/Low : en réel, un ordre stop sur une cassure rapide peut glisser de quelques points.
- En M1, le SL sous la bougie de cassure est souvent **serré** (≈ 10 à 60 pts sur NAS100) : vérifie que ton broker accepte ce stop et ajoute un buffer.
- Si le SL et le TP sont touchés dans la même bougie, le **SL est prioritaire** (hypothèse prudente).
- **Un trade par jour** : seule la première cassure compte.
- Sur un timeframe supérieur à M1 avec simulation intrabar, TradingView limite le rejeu à environ 100 000 bougies 1 minute : l'historique testé est plus court.
- Testé sur **NAS100 CFD en M1**. Les heures sont pensées pour le NQ ; pour un autre marché, adapte les sessions.

---

## Avertissement

Outil d'aide à la décision, pas un conseil en investissement. Les statistiques affichées portent sur l'historique visible et ne garantissent pas les résultats futurs. Teste en bar replay ou en démo avant de trader réel.

## Licence

[Mozilla Public License 2.0](../../LICENSE) · © OS Trading (SakuraTradeLab)
