# Server ondycloud

## Cíl
Domácí server pro vlastní služby: verzování kódu, cloudové úložiště, lokální AI a experimenty se sítěmi. Slouží mi jako místo, kde se učím správu Linuxu v praxi.

## Hardware a systém

| | |
|---|---|
| Systém | Ubuntu Server 26.04.1 LTS (kernel 7.0) |
| Procesor | Intel Core i3-4170, 4 vlákna, 3,7 GHz |
| Paměť | 5,2 GiB RAM |
| Disk | 220 GB, využito cca 11 % |
| Síť | kabel (LAN), přístup zvenku jen přes WireGuard |

Za běhu je zatížení nízké (zhruba 0,9 GiB RAM a nízký load), takže na stroji zbývá prostor pro další služby.

## Služby

| Služba | K čemu slouží |
|---|---|
| Gitea | vlastní Git server (repozitáře, webové rozhraní) |
| Nextcloud (snap) | cloudové úložiště pro rodinu |
| Ollama | lokální jazykové modely (llama3.2:3b) |
| i2pd + rnsd (Reticulum) | experimenty s anonymní sítí I2P a mesh sítí Reticulum |
| WireGuard (+ Tailscale jako záloha) | vzdálený přístup, viz [network/wireguard.md](../network/wireguard.md) |
| fail2ban, ufw | ochrana proti hádání hesel a firewall, viz [hardening.md](hardening.md) |
| smartmontools, thermald | kontrola stavu disku a teploty |

Webové služby (Gitea, Nextcloud) jsou dostupné jen uvnitř VPN, z internetu nejsou vidět.

## Údržba
- Aktualizace systému: `apt` a pravidelné restarty služeb
- Upgrade verze: 24.04 → 26.04 LTS (16. 10.), viz [upgrade-log.md](upgrade-log.md)

## Další kroky
- Odstranit Tailscale a jeho pravidla ve firewallu
- Doplnit zálohy a monitoring
