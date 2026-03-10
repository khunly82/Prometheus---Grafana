# Définition

*Les metrics sont des données numériques mesurées sur un intervalle de temps. C'est ce qui permet de créer des graphiques de performance.*

- **C'est quoi ?** Des compteurs ou des jauges (ex: CPU à 80%, 200 requêtes/seconde, 50ms de latence).
- **Pourquoi les utiliser ?** Pour l'alerte et la vue d'ensemble. C'est léger à stocker et très rapide à consulter.

# OpenMetrics

*C'est un standard ouvert sous l'égide de la <a href="https://www.cncf.io" target="_blank">`CNCF`</a> (Cloud Native Computing Fundation) qui définit comment les données de monitoring doivent être transmises via HTTP.*

## La structure du format

*Le format OpenMetrics est purement textuel, ce qui le rend lisible par l'humain. Chaque métrique est composée de trois éléments clés :*

- **HELP** : Une description de ce que mesure la métrique.
- **TYPE** : Le type de donnée (Counter, Gauge, Histogram, etc.).
- **La donnée** : Le nom de la métrique, ses labels (entre accolades) et sa valeur.

*Exemple :*

```prom
# HELP promhttp_metric_handler_requests_total Total number of scrapes by HTTP status code.
# TYPE promhttp_metric_handler_requests_total counter
promhttp_metric_handler_requests_total{code="200"} 1
promhttp_metric_handler_requests_total{code="500"} 0
promhttp_metric_handler_requests_total{code="503"} 0
```

## Types (liste non exhaustive)

- **counter** : C'est une valeur qui ne fait qu'augmenter (ou revient à zéro si le service redémarre). On ne regarde jamais sa valeur brute, mais sa vitesse d'évolution (le "rate"). *ex: Nombre total de requêtes HTTP, nombre d'erreurs, ...*
- **gauge** : C'est une valeur qui peut monter ou descendre. Elle représente un état instantané. *ex: Utilisation de la RAM, température CPU, nombre de personnes connectées, ...*
- **histogram** : Compte le nombre d'événements qui tombent dans des "paniers" (buckets) de tailles prédéfinies. *ex: Temps de réponse d'une API (combien de requêtes ont mis moins de 100ms ? moins de 500ms ?).*
- **stateSet** : C'est une variante de la Gauge, mais au lieu d'une valeur numérique fluctuante, elle représente un ensemble de booléens (vrai/faux) pour un même objet. *ex : L'état des services sur un serveur.*
- **info** : Ce type est utilisé pour exposer des données qui ne changent jamais (ou très rarement) et qui ne sont pas vraiment des mesures, mais des métadonnées. *ex: Savoir quelle version de ton logiciel tourne sur quel serveur.*

## Quelques métriques (Windows) ...

#### CPU

- **windows_cpu_processor_utility_total** : Permet de calculer le % d'utilisation CPU. *Si elle est proche de 100% sur tous les cœurs, le serveur sature.*
- **windows_system_processor_queue_length** : Nombre de processus qui attendent leur tour pour utiliser le CPU. *Si ce chiffre est supérieur à 2x le nombre de cœurs, le CPU est débordé.*

### RAM

- **windows_memory_available_bytes** : La quantité de RAM physiquement libre.
- **windows_memory_committed_bytes** : La mémoire totale utilisée (incluant le fichier d'échange/pagefile). *C'est souvent plus précis que la RAM physique pour détecter une saturation.*
- **go_memstats_heap_alloc_bytes** : Voir combien de mémoire le programme consomme réellement.

### Disques

- **windows_logical_disk_free_bytes** : Crucial pour les alertes. *Si le disque C: tombe à 0, le serveur plante.*
- **windows_logical_disk_read_latency_seconds** : write_latency_seconds : Mesure si le disque est lent. *Au-delà de 0.02s (20ms), les utilisateurs vont ressentir des lenteurs.*

### Réseau

- **windows_net_bytes_total** : Le trafic total qui entre et sort de la carte réseau.
- **windows_net_packets_errors_total** : Nombre d'erreurs de paquets réseau. *Si ce chiffre augmente, cela signifie des problèmes de câblage ou de configuration réseau (perte de paquets).*

### Système et Disponibilité

- **windows_system_system_calls_total** : Donne une idée de l'activité globale de l'OS.
- **windows_system_boot_time_timestamp** : Permet de calculer l'Uptime (depuis combien de temps le serveur est allumé). *Calcul Grafana : time() - windows_system_boot_time_timestamp*
- **windows_service_state** : Pour vérifier si les services critiques (Active Directory, SQL Server, IIS) sont bien en état running.

## Linux vs Windows

| Métrique Windows                        | Équivalent Linux (Node Exporter)      |
|-----------------------------------------|---------------------------------------|
| windows_cpu_processor_utility_total     | node_cpu_seconds_total                |
| windows_physical_memory_free_bytes      | node_memory_MemAvailable_bytes        |
| windows_logical_disk_free_bytes         | node_filesystem_avail_bytes           |
| windows_system_boot_time_timestamp      | node_boot_time_seconds                |
| windows_net_bytes_total                 | node_network_receive_bytes_total      |
