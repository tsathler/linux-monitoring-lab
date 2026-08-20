# Alertas e e-mail

## Estado atual

O Prometheus já avalia as regras em `prometheus/alerts.yml`. Sem Alertmanager, os estados ficam disponíveis em `/alerts` e pela API, mas nenhuma notificação externa é enviada.

## Alertmanager opcional

`alertmanager/alertmanager.yml.example` é um modelo com endereços fictícios. `alertas@example.com` não entrega mensagens; serve apenas para documentar a estrutura sem expor uma conta real.

Para ativar e-mail no laboratório:

1. Copie o arquivo de exemplo para `alertmanager/alertmanager.yml` no Ubuntu.
2. Substitua o SMTP, remetente, usuário, senha de aplicativo e destinatário por valores reais.
3. Nunca faça commit do arquivo real; ele é ignorado pelo Git.
4. Ative o perfil `alerting` do Compose.
5. Valide o Alertmanager e provoque um alerta controlado.

O Alertmanager recebe alertas do Prometheus, agrupa eventos, aplica silêncios e encaminha notificações. O Prometheus detecta a condição; o Alertmanager entrega a notificação.

## Segurança

- Use senha de aplicativo quando o provedor exigir autenticação SMTP.
- Mantenha credenciais fora do Git.
- Prefira TLS, normalmente porta 587 com STARTTLS.
- Não use `example.com` como SMTP real.
