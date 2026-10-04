# 03 - Rete Wi-Fi con WPA2

**Obiettivo:** configurare un access point Wi-Fi protetto, collegare più host wireless e verificare che possano comunicare e condividere risorse.

**Ambiente:** router Wi-Fi TP-Link TL-WR940N, due PC Windows, rete del laboratorio (aprile 2026).

## Topologia

```
   Rete del laboratorio
          |
     WAN (DHCP)
   +---------------+
   |  TP-Link AP   |  192.168.0.1
   +---------------+
     ))) SSID: LAB_CYBER_UFFICI - WPA2-PSK (AES)
       /        \
   PC1           PC2
   192.168.0.101 192.168.0.102
```

## Configurazione

**Fase 0 - Collegamento iniziale**
1. Porta WAN del router collegata allo switch del laboratorio, per l'accesso alla rete esterna.
2. PC collegato via cavo a una porta LAN per la prima configurazione.
3. Accesso all'interfaccia web del router su `192.168.0.1` con le credenziali di fabbrica.

**Fase 1 - Rete wireless**
1. SSID impostato su `LAB_CYBER_UFFICI`, scelto tra quelli autorizzati.
2. Sicurezza **WPA2-PSK** con cifratura **AES**.
3. Server DHCP attivo sulla LAN, connessione WAN in IP dinamico.
4. Verifica nella pagina *Status* che il router abbia ricevuto un indirizzo dalla rete del laboratorio.

**Fase 2 - Collegamento dei client**
1. Cavi LAN scollegati.
2. PC connessi all'SSID tramite la scheda wireless.
3. Verifica dell'indirizzo ottenuto con `ipconfig`.

## Verifiche

| Test | Risultato |
| --- | --- |
| `ping 192.168.0.1` dal client | il client comunica con il gateway |
| *Wireless Statistics* sul router | i client connessi compaiono nella lista |
| Secondo PC sulla stessa rete | ottiene `192.168.0.102` via DHCP |
| Cartella condivisa su PC1, aperta da PC2 con `\\192.168.0.101` | accesso riuscito |

## Considerazioni di sicurezza

- **Credenziali di fabbrica:** le credenziali di amministrazione del router erano quelle predefinite. È la prima cosa da cambiare su qualsiasi apparato.
- **WPA2-PSK** con AES è adeguato per una rete piccola, ma la chiave è condivisa da tutti. In un ufficio è meglio **WPA2/WPA3-Enterprise**, con autenticazione individuale tramite server RADIUS.
- **Condivisione per "Everyone":** comoda per il test, ma dà accesso a chiunque sia sulla rete. In un ambiente reale i permessi vanno limitati agli utenti che ne hanno bisogno.
- **Rete ospiti:** in un contesto aziendale i dispositivi esterni andrebbero separati su un SSID dedicato o una VLAN.

## Cosa ho imparato

- configurare un access point partendo dal collegamento fisico fino alla verifica dei client
- controllare il funzionamento da entrambi i lati: client (`ipconfig`, `ping`) e access point (lista dei dispositivi)
- distinguere una configurazione che funziona da una configurazione sicura
