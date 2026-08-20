# Linux Monitoring Lab

Laboratório prático de monitoramento de um Ubuntu Server com Docker Compose, Node Exporter, Prometheus e Grafana.

## Objetivo

Monitorar uma VM Ubuntu Server e visualizar no Grafana disponibilidade, CPU, memória, filesystem, carga do sistema e tráfego de rede.

## Arquitetura

```text
Ubuntu Server
├── Node Exporter :9100 — métricas do host
├── Prometheus :9090 — coleta a cada 15s e avalia alertas
└── Grafana :3000 — dashboard e visualização
```

O Node Exporter usa `/host` em modo somente leitura e `--path.rootfs=/host` para observar o filesystem real do Ubuntu.

## Tecnologias

- Ubuntu Server 24.04 LTS
- Docker e Docker Compose
- Node Exporter, Prometheus e Grafana
- Git e GitHub

## Estrutura

```text
.
├── compose.yaml
├── .env.example
├── prometheus/
│   ├── prometheus.yml
│   └── alerts.yml
├── grafana/dashboards/linux-server-monitoring.json
├── alertmanager/alertmanager.yml.example
└── docs/
    ├── architecture.md
    └── alerting.md
```

## Execução

```bash
git clone https://github.com/tsathler/linux-monitoring-lab.git
cd linux-monitoring-lab
docker compose up -d
docker compose ps
```

Acessos:

- Grafana: `http://IP_DO_SERVIDOR:3000`
- Prometheus: `http://IP_DO_SERVIDOR:9090`
- Node Exporter: `http://IP_DO_SERVIDOR:9100/metrics`

## Persistência

O projeto usa named volumes:

- `linux-monitoring-lab_prometheus-data` em `/prometheus`
- `linux-monitoring-lab_grafana-data` em `/var/lib/grafana`

`docker compose down` preserva esses volumes. Não use `docker compose down -v` sem intenção explícita de apagar os dados. Volume não substitui backup.

## Prometheus e alertas

O Prometheus coleta o Node Exporter a cada 15 segundos e carrega quatro regras:

- `NodeExporterDown`
- `HighCPUUsage`
- `HighMemoryUsage`
- `RootFilesystemAlmostFull`

As regras podem ser consultadas em `http://IP_DO_SERVIDOR:9090/alerts`. No estado atual elas não enviam e-mail; o Alertmanager opcional está documentado em [docs/alerting.md](docs/alerting.md).

## Dashboard

O dashboard `Linux Server Monitoring` contém Server Status, Server Uptime, CPU Usage, RAM Usage, Disk Usage `/`, System Load e Network Traffic.

Para acompanhar testes ao vivo, configure o refresh automático para 10 segundos. O Prometheus coleta a cada 15 segundos; o Grafana só atualiza ao carregar ou quando possui refresh automático.

## PromQL utilizada

```promql
up{job="node-exporter"}
time() - node_boot_time_seconds
100 - (avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
(1 - node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes) * 100
(1 - node_filesystem_avail_bytes{mountpoint="/"} / node_filesystem_size_bytes{mountpoint="/"}) * 100
rate(node_network_receive_bytes_total{device="eth0"}[5m])
rate(node_network_transmit_bytes_total{device="eth0"}[5m])
```

O nome da interface deve ser confirmado nas métricas do Node Exporter. Neste ambiente, a métrica validada usa `eth0`, embora a interface física seja `enp0s3`.

## Testes realizados

- Carga controlada de CPU com dois processos `yes > /dev/null`.
- Alocação temporária de aproximadamente 512 MiB de memória com Python.
- Escrita e leitura temporárias de 512 MiB em `/tmp`.
- Parada controlada do Node Exporter para validar `NodeExporterDown`.
- Reinicialização dos containers para confirmar a persistência.

## Segurança e melhorias futuras

- Fixar versões das imagens em vez de usar `:latest`.
- Restringir portas com firewall e evitar expor Prometheus e Node Exporter à internet.
- Alterar credenciais padrão do Grafana.
- Configurar Alertmanager com SMTP real e credenciais fora do Git.
- Criar backups dos named volumes.
- Revisar permissões, privilégios e screenshots do projeto.

## Estado

Monitoramento, dashboard, persistência e regras básicas de alerta estão funcionais. A configuração de e-mail usa um exemplo fictício até que um SMTP real seja escolhido.
