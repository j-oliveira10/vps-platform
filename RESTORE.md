# Runbook de restore

> Um repositório do qual nunca se restaurou não é backup.
> Última execução verificada: _(preencher: data e duração)_

## Pré-requisitos
- Credenciais B2 e senha do repositório disponíveis
- Docker em execução no destino

## 1. Verificar integridade
```bash
export RESTIC_REPOSITORY="b2:vps-platform-backup:/"
export RESTIC_PASSWORD_FILE=/root/.restic-password
restic snapshots
restic check --read-data-subset=5%
```

## 2. Restaurar para diretório temporário
```bash
restic restore latest --target /tmp/verify
ls -la /tmp/verify
```

## 3. Validar o dump do Postgres
```bash
docker run --rm -d --name pg-verify \
  -e POSTGRES_PASSWORD=verify -p 55432:5432 postgres:16-alpine
sleep 10
docker exec -i pg-verify psql -U postgres < /tmp/verify/dump.sql
docker exec pg-verify psql -U postgres -c "\dt"
docker rm -f pg-verify
```

## 4. Registrar
| Data | Tamanho | Duração | Resultado |
|---|---|---|---|
|2026-09-12|49.273 KiB|0m1.613s|OK|
