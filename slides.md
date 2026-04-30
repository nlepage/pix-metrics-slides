---
theme: default

title: Collecte de metrics avec Prometheus

# info: |
#   ## Slidev Starter Template
#   Presentation slides for developers.

# slide transition: https://sli.dev/guide/animations.html#slide-transitions
transition: slide-left

# enable Comark Syntax: https://comark.dev/syntax/markdown
comark: true

layout: intro
---

# Collecte de metrics avec Prometheus

## Mise en jambe du ICI LA DATE

---
layout: intro
---

# Pourquoi ?

## On a déjà des metrics dans DataDog non ?

<!--
 - metrics dans DataDog :
   - construits à partir des logs
   - coûtent cher
-->

---
layout: intro
---

# C’est quoi une metric ?

## Un truc qu’on mesure au cours du temps et qui a **un nom**.

## Ce truc peut être décliné en fonction de **labels**.

---
layout: cover
background: /LODGY.webp
---

> On est attaché à la bagnole, on aime la bagnole.
>
> Et moi je l’adore.

Emmanuel Macron, au JT de France 2 et TF1 en 2023

<style>
blockquote {
  p {
    font-size: 2.4rem;
    line-height: 3rem !important;
  }
}
</style>

---

## La pression des pneus de la bagnole

- **Nom** : `pression_pneu_bar` (lettres, chiffres, `_` et `:`)

- **Labels** :
  - Quelle bagnole ?
    - **Nom** : `voiture` (mêmes caractères que pour le nom)
    - **Valeurs** : `Mon Lodgy` ou `Le SUV du voisin` (caractères UTF-8)

  - Quelle roue ?
    - **Nom** : `roue`
    - **Valeurs** : `AvG`, `AvD`, `ArG` ou `ArD`

Exemple de metric :

```
pression_pneu_bar{voiture="Le SUV du voisin", roue="ArG"}
```

---

## La pression des pneus de la bagnole

| Metric                                                      | Valeur |
| ----------------------------------------------------------- | ------ |
| `pression_pneu_bar{voiture="Mon Lodgy", roue="AvG"}`        | 2.4    |
| `pression_pneu_bar{voiture="Mon Lodgy", roue="AvD"}`        | 2.3    |
| `pression_pneu_bar{voiture="Mon Lodgy", roue="ArG"}`        | 2.7    |
| `pression_pneu_bar{voiture="Mon Lodgy", roue="ArD"}`        | 2.6    |
| `pression_pneu_bar{voiture="Le SUV du voisin", roue="AvG"}` | 3.1    |
| `pression_pneu_bar{voiture="Le SUV du voisin", roue="AvD"}` | 3.2    |
| `pression_pneu_bar{voiture="Le SUV du voisin", roue="ArG"}` | 2.8    |
| `pression_pneu_bar{voiture="Le SUV du voisin", roue="ArD"}` | 2.9    |

---
layout: intro
---

# Types de metrics

 - Gauge
 - Counter
 - Histogram
 - Summary

<style>
li {
  font-size: 2em;
}
</style>

---

# Gauge

### Une valeur numérique simple qui peut augmenter et réduire

Exemples :

- Pression des pneus de la bagnole
- Niveau d’huile de la bagnole
- Température du moteur de la bagnole

Mais aussi :

- Consommation CPU de l’ordinateur de bord de la bagnole
- Consommation de RAM de l’ordinateur de bord de la bagnole
- Nombre de process dans l’ordinateur de bord de la bagnole

---

# Counter

### Une valeur cumulée qui peut seulement augmenter ou être réinitialisée

Exemples :

- Nombre de kilomètres parcourus par la bagnole
- Quantité de gazole consommée par la bagnole
- Nombre d’allers-retours des essuies glaces de la bagnole

Mais aussi :

- Nombre de requêtes HTTP traîtées
- Nombre de jobs exécutés
- Nombre de requêtes SQL exécutées

---

# Histogram

### Enregistre des observations en les comptant dans des **buckets** configurables
### Fournit également le compte et la somme de toutes les valeurs observées

Exemples :

- Prix du plein de la bagnole en euros
- Volume du plein de la bagnole en litres

Mais aussi :

- Durées de requêtes HTTP
- Tailles de réponses HTTP
- Temps d’exécution de requêtes SQL

---

# Histogram

### Enregistre en fait trois metrics

Par exemple pour le prix du plein de la bagnole :

- `prix_carburant_litre_euro_count` : Nombre total de pleins effectués
- `prix_carburant_litre_euro_sum` : Total de l’argent dépensé dans tous les pleins effectués
- `prix_carburant_litre_euro_bucket` : Répartition des pleins en fonction de leur prix avec le label **le** :
  - `le="50"` : inférieur ou égal à 50€
  - `le="75"` : inférieur ou égal à 75€
  - `le="100"` : inférieur ou égal à 100€
  - `le="+Inf"` : inférieur ou égal à l’infini

---

# Summary

### Comme les histograms mais avec "des **quantiles** configurables caluclés sur une fenêtre de temps glissante"

Pour en apprendre plus : https://prometheus.io/docs/practices/histograms/

---

# Références

 - https://prometheus.io/docs/concepts/data_model/
 - https://prometheus.io/docs/concepts/metric_types/

---
layout: cover
background: /LODGY.webp
---

# Merci !
## Avez-vous des questions ?
