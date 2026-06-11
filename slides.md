---
theme: default
title: Collecte de metrics avec Prometheus
transition: slide-left
comark: true
magicMoveDuration: 333

layout: intro
---

# Collecte de metrics avec Prometheus

## Mise en jambe du 17/06/2026

---
layout: intro
---

# Pourquoi ?

## On a déjà des metrics dans DataDog non ?

<!--
 - metrics dans DataDog :
   - construits à partir des logs
   - ou pas
   - coûtent un peu cher (surtout selon facteur...)
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

<!--
 - cardinalité de la metric = produit cardinalité labels
 - augmente vite (1 label par container chez Pix)
-->

---
layout: intro
---

# Types de metrics

 - Counter
 - Gauge
 - Histogram
 - Summary

<style>
li {
  font-size: 2em;
}
</style>

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

<style>
.slidev-code {
  --slidev-code-font-size: 18px;
  --slidev-code-line-height: 24px;
}
</style>

# Counter - Collecte dans l’API

````md magic-move
```js
import { createCounter } from 'src/shared/infrastructure/metrics/metrics.js';

const readsMetric = createCounter({
  name: 'lc_reads',
  help: 'Learning content cache reads',
  labelNames: ['table', 'cache'],
});
```
```js
import { createCounter } from 'src/shared/infrastructure/metrics/metrics.js';

const readsMetric = createCounter({
  name: 'lc_reads',
  help: 'Learning content cache reads',
  labelNames: ['table', 'cache'],
});

// puis :

readsMetric.inc({ table, cache });    // incrémenter de 1
readsMetric.inc({ table, cache }, n); // incrémenter de n
```
```js
import { createCounter } from 'src/shared/infrastructure/metrics/metrics.js';

const readsMetric = createCounter({
  name: 'lc_reads',
  help: 'Learning content cache reads',
  labelNames: ['table', 'cache'],
});

// ou bien dans une classe :

this.#readsMetric = readsMetric.labels({ table, cache });

this.#readsMetric.inc();  // incrémenter de 1
this.#readsMetric.inc(n); // incrémenter de n
```
````

<!--
[Exemple dans Grafana](https://grafana.production.pix.digital/explore?schemaVersion=1&panes=%7B%22mey%22:%7B%22datasource%22:%22prometheus%22,%22queries%22:%5B%7B%22refId%22:%22A%22,%22expr%22:%22pix_api_lc_cachemiss_count%7Binstance%3D%5C%22web-1%5C%22,%20table%3D%5C%22challenges%5C%22,%20cache%3D%5C%22entities%5C%22%7D%22,%22range%22:true,%22instant%22:true,%22datasource%22:%7B%22type%22:%22prometheus%22,%22uid%22:%22prometheus%22%7D,%22editorMode%22:%22builder%22,%22legendFormat%22:%22__auto%22%7D%5D,%22range%22:%7B%22from%22:%22now-1h%22,%22to%22:%22now%22%7D,%22compact%22:false%7D%7D&orgId=1)
-->

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

<style>
.slidev-code {
  --slidev-code-font-size: 18px;
  --slidev-code-line-height: 24px;
}
</style>

# Gauge - Collecte dans l’API

````md magic-move
```js
import { createGauge } from 'src/shared/infrastructure/metrics/metrics.js';

const cacheSizeMetric = createGauge({
  name: 'lc_cachesize',
  help: 'Learning content cache size',
  labelNames: ['table', 'cache'],
});
```
```js
import { createGauge } from 'src/shared/infrastructure/metrics/metrics.js';

const cacheSizeMetric = createGauge({
  name: 'lc_cachesize',
  help: 'Learning content cache size',
  labelNames: ['table', 'cache'],
});

// puis :

cacheSizeMetric.set({ table, cache }, value); // mettre à jour
cacheSizeMetric.inc({ table, cache });        // incrémenter
cacheSizeMetric.dec({ table, cache });        // décrémenter
cacheSizeMetric.reset({ table, cache });      // réinitialiser
```
```js
import { createGauge } from 'src/shared/infrastructure/metrics/metrics.js';

const cacheSizeMetric = createGauge({
  name: 'lc_cachesize',
  help: 'Learning content cache size',
  labelNames: ['table', 'cache'],
});

// ou bien dans une classe :

this.#cacheSizeMetric = cacheSizeMetric.labels({ table, cache });

this.#cacheSizeMetric.set(value); // mettre à jour
this.#cacheSizeMetric.inc();      // incrémenter
this.#cacheSizeMetric.dec();      // décrémenter
this.#cacheSizeMetric.reset();    // réinitialiser
```
```js
import { createGauge } from 'src/shared/infrastructure/metrics/metrics.js';

const cacheSizeMetric = createGauge({
  name: 'lc_cachesize',
  help: 'Learning content cache size',
  labelNames: ['table', 'cache'],
  // ou alors avec une fonction de collecte :
  collect() {
    this.set({ table, cache }, cache.size());
  },
});
```
````

<!--
[Exemple dans Grafana](https://grafana.production.pix.digital/explore?schemaVersion=1&panes=%7B%22mey%22:%7B%22datasource%22:%22prometheus%22,%22queries%22:%5B%7B%22refId%22:%22A%22,%22expr%22:%22pix_api_lc_cachesize%7Binstance%3D%5C%22web-1%5C%22,%20table%3D%5C%22challenges%5C%22,%20cache%3D%5C%22entities%5C%22%7D%22,%22range%22:true,%22instant%22:true,%22datasource%22:%7B%22type%22:%22prometheus%22,%22uid%22:%22prometheus%22%7D,%22editorMode%22:%22builder%22,%22legendFormat%22:%22__auto%22%7D%5D,%22range%22:%7B%22from%22:%22now-1h%22,%22to%22:%22now%22%7D,%22compact%22:false%7D%7D&orgId=1)
-->

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

- `prix_plein_euro_count` : Nombre total de pleins effectués
- `prix_plein_euro_sum` : Total du prix de tous les pleins effectués
- `prix_plein_euro_bucket` : Répartition des pleins en fonction de leur prix avec le label **le** :
  - `le="50"` : inférieur ou égal à 50€
  - `le="75"` : inférieur ou égal à 75€
  - `le="100"` : inférieur ou égal à 100€
  - `le="+Inf"` : inférieur ou égal à l’infini

<!--
- augmentation supplémentaire cardinalité
-->

---

# Histogram - Collecte dans l’API

````md magic-move
```js
import { createHistogram } from 'src/shared/infrastructure/metrics/metrics.js';

const cacheMissMetric = createHistogram({
  name: 'lc_cachemiss',
  help: 'Learning content cache miss count',
  labelNames: ['table', 'cache'],
});
```
```js
import { createHistogram } from 'src/shared/infrastructure/metrics/metrics.js';

const cacheMissMetric = createHistogram({
  name: 'lc_cachemiss',
  help: 'Learning content cache miss count',
  labelNames: ['table', 'cache'],
});

// puis

cacheMissMetric.observe({ table, cache }, value); // observer une valeur
```
```js
import { createHistogram } from 'src/shared/infrastructure/metrics/metrics.js';

const cacheMissMetric = createHistogram({
  name: 'lc_cachemiss',
  help: 'Learning content cache miss count',
  labelNames: ['table', 'cache'],
});

// ou bien dans une classe

this.#cacheMissMetric = cacheMissMetric.labels({ table, cache });

this.#cacheMissMetric.observe(value); // observer une valeur
```
````

<!--
Exemples dans Grafana :
 - [Nombre d’observations](https://grafana.production.pix.digital/explore?schemaVersion=1&panes=%7B%22djw%22:%7B%22datasource%22:%22prometheus%22,%22queries%22:%5B%7B%22refId%22:%22A%22,%22expr%22:%22rate%28pix_api_lc_read_count%7Bcache%3D%5C%22entities%5C%22,%20table%3D%5C%22skills%5C%22,%20instance%3D%5C%22web-1%5C%22%7D%5B$__rate_interval%5D%29%22,%22range%22:true,%22instant%22:true,%22datasource%22:%7B%22type%22:%22prometheus%22,%22uid%22:%22prometheus%22%7D,%22editorMode%22:%22builder%22,%22legendFormat%22:%22__auto%22%7D%5D,%22range%22:%7B%22from%22:%22now-24h%22,%22to%22:%22now%22%7D,%22compact%22:false%7D%7D&orgId=1)
 - [Somme des observations](https://grafana.production.pix.digital/explore?schemaVersion=1&panes=%7B%22djw%22:%7B%22datasource%22:%22prometheus%22,%22queries%22:%5B%7B%22refId%22:%22A%22,%22expr%22:%22rate%28pix_api_lc_read_sum%7Bcache%3D%5C%22entities%5C%22,%20table%3D%5C%22skills%5C%22,%20instance%3D%5C%22web-1%5C%22%7D%5B$__rate_interval%5D%29%22,%22range%22:true,%22instant%22:true,%22datasource%22:%7B%22type%22:%22prometheus%22,%22uid%22:%22prometheus%22%7D,%22editorMode%22:%22builder%22,%22legendFormat%22:%22__auto%22%7D%5D,%22range%22:%7B%22from%22:%22now-24h%22,%22to%22:%22now%22%7D,%22compact%22:false%7D%7D&orgId=1)
 - [Buckets](https://grafana.production.pix.digital/explore?schemaVersion=1&panes=%7B%22djw%22:%7B%22datasource%22:%22prometheus%22,%22queries%22:%5B%7B%22refId%22:%22A%22,%22expr%22:%22rate%28pix_api_lc_read_bucket%7Bcache%3D%5C%22entities%5C%22,%20table%3D%5C%22skills%5C%22,%20instance%3D%5C%22web-1%5C%22%7D%5B$__rate_interval%5D%29%22,%22range%22:true,%22instant%22:true,%22datasource%22:%7B%22type%22:%22prometheus%22,%22uid%22:%22prometheus%22%7D,%22editorMode%22:%22builder%22,%22legendFormat%22:%22__auto%22%7D%5D,%22range%22:%7B%22from%22:%22now-1h%22,%22to%22:%22now%22%7D,%22compact%22:false%7D%7D&orgId=1)
-->

---

# Histogram - Collecte dans l’API

````md magic-move
```js
import { createHistogram } from 'src/shared/infrastructure/metrics/metrics.js';

const cachePenaltyMetric = createHistogram({
  name: 'lc_cachepenalty',
  help: 'Learning content cache penalty',
  labelNames: ['table', 'cache'],
});
```
```js
import { createHistogram } from 'src/shared/infrastructure/metrics/metrics.js';

const cachePenaltyMetric = createHistogram({
  name: 'lc_cachepenalty',
  help: 'Learning content cache penalty',
  labelNames: ['table', 'cache'],
});

// puis (dans une classe)

this.#cachePenaltyMetric = cachePenaltyMetric.labels({ table, cache });

const stopTimer = this.#cachePenaltyMetric.startTimer(value); // démarrer un timer
```
```js
import { createHistogram } from 'src/shared/infrastructure/metrics/metrics.js';

const cachePenaltyMetric = createHistogram({
  name: 'lc_cachepenalty',
  help: 'Learning content cache penalty',
  labelNames: ['table', 'cache'],
});

// puis (dans une classe)

this.#cachePenaltyMetric = cachePenaltyMetric.labels({ table, cache });

const stopTimer = this.#cachePenaltyMetric.startTimer(value); // démarrer un timer

// puis plus tard

stopTimer(); // arrêter le timer
```
````

---

# Summary

### Comme les histograms mais avec "des **quantiles** configurables caluclés sur une fenêtre de temps glissante"

Pour en apprendre plus : https://prometheus.io/docs/practices/histograms/

---

# Des gotchas ?

 - On ne peut pas changer le type d’une metric (en gardant le même nom)

---

# Mais comment les metrics sont récupérées ?

## ![Schéma](/prometheus1.png)

---

# Mais comment les metrics sont récupérées ?

## ![Schéma](/prometheus2.png)

---
layout: intro
---

# Et OpenTelemetry alors ?

## Ce serait pas mieux d’utiliser un standard ?

<!--
- Prometheus/OpenTelemetry compatibles (prom_client compatible OpenTelemetry)
- OpenTelemetry = surensemble (metrics, traces, logs, et context propagation entre signaux)
- remplacer prom_client par SDK JS OpenTelemetry
-->

---

# Références

 - https://prometheus.io/docs/concepts/data_model/
 - https://prometheus.io/docs/concepts/metric_types/
 - https://www.npmjs.com/package/prom-client
 - https://opentelemetry.io/
 - https://www.npmjs.com/package/@opentelemetry/sdk-metrics

---
layout: cover
background: /LODGY.webp
---

# Merci !

## Avez-vous des questions ?
