# CC Model PRO · TheStrat Candle Confirmation

Indicateur TradingView (Pine Script v6) qui détecte le modèle **CC (Candle Confirmation) de TheStrat**, le filtre avec un **biais de structure ICT (BOS/CHoCH)** et affiche directement l'exécution : entrée, stop, TP1, TP2 et taille de lot.

> Concept : modèle CC de TheStrat, tel que présenté par Trader Mike. Ce script est une implémentation indépendante d'OS Trading, avec un filtre de biais et une aide à l'exécution ajoutés.

---

## Sommaire

1. [Installation](#1-installation)
2. [Le modèle en 3 étapes](#2-le-modèle-en-3-étapes)
3. [Lire le bandeau](#3-lire-le-bandeau)
4. [Lire le graphique](#4-lire-le-graphique)
5. [Paramètres](#5-paramètres)
6. [Alertes (texte ou JSON pour MT5)](#6-alertes)
7. [Tableau de statistiques](#7-tableau-de-statistiques)
8. [Limites connues](#8-limites-connues)

---

## 1. Installation

1. Ouvre [`CC_Model_PRO.pine`](./CC_Model_PRO.pine), clique sur **Copy raw file** (icône copier en haut à droite du code).
2. Sur TradingView : **Pine Editor** (bas de l'écran) → **Ouvrir** → **Nouvel indicateur**.
3. Supprime le code par défaut, colle le script, **Enregistrer**, puis **Ajouter au graphique**.
4. Mets le graphique sur le bon timeframe :

| Setup | Graphique (exécution) | Bougie signal | Biais (défaut) |
|---|---|---|---|
| **A** | M15 | H1 | H4 |
| **B** | M5 | M15 | H4 |

Le timeframe signal est choisi automatiquement : graphique ≤ M5 → M15 · M15 → H1 · H1 → H4.

---

## 2. Le modèle en 3 étapes

**① Bougie signal (timeframe supérieur)**
- Haussier : la bougie clôture **au-dessus du plus haut** de la bougie précédente.
- Baissier : la bougie clôture **sous le plus bas** de la bougie précédente.
- Si la bougie a aussi cassé l'autre extrême (outside bar), elle est marquée « 3 », sinon « 2 ».
- **Cible CC** = le plus haut (long) ou le plus bas (short) de la bougie signal.

**② Confirmation « F2 » (failed 2) sur le graphique**
Pendant la fenêtre de confirmation (par défaut la première moitié de la bougie suivante, 2 bougies minimum) :
- Long : une bougie passe **sous le plus bas** de la précédente puis **clôture au-dessus**.
- Short : une bougie passe **au-dessus du plus haut** de la précédente puis **clôture en dessous**.
- La **ligne orange** montre le niveau exact à surveiller ; elle se met à jour à chaque clôture.

**③ Exécution**
- Si la clôture du F2 donne au moins le RR minimum vers TP1 → **entrée au marché**.
- Sinon → **ordre limite** au prix qui donne ce RR.
- Le setup est annulé si la cible est atteinte avant le F2, ou si la fenêtre expire.

**Exemple (NAS100, setup A, biais H4 haussier)**
- La bougie H1 de 10:00 NY clôture au-dessus du plus haut de celle de 09:00 → setup **LONG aligné**, cible = plus haut de la bougie 10:00.
- À 11:15 en M15, une bougie mèche sous le plus bas de la précédente puis clôture au-dessus → **F2**.
- Le bandeau affiche `✔ LONG MAINTENANT` avec l'entrée, le SL, le TP1, le TP2 et le lot.
- TP1 touché → 50 % sortis, stop à breakeven, le reste court vers TP2.

---

## 3. Lire le bandeau

Seul le bandeau (en haut au centre) dicte l'action :

| Bandeau | Action |
|---|---|
| Couleur du biais (`H4 ▲ HAUSSIER · LONGS UNIQUEMENT`) | Pas de trade, attendre un setup aligné |
| `◉ EN PRÉPARATION` | Setup aligné : préparer l'ordre (lot déjà calculé) |
| `⚡ F2 EN FORMATION` | La bougie en cours forme le F2 : prêt à entrer à la clôture |
| `✔ LONG / SHORT MAINTENANT` | Entrer : entrée, SL, TP et lot sont écrits |
| `⏳ LIMITE` | Poser l'ordre limite au prix affiché |
| `● EN POSITION` / `● RUNNER` | Gérer la position (TP1, puis runner à BE vers TP2) |
| `✖ CONTRE-BIAIS` | Signal fantôme gris : **ne pas trader** |
| `⚠ TP1 … % → STOP & ANALYSE` | Le taux de réussite récent est passé sous ton seuil |

---

## 4. Lire le graphique

| Élément | Signification |
|---|---|
| Traits noirs fins | Début (plein) et fin (tirets) de la session des signaux |
| Boîte pleine « ALIGNÉ ✔ » | Bougie signal tradable |
| Boîte grise pointillée « CONTRE-BIAIS ✖ » | Signal contre le biais : jamais d'alerte, jamais de lignes d'entrée |
| Ligne orange pointillée | Zone F2 : le niveau à surveiller pour la confirmation |
| Ligne blanche | Entrée (pleine = marché, tirets = limite) |
| Ligne rouge | Stop loss |
| Lignes vertes | TP1 (pleine) / TP2 (tirets) |
| Ligne grise en escalier | Niveau de flip du biais : une clôture au-delà = CHoCH |

---

## 5. Paramètres

### 1 · Modèle
| Paramètre | Défaut | Rôle |
|---|---|---|
| Timeframe signal | auto | Vide = choix automatique selon le graphique |
| Bougies de confirmation max | 0 (auto) | Auto = première moitié de la bougie suivante, minimum 2 |
| Accepter une confirmation tardive | non | Désactivé = version stricte du modèle |

### 2 · Filtre de biais
| Paramètre | Défaut | Rôle |
|---|---|---|
| Filtre de biais | Structure BOS/CHoCH (ICT) | Tendance = dernier swing cassé **en clôture** sur le timeframe du biais. Autres choix : bougie TheStrat (original) ou aucun (déconseillé) |
| Timeframe du biais | H4 | |
| Force des swings | 5 | Bougies de chaque côté pour valider un swing. Plus haut = biais plus lent et plus stable |
| Signaux contre-biais en gris | oui | Affichés et simulés à part pour comparer les stats |
| Teinter le fond selon le biais | non | |
| Afficher le niveau de flip | oui | |

### 3 · Exécution
| Paramètre | Défaut | Rôle |
|---|---|---|
| RR minimum vers TP1 | 1.0 | En dessous, une limite est placée au prix qui donne ce RR |
| Stop | 1:1 symétrique | Symétrique = même distance que la cible (et au moins au-delà du F2). Structurel = extrême du F2 + marge |
| Marge au-delà du failed 2 | 0.30 | XAUUSD : 0.30 · NAS100 : 2 à 5 points. Ajoute ton spread pour les shorts |
| TP2 : 50 % au TP1, stop à BE | oui | |
| TP2 en R | 2.0 | |
| TP1 avant la cible | 0 | Sortir juste avant la liquidité. NAS100 : 2 à 3 points · XAUUSD : 0.30 |

### 4 · Risque & lot
| Paramètre | Défaut | Rôle |
|---|---|---|
| Capital ($) | 100 000 | |
| Risque par trade (%) | 0.25 | |
| Valeur de 1.0 de mouvement pour 1 lot ($) | 1.0 | MT5 → clic droit sur le symbole → Spécification → Taille du contrat. NAS100 : souvent 1 · XAUUSD : 100. **À vérifier chez ton broker** |
| Pas de lot | 0.01 | |
| Décalage broker | 0 | Prix MT5 − prix TradingView, mesuré au même instant. Les prix MT5 s'affichent alors entre parenthèses |

**Exemple de calcul du lot** : capital 100 000 $, risque 0,25 % → 250 $. Stop à 25 points sur NAS100 avec 1 $/point/lot → 250 / (25 × 1) = **10 lots**.

### 5 · Session
| Paramètre | Défaut | Rôle |
|---|---|---|
| Signaux seulement pendant la session | oui | Hors session : aucun nouveau setup, aucun F2 |
| Session (heure de New York) | 08:00-12:00 | = 13:00-17:00 Londres = 14:00-18:00 Paris |

### 6 · Alertes, 7 · Affichage, 8 · Suivi de performance
Voir les sections suivantes. Les couleurs, la taille du bandeau et la position du tableau sont réglables.

---

## 6. Alertes

Crée **une seule alerte** sur l'indicateur avec la condition **« N'importe quel appel de la fonction alert() »**. Seuls les setups **alignés** envoient des alertes.

### Format texte
```
CC Model · NAS100 · ✔ LONG MAINTENANT · Entrée 21 450.0 · SL 21 425.0 (25.0 pts) · TP1 21 475.0 · TP2 21 500.0 · LOT 10
```

### Format JSON (webhook vers un bridge MT5)
```json
{"symbol":"NAS100","action":"buy","entry":21450.0,"sl":21425.0,"tp1":21475.0,"tp2":21500.0,"lots":10,"msg":"..."}
```

| `action` | Quand |
|---|---|
| `info` | Nouveau setup, SL/TP touchés, alerte de performance |
| `prepare` | F2 en formation sur la bougie en cours |
| `buy` / `sell` | Entrée au marché à la clôture du F2 |
| `buy_limit` / `sell_limit` | Ordre limite à poser |
| `partial_be` | TP1 touché : fermer 50 % (`close_pct`) et stop à BE (`sl`) |
| `cancel` | Limite expirée ou cible atteinte sans remplissage : annuler l'ordre |

Le champ `symbol` vient du paramètre « Symbole MT5 ». Les prix tiennent compte du décalage broker.

---

## 7. Tableau de statistiques

Simulation sur l'historique visible, trades alignés uniquement (les contre-biais sont comptés à part) :

- Nombre de trades alignés et setups manqués (limite jamais remplie)
- % de TP1 atteint, global et sur les N derniers trades (défaut 20)
- R total et R moyen
- Détail long / short
- Détail avant / après l'ouverture cash de 09:30 NY
- Risque en $ par trade

Si le taux de TP1 glissant passe sous le seuil (défaut 55 %, à partir de 10 trades), une alerte part et le bandeau affiche `⚠ STOP & ANALYSE`.

---

## 8. Limites connues

- Stats **hors spread et slippage**.
- Stop et cible touchés dans la même bougie = compté comme **perte** (hypothèse prudente).
- **Un seul trade aligné suivi à la fois.**
- L'alerte `⚡ F2 en formation` part pendant la bougie : le F2 n'est valide qu'à la clôture.
- La bougie signal et le biais sont calculés sur des bougies clôturées (pas de repaint).
- Testé principalement sur **NAS100** et **XAUUSD**. Règle la valeur du point et la marge pour tout autre actif.

---

## Avertissement

Outil d'aide à la décision, pas un conseil en investissement. Les performances passées, y compris celles du tableau de statistiques, ne garantissent pas les résultats futurs. Teste en bar replay ou en démo avant de trader réel.

## Licence

[Mozilla Public License 2.0](../../LICENSE) · © OS Trading (SakuraTradeLab)
