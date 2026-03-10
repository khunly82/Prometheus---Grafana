# Introduction

## Prometheus

*`Prometheus` est une base de données orientée séries temporelles.* 

- **Son rôle** : Aller chercher (on dit `scrapper` ou `puller` ) des chiffres (`metrics`) sur des serveurs ou applications à intervalles réguliers.
- **Ce qu'il surveille** : L'utilisation du processeur (CPU), la mémoire vive, le nombre de visiteurs sur un site, ou encore le temps de réponse d'une base de données.
- **Son point fort** : Les alertes. Si un serveur dépasse 95% d'utilisation, Prometheus le remarque immédiatement et peut envoyer un signal.

*Remarque: par défaut ces metrics sont conservées 15 jours*

## Grafana

*`Grafana` ne stocke aucune donnée. C'est une interface de visualisation.* 

- **Son rôle** : Se connecter à des outils commme Prometheus pour transformer les données en tableaux de bord interactifs.
- **Ce qu'il fait** : Il crée des « Dashboards ». C'est l'écran que l'on projette souvent dans les bureaux des équipes techniques pour voir l'état de santé du système en temps réel.
- **Son point fort** : La polyvalence. Il peut mixer des données venant de Prometheus avec d'autres sources en même temps.

# Les 3 piliers de l'observabilité

## [Metrics](2_metrics.md)

*Les metrics sont des données numériques mesurées sur un intervalle de temps. C'est ce qui permet de créer des graphiques de performance.*

- **C'est quoi ?** Des compteurs ou des jauges (ex: CPU à 80%, 200 requêtes/seconde, 50ms de latence).
- **Pourquoi les utiliser ?** Pour l'alerte et la vue d'ensemble. C'est léger à stocker et très rapide à consulter.
- **Outils** : Prometheus 

## Logs

*Les logs sont des lignes de texte générées par l'application ou le système lorsqu'un événement précis survient.*

- **C'est quoi ?** "L'utilisateur X s'est connecté", "Erreur critique : base de données inaccessible à 14h02".
- **Pourquoi les utiliser ?** Pour comprendre le « quoi ». Une metric peut montrer que le taux d'erreur augmente, mais seul le log dira exactement quelle erreur s'est produite.
- **Outils** : Loki

## Traces

*Les traces (ou Distributed Tracing) suivent le cheminement d'une requête unique à travers tous les services d'une architecture (microservices).*

- **C'est quoi ?** Un diagramme qui montre combien de temps une requête a passé dans le service A, puis le service B, puis la base de données.
- **Pourquoi les utiliser ?** Pour identifier les goulots d'étranglement. Si une page met 5 secondes à charger, la trace te montre exactement quel service ralentit tout le système.
**Outils** : Tempo

# Installation

## Avec Docker

*bash*
```sh
docker run -d --name prometheus -p 9090:9090 -v ${pwd}/config/prometheus.yml:/etc/prometheus/prometheus.yml prom/prometheus:latest
```

*powershell*
```powershell
docker run -d --name prometheus -p 9090:9090 -v ${PWD}\config\prometheus.yml:/etc/prometheus/prometheus.yml prom/prometheus:latest
```

*Remarque: `pwd/PWD` (print working directory). c'est le dossier courant!*

## Configuration

*Le fichier `prometheus.yml` permet de configurer prometheus, il est divisé en 3 parties (principales)*

## Configuration globale

```yml
global:
  # Fréquences de collecte des métriques
  # Trop courte → surcharge Prometheus
  # Trop longue → métriques moins précises
  scrape_interval: 15s

  # Timeout pour chaque requête de scraping
  scrape_timeout: 10s

  # Retention des labels par défaut
  evaluation_interval: 15s
```

## Configuration des cibles

*Applications pour lesquels on souhaite collecter les données*

```yml
scrape_configs:
  # Prometheus lui-même
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]

  # Exemple pour une autre app
  - job_name: "my-app"
    # Url sur laquelle les métriques sont exposées (default: /metrics)
    metrics_path: /metrics_path
    static_configs:
      - targets: ["host.docker.internal:3000"]  
```

*Rappel: `host.docker.internal` (natif sur Windows) permet au conteneur docker de contacter la machine « host ». Pas nécessaire si l'app se trouve sur le même network que Prometheus*

*Remarque: Prometheus récolte ses propres données, c'est facultatif mais fortement recommandé!* 

*Utiles pour*

- *Mesurer les performances du serveur Prometheus (CPU, mémoire, temps de scrapping)*
- *Connaître la santé des jobs de scraping (est-ce que Prometheus arrive à collecter les métriques de tes autres services ?)*
- *Faire des alertes internes (par exemple, Prometheus peut générer une alerte si le scrapping rate est trop élevé ou si une cible ne répond pas)*

## Configuration des alertes (facultatif)

*Sert à indiquer à Prometheus où envoyer les alertes. (Nécessite au moins un alertManager)*

```yml
alerting:
  # Liste des « alertmanagers » que Prometheus peut contacter.
  alertmanagers:
    - static_configs:
      - targets: ["localhost:9093"]
```

# Collecte de métriques (Linux vs Windows)

*Prometheus ne sait pas lire les données système tout seul. Il a besoin d'un agent « Exporter » installé sur chaque machine à surveiller.*

## 1. Windows

*Remarque: Si vous travaillez sur Windows avec Docker Desktop, n'utilisez **pas** `node-exporter` en conteneur. Il ne verra pas votre vrai système.*

- **L'agent** : `windows_exporter` (anciennement WMI exporter).
- **Installation** : Téléchargez le `.msi` depuis le [GitHub officiel](https://github.com/prometheus-community/windows_exporter/releases) et lancez-le.
- **Fonctionnement** : Il tourne comme un service Windows en arrière-plan.
- **Port par défaut** : `9182`
- **Vérification** : Accédez à `http://localhost:9182/metrics` sur votre navigateur.

`prometheus.yml`

```yml
- job_name: "windows-host"
    static_configs:
      - targets: ["host.docker.internal:9182"]
```

## 2. Linux

*Pour une machine Linux, on utilise l'agent standard de l'écosystème.*

* **L'agent** : `node_exporter`.
* **Installation** : **Native (Recommandé) :** Télécharger le binaire et le lancer comme service (systemd).
    * **Docker** : Possible, mais nécessite des accès spécifiques au système hôte (`--pid="host"`).
* **Port par défaut** : `9100`.

`prometheus.yml`

```yml
- job_name: "linux-host"
    static_configs:
      - targets: ["host.docker.internal:9100"]
```