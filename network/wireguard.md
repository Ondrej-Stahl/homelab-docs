# WireGuard VPN

## Cíl
Bezpečný vzdálený přístup na server z jakékoli sítě, bez závislosti na cizí službě. Nahrazuje Tailscale, kterému vypršel klíč zařízení a vyzamkl mě ze serveru.

## Topologie

| Zařízení | Adresa v tunelu | Role |
|---|---|---|
| ondycloud (server) | 10.8.0.1/24 | server, naslouchá na UDP 51820 |
| notebook NixOS | 10.8.0.2/24 | klient |
| telefon | 10.8.0.3/24 | klient |
| notebook Debian | 10.8.0.4/24 | klient |

Síť `10.8.0.0/24`, hvězdicová topologie: klienti se připojují k serveru. Server je dosažitelný z internetu přes port forward na routeru (viz [port-forwarding.md](port-forwarding.md)).

## Konfigurace

### Server (`/etc/wireguard/wg0.conf`)

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

Každý klient má vlastní klíče a na serveru jedinou adresu `/32`. Server tak přijme od klienta jen provoz z jeho vlastní adresy.

### Klient (NixOS, `configuration.nix`)

```nix
networking.wireguard.interfaces.wg0 = {
  ips = [ "10.8.0.2/24" ];
  privateKeyFile = "/home/ondy/wg-laptop.key";
  peers = [{
    publicKey = "(veřejný klíč serveru)";
    allowedIPs = [ "10.8.0.0/24" ];
    endpoint = "X.X.X.X:51820";
    persistentKeepalive = 25;
  }];
};
```

- `allowedIPs = 10.8.0.0/24`: tunelem jde jen provoz do VPN sítě (split tunnel), ostatní internet jde normálně.
- `persistentKeepalive = 25`: udržuje spojení otevřené přes NAT.
- `privateKeyFile`: klíč je v souboru s právy 600. Nevkládám ho do konfigurace, protože ta končí v `/nix/store`, čitelném pro všechny uživatele.

## Proč WireGuard místo Tailscale
- Nezávisí na cizím koordinačním serveru a účtu (vypršení klíče mě zamklo mimo server).
- Malý a jednoduchý kód, v jádře Linuxu, rychlý.
- Na neautorizované pakety neodpovídá, takže port zvenku nevypadá jako služba.
- Nevýhoda: klíče a adresy spravuji ručně a nemám NAT traversal. Funguje to, protože server má port forward.

## Ověření

```bash
sudo wg show                  # latest handshake: před pár sekundami
ip addr show wg0              # inet 10.8.0.2/24
ip route get 10.8.0.1         # dev wg0
ping -c 3 10.8.0.1
ssh ondy@10.8.0.1
```

Spojení jsem ověřil i z mobilního hotspotu, tedy mimo domácí síť.

## Problémy
Podrobně v [troubleshooting/incidents.md](../troubleshooting/incidents.md):
- Po deaktivaci tunelu zůstalo rozhraní bez adresy, handshake byl v pořádku, ale provoz šel mimo tunel. Řešení: adresa je teď v NixOS konfiguraci, vrací se sama.
- Pořadí pravidel v ufw blokovalo SSH přes `wg0`.

## Další kroky
- Endpoint je napevno veřejná IP routeru, ta se může měnit. Řešení: doména s dynamickým DNS.
- Odstranit Tailscale a jeho pravidla v ufw.
