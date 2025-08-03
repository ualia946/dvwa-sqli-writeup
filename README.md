### Análisis de la Vulnerabilidad: SQL Injection (SQLi)

La inyección de SQL es una vulnerabilidad de alta criticidad que surge cuando la consulta que una aplicación envía a su base de datos se construye de forma insegura. Específicamente, ocurre al **concatenar directamente las entradas del usuario en la cadena de la consulta SQL** sin una validación o sanitización adecuadas.

Esto permite a un atacante manipular la consulta final, transformándola en una operación completamente diferente a la que el programador diseñó originalmente. Las consecuencias pueden ser graves, desde la extracción de información sensible de toda la base de datos hasta eludir los mecanismos de autenticación para obtener acceso no autorizado.

### Mecánica del Ataque

La explotación de esta vulnerabilidad se basa en "romper" la sintaxis de la consulta original e inyectar código SQL malicioso. Las técnicas más comunes incluyen:

- **Cierre de Comillas (`'`):** Se utiliza una comilla simple para cerrar una cadena de texto esperada por la aplicación, permitiendo que el resto de la entrada sea interpretado como código SQL.
- **Condiciones Lógicas (`OR '1'='1'`):** Se añaden expresiones que siempre evalúan como verdaderas para anular la lógica de filtrado de la cláusula `WHERE`, forzando a la base de datos a devolver más registros de los debidos.
- **Comentarios SQL (`--`):** Se utilizan para neutralizar el resto de la consulta original, evitando errores de sintaxis que harían fracasar el ataque.

### Estrategias de Mitigación

Para construir aplicaciones robustas y seguras frente a este ataque, es fundamental implementar una estrategia de **defensa en profundidad**, combinando las siguientes técnicas:

1.  **Sentencias Preparadas (Prepared Statements):** Esta es la defensa más importante y eficaz. Consiste en enviar la estructura de la consulta a la base de datos por separado de los datos del usuario. La consulta se pre-compila con marcadores de posición (`?`), y los datos del usuario se envían después, siendo tratados siempre como valores literales y nunca como código SQL ejecutable. Esto neutraliza la inyección por completo.

2.  **Validación de Entradas (Input Validation):** Como segunda capa de defensa, se debe validar siempre la entrada del usuario. Si un campo espera un ID numérico, la aplicación debe rechazar cualquier entrada que no sea un número. Si espera un correo electrónico, debe validar que el formato sea correcto. Esta técnica, conocida como "whitelisting", reduce drásticamente la superficie de ataque.

### Ejemplo Práctico: Explotación y Análisis

Para demostrar la mecánica del ataque, se utilizará el laboratorio para combinar las tres técnicas descritas y extraer todos los registros de usuarios de la base de datos.

#### Código Vulnerable

La vulnerabilidad en el módulo "SQL Injection" de DVWA reside en cómo el backend construye la consulta SQL. El siguiente fragmento de código PHP ilustra el problema:

```php
// 1. Obtener la entrada del usuario directamente desde la URL.
$id_usuario = $_GET['id'];

// 2. Construir la consulta concatenando la entrada del usuario directamente.
//    Esta línea es la que crea la vulnerabilidad.
$sql = "SELECT nombre, apellido FROM usuarios WHERE id = '" . $id_usuario . "'";

// 3. Ejecutar la consulta.
$resultado = $conn->query($sql);
```

El problema fundamental es que la variable `$id_usuario` se inserta directamente en la cadena de la consulta. La base de datos no distingue entre los datos que se esperan y el código SQL malicioso que un atacante puede inyectar, interpretando todo como parte de la misma instrucción.

#### El Payload

Para explotar esta vulnerabilidad, en lugar de un ID numérico, se introduce el siguiente payload en el campo de entrada:

```sql
' OR '1'='1' --
```

![Ejemplo de Inyección SQL en DVWA](CAPTURA_DE_PANTALLA)

#### Análisis de la Consulta Manipulada

La inyección del payload transforma la consulta original en la siguiente instrucción maliciosa:

```sql
SELECT nombre, apellido FROM usuarios WHERE id = '' OR '1'='1' -- ';
```

Esta nueva consulta se aprovecha de la sintaxis de SQL de la siguiente manera:

1.  **Cierre de Comillas (`'`):** La primera comilla del payload cierra la comilla que el código esperaba, completando la condición `id = ''`.
2.  **Condición Lógica (`OR '1'='1'`):** Se introduce una condición que siempre es verdadera. Esto provoca que la cláusula `WHERE` se cumpla para cada registro de la tabla, ignorando el filtro original por `id`.
3.  **Comentario (`-- `):** Los dos guiones seguidos de un espacio inician un comentario en SQL, lo que neutraliza la comilla simple final que el código PHP habría añadido, evitando así un error de sintaxis.

El resultado es que la base de datos devuelve todos los registros de la tabla de usuarios.

#### Impacto y Consecuencias

Aunque en este ejemplo se ha utilizado para extraer datos, la misma técnica podría aplicarse en un formulario de inicio de sesión para eludir la autenticación por completo y obtener acceso no autorizado al sistema.
