# Solución de almacenamiento

**Autor:** Rodrigo Trujillo

**Rol:** Especialista de almacenamiento

## Problema identificado

El servidor principal de InnovaCloud Solutions presenta fallos de disco y opera sin redundancia. Una falla de la unidad que contiene los datos podría interrumpir el servicio y ocasionar pérdida de información.

## Análisis

RAID combina varias unidades físicas para que el sistema las utilice como una unidad lógica. Según el material de la semana 5, el nivel elegido debe responder a las necesidades de almacenamiento, rendimiento, continuidad, presupuesto y complejidad de administración.

La prioridad del caso es mantener la información disponible cuando falle un disco. Por esa razón, un arreglo sin contingencia no resuelve el problema, aunque ofrezca más capacidad o velocidad.

## RAID evaluados

| Nivel | Funcionamiento resumido | Contingencia | Consideración para el caso |
|---|---|---|---|
| RAID 0 | Distribuye los datos y suma la capacidad de los discos. | No posee contingencia; la falla de una unidad implica la pérdida del arreglo. | Se descarta porque no protege la continuidad del servidor. |
| RAID 1 | Mantiene una copia espejo de la información en las unidades. | Permite reemplazar el disco dañado sin perder la información replicada. | Es la alternativa seleccionada por su redundancia directa. |
| RAID 5 | Distribuye datos y redundancia entre varias unidades. | Tolera la falla de un disco, pero su reconstrucción es más lenta. | Aporta mejor aprovechamiento de capacidad, con mayor complejidad que la requerida para este escenario. |

## Solución propuesta: RAID 1

Se propone RAID 1 para el servidor principal. El arreglo replica la misma información en dos discos; si uno se daña, el otro conserva la copia y permite mantener la disponibilidad mientras se reemplaza la unidad defectuosa.

En el escenario ilustrativo se utilizarían `/dev/sdb` y `/dev/sdc` como discos miembros y `/dev/md0` como dispositivo lógico. La capacidad útil correspondería aproximadamente a la de una sola unidad, porque la información se duplica.

**RAID proporciona redundancia, pero no sustituye una estrategia de copias de seguridad.**

## Implementación con mdadm

`mdadm` es la herramienta presentada en la semana 5 para crear, administrar y monitorear arreglos RAID en Ubuntu Server. Los siguientes comandos representan una implementación propuesta; no indican que el arreglo ya haya sido creado.

> **Advertencia:** se deben comprobar los nombres y el contenido de los discos antes de cualquier operación. Crear el arreglo sobre dispositivos equivocados puede destruir información. `/dev/sdb`, `/dev/sdc` y `/dev/md0` son nombres ilustrativos.

Primero se identificarían las unidades disponibles:

```bash
lsblk
```

Si `mdadm` no estuviera instalado, se actualizaría el índice de paquetes y se instalaría la herramienta:

```bash
sudo apt update
sudo apt install mdadm
```

Después de confirmar que `/dev/sdb` y `/dev/sdc` son las unidades correctas y que pueden destinarse al arreglo, la creación sería:

```bash
sudo mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sdb /dev/sdc
```

El comando define un arreglo RAID 1 llamado `/dev/md0` con dos discos miembros.

## Verificación

Para observar el estado general y el progreso de sincronización:

```bash
cat /proc/mdstat
```

Para mostrar el nivel, los discos miembros y el estado detallado del arreglo:

```bash
sudo mdadm --detail /dev/md0
```

La verificación esperada mostraría `/dev/md0` activo como RAID 1 y ambos discos en estado operativo. Cualquier disco ausente o degradado requeriría atención antes de considerar concluida la implementación.

## Beneficios para InnovaCloud Solutions

- Continuidad del servidor ante la falla de una de las dos unidades.
- Copia espejo automática de la información almacenada en el arreglo.
- Administración y monitoreo desde Ubuntu Server mediante `mdadm`.
- Reconstrucción directa al sustituir la unidad dañada.

## Limitaciones

- La duplicación reduce la capacidad útil respecto de la capacidad física total.
- Se requiere una unidad adicional y, por tanto, mayor inversión que con un solo disco.
- RAID 1 no protege contra borrados accidentales, corrupción replicada, errores administrativos ni incidentes que afecten ambas unidades.
- Debe mantenerse una estrategia independiente de copias de seguridad y revisarse periódicamente el estado del arreglo.
