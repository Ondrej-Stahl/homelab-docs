# Incidenty a troubleshooting

## SSH přes WireGuard nefungovalo, i když tunel měl handshake

**Příznak:** `ssh` na `10.8.0.1` visel na `Connecting...`, ping nešel, přitom `wg show` na obou stranách ukazoval čerstvý handshake.

**Diagnostika (po vrstvách):**
1. `sudo wg show` na serveru i klientovi: tunel spojený, handshake v pořádku
2. `ip route get 10.8.0.1`: provoz šel přes WiFi (`wlo1`) k routeru, ne přes `wg0`
3. `ip addr show wg0`: rozhraní bylo nahoře, ale **bez IP adresy**

**Příčina:** Po deaktivaci a znovuzapnutí tunelu zůstalo rozhraní `wg0` bez adresy. Kernel pak nevěděl, že `10.8.0.0/24` leží za ním, a posílal provoz výchozí branou.

**Řešení:**
```bash
sudo ip addr add 10.8.0.2/24 dev wg0
```
Routa do sítě se přidá automaticky spolu s adresou. Ověřeno přes `ip route get`, ping i SSH.

**Poučení:**
- Handshake říká jen to, že je tunel spojený, ne že je správně routovaný. Při potížích kontroluj `ip addr show wg0` a `ip route get`.
- Než vypnu jednu cestu na server, musí fungovat druhá (jako záloha jsem měl Tailscale).
- Diagnostika po vrstvách (spojení, adresa, routa, firewall) šetří čas oproti hádání.

## SSH drop-in konfigurace se nepoužila

**Příznak:** Moje `99-hardening.conf` neměla na sshd efekt.

**Příčina:** `sshd` bere první shodu a `50-cloud-init.conf` ve stejné složce se načítal dřív.

**Řešení:** Přejmenování na `01-hardening.conf`, ověřeno přes `sudo sshd -T`.

## Pravidla ufw nefungovala kvůli pořadí

**Příznak:** SSH přes `wg0` bylo blokované, i když existovalo povolující pravidlo.

**Příčina:** Povolení na `wg0` stálo pod obecnými `DENY` pravidly. ufw vyhodnocuje shora a první shoda vyhrává.

**Řešení:** Přeřazení pravidel, kontrola přes `sudo ufw status numbered`.

## Vypršelý klíč Tailscale zamkl přístup na server

**Příznak:** Nešlo se přihlásit na vlastní server.

**Příčina:** SSH bylo povolené jen přes Tailscale a jeho klíč zařízení vypršel.

**Řešení:** Přechod na vlastní WireGuard, který nezávisí na cizí službě.
