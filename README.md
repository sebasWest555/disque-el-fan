# Simulador de Aire Acondicionado

## 1. ¿Cómo funciona un aire acondicionado real?

Un aire acondicionado ayuda a mantener una habitación a una temperatura cómoda. No crea frío de la nada: **extrae calor del interior y lo expulsa al exterior**.

Sus componentes principales son:

- **Evaporador:** está en la unidad interior. El refrigerante absorbe calor del aire de la habitación.
- **Compresor:** hace circular el refrigerante y aumenta su presión y temperatura.
- **Condensador:** normalmente está en la unidad exterior. El refrigerante libera al exterior el calor que recogió dentro.
- **Válvula de expansión:** reduce la presión del refrigerante para que pueda volver a absorber calor.
- **Ventiladores:** mueven el aire a través de los intercambiadores de calor.
- **Sensor y termostato:** miden la temperatura y ayudan a decidir cuándo enfriar.

El refrigerante circula por un circuito cerrado y cambia de presión y estado durante el ciclo. De forma simplificada, el proceso es:

1. El refrigerante absorbe calor de la habitación en el evaporador.
2. El compresor impulsa el refrigerante por el circuito.
3. El refrigerante libera ese calor al exterior en el condensador.
4. La válvula de expansión reduce su presión y el ciclo vuelve a empezar.

### ¿Qué es la histéresis?

El termostato no necesita encender y apagar el compresor cada vez que la temperatura cambia una fracción de grado. Para evitar cambios constantes, se utilizan dos umbrales:

- **Umbral superior:** cuando la temperatura llega a este valor, el sistema empieza a enfriar.
- **Umbral inferior:** cuando la temperatura baja hasta este valor, el sistema detiene el enfriamiento.
- **Entre ambos umbrales:** el sistema conserva su estado anterior. Si estaba enfriando, continúa; si estaba apagado, permanece apagado.

En la configuración inicial documentada de este proyecto, la consigna es de **25.0 °C** y la diferencia de histéresis es de **±0.8 °C**. Esto da un umbral superior de **25.8 °C** y uno inferior de **24.2 °C**. Si los valores se cambian en el código, los umbrales también pueden cambiar.

### ¿Qué significa Inverter?

Un equipo tradicional puede trabajar principalmente encendido o apagado. Un equipo con tecnología **Inverter** puede variar la potencia del compresor según la necesidad de enfriamiento:

- Si la habitación está muy caliente, utiliza más potencia.
- Al acercarse a la temperatura objetivo, puede reducir la potencia.
- Si la temperatura llega al umbral inferior configurado, el controlador detiene el enfriamiento según la lógica de histéresis.

La histéresis decide cuándo se permite enfriar; la modulación Inverter regula la intensidad del enfriamiento mientras el sistema está activo.

## 2. ¿Cómo funciona el aire acondicionado simulado?

Este programa es una **simulación educativa por software**. No controla necesariamente un aire acondicionado físico ni reproduce todos los detalles de un circuito real. Usa clases de Python para representar los componentes y calcular cómo podría cambiar la temperatura de una habitación.

En términos generales, el ciclo es:

1. El sensor simulado entrega una lectura de temperatura.
2. El controlador compara esa lectura con los umbrales de histéresis.
3. Si corresponde enfriar, el controlador determina la capacidad del compresor y la velocidad objetivo del ventilador.
4. El modelo del ambiente calcula las ganancias de calor y el efecto del enfriamiento para estimar la temperatura siguiente.
5. El simulador coordina los pasos y puede mostrar datos en consola o mediante la interfaz gráfica.
6. El sistema revisa condiciones anormales y puede mostrar advertencias o alertas.

### Factores que influyen en la temperatura

El modelo del ambiente contempla factores como:

- La temperatura exterior y el intercambio de calor con el recinto.
- El tamaño de la habitación.
- El calor producido por las personas. En el modelo documentado se considera aproximadamente **110 W por persona**.
- La apertura de puertas o ventanas, que puede permitir la entrada de aire exterior.
- Cargas adicionales, como equipos informáticos o servidores en el escenario de centro de datos.
- La capacidad de enfriamiento que se atribuye al compresor simulado.

Por eso, el sistema podría tardar en enfriar o incluso no alcanzar la temperatura esperada si las ganancias de calor modeladas son altas frente a la capacidad de enfriamiento.

### Histéresis y modulación en esta simulación

Con los valores iniciales documentados:

| Condición | Comportamiento esperado |
|---|---|
| Temperatura igual o superior a 25.8 °C | Se activa el enfriamiento. |
| Temperatura entre 24.2 °C y 25.8 °C | Se conserva el estado previo del sistema. |
| Temperatura igual o inferior a 24.2 °C | Se detiene el enfriamiento por histéresis. |

**Importante:** los umbrales efectivos dependen de los valores que estén definidos en `controlador.py` y de cómo se construya el controlador. Si se modifica la configuración, actualiza esta tabla para que coincida con el código que se ejecuta.

## 3. Cómo ejecutar el programa

Abre una terminal (por ejemplo, PowerShell) en la carpeta del proyecto. Los comandos siguientes deben ejecutarse desde esa carpeta.

### Interfaz gráfica

```powershell
python gui.py
```

También se documenta esta alternativa:

```powershell
python main.py --gui
```

La interfaz gráfica permite observar la temperatura, el estado del sistema y los controles de simulación que estén implementados en la versión del proyecto.

### Menú de consola

```powershell
python main.py
```

### Ejecutar un escenario concreto

El proyecto documenta seis escenarios preconfigurados:

```powershell
python main.py --scenario 1
python main.py --scenario 2
python main.py --scenario 3
python main.py --scenario 4
python main.py --scenario 5
python main.py --scenario 6
```

Estos escenarios representan distintos casos de funcionamiento y fallas, como operación normal, ventilador lento, fuga de refrigerante, falla del sensor, puerta abierta y cargas elevadas. Para ver las opciones exactas de cada escenario, consulta `escenarios.py` y la ayuda de `main.py`.

Si la versión del programa admite el modo rápido descrito en el README anterior, puedes agregar `--fast` al comando del escenario para reducir o eliminar las pausas de visualización.

## 4. Cómo ejecutar las pruebas

Para ejecutar las pruebas automatizadas:

```powershell
python test_sistema.py
```

También está documentada esta alternativa:

```powershell
python main.py --test
```

Las pruebas ayudan a comprobar que las partes principales se comporten como se espera y a detectar regresiones cuando se modifica el código. Si una prueba falla, revisa el mensaje que aparece en la terminal: normalmente indica qué prueba falló y qué resultado se esperaba.

## 5. ¿Qué hace cada módulo?

### `controlador.py` — El cerebro del sistema

Este módulo contiene la lógica de decisión del aire acondicionado. Recibe los componentes necesarios mediante el constructor, por ejemplo el sensor de temperatura, el ventilador y el compresor.

Sus responsabilidades principales son:

- Leer la temperatura disponible a través del sensor.
- Comparar la temperatura con los umbrales de histéresis.
- Decidir si el enfriamiento debe estar activo o en reposo.
- Ajustar la capacidad del compresor y la velocidad objetivo del ventilador.
- Revisar condiciones anormales y generar diagnósticos o alertas cuando corresponda.

En pocas palabras: **el controlador decide qué debe hacer el sistema**; no es el módulo que calcula por sí solo toda la evolución térmica de la habitación.

### `ambiente.py` — El modelo de la habitación

Este módulo representa el recinto que se desea enfriar. Calcula cómo cambian las condiciones térmicas a lo largo de la simulación.

Entre los factores considerados están el volumen del cuarto, la temperatura exterior, el calor de las personas, las entradas de aire por puertas o ventanas y otras cargas térmicas configuradas.

En pocas palabras: **el ambiente calcula qué le pasa a la habitación** cuando gana calor y cuando el aire acondicionado extrae calor.

### `dispositivos.py` — Los sensores y actuadores

Este módulo define las interfaces o contratos que deben cumplir los componentes del sistema, además de las implementaciones simuladas descritas por el proyecto.

Entre los componentes están:

- **Sensor de temperatura:** proporciona la lectura de temperatura.
- **Ventilador:** recibe una velocidad objetivo y permite consultar sus revoluciones cuando la implementación lo admite.
- **Compresor:** recibe una capacidad de trabajo y puede proporcionar datos como la presión simulada del refrigerante.
- **Sensor de apertura:** indica si una puerta o ventana está abierta, cuando se usa.

Separar estos dispositivos del controlador permite sustituir una implementación simulada por otra que cumpla las mismas interfaces, sin tener que reescribir toda la lógica de control.

En pocas palabras: **este módulo representa con qué componentes se comunica el controlador y qué operaciones puede pedirles**.

### `simulador.py` — El coordinador de los pasos

Este módulo integra el controlador, los dispositivos y el ambiente durante la ejecución. Avanza la simulación paso a paso y puede presentar en consola información de telemetría, como temperaturas y estados de los componentes.

En pocas palabras: **el simulador hace avanzar el tiempo de la simulación y conecta las partes**.

### `escenarios.py` — Casos de prueba preparados

Contiene configuraciones de escenarios para probar diferentes situaciones, por ejemplo el funcionamiento normal o condiciones de falla y sobrecarga. Así se pueden repetir casos sin configurar todo manualmente cada vez.

### `test_sistema.py` — Pruebas automatizadas

Contiene pruebas que verifican comportamientos del sistema. Sirve para comprobar que los cambios de código no rompan funciones que ya trabajaban correctamente.

### `main.py` — Punto de entrada por consola

Permite iniciar el programa, elegir o ejecutar escenarios y acceder a las opciones de consola documentadas por el proyecto.

### `gui.py` — Interfaz gráfica

Muestra una representación visual de la simulación y ofrece controles para observar o modificar los parámetros que estén implementados en la interfaz.

## 6. Cómo se relacionan los módulos

Una forma sencilla de entender el proyecto es seguir este flujo:

```text
Sensor de temperatura
        |
        v
   controlador.py  ------>  dispositivos.py
        |                  (compresor y ventilador)
        v
   simulador.py
        |
        v
    ambiente.py
        |
        v
Temperatura del siguiente paso
        |
        +----> nueva lectura para el controlador
```

`main.py` o `gui.py` inicia y presenta la simulación; `escenarios.py` prepara casos concretos y `test_sistema.py` comprueba comportamientos.
