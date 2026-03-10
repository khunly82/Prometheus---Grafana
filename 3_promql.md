# Guide Complet : PromQL (Prometheus Query Language)

*PromQL est un langage de requête fonctionnel conçu pour extraire et traiter des données de séries temporelles. Contrairement au SQL, il n'est pas fait pour les relations entre tables, mais pour le calcul mathématique sur des flux de données.*

---

## 1. Les types de données (Indispensable)

PromQL manipule quatre types de données, mais deux sont au cœur de 99% des requêtes :

- **Instant Vector (Vecteur instantané)** : Un ensemble de séries temporelles où chaque série possède un seul échantillon correspondant au même instant précis (le plus proche de "maintenant").
  * *Exemple : `node_memory_Active_bytes`*
- **Range Vector (Vecteur de plage)** : Un ensemble de séries temporelles contenant une liste de points de données sur une durée définie. On le reconnaît aux crochets `[]`.
  * *Exemple : `node_cpu_seconds_total[5m]`*
- **Scalar (Scalaire)** : Une simple valeur numérique flottante (ex: `10.5`).
- **String (Chaîne)** : Une simple chaîne de caractères (rarement utilisée).

---

## 2. Sélecteurs et Filtres (Labels)

Les filtres permettent de passer de "tous les disques" à "un disque spécifique".

### Opérateurs de comparaison
| Opérateur | Signification          | Exemple                  |
|-----------|------------------------|--------------------------|
| **`=`**   | Égalité stricte        | `{job="prometheus"}`     |
| **`!=`**  | Différence             | `{status!="200"}`        |
| **`=~`**  | Regex (correspondance) | `{env=~"prod\|staging"}` |
| **`!~`**  | Regex (exclusion)      | `{instance!~"test-.*"}`  |

> **Astuce :** On peut combiner plusieurs labels : `{job="api", method="GET", status!~"5.."}`.

---

## 3. Fonctions de Calcul de Taux (Rate vs Irate)

Ces fonctions ne s'utilisent **QUE** sur des **Counters**.

### `rate()` (Moyenne lissée)
Calcule l'augmentation par seconde sur toute la période (ex: `[5m]`).
- **Usage** : Alerting et graphiques de tendance.
- **Avantage** : Lisse les pics, plus stable.

### `irate()` (Instantané)
Calcule le taux basé uniquement sur les deux derniers points de la plage.
- **Usage** : Tableaux de bord de supervision en temps réel.
- **Avantage** : Très réactif aux changements brusques.

### `increase()`
Donne l'augmentation totale sur la période.
- *Ex : `increase(http_requests_total[1h])` donne le nombre total de requêtes durant la dernière heure.*

---

## 4. Agrégations (Réduction de données)

L'agrégation permet de regrouper les données de plusieurs instances ou labels.

- **`sum()`** : Somme totale.
- **`avg()`** : Moyenne.
- **`min()` / `max()`** : Valeurs extrêmes.
- **`count()`** : Nombre de séries (ex: combien de serveurs sont UP).
- **`quantile(0.95, ...)`** : Le 95ème percentile (très utilisé pour la latence).

### Clause `by` et `without`
- **`by`** : Garde uniquement les labels cités.
  * `sum(rate(...)) by (instance)` -> Un résultat par serveur.
- **`without`** : Supprime les labels cités et garde le reste.

---

## 5. Opérateurs Logiques et Arithmétiques

### Arithmétique entre vecteurs
On peut diviser deux métriques, mais elles doivent avoir les **mêmes labels**.
* *Exemple (Taux d'erreur) :*
    `sum(rate(http_errors[5m])) / sum(rate(http_requests_total[5m]))`

### Opérateurs de comparaison (Filtres de sortie)
Utilisés pour les alertes :
- `metric > 80` : Ne retourne que les séries au-dessus de 80.
- `metric == 1` : Souvent utilisé pour vérifier un état (ex: `up == 0`).

---

## 6. Requêtes "Avancées" pour Windows (WMI Exporter)


### CPU par cœur (Détail)
`100 - (irate(windows_cpu_time_total{mode="idle"}[5m]) * 100)`
*Affiche la charge de chaque CPU individuellement.*

### Latence Disque (Lecture)
`rate(windows_logical_disk_read_latency_seconds_total[5m]) / rate(windows_logical_disk_reads_total[5m])`
*Donne le temps moyen par lecture (doit être < 0.02s).*

### Prédiction de remplissage FS
`predict_linear(windows_logical_disk_free_bytes{volume="C:"}[6h], 3600 * 24) < 0`
*Analyse la tendance des 6 dernières heures pour prédire si le disque sera plein dans 24h.*

---

## Synthèse : Workflow d'une requête

1. **Sélectionner** la métrique : `http_requests_total`
2. **Filtrer** par labels : `{method="POST"}`
3. **Définir la plage** (si besoin d'une fonction) : `[5m]`
4. **Appliquer la fonction** : `rate(...)`
5. **Agréger** le résultat : `sum(...) by (instance)`