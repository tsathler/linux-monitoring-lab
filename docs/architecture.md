# Arquitetura

## Fluxo principal

```text
Ubuntu host
    │
    └── node-exporter:9100
             │ rede linux-monitoring-lab_default
             ▼
       prometheus:9090
          │      │
          │      └── regras de alerta
          ▼
       grafana:3000
```

## Componentes

### Node Exporter

Expõe métricas do host Ubuntu. O bind mount `/host:ro,rslave` e `--path.rootfs=/host` permitem observar o filesystem real do servidor.

### Prometheus

Coleta `node-exporter:9100` a cada 15 segundos. `prometheus/prometheus.yml` é a configuração principal e `prometheus/alerts.yml` contém as regras de alerta.

### Grafana

Consulta o Prometheus pela rede interna do Compose. O banco SQLite e demais dados ficam no volume `grafana-data`.

### Persistência

```text
prometheus-data → /prometheus
grafana-data    → /var/lib/grafana
```

Os volumes sobrevivem à remoção dos containers, mas não são uma estratégia completa de backup.

## Exposição de portas

As portas atuais são publicadas em todas as interfaces do servidor:

- `3000`: Grafana
- `9090`: Prometheus
- `9100`: Node Exporter

Em uma implantação fora de laboratório, recomenda-se restringir o acesso com firewall e evitar expor Prometheus e Node Exporter à internet.

## Fluxo de alteração

```text
Windows / VS Code
        ↓ git push
GitHub
        ↓ git pull
Ubuntu Server
        ↓ docker compose up -d
Containers e volumes persistentes
```
