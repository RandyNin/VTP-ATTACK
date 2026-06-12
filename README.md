# VTP-ATTACK

> **Autor:** Randy Nin **Laboratorio de Seguridad de Redes | GNS3**

Demostración de dos ataques VTP (VLAN Trunking Protocol) encadenados mediante Yersinia: primero se establece un trunk malicioso explotando DTP (prerequisito), y desde esa posición se envían tramas VTP falsificadas con un Configuration Revision Number artificialmente elevado para agregar una VLAN no autorizada (VLAN 50 "PWned") y eliminar una VLAN operacional (VLAN 20 "TI") en todos los switches del dominio simultáneamente.

---

## Contenido del repositorio

```
VTP-ATTACK/
├── Documentación Técnica Profesional VTP-ATTACK (Randy Nin -- 2025-0660).pdf
└── README.md
```

> Este laboratorio utiliza Yersinia, herramienta preinstalada en Kali Linux. No se incluye script propio.

---

## Documentación técnica

La documentación técnica completa está disponible en:

**[Documentación Técnica Profesional VTP-ATTACK (Randy Nin -- 2025-0660).pdf](Documentación%20Técnica%20Profesional%20VTP-ATTACK%20(Randy%20Nin%20--%202025-0660).pdf)**

Incluye contexto técnico de VTP y su vulnerabilidad en v1/v2, análisis del Configuration Revision Number como vector de ataque, topología y configuración del entorno, procedimiento completo con Yersinia (prerequisito DTP + ataque VTP), evidencia de la VLAN inyectada y eliminada, y contramedidas con VTPv3 y contraseña.

---

## Requisitos

**Sistema:** Kali Linux (Yersinia preinstalado), ParrotSec OS o cualquier distribución Linux compatible.

**Privilegios:** Ejecución obligatoria con `sudo`.

**Instalación de Yersinia** (si no está disponible):

```bash
sudo apt update && sudo apt install yersinia -y
```

**Conectividad requerida:** El atacante debe estar conectado a un puerto del switch en modo `dynamic auto` para el prerequisito DTP. Sin trunk activo, las tramas VTP no llegan al dominio.

---

## Cómo funciona

El ataque se desarrolla en dos fases obligatoriamente secuenciales:

**Fase 1 - Trunk malicioso por DTP (prerequisito):**

```bash
sudo yersinia -I
```

Dentro de la interfaz: `g` -> seleccionar `DTP` -> `x` -> seleccionar `enabling trunking` -> `Enter`

El puerto Gi0/0 de Sw-2 (modo `dynamic auto`) transiciona a trunk con encapsulación `n-802.1q`. Las tramas VTP ahora pueden fluir entre Kali y el dominio RANDY.

**Fase 2 - Manipulación de VLANs por VTP:**

Una vez activo el trunk: `g` -> seleccionar `VTP` -> `x` -> configurar el tipo de ataque y parámetros -> `Enter` -> `x` -> seleccionar `sending VTP packet` -> `Enter`

Yersinia envía un VTP Summary Advertisement con un Configuration Revision Number mayor al del dominio. Al ser superior, todos los VTP Clients lo aceptan y reemplazan su base de datos de VLANs con la que el atacante envió.

|Ataque|Parámetros|Resultado|
|:--|:--|:--|
|Agregar VLAN|ID: 50, Name: PWned|VLAN 50 "PWned" aparece en todos los switches|
|Eliminar VLAN|ID: 20|VLAN 20 "TI" desaparece de toda la infraestructura|

---

## Controles de Yersinia

|Tecla|Acción|
|:-:|:--|
|`g`|Menú de selección de protocolo|
|`x`|Panel de ataques del protocolo activo|
|`Enter`|Confirmar selección o parámetros|
|`l`|Tabla de vecinos y tramas detectadas|
|`q` / `ESC`|Salir del menú|

---

## Entorno de laboratorio

|Dispositivo|Rol|VTP Mode|IP|
|:--|:--|:--|:--|
|R-1|Gateway / DHCP / NAT|N/A|25.6.60.1/25|
|Sw-1|VTP Server / Switch central|Server (RANDY)|N/A|
|Sw-2|VTP Client / Switch víctima|Client (RANDY)|N/A|
|Sw-3|VTP Client|Client (RANDY)|N/A|
|Kali Linux|Atacante|N/A|DHCP desde R-1|

**VLANs iniciales del dominio RANDY:**

|VLAN|Nombre|Estado tras ataques|
|:--|:--|:--|
|1|default|Sin cambios|
|10|RRHH|Sin cambios|
|20|TI|Eliminada por el atacante|
|30|Marketing|Sin cambios|
|50|PWned|Creada por el atacante|

---

## Impacto observado

- VLAN 50 "PWned" propagada a todos los switches del dominio sin intervención administrativa
- VLAN 20 "TI" eliminada de toda la infraestructura: cualquier host en esa VLAN pierde conectividad
- Ambas operaciones ocurren en segundos desde un único puerto de acceso
- No se requiere acceso físico al switch ni credenciales administrativas

---

## Mitigación

**En Sw-2 (switch del atacante):**

```
Sw-2(config)# vtp version 3
Sw-2(config)# vtp password randy123
Sw-2(config)# interface GigabitEthernet0/0
Sw-2(config-if)# switchport mode access
Sw-2(config-if)# switchport access vlan 1
Sw-2(config-if)# no negotiation auto
```

**En Sw-1 (VTP Server):**

```
Sw-1(config)# vtp version 3
Sw-1(config)# vtp password randy123
Sw-1(config)# end
Sw-1# vtp primary vlan
```

|Medida|Protege contra|
|:--|:--|
|`vtp version 3` + `vtp password`|VTP: solo el primary server puede originar cambios; el hash de contraseña debe coincidir|
|`switchport mode access` + `no negotiation auto`|DTP: el puerto no negocia trunks, bloqueando el prerequisito del ataque|

Con ambas medidas activas, el trunk malicioso no puede establecerse y las tramas VTP falsificadas del atacante no alcanzan el dominio.

---

## Video demostrativo

[https://www.youtube.com/watch?v=N-jP3jnqA5g](https://www.youtube.com/watch?v=N-jP3jnqA5g)

---

## Disclaimer

Este laboratorio fue desarrollado con fines exclusivamente académicos y educativos. Su uso está permitido únicamente en entornos propios o autorizados como GNS3, EVE-NG o laboratorios internos de prueba. El uso en redes de producción o de terceros sin autorización expresa constituye una violación legal.

---

_Randy Nin / Matrícula 2025-0660_

---
