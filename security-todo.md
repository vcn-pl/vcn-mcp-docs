# Deployment — wskazówki bezpieczeństwa

## Wymagane

- Serwer za HTTPS reverse proxy (nginx, Caddy) — tokeny JWT w plain HTTP = kradzież sesji
- Firewall: port 8000 (MCP) dostępny tylko lokalnie lub dla proxy; port 8001 (UI) według potrzeb
- Sekrety w `/etc/vcn-mcp.env` i `/etc/vcn-ui.env` z uprawnieniami `600`

## Zalecane

- Rotacja `SECRET_KEY` co kilka miesięcy — unieważnia wszystkie aktywne tokeny MCP
- Backup SQLite (cron): `cp /srv/vnet-mcp/vnet.db /backup/vnet-$(date +%F).db`
- Skróć `auth.token_expire_days` w `config.json` jeśli serwer jest publiczny (domyślnie: 90)

## Opcjonalne

- 2FA dla konta admin w panelu UI — `/account/2fa`
