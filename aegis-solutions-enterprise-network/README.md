<div align="center">

# Aegis Solutions: Infraestructura de Red Empresarial

![Cisco](https://img.shields.io/badge/CISCO-IOS-c084fc?style=flat-square&logo=cisco&logoColor=white)
![PNETLab](https://img.shields.io/badge/SIMULACI%C3%93N-PNETLAB-f472b6?style=flat-square)
![OSPF](https://img.shields.io/badge/ENRUTAMIENTO-OSPF%20MULTI--%C3%81REA-60a5fa?style=flat-square)
![DMVPN](https://img.shields.io/badge/WAN-DMVPN%20%2B%20IPSEC-8b5cf6?style=flat-square)
![Estado](https://img.shields.io/badge/PROYECTO-GRUPAL-fbbf24?style=flat-square)

</div>

---

> [!NOTE]
> **Proyecto grupal.** Este es el proyecto final de la materia *Conmutación y Enrutamiento (CCNA 3)* del ITLA (2026), realizado por un equipo de seis estudiantes. Esta carpeta documenta el proyecto y mi participación en él. El detalle de los roles está en la sección [Equipo y mi participación](#equipo-y-mi-participación).

## Descripción general

**Aegis Solutions** es una empresa ficticia del sector de ciberseguridad (proveedor de servicios gestionados de seguridad, MSSP) con presencia en cuatro sedes: **Santo Domingo** (sede central), **Santiago**, **La Romana** y **Puerto Plata**. El proyecto diseña e implementa su infraestructura de red completa sobre un modelo jerárquico de tres capas (núcleo, distribución y acceso), con una topología **hub-and-spoke** sobre **DMVPN**.

La red incluye segmentación por VLANs, direccionamiento con VLSM, enrutamiento dinámico OSPF multiárea, redundancia de primer salto con HSRP, agregación de enlaces con LACP, salida a Internet con NAT/PAT, servicios de red (DHCP, DNS, correo, FTP y RADIUS) y controles de seguridad en capa 2.

Todo se diseñó y validó en **PNETLab**, con routers Cisco 7200, switches Catalyst 3560 y 2960 y servidores Linux emulados.

---

## Equipo y mi participación

| Integrante | Rol en el proyecto |
|---|---|
| Reylin Santana Ortega | Líder / Arquitecto de Red y Seguridad |
| **Neury Jorge Montero Mejía** | **Co-Líder / Ingeniería WAN y Seguridad** (CTO y Arquitecto de Red en la empresa simulada) |
| Franchesca Soto Abreu | Equipo de Redes / Conectividad L3 |
| Darling Nathali Fortuna García | Documentación |
| Randy Gabriel Troncoso Tejeda | Documentación |
| Starlin De La Cruz Alonzo | Marketing y servidores |

### Mi participación

Según la documentación del proyecto, mis responsabilidades fueron:

- Co-liderazgo del equipo junto con el líder del proyecto.
- **Diseño de la topología y de la infraestructura LAN/WAN** de alto nivel.
- **Enrutamiento**: OSPF multiárea, traducción de direcciones (NAT/PAT) y redundancia de primer salto (HSRP).
- **Conectividad WAN** entre sedes.
- **Scripts de configuración de Santo Domingo** (router R-SD, switches de distribución SWM-1 y SWM-2 y switches de acceso SW-10, SW-11 y SW-12), en conjunto con el líder del proyecto.

El resto del trabajo fue del equipo: los scripts de La Romana, Santiago y los servidores, la documentación y los materiales de presentación.

---

## Arquitectura

### Sedes

| Sede | Función | Router | Rol DMVPN | Switches | Área OSPF |
|---|---|---|---|---|---|
| **Santo Domingo** | Administración, SOC e infraestructura corporativa | R-SD (Cisco 7200) | **Hub** | SWM-1 y SWM-2 (L3), SW-10, SW-11 y SW-12 (acceso) | 10 |
| **Santiago** | Centro de datos y servicios empresariales | R-SANTIAGO | Spoke | SW-1 y SW-2 (L3) y servidores Linux | 20 |
| **La Romana** | Consultoría e implementación de proyectos | R-ROMANA | Spoke | SW-3 (distribución) y SW-9 (acceso) | 30 |
| **Puerto Plata** | Laboratorio avanzado de ciberseguridad | R-PUERTO-PLATA | Spoke | SW-5 (L3), SW-30 y SW-31 (acceso) | 40 |

Los túneles DMVPN forman el **Área 0** (backbone) de OSPF, y cada sede tiene su propia área. La nube **ISP-NUBE** conecta las cuatro sedes mediante enlaces /30 y proporciona la salida a Internet con NAT.

### Tecnologías implementadas

| Tecnología | Uso en el proyecto |
|---|---|
| **DMVPN (mGRE + NHRP) con IPsec** | Túneles cifrados hub-and-spoke entre las sedes |
| **OSPF multiárea** | Área 0 (túnel) y un área por sede |
| **VLANs + 802.1Q** | Segmentación por departamento con enlaces troncales |
| **VLSM** | Direccionamiento privado eficiente y direccionamiento público para los enlaces con el ISP |
| **HSRP** | Redundancia de gateway entre los switches de distribución de Santo Domingo |
| **EtherChannel (LACP)** | Agregación de enlaces y redundancia en capa 2 |
| **Rapid-PVST, PortFast y BPDU Guard** | Prevención de bucles en capa 2 |
| **NAT/PAT** | Salida a Internet desde direccionamiento privado |
| **DHCP** | Pools por VLAN en el router central y `ip helper-address` en los switches de distribución |
| **Port Security y DHCP Snooping** | Seguridad en los puertos de acceso |
| **ACLs** | Acceso de administración restringido a las líneas VTY y filtrado de tráfico |
| **SSH v2** | Administración remota segura con claves RSA de 2048 bits |
| **Servicios en Santiago** | DHCP, DNS, correo, FTP y RADIUS sobre servidores Linux |

### Direccionamiento público

| Propósito | Red |
|---|---|
| Enlace ISP a Santo Domingo | `1.0.0.0/30` |
| Enlace ISP a Santiago | `1.0.0.4/30` |
| Enlace ISP a La Romana | `1.0.0.8/30` |
| Enlace ISP a Puerto Plata | `1.0.0.12/30` |
| NAT y servicios web y correo (Santiago) | `1.0.0.16/28` |

El direccionamiento privado completo (VLANs, hosts y subredes por departamento) está en [`docs/direccionamiento-ip.md`](./docs/direccionamiento-ip.md).

### Departamentos por sede

| Sede | Departamentos |
|---|---|
| Santo Domingo | Dirección General, Recursos Humanos, Cumplimiento y Auditoría, Soporte Técnico, Finanzas, Ventas y Marketing |
| Santiago | Centro de Datos, Ventas Corporativas, Administración y Servidores |
| La Romana | Dirección Regional, Consultoría TI e Ingeniería de Infraestructura |
| Puerto Plata | DFIR, Malware Research Lab, Threat Intelligence, Red Team y Ciberseguridad (SOC) |

---

## Estado de implementación

| Fase | Alcance | Estado |
|:--:|---|---|
| 1 | Santo Domingo (hub) + La Romana (spoke) + ISP-NUBE | Implementada |
| 2 | Santiago: router, switches y servidores Linux | Scripts de configuración creados |
| 3 | Puerto Plata: router y switches | Scripts de configuración creados |

---

## Contenido del repositorio

```text
aegis-solutions-enterprise-network/
├── configs/
│   ├── santo-domingo/      # R-SD, VPN, SWM-1, SWM-2, SW-10, SW-11, SW-12
│   ├── la-romana/          # R-Romana, SW-3, SW-9
│   ├── santiago/           # R-Santiago, switches y servicios (DHCP, DNS, FTP, correo, RADIUS)
│   ├── puerto-plata/       # R-PuertoPlata, SW-5, SW-30, SW-31
│   └── isp-nube.md         # Router del ISP
├── docs/
│   ├── direccionamiento-ip.md
│   ├── equipos.md
│   ├── organigrama.md
│   └── roles-y-cargos.md
├── scripts/                # Scripts de configuración y endurecimiento por equipo
└── README.md
```

> [!IMPORTANT]
> Las contraseñas, el *pre-shared key* de IPsec y la clave de autenticación de NHRP del proyecto original se reemplazaron en esta copia por marcadores (`<PASSWORD>`, `<PSK-VPN>` y `<NHRP-KEY>`). Es una buena práctica no publicar credenciales, aunque sean de laboratorio.

---

## Plataforma de simulación

- **PNETLab** como plataforma de emulación.
- **Cisco 7200** para los routers y **Catalyst 3560** y **2960** para los switches de distribución y acceso.
- **Linux (Ubuntu)** para los servidores de servicios.

---

## Documentación completa del equipo

El repositorio original del equipo contiene además la documentación profesional del proyecto (PDF), la presentación, los documentos de la empresa simulada (contrato, presupuesto y cotizaciones) y el archivo de topología de PNETLab: **[Aegis-Solution-Documentacion](https://github.com/TU-ENLACE-AL-REPO-ORIGINAL)**.

---

## Lo aprendido

Este proyecto permitió diseñar e implementar una red empresarial multisede de principio a fin: planificar el direccionamiento con VLSM, interconectar las sedes con DMVPN e IPsec, enrutar con OSPF multiárea y proteger la capa 2 con redundancia y controles de acceso.

También permitió trabajar en equipo con roles definidos, dividiendo la configuración por sede y manteniendo la documentación al día. En la revisión final del proyecto se unificaron las claves, el modo de EtherChannel (LACP), las áreas OSPF y los banners entre los documentos y los scripts, lo que reforzó la importancia de que la documentación sea coherente con las configuraciones reales.

---

## Licencia

Proyecto académico del ITLA (2026), materia Conmutación y Enrutamiento. La autoría es compartida con el equipo de trabajo; consulta el repositorio original para más detalles.

---

## 👨‍💻 Autor

**Neury Jorge Montero Mejía**
*Estudiante de Tecnólogo en Seguridad Informática*

[![LinkedIn](https://img.shields.io/badge/LINKEDIN-NEURY%20MONTERO-0A66C2?style=flat-square&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/TU-USUARIO)
[![GitHub](https://img.shields.io/badge/GITHUB-NEURYJMM-111111?style=flat-square&logo=github&logoColor=white)](https://github.com/NeuryJMM)

---
