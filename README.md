# ÁvilaOS – Simulador de un sistema operativo distribuido

Proyecto 1 de Sistemas Operativos (2627-1). Simula un clúster de 2 a 4
computadores, cada uno con su CPU, su RAM y su propio núcleo (planificador y
colas), sincronizados por un reloj global. Los procesos productores y
consumidores se comunican por buffers de capacidad limitada sincronizados con
semáforos, en el mismo computador o en otro (con latencia de red).

## Requisitos

- **Java 21 o superior** (el proyecto compila con nivel 21 y se probó con JDK 27).
- **NetBeans** (el proyecto es un proyecto Ant de NetBeans).
- Librerías incluidas en `lib/` (no hay que descargar nada):
  - `gson-2.14.0.jar`: leer y guardar la configuración en JSON.
  - `jfreechart-1.5.6.jar`: gráfica de utilización de CPU.

## Cómo abrirlo y ejecutarlo

1. Clonar el repositorio y, en NetBeans, `File > Open Project` y elegir la
   carpeta `AvilaOS_Proyecto1_SO`.
2. Si NetBeans pregunta por la plataforma Java, elegir un JDK 21 o superior
   (`Tools > Java Platforms`). Las dos librerías ya están en el classpath.
3. `Run > Run Project` (F6). La clase principal es `avilaos.AvilaOS`.
4. Aparece la **vista de configuración**: número de computadores (2-4),
   duración del ciclo, latencia de red y, por computador, RAM, política inicial
   y quantum. También se puede **cargar un archivo JSON** (por ejemplo
   `config/ejemplo.json`, que trae buffers y procesos productores/consumidores
   locales y remotos).
5. En la ventana principal: **Iniciar** arranca el reloj.

## Qué se ve en la ventana

- Por computador: proceso en CPU (PC, prioridad, deadline), modo (usuario /
  sistema operativo), colas de nuevos, listos, bloqueados (con el motivo) y
  terminados, ocupación de la RAM, política en uso con selector y quantum.
  Seleccionar un proceso y pulsar *Ver detalles del PCB* muestra su PCB.
- Global: ciclo de reloj, panel de buffers (elementos, capacidad, valores de
  los semáforos `s`, `n`, `e` y procesos esperando) y log de eventos.
- Botones: **Crear proceso**, **Crear buffer**, **Ver métricas** (por
  computador, del sistema completo y de productor-consumidor),
  **Ver gráficas** (utilización de CPU de todos los computadores en un mismo
  gráfico), **Guardar configuración** y la duración del ciclo en ejecución.

Todos los campos de entrada validan tipo y rango; un dato inválido muestra un
mensaje y no detiene la simulación.

## Archivo de configuración (JSON)

*Guardar configuración* escribe computadores (RAM, política, quantum),
latencia, duración del ciclo, buffers (capacidad y anfitrión) y todos los
procesos con sus atributos iniciales. Se carga desde la vista de configuración.

```json
{
  "cycleDurationMs": 300,
  "networkLatency": 3,
  "computers": [ { "ramMb": 1024, "policy": "ROUND_ROBIN", "quantum": 4 },
                 { "ramMb": 1024, "policy": "EDF", "quantum": 4 } ],
  "buffers":   [ { "capacity": 3, "host": 1 } ],
  "processes": [ { "name": "Prod", "computer": 1, "type": "PRODUCER", "priority": 2,
                   "memory": 100, "deadline": 300, "buffer": 1,
                   "cyclesPerItem": 2, "items": 6 } ]
}
```

- `policy`: `FCFS`, `ROUND_ROBIN`, `PRIORITY_PREEMPTIVE`, `EDF`.
- `type`: `CPU_BOUND`, `IO_BOUND`, `PRODUCER`, `CONSUMER`.
- `buffer` es la posición (desde 1) del buffer en la lista `buffers`.
- Rangos válidos: ver `DECISIONES.md`, sección 5.

## Estructura del código

| Paquete | Contenido |
| --- | --- |
| `avilaos.hardware` | Reloj global, CPU, `Computer`, `Cluster`, métricas |
| `avilaos.kernel` | `Kernel` (colas y admisión por memoria) |
| `avilaos.kernel.process` | PCB, enums de estado/tipo/bloqueo, modelo de instrucciones |
| `avilaos.kernel.scheduler` | Interfaz `SchedulingPolicy` y las 4 políticas |
| `avilaos.kernel.memory` | Control de RAM usada y libre |
| `avilaos.kernel.sync` | Semáforos, `Buffer`, `SyncExecutor` (productor-consumidor) |
| `avilaos.structures` | `LinkedQueue`: estructura propia (no se usan colecciones de Java) |
| `avilaos.config` | Configuración JSON y fábrica de la simulación |
| `avilaos.gui` | Ventanas: solo observan al sistema, no contienen lógica del SO |

Las reglas que el enunciado deja abiertas (prioridades, deadline, bloqueos,
métricas, valores por defecto) están en [`DECISIONES.md`](DECISIONES.md).

## Pruebas

Las clases `*Prueba.java` son programas con `main` que se ejecutan desde
NetBeans (clic derecho > *Run File*): semáforos, buffer, `SyncExecutor`,
latencia remota, memoria, cola enlazada y escenario local/remoto.

## Autores

Proyecto desarrollado en equipo para Sistemas Operativos (2627-1, Universidad
Metropolitana):

- **Alexandra Loynaz** ([@Aleloynazg](https://github.com/Aleloynazg))
- **Julieta Thery** ([@julietathery](https://github.com/julietathery))
- **Carlos** ([@CarlosEGonR](https://github.com/CarlosEGonR))

Este repositorio es una copia del repositorio del equipo
([Aleloynazg/AvilaOS_Proyecto1_SO](https://github.com/Aleloynazg/AvilaOS_Proyecto1_SO)).
El historial de commits conserva la autoría de cada integrante.
