# Hardening serveru ondycloud

## Cíl
Zmenšit útočnou plochu domácího serveru (Ubuntu 26.04, Gitea a další služby) po hrozbě DoS útoku. Žádná hesla, žádný root a žádná služba veřejně na internetu. Zvenku je dostupný jen UDP port WireGuardu.

## Přehled

| Vrstva | Opatření |
|---|---|
| SSH | jen klíče, root vypnutý, whitelist uživatelů |
| Firewall | ufw, výchozí deny, služby jen přes VPN rozhraní |
| VPN | WireGuard (`wg0`), 3 klienti, Tailscale jako záloha |
| Obrana proti útokům | fail2ban s eskalací banu, SYN flood ochrana |

## Konfigurace

### SSH
`/etc/ssh/sshd_config.d/01-hardening.conf`:

```
PasswordAuthentication no
PermitRootLogin no
PubkeyAuthentication yes
MaxAuthTries 3
LoginGraceTime 20
AllowUsers ondy gitea
X11Forwarding no
```

Každé zařízení má vlastní klíč (ed25519).

### Firewall (ufw)
- Výchozí politika: příchozí **deny**, odchozí allow
- SSH (22) a web (80/443) povoleny jen na VPN rozhraní (`wg0`, záloha `tailscale0`)
- Z veřejné sítě zakázáno 22, 80 a 443
- Jediný veřejně povolený port: `51820/udp` (WireGuard), na routeru přesměrovaný na server (viz `network/port-forwarding.md`)

### VPN (WireGuard)
- Rozhraní `wg0`, síť `10.8.0.0/24`, server `10.8.0.1`
- Klienti: notebook s NixOS, telefon, notebook s Debianem
- Každý klient má vlastní pár klíčů a `AllowedIPs` omezené na jedinou adresu `/32`

```
[Interface]
Address = 10.8.0.1/24
ListenPort = 51820
PrivateKey = (skrytý)

[Peer]
# klient
PublicKey = (veřejný klíč klienta)
AllowedIPs = 10.8.0.2/32
```

### fail2ban
`/etc/fail2ban/jail.local`:

```
[DEFAULT]
bantime = 1h
findtime = 10m
maxretry = 3
bantime.increment = true
bantime.factor = 2
bantime.maxtime = 1w

[sshd]
enabled = true
maxretry = 3
```

Při opakování se ban zdvojnásobuje, maximálně na týden.

### Kernel (sysctl)
`/etc/sysctl.d/99-hardening.conf`:

```
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_max_syn_backlog = 2048
```

## Ověření

Sken zvenku (mobilní hotspot, VPN vypnuté):

```bash
nmap -Pn -p 22,80,443,3000,4242 X.X.X.X
sudo nmap -Pn -sU -p 51820 X.X.X.X
nmap -6 -Pn -p 22,80,443,3000 Y::Y
```

| Cíl | Výsledek |
|---|---|
| IPv4 (router), TCP 22/80/3000/4242 | `filtered` |
| IPv4 (router), UDP 51820 | `open\|filtered` (WireGuard neodpovídá neautorizovaným) |
| IPv6 (přímo server, bez NATu), TCP 22/80/443/3000 | `filtered` (ufw) |

Zvenku není vidět žádná služba. Přístup na server je možný jen s klíčem WireGuardu, což jsem ověřil spojením z mobilního hotspotu.

Další kontroly přímo na serveru:

```bash
sudo sshd -T | grep -Ei 'permitrootlogin|passwordauthentication'
sudo ufw status verbose
sudo wg show
sudo fail2ban-client status sshd
```

## Co to chrání a co ne
- **Chrání** proti průniku: hádání hesel, skenování a útokům na služby, které zvenku nejsou vidět.
- **Nechrání** proti DDoS. Ten zahltí přípojku dřív, než pakety dorazí k firewallu. Skutečná obrana je malá viditelná plocha: WireGuard na pakety od neznámých klientů vůbec neodpovídá.

## Další kroky
- Odstranit Tailscale a s ním obecné pravidlo pro `tailscale0` v ufw
- Pokročilejší firewall (zatím stačí ufw)

## Problémy, na které jsem narazil
Podrobně v [troubleshooting/incidents.md](../troubleshooting/incidents.md):
- konfigurace sshd se nepoužila kvůli pořadí souborů (první shoda vyhrává)
- pravidla ufw nefungovala kvůli pořadí
- vypršelý klíč Tailscale zamkl přístup, proto přechod na WireGuard
- SSH přes WireGuard nešlo kvůli chybějící adrese rozhraní
