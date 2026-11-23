# Sistema de Rede e Monitorización para Contornos de Automoción Distribuídos

Proxecto de Fin de Ciclo (ASIR) centrado no deseño, despregue e monitorización dunha infraestrutura virtualizada sobre **Proxmox VE** para a xestión centralizada de servizos críticos nunha rede de talleres distribuídos.

## Arquitectura e Segmentación de Rede

A infraestrutura estrutúrase sobre un switch virtual `vmbr1` (modo VLAN aware) sobre Proxmox VE, aplicando illamento de capa 2 e filtrado perimetral mediante regras **iptables** baixo o principio de mínimo privilexio.

| VLAN | Subrede | Enderezo IP | Servizo / Función |
| :--- | :--- | :--- | :--- |
| **VLAN 10** | `10.0.10.0/24` | `10.0.10.10` | **Base de Datos:** Servidor MariaDB para almacenamento de métricas e eventos. |
| **VLAN 20** | `10.0.20.0/24` | `10.0.20.10` | **Xestión Web:** Servidor Apache HTTP + PHP (Panel de control e administración). |
| **VLAN 30** | `10.0.30.0/24` | `10.0.30.10` | **Identidade e DNS:** Controlador de Dominio Samba4 AD DC (`tfc.local`). |
| **VLAN 40** | `10.0.40.0/24` | DHCP | **Clientes e Monitorización:** Estacións dos talleres (Windows 11 / Debian) con axentes Telegraf. |

## Stack Tecnolóxico

* **Virtualización:** Proxmox VE (contedores LXC baseados en Debian 12 e máquinas virtuais KVM).
* **Seguridade Perimetral:** Netfilter / `iptables` (NAT e control estrito de tráfico inter-VLAN).
* **Directorio Activo e Resolución de Nomes:** Samba4 AD DC con integración Kerberos, LDAP e SSSD en clientes Linux.
* **Monitorización:** Axentes Telegraf configurados para o envío periódico de métricas (cada 5 segundos) mediante o plugin `outputs.sql`.
* **Persistencia de Datos:** MariaDB.
* **Interface de Xestión:** Aplicación web desenvolvida en PHP para a supervisión de servizos (`sssd`, `ssh`, `cron`, `ufw`, `telegraf`) e execución remota de comandos de control vía SSH con privilexios acoutados mediante `sudoers`.

## Estrutura do Repositorio

* `docs/`: Documentación completa do proxecto (Memoria técnica, Manual de Administración e Manual de Usuario).
* `network/`: Ficheiro de configuración de interfaces de rede, VLANs e regras de filtrado `iptables` do nodo Proxmox.
* `monitoring/`: Ficheiro de configuración do axente Telegraf (`telegraf.conf`) e política de permisos acoutados (`sudoers-telegraf`).
* `web-dashboard/`: Código fonte do panel de monitorización e control de servizos (`index.php`).

## Notas de Despregue

Por motivos de seguridade, as credenciais de acceso á base de datos nos ficheiros `telegraf.conf` e `index.php` foron substituídas polo marcador `CHANGE_ME`. Antes de despregar nun contorno de probas, débense configurar as variables de contorno ou credenciais correspondentes, así como os enderezos IP da interface WAN no ficheiro `network/interfaces`.
