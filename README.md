# fortigate-vpn-forti-forti-lab-0791
# Laboratorio de Redes Seguras: Infraestructura 1 (Site-to-Site IPsec VPN)

[![Demostración en Video](https://youtu.be/4iSpCCUE00M)

---

## Propósito del Laboratorio

El propósito de esta práctica es diseñar, implementar y documentar una infraestructura de red segura que interconecte dos sedes corporativas (Sede A / HQ y Sede B / Branch) a través de un túnel VPN IPsec Site-to-Site utilizando appliances de seguridad FortiGate.

El objetivo principal es garantizar que el tráfico generado entre la zona de usuarios (Sede A) y los servicios publicados en la DMZ (Sede B) fluya de manera cifrada, verificando además la dependencia estricta del túnel: si la VPN se interrumpe, el tráfico debe bloquearse por completo evitando fugas de información a través del proveedor de servicios (ISP).

---

## Diagrama de la Topología


<img width="404" height="380" alt="imagen_2026-10-02_181726138" src="https://github.com/user-attachments/assets/9bc276a9-9359-4b38-8358-b0cd8290b4b2" />.*

---

## Plan de Direccionamiento IP y Segmentación

| Dispositivo / Sede | Interfaz | Zona / Rol | Dirección IP / Máscara | Notas |
| :--- | :--- | :--- | :--- | :--- |
| **FortiGate-A** | `port1` | WAN (ISP) | `10.7.91.2/29` | IP Pública simulada |
| **FortiGate-A** | `port2` | LAN Users | `10.7.91.129/25` | Gateway VLAN 10 (DHCP Server) |
| **FortiGate-B** | `port1` | WAN (ISP) | `10.7.91.6/29` | IP Pública simulada |
| **FortiGate-B** | `port2` | DMZ Server | `10.7.91.17/28` | Gateway Servidor Web |
| **Webterm A** | `eth0` | Cliente LAN | `10.7.91.130/25` | Asignada por DHCP (`10.7.91.130 - 10.7.91.250`) |
| **Servidor Web B** | `eth0` | DMZ Server | `10.7.91.18/28` | IP Estática (Nginx) |

---

## Configuración e Implementación (GUI Fortinet)

### 1. Configuración de Interfaces y Servidor DHCP
- **FortiGate-A:** Se configuró la interfaz `port2` con la IP `10.7.91.129/25` y se habilitó el servidor DHCP en el rango `10.7.91.130 - 10.7.91.250`.
- **FortiGate-B:** Se configuró la interfaz `port2` con la IP `10.7.91.17/28` para la zona DMZ.

<img width="981" height="819" alt="Captura de pantalla 2026-10-02 173916" src="https://github.com/user-attachments/assets/26a1a1a8-d779-4542-88d7-591f994c507d" />

### 2. Configuración del Túnel IPsec Site-to-Site
Se utilizó el asistente **IPsec Wizard** en ambos FortiGate con los siguientes parámetros:
- **Nombre Túnel HQ (A):** `VPN_TO_B` (IP Remota: `10.7.91.6`)
- **Nombre Túnel Branch (B):** `VPN_TO_A` (IP Remota: `10.7.91.2`)
- **Pre-Shared Key (PSK):** `Itlazo12345@`
- **Subred Local A:** `10.7.91.128/25` | **Subred Remota B:** `10.7.91.16/28`

<img width="963" height="787" alt="Captura de pantalla 2026-10-02 174004" src="https://github.com/user-attachments/assets/7d5b6300-4bd4-4dc9-9aca-928c75b34280" />

### 3. Políticas de Firewall y NAT
Se crearon políticas bidireccionales en ambos equipos para permitir el tráfico entre las interfaces internas y las interfaces virtuales IPsec, deshabilitando el NAT sobre las rutas del túnel para conservar los direccionamientos de origen y destino.

---

## Pruebas y Validación de Resultados

### 1. Validación de Concesión DHCP en Sede A
El cliente `webterm` en la Sede A obtiene exitosamente su dirección dentro del rango /25.

<img width="648" height="128" alt="Captura de pantalla 2026-10-02 181946" src="https://github.com/user-attachments/assets/0dbc4d49-7728-4ed9-8343-e2657e6cac6d" />

<img width="981" height="819" alt="Captura de pantalla 2026-10-02 173916" src="https://github.com/user-attachments/assets/619c9d45-e3d7-4307-acfe-a34bd370a52e" />

### 2. Acceso al Servidor Web HTTPS/HTTP sobre la VPN (Túnel Activo)
Desde el navegador de la `webterm` A se accede a la IP del servidor Nginx (`10.7.91.18`), confirmando la conectividad de extremo a extremo.

<img width="898" height="615" alt="Captura de pantalla 2026-10-02 175433" src="https://github.com/user-attachments/assets/6fdd1730-09dc-4e75-aee2-a3dd115ce32d" />

### 3. Trazado de Ruta (Traceroute) con VPN Activa
Se ejecuta `traceroute 10.7.91.18` desde la `webterm` A, verificando que el tráfico salta directamente de la puerta de enlace local (`10.7.91.129`) al destino en la Sede B (`10.7.91.18`) a través del túnel seguro.

<img width="575" height="106" alt="Captura de pantalla 2026-10-02 175531" src="https://github.com/user-attachments/assets/dcc370c9-e016-4cd3-a58b-60daab85536b" />

### 4. Prueba de Interrupción de Tráfico (Túnel Inactivo / Bring Down)
Se fuerza la caída del túnel IPsec en la GUI de FortiGate-A seleccionando `VPN_TO_B` $\rightarrow$ **Bring Down**. Al repetir el trazado de ruta, las peticiones caen en tiempo de espera (*Timeout*), demostrando que la red no expone tráfico fuera del túnel VPN.

<img width="577" height="144" alt="Captura de pantalla 2026-10-02 175642" src="https://github.com/user-attachments/assets/08c8b218-5d53-428a-91d3-60c40a2eb695" />

---

## 📂 Archivos del Repositorio

- `/fortigateA_HQ.conf`: Backup de configuración completa de FortiGate-A.
- `/fortigateB_Branch.conf`: Backup de configuración completa de FortiGate-B.
- `/img/`: Capturas de pantalla utilizadas en la documentación.

---

## 🎥 Demostración en Video

El video con la explicación detallada, verificación en vivo y rostro del estudiante se encuentra disponible en el siguiente enlace:

👉 **[Ver Video Demostrativo del Laboratorio](https://youtu.be/4iSpCCUE00M)**




