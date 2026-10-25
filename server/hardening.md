# server hardening

## Cíl
Cílem bylo omezit útočnou plochu domácího serveru po výhružkách Dos útoku.

## Co jsem udělal
-nainstaloval jsem fail2ban a nakonfiguroval jsem ho
-nainstaloval jsem ufw a nakonfiguroval jsem na přístup jen pomocí tailscale, ale teď chci používat wireguard
-mám otevřený jen port 51820/UDP
-udělal jsem nmap local.ip.meho.se.rve.ru a také tailscale.ip.meho.se.rve.ru abych zjistil otevřené porty

## Konfigurace
