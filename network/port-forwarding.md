# Port forwarding na domácím routeru

## Cíl
Zpřístupnit server zvenku jediným způsobem: přes WireGuard VPN. Všechno ostatní zůstává zavřené.

## Pravidlo

Na routeru je jediné pravidlo pro port forwarding:

| Protokol | Vnější port | Cíl | Vnitřní port |
|---|---|---|---|
| UDP | 51820 | server (192.168.X.X) | 51820 |

SSH, web ani Gitea nejsou nikam přesměrované.

## Proč zrovna UDP 51820
- Je to port WireGuardu. Na pakety od neautorizovaných klientů vůbec neodpovídá, takže zvenku nelze poznat, jestli za portem něco běží.
- Bez privátního klíče klienta se nedá spojení navázat a není co hádat.
- Jeden otevřený port místo několika služeb (SSH, web) znamená menší útočnou plochu.

## Další nastavení routeru
- **DMZ:** vypnutá.
- **DHCP:** server má rezervovanou adresu podle MAC adresy, takže se port forward po restartu nerozbije.
- **UPnP:** zapnuté, viz poznámka níže.

## Ověření
Sken z internetu, podrobnosti v [server/hardening.md](../server/hardening.md):

- TCP 22, 80, 3000, 4242: `filtered`
- UDP 51820: `open|filtered` (u UDP a WireGuardu běžné, protože neodpovídá)
- Funkčnost forwardu: spojení do tunelu a SSH na server přes VPN z jiné sítě

Jeden neobjasněný údaj: port `443/tcp` odpověděl `closed` místo `filtered`. Server to nejspíš není (jeho firewall pakety zahazuje), takže odpovídá patrně router. Zatím jsem to nevyšetřoval.

## Poznámka k UPnP
UPnP umožňuje zařízením v síti otevírat porty na routeru samo. Zapnul jsem ho kvůli hraní v lokální síti, i když vím, že zvyšuje riziko (škodlivý program v síti si může otevřít port do internetu). Proto kontroluji skenem zvenku, že nejsou vidět žádné další porty než 51820/udp. Pokud by se objevil port, který jsem neotevřel, UPnP vypnu.

## Poznámky
- Z domácí sítě se klient připojuje přes veřejnou adresu routeru (hairpin NAT). Funguje to, ale je to jiná cesta než z internetu, proto je pro ověření lepší test z jiné sítě.
- Veřejná adresa routeru se může po výpadku poskytovatele změnit. V konfiguraci klienta je zatím napevno, řešením by byla doména s dynamickým DNS.
