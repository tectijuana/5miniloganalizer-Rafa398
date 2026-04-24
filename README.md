[![Review Assignment Due Date](https://classroom.github.com/assets/deadline-readme-button-22041afd0340ce965d47ae6ef1cefeee28c7c493a6346c4f15d667ab976d596c.svg)](https://classroom.github.com/a/QtRYN9D3)
[![Open in Codespaces](https://classroom.github.com/assets/launch-codespace-2972f46106e565e64193e422d61a12cf1da4916b45550586e14ef0a7c637dd04.svg)](https://classroom.github.com/open-in-codespaces?assignment_repo_id=23669377)

# Práctica 1

## Implementación de un Mini Cloud Log Analyzer en ARM64

**Modalidad:** Individual
**Entorno de trabajo:** AWS Ubuntu ARM64 + GitHub Classroom
**Lenguaje:** ARM64 Assembly (GNU Assembler) + Bash + GNU Make
Alumno: Rafael Del Callejo Tapia
---

## Introducción

Los sistemas modernos de cómputo en la nube generan continuamente registros (*logs*) que permiten monitorear el estado de servicios, detectar fallas y activar alertas ante eventos críticos.

En esta práctica se desarrollará un módulo simplificado de análisis de logs, implementado en **ARM64 Assembly**, inspirado en tareas reales de monitoreo utilizadas en sistemas cloud, observabilidad y administración de infraestructura.

El programa procesará códigos de estado HTTP suministrados mediante entrada estándar (stdin):

```bash id="y1gcmc"
cat logs.txt | ./analyzer
```

---

## Objetivo general

Diseñar e implementar, en lenguaje ensamblador ARM64, una solución para procesar registros de eventos y detectar condiciones definidas según la variante asignada.

---

## Objetivos específicos

El estudiante aplicará:

* programación en ARM64 bajo Linux
* manejo de registros
* direccionamiento y acceso a memoria
* instrucciones de comparación
* estructuras iterativas en ensamblador
* saltos condicionales
* uso de syscalls Linux
* compilación con GNU Make
* control de versiones con GitHub Classroom

Estos temas se alinean con contenidos clásicos de flujo de control, herramientas GNU, manejo de datos y convenciones de programación en ensamblador.   

---

## Material proporcionado

Se entregará un repositorio preconfigurado que contiene:

* plantilla base en ARM64
* archivo `Makefile`
* script Bash de ejecución
* archivo de datos (`logs.txt`)
* pruebas iniciales
* secciones marcadas con `TODO`

El estudiante deberá completar la lógica correspondiente.

---

## Variantes de la práctica


### Variante C
/*========================================================
Nombre: analyzer.s
Detecta el primer 503 leyendo carácter por carácter
==========================================================*/

.global _start

.section .data
msg: .ascii "503 detected\n"
len = . - msg

.section .bss
char: .skip 1

.section .text

_start:

    mov x2, 0      // número actual = 0

loop:
    // read 1 byte
    mov x0, 0
    ldr x1, =char
    mov x2, 1
    mov x8, 63
    svc 0

    cmp x0, 0
    beq exit

    ldrb w3, [x1]

    cmp w3, 10     // '\n'
    beq check

    sub w3, w3, '0'
    mov x4, 10
    mul x2, x2, x4
    add x2, x2, x3

    b loop

check:
    mov x5, 503
    cmp x2, x5
    beq found

    mov x2, 0      // reset número
    b loop

found:
    mov x0, 1
    ldr x1, =msg
    mov x2, len
    mov x8, 64
    svc 0

exit:
    mov x0, 0
    mov x8, 93
    svc 0

##Explicacion de diseño
## Diseño y lógica utilizada

El programa fue diseñado para detectar el primer evento crítico con código **503** a partir de un flujo de datos recibido por la entrada estándar (`stdin`). Este flujo proviene del archivo `logs.txt`, el cual se redirige al programa mediante el uso de tuberías en Linux.

### Diseño general

La solución sigue un enfoque de **procesamiento secuencial**, donde los datos son leídos carácter por carácter utilizando la syscall `read`. Este enfoque fue elegido debido a que en lenguaje ensamblador no existen funciones de alto nivel como `scanf`, por lo que es necesario implementar manualmente el análisis de los datos.

El programa se estructura en las siguientes etapas:

1. **Lectura de datos**
   Se utiliza la syscall `read` para obtener un byte a la vez desde la entrada estándar. Esto permite un control preciso sobre el flujo de datos y facilita la identificación de separadores (saltos de línea).

2. **Construcción del número**
   Cada carácter leído es evaluado:

   * Si es un dígito (`'0'` a `'9'`), se convierte de ASCII a su valor numérico.
   * Se construye el número acumulado multiplicando el valor actual por 10 y sumando el nuevo dígito.

   Este proceso simula el comportamiento de conversión de cadenas a enteros en lenguajes de alto nivel.

3. **Detección de fin de número**
   Cuando se detecta un salto de línea (`'\n'`), se considera que el número ha sido completamente leído.

4. **Comparación del valor**
   El número construido se compara con el valor objetivo (503) mediante instrucciones de comparación (`cmp`).

   * Si el valor coincide, se procede a mostrar el mensaje correspondiente.
   * Si no coincide, el acumulador se reinicia y el proceso continúa.

5. **Salida del programa**
   En caso de detectar el código 503, se utiliza la syscall `write` para imprimir el mensaje `"503 detected"` en la salida estándar.
   Posteriormente, el programa finaliza utilizando la syscall `exit`.

---

### Lógica de control

El flujo del programa se basa en:

* **Estructura iterativa**: implementada mediante etiquetas y saltos (`b`, `beq`, `bne`), simulando un ciclo `while`.
* **Condicionales**: implementados con la instrucción `cmp` seguida de saltos condicionales.
* **Acumulador numérico**: almacenado en un registro, el cual se reinicia cada vez que se termina de procesar un número.

---

### Justificación del enfoque

El uso de lectura carácter por carácter permite:

* Control total sobre el parsing de datos
* Independencia de funciones externas (como librerías C)
* Cumplimiento de la restricción de utilizar exclusivamente ensamblador ARM64
* Aplicación directa de conceptos como manejo de registros, memoria y flujo de control

---

### Conclusión

El programa demuestra cómo un problema de procesamiento de datos, que normalmente se resolvería fácilmente en lenguajes de alto nivel, puede ser implementado a bajo nivel utilizando instrucciones de ensamblador ARM64. Se aplican conceptos fundamentales como:

* Manipulación de registros
* Conversión de datos
* Control de flujo
* Uso de syscalls de Linux

Esto permite comprender de manera más profunda cómo la arquitectura procesa información a nivel máquina.

Cada estudiante deberá entregar en su repositorio:

* archivo fuente ARM64 funcional
* solución implementada
* README explicando diseño y lógica utilizada
* evidencia de ejecución
* commits realizados en GitHub Classroom

---

## Criterios de evaluación

| Criterio                    | Ponderación |
| --------------------------- | ----------- |
| Compilación correcta        | 20%         |
| Correctitud de la solución  | 35%         |
| Uso adecuado de ARM64       | 25%         |
| Documentación y comentarios | 10%         |
| Evidencia de pruebas        | 10%         |

---

## Restricciones

No está permitido:

* resolver la lógica en C
* resolver la lógica en Python
* modificar la variante asignada
* omitir el uso de ARM64 Assembly

---

## Competencia a desarrollar

Comprender cómo un problema de procesamiento de datos es implementado a nivel máquina mediante instrucciones ARM64.

---

## Nota

Aunque este problema puede resolverse fácilmente en lenguajes de alto nivel, el propósito de la práctica es implementar **cómo lo resolvería la arquitectura**, no únicamente obtener el resultado.
 https://asciinema.org/a/2VXRr4yMx3PpiOmQ
