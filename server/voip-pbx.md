# VoIP ústředna (Asterisk na NixOS)

## Cíl
Malá domácí telefonní ústředna na serveru ondycloud. Hovory mezi dvěma telefony s Androidem v LAN, bez externího poskytovatele a bez VPN. Později přibude stolní telefon Yealink jako třetí linka.

## Přehled

| | |
|---|---|
| Software | Asterisk, modul `pjsip` |
| Systém | NixOS, služba `services.asterisk` |
| Linky | `100`, `101` (Android, aplikace Linphone) |
| Testovací linka | `600` (echo) |
| Dosah | jen LAN, firewall otevřený jen na LAN rozhraní |
| Porty | UDP 5060 (SIP), UDP 10000–10100 (RTP, zvuk) |

## Konfigurace

Do `configuration.nix`. Hesla zde nejsou, `ZMEN_HESLO_X` nahraď vlastním. Rozhraní `enp2s0` je LAN rozhraní serveru.

```nix
services.asterisk = {
  enable = true;
  confFiles = {
    "rtp.conf" = ''
      [general]
      rtpstart=10000
      rtpend=10100
    '';

    "pjsip.conf" = ''
      [transport-udp]
      type=transport
      protocol=udp
      bind=0.0.0.0:5060

      [100]
      type=endpoint
      context=interni
      disallow=all
      allow=alaw,ulaw,opus
      auth=100
      aors=100
      direct_media=no
      rtp_symmetric=yes
      force_rport=yes
      rewrite_contact=yes
      [100]
      type=auth
      auth_type=userpass
      username=100
      password=ZMEN_HESLO_1
      [100]
      type=aor
      max_contacts=1
      remove_existing=yes

      [101]
      type=endpoint
      context=interni
      disallow=all
      allow=alaw,ulaw,opus
      auth=101
      aors=101
      direct_media=no
      rtp_symmetric=yes
      force_rport=yes
      rewrite_contact=yes
      [101]
      type=auth
      auth_type=userpass
      username=101
      password=ZMEN_HESLO_2
      [101]
      type=aor
      max_contacts=1
      remove_existing=yes
    '';

    "extensions.conf" = ''
      [interni]
      exten => 100,1,Dial(PJSIP/100,30)
      exten => 101,1,Dial(PJSIP/101,30)
      exten => 600,1,Answer()
       same => n,Echo()
       same => n,Hangup()
    '';
  };
};

networking.firewall.interfaces."enp2s0" = {
  allowedUDPPorts = [ 5060 ];
  allowedUDPPortRanges = [ { from = 10000; to = 10100; } ];
};
```

Pak `sudo nixos-rebuild switch` a `systemctl status asterisk`.

Poznámka: hesla v `configuration.nix` skončí v `/nix/store`, kde je přečte každý uživatel serveru. Pro LAN-only provoz je to přijatelné. Při vystavení do internetu je potřeba použít soubor se secrety mimo store.

## Nastavení telefonů

V aplikaci Linphone volba "Use SIP account":

| Pole | Hodnota |
|---|---|
| Username | `100` (druhý telefon `101`) |
| Password | heslo z configu |
| Domain | IP serveru |
| Transport | UDP |

V nastavení účtu vypnout Push notifications a vypnout optimalizaci baterie pro Linphone, jinak Android aplikaci uspí a hovor nepřijde.

## Ověření

```
sudo asterisk -rx "pjsip show contacts"
```
Oba kontakty (100, 101) jsou vidět. Stav `NonQual` je v pořádku. Znamená jen, že Asterisk neposílá kontrolní pingy. Pro stav `Avail` stačí přidat do každého bloku `type=aor` řádek `qualify_frequency=30`.

Testovací hovory:
1. Z `100` na `600`: uslyším sám sebe (echo), tedy funguje zvuk.
2. Z `100` na `101` a zpět: funguje oběma směry.

Živý výpis při ladění: `sudo asterisk -rvvv`.

## Problémy a diagnostika

| Příznak | Pravděpodobná příčina |
|---|---|
| telefon se neregistruje | špatná IP, heslo nebo firewall (`sudo nft list ruleset \| grep 5060`) |
| hovor jde, ale je ticho | zavřené RTP porty nebo izolace klientů na Wi-Fi |
| hovor funguje jen jedním směrem | aplikace uspaná na pozadí |

## Plán: stolní telefon Yealink (linka 102)

1. Do `pjsip.conf` přidat trojici endpoint, auth, aor pro `102` se svým heslem.
2. Do `extensions.conf` přidat `exten => 102,1,Dial(PJSIP/102,30)`.
3. `sudo nixos-rebuild switch`.
4. V telefonu (webové rozhraní, Account): Register Name a Username `102`, heslo, SIP Server = IP serveru, port 5060, transport UDP.
5. Yealinku nastavit na routeru pevnou IP. Kodeky `alaw` a `ulaw` jsou už povolené
