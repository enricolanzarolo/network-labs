# 01 - Firewall OPNsense in Hyper-V

**Obiettivo:** proteggere una rete interna con un firewall OPNsense virtualizzato, applicando una politica *deny by default*: tutto è bloccato tranne il traffico esplicitamente consentito.

**Ambiente:** Hyper-V, OPNsense, client Windows 10.

## Topologia

```
  Rete del laboratorio / Internet
              |
     [vSwitch WAN - esterno]
              |
         hn0 (WAN, DHCP)
      +------------------+
      |  OPNsense (VM)   |
      +------------------+
         hn1 (LAN, 192.168.50.1/24)
              |
     [vSwitch LAN - interno]
              |
   Client Windows 10 (VM)
   192.168.50.150, gateway 192.168.50.1
```

## Infrastruttura virtuale

Sull'host Hyper-V ci sono due switch virtuali:

- **vSwitch WAN (esterno):** collegato alla scheda di rete fisica, permette al firewall di raggiungere la rete del laboratorio e quindi Internet.
- **vSwitch LAN (interno):** una rete privata visibile solo alle VM e all'host. Dall'esterno non arriva traffico se non passa dal firewall.

## Firewall

Il firewall è il punto di passaggio obbligato tra LAN e Internet, con due interfacce:

- **WAN (hn0):** indirizzo ottenuto via DHCP dalla rete del laboratorio. È il lato esposto.
- **LAN (hn1):** indirizzo statico `192.168.50.1/24`, gateway predefinito della rete interna.

Sulla LAN il firewall offre:

- **DNS:** risolve i nomi per i client interni
- **NAT:** traduce gli indirizzi privati della LAN nell'indirizzo della WAN (outbound NAT in modalità *automatic*)
- **Filtraggio:** applica le regole descritte sotto

Il client Windows ha un indirizzo statico `192.168.50.150` e usa il firewall come gateway. Il server DHCP sulla LAN è stato disattivato su indicazione del docente, per evitare conflitti con il DHCP del laboratorio.

## Regole (interfaccia LAN)

| # | Azione | Protocollo | Destinazione | Scopo |
| --- | --- | --- | --- | --- |
| 1 | Pass | TCP/UDP | porta 53 | risoluzione DNS |
| 2 | Pass | TCP | porte 80 e 443 | navigazione web HTTP/HTTPS |
| 3 | Block | ICMP | qualsiasi | blocco del ping verso l'esterno |
| 4 | Block | qualsiasi | qualsiasi | default deny: tutto il resto è bloccato |

## Verifiche

| Regola | Test | Risultato atteso |
| --- | --- | --- |
| DNS | `nslookup google.com` dal client | l'IP viene risolto e nella vista *Live* del firewall compaiono connessioni consentite |
| HTTP/HTTPS | navigazione dal browser del client | le pagine si caricano |
| ICMP | `ping 8.8.8.8` dal client | "Richiesta scaduta": il ping viene bloccato |
| Default deny | qualsiasi altro servizio | traffico bloccato, visibile nella vista *Live* |

## Correzione: la regola HTTP/HTTPS

Nella prima versione la regola 2 usava un intervallo di porte **da 80 a 443**. In questo modo il firewall consentiva tutte le porte comprese tra 80 e 443, cioè 364 porte invece di 2, contraddicendo la logica *deny by default*.

La configurazione corretta usa un **alias** di tipo *Port(s)* che contiene solo `80` e `443`, oppure due regole separate. È un errore facile da fare nell'interfaccia, e mostra perché ogni regola va verificata anche su ciò che **non** dovrebbe passare, non solo su ciò che deve funzionare.

## Problema aperto

Durante il laboratorio il client non riusciva a navigare. Le cause più probabili individuate:

- outbound NAT sulla WAN configurato in modo errato
- vSwitch WAN di Hyper-V non associato correttamente alla scheda fisica

Soluzione proposta: impostare *Firewall > NAT > Outbound* su **Automatic** e verificare l'associazione del vSwitch esterno alla scheda di rete fisica in Hyper-V.

## Cosa ho imparato

- separare reti con switch virtuali interni ed esterni
- ragionare in logica *deny by default*, aprendo solo il necessario
- verificare ogni regola con un test concreto, guardando i log in tempo reale
- un intervallo di porte sbagliato può aprire molto più del previsto
