# Changelog

## London Breakout NQ

### 1.0 · 2026-11-27
- Première publication.
- Range London 04:00-09:00 ET, fenêtre d'entrée 09:30-11:00, sortie forcée 16:00.
- Entrée à la première cassure du High/Low, SL au-delà de la bougie de cassure, TP en R, TP1 partiel + BE en option.
- Jours sautés : ouverture hors range, deux côtés cassés dans la même bougie, filtre de taille du range (points ou × ATR jour).
- Simulation intrabar 1 minute sur les timeframes supérieurs.
- Panneau « Aujourd'hui », tableau de statistiques, grille SL fixe = k × range.
- Alertes : range prêt, cassure en temps réel, entrée.

## CC Model PRO

### 1.0 · 2026-11-25
- Première publication.
- Modèle CC TheStrat (bougie signal + confirmation failed 2).
- Filtre de biais : structure BOS/CHoCH (ICT), bougie TheStrat ou aucun.
- Exécution : entrée marché ou limite selon le RR minimum, stop symétrique ou structurel, TP1 / TP2 avec passage à BE.
- Calcul du lot selon capital et risque, décalage de prix broker MT5.
- Alertes texte ou JSON (webhook bridge MT5).
- Tableau de statistiques et alerte sur le taux de TP1 glissant.
