# Patrones de Diseño de Software

Este directorio contiene la documentación y ejemplos prácticos de los 22 patrones de diseño clásicos, categorizados según su propósito. Cada patrón busca resolver problemas comunes en el diseño de software, proporcionando soluciones reutilizables y eficientes.

## Patrones Creacionales

Estos patrones se centran en la creación de objetos, proporcionando mecanismos que incrementan la flexibilidad y la reutilización del código.

* **[Factory Method](https://github.com/SebastianSoftwareDeveloper/docs/tree/main/design-patterns/factory-method):** Define una interfaz para crear un objeto, pero deja que las subclases decidan qué clase instanciar. Permite que una clase delegue la creación de objetos a sus subclases.
* **[Abstract Factory](https://github.com/SebastianSoftwareDeveloper/docs/tree/main/design-patterns/abstract-factory):** Proporciona una interfaz para crear familias de objetos relacionados o dependientes sin especificar sus clases concretas.
* **Builder:** Permite construir objetos complejos paso a paso. El patrón permite producir diferentes tipos y representaciones de un objeto utilizando el mismo código de construcción.
* **Prototype:** Permite copiar objetos existentes sin hacer que tu código dependa de sus clases concretas.
* **Singleton:** Garantiza que una clase tenga solo una instancia, y proporciona un punto de acceso global a ella.

## Patrones Estructurales

Estos patrones explican cómo ensamblar objetos y clases en estructuras más grandes, manteniendo estas estructuras flexibles y eficientes.

* **Adapter:** Permite que objetos con interfaces incompatibles colaboren entre sí.
* **Bridge:** Desacopla una abstracción de su implementación, permitiendo que ambas evolucionen de forma independiente.
* **Composite:** Compone objetos en estructuras de árbol para representar jerarquías de parte-todo. Permite que los clientes traten a los objetos individuales y a los compuestos de manera uniforme.
* **Decorator:** Permite añadir nuevas funcionalidades a un objeto existente dinámicamente, sin alterar su estructura.
* **Facade:** Proporciona una interfaz unificada y simplificada a un conjunto complejo de interfaces en un subsistema.
* **Flyweight:** Minimiza el uso de memoria o los costes computacionales compartiendo la mayor cantidad posible de datos similares entre múltiples objetos.
* **Proxy:** Proporciona un sustituto o marcador de posición para otro objeto para controlar el acceso a este.

## Patrones de Comportamiento

Estos patrones se ocupan de los algoritmos y la asignación de responsabilidades entre objetos.

* **Chain of Responsibility:** Permite pasar solicitudes a lo largo de una cadena de manejadores. Al recibir una solicitud, cada manejador decide si procesarla o pasarla al siguiente en la cadena.
* **Command:** Convierte una solicitud en un objeto independiente que contiene toda la información sobre la solicitud. Permite parametrizar clientes con diferentes solicitudes, poner solicitudes en una cola o registrarlas, y soportar operaciones des-hacibles.
* **Iterator:** Permite recorrer los elementos de una colección secuencialmente sin exponer su representación subyacente.
* **Mediator:** Reduce las dependencias caóticas entre objetos, forzándolos a comunicarse solo a través de un objeto mediador.
* **Memento:** Permite guardar y restaurar el estado anterior de un objeto sin revelar los detalles de su implementación.
* **Observer:** Define un mecanismo de suscripción para notificar a múltiples objetos sobre cualquier evento que le suceda al objeto que están observando.
* **State:** Permite que un objeto altere su comportamiento cuando su estado interno cambia. Parece como si el objeto hubiera cambiado de clase.
* **Strategy:** Define una familia de algoritmos, encapsula cada uno de ellos y los hace intercambiables. Permite que el algoritmo varíe independientemente de los clientes que lo utilizan.
* **Template Method:** Define el esqueleto de un algoritmo en una superclase, pero permite que las subclases sobrescriban pasos específicos del algoritmo sin cambiar su estructura general.
* **Visitor:** Permite separar algoritmos de los objetos sobre los que operan. Puedes añadir nuevas operaciones a una clase sin modificarla.

---

* **Fuente de referencia:** [Refactoring.Guru - El catálogo de patrones de diseño](https://refactoring.guru/es/design-patterns/catalog)
* **IA**
  * **Name**: Gemini 2.5 Flash.
  * **Prompt**: "Necesito que me explique el patrón [patron-name] para [descripción] aplicado en [lang-name], primero explicando que resuelve, luego el código lo mas simple posible para entender el concepto y otro código aparte mas avanzado y por último un resumen al respecto."
    El prompt utilizado de cada patrón se puede ver al final del mismo, pero la estructura es la misma que se especifica anteriormente.
