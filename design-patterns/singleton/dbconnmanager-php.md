El patrón **Singleton** (Instancia Única) es un patrón de diseño creacional que te permite **garantizar que una clase tenga una sola instancia a lo largo de toda tu aplicación, a la vez que proporciona un punto de acceso global a esa instancia**. Es como tener un recurso que debe ser compartido por todos y solo existe una vez, como un único encargado de una tarea específica.

---

### ¿Qué problema resuelve el Singleton para la conexión a la base de datos?

El patrón Singleton en el manejo de conexiones a bases de datos resuelve el problema de **garantizar que solo exista una única conexión activa a la base de datos en toda tu aplicación**. Esto es crucial por varias razones:

* **Optimización de recursos:** Abrir y cerrar múltiples conexiones a la base de datos es un proceso costoso en términos de tiempo y recursos del sistema. Con Singleton, solo se establece una conexión inicial, que luego se reutiliza.
* **Consistencia y control:** Al tener una única instancia de conexión, se evita la inconsistencia que podría surgir si diferentes partes del código abren sus propias conexiones, potencialmente con configuraciones distintas o llevando a problemas de concurrencia.
* **Punto de acceso global:** Permite que cualquier parte de tu aplicación acceda a la misma conexión de base de datos de manera sencilla y controlada, sin necesidad de pasar la instancia de conexión de un lado a otro.
* **Gestión de estado:** Si la conexión tiene un estado (como transacciones en curso), el Singleton asegura que este estado sea consistente y compartido por todos los usuarios de la conexión.

---

### Código Simple para entender el concepto

Este ejemplo muestra la implementación más básica del patrón Singleton para una conexión de base de datos simulada.

```php
<?php

class DatabaseConnection
{
    // 1. Almacena la única instancia de la clase.
    private static ?DatabaseConnection $instance = null;

    // 2. Hace el constructor privado para evitar la creación externa de nuevas instancias.
    private function __construct()
    {
        // Simulamos la lógica de conexión real a la base de datos
        echo "Estableciendo nueva conexión a la base de datos...\n";
        // Aquí iría tu código de conexión PDO, MySQLi, etc.
        // Por ejemplo: $this->pdo = new PDO(...);
    }

    // 3. Hace el método __clone() privado para evitar la clonación de la instancia.
    private function __clone()
    {
        // Lanzar una excepción es opcional, pero ayuda a prevenir errores.
        throw new Exception("No puedes clonar una instancia de Singleton.");
    }

    // 4. Hace el método __wakeup() privado para evitar la deserialización de la instancia.
    // Esto previene que se cree una nueva instancia al usar unserialize().
    private function __wakeup()
    {
        throw new Exception("No puedes deserializar una instancia de Singleton.");
    }

    // 5. El método público estático para obtener la única instancia.
    public static function getInstance(): DatabaseConnection
    {
        if (self::$instance === null) {
            self::$instance = new self();
        }
        return self::$instance;
    }

    // Método de ejemplo para usar la conexión
    public function query(string $sql): string
    {
        return "Ejecutando consulta: '{$sql}' usando la conexión única.";
    }
}

// --- Uso en tu aplicación (el "Cliente") ---

echo "--- Primer acceso a la conexión ---\n";
$db1 = DatabaseConnection::getInstance(); // La conexión se establece aquí
echo $db1->query("SELECT * FROM users;") . "\n";

echo "\n--- Segundo acceso a la conexión ---\n";
$db2 = DatabaseConnection::getInstance(); // Se devuelve la misma instancia existente
echo $db2->query("INSERT INTO products VALUES (...);") . "\n";

echo "\n--- Verificando si son la misma instancia ---\n";
if ($db1 === $db2) {
    echo "¡Las dos variables apuntan a la misma instancia de conexión!\n";
} else {
    echo "¡Error: Se crearon múltiples instancias de conexión!\n";
}

// Intentos fallidos de crear o clonar (descomentar para ver las excepciones)
/*
try {
    $db3 = new DatabaseConnection(); // Esto generará un error fatal (constructor privado)
} catch (Error $e) {
    echo "\nError al intentar instanciar directamente: " . $e->getMessage() . "\n";
}

try {
    $db4 = clone $db1; // Esto generará una excepción (método __clone() privado)
} catch (Exception $e) {
    echo "\nError al intentar clonar: " . $e->getMessage() . "\n";
}
*/

?>
```

---

### Código más avanzado (Singleton con PDO y Configuración)

Este ejemplo incorpora una conexión real a la base de datos usando **PDO** y muestra cómo el Singleton puede manejar la configuración y reconexión básica.

```php
<?php

class AdvancedDatabaseConnection
{
    private static ?AdvancedDatabaseConnection $instance = null;
    private ?PDO $pdo = null; // Instancia de PDO real

    private array $config; // Configuración de la base de datos

    // Constructor privado que recibe la configuración de la DB
    private function __construct(array $config)
    {
        $this->config = $config;
        $this->connect(); // Establece la conexión al construir la instancia
    }

    private function __clone()
    {
        throw new Exception("Cloning of a Singleton instance is not allowed.");
    }

    private function __wakeup()
    {
        throw new Exception("Deserialization of a Singleton instance is not allowed.");
    }

    // Método para establecer la conexión PDO
    private function connect(): void
    {
        $dsn = "mysql:host={$this->config['host']};dbname={$this->config['dbname']};charset={$this->config['charset']}";
        try {
            $this->pdo = new PDO(
                $dsn,
                $this->config['user'],
                $this->config['password'],
                $this->config['options'] ?? []
            );
            $this->pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
            echo "Conexión PDO establecida a {$this->config['dbname']} en {$this->config['host']}.\n";
        } catch (PDOException $e) {
            // En un entorno real, aquí se debería loggear el error y/o mostrar una página de error amigable.
            die("Error de conexión a la base de datos: " . $e->getMessage());
        }
    }

    // Método estático para obtener la instancia única.
    // Ahora, getInstance requiere la configuración la primera vez.
    public static function getInstance(array $config = []): AdvancedDatabaseConnection
    {
        if (self::$instance === null) {
            if (empty($config)) {
                throw new InvalidArgumentException("La configuración de la base de datos es necesaria para la primera instancia de Singleton.");
            }
            self::$instance = new self($config);
        }
        return self::$instance;
    }

    // Obtener el objeto PDO subyacente para realizar operaciones
    public function getPdo(): PDO
    {
        if ($this->pdo === null) {
            // Esto podría pasar si la conexión falló o si se quiere reconectar.
            // Para una reconexión, se necesitaría una lógica adicional.
            throw new RuntimeException("La conexión PDO no está establecida.");
        }
        return $this->pdo;
    }

    // Método de ejemplo para una operación simple (usando PDO)
    public function fetchAllUsers(): array
    {
        try {
            $stmt = $this->getPdo()->query("SELECT id, name, email FROM users LIMIT 3");
            return $stmt->fetchAll(PDO::FETCH_ASSOC);
        } catch (PDOException $e) {
            echo "Error en la consulta: " . $e->getMessage() . "\n";
            return [];
        }
    }
}

// --- Configuración de la base de datos (¡Reemplaza con tus datos reales!) ---
$dbConfig = [
    'host'     => 'localhost',
    'dbname'   => 'test_db', // Asegúrate de tener una DB llamada 'test_db' con una tabla 'users'
    'user'     => 'root',
    'password' => '',
    'charset'  => 'utf8mb4',
    'options'  => [
        PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
        PDO::ATTR_EMULATE_PREPARES   => false,
    ],
];

// --- Uso en tu aplicación (el "Cliente") ---

echo "--- Primera inicialización y consulta ---\n";
try {
    $dbInstance1 = AdvancedDatabaseConnection::getInstance($dbConfig);
    $users = $dbInstance1->fetchAllUsers();
    echo "Usuarios obtenidos:\n";
    print_r($users);
    echo "\n";
} catch (Exception $e) {
    echo "Error: " . $e->getMessage() . "\n";
}


echo "--- Segundo acceso y consulta (reutilizando la misma conexión) ---\n";
try {
    // No necesitamos pasar la configuración de nuevo, ya que la instancia ya existe
    $dbInstance2 = AdvancedDatabaseConnection::getInstance();
    $moreUsers = $dbInstance2->fetchAllUsers();
    echo "Más usuarios obtenidos (misma conexión):\n";
    print_r($moreUsers);
    echo "\n";
} catch (Exception $e) {
    echo "Error: " . $e->getMessage() . "\n";
}

echo "--- Verificando si son la misma instancia ---\n";
if (isset($dbInstance1) && isset($dbInstance2) && $dbInstance1 === $dbInstance2) {
    echo "¡Las dos variables apuntan a la misma instancia de conexión Singleton!\n";
} else {
    echo "¡Error: No se obtuvo la misma instancia Singleton!\n";
}

// Intentar crear otra instancia directamente (fallará)
/*
try {
    $anotherInstance = new AdvancedDatabaseConnection($dbConfig);
} catch (Error $e) {
    echo "\nError esperado al intentar instanciar directamente: " . $e->getMessage() . "\n";
}
*/
?>
```

---

### Resumen al Respecto

El patrón **Singleton** para la conexión a bases de datos en PHP asegura que **solo haya una única instancia de la clase de conexión disponible en toda tu aplicación**. Esto se logra haciendo el constructor y los métodos de clonación/deserialización privados, y proporcionando un **método estático público (`getInstance()`)** que es el único punto de entrada para obtener la instancia.

La principal ventaja es la **optimización de recursos** al evitar múltiples conexiones costosas y la **garantía de consistencia** al tener un único punto de control sobre el estado de la conexión a la base de datos. Si bien es útil para recursos compartidos y únicos como las conexiones a bases de datos, es importante usarlo con cautela, ya que el uso excesivo puede introducir dependencias globales y dificultar las pruebas unitarias.

### Prompt

Necesito que me explique el patrón Singleton para el manejo de conexión a la base de datos aplicado en php, primero explicando que resuelve, luego el código lo mas simple posible para entender el concepto y otro código aparte mas avanzado y por último un resumen al respecto.
