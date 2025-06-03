El patrón **Builder** (Constructor) es un patrón de diseño creacional que te permite construir objetos complejos paso a paso. Se diferencia de otros patrones creacionales en que no solo crea un objeto al final, sino que te permite construir diferentes representaciones del mismo objeto usando el mismo proceso de construcción. En el contexto de las consultas SQL, el Builder es ideal para construir queries complejas (SELECT, INSERT, UPDATE, DELETE) que pueden tener muchas partes opcionales, como cláusulas `WHERE`, `ORDER BY`, `JOIN`, etc.

---

### ¿Qué problema resuelve el Builder para armar queries SQL?

El Builder resuelve el problema de **cómo construir objetos complejos (queries SQL) que tienen muchas partes opcionales o que requieren una secuencia específica de construcción, sin que el código cliente se complique con la lógica de ensamblaje**. Para las consultas SQL, esto significa:

* **Simplificación del código cliente:** En lugar de tener largas cadenas de `if/else` o métodos con muchos parámetros para construir una query, el cliente interactúa con un objeto `Builder` que ofrece métodos intuitivos para añadir partes a la consulta (`select()`, `where()`, `orderBy()`).
* **Construcción paso a paso:** Permite construir la query gradualmente, añadiendo las partes necesarias en el orden correcto. Esto es muy útil porque las cláusulas SQL a menudo tienen dependencias de orden.
* **Creación de diferentes representaciones:** Con el mismo proceso de construcción (los mismos métodos `select()`, `where()`, etc.), podés generar diferentes tipos de queries (ej. `SELECT *` vs `SELECT id, name`) o incluso diferentes dialectos SQL si la lógica del Builder lo permite.
* **Separación de la lógica de construcción:** La lógica para armar la query se encapsula dentro del Builder, manteniendo el código cliente limpio y enfocado en qué query quiere, no en cómo se construye.

---

### Código Simple para entender el concepto

Este ejemplo básico muestra cómo construir una consulta `SELECT` simple con cláusulas `WHERE` y `LIMIT` usando el patrón Builder.

```php
<?php

// 1. Producto: La query SQL que se está construyendo.
class SQLQuery {
    private string $query = '';

    public function setPart(string $part): void {
        $this->query .= $part . ' ';
    }

    public function getSQL(): string {
        return trim($this->query);
    }
}

// 2. Builder: Interfaz que define los pasos de construcción.
interface QueryBuilder {
    public function select(string $fields): self;
    public function from(string $table): self;
    public function where(string $condition): self;
    public function limit(int $limit): self;
    public function getResult(): SQLQuery; // Devuelve el producto final
}

// 3. Concrete Builder: Implementa los pasos de construcción.
class MySQLQueryBuilder implements QueryBuilder {
    private SQLQuery $query;

    public function __construct() {
        $this->reset();
    }

    // Reinicia el constructor para construir una nueva query.
    public function reset(): void {
        $this->query = new SQLQuery();
    }

    public function select(string $fields): self {
        $this->query->setPart("SELECT {$fields}");
        return $this; // Permite el encadenamiento de métodos
    }

    public function from(string $table): self {
        $this->query->setPart("FROM {$table}");
        return $this;
    }

    public function where(string $condition): self {
        $this->query->setPart("WHERE {$condition}");
        return $this;
    }

    public function limit(int $limit): self {
        $this->query->setPart("LIMIT {$limit}");
        return $this;
    }

    public function getResult(): SQLQuery {
        $result = $this->query;
        $this->reset(); // Importante: reiniciar el constructor para futuras queries
        return $result;
    }
}

// 4. Director (Opcional pero útil): Orquesta los pasos de construcción.
// En este caso simple, no es estrictamente necesario, pero lo incluimos para ilustrar.
class QueryDirector {
    public function buildSimpleSelect(QueryBuilder $builder): SQLQuery {
        return $builder->select("*")->from("users")->getResult();
    }

    public function buildConditionalSelect(QueryBuilder $builder, string $condition, int $limit): SQLQuery {
        return $builder->select("id, name")->from("products")->where($condition)->limit($limit)->getResult();
    }
}

// --- Uso en tu aplicación (el "Cliente") ---

$builder = new MySQLQueryBuilder();
$director = new QueryDirector();

echo "--- Construyendo query simple ---\n";
$simpleQuery = $director->buildSimpleSelect($builder);
echo $simpleQuery->getSQL() . "\n\n";

echo "--- Construyendo query condicional ---\n";
$conditionalQuery = $director->buildConditionalSelect($builder, "price > 100", 5);
echo $conditionalQuery->getSQL() . "\n\n";

// También puedes construir paso a paso sin un Director
echo "--- Construcción paso a paso directa ---\n";
$customQuery = $builder->select("category, COUNT(*) as total")
                       ->from("items")
                       ->where("active = 1")
                       ->getResult();
echo $customQuery->getSQL() . "\n";

?>
```

---

### Código más avanzado (con soporte para `JOIN`, `GROUP BY`, y múltiples condiciones `WHERE`)

Este ejemplo expande la funcionalidad del Builder para soportar consultas más complejas, gestionando múltiples partes de la query de forma estructurada.

```php
<?php

// --- Parte 1: Producto (SQLQuery mejorado) ---
// Representa la query SQL final, gestionando sus diferentes componentes.
class SQLQueryAdvanced {
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

    public function setSelect(array $fields): void {
        $this->parts['select'] = $fields;
    }

    public function setFrom(string $table): void {
        $this->parts['from'] = $table;
    }

    public function addJoin(string $type, string $table, string $onCondition): void {
        $this->parts['joins'][] = strtoupper($type) . " JOIN " . $table . " ON " . $onCondition;
    }

    public function addWhere(string $condition): void {
        $this->parts['where'][] = $condition;
    }

    public function addGroupBy(string $field): void {
        $this->parts['groupBy'][] = $field;
    }

    public function addOrderBy(string $field, string $direction = 'ASC'): void {
        $this->parts['orderBy'][] = "{$field} " . strtoupper($direction);
    }

    public function setLimit(int $limit): void {
        $this->parts['limit'] = $limit;
    }

    public function setOffset(int $offset): void {
        $this->parts['offset'] = $offset;
    }

    public function getSQL(): string {
        $sql = "SELECT " . (empty($this->parts['select']) ? "*" : implode(", ", $this->parts['select']));
        $sql .= " FROM " . $this->parts['from'];

        foreach ($this->parts['joins'] as $join) {
            $sql .= " " . $join;
        }

        if (!empty($this->parts['where'])) {
            $sql .= " WHERE " . implode(" AND ", $this->parts['where']);
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

        return trim($sql) . ";";
    }
}

// --- Parte 2: Builder (Interfaz y Concrete Builder) ---

interface QueryBuilderAdvanced {
    public function select(array $fields = ['*']): self;
    public function from(string $table): self;
    public function join(string $table, string $onCondition, string $type = 'INNER'): self;
    public function where(string $condition): self;
    public function andWhere(string $condition): self; // Para añadir más condiciones WHERE
    public function groupBy(string $field): self;
    public function orderBy(string $field, string $direction = 'ASC'): self;
    public function limit(int $limit): self;
    public function offset(int $offset): self;
    public function getResult(): SQLQueryAdvanced;
    public function reset(): void;
}

class MySQLQueryBuilderAdvanced implements QueryBuilderAdvanced {
    private SQLQueryAdvanced $query;

    public function __construct() {
        $this->reset();
    }

    public function reset(): void {
        $this->query = new SQLQueryAdvanced();
    }

    public function select(array $fields = ['*']): self {
        $this->query->setSelect($fields);
        return $this;
    }

    public function from(string $table): self {
        $this->query->setFrom($table);
        return $this;
    }

    public function join(string $table, string $onCondition, string $type = 'INNER'): self {
        $this->query->addJoin($type, $table, $onCondition);
        return $this;
    }

    public function where(string $condition): self {
        // En este Builder simple, 'where' siempre añade,
        // pero un Builder más avanzado podría manejar la primera condición diferente.
        $this->query->addWhere($condition);
        return $this;
    }

    public function andWhere(string $condition): self {
        // Simplemente añade otra condición WHERE.
        $this->query->addWhere($condition);
        return $this;
    }

    public function groupBy(string $field): self {
        $this->query->addGroupBy($field);
        return $this;
    }

    public function orderBy(string $field, string $direction = 'ASC'): self {
        $this->query->addOrderBy($field, $direction);
        return $this;
    }

    public function limit(int $limit): self {
        $this->query->setLimit($limit);
        return $this;
    }

    public function offset(int $offset): self {
        $this->query->setOffset($offset);
        return $this;
    }

    public function getResult(): SQLQueryAdvanced {
        $result = $this->query;
        $this->reset(); // Reiniciar para construir una nueva query
        return $result;
    }
}

// --- Parte 3: Director (Opcional, pero útil para recetas de construcción) ---

class QueryDirectorAdvanced {
    public function buildUsersWithOrders(QueryBuilderAdvanced $builder): SQLQueryAdvanced {
        return $builder->select(['u.id', 'u.name', 'o.order_id', 'o.amount'])
                      ->from('users u')
                      ->join('orders o', 'u.id = o.user_id', 'LEFT')
                      ->where('u.active = 1')
                      ->orderBy('u.name')
                      ->limit(10)
                      ->getResult();
    }

    public function buildProductSummary(QueryBuilderAdvanced $builder): SQLQueryAdvanced {
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

echo "--- Consulta de usuarios con órdenes ---\n";
$query1 = $director->buildUsersWithOrders($builder);
echo $query1->getSQL() . "\n\n";

echo "--- Consulta de resumen de productos ---\n";
$query2 = $director->buildProductSummary($builder);
echo $query2->getSQL() . "\n\n";

echo "--- Consulta construida paso a paso (sin Director) ---\n";
$query3 = $builder->select(['c.name', 'COUNT(p.id) as num_products'])
                  ->from('categories c')
                  ->join('products p', 'c.id = p.category_id', 'INNER')
                  ->where('p.active = 1')
                  ->andWhere('c.is_visible = 1') // Añadiendo otra condición WHERE
                  ->groupBy('c.name')
                  ->orderBy('num_products', 'DESC')
                  ->limit(3)
                  ->offset(0)
                  ->getResult();
echo $query3->getSQL() . "\n";

?>
```

---

### Resumen al Respecto

El patrón **Builder** para la construcción de queries SQL te permite **crear consultas complejas y configurables paso a paso**, delegando la lógica de ensamblaje a un objeto `Builder`. En lugar de concatenar cadenas o usar métodos con muchos parámetros, el cliente interactúa con una interfaz fluida (encadenamiento de métodos como `select()->from()->where()`), que facilita la lectura y escritura de queries.

Esto resulta en un código **más limpio, más mantenible y más flexible**. Puedes crear diferentes tipos de queries con la misma lógica de construcción, añadir o quitar partes de la consulta fácilmente, y mantener el código cliente libre de los detalles de cómo se forman las sentencias SQL, promoviendo un bajo acoplamiento y una alta cohesión en tu aplicación. Es particularmente útil cuando las queries tienen muchas partes opcionales o cuando la estructura de la consulta puede variar significativamente.

### Prompt

Necesito que me explique el patrón builder para armar querys SQL aplicado en php, primero explicando que resuelve, luego el código lo mas simple posible para entender el concepto y otro código aparte mas avanzado y por último un resumen al respecto.
