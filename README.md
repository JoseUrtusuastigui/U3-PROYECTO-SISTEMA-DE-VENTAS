# U3-PROYECTO-SISTEMA-DE-VENTAS
 MACROS, PROCEDIMIENTOS Y USES
 
Tabla antes y después(no es una tabla como tal debido a las limitaciones del readme en github):
1. mEscribir texto (17 usos)
- Parte del programa: Impresión de cadenas sin salto de línea (MostrarNumero, MostrarImporte, PedirCodigo, PedirNombre, MostrarProducto, MostrarDesglose y MostrarResumen).
- Problema detectado: Repetición de mov edx, OFFSET texto seguido de call WriteString en 17 puntos distintos al formatear salidas.
- Cambio realizado: Creación de la macro mEscribir texto para encapsular la carga en EDX y la llamada a WriteString.
- Beneficio: Reduce 17 pares de instrucciones repetitivas a llamadas de una sola línea, mejorando la legibilidad del formato en consola.

2. mEscribirLinea texto (38 usos)
- Parte del programa: Despliegue de menús y mensajes con salto de línea (main, MostrarMenu, RegistrarProducto, VenderProducto, RecibirMercancia, etc.).
- Problema detectado: Uso reiterado de mov edx, OFFSET texto junto con call EscribirLinea en 38 ocasiones.
- Cambio realizado: Creación de la macro mEscribirLinea texto que carga el desplazamiento y llama directamente a EscribirLinea.
- Beneficio: Elimina 38 líneas redundantes y compacta a la mitad procedimientos como MostrarMenu.

3. mVerificarRango valor, minimo, maximo, etqError (7 usos)
- Parte del programa: Validación de dígitos, caracteres ASCII y rangos numéricos (ConvertirEntero, ConvertirPrecio, ValidarFormatoCodigo y ValidarCatalogoTemporal).
- Problema detectado: Comparaciones dobles repetidas (cmp + jb y cmp + ja) en 7 puntos distintos para validar límites inferior y superior.
- Cambio realizado: Implementación de mVerificarRango utilizando etiquetas internas declaradas con LOCAL vrMinOk, vrFin.
- Beneficio: Evita colisiones de etiquetas al expandirse varias veces y centraliza la validación de rangos sin riesgo de saltos invertidos.

4. mEscribirArchivo buffer, tamano, etqError (5 usos)
- Parte del programa: Guardado secuencial del catálogo en disco (GuardarCatalogo).
- Problema detectado: Bloque idéntico de 5 líneas (mov edx, mov ecx, call EscribirBloque, cmp ebx, 0, je) repetido 5 veces consecutivas.
- Cambio realizado: Creación de la macro mEscribirArchivo que prepara parámetros, escribe el bloque y salta si ocurre error.
- Beneficio: Reduce 25 líneas repetitivas de E/S a 5 líneas declarativas y facilita verificar cada bloque guardado en Catalogo_Cubos.dat.

5. mLeerArchivo buffer, tamano, etqError (5 usos)
- Parte del programa: Carga secuencial del catálogo desde archivo binario (CargarCatalogo).
- Problema detectado: Secuencia de 5 líneas (mov edx, mov ecx, call LeerBloque, cmp ebx, 0, je) repetida 5 veces para leer los arreglos temporales.
- Cambio realizado: Creación de la macro mLeerArchivo que encapsula la lectura del bloque y el salto ante fallo o archivo truncado.
- Beneficio: Compacta 25 líneas a 5 invocaciones directas y elimina el riesgo de omitir la validación de EBX == 0 en alguna lectura.

6. mCopiarArreglo origen, destino, tamano (4 usos)
- Parte del programa: Volcado de arreglos temporales al inventario activo tras validar el archivo (CargarCatalogo).
- Problema detectado: Repetición de 4 bloques de 4 líneas (mov esi, mov edi, mov ecx, call CopiarBloque) al confirmar los datos cargados.
- Cambio realizado: Creación de la macro mCopiarArreglo para configurar los punteros/tamaño y ejecutar CopiarBloque.
- Beneficio: Reduce 16 líneas a 4 instrucciones claras donde se aprecia de inmediato qué arreglos se actualizan en memoria.


Fragmentos representativos mostrando procedimiento reutilizados:

Puntos de reutilización: BuscarProducto (línea 1026), RecibirMercancia (línea 1042), VenderProducto (línea 1075) y SimularDescuento (línea 1123).

CapturarYBuscarProducto PROC USES ebx edx edi
    call PedirCodigo
    cmp ebx, 1
    jne cybNoValido
    mov edi, OFFSET entrada
    call BuscarCodigo
    cmp eax, -1
    jne cybSalir
    mEscribirLinea msgNoExiste
    jmp cybSalir
cybNoValido:
    mov eax, -1
cybSalir:
    ret
CapturarYBuscarProducto ENDP

Puntos de uso: ConvertirEntero (línea 408), ConvertirPrecio (líneas 448, 471 y 499), ValidarFormatoCodigo (líneas 524 y 534) y ValidarCatalogoTemporal (línea 1314).

mVerificarRango MACRO valor, minimo, maximo, etqError
    LOCAL vrMinOk, vrFin
    cmp valor, minimo
    jae vrMinOk
    jmp etqError
vrMinOk:
    cmp valor, maximo
    jbe vrFin
    jmp etqError
vrFin:
ENDM


Comparación entre las líneas de código antes y despues:
El código fuente total pasó de 1,617 líneas a 1,388 líneas
Esta reducción se logro gracias a la eliminación de 4 bloques principales.

1.Respaldo y restauración manual de registros en la pila: En la versión original, cada uno de los 30 procedimientos contenía entre 1 y 6 instrucciones push al inicio y sus correspondientes pop al final. Al sustituirlos por la directiva USES en la cabecera de cada procedimiento, se eliminaron más de 170 líneas de código repetitivo y el riesgo de desequilibrar la pila en saltos de salida.

2.Carga de direcciones de cadena para impresión en consola: El patrón mov edx, OFFSET cadena seguido de call WriteString o call EscribirLinea aparecía 55 veces en total. Al reemplazarlo por las macros mEscribir y mEscribirLinea, cada impresión pasó de ocupar dos líneas a una sola.

3.Rutinas de captura y validación de entrada en las opciones del menú: El bloque de 10 líneas encargado de pedir un código, validar su formato, buscarlo en el arreglo y mostrar el mensaje de error si no existía se repetía idéntico en cuatro procedimientos: BuscarProducto, RecibirMercancia, VenderProducto y SimularDescuento; se extrajo al nuevo procedimiento CapturarYBuscarProducto. De igual forma, la validación de que una cantidad ingresada fuera mayor que cero se repetía en tres opciones del menú y se centralizó en PedirCantidadPositiva.

4.Operaciones secuenciales de archivos y copia de arreglos: En GuardarCatalogo y CargarCatalogo se repetían bloques de 5 líneas por cada arreglo escrito o leído en disco, además de 4 bloques de copia de memoria al restaurar los arreglos temporales. Con las macros mEscribirArchivo, mLeerArchivo y mCopiarArreglo, esas 66 líneas se compactaron en 14 llamadas directas y legibles.


Tabla de pruebas actualizada: Los obtenidos fueron idénticos a los obtenidos en la versión del código de la unidad 2

