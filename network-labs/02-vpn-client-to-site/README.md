# 02 - VPN client-to-site su firewall ZyWALL

**Obiettivo:** permettere a un PC esterno di accedere in modo sicuro alla rete interna di un firewall, tramite un tunnel VPN di accesso remoto.

**Ambiente:** firewall fisico ZyWALL, due PC Windows, rete del laboratorio.

**Svolto in coppia** durante il laboratorio di Sistemi e reti (marzo 2026).

## Topologia

```
   PC2 - client VPN                       PC1 - client LAN
   (rete del laboratorio)                 192.168.1.33
          |                                     |
          |  rete laboratorio 10.0.6.0          |
          +-------- WAN [ ZyWALL ] LAN ---------+
                   10.0.6.35     192.168.1.1

   Tunnel VPN: PC2 riceve un indirizzo dal pool 192.168.50.1 - 192.168.50.250
```

- **WAN** del firewall collegata alla rete del laboratorio
- **LAN** del firewall collegata a PC1
- **PC2** lasciato sulla rete del laboratorio, per simulare un utente esterno

## Configurazione

1. **Individuazione del firewall:** l'indirizzo LAN del firewall corrisponde al gateway predefinito di PC1.
2. **Accesso alla GUI:** la password di amministrazione era stata cambiata da un gruppo precedente, quindi abbiamo fatto un reset del firewall e impostato una nuova password al primo accesso.
3. **Raccolta informazioni:** in *Network Status > Interface Status* abbiamo annotato gli indirizzi WAN e LAN e gli host collegati.
4. **Creazione della VPN** con il wizard di accesso remoto:
   - autenticazione con chiave pre-condivisa (PSK)
   - pool di indirizzi per i client VPN: `192.168.50.1 - 192.168.50.250`
   - regola di accesso creata dal wizard (`RemoteAccess_L2TP_Wiz`)
   - DNS fornito dal firewall
   - creazione di un utente dedicato all'accesso VPN
5. **Configurazione del client:** lo script batch generato dal firewall non funzionava, probabilmente per mancanza di privilegi. Abbiamo quindi configurato la VPN a mano nelle impostazioni di rete di Windows su PC2.

## Verifiche

| Test | Risultato |
| --- | --- |
| Il firewall vede il client remoto connesso | confermato dalla GUI del firewall |
| `ping` da PC2 (VPN) a PC1 (LAN) | riuscito |
| `ping` da PC1 (LAN) a PC2 (VPN) | riuscito: la comunicazione funziona in entrambe le direzioni |
| Accesso alla GUI del firewall (`192.168.1.1`) da PC2 | riuscito |
| Indirizzo di PC2 dopo la connessione | assegnato dal pool VPN |

## Considerazioni di sicurezza

- Le credenziali di fabbrica e quelle lasciate dai gruppi precedenti sono il primo punto debole di qualsiasi apparato: andrebbero cambiate subito e conservate in un password manager.
- Una PSK condivisa da tutti gli utenti è comoda ma debole: se un utente la diffonde, va cambiata per tutti. In un ambiente reale è meglio un'autenticazione per singolo utente, ad esempio con certificati o un server RADIUS.
- Dare ai client VPN accesso alla GUI di amministrazione del firewall è utile in laboratorio, ma in produzione l'accesso alla gestione andrebbe limitato a una rete di management.

## Cosa ho imparato

- come funziona un tunnel di accesso remoto e come il client riceve un indirizzo interno
- verificare la connettività in entrambe le direzioni, non solo dal client
- quando uno strumento automatico (lo script) fallisce, capire cosa fa e farlo a mano
