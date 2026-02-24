[Volver al índice general](../README.md)
# UD2 - Diseño técnico de la infraestructura empresarial segura y automatizada

## 1. Recopilación de información técnica

Para el despliegue del proyecto de la **Librería Digital** (venta de libros online y gestión de inventario), se han establecido los siguientes requisitos técnicos, consolidando los servicios en zonas de red segmentadas para optimizar la seguridad y permitir una administración ágil bajo filosofía DevOps.

### 1.1 Inventario de Hardware (Equipos/Nodos Virtuales)

1. **Servidor Perimetral (Firewall y Enrutador):**
   * **Rol:** Enrutador principal y barrera de seguridad. Conecta y aísla la WAN (Internet), la red de gestión (LAN), la zona de servidores públicos (DMZ), la red de monitorización y los túneles VPN.
2. **Servidor de Gestión (Zona LAN - VLAN 20):**
   * **Rol:** Controlador de Dominio (Active Directory). Centraliza las identidades de los empleados de la librería y las políticas de seguridad de los equipos de trabajo.
3. **Servidor Web / Nodo Orquestador (Zona DMZ - VLAN 10):**
   * **Rol:** Servidor público que actúa como nodo de un clúster Kubernetes. Contiene la tienda online de la librería, el catálogo de libros y el Ingress Controller, aislado de la red interna de la empresa.
4. **Servidor de Monitorización (Zona SOC - VLAN 30):**
   * **Rol:** Centro de Operaciones de Seguridad (SOC). Recolecta logs y métricas de toda la red para detectar intrusiones o caídas en la tienda online.

### 1.2 Requisitos de Software y Stack Tecnológico

* **Seguridad Perimetral y Acceso Remoto:** pfSense integrado con WireGuard VPN para la administración remota.
* **Orquestación de Contenedores:** K3s (Lightweight Kubernetes). Sustituye el uso de contenedores aislados por un entorno de alta disponibilidad que garantiza la autorrecuperación de los servicios (self-healing).
* **Servicios Web:** Despliegue de la tienda web mediante *Deployments* y enrutamiento del tráfico a través de un *Ingress Controller* (Traefik o Nginx Ingress).
* **Gestión de Identidad:** Windows Server 2022 (Active Directory) para la red local.
* **Monitorización y SIEM:** Wazuh (para detección de amenazas) y Grafana (para métricas de rendimiento y disponibilidad). Integración de copias de seguridad en la nube mediante AWS S3.

---

## 2. Diseño lógico y físico de la infraestructura

El diseño se basa en una arquitectura fuertemente segmentada gestionada por el firewall pfSense, garantizando que si la tienda online sufre un ciberataque, los datos internos de la empresa (facturación, empleados) queden protegidos.

### 2.1 Topología de Red y Segmentación

* **Zona WAN (Internet):** Recibe el tráfico de los clientes que compran libros.
* **VLAN 10 - DMZ (Red Perimetral):** Alojamiento del clúster K3s con la tienda web. Tiene bloqueado por firewall cualquier intento de conexión hacia la red de los empleados.
* **VLAN 20 - LAN (Red Interna):** Zona de alta seguridad para los empleados de la librería, con el Active Directory y las estaciones de trabajo.
* **VLAN 30 - SOC (Monitorización):** Red dedicada exclusivamente al stack de seguridad (Wazuh).
* **Zona VPN:** Túnel cifrado para que los administradores técnicos accedan a la infraestructura de forma segura desde el exterior.

### 2.2 Mapa de Servicios por Servidor

* **En la DMZ (Clúster K3s):** El Ingress Controller recibe las peticiones HTTP/HTTPS y redirige el tráfico limpio a los *Pods* de la App de la Librería. La base de datos de libros (MariaDB/MySQL) se despliega como un *StatefulSet* dentro de la red interna del clúster, totalmente inaccesible desde Internet de forma directa.
* **En la LAN:** Windows Server aplica políticas de grupo (GPOs) a los PCs del almacén y oficinas.
* **En la Nube (AWS):** Se envían backups cifrados diarios de los Volúmenes Persistentes (PVCs) de la base de datos de libros al bucket S3.

---

## 3. Definición de objetivos y fases del proyecto

### 3.1 Objetivos del Proyecto (SMART)

1. **Alta Disponibilidad Comercial (DevOps):** Garantizar que la tienda online de libros esté siempre operativa utilizando Kubernetes. Si un *Pod* falla, el orquestador detectará el error y levantará uno nuevo automáticamente en segundos.
2. **Aislamiento y Seguridad Corporativa:** Separar estrictamente la tienda web (DMZ) de los datos de gestión y contabilidad de la librería (LAN).
3. **Monitorización Proactiva:** Implementar un SIEM (Wazuh) para detectar intentos de intrusión o fuerza bruta contra la web de la librería en tiempo real.
4. **Resiliencia ante Ransomware:** Asegurar la persistencia del catálogo de libros mediante backups inmutables en AWS S3 de los volúmenes de Kubernetes.

### 3.2 Fases de Implementación

* **Fase 1 (Enrutamiento):** Instalación de pfSense, configuración de las 3 VLANs (DMZ, LAN, SOC) y túnel VPN.
* **Fase 2 (Infraestructura Local):** Despliegue de Windows Server (Active Directory) y unión de los equipos de los empleados al dominio.
* **Fase 3 (Orquestación K3s):** Instalación del plano de control de Kubernetes en el servidor de la DMZ.
* **Fase 4 (Servicios Web):** Despliegue de la tienda de libros y la base de datos mediante manifiestos YAML (Deployments, Services, Ingress, PVCs).
* **Fase 5 (Monitorización y Backups):** Instalación de Wazuh con despliegue de agentes en los nodos, y configuración de automatización de respaldos hacia AWS S3.

---

## 4. Recursos y presupuesto

El proyecto prioriza el uso de soluciones *Cloud Native* de código abierto, lo que elimina el coste de licenciamiento empresarial tradicional y permite a la librería invertir su presupuesto en inventario.

| Componente Lógico | Solución Elegida | Coste | Solución Propietaria (Alternativa) | Ahorro |
| :--- | :--- | :--- | :--- | :--- |
| **Firewall y VPN** | pfSense + WireGuard | 0 € | Hardware Fortinet / Cisco | > 1.000 € / año |
| **Orquestación** | Kubernetes (K3s) | 0 € | VMware Tanzu / OpenShift | > 1.500 € |
| **SIEM / Monitorización** | Wazuh + Grafana | 0 € | Splunk Enterprise | > 2.000 € |
| **Directorio Activo** | Windows Server 2022 | Licencia Pyme | - | - |
| **Backups en Nube** | AWS S3 | Pago por uso | Soluciones BaaS Corporativas | > 500 € / año |

---

## 5. Documentación técnica

Para asegurar el mantenimiento futuro y la escalabilidad de la librería digital, se generarán los siguientes documentos técnicos (Anexos):

1. **Esquema de Red:** Topología física y lógica con direccionamiento IP y reglas de firewall en pfSense.
2. **Manifiestos de Kubernetes:** Repositorio con los archivos `.yaml` (Deployments, Services, Ingress, PVCs) para poder recrear toda la infraestructura web en minutos.
3. **Manual de Monitorización:** Guía de respuestas ante alertas de seguridad generadas por Wazuh y visualización de paneles en Grafana.
4. **Plan de Recuperación ante Desastres (DRP):** Procedimiento paso a paso para restaurar los volúmenes de la base de datos desde los backups de AWS S3.

## Enlaces a recursos de la unidad

- [Documentos de la unidad](./documentos/)
- [Diagramas e imágenes](./img/)

## Bibliografía / Webgrafía
- Autor1, Título del libro o artículo, Editorial/Año.
- Sitio web oficial: [Enlace](https://www.ejemplo.com)
- Tutoriales y guías recomendadas: [Enlace](https://www.ejemplo2.com)
