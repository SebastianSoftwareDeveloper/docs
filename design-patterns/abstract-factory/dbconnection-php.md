El patrón **Abstract Factory** (Fábrica Abstracta) es un patrón de diseño creacional que te permite crear familias de objetos relacionados o dependientes sin especificar sus clases concretas. En el contexto de la conexión a bases de datos, esto significa que podés tener un conjunto de clases que representan conexiones y comandos para una base de datos (por ejemplo, MySQL), y otro conjunto para otra base de datos (por ejemplo, PostgreSQL), y tu código cliente puede trabajar con cualquiera de ellas sin saber los detalles específicos de cada implementación.

---

### ¿Qué problema resuelve el Abstract Factory para la conexión a bases de datos?

El Abstract Factory resuelve el problema de **cómo crear familias enteras de objetos relacionados sin acoplar el código que los usa a las clases concretas de esos objetos**. Para las conexiones a bases de datos, esto significa:

* **Intercambiabilidad de bases de datos:** Podés cambiar fácilmente de un motor de base de datos a otro (ej., de MySQL a PostgreSQL) sin tener que modificar el código que realiza las operaciones (crear conexiones, ejecutar comandos). Solo necesitás cambiar la fábrica concreta que utilizás.
* **Coherencia de la familia de productos:** Garantiza que los objetos creados (conexión y comando) siempre sean compatibles entre sí, ya que son producidos por la misma fábrica. Por ejemplo, una fábrica de MySQL siempre te dará una `MySQLConnection` y un `MySqlCommand`.
* **Aislamiento del cliente:** El código de tu aplicación (el "cliente") trabaja con interfaces genéricas (`DBConnection`, `DBCommand`), no con las clases concretas (`MySQLConnection`, `PostgreSQLCommand`). Esto reduce el acoplamiento y hace que tu sistema sea más robusto y fácil de mantener.

---

### Código Simple para entender el concepto

Este ejemplo muestra cómo podés usar Abstract Factory para obtener conexiones y comandos de bases de datos para MySQL o PostgreSQL, sin que el cliente necesite saber los detalles internos.

```php
<?php

// 1. Interfaces de Productos Abstractos:
// Define qué tipo de conexión y comando pueden existir.
interface DBConnection {
    public function connect(): string;
}

interface DBCommand {
    public function execute(string $query): string;
}

// 2. Productos Concretos: Implementaciones para MySQL
class MySQLConnection implements DBConnection {
    public function connect(): string {
        return "Conexión a MySQL establecida.";
    }
}

class MySqlCommand implements DBCommand {
    public function execute(string $query): string {
        return "Comando MySQL ejecutado: '{$query}'";
    }
}

// 3. Productos Concretos: Implementaciones para PostgreSQL
class PostgreSQLConnection implements DBConnection {
    public function connect(): string {
        return "Conexión a PostgreSQL establecida.";
    }
}

class PostgreSQLCommand implements DBCommand {
    public function execute(string $query): string {
        return "Comando PostgreSQL ejecutado: '{$query}'";
    }
}

// 4. Interfaz de Fábrica Abstracta:
// Declara los métodos para crear cada tipo de producto abstracto.
interface DBFactory {
    public function createConnection(): DBConnection;
    public function createCommand(): DBCommand;
}

// 5. Fábricas Concretas: Implementan la interfaz de fábrica.
class MySQLFactory implements DBFactory {
    public function createConnection(): DBConnection {
        return new MySQLConnection();
    }

    public function createCommand(): DBCommand {
        return new MySqlCommand();
    }
}

class PostgreSQLFactory implements DBFactory {
    public function createConnection(): DBConnection {
        return new PostgreSQLConnection();
    }

    public function createCommand(): DBCommand {
        return new PostgreSQLCommand();
    }
}

// --- Uso en tu aplicación (el "Cliente") ---

// La función cliente trabaja solo con la interfaz de la fábrica abstracta
function clientCode(DBFactory $factory) {
    $connection = $factory->createConnection();
    $command = $factory->createCommand();

    echo $connection->connect() . "\n";
    echo $command->execute("SELECT * FROM users") . "\n";
}

echo "--- Usando la fábrica de MySQL ---\n";
$mysqlFactory = new MySQLFactory();
clientCode($mysqlFactory);
echo "\n";

echo "--- Usando la fábrica de PostgreSQL ---\n";
$pgFactory = new PostgreSQLFactory();
clientCode($pgFactory);

?>
```

---

### Código más avanzado (con un selector de fábrica basado en configuración)

Este ejemplo muestra cómo podrías seleccionar dinámicamente la fábrica correcta basándote en una configuración (como un entorno de desarrollo o producción), sin modificar el código principal.

```php
<?php

// --- Parte 1: Interfaces de Productos y Productos Concretos ---

// Mismas interfaces y productos que en el ejemplo simple
interface DBConnection {
    public function connect(): string;
    public function getStatus(): string; // Nuevo método
}

interface DBCommand {
    public function execute(string $query): string;
    public function getAffectedRows(): int; // Nuevo método
}

// MySQL Implementations
class MySQLConnection implements DBConnection {
    public function connect(): string {
        return "Conexión a MySQL establecida exitosamente.";
    }
    public function getStatus(): string {
        return "Estado: Conectado a MySQL.";
    }
}

class MySqlCommand implements DBCommand {
    public function execute(string $query): string {
        return "Comando MySQL ejecutado: '{$query}'";
    }
    public function getAffectedRows(): int {
        return rand(1, 100); // Simulando filas afectadas
    }
}

// PostgreSQL Implementations
class PostgreSQLConnection implements DBConnection {
    public function connect(): string {
        return "Conexión a PostgreSQL establecida exitosamente.";
    }
    public function getStatus(): string {
        return "Estado: Conectado a PostgreSQL.";
    }
}

class PostgreSQLCommand implements DBCommand {
    public function execute(string $query): string {
        return "Comando PostgreSQL ejecutado: '{$query}'";
    }
    public function getAffectedRows(): int {
        return rand(101, 200); // Simulando filas afectadas
    }
}

// --- Parte 2: Interfaz de Fábrica Abstracta y Fábricas Concretas ---

interface DBFactory {
    public function createConnection(): DBConnection;
    public function createCommand(): DBCommand;
}

class MySQLFactory implements DBFactory {
    public function createConnection(): DBConnection {
        return new MySQLConnection();
    }
    public function createCommand(): DBCommand {
        return new MySqlCommand();
    }
}

class PostgreSQLFactory implements DBFactory {
    public function createConnection(): DBConnection {
        return new PostgreSQLConnection();
    }
    public function createCommand(): DBCommand {
        return new PostgreSQLCommand();
    }
}

// --- Parte 3: Configuración y Uso en la Aplicación ---

// Esta función simula un "Director" o "Configurador"
// que decide qué fábrica concreta usar en tiempo de ejecución.
function getDatabaseFactory(string $dbType): DBFactory {
    switch (strtolower($dbType)) {
        case 'mysql':
            return new MySQLFactory();
        case 'postgresql':
            return new PostgreSQLFactory();
        default:
            throw new \InvalidArgumentException("Tipo de base de datos no soportado: {$dbType}");
    }
}

// --- Código de la Aplicación (Cliente) ---
// El cliente solo depende de la interfaz de la fábrica y los productos.

function runDatabaseOperations(DBFactory $factory) {
    echo "--- Realizando operaciones de DB con " . get_class($factory) . " ---\n";
    try {
        $connection = $factory->createConnection();
        echo $connection->connect() . "\n";
        echo $connection->getStatus() . "\n";

        $command = $factory->createCommand();
        $query = "SELECT data FROM records WHERE id = 1";
        echo $command->execute($query) . "\n";
        echo "Filas afectadas (simulado): " . $command->getAffectedRows() . "\n";
    } catch (\Exception $e) {
        echo "Error: " . $e->getMessage() . "\n";
    }
    echo "----------------------------------------\n\n";
}

// Uso basado en una "configuración" (podría ser un archivo .env, una constante, etc.)
$databaseConfig = 'mysql'; // Podría ser 'postgresql' para cambiar el comportamiento
echo "Configuración actual: Usando base de datos -> " . strtoupper($databaseConfig) . "\n\n";

try {
    $currentFactory = getDatabaseFactory($databaseConfig);
    runDatabaseOperations($currentFactory);
} catch (\InvalidArgumentException $e) {
    echo $e->getMessage() . "\n";
}

// Cambiamos la configuración para demostrar la flexibilidad
$databaseConfig = 'postgresql';
echo "Cambiando configuración: Usando base de datos -> " . strtoupper($databaseConfig) . "\n\n";

try {
    $currentFactory = getDatabaseFactory($databaseConfig);
    runDatabaseOperations($currentFactory);
} catch (\InvalidArgumentException $e) {
    echo $e->getMessage() . "\n";
}

// Intentando un tipo de DB no soportado
$databaseConfig = 'oracle';
echo "Intentando un tipo de base de datos no soportado: " . strtoupper($databaseConfig) . "\n\n";
try {
    $currentFactory = getDatabaseFactory($databaseConfig);
    runDatabaseOperations($currentFactory);
} catch (\InvalidArgumentException $e) {
    echo "¡Capturado el error! " . $e->getMessage() . "\n";
}

?>
```

---

### Resumen al Respecto

El patrón **Abstract Factory** para la conexión a bases de datos te permite **crear familias completas de objetos relacionados (como una conexión y un comando para MySQL, o una conexión y un comando para PostgreSQL) sin que tu código cliente tenga que saber qué clases concretas está utilizando**. La clave es que tenés una **interfaz de fábrica abstracta (`DBFactory`)** que define los métodos para crear cada tipo de producto (conexión, comando). Luego, tenés **fábricas concretas** (`MySQLFactory`, `PostgreSQLFactory`) que implementan esos métodos para producir sus productos específicos y compatibles.

Esto significa que podés **cambiar el motor de la base de datos de tu aplicación con solo modificar la instancia de la fábrica que pasas a tu código**, sin tocar la lógica de negocio que interactúa con la base de datos. El resultado es un sistema **altamente desacoplado, flexible y extensible**, ideal para aplicaciones que necesitan soportar múltiples implementaciones de una misma funcionalidad o que pueden evolucionar para usar diferentes tecnologías subyacentes.

### Prompt

Necesito que me explique el patrón abstract factory para la conexión a base de datos aplicado en php, primero explicando que resuelve, luego el código lo mas simple posible para entender el concepto y otro código aparte mas avanzado y por último un resumen al respecto.
