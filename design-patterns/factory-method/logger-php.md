El patrón **Factory Method** es una solución de diseño orientada a objetos que te permite crear objetos sin tener que especificar la clase exacta del objeto que se creará. Imaginate que tenés un sistema de registro (logger) y querés que pueda guardar mensajes en diferentes lugares: en un archivo, en una base de datos, o simplemente mostrarlos en la consola. Sin el Factory Method, tu código principal tendría que decidir explícitamente qué tipo de logger instanciar, lo que lo haría rígido y difícil de extender.

---

### ¿Qué problema resuelve el Factory Method para un Logger?

El Factory Method resuelve el problema de **cómo crear diferentes tipos de objetos (en este caso, distintos tipos de loggers) sin acoplar el código que los utiliza (el "cliente") a las clases concretas de esos objetos**. En un sistema de registro, esto significa que:

* **Flexibilidad:** Podés cambiar el tipo de logger que usás (de archivo a base de datos, por ejemplo) con solo modificar la fábrica que lo crea, sin tocar el código que envía los mensajes al logger.
* **Extensibilidad:** Añadir un nuevo tipo de logger (ej. uno que envíe logs a un servicio en la nube) es sencillo. Solo necesitás crear una nueva clase de logger y una nueva fábrica, sin alterar las clases existentes.
* **Menor acoplamiento:** El código que usa el logger (tu aplicación) trabaja con una **interfaz genérica (`Logger`)**, no con las clases concretas (`FileLogger`, `DatabaseLogger`). Esto hace que tu sistema sea más robusto y fácil de mantener.

---

### Código Simple para entender el concepto

Este ejemplo muestra cómo el Factory Method te permite obtener diferentes tipos de loggers sin que tu aplicación tenga que saber cómo se crean.

```php
<?php

// 1. Interfaz del Producto: Define lo que todos los loggers deben hacer.
interface Logger {
    public function log(string $message): void;
}

// 2. Productos Concretos: Implementaciones específicas de los loggers.
class FileLogger implements Logger {
    private string $filePath;

    public function __construct(string $filePath) {
        $this->filePath = $filePath;
    }

    public function log(string $message): void {
        file_put_contents($this->filePath, date('Y-m-d H:i:s') . " [ARCHIVO]: " . $message . PHP_EOL, FILE_APPEND);
    }
}

class ConsoleLogger implements Logger {
    public function log(string $message): void {
        echo date('Y-m-d H:i:s') . " [CONSOLA]: " . $message . PHP_EOL;
    }
}

// 3. Clase Creadora Abstracta (Fábrica): Declara el método fábrica.
abstract class LoggerFactory {
    // Este es el "Factory Method"
    abstract public function createLogger(): Logger;

    // La fábrica también puede contener lógica de negocio que utiliza el logger.
    public function logMessage(string $message): void {
        $logger = $this->createLogger(); // Aquí se llama al método fábrica
        $logger->log($message);
    }
}

// 4. Clases Creadoras Concretas: Implementan el método fábrica para crear un producto específico.
class FileLoggerFactory extends LoggerFactory {
    private string $filePath;

    public function __construct(string $filePath) {
        $this->filePath = $filePath;
    }

    public function createLogger(): Logger {
        return new FileLogger($this->filePath);
    }
}

class ConsoleLoggerFactory extends LoggerFactory {
    public function createLogger(): Logger {
        return new ConsoleLogger();
    }
}

// --- Uso en tu aplicación (el "Cliente") ---

echo "--- Logger de Consola ---\n";
$consoleFactory = new ConsoleLoggerFactory();
$consoleFactory->logMessage("¡Hola desde la consola!");
echo "\n";

echo "--- Logger de Archivo ---\n";
$fileFactory = new FileLoggerFactory(__DIR__ . '/app.log'); // El archivo se creará en el mismo directorio
$fileFactory->logMessage("Mensaje guardado en el archivo.");
$fileFactory->logMessage("Otro mensaje para el archivo.");

// Si querés ver el contenido del archivo (descomenta para probar)
// echo "\nContenido de app.log:\n";
// echo file_get_contents(__DIR__ . '/app.log');

?>
```

---

### Código más avanzado (con un Logger de Base de Datos y un selector de fábrica)

Este ejemplo expande el anterior, incluyendo un logger de base de datos simulado y una función que selecciona la fábrica adecuada según una configuración, mostrando cómo el patrón facilita la gestión en un entorno más complejo.

```php
<?php

// --- Parte 1: Interfaz y Productos Concretos (Loggers) ---

interface Logger {
    public function log(string $message): void;
}

class FileLogger implements Logger {
    private string $filePath;

    public function __construct(string $filePath) {
        $this->filePath = $filePath;
    }

    public function log(string $message): void {
        file_put_contents($this->filePath, date('Y-m-d H:i:s') . " [ARCHIVO]: " . $message . PHP_EOL, FILE_APPEND);
        // Opcional: echo "Log guardado en archivo: " . $message . "\n";
    }
}

class ConsoleLogger implements Logger {
    public function log(string $message): void {
        echo date('Y-m-d H:i:s') . " [CONSOLA]: " . $message . PHP_EOL;
    }
}

class DatabaseLogger implements Logger {
    // En un caso real, aquí iría la lógica de conexión y escritura en DB.
    // Por simplicidad, solo mostramos un mensaje.
    public function log(string $message): void {
        echo date('Y-m-d H:i:s') . " [BASE DE DATOS]: " . $message . " (Guardado en DB simulado)\n";
    }
}

// --- Parte 2: Clases Creadoras (Fábricas de Loggers) ---

abstract class LoggerFactory {
    abstract public function createLogger(): Logger;

    // Método que utiliza el logger, encapsulando la lógica de negocio
    public function performLogging(string $message): void {
        $logger = $this->createLogger(); // El Factory Method es llamado aquí
        $logger->log($message);
    }
}

class FileLoggerFactory extends LoggerFactory {
    private string $filePath;

    public function __construct(string $filePath) {
        $this->filePath = $filePath;
    }

    public function createLogger(): Logger {
        return new FileLogger($this->filePath);
    }
}

class ConsoleLoggerFactory extends LoggerFactory {
    public function createLogger(): Logger {
        return new ConsoleLogger();
    }
}

class DatabaseLoggerFactory extends LoggerFactory {
    public function createLogger(): Logger {
        // En un caso real, aquí se pasarían los detalles de conexión a la DB
        return new DatabaseLogger();
    }
}

// --- Parte 3: Configuración y Uso en la Aplicación ---

// Esta función simula un punto de entrada que decide qué fábrica usar
// Basado en una configuración (ej. un archivo .env, una constante)
function getLoggerFactory(string $type): LoggerFactory {
    switch ($type) {
        case 'file':
            // En un entorno real, la ruta del archivo vendría de la configuración
            return new FileLoggerFactory(__DIR__ . '/application.log');
        case 'database':
            return new DatabaseLoggerFactory();
        case 'console':
        default:
            return new ConsoleLoggerFactory();
    }
}

// --- Código de la Aplicación (Cliente) ---
// El cliente no necesita saber los detalles de creación del logger.
// Solo le pide a la función getLoggerFactory que le dé la fábrica adecuada.

$config_logger_type = 'file'; // Esto podría venir de una variable de entorno o un archivo de configuración
echo "Configuración: Usando logger de tipo: " . $config_logger_type . "\n";
$loggerFactory = getLoggerFactory($config_logger_type);
$loggerFactory->performLogging("Inicio de la aplicación: " . date('Y-m-d H:i:s'));
$loggerFactory->performLogging("Se ha procesado un evento importante.");
echo "\n";


$config_logger_type = 'console';
echo "Configuración: Cambiando a logger de tipo: " . $config_logger_type . "\n";
$loggerFactory = getLoggerFactory($config_logger_type);
$loggerFactory->performLogging("El usuario ha iniciado sesión.");
echo "\n";


$config_logger_type = 'database';
echo "Configuración: Cambiando a logger de tipo: " . $config_logger_type . "\n";
$loggerFactory = getLoggerFactory($config_logger_type);
$loggerFactory->performLogging("Error: Conexión fallida al microservicio X.");

// Para verificar el log de archivo, podrías necesitar abrir 'application.log' manualmente
// echo "\nVerificando contenido de 'application.log' si existe.\n";

?>
```

---

### Resumen al Respecto

El **Factory Method** en el contexto de un sistema de registro te permite **producir diferentes tipos de loggers sin que tu código principal (el cliente) tenga que conocer los detalles de su creación**. La lógica de instanciación se delega a **fábricas especializadas** (`FileLoggerFactory`, `ConsoleLoggerFactory`, `DatabaseLoggerFactory`), las cuales saben cómo crear su logger correspondiente. Tu aplicación solo interactúa con una **interfaz común (`Logger`)** y le pide a la fábrica el tipo de logger que necesita. Esto resulta en un diseño de software **más flexible, extensible y con bajo acoplamiento**, haciendo que sea mucho más fácil añadir o cambiar los mecanismos de registro en el futuro sin modificar el código que ya funciona.
