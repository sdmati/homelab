# Homelab — dokumentacja infrastruktury

> **Stan dokumentacji:** 27 października 2026 r.  

## 1. Cel projektu

Homelab służy do praktycznej nauki administracji Linuksem, wirtualizacji, sieci komputerowych, hostowania usług, zdalnego dostępu oraz diagnostyki. Infrastruktura jest rozwijana iteracyjnie — od konfiguracji sieci po uruchamianie i utrzymanie usług.

## 2. Sprzęt i wirtualizacja

| Element | Konfiguracja / rola |
| --- | --- |
| Serwer fizyczny | HP EliteDesk 800 G2 Mini |
| Procesor | Intel Core i7-6700 |
| Pamięć RAM | 16 GB |
| Dysk | SSD 128 GB |
| Hypervisor | Proxmox VE 9.2 |
| Adres Proxmoxa (LAN) | `192.168.0.2` |
| Panel zarządzania | `https://192.168.0.2:8006` (HTTPS, port 8006) |

> **Uwaga bezpieczeństwa:** adresy `192.168.x.x` są prywatne. Przed publikacją repozytorium sprawdź jednak, czy nie zawiera publicznych adresów IP, tokenów, kluczy prywatnych, haseł ani danych dostępowych.

## 3. Topologia sieci

```text
Internet
   |
Router operatora
LAN: 192.168.1.1
   |
AmpliFi HD (Wi-Fi / bridge)
sieć: 192.168.174.0/24
   |
   | Wi-Fi (klient)
   v
TP-Link Archer C6 — OpenWrt
LAN: 192.168.0.1/24
   |
   +-- HP EliteDesk / Proxmox VE   192.168.0.2
   |      +-- VM Debian / Nginx + Tailscale   192.168.0.4
   |      +-- VM Debian / Minecraft Paper    192.168.0.5
   |
   +-- Raspberry Pi / Ubuntu Server         192.168.0.3
```

**Podział na podsieci:**

- `192.168.1.0/24` — sieć po stronie routera operatora (adres bramy `192.168.1.1`).
- `192.168.174.0/24` — sieć Wi-Fi AmpliFi, do której OpenWrt łączy się jako klient.
- `192.168.0.0/24` — wydzielona sieć homelaba za OpenWrt; brama `192.168.0.1`.

Aktualnie router TP-Link z OpenWRT odbiera przez Wi-Fi sieć od Amplifi i kablem ethernet rozglasza sieć dla HP i Raspberry. Docelowo Amplifi zostanie wyłączone z użytku, do Routera ISP będzie podpięty TP-Link, a on zostanie połączony ze switchem przy homelabie


## 4. Usługi

| Host | Adres LAN | System | Rola | Status wg ostatnich informacji |
| --- | --- | --- | --- | --- |
| Proxmox | `192.168.0.2` | Proxmox VE 9.2 | Hypervisor, zarządzanie VM | Działa |
| Raspberry Pi | `192.168.0.3` | Ubuntu Server | Pi-hole | Działa |
| VM — WWW | `192.168.0.4` | Debian | Nginx, Tailscale | Działa |
| VM — Minecraft | `192.168.0.5` | Debian 13 | Paper Minecraft Java Edition | Działa |


## 5. Serwer Minecraft

### Parametry VM

| Parametr | Wartość |
| --- | --- |
| System | Debian 13 |
| vCPU | 2 |
| RAM | 4 GB |
| Dysk | ok. 32 GB |
| Adres | `192.168.0.5` |
| Silnik | Paper, Minecraft Java Edition 26.1 (planowane przejscie na Spigot)|
| Java | Java 25 (zainstalowana także Java 21) |
| Katalog | `/opt/minecraft` |
| Plik serwera | `/opt/minecraft/server.jar` |
| Użytkownik usługi | `minecraft` |
| Port gry | TCP `25565` (domyślny; potwierdzić w `server.properties`) |

### Uruchamianie i konsola

Serwer jest uruchamiany w sesji `tmux` zarządzanej przez usługę `minecraft.service` w `systemd`. W ostatniej potwierdzonej konfiguracji zastosowano `Type=oneshot` i `RemainAfterExit=yes`.

```bash
# Start / stop usługi (jako root)
systemctl start minecraft
systemctl stop minecraft

# Lista sesji tmux
runuser -u minecraft -- tmux -L minecraft list-sessions

# Interaktywna konsola serwera
runuser -u minecraft -- tmux -L minecraft attach -t server

#na koncie root alias do powyższej komendy
server-console
```

Odłączanie od konsoli bez wyłączania serwera: **Ctrl+B**, następnie **D**.

> **Ograniczenie obecnej integracji:** `systemd` może pokazywać `active (exited)` mimo zakończenia procesu Paper. Ta konfiguracja nie zapewnia wiarygodnego nadzoru procesu ani automatycznego restartu po awarii. Dodatkowo należy przetestować łagodne zatrzymywanie usługi i zapis świata. Nie traktować samego statusu `systemctl` jako potwierdzenia działania serwera.

### Dostęp i bezpieczeństwo

- RCON został wyłączony (`enable-rcon=false`); wcześniej testowany `mcrcon` nie jest wymagany.
- Do ograniczania graczy planowano / omawiano whitelistę; **aktualny stan `white-list`, `enforce-whitelist` i `online-mode` wymaga weryfikacji**.
- Omawiano dostęp znajomych przez Tailscale, z ograniczeniem do TCP `25565`. **Nie potwierdzono ukończenia konfiguracji udostępniania i polityki dostępu**.
- Kopie zapasowe świata: **brak potwierdzonej automatyzacji**.


## 6. Administracja i używane technologie

- **Wirtualizacja:** Proxmox VE, maszyny wirtualne.
- **Systemy:** Debian, Ubuntu Server, Linux.
- **Sieci:** OpenWrt, DHCP, adresacja IPv4, routing, NAT, sieć za dodatkowym routerem.
- **Usługi:** Nginx, Minecraft Paper.
- **Administracja:** SSH.
- **Zdalny dostęp:** Tailscale.


---
