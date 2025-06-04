El patrón **Factory Method** (Método de Fábrica) es un patrón de diseño creacional que te permite **crear objetos sin especificar la clase exacta del objeto que será creado**. En lugar de usar el operador `new` directamente, delegas la creación de objetos a un método de fábrica. Este método de fábrica puede ser implementado por subclases para crear diferentes tipos de objetos.

---

### Analogía del Factory Method: Una Cadena de Restaurantes de Comida Rápida

Imaginá una cadena de restaurantes de comida rápida como "Burger House". No todos los restaurantes de la cadena son idénticos; algunos solo sirven hamburguesas, otros también tienen opciones vegetarianas, y otros incluso ofrecen comida de mar.

* **Definición:** "Burger House" es el **Creador** (la clase que declara el método de fábrica). Las hamburguesas, ensaladas o pescados son los **Productos** (los objetos que se van a crear). El método para "preparar una comida" es el **Método de Fábrica**.
* **Creación Abierta a Variaciones:** Vos, como cliente, vas a cualquier "Burger House" y pedís "una comida". No te importa si es una hamburguesa o una ensalada. Internamente, cada sucursal de "Burger House" (que serían las **Subclases Creadoras Concretas**) sabe qué tipo de comida preparar para satisfacer ese pedido:
    * La sucursal "Burger House Clásica" tiene un método de fábrica que siempre produce una **Hamburguesa**.
    * La sucursal "Burger House Saludable" tiene un método de fábrica que produce una **Ensalada Vegetariana**.
    * La sucursal "Burger House Costera" tiene un método de fábrica que produce un **Filete de Pescado**.
* **Delegación de la Creación:** Vos, como cliente (el código cliente), interactuás con la interfaz genérica de "Burger House" (el Creador). No tenés que saber si estás en la sucursal clásica o la saludable. La sucursal es quien decide *qué* objeto concreto (Hamburguesa, Ensalada, Pescado) debe instanciar, basándose en su propia lógica interna o en la configuración.

El patrón Factory Method aplica esta misma lógica al software: una clase (el Creador) declara un método para crear objetos, pero son sus subclases (Creadores Concretos) quienes deciden qué clase concreta de objeto (el Producto Concreto) instanciar. Esto te da flexibilidad y permite que tu código trabaje con interfaces o clases abstractas de productos, sin depender de sus implementaciones concretas.

---

### ¿Qué problema resuelve el Factory Method para un Logger en una aplicación?

El patrón Factory Method, aplicado a un Logger (sistema de registro de eventos), resuelve el problema de **cómo instanciar diferentes tipos de loggers (ej. log a archivo, log a base de datos, log a consola) sin acoplar el código que necesita registrar mensajes a las clases concretas de esos loggers**.

Consideremos los siguientes puntos clave:

* **Extensibilidad y Flexibilidad:** Una aplicación puede necesitar registrar mensajes en diferentes destinos (un archivo, una base de datos, la consola, un servicio en la nube). Sin el Factory Method, cada vez que quisieras cambiar el tipo de logger, tendrías que modificar el código en todos los lugares donde se instanciaba el logger (usando `new FileLogger()`, `new DatabaseLogger()`, etc.). Con el Factory Method, solo modificarías la implementación del método de fábrica, y el resto del código permanecería intacto.
* **Reducción del Acoplamiento:** El código que registra mensajes (el "cliente" del logger) interactúa únicamente con una interfaz o clase abstracta de logger (el `LoggerInterface`). No necesita saber si el logger real es un `FileLogger` o un `DatabaseLogger`. Esto hace que tu código sea más modular y fácil de mantener.
* **Delegación de la Responsabilidad de Creación:** La lógica para decidir qué tipo de logger crear se encapsula dentro del método de fábrica. Esto puede basarse en la configuración de la aplicación, el entorno, o cualquier otra regla de negocio, sin que el cliente del logger necesite conocer esa lógica.
* **Manejo de Dependencias:** Si un logger necesita dependencias complejas (ej. el `DatabaseLogger` necesita una conexión a la base de datos), el Factory Method puede gestionar esas dependencias internamente durante la creación, manteniendo el cliente simple.

---

### Código Simple para entender el concepto (Logger Básico)

Este ejemplo muestra cómo un `LoggerFactory` puede crear diferentes tipos de `Logger` (simulados).

```php
<?php

// --- Parte 1: Producto (Interfaz y Productos Concretos) ---

// Interfaz Product: Define la interfaz común para todos los loggers.
interface Logger
{
    public function log(string $message): void;
}

// Concrete Product A: Un logger que escribe en un archivo.
class FileLogger implements Logger
{
    public function log(string $message): void
    {
        echo "LOG al archivo: " . $message . "\n";
        // En un caso real, aquí iría el código para escribir en un archivo.
        // file_put_contents('app.log', $message . "\n", FILE_APPEND);
    }
}

// Concrete Product B: Un logger que escribe en la consola.
class ConsoleLogger implements Logger
{
    public function log(string $message): void
    {
        echo "LOG en consola: " . $message . "\n";
    }
}

// --- Parte 2: Creador (Clase Abstracta y Creadores Concretos) ---

// Creador Abstracto: Declara el método de fábrica que debe ser implementado por las subclases.
abstract class LoggerFactory
{
    // El "Factory Method" (Método de Fábrica)
    abstract public function createLogger(): Logger;

    // También puede contener lógica de negocio que utiliza el logger.
    public function someApplicationLogic(string $data): void
    {
        $logger = $this->createLogger(); // Usa el método de fábrica para obtener un logger.
        $logger->log("Procesando datos: " . $data);
        // Más lógica de la aplicación...
    }
}

// Concrete Creator A: Implementa el método de fábrica para crear un FileLogger.
class FileLoggerFactory extends LoggerFactory
{
    public function createLogger(): Logger
    {
        return new FileLogger();
    }
}

// Concrete Creator B: Implementa el método de fábrica para crear un ConsoleLogger.
class ConsoleLoggerFactory extends LoggerFactory
{
    public function createLogger(): Logger
    {
        return new ConsoleLogger();
    }
}

// --- Parte 3: Uso en tu aplicación (el "Cliente") ---

echo "--- Usando el Logger de Archivo ---\n";
$fileLoggerCreator = new FileLoggerFactory();
$fileLoggerCreator->someApplicationLogic("Datos importantes para guardar en archivo");
// También puedes obtener el logger directamente:
$anotherFileLogger = $fileLoggerCreator->createLogger();
$anotherFileLogger->log("Otro mensaje para el archivo");


echo "\n--- Usando el Logger de Consola ---\n";
$consoleLoggerCreator = new ConsoleLoggerFactory();
$consoleLoggerCreator->someApplicationLogic("Mensaje de depuración en consola");
// O directamente:
$anotherConsoleLogger = $consoleLoggerCreator->createLogger();
$anotherConsoleLogger->log("Otro mensaje en la consola");

echo "\n--- Cambiando el tipo de logger dinámicamente (en un escenario real) ---\n";
// En una aplicación real, podrías decidir qué fábrica usar basándote en una configuración.
// Por ejemplo, si una variable de entorno 'APP_ENV' es 'production', usar FileLoggerFactory.
// Si es 'development', usar ConsoleLoggerFactory.

$appEnv = 'production'; // Simulando el entorno de la aplicación

if ($appEnv === 'production') {
    $currentLoggerCreator = new FileLoggerFactory();
    echo "Configuración: Modo de producción, usando FileLogger.\n";
} else {
    $currentLoggerCreator = new ConsoleLoggerFactory();
    echo "Configuración: Modo de desarrollo, usando ConsoleLogger.\n";
}

$currentLoggerCreator->someApplicationLogic("Un mensaje que va al logger según la configuración");

?>
```

---

### Código más avanzado (Logger con Niveles y Múltiples Destinos)

Este ejemplo muestra un Factory Method más sofisticado que puede crear loggers con diferentes niveles de severidad y que potencialmente envían mensajes a diferentes destinos, encapsulando la configuración.

```php
<?php

// --- Parte 1: Producto (Interfaz y Productos Concretos) ---

// Enum para niveles de logging (PHP 8.1+)
enum LogLevel: string
{
    case INFO = 'INFO';
    case WARNING = 'WARNING';
    case ERROR = 'ERROR';
    case DEBUG = 'DEBUG';
}

// Interfaz del Logger
interface LoggerAdvanced
{
    public function log(LogLevel $level, string $message): void;
}

// Concrete Product A: Logger de Archivo que filtra por nivel
class AdvancedFileLogger implements LoggerAdvanced
{
    private string $filePath;
    private LogLevel $minLevel;

    public function __construct(string $filePath, LogLevel $minLevel = LogLevel::INFO)
    {
        $this->filePath = $filePath;
        $this->minLevel = $minLevel;
    }

    public function log(LogLevel $level, string $message): void
    {
        // Solo registra si el nivel del mensaje es igual o superior al nivel mínimo configurado
        if ($level->isGreaterThanOrEqualTo($this->minLevel)) {
            $formattedMessage = sprintf("[%s] [%s] %s - %s\n",
                date('Y-m-d H:i:s'),
                $level->value,
                $message,
                $_SERVER['REMOTE_ADDR'] ?? 'CLI' // Ejemplo: añadir IP si existe
            );
            file_put_contents($this->filePath, $formattedMessage, FILE_APPEND);
            echo "Archivo LOG: " . $formattedMessage; // Para demostración en consola
        }
    }
}

// Agregamos un método a la enum LogLevel para comparar niveles
// Esto se haría en un trait o una clase de utilidad si no fuera PHP 8.1+
// Para PHP 8.1+ se puede añadir directamente en el enum
if (!method_exists(LogLevel::class, 'isGreaterThanOrEqualTo')) {
    function isGreaterThanOrEqualTo(LogLevel $self, LogLevel $other): bool {
        $levels = [
            LogLevel::DEBUG->value => 1,
            LogLevel::INFO->value => 2,
            LogLevel::WARNING->value => 3,
            LogLevel::ERROR->value => 4,
        ];
        return $levels[$self->value] >= $levels[$other->value];
    }
    // Añadimos el método a LogLevel de forma dinámica (solo para demo, no es práctica común)
    // En un proyecto real, se usaría un paquete de librerías o se definiría la lógica de comparación
    // de otra manera para los enums si no se desea modificar la clase directamente.
    // Esto es un hack para la demostración.
    // Se recomienda usar un helper externo o un diseño más robusto.
    // Para una aplicación real, se podría definir un orden numérico en el enum.
    // Ej: enum LogLevel: int { case DEBUG = 1; case INFO = 2; ... }
    // y luego comparar $level->value >= $this->minLevel->value;
}


// Concrete Product B: Logger de Base de Datos
class DatabaseLogger implements LoggerAdvanced
{
    private PDO $pdo;
    private LogLevel $minLevel;

    public function __construct(PDO $pdo, LogLevel $minLevel = LogLevel::WARNING)
    {
        $this->pdo = $pdo;
        $this->minLevel = $minLevel;
        $this->ensureTableExists();
    }

    private function ensureTableExists(): void
    {
        $sql = "CREATE TABLE IF NOT EXISTS application_logs (
            id INT AUTO_INCREMENT PRIMARY KEY,
            level VARCHAR(20) NOT NULL,
            message TEXT NOT NULL,
            timestamp DATETIME DEFAULT CURRENT_TIMESTAMP
        )";
        $this->pdo->exec($sql);
    }

    public function log(LogLevel $level, string $message): void
    {
        if ($level->isGreaterThanOrEqualTo($this->minLevel)) {
            try {
                $stmt = $this->pdo->prepare("INSERT INTO application_logs (level, message) VALUES (:level, :message)");
                $stmt->bindValue(':level', $level->value);
                $stmt->bindValue(':message', $message);
                $stmt->execute();
                echo "DB LOG: " . $level->value . " - " . $message . " (Guardado en DB)\n";
            } catch (PDOException $e) {
                error_log("Error al registrar en DB: " . $e->getMessage());
                echo "DB LOG ERROR: Falló al guardar en DB.\n";
            }
        }
    }
}

// --- Parte 2: Creador (Clase Abstracta y Creadores Concretos) ---

// Creador Abstracto: Define el Factory Method
abstract class LoggerFactoryAdvanced
{
    abstract public function createLogger(array $config = []): LoggerAdvanced;

    // Lógica de negocio que usa el logger (la misma para todas las subclases)
    public function doSomethingThatNeedsLogging(LogLevel $level, string $activity): void
    {
        $logger = $this->createLogger(); // El método de fábrica es llamado aquí.
        $logger->log($level, "Actividad: " . $activity);
    }
}

// Concrete Creator A: Fábrica para el Logger de Archivo
class FileLoggerFactoryAdvanced extends LoggerFactoryAdvanced
{
    private string $defaultFilePath;
    private LogLevel $defaultMinLevel;

    public function __construct(string $defaultFilePath = 'app_advanced.log', LogLevel $defaultMinLevel = LogLevel::INFO)
    {
        $this->defaultFilePath = $defaultFilePath;
        $this->defaultMinLevel = $defaultMinLevel;
    }

    public function createLogger(array $config = []): LoggerAdvanced
    {
        $filePath = $config['path'] ?? $this->defaultFilePath;
        $minLevel = $config['min_level'] ?? $this->defaultMinLevel;
        return new AdvancedFileLogger($filePath, $minLevel);
    }
}

// Concrete Creator B: Fábrica para el Logger de Base de Datos
class DatabaseLoggerFactoryAdvanced extends LoggerFactoryAdvanced
{
    private PDO $pdoConnection; // La conexión PDO se inyecta aquí.
    private LogLevel $defaultMinLevel;

    public function __construct(PDO $pdo, LogLevel $defaultMinLevel = LogLevel::WARNING)
    {
        $this->pdoConnection = $pdo;
        $this->defaultMinLevel = $defaultMinLevel;
    }

    public function createLogger(array $config = []): LoggerAdvanced
    {
        $minLevel = $config['min_level'] ?? $this->defaultMinLevel;
        return new DatabaseLogger($this->pdoConnection, $minLevel);
    }
}

// --- Parte 3: Uso en la aplicación (el "Cliente") ---

// Configuración de la base de datos para el DatabaseLogger
$dsn = 'mysql:host=localhost;dbname=test_db;charset=utf8mb4';
$user = 'root';
$password = 'password'; // ¡Cambia esto a tu contraseña!
try {
    $pdo = new PDO($dsn, $user, $password, [PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION]);
    // Definimos la función isGreaterThanOrEqualTo en LogLevel para que el ejemplo funcione.
    // En un entorno real, esto se manejaría de forma diferente (ej. un trait o una clase helper).
    if (!method_exists(LogLevel::class, 'isGreaterThanOrEqualTo')) {
        eval('namespace { enum LogLevel: string { case INFO = "INFO"; case WARNING = "WARNING"; case ERROR = "ERROR"; case DEBUG = "DEBUG"; public function isGreaterThanOrEqualTo(LogLevel $other): bool { $levels = [self::DEBUG->value => 1, self::INFO->value => 2, self::WARNING->value => 3, self::ERROR->value => 4,]; return $levels[$this->value] >= $levels[$other->value]; } } }');
    }

} catch (PDOException $e) {
    die("No se pudo conectar a la base de datos para el DatabaseLogger: " . $e->getMessage());
}


echo "--- Usando File Logger (Nivel INFO) ---\n";
$fileLoggerFactory = new FileLoggerFactoryAdvanced('app_info.log', LogLevel::INFO);
$fileLoggerFactory->doSomethingThatNeedsLogging(LogLevel::INFO, "Usuario 'admin' inició sesión.");
$fileLoggerFactory->doSomethingThatNeedsLogging(LogLevel::DEBUG, "Debug info: Array de configuración cargado."); // No se registra si minLevel es INFO

echo "\n--- Usando File Logger (Nivel DEBUG) ---\n";
// Sobrescribir la configuración predeterminada del Factory
$fileLoggerFactoryDebug = new FileLoggerFactoryAdvanced();
$debugLogger = $fileLoggerFactoryDebug->createLogger(['path' => 'app_debug.log', 'min_level' => LogLevel::DEBUG]);
$debugLogger->log(LogLevel::DEBUG, "Este mensaje de depuración ahora sí se registra.");
$debugLogger->log(LogLevel::INFO, "Este mensaje INFO también se registra.");

echo "\n--- Usando Database Logger (Nivel WARNING) ---\n";
$dbLoggerFactory = new DatabaseLoggerFactoryAdvanced($pdo, LogLevel::WARNING);
$dbLoggerFactory->doSomethingThatNeedsLogging(LogLevel::INFO, "Producto 'X' añadido al carrito."); // No se registra
$dbLoggerFactory->doSomethingThatNeedsLogging(LogLevel::WARNING, "Advertencia: Bajo stock para 'Producto Y'."); // Se registra
$dbLoggerFactory->doSomethingThatNeedsLogging(LogLevel::ERROR, "ERROR: Fallo en el pago del usuario 'Z'."); // Se registra

echo "\n--- Configuración dinámica de Logger ---\n";
$appMode = 'development'; // Podría venir de un archivo de configuración o variable de entorno

$loggerInstance = null;
if ($appMode === 'production') {
    $factory = new FileLoggerFactoryAdvanced('production.log', LogLevel::ERROR);
    $loggerInstance = $factory->createLogger();
    echo "Aplicación en modo PRODUCCIÓN. Logger de Errores a archivo.\n";
} else {
    $factory = new ConsoleLoggerFactory(); // Podríamos tener un ConsoleLoggerFactoryAdvanced si queremos niveles
    $loggerInstance = $factory->createLogger(); // Usamos el simple ConsoleLogger del primer ejemplo por simplicidad
    echo "Aplicación en modo DESARROLLO. Logger a consola.\n";
}

$loggerInstance->log(LogLevel::INFO, "Mensaje general de la aplicación.");
$loggerInstance->log(LogLevel::ERROR, "Un error crítico ha ocurrido!");

?>
```

---

### Resumen al Respecto (Pros y Contras del Factory Method para un Logger)

El patrón **Factory Method** en PHP te permite **crear objetos (productos) a través de un método especializado (el método de fábrica)** definido en una clase Creadora. Las subclases de este Creador implementan el método de fábrica para decidir qué clase concreta de Producto instanciar. Para un Logger, esto significa que puedes cambiar fácilmente el tipo de logger (archivo, base de datos, consola, etc.) sin modificar el código que *usa* el logger.

#### **Pros (Ventajas):**

* **Mayor Flexibilidad y Extensibilidad:** Es la ventaja principal. Puedes introducir nuevos tipos de loggers (ej. `CloudLogger`, `SlackLogger`) sin necesidad de modificar las clases existentes que utilizan el logger. Solo necesitas crear una nueva subclase de `LoggerFactory` y una nueva clase `Logger`.
* **Reducción del Acoplamiento:** El código cliente (donde llamas a `log()`) depende únicamente de la interfaz `Logger` (o `LoggerAdvanced`), no de sus implementaciones concretas (`FileLogger`, `DatabaseLogger`). Esto hace que el sistema sea más modular y fácil de mantener.
* **Encapsulación de la Lógica de Creación:** La lógica compleja para instanciar un logger (qué parámetros necesita, cómo manejar dependencias como una conexión PDO) está encapsulada dentro del método de fábrica en cada `LoggerFactory` concreto. El cliente no necesita saber estos detalles.
* **Facilita la Configuración Dinámica:** Puedes decidir qué tipo de logger usar en tiempo de ejecución (por ejemplo, basándote en un archivo de configuración, variables de entorno, o el perfil de usuario), simplemente instanciando la fábrica adecuada.
* **Cumple el Principio de Abierto/Cerrado (Open/Closed Principle - OCP):** Puedes extender el sistema con nuevos loggers sin modificar el código existente. Es "abierto para extensión" pero "cerrado para modificación".

#### **Contras (Desventajas):**

* **Aumento de la Complejidad Inicial:** Para un sistema de logging muy simple (donde solo hay un tipo de logger y nunca cambiará), el Factory Method añade un nivel de abstracción y más clases de las que serían estrictamente necesarias.
* **Más Clases (Sutilmente):** Requiere la creación de al menos dos nuevas jerarquías (una para los productos/loggers y otra para los creadores/fábricas), lo que aumenta el número de archivos en el proyecto. Sin embargo, esta "sobrecarga" es a menudo un precio que vale la pena pagar por la flexibilidad.
* **Puede Ser Confuso para Problemas Simples:** Si el problema de creación es trivial (ej. siempre crear el mismo tipo de objeto sin variaciones), el Factory Method podría ser excesivo y un simple método estático en una clase `Logger` podría bastar (`Logger::createFileLogger()`).

En resumen, el patrón Factory Method es una excelente elección para un sistema de logging en PHP cuando esperas tener **múltiples tipos de loggers** en el futuro o cuando la **lógica para decidir qué logger usar es compleja y/o dinámica**. Te permite construir un sistema de registro robusto, extensible y fácil de mantener, reduciendo el acoplamiento entre los componentes de tu aplicación.

### Prompt

Necesito que expliques el patrón Factory Method (teniendo en cuenta que se va aplicar en PHP), primero con la definición concreta y otra utilizando una analogía para un mejor entendimiento, luego aplicar el patrón para un Logger en una aplicación, el código lo mas simple posible para entender el concepto y otro código aparte mas avanzado, por último un resumen al respecto teniendo en cuenta los pro y contras de este patrón para el caso de uso en cuestión.
