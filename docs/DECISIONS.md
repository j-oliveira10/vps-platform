# Decisões de arquitetura

## DNS-01 em vez de HTTP-01
Os serviços administrativos escutam apenas na interface WireGuard e não têm
porta pública. O desafio HTTP-01 exige alcance na porta 80 a partir da
internet, o que forçaria expor Grafana e Prometheus. DNS-01 emite certificado
válido sem nenhuma exposição, ao custo de um token de API do provedor de DNS.

## Token da Cloudflare com escopo de zona única
A conta que hospeda a zona pessoal é separada da conta corporativa. O token do
Traefik tem permissão `Zone:DNS:Edit` + `Zone:Zone:Read` restrita a uma zona,
com filtro de IP de origem. Comprometimento da VPS não alcança nenhum outro
domínio.

## default-address-pools do Docker em 10.201.0.0/16
O padrão do Docker aloca a partir de 172.17.0.0/16 e avança sobre
172.18–172.31. Em redes que usam 172.16.0.0/12 internamente isso causa
colisão de rota silenciosa: o container passa a considerar a LAN corporativa
como rede local e o tráfego nunca sai para o gateway. Definido antes do
primeiro container, porque o daemon não redistribui pools retroativamente.

## Prometheus em vez de Zabbix
Zabbix é superior para descoberta de rede e SNMP. Prometheus foi escolhido
aqui porque as regras de alerta vivem em arquivo versionado no Git, o modelo
de labels casa com cargas em container efêmeras, e o scrape pull não exige
agente configurado por host.

## Restic em vez de snapshot do provedor
Snapshot de disco é do provedor, no mesmo provedor, e não é testável de forma
granular. Restic para B2 é offsite, deduplicado, cifrado no cliente e
restaurável para diretório arbitrário — o que torna o teste de restore parte
da rotina em vez de um evento.

## TODO
- [ ] Substituir montagem direta de `/var/run/docker.sock` no Traefik por
      socket-proxy com escopo mínimo (`CONTAINERS=1`, restante 0).
- [ ] Rotacionar o token da Cloudflare criado sem filtro de IP (fase local)
      por um token novo com filtro, e revogar o antigo.
