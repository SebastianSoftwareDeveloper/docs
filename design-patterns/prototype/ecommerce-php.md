El patrón **Prototype** (Prototipo) es un patrón de diseño creacional que te permite **copiar objetos existentes sin que tu código tenga que depender de sus clases concretas**. En otras palabras, en lugar de crear un objeto desde cero, clonás uno ya existente. Es como tener una "plantilla" de un objeto y hacer una copia de ella para crear uno nuevo.

---

### ¿Qué problema resuelve el Prototype en una aplicación de e-commerce?

El patrón Prototype es útil en un e-commerce para resolver el problema de **crear objetos nuevos que son muy similares a objetos existentes, pero con algunas variaciones, de una manera eficiente y sin acoplamiento a las clases concretas**.

Consideremos estos escenarios en un e-commerce:

* **Creación de Productos Variantes:** Imagina que vendes remeras. Tenés una remera base con su descripción, precio, SKU base, etc. Querés crear variantes de esa remera para diferentes tallas y colores. En lugar de instanciar una nueva `Remera` cada vez y copiar manualmente todas las propiedades comunes, podés clonar la remera base y solo modificar la talla y el color. Esto es más rápido y menos propenso a errores.
* **Carrito de Compras / Historial de Pedidos:** Cuando un cliente agrega un producto al carrito, no estás agregando el objeto *original* del catálogo (que podría cambiar de precio o stock). Estás agregando una *copia* del producto tal como estaba en el momento de la adición. Lo mismo ocurre al guardar un pedido: los productos en el pedido son "fotos" (copias) de los productos en ese momento.
* **Creación de Órdenes o Documentos Complejos:** Si tenés objetos de `Order` o `Invoice` que a menudo comparten muchas propiedades iniciales (cliente, fecha, estado inicial), podés clonar una "orden plantilla" y luego rellenar solo los detalles específicos del nuevo pedido.

El Prototype resuelve el problema de la **"explosión de constructores"** (tener muchos constructores o métodos de fábrica para crear variaciones de un mismo objeto) y **reduce el acoplamiento** al permitirte trabajar con interfaces o clases abstractas al clonar, en lugar de con clases concretas.

---

### Código Simple para entender el concepto (Producto Básico de E-commerce)

Este ejemplo muestra cómo crear una "plantilla" de producto y luego clonarla para crear variaciones de tallas.

```php
<?php

// 1. Interfaz Prototype: Declara el método de clonación.
interface ClonableProduct {
    public function __clone(); // PHP tiene un método mágico __clone()
    public function getDetails(): string;
    public function setSize(string $size): void;
}

// 2. Clase Concreta (Producto): Implementa la interfaz Prototype.
class TShirt implements ClonableProduct {
    private string $name;
    private string $color;
    private string $size;
    private float $price;
    private array $features = []; // Propiedad para demostrar copia profunda/superficial

    public function __construct(string $name, string $color, float $price) {
        $this->name = $name;
        $this->color = $color;
        $this->price = $price;
        $this->size = "M"; // Talla predeterminada
        $this->features[] = "100% Algodón";
    }

    // El método mágico __clone() es llamado automáticamente por PHP al usar 'clone'.
    // Aquí podemos ajustar si necesitamos copia profunda (deep copy) para propiedades complejas.
    public function __clone() {
        // Si 'features' fuera un objeto complejo y necesitaras una copia independiente,
        // harías: $this->features = clone $this->features;
        // Para arrays simples, la copia superficial suele ser suficiente.
        $this->features = $this->features; // Copia superficial por defecto para array
    }

    public function getDetails(): string {
        return "Remera: {$this->name}, Color: {$this->color}, Talla: {$this->size}, Precio: \${$this->price}, Características: " . implode(", ", $this->features);
    }

    public function setSize(string $size): void {
        $this->size = $size;
    }

    public function setColor(string $color): void {
        $this->color = $color;
    }

    public function setPrice(float $price): void {
        $this->price = $price;
    }

    public function addFeature(string $feature): void {
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
$tshirtL = clone $baseTShirt;
$tshirtL->setSize("L");
echo $tshirtL->getDetails() . "\n";

// Clonar para una talla S
$tshirtS = clone $baseTShirt;
$tshirtS->setSize("S");
echo $tshirtS->getDetails() . "\n";

// Clonar para una talla XL y otro color
$tshirtXLBlue = clone $baseTShirt;
$tshirtXLBlue->setSize("XL");
$tshirtXLBlue->setColor("Azul");
$tshirtXLBlue->setPrice(22.99); // Podemos cambiar el precio si esta variante es más cara
echo $tshirtXLBlue->getDetails() . "\n";

// Demostrar que el original no cambió
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

class Category {
    private string $name;
    private string $slug;

    public function __construct(string $name, string $slug) {
        $this->name = $name;
        $this->slug = $slug;
    }

    public function getName(): string { return $this->name; }
    public function getSlug(): string { return $this->slug; }

    // Necesitamos que Category sea clonable si está dentro de un Product y Product es clonado.
    public function __clone() {
        // No hay objetos complejos dentro de Category, así que la copia superficial está bien.
    }
}

class ImageCollection {
    private array $images = []; // Array de strings (URLs de imágenes)

    public function addImage(string $url): void {
        $this->images[] = $url;
    }

    public function getImages(): array {
        return $this->images;
    }

    public function __clone() {
        // Para asegurar que el array de imágenes sea una nueva instancia
        // y no una referencia al array original, hacemos una copia profunda si es necesario.
        // En este caso, al ser un array de strings, la copia superficial suele ser suficiente
        // ya que los strings son tipos primitivos. Pero si fueran objetos Image, necesitaríamos iterar.
        $this->images = $this->images; // Copia el array (superficial)
    }
}

// --- Parte 2: Interfaz Prototype y Clase Producto Compleja ---

interface ProductPrototype {
    public function __clone();
    public function getFullDetails(): string;
    public function setSku(string $sku): void;
    public function setPrice(float $price): void;
    public function addAttribute(string $key, string $value): void;
    public function setCategory(Category $category): void;
}

class ComplexProduct implements ProductPrototype {
    private string $id;
    private string $name;
    private string $sku;
    private float $basePrice;
    private Category $category;
    private ImageCollection $images;
    private array $attributes = []; // Ej: ['material' => 'algodón', 'brand' => 'XYZ']

    public function __construct(string $id, string $name, string $sku, float $basePrice, Category $category) {
        $this->id = $id;
        $this->name = $name;
        $this->sku = $sku;
        $this->basePrice = $basePrice;
        $this->category = $category;
        $this->images = new ImageCollection();
    }

    // Implementación crucial del clonado profundo
    public function __clone() {
        // Clonar objetos complejos para asegurar que no compartan referencias
        $this->category = clone $this->category;
        $this->images = clone $this->images;
        // Para arrays de tipos primitivos (como 'attributes'), una simple reasignación ya hace una copia superficial
        $this->attributes = $this->attributes; // Copia superficial del array
    }

    public function getFullDetails(): string {
        $details = "ID: {$this->id}, Nombre: {$this->name}, SKU: {$this->sku}, Precio: \${$this->basePrice}\n";
        $details .= "  Categoría: {$this->category->getName()} ({$this->category->getSlug()})\n";
        $details .= "  Imágenes: " . implode(", ", $this->images->getImages()) . "\n";
        $details .= "  Atributos: " . json_encode($this->attributes);
        return $details;
    }

    // Setters para modificar el clon
    public function setSku(string $sku): void { $this->sku = $sku; }
    public function setPrice(float $price): void { $this->basePrice = $price; }
    public function addAttribute(string $key, string $value): void { $this->attributes[$key] = $value; }
    public function setCategory(Category $category): void { $this->category = $category; }
    public function addImage(string $url): void { $this->images->addImage($url); }
}

// --- Parte 3: Uso en la Aplicación ---

// Crear un producto base complejo
$electronicsCategory = new Category("Electrónica", "electronica");
$baseLaptop = new ComplexProduct("PROD001", "Laptop Ultrabook", "LAP-ULT-BASE", 1200.00, $electronicsCategory);
$baseLaptop->addImage("laptop_base_1.jpg");
$baseLaptop->addImage("laptop_base_2.jpg");
$baseLaptop->addAttribute("procesador", "Intel i5");
$baseLaptop->addAttribute("ram", "8GB");

echo "--- Producto Base (Prototipo Complejo) ---\n";
echo $baseLaptop->getFullDetails() . "\n\n";

// Clonar para crear una variante premium
echo "--- Clonando para Laptop Premium (con cambios) ---\n";
$premiumLaptop = clone $baseLaptop;
$premiumLaptop->setSku("LAP-ULT-PREM");
$premiumLaptop->setPrice(1500.00);
$premiumLaptop->addAttribute("procesador", "Intel i7");
$premiumLaptop->addAttribute("ram", "16GB");
$premiumLaptop->addImage("laptop_premium_1.jpg"); // Añadir una nueva imagen solo al clon
echo $premiumLaptop->getFullDetails() . "\n";

// Clonar para una variante económica
echo "\n--- Clonando para Laptop Económica (con cambios) ---\n";
$economicLaptop = clone $baseLaptop;
$economicLaptop->setSku("LAP-ULT-ECO");
$economicLaptop->setPrice(900.00);
$economicLaptop->addAttribute("procesador", "Intel i3");
$economicLaptop->addAttribute("ram", "4GB");
echo $economicLaptop->getFullDetails() . "\n";

// Demostrar que el original no cambió y sus objetos anidados tampoco
echo "\n--- Producto Base original después de clonar ---\n";
echo $baseLaptop->getFullDetails() . "\n";

// Verificar si las colecciones de imágenes son diferentes instancias
echo "\nVerificando si las instancias de imágenes son diferentes:\n";
echo "Base images hash: " . spl_object_hash($baseLaptop->getImages()) . "\n";
echo "Premium images hash: " . spl_object_hash($premiumLaptop->getImages()) . "\n";
echo "Economic images hash: " . spl_object_hash($economicLaptop->getImages()) . "\n";
// Deberían ser hashes diferentes, indicando que son objetos distintos gracias a __clone()

?>
```

---

### Resumen al Respecto

El patrón **Prototype** en una aplicación de e-commerce te permite **crear nuevos objetos (como productos o pedidos) copiando objetos existentes en lugar de construirlos desde cero**. Esto es particularmente útil cuando los objetos son complejos, tienen muchas propiedades en común, o la creación directa es costosa.

Al implementar el método mágico `__clone()` en PHP, podés controlar cómo se realiza la copia. Para objetos anidados, es fundamental realizar una **clonación profunda** para asegurar que el objeto copiado tenga sus propias instancias de sus componentes, y no solo referencias a los del original. Esto garantiza que las modificaciones en el objeto clonado no afecten al prototipo original ni a otros clones, manteniendo la integridad de tus datos y facilitando la creación de variantes de productos o la gestión de estados de objetos complejos de manera eficiente y desacoplada.

### Prompt

Necesito que me explique el patrón prototype para algún caso de uso real en una aplicación de e-commerce en php, primero explicando que resuelve, luego el código lo mas simple posible para entender el concepto y otro código aparte mas avanzado y por último un resumen al respecto.
