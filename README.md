
# Actividad 6: Seguridad de dispositivos, redes y virtualización

2. [Punto 1: Resumen de seguridad de dispositivos](#punto-1-resumen-de-seguridad-de-dispositivos)
3. [Punto 2: Tecnologías de virtualización](#punto-2-tecnologías-de-virtualización)
4. [Punto 3.1: Switch con tres VLAN](#punto-31-switch-con-tres-vlan)
5. [Punto 3.2: Router y dos switches](#punto-32-router-y-dos-switches)
6. [Punto 3.3: Máquinas virtuales](#punto-33-máquinas-virtuales)
7. [Simulación en Python](#simulación-en-python)
8. [Conclusiones](#conclusiones)



## Punto 1: Resumen de seguridad de dispositivos

Hoy casi todos usamos un celular, un computador o una tablet con acceso a Internet, y en ellos guardamos fotos, videos, datos bancarios y conversaciones privadas. Sin medidas de protección, esa información queda vulnerable y los ciberdelincuentes pueden robarla o infectar el dispositivo. La guía propone tres hábitos: mantener los dispositivos actualizados, verificar que el antivirus esté activo y controlar el origen de las aplicaciones y sus permisos.

### 1.1 Cómo actualizar los dispositivos

Las actualizaciones corrigen fallos y vulnerabilidades que pueden ser aprovechados por ciberdelincuentes. En Windows 10:

1. Clic en el icono de Windows y luego en la rueda dentada (**Configuración**).
2. Entrar a **Actualización y seguridad**.
3. En **Windows Update**, pulsar **Buscar actualizaciones** si aparece "No está todo actualizado".
4. Activar **Instalar actualizaciones tan pronto como sea posible** y reiniciar cuando se solicite.

![Windows Update](01-resumen-seguridad/windows-update.png)

### 1.2 Cómo comprobar que los dispositivos están protegidos (antivirus)

| Sistema | Pasos |
|---|---|
| **Windows 10** | Configuración → Actualización y seguridad → Seguridad de Windows → Protección contra virus y amenazas. Un círculo verde con check indica que está activa. Se puede hacer un examen rápido. |
| **Android** | Play Store → icono de perfil → **Play Protect**. Debe indicar que no hay aplicaciones dañinas. Pulsar **Analizar** y activar "Analizar aplicaciones con Play Protect" y "Mejorar la detección de aplicaciones dañinas". |
| **macOS** | Menú Apple → Preferencias del Sistema → Seguridad y privacidad → General. Desbloquear el candado y elegir **App Store** en "Permitir apps descargadas de". |
| **iOS** | La protección principal son los filtros de la App Store. En Ajustes → Privacidad se revisan los permisos de cada app; se deshabilitan los que no correspondan y se mantienen las apps actualizadas. |

![Seguridad de Windows](01-resumen-seguridad/seguridad-windows.png)

> Los temas 1.3 (bloqueo de acceso), 1.4 (cifrado) y 1.5 (descarga segura de aplicaciones) aparecen en el índice de la guía, pero el material compartido no los desarrolla.

## Punto 2: Tecnologías de virtualización

| Tecnología | Tipo | Costo | Anfitrión | Fortaleza | Aplicación típica |
|---|---|---|---|---|---|
| **VirtualBox** | Hipervisor tipo 2 | Gratis (código abierto) | Windows, Linux, macOS | Sencillez, snapshots | Laboratorios y estudio |
| **WSL** | Capa de compatibilidad / VM ligera (WSL 2) | Gratis | Windows | Arranque rápido, integración con Windows | Desarrollo con herramientas Linux |
| **VMware** | Tipo 2 (Workstation) y tipo 1 (ESXi) | Mixto | Windows, Linux, macOS; ESXi sobre hardware | Rendimiento y gestión empresarial | Empresas y centros de datos |
| **QEMU** | Emulador / virtualizador | Gratis | Linux, Windows, macOS | Emula otras arquitecturas (ARM, RISC-V, MIPS) | Desarrollo embebido y pruebas |
| **KVM** | Módulo del kernel de Linux (tipo 1) | Gratis | Solo Linux | Rendimiento casi nativo | Nube y servidores |

- **VirtualBox:** producto de Oracle, fácil de usar, con carpetas compartidas e instantáneas.
- **WSL:** WSL 1 traduce llamadas del sistema; WSL 2 usa una VM ligera con un kernel Linux real.
- **VMware:** Workstation y Fusion para escritorio; ESXi y vSphere para servidores.
- **QEMU:** se maneja por línea de comandos; sin aceleración es lento.
- **KVM:** usa las extensiones Intel VT-x / AMD-V; se combina con QEMU (QEMU/KVM) y se gestiona con libvirt o virt-manager.

## Punto 3.1: Switch con tres VLAN

**Objetivo:** configurar un switch básico (24/48 puertos) con tres interfaces VLAN con máscaras /24, /16 y /8.

### Topología

```
          +------------------+
          |    SW-Compensar  |
          |  (switch capa 3) |
          +--+------+------+-+
         Fa0/1   Fa0/2   Fa0/3
            |       |       |
           PC1     PC2     PC3
```

![Topología switch](03-switch/capturas/topologia.png)

### Direccionamiento

| VLAN | Red | Máscara | IP interfaz VLAN | PC | Puerto |
|---|---|---|---|---|---|
| 10 | 192.168.10.0 | 255.255.255.0 (/24) | 192.168.10.1 | PC1: 192.168.10.10 | Fa0/1 |
| 20 | 172.16.0.0 | 255.255.0.0 (/16) | 172.16.0.1 | PC2: 172.16.0.10 | Fa0/2 |
| 30 | 10.0.0.0 | 255.0.0.0 (/8) | 10.0.0.1 | PC3: 10.0.0.10 | Fa0/3 |

> Un switch de capa 2 (por ejemplo el 2960) solo admite una IP de gestión. Para varias interfaces VLAN con IP y enrutamiento entre ellas se necesita un switch de capa 3 (por ejemplo el 3560 en Packet Tracer) con `ip routing`.

### Conexión con PuTTY

- **Serial (consola física):** cable de consola al PC, puerto **COMx**, **9600** baudios, 8 bits de datos, 1 de parada, sin paridad y sin control de flujo.
- **SSH (remoto):** puerto 22, tráfico cifrado, requiere una IP de administración.
- **Telnet:** remoto pero sin cifrar; no se recomienda.

### Configuración

```
enable
configure terminal
hostname SW-Compensar
service password-encryption
banner motd # Acceso solo autorizado #

line console 0
 password cisco123
 login
 logging synchronous
 exec-timeout 5 0
 exit

ip routing

vlan 10
 name VLAN10
vlan 20
 name VLAN20
vlan 30
 name VLAN30
exit

interface vlan 10
 ip address 192.168.10.1 255.255.255.0
 no shutdown
interface vlan 20
 ip address 172.16.0.1 255.255.0.0
 no shutdown
interface vlan 30
 ip address 10.0.0.1 255.0.0.0
 no shutdown
exit

interface fastEthernet 0/1
 switchport mode access
 switchport access vlan 10
interface fastEthernet 0/2
 switchport mode access
 switchport access vlan 20
interface fastEthernet 0/3
 switchport mode access
 switchport access vlan 30
end

copy running-config startup-config
```

### NVRAM y dirección MAC

```
show startup-config       ! configuración guardada en NVRAM
show running-config       ! configuración activa en RAM
show version              ! modelo, IOS, memoria, MAC base, número de serie
show interfaces vlan 10   ! MAC de la interfaz ("address is ...")
show mac address-table
show vlan brief
show ip interface brief
```

**Significado de la MAC:** dirección física de 48 bits (12 dígitos hexadecimales). Los primeros 24 bits (OUI) identifican al fabricante y los últimos 24 bits son el identificador único del equipo.

### Pruebas

```
PC1> ping 192.168.10.1
PC1> ping 172.16.0.10
```

![Verificación](03-switch/capturas/verificacion.png)

## Punto 3.2: Router y dos switches

**Objetivo:** configurar un router básico y dos switches, cada uno con una VLAN y un PC. El PC superior administra el router por consola.

### Direccionamiento

| Segmento | VLAN | Red | Gateway (router) | PC | IP del switch |
|---|---|---|---|---|---|
| Switch 1 | 10 | 192.168.10.0/24 | 192.168.10.1 (Gi0/0) | 192.168.10.10 | 192.168.10.2 |
| Switch 2 | 20 | 192.168.20.0/24 | 192.168.20.1 (Gi0/1) | 192.168.20.10 | 192.168.20.2 |

![Topología router](04-router/capturas/topologia.png)

### Router

```
enable
configure terminal
hostname R-Compensar
service password-encryption
banner motd # Acceso solo autorizado #
line console 0
 password cisco123
 login
 logging synchronous
 exit

interface gigabitEthernet 0/0
 description Hacia SW1 - VLAN 10
 ip address 192.168.10.1 255.255.255.0
 no shutdown
interface gigabitEthernet 0/1
 description Hacia SW2 - VLAN 20
 ip address 192.168.20.1 255.255.255.0
 no shutdown
end
copy running-config startup-config
```

### Switch 1 (el Switch 2 es igual con VLAN 20, IP 192.168.20.2 y gateway 192.168.20.1)

```
enable
configure terminal
hostname SW1
vlan 10
 name VLAN10
exit
interface fastEthernet 0/1
 switchport mode access
 switchport access vlan 10
interface fastEthernet 0/24
 switchport mode access
 switchport access vlan 10
exit
interface vlan 10
 ip address 192.168.10.2 255.255.255.0
 no shutdown
exit
ip default-gateway 192.168.10.1
end
copy running-config startup-config
```

### Alternativa: router-on-a-stick (un solo enlace troncal)

```
! Router
interface gigabitEthernet 0/0
 no shutdown
interface gigabitEthernet 0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
interface gigabitEthernet 0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0

! Switch, puerto hacia el router
interface gigabitEthernet 0/1
 switchport mode trunk
```

### Verificación

```
show startup-config
show version
show interfaces gigabitEthernet 0/0     ! MAC del router
show ip interface brief
show ip route
```

Prueba: `ping` de PC1 a PC2 debe responder gracias al router.

![Verificación router](04-router/capturas/verificacion.png)

## Punto 3.3: Máquinas virtuales

### ¿Qué es una máquina virtual?

Un computador simulado por software que corre sobre un equipo físico mediante un hipervisor, con su propio sistema operativo, CPU, memoria, disco y red virtuales. Características: aislamiento, portabilidad, instantáneas (snapshots), uso compartido del hardware y capacidad de ejecutar sistemas operativos distintos al anfitrión.

### Ubuntu en VirtualBox

1. Instalar VirtualBox y descargar la ISO de Ubuntu Desktop.
2. **Nueva** → nombre "Ubuntu", seleccionar la ISO, tipo Linux / Ubuntu (64-bit).
3. Asignar 4 GB de RAM, 2 CPU y disco de 25 GB.
4. Iniciar e instalar con **Install Ubuntu**; crear usuario y contraseña.
5. Reiniciar, retirar la ISO e instalar Guest Additions (opcional).

![VirtualBox](05-maquinas-virtuales/capturas/virtualbox.png)

### Ubuntu en VMware

1. Instalar VMware Workstation Player (o Workstation).
2. **Create a New Virtual Machine** → **Installer disc image file (iso)**.
3. Indicar usuario y contraseña (Easy Install), nombre y ubicación.
4. Asignar 25 GB de disco y 4 GB de memoria; finalizar e iniciar.
5. VMware instala las VMware Tools al terminar.

![VMware](05-maquinas-virtuales/capturas/vmware.png)

### Ubuntu en QEMU

```bash
sudo apt install qemu-system-x86 qemu-utils
qemu-img create -f qcow2 ubuntu.qcow2 25G
qemu-system-x86_64 -enable-kvm -m 4096 -smp 2 \
  -hda ubuntu.qcow2 -cdrom ubuntu.iso -boot d -vga virtio
```

En Windows se omite `-enable-kvm` (se puede usar `-accel whpx`) y el rendimiento es menor. Tras instalar, se inicia sin el parámetro `-cdrom`.

![QEMU](05-maquinas-virtuales/capturas/qemu.png)

### Diferencias entre VirtualBox, VMware y QEMU

| Aspecto | VirtualBox | VMware | QEMU |
|---|---|---|---|
| Licencia | Código abierto, gratis | Comercial (Player gratis para uso personal) | Código abierto, gratis |
| Interfaz | Gráfica sencilla | Gráfica pulida | Principalmente línea de comandos |
| Rendimiento | Bueno | Muy bueno | Medio; casi nativo con KVM |
| Arquitecturas | x86/x64 | x86/x64 | Muchas (ARM, RISC-V, MIPS, etc.) |
| Uso típico | Estudio y pruebas | Entornos profesionales | Desarrollo, emulación y servidores con KVM |

### Diferencias entre Docker y máquinas virtuales

| Aspecto | Máquina virtual | Docker (contenedor) |
|---|---|---|
| Qué virtualiza | Hardware completo | El sistema operativo (aísla procesos) |
| Sistema operativo | Cada VM trae el suyo | Comparten el kernel del anfitrión |
| Tamaño | GB | MB |
| Arranque | Minutos | Segundos |
| Aislamiento | Más fuerte | Más ligero |
| Uso típico | Correr otro SO completo | Empaquetar y desplegar aplicaciones |

## Simulación en Python

Archivo `python/simulacion_red.py`: verifica qué equipos se comunican directamente (misma red) y cuáles necesitan el router.

```python
import ipaddress

equipos = {
    "PC1": "192.168.10.10/24",
    "PC2": "192.168.20.10/24",
    "Router-Gi0/0": "192.168.10.1/24",
    "Router-Gi0/1": "192.168.20.1/24",
}

def misma_red(a, b):
    return ipaddress.ip_interface(a).network == ipaddress.ip_interface(b).network

def ping(origen, destino):
    a, b = equipos[origen], equipos[destino]
    if misma_red(a, b):
        return f"{origen} -> {destino}: directo por el switch (misma red)"
    gw = [n for n, ip in equipos.items() if n.startswith("Router") and misma_red(a, ip)]
    if gw:
        return f"{origen} -> {destino}: enviado al gateway {gw[0]}, que enruta"
    return f"{origen} -> {destino}: sin ruta"

print(ping("PC1", "Router-Gi0/0"))
print(ping("PC1", "PC2"))
```

## Conclusiones

- Mantener los dispositivos actualizados y con antivirus activo reduce de forma importante el riesgo de ataques.
- Un switch de capa 3 permite crear varias interfaces VLAN con IP y enrutar entre ellas; uno de capa 2 solo ofrece una IP de gestión.
- Un router conecta redes distintas; sin él, los equipos de VLAN diferentes no se comunican.
- Las máquinas virtuales dan aislamiento y flexibilidad para probar sistemas completos, mientras que Docker es más liviano para desplegar aplicaciones.

## Cómo publicar este repositorio

```bash
git init
git add .
git commit -m "Actividad 6 - redes y virtualización"
git branch -M main
git remote add origin https://github.com/TU_USUARIO/actividad6-redes-virtualizacion.git
git push -u origin main
```
