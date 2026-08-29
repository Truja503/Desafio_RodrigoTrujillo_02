# Verificación y diagnóstico de red

**Autor:** Rodrigo Trujillo

**Rol:** Responsable de diagnóstico de red

## Problema identificado

InnovaCloud Solutions no dispone de una secuencia común para comprobar interfaces, direccionamiento, rutas y conectividad. Sin un orden definido, una falla local puede confundirse con un problema del gateway o del destino remoto.

## Procedimiento estandarizado

La revisión propuesta avanza desde los componentes locales hasta el destino remoto. Cada paso depende del anterior: primero se confirma la interfaz, luego su dirección, la ruta de salida, la accesibilidad y finalmente el trayecto de los paquetes.

### 1. Identificación de interfaces

```bash
ip link
```

Este comando permite identificar las interfaces presentes, sus nombres y su estado. La interfaz que se configurará o diagnosticará debe aparecer y encontrarse activa antes de continuar.

### 2. Verificación de direccionamiento

```bash
ip addr
```

Permite comprobar las direcciones asociadas a cada interfaz. Se validaría que la VM tenga la dirección, el prefijo y la interfaz previstos en la configuración de Netplan, y que no se esté evaluando una interfaz distinta.

### 3. Tabla de enrutamiento

```bash
ip route
```

Muestra las rutas conocidas por el sistema. Se revisaría especialmente que exista una ruta hacia la red de destino o una ruta predeterminada con el gateway correcto.

### 4. Prueba de conectividad

```bash
ping <IP_DESTINO>
```

`ping` utiliza ICMP para comprobar si el destino responde y permite observar pérdida de paquetes y latencia. `<IP_DESTINO>` es un marcador que debe reemplazarse por una dirección autorizada: primero podría probarse un equipo de la red local o el gateway y después un destino remoto.

Una falta de respuesta no identifica por sí sola el punto exacto de la falla; debe interpretarse junto con la interfaz, el direccionamiento y las rutas.

### 5. Rastreo de ruta

```bash
traceroute <IP_DESTINO>
```

o, como alternativa enseñada en el material:

```bash
tracepath <IP_DESTINO>
```

Estos comandos muestran los nodos intermedios observados hasta un destino. Se utilizarían cuando la conectividad falla o presenta retrasos, con el fin de ubicar hasta qué punto avanza el tráfico.

### 6. Puertos y servicios

Para identificar los puertos TCP y UDP que permanecen en escucha en el servidor, junto con el proceso asociado, se utilizaría:

```bash
sudo ss -tulpn
```

Las opciones indican sockets TCP (`-t`), UDP (`-u`), en escucha (`-l`), procesos asociados (`-p`) y valores numéricos para direcciones y puertos (`-n`). La salida permitiría relacionar cada puerto local con el proceso que lo mantiene abierto.

Para auditar los servicios administrados por `systemd` que están en ejecución:

```bash
systemctl --type=service --state=running
```

El listado se revisaría para confirmar que cada servicio activo sea necesario para la función del servidor. Un puerto inesperado o un servicio no reconocido requeriría validación antes de modificar su estado.

## Secuencia recomendada de diagnóstico

La ejecución ordenada sería:

```bash
ip link
ip addr
ip route
ping <IP_DESTINO>
traceroute <IP_DESTINO>
tracepath <IP_DESTINO>
sudo ss -tulpn
systemctl --type=service --state=running
```

1. Si la interfaz no aparece o está inactiva, se atiende primero el adaptador y su configuración.
2. Si no existe una dirección correcta, se revisa el archivo de Netplan.
3. Si falta una ruta válida, se comprueba el gateway configurado.
4. Si la configuración local es correcta, `ping` permite probar accesibilidad.
5. Si el destino no responde o existe una ruta problemática, `traceroute` o `tracepath` permiten examinar el trayecto.
6. Finalmente, `ss` permite registrar los puertos locales en escucha y `systemctl` los servicios activos.

No es necesario ejecutar ambos comandos de rastreo en todos los casos; se presentan como alternativas disponibles en el material.

## Resultado esperado

El procedimiento permite registrar, en orden, el nombre y estado de la interfaz, su direccionamiento, las rutas disponibles, la respuesta del destino, el punto hasta el cual avanza el tráfico, los puertos locales en escucha y los servicios activos. Con esta evidencia, el equipo puede distinguir una falla de interfaz, configuración IP, enrutamiento o comunicación remota y detectar servicios que necesiten revisión.
