Pour monitorer notre chatbot hébergé via AWS et Gradio, vous pouvez suivre ce runbook ajusté pour inclure la configuration spécifique nécessaire pour le monitoring du Chatbot et des endpoints associés.

## Runbook SRE pour le Monitoring d'un Chatbot Gradio

### Table des matières

1. [Pré-requis](#pré-requis)
2. [Installation de Prometheus](#installation-de-prometheus)
3. [Configuration de Prometheus](#configuration-de-prometheus)
4. [Installation de Grafana](#installation-de-grafana)
5. [Configuration de Grafana](#configuration-de-grafana)
6. [Ajout de sources de données et de tableaux de bord](#ajout-de-sources-de-données-et-de-tableaux-de-bord)
7. [Alerting avec Prometheus et Grafana](#alerting-avec-prometheus-et-grafana)
8. [Monitoring de l'instance Gradio](#monitoring-de-linstance-gradio)
9. [Dépannage](#dépannage)

### Pré-requis

- Serveur ou machine virtuelle pour héberger Prometheus et Grafana
- Docker installé sur le serveur (optionnel mais recommandé)
- Accès à internet pour télécharger les images et les dépendances

### Installation de Prometheus

1. **Télécharger et exécuter l'image Docker de Prometheus :**

    ```bash
    docker run -d --name=prometheus -p 9090:9090 prom/prometheus
    ```

2. **Vérifier que Prometheus est en cours d'exécution :**

    Accédez à `http://localhost:9090` dans votre navigateur.

### Configuration de Prometheus

1. **Créer un fichier de configuration `prometheus.yml` :**

    ```yaml
    global:
      scrape_interval: 15s

    scrape_configs:
      - job_name: 'gradio_chatbot'
        metrics_path: /metrics
        static_configs:
          - targets: ['<your-gradio-instance-domain>']
    ```

2. **Monter le fichier de configuration dans le conteneur Docker :**

    ```bash
    docker run -d --name=prometheus -p 9090:9090 -v /path/to/prometheus.yml:/etc/prometheus/prometheus.yml prom/prometheus
    ```

3. **Redémarrer le conteneur Prometheus pour appliquer la nouvelle configuration :**

    ```bash
    docker restart prometheus
    ```

### Installation de Grafana

1. **Télécharger et exécuter l'image Docker de Grafana :**

    ```bash
    docker run -d --name=grafana -p 3000:3000 grafana/grafana
    ```

2. **Vérifier que Grafana est en cours d'exécution :**

    Accédez à `http://localhost:3000` dans votre navigateur.
    - Les identifiants par défaut sont `admin`/`admin`.

### Configuration de Grafana

1. **Ajouter Prometheus comme source de données dans Grafana :**

    - Connectez-vous à Grafana.
    - Allez dans `Configuration` -> `Data Sources`.
    - Cliquez sur `Add data source` et sélectionnez `Prometheus`.
    - Configurez l'URL de Prometheus : `http://localhost:9090`.

2. **Enregistrer et tester la source de données.**

### Ajout de sources de données et de tableaux de bord

1. **Créer un nouveau tableau de bord :**

    - Allez dans `Create` -> `Dashboard`.
    - Cliquez sur `Add new panel`.

2. **Configurer un graphique pour afficher les métriques :**

    - Sélectionnez la source de données Prometheus.
    - Entrez une requête Prometheus, par exemple : `up{job="gradio_chatbot"}`.

3. **Enregistrer le tableau de bord :**

    - Donnez un nom au tableau de bord et cliquez sur `Save`.

### Alerting avec Prometheus et Grafana

1. **Configurer des règles d'alerte dans Prometheus :**

    - Ajouter des règles d'alerte dans `prometheus.yml` :

    ```yaml
    rule_files:
      - "alert.rules"

    alerting:
      alertmanagers:
      - static_configs:
        - targets:
          - 'localhost:9093'
    ```

    - Créer un fichier `alert.rules` :

    ```yaml
    groups:
      - name: example
        rules:
        - alert: InstanceDown
          expr: up == 0
          for: 1m
          labels:
            severity: page
          annotations:
            summary: "Instance {{ $labels.instance }} down"
            description: "{{ $labels.instance }} of job {{ $labels.job }} has been down for more than 1 minute."
    ```

2. **Configurer l'Alertmanager :**

    - Télécharger et exécuter l'Alertmanager :

    ```bash
    docker run -d --name=alertmanager -p 9093:9093 prom/alertmanager
    ```

    - Créer un fichier de configuration `alertmanager.yml` :

    ```yaml
    global:
      resolve_timeout: 5m

    route:
      receiver: 'team-X-mails'

    receivers:
    - name: 'team-X-mails'
      email_configs:
      - to: 'your-email@example.com'
        from: 'alertmanager@example.com'
        smarthost: 'smtp.example.com:587'
        auth_username: 'your-email@example.com'
        auth_identity: 'your-email@example.com'
        auth_password: 'your-password'
    ```

    - Monter le fichier de configuration dans le conteneur Docker :

    ```bash
    docker run -d --name=alertmanager -p 9093:9093 -v /path/to/alertmanager.yml:/etc/alertmanager/alertmanager.yml prom/alertmanager
    ```

### Monitoring de l'instance Gradio

1. **Assurez-vous que votre instance Gradio expose des métriques pour Prometheus.**

    - Pour cela, vous devrez probablement ajouter un middleware ou des points de terminaison spécifiques pour exposer les métriques, ou configurer votre application pour le faire.

2. **Vérifiez les métriques disponibles en accédant à `http://<your-gradio-instance-domain>/metrics` pour vous assurer que Prometheus peut les scraper.**

3. **Mettez à jour votre fichier `prometheus.yml` pour inclure le point de terminaison correct.**

### Dépannage

1. **Prometheus ne scrape pas les métriques :**

    - Vérifiez que l'application Gradio expose les métriques à l'endpoint `/metrics`.
    - Vérifiez que la configuration de Prometheus pointe vers le bon endpoint.

2. **Grafana ne peut pas accéder à Prometheus :**

    - Vérifiez que l'URL de Prometheus est correcte dans la configuration de la source de données Grafana.
    - Vérifiez que le conteneur Prometheus est en cours d'exécution et accessible.

3. **Les alertes ne sont pas envoyées :**

    - Vérifiez que l'Alertmanager est configuré correctement et qu'il est en cours d'exécution.
    - Vérifiez les logs de l'Alertmanager pour toute erreur de configuration ou d'envoi d'email.

En suivant ce runbook, vous pourrez configurer une solution de monitoring robuste pour votre chatbot Gradio, assurant une surveillance continue et des alertes en cas de problème.
