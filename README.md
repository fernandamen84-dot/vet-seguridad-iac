# Proyecto Vet_Seguridad - Veterinaria Huellitas

## 📋 Descripción

Aplicación web de gestión veterinaria desplegada en GNS3 con Docker, aplicando:
- **Modelo de 3 capas** (Frontend, Backend, Base de Datos)
- **HTTPS/TLS** con certificados SSL
- **High Availability (HA)** con múltiples Frontends y Load Balancer
- **Fault Tolerance (FT)** con replicación MySQL Maestro-Esclavo
- **NAT** para salida a Internet
- **DNS** interno con dnsmasq
- **Subnetting** por VLANs
- **IaC (Infrastructure as Code)** con Dockerfiles y docker-compose

---

## 🏗️ Arquitectura de Red

### Topología

[Webterm]
↓
veterinaria.local
↓
[LOAD-BALANCER: 192.168.30.5]
(HAProxy - Puerto 443)
↙ ↘
[FRONTEND-1] [FRONTEND-2]
(192.168.30.10) (192.168.30.11)
(Nginx + HTTPS) (Nginx + HTTPS)
↘ ↙
[BACKEND: 192.168.20.10]
(Flask - Puerto 80)
↓
┌─────────┴─────────┐
↓ ↓
[MYSQL-1: .12] ←──replica──→ [MYSQL-2: .13]
(Maestro) (Esclavo)

[DNS: 192.168.20.20]
[PC1: 192.168.10.10]



### Tabla de Direccionamiento

| Componente | IP | VLAN | Puerto | Descripción |
|-----------|-----|------|--------|-------------|
| PC1 | 192.168.10.10 | 10 (Usuarios) | - | Cliente de pruebas |
| BACKENDBD-1 | 192.168.20.10 | 20 (Core) | 80 | Backend Flask |
| MYSQL-1 | 192.168.20.12 | 20 (Core) | 3306 | Base de datos Maestro |
| MYSQL-2 | 192.168.20.13 | 20 (Core) | 3306 | Base de datos Esclavo |
| DNS-1-1 | 192.168.20.20 | 20 (Core) | 53 | Servidor DNS dnsmasq |
| FRONTEND-1 | 192.168.30.10 | 30 (DMZ) | 443 | Nginx HTTPS |
| FRONTEND-2 | 192.168.30.11 | 30 (DMZ) | 443 | Nginx HTTPS |
| LOAD-BALANCER-1 | 192.168.30.5 | 30 (DMZ) | 443 | HAProxy |
| Webterm | 192.168.100.10 | Externa | - | Cliente web |
| R1 | 10.0.0.1 / 192.168.x.1 | - | - | Router core |
| R2 | 10.0.0.2 / 192.168.100.1 | - | - | Router externo (NAT) |

---

## 🔒 Características de Seguridad Implementadas

### 1. HTTPS/TLS
- Certificados SSL autofirmados generados con OpenSSL.
- Nginx escucha en puerto 443 (HTTPS).
- Puerto 80 redirige automáticamente a 443.
- Archivos: `/etc/nginx/ssl/veterinaria.crt` y `.key`

### 2. High Availability (HA)
- 2 Frontends (Nginx) balanceados por HAProxy.
- Algoritmo: `roundrobin`.
- Health checks con `option ssl-hello-chk`.
- Si un Frontend cae, el otro sigue respondiendo.

### 3. Fault Tolerance (FT)
- Replicación MySQL Maestro-Esclavo.
- `server-id=1` en Maestro, `server-id=2` en Esclavo.
- Binlog activado en el Maestro.
- Usuario `replicator` con permisos de replicación.
- Los datos se replican en tiempo real.

### 4. NAT
- Configurado en R2 (Router externo).
- `ip nat inside source list 1 interface f2/0 overload` (PAT).
- Permite salida a Internet desde cualquier VLAN interna.

### 5. DNS
- Servidor `dnsmasq` en 192.168.20.20.
- Resuelve `veterinaria.local` → 192.168.30.5 (Load-Balancer).
- Reenvía consultas externas a 8.8.8.8 y 1.1.1.1.

### 6. Subnetting
| VLAN | Red | Máscara | Uso |
|------|-----|---------|-----|
| 10 | 192.168.10.0/24 | /24 | Usuarios |
| 20 | 192.168.20.0/24 | /24 | Core Services |
| 30 | 192.168.30.0/24 | /24 | DMZ |
| WAN | 10.0.0.0/30 | /30 | Enlace R1-R2 |
| Externa | 192.168.100.0/24 | /24 | Red externa |

### 7. IaC (Infrastructure as Code)
- Dockerfiles para cada contenedor.
- `docker-compose.yml` para orquestación.
- Documentación completa en este repositorio.

---

## 📦 Estructura del Proyecto

vet-seguridad-iac/
├── README.md # Este archivo
├── docker-compose.yml # Orquestación de contenedores
└── dockerfiles/
└── backend/
└── Dockerfile # Ejemplo de Dockerfile del Backend


---

## 🚀 Despliegue con docker-compose

**Nota:** Este `docker-compose.yml` es una representación de la arquitectura. En el proyecto real se usa GNS3 con contenedores Docker individuales.

```bash
# Levantar todos los servicios
docker-compose up -d

# Ver estado
docker-compose ps

# Ver logs
docker-compose logs -f

# Detener
docker-compose down