# Diagrama de red

## Introducción

Esquema general de la arquitectura de **HoneyNet**: la red del cliente, segmentada en cinco VLANs por un firewall con detección de intrusiones, y el flujo de tráfico permitido entre ellas.

Todo ello se simula dentro de un **laboratorio**: un único PC con **Proxmox** que representa la infraestructura del cliente. En un cliente real, la red ya existiría; lo que aporta HoneyNet es la segmentación, el firewall, los señuelos y la monitorización.

---

## Esquema general

```mermaid
flowchart TB
    INTERNET(("Internet"))
    ADMIN(("Administrador<br/>(NetBird)"))

    subgraph LAB["Laboratorio de simulación — PC con Proxmox"]

        subgraph WAN["VLAN 99 — Exterior (WAN)"]
            KALI["Kali Linux<br/>(atacante de pruebas)"]
        end

        PF{{"pfSense + Suricata<br/>Firewall / Router / IDS"}}

        subgraph DMZ["VLAN 10 — DMZ"]
            COWRIE["Cowrie<br/>SSH / Telnet"]
            DIONAEA["Dionaea<br/>SMB / FTP / MySQL"]
            DVWA["DVWA<br/>Web vulnerable"]
        end

        subgraph SRV["VLAN 20 — Servidores internos (cliente y HoneyNet)"]
            DNS["Bind9 (DNS)"]
            DHCP["isc-dhcp-server"]
            MYSQL[("MySQL de HoneyNet<br/>datos de ataques")]
            BACKUP["rsync + cron<br/>(Backup de HoneyNet)"]
        end

        subgraph MGMT["VLAN 30 — Gestión"]
            WAZUH["Wazuh (SIEM)"]
            GRAFANA["Grafana"]
        end

        subgraph USERS["VLAN 40 — Usuarios"]
            CLIENT["VM Windows/Linux<br/>(empleado del cliente)"]
        end
    end

    INTERNET --> PF
    KALI -- "ataques de prueba" --> PF
    PF -- "tráfico permitido" --> DMZ
    PF <--> SRV
    PF <--> MGMT
    PF <--> USERS

    USERS -- "DNS / DHCP" --> SRV
    ADMIN -- "único acceso directo" --> MGMT

    DMZ -. "logs (solo salida)" .-> MGMT
    SRV -. "logs" .-> MGMT
    USERS -. "logs" .-> MGMT

    classDef honeynet fill:#ffedd5,stroke:#ea580c,color:#000
    classDef cliente fill:#dbeafe,stroke:#2563eb,color:#000
    classDef lab fill:#f3f4f6,stroke:#6b7280,color:#000
    class PF,COWRIE,DIONAEA,DVWA,WAZUH,GRAFANA,MYSQL,BACKUP honeynet
    class DNS,DHCP,CLIENT cliente
    class KALI lab
```

### Leyenda

| Color | Rol | Significado |
|---|---|---|
| 🟧 Naranja | **HoneyNet** | Lo que implanta la empresa en el cliente |
| 🟦 Azul | **Cliente** | La infraestructura del cliente (simulada) |
| ⬜ Gris | **Laboratorio** | Solo existe para simular el escenario |

> La zona de servidores internos es mixta: DNS y DHCP son del cliente; la base de datos y las copias de seguridad son de HoneyNet.

---

## Reglas de tráfico

| Origen | Destino | ¿Permitido? | Motivo |
|---|---|---|---|
| Exterior (Kali / Internet) | DMZ, vía pfSense | ✅ Sí | Es el tráfico que el proyecto busca atraer y capturar |
| Exterior (Kali / Internet) | Servidores / Gestión / Usuarios | ❌ No | pfSense nunca expone directamente las VLANs internas al exterior |
| DMZ | Cualquier otra VLAN | ❌ No (salvo logs) | Si un honeypot se compromete, debe quedar aislado |
| DMZ | Gestión | ✅ Solo logs (un sentido) | Monitorización, no es tráfico de ataque |
| Servidores | Gestión | ✅ Solo logs | Igual que el caso anterior |
| Usuarios | Servidores | ✅ Sí | Consumo normal de DNS/DHCP |
| Usuarios | DMZ / Gestión | ❌ No | Los empleados del cliente no tienen motivo para tocar esas VLANs |
| Administrador (NetBird) | Gestión | ✅ Único acceso directo | Es la VLAN más protegida de todas |

[← Introducción](README.md)
