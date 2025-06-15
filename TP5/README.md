
# TP 5 - Device Driver (CDD) para Sensado de Señales Externas

## Marco Teórico

### ¿Qué es un Controlador de Dispositivo (Device Driver)?

En sistemas operativos como Linux, un **controlador de dispositivo** es un módulo que permite la comunicación entre el kernel y un dispositivo físico o virtual. En el caso de un **Controlador de Dispositivo de Caracteres (CDD)**, la comunicación se realiza a través de un flujo secuencial de caracteres, como si se tratara de un archivo.

Los CDD se utilizan para representar sensores, periféricos o dispositivos de entrada/salida que manejan información en forma de bytes.

### Espacio de Usuario vs Espacio de Kernel

- **Espacio de kernel**: es donde se ejecutan los controladores y el núcleo del sistema. Tiene acceso directo al hardware y privilegios máximos.
- **Espacio de usuario**: es donde se ejecutan las aplicaciones del usuario. Tiene privilegios restringidos y accede al hardware solo a través del kernel.

### Señales digitales simuladas

Una **señal digital** puede tomar valores lógicos (0 o 1). En este proyecto se simulan dos señales periódicas:
- Una cambia de valor cada segundo.
- Otra oscila rápidamente, pero se evalúa solo una vez por segundo.

Estas señales simulan la lectura desde GPIOs reales, usando mecanismos internos del kernel (como `jiffies` y `timer_list`).

## Objetivo del Trabajo Práctico

Diseñar e implementar un **Controlador de Dispositivo de Caracteres (CDD)** en Linux que permita sensar dos señales digitales externas, graficando su evolución desde una aplicación de usuario.

La solución debe:

- Exponer un archivo en `/dev/` que permita leer señales.
- Permitir elegir entre dos señales mediante escritura al dispositivo.
- Tener una aplicación en Python que:
  - Seleccione la señal.
  - Lea su valor cada segundo.
  - Grafique su evolución en tiempo real.

## Arquitectura del Sistema

El sistema se compone de:

1. **Driver del kernel (CDD)**
2. **Aplicación de usuario en Python (graficador)**

### 1. Módulo del kernel - CDD

Se implementaron **dos versiones del driver**:
- `cdd_driver_virtual.c`: simula señales en software (útil para entorno sin hardware).
- `cdd_driver.c`: pensado para Raspberry Pi con GPIO reales.

Características:
- Dispositivo creado: `/dev/cdd_signal`
- Selección de señal con `echo 0 > /dev/cdd_signal` o `echo 1 > /dev/cdd_signal`
- Lectura con `cat /dev/cdd_signal`: muestra valor actual.
- Temporizador (`timer_list`) que actualiza las señales cada 1000 ms.
- Comunicación mediante funciones estándar del kernel: `alloc_chrdev_region()`, `cdev_init()`, `file_operations`, etc.

### 2. Aplicación de usuario - Graficador

Es un script en Python 3 (`user.py`) que usa:
- `matplotlib`: para graficar.
- `animation.FuncAnimation`: para actualizar el gráfico en tiempo real.
- `matplotlib.widgets.Button`: para seleccionar la señal.

Cada segundo:
- Lee el valor lógico actual desde `/dev/cdd_signal`.
- Actualiza el gráfico.
- Al cambiar de señal, reinicia el gráfico.

## Instrucciones de uso

1. Compilar el driver:

```bash
make
```

2. Insertar módulo en el kernel:

```bash
sudo insmod cdd_driver_virtual.ko
```

3. Dar permisos al dispositivo:

```bash
sudo chmod 666 /dev/cdd_signal
```

4. Ejecutar la app de usuario:

```bash
python3 user/user.py
```

## Ejemplo de funcionamiento

### Cambio de señal

```bash
echo 0 > /dev/cdd_signal  # Seleccionar señal 0
cat /dev/cdd_signal       # Leer valor actual
```

### Capturas de prueba

- Señal 0 (baja una vez y queda en 0):
  ![](./images/Señal0.png)

- Señal 1 (oscilación periódica cada segundo):
  ![](./images/Señal1.png)

## Pruebas realizadas

- Carga y creación del dispositivo: OK
- Lectura y escritura desde espacio de usuario: OK
- Respuesta gráfica en tiempo real: OK
- Cambio de señal durante ejecución: OK

## Conclusion

Durante el desarrollo de este trabajo práctico, se abordaron conceptos avanzados de programación en espacio de kernel y espacio de usuario en Linux. Si bien en principio la consigna parecía sencilla, en la práctica surgieron varios desafíos que requirieron investigación y adaptación, especialmente en lo referido al entorno de trabajo.

- Dificultades encontradas
  - Uno de los principales obstáculos fue la imposibilidad de contar con una Raspberry Pi física, tal como sugiere la consigna. Inicialmente se intentó emular GPIOs usando herramientas como QEMU, gpio-mockup y qemu-rpi-gpio, pero todos los enfoques resultaron muy inestables o imposibles de integrar completamente en un entorno virtual accesible.

Finalmente, optamos por utilizar una máquina virtual (VirtualBox con Debian) y simular las señales directamente desde el kernel usando jiffies, lo que permitió cumplir todos los requisitos del TP sin hardware adicional.

Cabe destacar que la búsqueda de soluciones llevó tiempo y frustración. Afortunadamente, encontramos una guía muy útil en esta página web: "https://raspberrytips.es/raspberry-pi-os-maquina-virtual/", que nos orientó para montar un entorno de trabajo compatible con herramientas básicas de desarrollo para Raspberry Pi OS dentro de VirtualBox. Esto fue clave para avanzar.

- Evaluación general del trabajo
A pesar de las dificultades, el TP permitió poner en práctica conocimientos fundamentales sobre:
- Creación y registro de dispositivos de caracteres en Linux.
- Manejo de estructuras del kernel (cdev, timer_list, file_operations).
- Comunicación entre espacio de usuario y kernel mediante archivos especiales.
- Desarrollo de interfaces gráficas simples para visualización de señales.
- Sincronización de datos en tiempo real y diseño modular.

El sistema desarrollado cumple todos los puntos solicitados en la consigna, incluyendo la visualización gráfica con selección de señales, reinicio automático del gráfico, y simulación correcta de señales digitales con periodo de 1 segundo.

En definitiva, fue un trabajo desafiante, que nos obligó a pensar como desarrolladores de bajo nivel, a investigar nuevas herramientas y a buscar soluciones alternativas frente a las limitaciones del entorno.
