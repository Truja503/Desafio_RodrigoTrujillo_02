# Solución para la configuración de red

**Autor:** Rodrigo Trujillo

**Rol:** Responsable de redes y conectividad

## Problema identificado

Las máquinas virtuales de InnovaCloud Solutions utilizan el modo NAT predeterminado de VirtualBox. Aunque pueden salir a Internet mediante la dirección del anfitrión, quedan ocultas detrás de este y no tienen la visibilidad directa que requieren los servidores y estaciones de desarrollo dentro de la red corporativa.

## Limitaciones del modo NAT

NAT es sencillo de configurar y adecuado cuando la máquina virtual solo necesita acceso saliente. Sin embargo, la traducción de direcciones y las restricciones de puertos dificultan que otros equipos corporativos inicien comunicación directa con la máquina virtual. Esta limitación coincide con el problema descrito por el cliente.

## Comparación de modos de VirtualBox

| Modo | Características | Ventajas | Limitaciones | Uso |
|---|---|---|---|---|
| NAT | La VM usa la dirección del anfitrión para salir a Internet y permanece oculta detrás de él. | Configuración sencilla y aislamiento respecto de conexiones entrantes directas. | Restricciones de puertos y menor visibilidad desde la red física. | Equipos que requieren principalmente acceso saliente. |
| Puente | Conecta la VM directamente a la red física; se comporta como un dispositivo independiente y obtiene su propia IP. | Visibilidad directa, flexibilidad y comunicación con recursos de la misma red. | Requiere seleccionar correctamente el adaptador físico y aplicar la configuración de red corporativa. | Servidores y estaciones que deben ser accesibles en la red. |
| Red Interna | Crea un segmento privado donde se comunican únicamente las VMs del mismo segmento. | Aislamiento apropiado para pruebas y laboratorios. | No accede a la red física ni a Internet salvo que se configure un gateway. | Laboratorios aislados y pruebas entre varias VMs. |

## Solución propuesta: Adaptador Puente

Se propone configurar las máquinas virtuales del entorno de desarrollo con Adaptador Puente. De esta forma, cada VM se integraría a la red corporativa como un dispositivo independiente, podría recibir o configurar su propia dirección IP y tendría comunicación directa con el mirror y los demás recursos autorizados.

En un escenario de implementación, la VM se apagaría, se cambiaría su adaptador de red en VirtualBox al modo **Adaptador Puente** y se seleccionaría la interfaz física correcta del anfitrión. Después se iniciaría la VM y se comprobaría su interfaz Linux antes de asignar una dirección.

Red Interna no es la propuesta principal porque aislaría las VMs de la red física, y NAT mantendría la limitación que originó el caso.

## Identificación de la interfaz Linux

Antes de editar Netplan se debe identificar el nombre real de la interfaz y su estado:

```bash
ip link
```

Linux puede utilizar nombres como `enp0s3` o `eth0`. El nombre observado debe emplearse en el archivo YAML; no se debe asumir que todas las máquinas utilizan `enp0s3`.

Para conocer los archivos de configuración existentes:

```bash
ls /etc/netplan
```

## Configuración de IP estática mediante Netplan

Una dirección estática resulta apropiada para un servidor porque mantiene una ubicación predecible. El material de la semana 7 presenta `enp0s3`, `192.168.1.100/24`, `192.168.1.1`, `8.8.8.8` y `8.8.4.4` como ejemplo académico. Estos valores **no representan la red real de InnovaCloud Solutions** y deben sustituirse por los datos autorizados de su infraestructura.

Si el archivo identificado fuera `01-netcfg.yaml`, se editaría con:

```bash
sudo nano /etc/netplan/01-netcfg.yaml
```

La configuración de ejemplo, con indentación YAML, sería:

```yaml
network:
  version: 2
  ethernets:
    enp0s3:
      addresses:
        - 192.168.1.100/24
      gateway4: 192.168.1.1
      nameservers:
        addresses:
          - 8.8.8.8
          - 8.8.4.4
```

Antes de aplicarla se reemplazarían el nombre de interfaz, la dirección, el prefijo, el gateway y los DNS según la red real. También se comprobaría que la dirección seleccionada no esté asignada a otro equipo.

## Aplicación y verificación

Después de guardar un archivo validado, la configuración se aplicaría con:

```bash
sudo netplan apply
```

La dirección asignada se comprobaría con:

```bash
ip addr
```

La salida esperada mostraría la interfaz correcta en estado activo y la dirección estática autorizada. Las pruebas de ruta y conectividad se detallan en [Verificación y diagnóstico de red](./diagnostics.md).

## Beneficios para InnovaCloud Solutions

- Comunicación directa entre las VMs y los recursos permitidos de la red corporativa.
- Dirección propia y predecible para cada servidor configurado de forma estática.
- Acceso local al repositorio mirror sin depender de traducciones de NAT.
- Configuración de red documentada y repetible mediante Netplan.
- Mayor facilidad para aplicar el procedimiento estandarizado de diagnóstico.
