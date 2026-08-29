# Solución para la gestión de paquetes

## Problema identificado

InnovaCloud Solutions instala software manualmente en sus servidores y máquinas virtuales. Este método favorece diferencias entre versiones, errores al resolver dependencias y descargas repetidas de los mismos paquetes, con consumo innecesario de ancho de banda.

## Análisis

En Ubuntu, APT permite instalar, actualizar y eliminar paquetes junto con sus dependencias. `apt-get` ofrece operaciones equivalentes desde la línea de comandos y trabaja con `dpkg`, el gestor de bajo nivel para paquetes `.deb`. El material también presenta `wget` para descargar recursos desde una URL; ese flujo mantiene su utilidad para casos puntuales, pero no reemplaza la administración centralizada propuesta con APT.

La administración centralizada requiere que los equipos consulten una fuente común y actualizada. El archivo principal de fuentes es `/etc/apt/sources.list`; el directorio `/etc/apt/sources.list.d/` permite agregar fuentes mediante archivos `.list` independientes.

Cada línea tradicional de repositorio sigue esta estructura:

```text
[tipo] [url] [distribución] [secciones]
```

- `deb` identifica paquetes binarios instalables.
- `deb-src` identifica el código fuente de los paquetes.
- La URL indica el servidor o mirror.
- `jammy` es un ejemplo de nombre de distribución.
- `main`, `restricted`, `universe` y `multiverse` son secciones del repositorio.

## ¿Qué es un repositorio mirror?

Un mirror es una copia de un repositorio oficial de software. Se sincroniza periódicamente con la fuente oficial para ofrecer paquetes desde un servidor alternativo. Cuando el mirror está próximo o dentro de la infraestructura, reduce la latencia y evita que cada equipo descargue repetidamente desde Internet.

Para InnovaCloud Solutions, sus aportes serían:

- menor consumo del enlace externo;
- descargas más rápidas dentro de la red local;
- mayor consistencia al utilizar una fuente común;
- mejor disponibilidad si la fuente oficial presenta problemas de acceso, siempre que el mirror esté operativo y sincronizado.

## Solución propuesta

Se propone centralizar las descargas mediante un mirror accesible dentro de la infraestructura de InnovaCloud Solutions. Los servidores y las máquinas virtuales utilizarían ese origen común mediante APT.

La propuesta depende de una sincronización periódica del mirror. La consistencia se obtendría al mantener esa sincronización y al configurar los clientes para consultar la misma fuente.

## Configuración del repositorio

La dirección real del mirror no fue proporcionada. Por ello se utiliza el marcador explícito `http://<SERVIDOR_MIRROR>/ubuntu`; no representa una dirección operativa.

`<SERVIDOR_MIRROR>` debe sustituirse por la IP o el nombre DNS real del servidor configurado por InnovaCloud Solutions.

Para editar el archivo principal de fuentes:

```bash
sudo nano /etc/apt/sources.list
```

La línea conceptual sería:

```text
deb http://<SERVIDOR_MIRROR>/ubuntu jammy main restricted universe multiverse
```

Si también se requiriera código fuente, la misma estructura permite el tipo `deb-src`:

```text
deb-src http://<SERVIDOR_MIRROR>/ubuntu jammy main restricted universe multiverse
```

Antes de aplicar la configuración se deben validar la distribución del sistema, las secciones disponibles, la dirección real del mirror y su acceso desde la red corporativa.

## Administración con APT

Después de configurar y validar la fuente, se actualizaría el índice local de paquetes:

```bash
sudo apt-get update
```

La instalación de un paquete se realizaría con:

```bash
sudo apt install <nombre_del_paquete>
```

`<nombre_del_paquete>` es un marcador que debe sustituirse por el nombre real del software autorizado. Al resolver las dependencias desde el repositorio configurado, se reduce la necesidad de descargar e instalar manualmente archivos individuales.

Cuando se disponga de un paquete `.deb` previamente obtenido y validado, `dpkg` permite instalarlo directamente:

```bash
sudo dpkg -i <nombre_del_paquete>.deb
```

Para la operación ordinaria se prioriza APT, porque consulta repositorios y gestiona dependencias.

## Verificación

Para comprobar los paquetes registrados como instalados:

```bash
apt list --installed
```

Una segunda vista del inventario se obtiene con:

```bash
dpkg --get-selections
```

En un escenario de implementación también se verificaría que `sudo apt-get update` consulte el mirror real sin errores. La revisión del inventario permitiría comparar los equipos y detectar diferencias de software.

## Beneficios para InnovaCloud Solutions

- Menor cantidad de instalaciones manuales y menor riesgo de errores de dependencias.
- Reducción de descargas duplicadas y del consumo de ancho de banda externo.
- Fuente común para mejorar la consistencia entre servidores y máquinas virtuales.
- Inventario verificable mediante APT y `dpkg`.
- Mayor control sobre el origen utilizado para obtener paquetes.
