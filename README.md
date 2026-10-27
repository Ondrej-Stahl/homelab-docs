
# homelab-docs

Dokumentace mého domácího serveru a sítě. Server běží na starém stolním počítači (Intel i3, 5 GB RAM, Ubuntu Server) a slouží jako moje cvičiště pro správu Linuxu, sítě a zabezpečení. Všechno tady jsem nastavoval, rozbíjel a opravoval sám.

Adresy v dokumentaci jsou záměrně zobecněné (`10.X.X.x`, `X.X.X.X`) a žádné klíče ani hesla v repozitáři nejsou.

## Co na serveru běží
Gitea (vlastní Git), Nextcloud (cloudové úložiště), Ollama (lokální jazykové modely), I2P a Reticulum (experimenty se sítěmi), WireGuard (vzdálený přístup) a fail2ban s ufw. Podrobně v [server/overview.md](server/overview.md).

## Obsah

| Dokument | O čem je |
|---|---|
| [server/overview.md](server/overview.md) | hardware, systém, služby |
| [server/hardening.md](server/hardening.md) | SSH, firewall, fail2ban, ověření skenem |
| [server/upgrade-log.md](server/upgrade-log.md) | upgrade Ubuntu 24.04 → 26.04 LTS |
| [network/topology.md](network/topology.md) | diagram sítě |
| [network/wireguard.md](network/wireguard.md) | VPN a proč WireGuard |
| [network/port-forwarding.md](network/port-forwarding.md) | jediné pravidlo na routeru a co je zavřené |
| [troubleshooting/incidents.md](troubleshooting/incidents.md) | skutečné problémy a jejich diagnostika |

## Principy
- **Minimální viditelná plocha:** zvenku je dostupný jediný port, UDP 51820 (WireGuard). SSH, web ani Gitea nejsou na internetu.
- **Ověřuji, ne předpokládám:** zabezpečení testuji skenem z jiné sítě (nmap) a kontrolou po změnách.
- **Dokumentuji i chyby:** u každého incidentu je příznak, příčina, řešení a poučení.

## Co jsem se naučil
- diagnostika po vrstvách (spojení, adresa, routování, firewall)
- pořadí pravidel ve firewallu a pořadí konfiguračních souborů (`sshd`)
- rozdíl mezi spojeným tunelem a správně routovaným tunelem
- kontrola konfigurací a repozitářů po release upgradu

## Související
Moje cesta s Linuxem od první instalace po NixOS: [linux-journey](https://github.com/Ondrej-Stahl/linux-journey).
