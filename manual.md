# Manual de Estudio Completo: Sistema de Reservas de Laboratorios

Este manual ha sido diseñado como una guía académica autodidacta para comprender a fondo, línea por línea, la arquitectura, los algoritmos y el diseño de la aplicación de **Sistema de Reservas de Laboratorios**.

El documento contiene el código íntegro de los 12 archivos fuentes del proyecto, organizados por capas de software, junto con explicaciones de su funcionamiento y un glosario técnico detallado al final de cada sección.

---

## 1. Arquitectura General del Sistema

La aplicación está diseñada bajo el patrón de **Arquitectura Multicapa**, el cual separa las responsabilidades de la aplicación en componentes independientes:

```text
┌─────────────────────────────────────────────────────────┐
│              Capa de Presentación (interfaz)            │
│  [VentanaPrincipal] [PanelReservas] [PanelUsuarios]...   │
└────────────────────────────┬────────────────────────────┘
                             │ (Invoca métodos de negocio)
┌────────────────────────────▼────────────────────────────┐
│               Capa de Negocio (negocio)                 │
│                   [SistemaReservas]                     │
└────────────────────────────┬────────────────────────────┘
                             │ (Controla objetos e interactúa con persistencia)
┌────────────────────────────┼────────────────────────────┐
│   Capa de Modelo (modelo)  │ Capa de Acceso a Datos (p) │
│ [Usuario] [Lab] [Reserva]  │       [StorageUtil]        │
└────────────────────────────┴────────────────────────────┘
```

1.  **Capa de Presentación (`interfaz`):** Construida enteramente con la biblioteca clásica **Java AWT**. Es la encargada de pintar los formularios, botones, listas y capturar los eventos del ratón y teclado.
2.  **Capa de Negocio (`negocio`):** Actúa como el controlador mediador. Valida reglas del negocio (como que no haya colisión de horarios), mantiene la consistencia de datos y orquesta las operaciones.
3.  **Capa de Datos (`persistencia`):** Encapsula el acceso de bajo nivel al sistema de archivos del disco duro (lectura y escritura de archivos de texto).
4.  **Capa del Modelo (`modelo`):** Define las estructuras de datos (Usuario, Laboratorio y Reserva) y cómo se convierten a/desde cadenas de texto delimitadas por punto y coma (`;`).

---

## 2. El Punto de Entrada de la Aplicación

### 2.1 Código de `SistemaReservasLaboratorios.java`
Ubicado en `src/sistemareservaslaboratorios/SistemaReservasLaboratorios.java`.

```java
package sistemareservaslaboratorios;

import interfaz.VentanaPrincipal;

public class SistemaReservasLaboratorios {
    public static void main(String[] args) {
        new VentanaPrincipal().setVisible(true);
    }
}
```

### 2.2 Explicación del Código y Funcionamiento
*   **¿Cómo funciona?** Al iniciar la aplicación, la Máquina Virtual de Java (JVM) busca el método `main`. Este método crea un objeto de la ventana gráfica en memoria y lo levanta como un proceso del sistema operativo.
*   **Línea por línea:**
    *   `package`: Indica en qué carpeta/paquete del proyecto se encuentra guardada esta clase física.
    *   `import`: Trae la clase `VentanaPrincipal` desde el paquete `interfaz` para poder usarla en este archivo.
    *   `public static void main`: El método especial que el compilador utiliza como interruptor de encendido.
    *   `new VentanaPrincipal().setVisible(true)`: Instancia la ventana en memoria e inmediatamente le ordena al manejador gráfico mostrarla en pantalla.

### 2.3 Glosario del Punto de Entrada
*   **`package`:** Palabra clave de Java que organiza las clases en carpetas lógicas para evitar colisiones de nombres.
*   **`import`:** Instrucción que declara la dependencia de una clase respecto a otra ubicada en otro paquete.
*   **`main(String[] args)`:** Punto de entrada obligatorio y estático. La JVM lo invoca pasando los argumentos del terminal en el arreglo `args`.
*   **`setVisible(boolean)`:** Método heredado de la clase `Window` en AWT. Sirve para ocultar (`false`) o mostrar en pantalla (`true`) una ventana gráfica.

---

## 3. Capa del Modelo de Datos (`modelo`)

Representan los registros de información estructurados. Traducen las cadenas de texto del disco plano en objetos de Java y viceversa.

### 3.1 Código de `Usuario.java`
Ubicado en `src/modelo/Usuario.java`.

```java
package modelo;

public class Usuario {
    private String rut;
    private String nombre;
    private String tipo;
    private String correo;

    public Usuario(String rut, String nombre, String tipo, String correo) {
        this.rut = rut; this.nombre = nombre; this.tipo = tipo; this.correo = correo;
    }
    public String getRut() { return rut; }
    public String getNombre() { return nombre; }
    public String getTipo() { return tipo; }
    public String getCorreo() { return correo; }

    @Override
    public String toString() {
        return rut + ";" + nombre + ";" + tipo + ";" + correo;
    }
    public static Usuario fromString(String line) {
        String[] p = line.split(";", -1);
        String rut = p.length>0? p[0] : "";
        String nombre = p.length>1? p[1] : "";
        String tipo = p.length>2? p[2] : "";
        String correo = p.length>3? p[3] : "";
        return new Usuario(rut, nombre, tipo, correo);
    }
}
```

---

### 3.2 Código de `Laboratorio.java`
Ubicado en `src/modelo/Laboratorio.java`.

```java
package modelo;

public class Laboratorio {
    private String codigo;
    private String nombre;
    private int capacidad;
    private String ubicacion;
    private String equipamiento;

    public Laboratorio(String codigo, String nombre, int capacidad, String ubicacion, String equipamiento) {
        this.codigo = codigo; this.nombre = nombre; this.capacidad = capacidad;
        this.ubicacion = ubicacion; this.equipamiento = equipamiento;
    }
    public String getCodigo() { return codigo; }
    public String getNombre() { return nombre; }
    public int getCapacidad() { return capacidad; }
    public String getUbicacion() { return ubicacion; }
    public String getEquipamiento() { return equipamiento; }

    @Override
    public String toString() {
        return codigo + ";" + nombre + ";" + capacidad + ";" + ubicacion + ";" + equipamiento;
    }
    public static Laboratorio fromString(String line) {
        String[] p = line.split(";", -1);
        String codigo = p.length>0? p[0] : "";
        String nombre = p.length>1? p[1] : "";
        int capacidad = p.length>2 && !p[2].isEmpty() ? Integer.parseInt(p[2]) : 0;
        String ubicacion = p.length>3? p[3] : "";
        String equipamiento = p.length>4? p[4] : "";
        return new Laboratorio(codigo, nombre, capacidad, ubicacion, equipamiento);
    }
}
```

---

### 3.3 Código de `Reserva.java`
Ubicado en `src/modelo/Reserva.java`.

```java
package modelo;

public class Reserva {
    private String id;
    private String rutUsuario;
    private String codigoLab;
    private String fecha; // yyyy-MM-dd
    private String bloque;
    private String motivo;

    public Reserva(String id, String rutUsuario, String codigoLab, String fecha, String bloque, String motivo) {
        this.id = id; this.rutUsuario = rutUsuario; this.codigoLab = codigoLab;
        this.fecha = fecha; this.bloque = bloque; this.motivo = motivo;
    }
    public String getId() { return id; }
    public String getRutUsuario() { return rutUsuario; }
    public String getCodigoLab() { return codigoLab; }
    public String getFecha() { return fecha; }
    public String getBloque() { return bloque; }
    public String getMotivo() { return motivo; }

    @Override
    public String toString() {
        return id + ";" + rutUsuario + ";" + codigoLab + ";" + fecha + ";" + bloque + ";" + motivo;
    }
    public static Reserva fromString(String line) {
        String[] p = line.split(";", -1);
        String id = p.length>0? p[0] : "";
        String rut = p.length>1? p[1] : "";
        String lab = p.length>2? p[2] : "";
        String fecha = p.length>3? p[3] : "";
        String bloque = p.length>4? p[4] : "";
        String motivo = p.length>5? p[5] : "";
        return new Reserva(id, rut, lab, fecha, bloque, motivo);
    }
}
```

### 3.4 Explicación del Funcionamiento de los Modelos
*   **¿Cómo se hace la serialización?** Cada clase modelo implementa el método `toString()`. Cuando se invoca, concatena todas las variables separándolas con un punto y coma (`;`).
*   **¿Cómo se hace la deserialización segura?** El método estático `fromString(String line)` realiza el proceso inverso:
    1.  Recibe la línea leída del archivo de texto.
    2.  Llama a `line.split(";", -1)` para crear un arreglo de cadenas (`String[] p`). El valor `-1` asegura que los campos vacíos al final no se omitan.
    3.  Aplica el **operador ternario** (`p.length > index ? p[index] : ""`). Si el arreglo es más corto de lo esperado (debido a un archivo dañado o incompleto), asigna un texto vacío en lugar de provocar un fallo por índice fuera de rango.
    4.  Crea e instancia un nuevo objeto con los parámetros limpios.

### 3.5 Glosario de la Capa de Modelos
*   **`private`:** Modificador de acceso que encapsula los atributos. Significa que ninguna otra clase del programa puede modificar estas variables directamente, protegiendo la integridad del objeto.
*   **`@Override`:** Anotación opcional pero recomendada. Le indica al compilador que se está sobrescribiendo un método de la clase padre (en este caso, el método `toString()` heredado de `java.lang.Object`).
*   **`static`:** Define que un método o variable pertenece a la clase misma y no a una instancia particular. Permite llamar a `Usuario.fromString(line)` directamente sin necesidad de crear un usuario vacío previo.
*   **`split(String regex, int limit)`:** Método de la clase `String`. Divide un texto en partes utilizando un separador (`regex`). Si `limit` es `-1`, no corta los campos vacíos finales del resultado.
*   **`Integer.parseInt(String s)`:** Convierte una cadena de texto en un número entero (`int`). Lanza una excepción si el texto contiene letras o caracteres no numéricos.
*   **`isEmpty()`:** Método de la clase `String`. Retorna `true` si la cadena tiene una longitud de cero caracteres (`""`).
*   **Operador Ternario (`condicion ? valorSiVerdadero : valorSiFalso`):** Forma compacta de escribir un condicional `if-else` en una sola línea de código para asignación de variables.

---

## 4. Capa de Acceso a Datos (`persistencia`)

Abstrae la entrada y salida (I/O) del disco, liberando a la capa de negocio de lidiar con flujos de archivos complejos.

### 4.1 Código de `StorageUtil.java`
Ubicado en `src/persistencia/StorageUtil.java`.

```java
package persistencia;

import java.io.*;
import java.util.*;

public class StorageUtil {
    public static List<String> readAll(String path) {
        List<String> lines = new ArrayList<>();
        File f = new File(path);
        if (!f.exists()) return lines;
        try (BufferedReader r = new BufferedReader(new FileReader(f))) {
            String l;
            while ((l = r.readLine()) != null) lines.add(l);
        } catch (IOException e) { e.printStackTrace(); }
        return lines;
    }
    public static void writeAll(String path, List<String> lines) {
        try (PrintWriter w = new PrintWriter(new FileWriter(path))) {
            for (String l : lines) w.println(l);
        } catch (IOException e) { e.printStackTrace(); }
    }
}
```

### 4.2 Explicación del Funcionamiento de la Persistencia
*   **¿Cómo se lee un archivo?** 
    1.  `File f = new File(path)` apunta a la ruta (ej. `"usuarios.txt"`).
    2.  `if (!f.exists())` verifica si no existe en la carpeta raíz del proyecto. Si no existe, retorna una lista vacía para que no se caiga la app.
    3.  Abre un `FileReader` (flujo de lectura física de caracteres) y encima un `BufferedReader` (agrega memoria intermedia para leer líneas completas de forma rápida).
    4.  Utiliza un bucle `while` para almacenar línea por línea en el arreglo dinámico `lines` hasta que `readLine()` devuelve `null` (final del archivo).
*   **¿Cómo se escribe un archivo?**
    1.  `new FileWriter(path)` abre el archivo para escritura de texto plano.
    2.  `PrintWriter w` provee el método cómodo `println()` para escribir líneas agregando los saltos de carro del sistema operativo correspondiente de manera automática.
    3.  Al usarse **try-with-resources**, Java cierra el flujo al finalizar el bloque de escritura, asegurando la grabación física en el disco duro.

### 4.3 Glosario de la Capa de Persistencia
*   **`List<String>` / `ArrayList`:** Estructura de datos dinámica de Java que representa una lista ordenada de elementos (en este caso cadenas). A diferencia de los arreglos estáticos (`String[]`), su tamaño se expande automáticamente al insertar elementos.
*   **`File`:** Clase del paquete `java.io` que representa rutas de archivos y directorios en disco físico. Permite verificar permisos, existencia y tamaño de archivos.
*   **`FileReader` / `FileWriter`:** Flujos (Streams) que permiten leer y escribir archivos a nivel de caracteres en formato de codificación por defecto.
*   **`BufferedReader`:** Clase que lee texto de una corriente de entrada de caracteres, almacenando los caracteres en búfer para proporcionar una lectura eficiente de caracteres, arreglos y líneas.
*   **`PrintWriter`:** Envuelve a un escritor de archivos y provee métodos de formato cómodos (`print`, `println`, `printf`) de alto nivel.
*   **`try-with-resources`:** Declaración `try` que asegura el cierre automático de los recursos declarados en su cabecera al finalizar el bloque, evitando fugas de memoria y bloqueos de archivos en el sistema operativo.
*   **`IOException`:** Excepción general que captura cualquier error físico al leer o escribir datos en disco (disco lleno, falta de permisos, archivo protegido, etc.).
*   **`e.printStackTrace()`:** Imprime en la consola de depuración el historial detallado de llamadas de métodos que causó el error de ejecución.

---

## 5. Capa de Reglas de Negocio y Control (`negocio`)

Es el cerebro del programa. Mantiene en memoria RAM los datos del sistema para que no sea necesario consultar el disco duro en cada clic de la interfaz, aplicando restricciones y procesamientos lógicos.

### 5.1 Código de `SistemaReservas.java`
Ubicado en `src/negocio/SistemaReservas.java`.

```java
package negocio;

import modelo.*;
import persistencia.StorageUtil;

import java.time.LocalDate;
import java.util.*;
import java.util.stream.Collectors;

public class SistemaReservas {
    private Map<String, Usuario> usuarios = new LinkedHashMap<>();
    private Map<String, Laboratorio> labs = new HashMap<>();
    private Map<String, Reserva> reservas = new LinkedHashMap<>();

    private final String USUARIOS_FILE = "usuarios.txt";
    private final String LABS_FILE = "laboratorios.txt";
    private final String RESERVAS_FILE = "reservas.txt";

    public SistemaReservas() { }

    /**
     * Carga datos desde archivos. No crea datos de prueba automáticamente.
     */
    public void cargarDatos() {
        usuarios.clear();
        labs.clear();
        reservas.clear();

        StorageUtil.readAll(USUARIOS_FILE).forEach(l -> {
            try {
                Usuario u = Usuario.fromString(l);
                if (u.getRut() != null && !u.getRut().isEmpty()) usuarios.put(u.getRut(), u);
            } catch (Exception ex) { /* ignorar línea corrupta */ }
        });

        StorageUtil.readAll(LABS_FILE).forEach(l -> {
            try {
                Laboratorio lab = Laboratorio.fromString(l);
                if (lab.getCodigo() != null && !lab.getCodigo().isEmpty()) labs.put(lab.getCodigo(), lab);
            } catch (Exception ex) { /* ignorar */ }
        });

        StorageUtil.readAll(RESERVAS_FILE).forEach(l -> {
            try {
                Reserva r = Reserva.fromString(l);
                if (r.getId() != null && !r.getId().isEmpty()) reservas.put(r.getId(), r);
            } catch (Exception ex) { /* ignorar */ }
        });
    }

    public void guardarDatos() {
        List<String> us = usuarios.values().stream().map(Usuario::toString).collect(Collectors.toList());
        StorageUtil.writeAll(USUARIOS_FILE, us);

        List<String> ls = labs.values().stream().map(Laboratorio::toString).collect(Collectors.toList());
        StorageUtil.writeAll(LABS_FILE, ls);

        List<String> rs = reservas.values().stream().map(Reserva::toString).collect(Collectors.toList());
        StorageUtil.writeAll(RESERVAS_FILE, rs);
    }

    // ---------------- CRUD básicos ----------------

    public boolean registrarUsuario(Usuario u) {
        if (u == null || u.getRut() == null || u.getRut().trim().isEmpty()) return false;
        if (usuarios.containsKey(u.getRut())) return false;
        usuarios.put(u.getRut(), u);
        guardarDatos();
        return true;
    }

    public boolean registrarLaboratorio(Laboratorio l) {
        if (l == null || l.getCodigo() == null || l.getCodigo().trim().isEmpty()) return false;
        if (labs.containsKey(l.getCodigo())) return false;
        labs.put(l.getCodigo(), l);
        guardarDatos();
        return true;
    }

    public String intentarCrearReserva(Reserva r) {
        if (r == null) return "Reserva inválida";
        if (r.getRutUsuario() == null || r.getRutUsuario().trim().isEmpty()) return "RUT vacío";
        if (r.getCodigoLab() == null || r.getCodigoLab().trim().isEmpty()) return "Código de laboratorio vacío";
        if (r.getFecha() == null || r.getFecha().trim().isEmpty()) return "Fecha vacía";
        if (r.getBloque() == null || r.getBloque().trim().isEmpty()) return "Bloque vacío";

        String rut = r.getRutUsuario().trim();
        String lab = r.getCodigoLab().trim();
        String fecha = r.getFecha().trim();
        String bloque = r.getBloque().trim();

        if (!usuarios.containsKey(rut)) return "Usuario no registrado: " + rut;
        if (!labs.containsKey(lab)) return "Laboratorio no registrado: " + lab;

        try {
            LocalDate.parse(fecha);
        } catch (Exception ex) {
            return "Formato de fecha inválido (usar yyyy-MM-dd)";
        }

        for (Reserva ex : reservas.values()) {
            if (ex.getCodigoLab().equals(lab) && ex.getFecha().equals(fecha) && ex.getBloque().equals(bloque)) {
                return "Conflicto: ya existe una reserva para " + lab + " " + fecha + " " + bloque;
            }
        }

        reservas.put(r.getId(), r);
        guardarDatos();
        return "OK";
    }

    public List<Reserva> listarReservas() { return new ArrayList<>(reservas.values()); }
    public List<Usuario> listarUsuarios() { return new ArrayList<>(usuarios.values()); }
    public List<Laboratorio> listarLaboratorios() { return new ArrayList<>(labs.values()); }
    public Reserva obtenerReservaPorId(String id) { return reservas.get(id); }

    public boolean eliminarReserva(String id) {
        if (id == null || !reservas.containsKey(id)) return false;
        reservas.remove(id);
        guardarDatos();
        return true;
    }

    public boolean modificarReserva(String id, String nuevaFecha, String nuevoBloque, String nuevoMotivo) {
        Reserva r = reservas.get(id);
        if (r == null) return false;
        for (Reserva ex : reservas.values()) {
            if (!ex.getId().equals(id)
                && ex.getCodigoLab().equals(r.getCodigoLab())
                && ex.getFecha().equals(nuevaFecha)
                && ex.getBloque().equals(nuevoBloque)) {
                return false;
            }
        }
        Reserva mod = new Reserva(r.getId(), r.getRutUsuario(), r.getCodigoLab(), nuevaFecha, nuevoBloque, nuevoMotivo);
        reservas.put(id, mod);
        guardarDatos();
        return true;
    }

    // ----------------- Métodos de limpieza -----------------

    public int eliminarUsuariosDuplicados() {
        Map<String, Usuario> reconstruido = new LinkedHashMap<>();
        int eliminados = 0;
        for (Usuario u : usuarios.values()) {
            if (!reconstruido.containsKey(u.getRut())) reconstruido.put(u.getRut(), u);
            else eliminados++;
        }
        usuarios = reconstruido;
        return eliminados;
    }

    public int eliminarReservasHuerfanas() {
        List<String> aEliminar = new ArrayList<>();
        for (Reserva r : reservas.values()) if (!usuarios.containsKey(r.getRutUsuario())) aEliminar.add(r.getId());
        for (String id : aEliminar) reservas.remove(id);
        return aEliminar.size();
    }

    public String limpiarDuplicadosYHuerfanos() {
        int u = eliminarUsuariosDuplicados();
        int r = eliminarReservasHuerfanas();
        guardarDatos();
        return String.format("Usuarios eliminados: %d\nReservas eliminadas: %d", u, r);
    }

    public void backupDatos(String prefix) {
        try {
            StorageUtil.writeAll(prefix + "_usuarios.txt", usuarios.values().stream().map(Usuario::toString).collect(Collectors.toList()));
            StorageUtil.writeAll(prefix + "_laboratorios.txt", labs.values().stream().map(Laboratorio::toString).collect(Collectors.toList()));
            StorageUtil.writeAll(prefix + "_reservas.txt", reservas.values().stream().map(Reserva::toString).collect(Collectors.toList()));
        } catch (Exception ex) { ex.printStackTrace(); }
    }

    // Método de depuración opcional
    public void printDebug() {
        System.out.println("=== DEBUG SistemaReservas ===");
        System.out.println("Usuarios: " + usuarios.size());
        usuarios.values().forEach(u -> System.out.println("U: " + u.getRut() + " | " + u.getNombre()));
        System.out.println("Laboratorios: " + labs.size());
        labs.values().forEach(l -> System.out.println("L: " + l.getCodigo()));
        System.out.println("Reservas: " + reservas.size());
        reservas.values().forEach(r -> System.out.println("R: " + r.getId() + " | " + r.getRutUsuario() + " | " + r.getCodigoLab()));
        System.out.println("=============================");
    }
}
```

### 5.2 Explicación Profunda de los Métodos de `SistemaReservas.java`
A continuación se detalla **cómo se hace** y el **funcionamiento interno** de cada proceso de la clase negocio:

#### A. Carga de Datos (`cargarDatos`):
*   **¿Cómo se hace?** Invoca a `StorageUtil.readAll()` para leer los archivos planos línea por línea y poblar los mapas en RAM.
*   **Funcionamiento:**
    *   Llama a `.clear()` en los mapas de usuarios, laboratorios y reservas para vaciar la memoria RAM previa.
    *   Itera sobre las líneas con `.forEach(...)`. Convierte la línea a un objeto llamando al método estático `fromString()`.
    *   Verifica si la clave primaria no está vacía antes de guardarla en el mapa (`usuarios.put(u.getRut(), u)`).
    *   Cada iteración tiene un `try-catch` independiente. Si el archivo de texto tiene una línea corrupta, el try-catch evita que la carga falle, omitiendo la línea dañada en silencio y continuando con las siguientes.

#### B. Registro de Entidades (`registrarUsuario` y `registrarLaboratorio`):
*   **¿Cómo se hace?** Recibe un objeto del modelo, valida que no esté vacío, que su identificador no exista y lo guarda.
*   **Funcionamiento:**
    *   Valida parámetros nulos o cadenas vacías con `.trim().isEmpty()`.
    *   Usa `usuarios.containsKey(u.getRut())`. Si da `true`, significa que el RUT ya está en uso. Retorna `false` para impedir el registro de duplicados.
    *   Si el RUT es único, inserta el objeto en el mapa en memoria (`usuarios.put(...)`) e inmediatamente llama a `guardarDatos()` para persistir el nuevo registro en el disco de forma síncrona.

#### C. Lógica de Agendamiento (`intentarCrearReserva`):
*   **¿Cómo se hace?** Es el algoritmo más estricto. Valida existencia de relaciones físicas en los mapas y previene solapamientos de horario en el mismo laboratorio físico.
*   **Funcionamiento:**
    1.  **Validación de Claves Foráneas:** Revisa que el RUT del usuario y el código del laboratorio ingresados estén registrados en los mapas de memoria del sistema (`usuarios.containsKey(rut)` y `labs.containsKey(lab)`). Si alguno no existe, rechaza el agendamiento y devuelve un error.
    2.  **Validación de Formato Temporal:** Intenta parsear la fecha con `LocalDate.parse(fecha)`. Si arroja una excepción, significa que la fecha está mal estructurada (por ejemplo, `30-12-2026` en lugar de `2026-12-30`) o que el día es inexistente, deteniendo el registro.
    3.  **Algoritmo de Detección de Colisión (Solapamiento):**
        ```java
        for (Reserva ex : reservas.values()) {
            if (ex.getCodigoLab().equals(lab) && ex.getFecha().equals(fecha) && ex.getBloque().equals(bloque)) {
                return "Conflicto: ya existe una reserva...";
            }
        }
        ```
        Recorre todos los agendamientos almacenados en memoria. Si coincide el código del laboratorio **y** la fecha **y** el bloque de tiempo seleccionado, bloquea la creación de la reserva y retorna un mensaje de conflicto detallado.
    4.  Si supera todas las pruebas, inserta el agendamiento en el mapa `reservas` y persiste el estado completo llamando a `guardarDatos()`.

#### D. Modificación de Reserva (`modificarReserva`):
*   **¿Cómo se hace?** Recibe el ID de una reserva existente junto con los nuevos valores de fecha, bloque y motivo para su actualización.
*   **Funcionamiento:**
    *   Busca la reserva en memoria usando `reservas.get(id)`. Si es nula, finaliza con `false`.
    *   Ejecuta una validación de colisiones similar a la de creación, pero con una excepción crítica:
        `!ex.getId().equals(id)`
        Esto significa: "ignora la reserva actual al buscar colisiones". Si el usuario solo modifica el motivo o la descripción de la reserva sin alterar el horario, el sistema no considerará que la reserva colisiona consigo misma, permitiendo la modificación.
    *   Instancia una nueva reserva con los parámetros actualizados manteniendo el mismo ID y sobreescribe la entrada anterior del mapa (`reservas.put(id, mod)`).

#### E. Consistencia y Mantenimiento Masivo (`limpiarDuplicadosYHuerfanos`):
*   **¿Cómo se hace?** Elimina inconsistencias de datos, tales como múltiples registros con un mismo RUT o reservas asignadas a usuarios eliminados (huérfanos).
*   **Funcionamiento:**
    1.  `eliminarUsuariosDuplicados()`: Recorre los usuarios en orden cronológico e inserta cada uno en un mapa temporal `reconstruido` únicamente si no ha sido insertado antes (`!reconstruido.containsKey(...)`). Si ya estaba, cuenta el registro como duplicado eliminado. Sobrescribe el mapa en RAM con el nuevo mapa limpio.
    2.  `eliminarReservasHuerfanas()`: Recorre las reservas buscando referencias huérfanas: si el `rutUsuario` de la reserva no figura en el mapa de usuarios (`!usuarios.containsKey(...)`), almacena su ID en una lista de purga y la elimina de las reservas del sistema.
    3.  Persiste el estado consistente llamando a `guardarDatos()`.

### 5.3 Glosario de la Capa de Negocio
*   **`Map<K, V>`:** Interfaz de Java que representa un mapa asociativo de clave-valor. Asocia una clave única `K` (como el RUT) con un valor `V` (como el objeto Usuario), permitiendo búsquedas rápidas.
*   **`HashMap`:** Implementación de la interfaz `Map` basada en tablas Hash. No garantiza ningún orden de los elementos. Es extremadamente rápida para buscar, insertar y borrar.
*   **`LinkedHashMap`:** Implementación de `Map` que mantiene una lista doblemente enlazada a través de todos sus elementos. Garantiza que la iteración de los elementos siga exactamente el orden cronológico en que fueron insertados.
*   **`containsKey(Object key)`:** Método de `Map`. Retorna `true` si el mapa contiene una asociación para la clave especificada. Su rendimiento es cercano a tiempo constante $O(1)$.
*   **`put(K key, V value)`:** Método de `Map` para insertar o actualizar un valor `V` asociado a una clave `K`.
*   **`get(Object key)`:** Método de `Map` que retorna el objeto asociado a la clave, o `null` si la clave no existe en la colección.
*   **`remove(Object key)`:** Elimina la asociación de la clave especificada del mapa físico de la memoria.
*   **`stream()`:** API de Java 8 que permite procesar colecciones de datos de forma declarativa mediante flujos de datos continuos.
*   **`map(Function mapper)`:** Operador intermedio de streams. Transforma cada elemento de un flujo aplicando una función (ej: mapear el objeto `Usuario` a su representación de texto `.toString()`).
*   **`collect(Collector)` / `Collectors.toList()`:** Operación terminal de streams. Agrupa el flujo de datos resultante del procesamiento dentro de una nueva colección física.
*   **`forEach(Consumer)`:** Itera sobre todos los elementos de un stream o colección ejecutando la acción programada sobre cada uno de ellos.
*   **`LocalDate`:** Clase del paquete `java.time` que modela fechas de calendario sin zona horaria (formato `yyyy-MM-dd`).
*   **`LocalDate.parse(CharSequence)`:** Analiza sintácticamente un texto para convertirlo en un objeto `LocalDate`. Lanza `DateTimeParseException` si el formato no se ajusta al estándar ISO-8601.
*   **`System.currentTimeMillis()`:** Retorna la fecha y hora actual del sistema representada en milisegundos transcurridos desde el 1 de enero de 1970 (época Unix). Útil para generar nombres únicos de archivos de respaldo.
*   **`String.format(String format, Object... args)`:** Genera una cadena formateada usando marcadores de posición (como `%d` para números enteros o `%s` para textos).

---

## 6. Capa de Presentación de Usuario (`interfaz`)

Esta capa maneja el dibujo de la ventana principal, formularios de pestañas y cuadros de diálogo bloqueantes en AWT puro.

### 6.1 Código de `VentanaPrincipal.java`
Ubicado en `src/interfaz/VentanaPrincipal.java`.

```java
package interfaz;

import java.awt.*;
import java.awt.event.*;
import negocio.SistemaReservas;

public class VentanaPrincipal extends Frame {
    private SistemaReservas sistema;
    private PanelUsuarios panelUsuarios;
    private PanelLaboratorios panelLaboratorios;
    private PanelReservas panelReservas;

    public VentanaPrincipal() {
        super("Sistema Reservas");
        sistema = new SistemaReservas();
        sistema.cargarDatos();          // cargar antes de crear paneles
        sistema.printDebug();          // opcional: ver en consola qué se cargó

        setLayout(new BorderLayout(8,8));

        // Menú
        MenuBar mb = new MenuBar();
        Menu mArchivo = new Menu("Archivo");
        MenuItem miLimpiar = new MenuItem("Limpiar");
        MenuItem miLimpiarDuplicados = new MenuItem("Limpiar duplicados");
        MenuItem miSalir = new MenuItem("Salir");
        mArchivo.add(miLimpiar);
        mArchivo.add(miLimpiarDuplicados);
        mArchivo.addSeparator();
        mArchivo.add(miSalir);
        mb.add(mArchivo);
        setMenuBar(mb);

        // Barra de navegación superior (pestañas)
        Panel navPanel = new Panel(new FlowLayout(FlowLayout.CENTER, 15, 8));
        navPanel.setBackground(new Color(235, 235, 235));

        Button btnNavReservas = new Button("Reservas");
        Button btnNavUsuarios = new Button("Usuarios");
        Button btnNavLaboratorios = new Button("Laboratorios");

        Font fontNav = new Font("SansSerif", Font.BOLD, 13);
        btnNavReservas.setFont(fontNav);
        btnNavUsuarios.setFont(fontNav);
        btnNavLaboratorios.setFont(fontNav);

        Dimension btnSize = new Dimension(150, 32);
        btnNavReservas.setPreferredSize(btnSize);
        btnNavUsuarios.setPreferredSize(btnSize);
        btnNavLaboratorios.setPreferredSize(btnSize);

        navPanel.add(btnNavReservas);
        navPanel.add(btnNavUsuarios);
        navPanel.add(btnNavLaboratorios);
        add(navPanel, BorderLayout.NORTH);

        // Paneles de gestión
        panelUsuarios = new PanelUsuarios(sistema);
        panelLaboratorios = new PanelLaboratorios(sistema);
        panelReservas = new PanelReservas(sistema);

        // Contenedor dinámico (CardLayout)
        CardLayout cl = new CardLayout();
        Panel container = new Panel(cl);
        container.add(panelReservas, "Reservas");
        container.add(panelUsuarios, "Usuarios");
        container.add(panelLaboratorios, "Laboratorios");
        add(container, BorderLayout.CENTER);

        // Lógica de alternancia de pestañas
        btnNavReservas.addActionListener(e -> {
            cl.show(container, "Reservas");
            setButtonActive(btnNavReservas, btnNavUsuarios, btnNavLaboratorios);
        });
        btnNavUsuarios.addActionListener(e -> {
            cl.show(container, "Usuarios");
            setButtonActive(btnNavUsuarios, btnNavReservas, btnNavLaboratorios);
        });
        btnNavLaboratorios.addActionListener(e -> {
            cl.show(container, "Laboratorios");
            setButtonActive(btnNavLaboratorios, btnNavUsuarios, btnNavReservas);
        });

        // Activar pestaña por defecto
        cl.show(container, "Reservas");
        setButtonActive(btnNavReservas, btnNavUsuarios, btnNavLaboratorios);

        // Acciones menú
        miLimpiar.addActionListener(e -> {
            try { panelUsuarios.limpiarCampos(); } catch (Exception ex) {}
            try { panelLaboratorios.limpiarCampos(); panelLaboratorios.refrescarListado(); } catch (Exception ex) {}
            try { panelReservas.limpiarCampos(); panelReservas.refrescarListado(); } catch (Exception ex) {}
        });

        miLimpiarDuplicados.addActionListener(e -> {
            boolean confirmar = UtilConfirm.showConfirm(this, "¿Eliminar RUT duplicados y reservas huérfanas?");
            if (!confirmar) return;
            // respaldo antes de limpiar
            sistema.backupDatos("backup_" + System.currentTimeMillis());
            String resultado = sistema.limpiarDuplicadosYHuerfanos();
            SimpleDialog.show(resultado);
            panelReservas.refrescarListado();
            panelUsuarios.refrescarListado();
            panelLaboratorios.refrescarListado();
            panelUsuarios.limpiarCampos();
            panelLaboratorios.limpiarCampos();
        });

        miSalir.addActionListener(e -> {
            sistema.guardarDatos();
            System.exit(0);
        });

        addWindowListener(new WindowAdapter() {
            public void windowClosing(WindowEvent e) {
                sistema.guardarDatos();
                System.exit(0);
            }
        });

        // Tamaño compacto y centrado
        setSize(760, 500);
        setLocationRelativeTo(null);
    }

    private void setButtonActive(Button activeBtn, Button... inactiveBtns) {
        activeBtn.setBackground(new Color(70, 130, 180)); // Steel Blue
        activeBtn.setForeground(Color.WHITE);
        for (Button b : inactiveBtns) {
            b.setBackground(new Color(225, 225, 225)); // Gris suave
            b.setForeground(Color.BLACK);
        }
    }

    public void refrescarCombosDeReservas() {
        if (panelReservas != null) {
            panelReservas.refrescarCombos();
        }
    }

    // método para lanzar la ventana (opcional)
    public static void main(String[] args) {
        EventQueue.invokeLater(() -> {
            VentanaPrincipal vp = new VentanaPrincipal();
            vp.setVisible(true);
        });
    }
}
```

---

### 6.2 Código de `PanelUsuarios.java`
Ubicado en `src/interfaz/PanelUsuarios.java`.

```java
package interfaz;

import java.awt.*;
import java.awt.event.*;
import negocio.SistemaReservas;
import modelo.Usuario;

public class PanelUsuarios extends Panel {
    private TextField tfRut;
    private TextField tfNombre;
    private Choice choiceTipo;
    private TextField tfCorreo;
    private Button btnRegistrar, btnListar;
    private TextArea taListado;
    private SistemaReservas sistema;

    public PanelUsuarios(SistemaReservas sistema) {
        this.sistema = sistema;
        setLayout(new BorderLayout(6, 6));
        setBackground(new Color(250,250,250));

        // Formulario (GridBagLayout)
        Panel formPanel = new Panel(new GridBagLayout());
        GridBagConstraints c = new GridBagConstraints();
        c.insets = new Insets(3, 4, 3, 4);
        c.fill = GridBagConstraints.HORIZONTAL;

        Label section = new Label("Registro de Usuarios");
        section.setFont(new Font("SansSerif", Font.BOLD, 12));
        c.gridx = 0; c.gridy = 0; c.gridwidth = 2;
        formPanel.add(section, c);

        c.gridwidth = 1;
        c.gridy++;
        c.gridx = 0;
        formPanel.add(new Label("RUT:"), c);
        tfRut = new TextField(12);
        tfRut.setPreferredSize(new Dimension(140, 20));
        c.gridx = 1;
        formPanel.add(tfRut, c);

        c.gridy++;
        c.gridx = 0;
        formPanel.add(new Label("Nombre:"), c);
        tfNombre = new TextField(14);
        tfNombre.setPreferredSize(new Dimension(140, 20));
        c.gridx = 1;
        formPanel.add(tfNombre, c);

        c.gridy++;
        c.gridx = 0;
        formPanel.add(new Label("Tipo:"), c);
        choiceTipo = new Choice();
        choiceTipo.add("estudiante");
        choiceTipo.add("académico");
        choiceTipo.add("funcionario");
        c.gridx = 1;
        formPanel.add(choiceTipo, c);

        c.gridy++;
        c.gridx = 0;
        formPanel.add(new Label("Correo:"), c);
        tfCorreo = new TextField(14);
        tfCorreo.setPreferredSize(new Dimension(140, 20));
        c.gridx = 1;
        formPanel.add(tfCorreo, c);

        c.gridy++;
        c.gridx = 0; c.gridwidth = 2;
        Panel btnPanel = new Panel(new FlowLayout(FlowLayout.CENTER, 6, 0));
        btnRegistrar = new Button("Registrar");
        btnRegistrar.setBackground(new Color(60,179,113));
        btnRegistrar.setForeground(Color.WHITE);
        btnRegistrar.setFont(new Font("SansSerif", Font.PLAIN, 12));
        btnRegistrar.setPreferredSize(new Dimension(100, 26));
        btnPanel.add(btnRegistrar);

        btnListar = new Button("Listar / Ver");
        btnListar.setBackground(new Color(70, 130, 180));
        btnListar.setForeground(Color.WHITE);
        btnListar.setFont(new Font("SansSerif", Font.PLAIN, 12));
        btnListar.setPreferredSize(new Dimension(100, 26));
        btnPanel.add(btnListar);
        
        formPanel.add(btnPanel, c);

        add(formPanel, BorderLayout.NORTH);

        // Listado de usuarios
        Panel listPanel = new Panel(new BorderLayout(4, 4));
        Label lblListado = new Label("Usuarios Registrados:");
        lblListado.setFont(new Font("SansSerif", Font.BOLD, 11));
        listPanel.add(lblListado, BorderLayout.NORTH);

        taListado = new TextArea();
        taListado.setEditable(false);
        listPanel.add(taListado, BorderLayout.CENTER);

        add(listPanel, BorderLayout.CENTER);

        btnRegistrar.addActionListener(new ActionListener() {
            public void actionPerformed(ActionEvent e) {
                String rut = tfRut.getText().trim();
                String nombre = tfNombre.getText().trim();
                String tipo = choiceTipo.getSelectedItem();
                String correo = tfCorreo.getText().trim();

                if (rut.isEmpty() || nombre.isEmpty() || correo.isEmpty()) {
                    SimpleDialog.show("Complete todos los campos");
                    return;
                }
                Usuario u = new Usuario(rut, nombre, tipo, correo);
                boolean ok = sistema.registrarUsuario(u);
                if (ok) {
                    SimpleDialog.show("Usuario registrado");
                    limpiarCampos();
                    refrescarListado();
                    try {
                        Component parent = getParent();
                        while (parent != null && !(parent instanceof VentanaPrincipal)) {
                            parent = parent.getParent();
                        }
                        if (parent != null) {
                            ((VentanaPrincipal) parent).refrescarCombosDeReservas();
                        }
                    } catch (Exception ex) {}
                } else {
                    SimpleDialog.show("No se pudo registrar (RUT duplicado o inválido)");
                }
            }
        });

        btnListar.addActionListener(new ActionListener() {
            public void actionPerformed(ActionEvent e) {
                refrescarListado();
                SimpleDialog.show("Listado de usuarios actualizado.");
            }
        });

        setPreferredSize(new Dimension(250, 380));
        refrescarListado();
    }

    public void refrescarListado() {
        java.util.List<Usuario> lista = sistema.listarUsuarios();
        StringBuilder sb = new StringBuilder();
        for (Usuario u : lista) {
            sb.append(u.getRut()).append(" | ").append(u.getNombre())
              .append(" (").append(u.getTipo()).append(")\n")
              .append("  ").append(u.getCorreo()).append("\n\n");
        }
        taListado.setText(sb.toString());
    }

    public void limpiarCampos() {
        tfRut.setText("");
        tfNombre.setText("");
        tfCorreo.setText("");
        if (choiceTipo.getItemCount() > 0) {
            choiceTipo.select(0);
        }
    }
}
```

---

### 6.3 Código de `PanelLaboratorios.java`
Ubicado en `src/interfaz/PanelLaboratorios.java`.

```java
package interfaz;

import java.awt.*;
import java.awt.event.*;
import negocio.SistemaReservas;
import modelo.Laboratorio;

public class PanelLaboratorios extends Panel {
    private TextField tfCodigo;
    private TextField tfNombre;
    private TextField tfCapacidad;
    private TextField tfUbicacion;
    private TextField tfEquipamiento;
    private Button btnRegistrar, btnListar;
    private TextArea taListado;
    private SistemaReservas sistema;

    public PanelLaboratorios(SistemaReservas sistema) {
        this.sistema = sistema;
        setLayout(new BorderLayout(6, 6));
        setBackground(new Color(250,250,250));

        // Formulario (GridBagLayout)
        Panel formPanel = new Panel(new GridBagLayout());
        GridBagConstraints c = new GridBagConstraints();
        c.insets = new Insets(3, 4, 3, 4);
        c.fill = GridBagConstraints.HORIZONTAL;

        Label section = new Label("Registro de Laboratorios");
        section.setFont(new Font("SansSerif", Font.BOLD, 12));
        c.gridx = 0; c.gridy = 0; c.gridwidth = 2;
        formPanel.add(section, c);

        c.gridwidth = 1;
        c.gridy++;
        c.gridx = 0;
        formPanel.add(new Label("Código:"), c);
        tfCodigo = new TextField(12);
        tfCodigo.setPreferredSize(new Dimension(140, 20));
        c.gridx = 1;
        formPanel.add(tfCodigo, c);

        c.gridy++;
        c.gridx = 0;
        formPanel.add(new Label("Nombre:"), c);
        tfNombre = new TextField(14);
        tfNombre.setPreferredSize(new Dimension(140, 20));
        c.gridx = 1;
        formPanel.add(tfNombre, c);

        c.gridy++;
        c.gridx = 0;
        formPanel.add(new Label("Capacidad:"), c);
        tfCapacidad = new TextField(6);
        tfCapacidad.setPreferredSize(new Dimension(140, 20));
        c.gridx = 1;
        formPanel.add(tfCapacidad, c);

        c.gridy++;
        c.gridx = 0;
        formPanel.add(new Label("Ubicación:"), c);
        tfUbicacion = new TextField(14);
        tfUbicacion.setPreferredSize(new Dimension(140, 20));
        c.gridx = 1;
        formPanel.add(tfUbicacion, c);

        c.gridy++;
        c.gridx = 0;
        formPanel.add(new Label("Equipamiento:"), c);
        tfEquipamiento = new TextField(14);
        tfEquipamiento.setPreferredSize(new Dimension(140, 20));
        c.gridx = 1;
        formPanel.add(tfEquipamiento, c);

        c.gridy++;
        c.gridx = 0; c.gridwidth = 2;
        Panel btnPanel = new Panel(new FlowLayout(FlowLayout.CENTER, 6, 0));
        btnRegistrar = new Button("Registrar");
        btnRegistrar.setBackground(new Color(60,179,113));
        btnRegistrar.setForeground(Color.WHITE);
        btnRegistrar.setFont(new Font("SansSerif", Font.PLAIN, 12));
        btnRegistrar.setPreferredSize(new Dimension(100, 26));
        btnPanel.add(btnRegistrar);

        btnListar = new Button("Listar / Ver");
        btnListar.setBackground(new Color(70, 130, 180));
        btnListar.setForeground(Color.WHITE);
        btnListar.setFont(new Font("SansSerif", Font.PLAIN, 12));
        btnListar.setPreferredSize(new Dimension(100, 26));
        btnPanel.add(btnListar);
        
        formPanel.add(btnPanel, c);

        add(formPanel, BorderLayout.NORTH);

        // Listado de laboratorios
        Panel listPanel = new Panel(new BorderLayout(4, 4));
        Label lblListado = new Label("Laboratorios Registrados:");
        lblListado.setFont(new Font("SansSerif", Font.BOLD, 11));
        listPanel.add(lblListado, BorderLayout.NORTH);

        taListado = new TextArea();
        taListado.setEditable(false);
        listPanel.add(taListado, BorderLayout.CENTER);

        add(listPanel, BorderLayout.CENTER);

        btnRegistrar.addActionListener(new ActionListener() {
            public void actionPerformed(ActionEvent e) {
                String codigo = tfCodigo.getText().trim();
                String nombre = tfNombre.getText().trim();
                String capStr = tfCapacidad.getText().trim();
                String ubicacion = tfUbicacion.getText().trim();
                String equipamiento = tfEquipamiento.getText().trim();

                if (codigo.isEmpty() || nombre.isEmpty() || capStr.isEmpty() || ubicacion.isEmpty() || equipamiento.isEmpty()) {
                    SimpleDialog.show("Complete todos los campos");
                    return;
                }

                int capacidad = 0;
                try {
                    capacidad = Integer.parseInt(capStr);
                } catch (NumberFormatException ex) {
                    SimpleDialog.show("La capacidad debe ser un número entero");
                    return;
                }

                if (capacidad <= 0) {
                    SimpleDialog.show("La capacidad debe ser mayor a 0");
                    return;
                }

                Laboratorio l = new Laboratorio(codigo, nombre, capacidad, ubicacion, equipamiento);
                boolean ok = sistema.registrarLaboratorio(l);
                if (ok) {
                    SimpleDialog.show("Laboratorio registrado");
                    limpiarCampos();
                    refrescarListado();
                    try {
                        Component parent = getParent();
                        while (parent != null && !(parent instanceof VentanaPrincipal)) {
                            parent = parent.getParent();
                        }
                        if (parent != null) {
                            ((VentanaPrincipal) parent).refrescarCombosDeReservas();
                        }
                    } catch (Exception ex) {}
                } else {
                    SimpleDialog.show("No se pudo registrar (Código duplicado o inválido)");
                }
            }
        });

        btnListar.addActionListener(new ActionListener() {
            public void actionPerformed(ActionEvent e) {
                refrescarListado();
                SimpleDialog.show("Listado de laboratorios actualizado.");
            }
        });

        setPreferredSize(new Dimension(250, 380));
        refrescarListado();
    }

    public void refrescarListado() {
        java.util.List<Laboratorio> lista = sistema.listarLaboratorios();
        StringBuilder sb = new StringBuilder();
        for (Laboratorio l : lista) {
            sb.append(l.getCodigo()).append(" | ").append(l.getNombre()).append("\n")
              .append("  Cap: ").append(l.getCapacidad()).append(" | Ubic: ").append(l.getUbicacion()).append("\n")
              .append("  Equip: ").append(l.getEquipamiento()).append("\n\n");
        }
        taListado.setText(sb.toString());
    }

    public void limpiarCampos() {
        tfCodigo.setText("");
        tfNombre.setText("");
        tfCapacidad.setText("");
        tfUbicacion.setText("");
        tfEquipamiento.setText("");
    }
}
```

---

### 6.4 Código de `PanelReservas.java`
Ubicado en `src/interfaz/PanelReservas.java`.

```java
package interfaz;

import negocio.SistemaReservas;
import modelo.*;

import java.awt.*;
import java.awt.event.*;
import java.util.List;
import java.util.ArrayList;
import java.util.UUID;

public class PanelReservas extends Panel {
    private Choice choiceRut, choiceLab, choiceBloque;
    private TextField tfFecha;
    private TextArea taMotivo;
    private java.awt.List lstReservas;
    private Button btnAgendar, btnRefrescar, btnLimpiar, btnModificar, btnEliminar;
    private Choice choiceFiltro;
    private TextField tfFiltro;
    private Button btnFiltrar;
    private SistemaReservas sistema;
    
    // Rastreo de reservas mostradas para mapear index de List a objeto Reserva
    private java.util.List<Reserva> reservasMostradas = new ArrayList<>();
    private String selectedReservaId = null;

    public PanelReservas(SistemaReservas sistema) {
        this.sistema = sistema;
        setLayout(new BorderLayout(6,6));

        // Formulario compacto
        Panel form = new Panel(new GridLayout(5,2,6,6));
        form.add(new Label("Usuario (RUT):"));
        choiceRut = new Choice();
        form.add(choiceRut);
        
        form.add(new Label("Laboratorio:"));
        choiceLab = new Choice();
        form.add(choiceLab);
        
        form.add(new Label("Fecha (yyyy-MM-dd):"));
        tfFecha = new TextField();
        form.add(tfFecha);
        
        form.add(new Label("Bloque:"));
        choiceBloque = new Choice();
        choiceBloque.add("Bloque 1 (08:30 - 10:00)");
        choiceBloque.add("Bloque 2 (10:15 - 11:45)");
        choiceBloque.add("Bloque 3 (12:00 - 13:30)");
        choiceBloque.add("Bloque 4 (14:30 - 16:00)");
        choiceBloque.add("Bloque 5 (16:15 - 17:45)");
        choiceBloque.add("Bloque 6 (18:00 - 19:30)");
        form.add(choiceBloque);
        
        form.add(new Label("Motivo:"));
        taMotivo = new TextArea(2, 20);
        form.add(taMotivo);

        // Botones
        Panel botones = new Panel(new FlowLayout(FlowLayout.LEFT,4,0));
        btnAgendar = new Button("Agendar");
        btnModificar = new Button("Modificar");
        btnEliminar = new Button("Eliminar");
        btnRefrescar = new Button("Refrescar");
        btnLimpiar = new Button("Limpiar");

        btnAgendar.setBackground(new Color(60, 179, 113));
        btnAgendar.setForeground(Color.WHITE);
        btnModificar.setBackground(new Color(70, 130, 180));
        btnModificar.setForeground(Color.WHITE);
        btnEliminar.setBackground(new Color(220, 20, 60));
        btnEliminar.setForeground(Color.WHITE);

        botones.add(btnAgendar);
        botones.add(btnModificar);
        botones.add(btnEliminar);
        botones.add(btnRefrescar);
        botones.add(btnLimpiar);

        // Listado
        lstReservas = new java.awt.List();

        // Filtros
        Panel filterPanel = new Panel(new FlowLayout(FlowLayout.LEFT, 4, 0));
        filterPanel.add(new Label("Filtrar por:"));
        choiceFiltro = new Choice();
        choiceFiltro.add("Todos");
        choiceFiltro.add("Laboratorio");
        choiceFiltro.add("Fecha");
        choiceFiltro.add("Usuario (RUT)");
        filterPanel.add(choiceFiltro);

        tfFiltro = new TextField(10);
        filterPanel.add(tfFiltro);

        btnFiltrar = new Button("Filtrar");
        filterPanel.add(btnFiltrar);

        Panel rightPanel = new Panel(new BorderLayout(4, 4));
        rightPanel.add(filterPanel, BorderLayout.NORTH);
        rightPanel.add(lstReservas, BorderLayout.CENTER);

        // Ensamblaje
        Panel left = new Panel(new BorderLayout(6,6));
        left.add(form, BorderLayout.NORTH);
        left.add(botones, BorderLayout.SOUTH);
        add(left, BorderLayout.WEST);
        add(rightPanel, BorderLayout.CENTER);

        // Listeners
        btnAgendar.addActionListener(e -> onAgendar());
        
        btnRefrescar.addActionListener(e -> {
            choiceFiltro.select(0);
            tfFiltro.setText("");
            refrescarListado();
        });
        
        btnLimpiar.addActionListener(e -> limpiarCampos());
        
        btnFiltrar.addActionListener(e -> refrescarListado());

        // Evento de selección en la lista
        lstReservas.addItemListener(new ItemListener() {
            public void itemStateChanged(ItemEvent e) {
                int idx = lstReservas.getSelectedIndex();
                if (idx >= 0 && idx < reservasMostradas.size()) {
                    Reserva r = reservasMostradas.get(idx);
                    selectedReservaId = r.getId();
                    
                    // Posicionar choiceRut
                    for (int i = 0; i < choiceRut.getItemCount(); i++) {
                        if (choiceRut.getItem(i).startsWith(r.getRutUsuario() + " - ")) {
                            choiceRut.select(i);
                            break;
                        }
                    }
                    
                    // Posicionar choiceLab
                    for (int i = 0; i < choiceLab.getItemCount(); i++) {
                        if (choiceLab.getItem(i).startsWith(r.getCodigoLab() + " - ")) {
                            choiceLab.select(i);
                            break;
                        }
                    }
                    
                    tfFecha.setText(r.getFecha());
                    
                    // Posicionar choiceBloque
                    for (int i = 0; i < choiceBloque.getItemCount(); i++) {
                        if (choiceBloque.getItem(i).equals(r.getBloque())) {
                            choiceBloque.select(i);
                            break;
                        }
                    }
                    
                    taMotivo.setText(r.getMotivo());
                }
            }
        });

        // Evento modificar
        btnModificar.addActionListener(new ActionListener() {
            public void actionPerformed(ActionEvent e) {
                if (selectedReservaId == null) {
                    SimpleDialog.show("Seleccione una reserva de la lista para modificar");
                    return;
                }
                String fecha = tfFecha.getText().trim();
                String bloque = choiceBloque.getSelectedItem();
                String motivo = taMotivo.getText().trim();

                if (fecha.isEmpty() || bloque == null || bloque.isEmpty()) {
                    SimpleDialog.show("La fecha y el bloque son obligatorios");
                    return;
                }

                try {
                    java.time.LocalDate.parse(fecha);
                } catch (Exception ex) {
                    SimpleDialog.show("Formato de fecha inválido (usar yyyy-MM-dd)");
                    return;
                }

                boolean ok = sistema.modificarReserva(selectedReservaId, fecha, bloque, motivo);
                if (ok) {
                    SimpleDialog.show("Reserva modificada correctamente");
                    limpiarCampos();
                    refrescarListado();
                } else {
                    SimpleDialog.show("No se pudo modificar (conflicto de laboratorio, fecha o bloque)");
                }
            }
        });

        // Evento eliminar
        btnEliminar.addActionListener(new ActionListener() {
            public void actionPerformed(ActionEvent e) {
                if (selectedReservaId == null) {
                    SimpleDialog.show("Seleccione una reserva de la lista para eliminar");
                    return;
                }

                Frame parent = getParentFrame();
                boolean confirmar = UtilConfirm.showConfirm(parent, "¿Eliminar la reserva seleccionada?");
                if (!confirmar) return;

                boolean ok = sistema.eliminarReserva(selectedReservaId);
                if (ok) {
                    SimpleDialog.show("Reserva de usuario eliminada correctamente");
                    limpiarCampos();
                    refrescarListado();
                } else {
                    SimpleDialog.show("No se pudo eliminar la reserva");
                }
            }
        });

        refrescarCombos();
        refrescarListado();
    }

    private void onAgendar() {
        String rutSel = choiceRut.getSelectedItem();
        String labSel = choiceLab.getSelectedItem();
        String fecha = tfFecha.getText().trim();
        String bloque = choiceBloque.getSelectedItem();
        String motivo = taMotivo.getText().trim();

        if (rutSel == null || labSel == null || fecha.isEmpty() || bloque == null || bloque.isEmpty()) {
            SimpleDialog.show("Complete los campos obligatorios: RUT, laboratorio, fecha, bloque.");
            return;
        }

        String rut = rutSel.split(" - ")[0].trim();
        String lab = labSel.split(" - ")[0].trim();

        Reserva r = new Reserva(UUID.randomUUID().toString(), rut, lab, fecha, bloque, motivo);

        // Depuración en consola
        System.out.println("[Reserva] Intento: " + r.toString());

        String resultado = sistema.intentarCrearReserva(r);

        System.out.println("[Reserva] Result: " + resultado);

        if ("OK".equals(resultado)) {
            SimpleDialog.show("Reserva creada correctamente.");
            refrescarListado();
            limpiarCampos();
        } else {
            SimpleDialog.show("No se pudo crear la reserva:\n" + resultado);
        }
    }

    public void refrescarCombos() {
        choiceRut.removeAll();
        for (Usuario u : sistema.listarUsuarios()) {
            choiceRut.add(u.getRut() + " - " + u.getNombre());
        }

        choiceLab.removeAll();
        for (Laboratorio l : sistema.listarLaboratorios()) {
            choiceLab.add(l.getCodigo() + " - " + l.getNombre());
        }
    }

    public void refrescarListado() {
        lstReservas.removeAll();
        reservasMostradas.clear();
        selectedReservaId = null;

        List<Reserva> lista = sistema.listarReservas();
        
        String tipoFiltro = choiceFiltro != null ? choiceFiltro.getSelectedItem() : "Todos";
        String textoFiltro = tfFiltro != null ? tfFiltro.getText().trim().toLowerCase() : "";

        for (Reserva r : lista) {
            boolean matches = true;
            if (!textoFiltro.isEmpty()) {
                if ("Laboratorio".equals(tipoFiltro)) {
                    matches = r.getCodigoLab().toLowerCase().contains(textoFiltro);
                } else if ("Fecha".equals(tipoFiltro)) {
                    matches = r.getFecha().toLowerCase().contains(textoFiltro);
                } else if ("Usuario (RUT)".equals(tipoFiltro)) {
                    matches = r.getRutUsuario().toLowerCase().contains(textoFiltro);
                }
            }

            if (matches) {
                reservasMostradas.add(r);
                String displayString = String.format("%s | RUT: %s | Lab: %s | %s | Blq: %s",
                        r.getId().substring(0, 8) + "...", r.getRutUsuario(), r.getCodigoLab(), r.getFecha(), r.getBloque());
                lstReservas.add(displayString);
            }
        }
    }

    public void limpiarCampos() {
        if (choiceRut.getItemCount() > 0) choiceRut.select(0);
        if (choiceLab.getItemCount() > 0) choiceLab.select(0);
        tfFecha.setText("");
        if (choiceBloque.getItemCount() > 0) choiceBloque.select(0);
        taMotivo.setText("");
        selectedReservaId = null;
        try {
            int selectedIdx = lstReservas.getSelectedIndex();
            if (selectedIdx >= 0) {
                lstReservas.deselect(selectedIdx);
            }
        } catch (Exception ex) {}
    }

    private Frame getParentFrame() {
        Component c = this;
        while (c != null && !(c instanceof Frame)) {
            c = c.getParent();
        }
        return (Frame) c;
    }
}
```

### 6.5 Explicación del Funcionamiento de la UI
*   **¿Cómo funciona la navegación de pestañas?** `VentanaPrincipal` inicializa el diseño `CardLayout` en el contenedor de paneles `container`. Al pulsar un botón de la barra superior, el gestor de diseño esconde la tarjeta actual y despliega la tarjeta requerida por su identificador de texto.
*   **¿Cómo se actualizan los datos de reservas al crear un usuario o laboratorio?**
    *   `PanelUsuarios` tiene lógica en su botón "Registrar" para realizar una búsqueda interactiva subiendo por la jerarquía visual de componentes gráficos llamando a `.getParent()` hasta localizar a la clase padre `VentanaPrincipal`.
    *   Una vez encontrada la ventana, la castea y ejecuta `refrescarCombosDeReservas()`, provocando que `PanelReservas` reconstruya sus selectores Choice con la base de datos actualizada al instante.
*   **¿Cómo se selecciona una reserva para borrarla o modificarla?**
    *   `lstReservas` de tipo `java.awt.List` lanza eventos de selección de elementos.
    *   Se captura la fila clickeada (`lstReservas.getSelectedIndex()`) y se mapea con la misma posición del arreglo `reservasMostradas` para extraer el objeto `Reserva` real.
    *   Esto permite autocompletar el formulario y guardar el identificador UUID físico de la reserva seleccionada en la variable `selectedReservaId` para modificarla o eliminarla.

### 6.6 Glosario de la Capa de Interfaz
*   **`Frame`:** Ventana principal de escritorio nativa e independiente que posee borde, barra de título y soporte para menús.
*   **`Panel`:** Componente contenedor simple que agrupa lógicamente otros componentes de UI en zonas específicas. No tiene barra de título ni bordes propios.
*   **`CardLayout`:** Administrador de diseño que trata cada componente hijo del panel como una tarjeta de juego de mesa. Solo una tarjeta es visible en pantalla a la vez.
*   **`BorderLayout`:** Distribuye el espacio de un contenedor en cinco zonas cardinales: `NORTH`, `SOUTH`, `EAST`, `WEST` y `CENTER`.
*   **`FlowLayout`:** Alinea los componentes gráficos en una fila horizontal consecutiva. Si el espacio no es suficiente, salta a la siguiente línea de forma automática.
*   **`GridBagLayout` / `GridBagConstraints`:** El administrador de diseño más complejo de AWT. Permite alinear componentes en rejillas de coordenadas flexibles, modificando márgenes con `Insets` y espaciados dinámicos.
*   **`Button` / `Label` / `TextField` / `TextArea`:** Componentes de interfaz AWT básicos para disparar eventos de clic, mostrar textos estáticos, ingresar líneas de entrada de texto cortas y escribir bloques de texto multilinea, respectivamente.
*   **`Choice`:** Componente de menú desplegable (ComboBox). El usuario selecciona una opción de una lista cerrada, previniendo errores de tipeo o inserción de datos.
*   **`java.awt.List`:** Lista interactiva de desplazamiento vertical que muestra múltiples filas de texto y permite asociar eventos al seleccionar cualquiera de ellas.
*   **`ActionListener` / `ActionEvent`:** Interfaz de escucha que atiende eventos de acción general, tales como hacer clic en un botón gráfico o seleccionar un elemento del menú.
*   **`ItemListener` / `ItemEvent`:** Escuchador específico para monitorear eventos de selección o deselección de elementos en listas o casillas de verificación.
*   **`WindowAdapter` / `WindowEvent`:** Clase de ayuda que simplifica la implementación de eventos del ciclo de vida de las ventanas (abrir, cerrar, minimizar), permitiendo implementar solo los métodos necesarios en lugar de toda la interfaz `WindowListener`.
*   **`dispose()`:** Destruye físicamente la ventana y libera los recursos de memoria y del procesador gráfico asignados por el sistema operativo.
*   **`getParent()`:** Método heredado de `Component`. Devuelve el contenedor padre directo del elemento visual, facilitando el recorrido ascendente por el árbol visual de componentes.
*   **`varargs` (`Tipo... nombre`):** Parámetro de longitud variable en Java. Permite pasar múltiples argumentos separados por comas del mismo tipo a una función, la cual los procesa internamente como un arreglo estándar.
*   **Expresión Lambda (`(parametros) -> { codigo }`):** Forma ultra compacta introducida en Java 8 para representar interfaces funcionales de un solo método, facilitando el diseño y lectura de los manejadores de eventos.
*   **`EventQueue.invokeLater(Runnable)`:** Envía la inicialización de la ventana gráfica al final de la cola de eventos del hilo especial de despacho de AWT (Event Dispatch Thread - EDT). Garantiza que la interfaz gráfica no sufra bloqueos por concurrencia al iniciarse.

---

## 7. Componentes de Diálogo Auxiliares

### 7.1 Código de `SimpleDialog.java`
Ubicado en `src/interfaz/SimpleDialog.java`.

```java
package interfaz;

import java.awt.*;
import java.awt.event.*;

public class SimpleDialog {
    public static void show(String msg) {
        Frame f = new Frame();
        Dialog d = new Dialog(f, "Mensaje", true);
        d.setLayout(new BorderLayout());
        d.add(new Label(msg), BorderLayout.CENTER);
        Panel p = new Panel();
        Button ok = new Button("OK");
        ok.addActionListener(new ActionListener() {
            public void actionPerformed(ActionEvent e) { d.dispose(); f.dispose(); }
        });
        p.add(ok);
        d.add(p, BorderLayout.SOUTH);
        d.setSize(360,160);
        d.setLocationRelativeTo(null);
        d.setVisible(true);
    }
}
```

---

### 7.2 Código de `UtilConfirm.java`
Ubicado en `src/interfaz/UtilConfirm.java`.

```java
package interfaz;

import java.awt.*;
import java.awt.event.*;

public class UtilConfirm {
    public static boolean showConfirm(Frame owner, String msg) {
        Dialog d = new Dialog(owner, "Confirmar", true);
        d.setLayout(new BorderLayout());
        d.add(new Label(msg), BorderLayout.CENTER);
        Panel p = new Panel();
        final boolean[] res = {false};
        Button yes = new Button("Sí");
        Button no = new Button("No");
        yes.addActionListener(e -> { res[0] = true; d.dispose(); });
        no.addActionListener(e -> { res[0] = false; d.dispose(); });
        p.add(yes); p.add(no);
        d.add(p, BorderLayout.SOUTH);
        d.setSize(360,160);
        d.setLocationRelativeTo(owner);
        d.setVisible(true);
        return res[0];
    }
}
```

### 7.3 Explicación del Funcionamiento de los Diálogos
*   **¿Cómo bloquean la interfaz al mostrarse?** Ambas clases utilizan la propiedad `modal = true` al instanciar el componente `java.awt.Dialog`. Al hacer esto, Java interrumpe el flujo del programa en la línea `setVisible(true)` y bloquea la interacción con las demás ventanas del usuario, forzándolo a responder antes de continuar con la ejecución de cualquier otra acción en la interfaz gráfica principal.
*   **¿Cómo devuelven la respuesta del usuario?** 
    *   En `UtilConfirm.showConfirm`, se utiliza un arreglo de un solo elemento `final boolean[] res = {false}`.
    *   Al hacer clic en "Sí" o "No", se cambia el valor del arreglo en la posición cero (`res[0]`) y se destruye el diálogo.
    *   Al destruirse el diálogo modal, la ejecución bloqueada en `setVisible(true)` continúa, retornando la respuesta del usuario almacenada en `res[0]`.

### 7.4 Glosario de los Diálogos
*   **`Dialog`:** Ventana secundaria emergente bloqueante o informativa de menor jerarquía que el `Frame`.
*   **Modalidad (`modal`):** Propiedad de un cuadro de diálogo. Si es modal, bloquea la entrada del ratón y teclado hacia todas las demás ventanas de la aplicación hasta que sea cerrado.
*   **`setLocationRelativeTo(Component c)`:** Método heredado de `Window`. Ajusta la posición de despliegue de la ventana en pantalla. Si se le pasa `null`, la ventana emergente aparecerá perfectamente centrada en el escritorio físico del usuario.
