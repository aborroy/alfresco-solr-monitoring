# Alfresco SOLR Monitoring

Deployment template for Alfresco SOLR Monitoring using:

* [prometheus-exporter](https://github.com/apache/lucene-solr/tree/releases/lucene-solr/7.3.0/solr/contrib/prometheus-exporter) to build Prometheus compliant endpoint from selected [SOLR Metrics](http://localhost:8083/solr/admin/metrics)
* [Prometheus](https://prometheus.io) to get monitoring data from `prometheus-exporter` as a Target
* [Grafana](https://grafana.com) to build a monitoring dashboard with `Prometheus` as data source

[SOLR Metrics](https://lucene.apache.org/solr/guide/6_6/metrics-reporting.html) are used as source for `prometheus-exporter`.

## Services

```
solr6:8983 << solr-exporter:9854 << prometheus-server:9090 << grafana-ui:3000
```

* **SOLR** service is indexing Alfresco Repository contents
* **SOLR Exporter** service is available from SOLR 7.3.0. This program gets metrics from SOLR and make them available for Prometheus
* **Prometheus** service is an open source product used for event monitoring and alerting
* **Grafana** service is an open source product used for analytics

**Configuration** files:

* `solr-exporter/solr6-exporter-config.xml` parsing rules for JSON Response from http://127.0.0.1:8083/solr6/admin/metrics. Additional information to configure this file can be obtained in https://github.com/apache/lucene-solr/tree/releases/lucene-solr/7.3.0/solr/contrib/prometheus-exporter/conf
* `prometheus/prometheus.yml` adds new job for `solr-exporter` target in order to include results in Prometheus
* `grafana/grafana-solr6-dashboard.json` sample Grafana Dashboard showing metrics from Prometheus

# How to use this composition

## Start Docker

Start docker and check the ports are correctly bound.

```bash
$ docker-compose up --build --force-recreate
$ docker ps --format '{{.Names}}\t{{.Image}}\t{{.Ports}}'

proxy_1                  alfresco/acs-community-ngnix:1.0.0       80/tcp, 0.0.0.0:8080->8080/tcp

alfresco_1               alfresco/alfresco-content-repository-community:6.2.0-A11    8080/tcp
libreoffice_1            alfresco/alfresco-libreoffice:2.1.0      0.0.0.0:8092->8090/tcp
imagemagick_1            alfresco/alfresco-imagemagick:2.1.0      0.0.0.0:8091->8090/tcp
alfresco-pdf-renderer_1  alfresco/alfresco-pdf-renderer:2.1.0     0.0.0.0:8090->8090/tcp
transform-misc_1         alfresco/alfresco-transform-misc:2.1.0   0.0.0.0:8094->8090/tcp
tika_1                   alfresco/alfresco-tika:2.1.0             0.0.0.0:8093->8090/tcp

postgres_1               postgres:11.4                            0.0.0.0:5432->5432/tcp
activemq_1               alfresco/alfresco-activemq:5.15.8        0.0.0.0:5672->5672/tcp, ...

share_1                  alfresco/alfresco-share:6.2.0            8000/tcp, 8080/tcp

solr6_1                  alfresco/alfresco-search-services:1.4.0  0.0.0.0:8083->8983/tcp
solr-exporter_1          solr:7.3.0                               8983/tcp, 0.0.0.0:9854->9854/tcp
prometheus-server_1      prom/prometheus                          0.0.0.0:9090->9090/tcp
grafana-ui_1             grafana/grafana                          0.0.0.0:3000->3000/tcp
```

## Access

Use the following username/password combination to login in Alfresco services.

 - User: admin
 - Password: admin

Alfresco and related web applications can be accessed from the below URIs when the servers have started.

```
http://localhost:8080/share         - Alfresco Share WebApp
http://localhost:8080/alfresco      - Alfresco Repository (REST)
http://localhost:8083/solr          - Alfresco Search Services (Basic Auth, admin/admin by default)
```

Solr Exporter and Prometheus services are provided without authentication:

```
http://localhost:9854     - Solr Exporter metrics
http://localhost:9090     - Prometheus UI
```

Use the following username/password combination to login in Grafana UI.

 - User: admin
 - Password: secret

```
http://localhost:3000
```

# Configuring Grafana Dashboard

Once everything is running, access to Grafana UI.

http://localhost:3000


**Adding Prometheus Data Source**

Use `Configuration > Data Sources > Add data source` option and choose `Prometheus` data source.

Configure Prometheus Data Source with following setting:

```
URL: http://prometheus-server:9090
```

Click `Save & Test` button.


**Importing SOLR Dashboard**

When Prometheus Data Source is running, create your first Dashboard using `+ > Create > Import` option.

Click the `Upload.json file` button and add the sample dashboard from project folder:

```
grafana/grafana-solr6-dashboard.json
```

In the Prometheus field, set `Prometheus` as value (this is the Data Source configured in the previous step).

Click the `Import` button and Grafana will show your **Solr Dashboard**
