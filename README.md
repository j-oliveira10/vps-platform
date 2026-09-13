# vps-platform

Infraestrutura reproduzível de nó único, definida em Ansible. Um servidor é
reconstruído do zero a partir de uma VPS limpa com um único comando.

## Arquitetura

```
                    Internet
                       |
            +----------+----------+
            |  :80/:443    :51820 |
            v                     v
        [ Traefik ]          [ WireGuard ]
            |                     |
            |  proxy net          | wg0 10.100.0.0/24
            |                     |
        [  api  ]            serviços privados:
            |                grafana / prometheus / traefik-dashboard
            | backend net    (sem porta pública, TLS via DNS-01)
        [ Postgres ]
            |
        [ Restic ] --> Backblaze B2 (offsite)
```

| Nome | Exposição |
|---|---|
| `app.jairooferreira.online` | Pública |
| `vpn.jairooferreira.online` | UDP 51820 |
| `*.lab.jairooferreira.online` | Resolve para 10.100.0.1 — alcançável só via VPN |

## Decisões

Ver [docs/DECISIONS.md](docs/DECISIONS.md).

## Uso

```bash
ansible-galaxy install -r requirements.yml
ansible-playbook site.yml --check          # dry run
ansible-playbook site.yml                  # aplicar
ansible-playbook site.yml --tags traefik   # parcial
```

Idempotência é requisito: a segunda execução deve reportar `changed=0`.

## Restore

Procedimento testado e cronometrado em [RESTORE.md](RESTORE.md).

## Status

- [x] Fim de semana 1 — baseline manual (SSH, ufw, Docker, Traefik, app)
- [x] Fim de semana 2 — WireGuard, Restic, restore testado
- [x] Fim de semana 3 — Prometheus, Loki, Grafana, alertas
- [ ] Fim de semana 4 — tudo em Ansible, rebuild do zero, CI/CD
