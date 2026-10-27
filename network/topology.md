# Topologie sítě

## Cíl
Přehled, jak spolu zařízení komunikují a jak se z internetu dostanu na server. Adresy jsou záměrně zobecněné.

## Diagram

```mermaid
flowchart LR
    subgraph NET["Internet"]
        PHONE["Telefon<br/>WireGuard klient"]
        AWAY["Notebook mimo domov<br/>WireGuard klient"]
    end

    ISP["Poskytovatel připojení"]
    ROUTER["Domácí router<br/>NAT, forward UDP 51820"]

    subgraph LAN["Domácí síť 192.168.X.0/24"]
        SERVER["ondycloud<br/>Ubuntu Server 26.04<br/>WireGuard server 10.X.X.1"]
        NIXOS["Notebook NixOS<br/>10.X.X.2"]
        DEBIAN["Notebook Debian<br/>10.X.X.4"]
    end

    PHONE -- "UDP 51820" --> ISP
    AWAY -- "UDP 51820" --> ISP
    ISP --> ROUTER
    ROUTER -- "forward" --> SERVER
    NIXOS -. "LAN a VPN" .-> SERVER
    DEBIAN -. "LAN a VPN" .-> SERVER
```

## Popis
- **Hvězdicová VPN:** server je střed, klienti (notebooky, telefon) se k němu připojují přes WireGuard (`10.X.X.0/24`). Detaily v [wireguard.md](wireguard.md).
- **Jediný vstup zvenku:** UDP 51820 na routeru, přesměrovaný na server. Viz [port-forwarding.md](port-forwarding.md).
- **Služby jen přes VPN:** SSH, Gitea a Nextcloud jsou dostupné z rozhraní `wg0`, z veřejné sítě ne. Viz [hardening.md](../server/hardening.md).
- **Doma:** notebooky se k serveru připojují i přes veřejnou adresu routeru (hairpin NAT), takže jdou stejnou cestou jako klient zvenku.
- **Záloha přístupu:** Tailscale zatím zůstává jako druhá cesta na server, než ho odstraním.

## Toky

| Odkud | Kam | Protokol | Poznámka |
|---|---|---|---|
| Klient mimo domov | router (veřejná adresa) | UDP 51820 | jediný otevřený port |
| Router | server | UDP 51820 | port forward |
| Klient ve VPN | server | TCP 22, 80, 443, 3000 | jen přes `wg0` |
| Internet | server (ostatní porty) | cokoli jiného | zahozeno (router, ufw) |
