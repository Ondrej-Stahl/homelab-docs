# Upgrade serveru: Ubuntu 24.04 → 26.04 LTS

## Cíl
Přejít na aktuální LTS verzi, aby server dostával bezpečnostní aktualizace, bez přeinstalace a bez ztráty služeb.

## Postup
- Datum: 16. 10. 2026 (log v `/var/log/dist-upgrade/20261016-2002`)
- Metoda: `do-release-upgrade` přes SSH (po VPN), nástroj sám běžel v `screen`
- Z verze 24.04 LTS na 26.04 LTS (přímo, LTS na LTS)

## Ověření po upgradu

```bash
lsb_release -a                                          # Ubuntu 26.04
systemctl --failed                                      # žádné selhané služby
sudo find /etc \( -name '*.dpkg-*' -o -name '*.ucf-*' \)   # soubory konfigurací řešené při upgradu
ls /etc/apt/sources.list.d/                             # stav repozitářů třetích stran
```

## Co upgrade změnil
- **Konfigurace:** u tří souborů vznikla vedle verze od balíčku: `ca-certificates.conf` a `i2pd.conf` (balíček je nahradil novou verzí, moje zůstala jako `.dpkg-old`) a `default/grub` (nová verze uložena jako `.ucf-dist`, moje zůstala).
- **Repozitáře třetích stran:** upgrade je vypnul (soubory `*.disabled`). Tailscale repo je aktivní, `purplei2p` zůstal vypnutý a repo pro `fastfetch` míří na starou verzi `noble`.
- **Služby:** systém naběhl na 26.04.1 LTS a služby (Gitea, Nextcloud, Ollama, fail2ban, WireGuard, i2pd, rnsd) běží.

## Poučení
- Po každém release upgradu projít `*.dpkg-*` a `*.ucf-*` a porovnat je s aktivními soubory (`diff`). Nahrazená konfigurace se jinak ztratí bez povšimnutí, i když služba dál běží.
- Zkontrolovat `sources.list.d`: třetí strany se při upgradu vypínají a samy se nevrátí.

## Co bych příště udělal jinak
- **Zálohu před upgradem.** Server zatím zálohy nemá (viz [overview.md](overview.md)), a kdyby upgrade selhal, nebylo by kam se vrátit.
