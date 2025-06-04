El patrón **Prototype** (Prototipo) es un patrón de diseño creacional que te permite **copiar objetos existentes sin que tu código tenga que depender de sus clases concretas**. En otras palabras, en lugar de crear un objeto desde cero, clonás uno ya existente. Es como tener una "plantilla" de un objeto y hacer una copia de ella para crear uno nuevo.

---

### Analogía del Prototype: La Impresora 3D de Juguetes

Imaginá que tenés una **impresora 3D** en casa y te encanta hacer juguetes. En lugar de diseñar cada juguete desde cero cada vez, tenés un **modelo base** ya cargado en la impresora (un prototipo de juguete, digamos, un cochecito estándar).

* **Definición:** El modelo base del cochecito en la impresora es el Prototipo.
* **Copiar sin dependencia de clase concreta:** No tenés que saber *cómo* se fabricó el modelo del cochecito (sus planos internos, los materiales exactos que usó el diseñador original). Simplemente le decís a la impresora: "Haz una copia de este cochecito".
* **Creación de Variaciones:** Una vez que la impresora hace una copia, podés tomar esa copia y:
    * Cambiarle el color.
    * Añadirle alerones.
    * Ponerle ruedas más grandes.
    * Incluso podrías crear un modelo de camión a partir de él, haciendo modificaciones significativas.
* **Eficiencia:** Es mucho más rápido copiar un modelo existente y modificarlo que diseñar un cochecito nuevo desde cero cada vez que querés una pequeña variación.

El patrón Prototype aplica esta misma lógica al software: te permite crear nuevos objetos de manera eficiente, tomando una copia de un objeto existente y luego modificando solo las propiedades que necesitas cambiar, sin preocuparte por cómo se construyó el objeto original.

---

### ¿Qué problema resuelve el Prototype en una aplicación de e-commerce?

El patrón Prototype es útil en un e-commerce para resolver el problema de **crear objetos nuevos que son muy similares a objetos existentes, pero con algunas variaciones, de una manera eficiente y sin acoplamiento a las clases concretas**.

Consideremos estos escenarios en un e-commerce:

* **Creación de Productos Variantes:** Imaginate que vendés remeras. Tenés una remera base con su descripción, precio, SKU base, etc. Querés crear variantes de esa remera para diferentes tallas y colores. En lugar de instanciar una nueva `Remera` cada vez y copiar manualmente todas las propiedades comunes, podés **clonar la remera base** y solo modificar la talla y el color. Esto es más rápido y menos propenso a errores.
* **Carrito de Compras / Historial de Pedidos:** Cuando un cliente agrega un producto al carrito, no estás agregando el objeto *original* del catálogo (que podría cambiar de precio o stock en el futuro). Estás agregando una ***copia*** del producto tal como estaba en el momento de la adición. Lo mismo ocurre al guardar un pedido: los productos en el pedido son "fotos" (copias) de los productos en ese momento.
* **Creación de Órdenes o Documentos Complejos:** Si tenés objetos de `Order` o `Invoice` que a menudo comparten muchas propiedades iniciales (cliente, fecha, estado inicial), podés **clonar una "orden plantilla"** y luego rellenar solo los detalles específicos del nuevo pedido.

El Prototype resuelve el problema de la **"explosión de constructores"** (tener muchos constructores o métodos de fábrica para crear variaciones de un mismo objeto) y **reduce el acoplamiento** al permitirte trabajar con interfaces o clases abstractas al clonar, en lugar de con clases concretas.

---

### Código Simple para entender el concepto (Producto Básico de E-commerce)

Este ejemplo muestra cómo crear una "plantilla" de producto y luego clonarla para crear variaciones de tallas en un e-commerce.

```php
<?php

// 1. Interfaz Prototype: Declara el método de clonación.
// En PHP, esto se logra con el método mágico __clone().
interface ClonableProduct
{
    public function __clone(); // PHP tiene un método mágico __clone()
    public function getDetails(): string;
    public function setSize(string $size): void;
}

// 2. Clase Concreta (Producto): Implementa la interfaz Prototype.
class TShirt implements ClonableProduct
{
    private string $name;
    private string $color;
    private string $size;
    private float $price;
    // Propiedad para demostrar copia profunda/superficial
    private array $features = [];

    public function __construct(string $name, string $color, float $price)
    {
        $this->name = $name;
        $this->color = $color;
        $this->price = $price;
        $this->size = "M"; // Talla predeterminada
        $this->features[] = "100% Algodón";
    }

    // El método mágico __clone() es llamado automáticamente por PHP al usar 'clone $obj;'.
    // Aquí podemos ajustar si necesitamos copia profunda (deep copy) para propiedades complejas.
    public function __clone()
    {
        // Si 'features' fuera un objeto complejo y necesitaras una copia independiente,
        // harías: $this->features = clone $this->features;
        // Para arrays simples como este (de strings), la copia superficial por defecto es suficiente.
        // PHP por defecto hace una copia superficial de las propiedades al clonar.
        // Si una propiedad es un objeto, el clon tendrá una referencia al *mismo* objeto original,
        // a menos que lo clones explícitamente aquí.
    }

    public function getDetails(): string
    {
        return "Remera: {$this->name}, Color: {$this->color}, Talla: {$this->size}, Precio: \${$this->price}, Características: " . implode(", ", $this->features);
    }

    // Métodos para modificar las propiedades del objeto (especialmente del clon)
    public function setSize(string $size): void
    {
        $this->size = $size;
    }

    public function setColor(string $color): void
    {
        $this->color = $color;
    }

    public function setPrice(float $price): void
    {
        $this->price = $price;
    }

    public function addFeature(string $feature): void
    {
        $this->features[] = $feature;
    }
}

// --- Uso en tu aplicación (el "Cliente") ---

echo "--- Creando la Remera Base (Prototipo) ---\n";
$baseTShirt = new TShirt("Remera Básica", "Negro", 19.99);
$baseTShirt->addFeature("Estampado simple");
echo $baseTShirt->getDetails() . "\n\n";

echo "--- Clonando para crear variantes ---\n";

// Clonar para una talla L
$tshirtL = clone $baseTShirt; // Aquí se invoca __clone()
$tshirtL->setSize("L");
echo $tshirtL->getDetails() . "\n";

// Clonar para una talla S
$tshirtS = clone $baseTShirt; // Se invoca __clone()
$tshirtS->setSize("S");
echo $tshirtS->getDetails() . "\n";

// Clonar para una talla XL y otro color, y otro precio
$tshirtXLBlue = clone $baseTShirt; // Se invoca __clone()
$tshirtXLBlue->setSize("XL");
$tshirtXLBlue->setColor("Azul");
$tshirtXLBlue->setPrice(22.99); // Podemos cambiar el precio si esta variante es más cara
echo $tshirtXLBlue->getDetails() . "\n";

// Demostrar que el objeto original no cambió
echo "\n--- Remera Base original después de clonar ---\n";
echo $baseTShirt->getDetails() . "\n";

?>
```

---

### Código más avanzado (Gestión de Productos Complejos con Clonación Profunda)

En un escenario de e-commerce real, los productos suelen tener objetos anidados (ej. `Category`, `ImageCollection`, `ReviewCollection`). El patrón Prototype con **clonación profunda** es crucial aquí para asegurar que los objetos clonados tengan sus propias copias de estos objetos anidados, no referencias a los del original.

```php
<?php

// --- Parte 1: Objetos Anidados / Componentes del Producto ---

class Category
{
    private string $name;
    private string $slug;

    public function __construct(string $name, string $slug)
    {
        $this->name = $name;
        $this->slug = $slug;
    }

    public function getName(): string { return $this->name; }
    public function getSlug(): string { return $this->slug; }

    // Es importante que los objetos anidados también sean clonables si van a ser clonados profundamente.
    public function __clone()
    {
        // No hay objetos complejos dentro de Category, así que la copia superficial por defecto está bien.
        // Si Category tuviera, por ejemplo, un objeto 'ParentCategory', necesitaríamos 'clone $this->parentCategory;'.
    }
}

class ImageCollection
{
    private array $images = []; // Array de strings (URLs de imágenes)

    public function addImage(string $url): void
    {
        $this->images[] = $url;
    }

    public function getImages(): array
    {
        return $this->images;
    }

    public function __clone()
    {
        // Para asegurar que el array de imágenes sea una nueva instancia
        // y no una referencia al array original, debemos recrearlo o copiarlo explícitamente.
        // En PHP, al reasignar un array, se crea una copia superficial por defecto.
        // Si 'images' fuera un array de OBJETOS Image, necesitaríamos iterar y clonar cada objeto:
        // foreach ($this->images as $key => $image) { $this->images[$key] = clone $image; }
        $this->images = $this->images; // Esto ya crea una copia superficial del array
    }
}

// --- Parte 2: Interfaz Prototype y Clase Producto Compleja ---

interface ProductPrototype
{
    public function __clone();
    public function getFullDetails(): string;
    public function setSku(string $sku): void;
    public function setPrice(float $price): void;
    public function addAttribute(string $key, string $value): void;
    public function setCategory(Category $category): void;
    public function addImage(string $url): void; // Para añadir imágenes al clon
}

class ComplexProduct implements ProductPrototype
{
    private string $id;
    private string $name;
    private string $sku;
    private float $basePrice;
    private Category $category; // Objeto anidado
    private ImageCollection $images; // Objeto anidado que es una colección
    private array $attributes = []; // Ej: ['material' => 'algodón', 'brand' => 'XYZ']

    public function __construct(string $id, string $name, string $sku, float $basePrice, Category $category)
    {
        $this->id = $id;
        $this->name = $name;
        $this->sku = $sku;
        $this->basePrice = $basePrice;
        $this->category = $category;
        $this->images = new ImageCollection();
    }

    // Implementación crucial del clonado profundo para los objetos anidados.
    public function __clone()
    {
        // Clonar objetos complejos para asegurar que no compartan referencias con el original.
        // Si no clonamos aquí, $this->category y $this->images del clon seguirían referenciando
        // los mismos objetos que el prototipo original.
        $this->category = clone $this->category;
        $this->images = clone $this->images;
        // Para arrays de tipos primitivos (como 'attributes'), una simple reasignación o la copia superficial por defecto
        // de PHP al clonar el objeto padre es suficiente.
        $this->attributes = $this->attributes; // Esto crea una copia superficial del array de atributos.
    }

    public function getFullDetails(): string
    {
        $details = "ID: {$this->id}, Nombre: {$this->name}, SKU: {$this->sku}, Precio: \${$this->basePrice}\n";
        $details .= "  Categoría: {$this->category->getName()} ({$this->category->getSlug()})\n";
        $details .= "  Imágenes: " . implode(", ", $this->images->getImages()) . "\n";
        $details .= "  Atributos: " . json_encode($this->attributes);
        return $details;
    }

    // Setters para modificar las propiedades del objeto (especialmente del clon)
    public function setSku(string $sku): void { $this->sku = $sku; }
    public function setPrice(float $price): void { $this->basePrice = $price; }
    public function addAttribute(string $key, string $value): void { $this->attributes[$key] = $value; }
    public function setCategory(Category $category): void { $this->category = $category; }
    public function addImage(string $url): void { $this->images->addImage($url); }

    // Getter para acceder a la colección de imágenes (útil para verificar clonación profunda)
    public function getImages(): ImageCollection { return $this->images; }
}

// --- Parte 3: Uso en la Aplicación (Cliente) ---

// Crear un producto base complejo que servirá como prototipo.
$electronicsCategory = new Category("Electrónica", "electronica");
$baseLaptop = new ComplexProduct("PROD001", "Laptop Ultrabook", "LAP-ULT-BASE", 1200.00, $electronicsCategory);
$baseLaptop->addImage("laptop_base_1.jpg");
$baseLaptop->addImage("laptop_base_2.jpg");
$baseLaptop->addAttribute("procesador", "Intel i5");
$baseLaptop->addAttribute("ram", "8GB");

echo "--- Producto Base (Prototipo Complejo) ---\n";
echo $baseLaptop->getFullDetails() . "\n\n";

// Clonar para crear una variante premium de la laptop.
echo "--- Clonando para Laptop Premium (con cambios) ---\n";
$premiumLaptop = clone $baseLaptop; // Se invoca __clone() para ComplexProduct y sus objetos anidados.
$premiumLaptop->setSku("LAP-ULT-PREM");
$premiumLaptop->setPrice(1500.00);
$premiumLaptop->addAttribute("procesador", "Intel i7"); // Modifica solo el clon
$premiumLaptop->addAttribute("ram", "16GB"); // Modifica solo el clon
$premiumLaptop->addImage("laptop_premium_1.jpg"); // Añadir una nueva imagen solo al clon
echo $premiumLaptop->getFullDetails() . "\n";

// Clonar para una variante económica.
echo "\n--- Clonando para Laptop Económica (con cambios) ---\n";
$economicLaptop = clone $baseLaptop; // Se invoca __clone() de nuevo.
$economicLaptop->setSku("LAP-ULT-ECO");
$economicLaptop->setPrice(900.00);
$economicLaptop->addAttribute("procesador", "Intel i3"); // Modifica solo el clon
$economicLaptop->addAttribute("ram", "4GB"); // Modifica solo el clon
echo $economicLaptop->getFullDetails() . "\n";

// Demostrar que el objeto original no cambió y sus objetos anidados tampoco,
// gracias a la clonación profunda en __clone().
echo "\n--- Producto Base original después de clonar ---\n";
echo $baseLaptop->getFullDetails() . "\n";

// Verificar si las colecciones de imágenes son diferentes instancias (prueba de clonación profunda)
echo "\nVerificando si las instancias de colecciones de imágenes son diferentes:\n";
echo "Base images object hash: " . spl_object_hash($baseLaptop->getImages()) . "\n";
echo "Premium images object hash: " . spl_object_hash($premiumLaptop->getImages()) . "\n";
echo "Economic images object hash: " . spl_object_hash($economicLaptop->getImages()) . "\n";
// Deberían ser hashes diferentes, indicando que son objetos distintos gracias a __clone()
// de ComplexProduct llamando a __clone() de ImageCollection.

?>
```

---

### Resumen al Respecto (Pros y Contras del Prototype para E-commerce)

El patrón **Prototype** en una aplicación de e-commerce te permite **crear nuevos objetos (como productos o pedidos) copiando objetos existentes en lugar de construirlos desde cero**. Esto es particularmente útil cuando los objetos son complejos, tienen muchas propiedades en común, o la creación directa es costosa.

En PHP, el método mágico `__clone()` es fundamental. Al implementarlo, podés controlar cómo se realiza la copia. Para objetos anidados, es crucial realizar una **clonación profunda** para asegurar que el objeto copiado tenga sus propias instancias de estos componentes, y no solo referencias a los del original. Esto garantiza que las modificaciones en el objeto clonado no afecten al prototipo original ni a otros clones, manteniendo la integridad de tus datos.

#### **Pros (Ventajas):**

* **Eficiencia en la Creación de Objetos:** Es mucho más rápido copiar un objeto existente que instanciar uno nuevo y configurar todas sus propiedades desde cero, especialmente si el constructor es complejo o requiere la carga de datos.
* **Creación de Variantes Simplificada:** Ideal para escenarios donde tenés un objeto base y necesitas crear muchas versiones ligeramente modificadas (ej., productos con diferentes tallas, colores o configuraciones).
* **Reduce la Dependencia de Clases Concretas:** El código cliente no necesita conocer la clase específica de los objetos que está creando; solo necesita saber cómo clonar un prototipo. Esto reduce el acoplamiento y promueve el uso de interfaces.
* **Evita la "Explosión de Constructores":** En lugar de tener múltiples constructores o métodos de fábrica para diferentes combinaciones de propiedades, el Prototype permite una única forma de crear objetos y luego modificarlos.
* **Gestión del Estado de los Objetos:** Permite capturar el estado actual de un objeto como un prototipo y crear copias de ese estado en un momento dado (útil para carritos de compras o historial de pedidos).

#### **Contras (Desventajas):**

* **Complejidad en la Clonación Profunda:** Si los objetos tienen muchas referencias a otros objetos complejos, la implementación de `__clone()` para asegurar una clonación profunda correcta puede volverse intrincada y propensa a errores. Requiere que todos los objetos anidados también sean clonables.
* **Problemas con Referencias Circulares:** Si hay referencias circulares entre objetos, la clonación profunda puede entrar en bucles infinitos a menos que se maneje cuidadosamente.
* **No siempre Aplicable:** No todos los objetos son adecuados para ser prototipos. Algunos objetos pueden tener estados internos (ej., conexiones a bases de datos, gestores de archivos) que no deben ser simplemente copiados.
* **Posible Confusión entre Original y Copia:** Si los desarrolladores no entienden bien el patrón, podrían accidentalmente modificar el objeto prototipo en lugar de su clon, llevando a errores inesperados.

En el contexto de un e-commerce en PHP, el patrón Prototype es una herramienta poderosa para manejar la creación de **variantes de productos** y la **gestión de objetos en estados específicos** (como los elementos del carrito de compras). Sin embargo, su implementación requiere un buen entendimiento de cómo funciona la clonación en PHP, especialmente cuando se trata de la clonación profunda de objetos complejos y anidados.

### Prompt

Necesito que expliques el patrón Prototype (teniendo en cuenta que se va aplicar en PHP), primero con la definición concreta y otra utilizando una analogía para un mejor entendimiento, luego aplicar el patrón para el uso del manejo de productos en una aplicación de e-commerce, el código lo mas simple posible para entender el concepto y otro código aparte mas avanzado, por último un resumen al respecto teniendo en cuenta los pro y contras de este patrón para el caso de uso en cuestión.
