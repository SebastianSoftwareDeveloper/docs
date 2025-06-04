El patrón **Builder** (Constructor) es un patrón de diseño creacional que te permite **construir objetos complejos paso a paso, utilizando el mismo proceso de construcción para producir diferentes representaciones del objeto**. Se enfoca en separar la construcción de un objeto complejo de su representación, permitiendo que el mismo proceso de construcción cree objetos con diferentes propiedades o estructuras.

---

### Analogía del Builder: Armar una Hamburguesa Personalizada

Imaginá que vas a un restaurante de hamburguesas gourmet. En lugar de elegir una hamburguesa predefinida del menú (como una "clásica" o una "con queso"), el Builder te permite **personalizar tu propia hamburguesa**:

* **Definición:** El chef (o el sistema) es el Director, y los ingredientes y los pasos para armar la hamburguesa son el Builder. La hamburguesa final es el Producto.
* **Construcción Paso a Paso:** No pedís una hamburguesa entera de golpe. Decís: "Quiero una base de pan brioche" (esto es el `setBase()`). Luego, "agrega una carne de 200g" (`addPatty()`). Después, "ponle queso cheddar" (`addCheese()`), "lechuga" (`addVegetable()`), "cebolla caramelizada" (`addCondiment()`), y finalmente "un poco de salsa barbacoa" (`addSauce()`).
* **Diferentes Representaciones del Mismo Proceso:** Con el mismo "proceso de construcción de hamburguesas" (los mismos pasos `addPatty()`, `addCheese()`, etc.), podés obtener una hamburguesa vegetariana, una doble carne, una sin cebolla, etc. La lógica de cómo se ensambla la hamburguesa (el orden de los ingredientes, cómo se cocinan) está encapsulada por el chef/builder, no por vos, el cliente.
* **Simplificación para el Cliente:** Vos no tenés que saber cómo se hace el pan, cómo se cocina la carne o cómo se corta la lechuga. Simplemente decís qué querés agregar y en qué orden (o en el orden que te permite el constructor).

El patrón Builder aplica esta misma lógica al software, permitiéndote construir objetos complejos como queries SQL de manera incremental y flexible, sin tener que lidiar con toda la complejidad de la construcción en un solo lugar.

---

### ¿Qué problema resuelve el Builder para armar queries SQL?

El patrón Builder, aplicado a la construcción de consultas SQL, resuelve el problema de **cómo construir queries complejas que tienen muchas partes opcionales o que requieren una secuencia específica de construcción, sin que el código cliente se complique con la lógica de ensamblaje de la cadena SQL**.

Consideremos estos puntos clave:

* **Simplificación del Código Cliente:** Imaginate armar una query `SELECT` con `WHERE`, `JOIN`, `GROUP BY`, `ORDER BY`, `LIMIT`, etc. Sin el Builder, terminarías con funciones con demasiados parámetros, largas cadenas de `if/else`, o concatenaciones de strings difíciles de leer y mantener. El Builder te ofrece una interfaz limpia y fluida (encadenamiento de métodos como `select()->from()->where()`) para añadir cada parte de la query.
* **Construcción Paso a Paso:** Las cláusulas SQL tienen un orden. No podés tener un `WHERE` antes de un `FROM`. El Builder guía la construcción, permitiendo añadir partes de la query en el orden correcto y de forma incremental.
* **Creación de Diferentes Representaciones (Queries):** Con el mismo conjunto de métodos (`select()`, `where()`, `join()`), podés construir una simple `SELECT * FROM users` o una compleja `SELECT u.name, COUNT(o.id) FROM users u JOIN orders o ON u.id = o.user_id WHERE u.active = 1 GROUP BY u.name ORDER BY COUNT(o.id) DESC LIMIT 10`. La flexibilidad es enorme.
* **Separación de la Lógica de Construcción:** La compleja lógica de cómo se ensambla la query (cómo se manejan las comas entre campos, cómo se encadenan las condiciones `WHERE`, cómo se formatean los `JOIN`) se encapsula dentro del Builder. Tu código cliente solo se preocupa de *qué* query quiere, no de *cómo* se construye.
* **Validación de Construcción:** Un Builder más sofisticado podría incluso validar que la query se está construyendo de forma lógica (ej., no permitir un `GROUP BY` sin un `SELECT` de agregación).

---

### Código Simple para entender el concepto

Este ejemplo básico muestra cómo construir una consulta `SELECT` simple con cláusulas `WHERE` y `LIMIT` utilizando el patrón Builder.

```php
<?php

// 1. Producto: La query SQL que se está construyendo.
// Este objeto acumulará las partes de la consulta.
class SQLQuery
{
    private string $query = '';

    public function addPart(string $part): void
    {
        // Agrega una parte a la query, añadiendo un espacio para separar.
        $this->query .= $part . ' ';
    }

    public function getSQL(): string
    {
        // Retorna la query final, eliminando espacios extra al final.
        return trim($this->query);
    }
}

// 2. Builder (Interfaz): Define los pasos que debe seguir cualquier constructor de queries.
interface QueryBuilder
{
    public function select(string $fields): self; // Permite seleccionar campos. Retorna $this para encadenar.
    public function from(string $table): self;   // Define la tabla principal.
    public function where(string $condition): self; // Añade una condición WHERE.
    public function limit(int $limit): self;     // Añade una cláusula LIMIT.
    public function getResult(): SQLQuery;       // Devuelve el objeto SQLQuery construido.
    public function reset(): void;               // Reinicia el constructor para una nueva query.
}

// 3. Concrete Builder: Implementa la interfaz para construir queries específicas (ej. para MySQL).
class MySQLQueryBuilder implements QueryBuilder
{
    private SQLQuery $query; // La instancia del producto (SQLQuery) que estamos construyendo.

    public function __construct()
    {
        $this->reset(); // Inicializa una nueva query al crear el constructor.
    }

    // Reinicia el constructor, preparando una nueva instancia de SQLQuery.
    public function reset(): void
    {
        $this->query = new SQLQuery();
    }

    public function select(string $fields): self
    {
        $this->query->addPart("SELECT {$fields}");
        return $this; // Permite el encadenamiento de métodos: $builder->select()->from()
    }

    public function from(string $table): self
    {
        $this->query->addPart("FROM {$table}");
        return $this;
    }

    public function where(string $condition): self
    {
        $this->query->addPart("WHERE {$condition}");
        return $this;
    }

    public function limit(int $limit): self
    {
        $this->query->addPart("LIMIT {$limit}");
        return $this;
    }

    public function getResult(): SQLQuery
    {
        $result = $this->query; // Obtenemos el producto final.
        $this->reset();         // ¡Importante! Reiniciar el constructor para futuras queries.
        return $result;
    }
}

// 4. Director (Opcional pero útil): Orquesta los pasos de construcción para queries comunes.
// Un Director encapsula "recetas" de construcción.
class QueryDirector
{
    public function buildSimpleSelect(QueryBuilder $builder): SQLQuery
    {
        // Receta para una query simple: SELECT * FROM users
        return $builder->select("*")->from("users")->getResult();
    }

    public function buildConditionalSelect(QueryBuilder $builder, string $condition, int $limit): SQLQuery
    {
        // Receta para una query con WHERE y LIMIT
        return $builder->select("id, name")->from("products")->where($condition)->limit($limit)->getResult();
    }
}

// --- Uso en tu aplicación (el "Cliente") ---

$builder = new MySQLQueryBuilder(); // Creamos una instancia del constructor concreto.
$director = new QueryDirector();     // Creamos una instancia del Director (opcional).

echo "--- Construyendo query simple usando Director ---\n";
$simpleQuery = $director->buildSimpleSelect($builder);
echo $simpleQuery->getSQL() . "\n\n"; // Salida: SELECT * FROM users

echo "--- Construyendo query condicional usando Director ---\n";
$conditionalQuery = $director->buildConditionalSelect($builder, "price > 100", 5);
echo $conditionalQuery->getSQL() . "\n\n"; // Salida: SELECT id, name FROM products WHERE price > 100 LIMIT 5

// También puedes construir queries paso a paso directamente con el Builder (sin el Director)
echo "--- Construcción paso a paso directa (sin Director) ---\n";
$customQuery = $builder->select("category, COUNT(*) as total")
                       ->from("items")
                       ->where("active = 1")
                       ->getResult(); // No olvidemos getResult()
echo $customQuery->getSQL() . "\n"; // Salida: SELECT category, COUNT(*) as total FROM items WHERE active = 1

?>
```

---

### Código más avanzado (con soporte para `JOIN`, `GROUP BY`, y múltiples condiciones `WHERE`)

Este ejemplo expande la funcionalidad del Builder para soportar consultas más complejas, gestionando múltiples partes de la query de forma estructurada dentro del objeto `SQLQueryAdvanced`.

```php
<?php

// --- Parte 1: Producto (SQLQueryAdvanced mejorado) ---
// Este objeto ahora gestiona de manera más estructurada los componentes de la query.
class SQLQueryAdvanced
{
    private array $parts = [
        'select' => [],
        'from' => '',
        'joins' => [],
        'where' => [],
        'groupBy' => [],
        'orderBy' => [],
        'limit' => null,
        'offset' => null,
    ];

    public function setSelect(array $fields): void
    {
        $this->parts['select'] = $fields;
    }

    public function setFrom(string $table): void
    {
        $this->parts['from'] = $table;
    }

    public function addJoin(string $type, string $table, string $onCondition): void
    {
        $this->parts['joins'][] = strtoupper($type) . " JOIN " . $table . " ON " . $onCondition;
    }

    public function addWhere(string $condition): void
    {
        $this->parts['where'][] = $condition;
    }

    public function addGroupBy(string $field): void
    {
        $this->parts['groupBy'][] = $field;
    }

    public function addOrderBy(string $field, string $direction = 'ASC'): void
    {
        $this->parts['orderBy'][] = "{$field} " . strtoupper($direction);
    }

    public function setLimit(?int $limit): void // Puede ser null si no hay límite
    {
        $this->parts['limit'] = $limit;
    }

    public function setOffset(?int $offset): void // Puede ser null si no hay offset
    {
        $this->parts['offset'] = $offset;
    }

    // Método para ensamblar todas las partes en la query SQL final.
    public function getSQL(): string
    {
        $sql = "SELECT " . (empty($this->parts['select']) ? "*" : implode(", ", $this->parts['select']));
        $sql .= " FROM " . $this->parts['from'];

        foreach ($this->parts['joins'] as $join) {
            $sql .= " " . $join;
        }

        if (!empty($this->parts['where'])) {
            $sql .= " WHERE " . implode(" AND ", $this->parts['where']); // Múltiples WHEREs se unen con AND
        }

        if (!empty($this->parts['groupBy'])) {
            $sql .= " GROUP BY " . implode(", ", $this->parts['groupBy']);
        }

        if (!empty($this->parts['orderBy'])) {
            $sql .= " ORDER BY " . implode(", ", $this->parts['orderBy']);
        }

        if ($this->parts['limit'] !== null) {
            $sql .= " LIMIT " . $this->parts['limit'];
        }
        if ($this->parts['offset'] !== null) {
            $sql .= " OFFSET " . $this->parts['offset'];
        }

        return trim($sql) . ";"; // Añade punto y coma al final de la query
    }
}

// --- Parte 2: Builder (Interfaz y Concrete Builder avanzado) ---

interface QueryBuilderAdvanced
{
    public function select(array $fields = ['*']): self;
    public function from(string $table): self;
    // Tipo de JOIN (INNER, LEFT, RIGHT)
    public function join(string $table, string $onCondition, string $type = 'INNER'): self;
    public function where(string $condition): self; // La primera condición WHERE
    public function andWhere(string $condition): self; // Para añadir condiciones WHERE adicionales con AND
    public function groupBy(string $field): self;
    public function orderBy(string $field, string $direction = 'ASC'): self;
    public function limit(?int $limit): self;
    public function offset(?int $offset): self;
    public function getResult(): SQLQueryAdvanced;
    public function reset(): void;
}

class MySQLQueryBuilderAdvanced implements QueryBuilderAdvanced
{
    private SQLQueryAdvanced $query;

    public function __construct()
    {
        $this->reset();
    }

    public function reset(): void
    {
        $this->query = new SQLQueryAdvanced();
    }

    public function select(array $fields = ['*']): self
    {
        $this->query->setSelect($fields);
        return $this;
    }

    public function from(string $table): self
    {
        $this->query->setFrom($table);
        return $this;
    }

    public function join(string $table, string $onCondition, string $type = 'INNER'): self
    {
        $this->query->addJoin($type, $table, $onCondition);
        return $this;
    }

    public function where(string $condition): self
    {
        $this->query->addWhere($condition);
        return $this;
    }

    public function andWhere(string $condition): self
    {
        // Simplemente añade otra condición WHERE. La lógica de unión (AND) está en SQLQueryAdvanced.
        $this->query->addWhere($condition);
        return $this;
    }

    public function groupBy(string $field): self
    {
        $this->query->addGroupBy($field);
        return $this;
    }

    public function orderBy(string $field, string $direction = 'ASC'): self
    {
        $this->query->addOrderBy($field, $direction);
        return $this;
    }

    public function limit(?int $limit): self
    {
        $this->query->setLimit($limit);
        return $this;
    }

    public function offset(?int $offset): self
    {
        $this->query->setOffset($offset);
        return $this;
    }

    public function getResult(): SQLQueryAdvanced
    {
        $result = $this->query;
        $this->reset(); // Reiniciar el constructor para que esté listo para la siguiente query.
        return $result;
    }
}

// --- Parte 3: Director (Opcional, pero útil para encapsular "recetas" de construcción) ---

class QueryDirectorAdvanced
{
    public function buildUsersWithOrders(QueryBuilderAdvanced $builder): SQLQueryAdvanced
    {
        // Una receta para obtener usuarios con sus órdenes.
        return $builder->select(['u.id', 'u.name', 'o.order_id', 'o.amount'])
                      ->from('users u')
                      ->join('orders o', 'u.id = o.user_id', 'LEFT')
                      ->where('u.active = 1')
                      ->orderBy('u.name')
                      ->limit(10)
                      ->getResult();
    }

    public function buildProductSummary(QueryBuilderAdvanced $builder): SQLQueryAdvanced
    {
        // Una receta para obtener un resumen de productos por categoría.
        return $builder->select(['p.category', 'COUNT(p.id) as total_products', 'AVG(p.price) as avg_price'])
                      ->from('products p')
                      ->where('p.stock > 0')
                      ->groupBy('p.category')
                      ->orderBy('total_products', 'DESC')
                      ->getResult();
    }
}

// --- Uso en tu aplicación (el "Cliente") ---

$builder = new MySQLQueryBuilderAdvanced();
$director = new QueryDirectorAdvanced();

echo "--- Consulta de usuarios con órdenes (usando Director) ---\n";
$query1 = $director->buildUsersWithOrders($builder);
echo $query1->getSQL() . "\n\n";

echo "--- Consulta de resumen de productos (usando Director) ---\n";
$query2 = $director->buildProductSummary($builder);
echo $query2->getSQL() . "\n\n";

echo "--- Consulta construida paso a paso directamente (sin Director) ---\n";
// Ejemplo de cómo el cliente puede usar el Builder directamente para una query ad-hoc.
$query3 = $builder->select(['c.name', 'COUNT(p.id) as num_products'])
                  ->from('categories c')
                  ->join('products p', 'c.id = p.category_id', 'INNER')
                  ->where('p.active = 1')
                  ->andWhere('c.is_visible = 1') // Encadenando otra condición WHERE
                  ->groupBy('c.name')
                  ->orderBy('num_products', 'DESC')
                  ->limit(3)
                  ->offset(0)
                  ->getResult();
echo $query3->getSQL() . "\n";

?>
```

---

### Resumen al Respecto (Pros y Contras del Builder para Queries SQL)

El patrón **Builder** para la construcción de queries SQL te permite **crear consultas complejas y configurables paso a paso**, delegando la lógica de ensamblaje a un objeto `Builder`. En lugar de concatenar cadenas o usar métodos con muchos parámetros, el cliente interactúa con una interfaz fluida (encadenamiento de métodos como `select()->from()->where()`), que facilita la lectura y escritura de queries.

#### **Pros (Ventajas):**

* **Simplificación del Código Cliente:** El cliente no necesita conocer los detalles internos de cómo se arma una query. Solo invoca métodos descriptivos, lo que reduce la complejidad y mejora la legibilidad.
* **Construcción Flexible Paso a Paso:** Permite construir queries en múltiples pasos, añadiendo componentes según sea necesario. Esto es ideal para queries con muchas cláusulas opcionales.
* **Reutilización del Proceso de Construcción:** El mismo `QueryBuilder` puede ser utilizado para construir diferentes tipos de queries, o incluso para adaptar la construcción a diferentes dialectos SQL (ej., `MySQLQueryBuilder`, `PostgreSQLQueryBuilder`).
* **Separación de Responsabilidades:** La lógica de construcción (`Builder`) se separa de la representación final de la query (`SQLQuery`). Esto sigue el Principio de Responsabilidad Única.
* **Mayor Mantenibilidad:** Es mucho más fácil añadir nuevas cláusulas (ej., `HAVING`, `UNION`) o modificar la forma en que se construyen las existentes sin afectar el código cliente.
* **Prevención de Errores Comunes:** Al encapsular la lógica de ensamblaje, se pueden evitar errores como la falta de espacios, comas incorrectas o el orden inválido de las cláusulas.

#### **Contras (Desventajas):**

* **Aumento de la Complejidad Inicial:** Para queries muy simples, implementar el patrón Builder puede parecer una sobreingeniería, añadiendo más clases de las que la tarea "simple" inicialmente requeriría.
* **Clases Adicionales:** Requiere la creación de varias clases (Producto, Interfaz Builder, Concrete Builder, y opcionalmente Director), lo que aumenta el número total de archivos en el proyecto.
* **Curva de Aprendizaje:** Para desarrolladores no familiarizados con patrones de diseño, entender la estructura del Builder puede requerir una pequeña curva de aprendizaje inicial.
* **Posible Sobre-Abstracción:** Si las queries nunca se vuelven realmente complejas o no hay necesidad de múltiples representaciones, el patrón podría ser excesivo.

En resumen, el patrón Builder es una excelente opción para **gestionar la complejidad de la construcción de queries SQL en PHP** cuando estas queries son complejas, varían con frecuencia, o necesitas generar diferentes tipos de queries a partir de un proceso de construcción similar. Mejora significativamente la **legibilidad, mantenibilidad y flexibilidad** de tu código de acceso a datos, convirtiéndose en una herramienta poderosa para ORMs o librerías de construcción de consultas personalizadas.

### Prompt

Necesito que expliques el patrón Builder (teniendo en cuenta que se va aplicar en PHP), primero con la definición concreta y otra utilizando una analogía para un mejor entendimiento, luego aplicar el patrón para armar queries SQL, el código lo mas simple posible para entender el concepto y otro código aparte mas avanzado, por último un resumen al respecto teniendo en cuenta los pro y contras de este patrón para el caso de uso en cuestión.
