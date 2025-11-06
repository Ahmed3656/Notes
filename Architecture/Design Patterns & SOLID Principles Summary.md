
### Design Patterns

| **Pattern**     | **Description**                                                                                                        | **Use Case**                                                |
| --------------- | ---------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------- |
| **Null Object** | Provides a **default "do-nothing" behavior** instead of returning `null` to prevent errors.                            | Avoiding `null` checks, handling missing configurations.    |
| **Builder**     | Separates **object construction** from its representation, allowing step-by-step creation.                             | Creating complex objects with many parameters.              |
| **Singleton**   | Ensures a class has **only one instance** and provides a **global access point**.                                      | Database connections, caching, logging.                     |
| **Facade**      | Provides a **simplified interface** to a complex system, reducing dependencies between clients and subsystems.         | Wrapping complex APIs, simplifying library usage.           |
| **Command**     | Encapsulates **requests as objects**, allowing them to be executed, delayed, or queued.                                | Undo/redo systems, task scheduling, event handling.         |
| **Factory**     | Defines an **interface for creating objects**, allowing subclasses to determine the concrete implementation.           | Object creation when types are unknown at compile time.     |
| **Observer**    | Implements a **one-to-many dependency**, so multiple objects get notified when the subject changes.                    | Event-driven systems, UI updates, React state management.   |
| **Strategy**    | Defines a **family of interchangeable algorithms**, letting clients choose at runtime.                                 | Payment processing, sorting algorithms, AI decision-making. |
| **Adapter**     | Converts an existing interface into one that a client expects, acting as a **bridge between incompatible interfaces**. | Integrating legacy systems, API compatibility layers.       |
| **Decorator**   | Dynamically **adds responsibilities** to an object without modifying its structure.                                    | Extending UI components, adding middleware.                 |
| **Proxy**       | Acts as a placeholder to **control access** to another object, often for security or optimization.                     | Security proxies, caching, lazy loading.                    |

---

### SOLID Principles

| **Principle**                   | **Description**                                                                                                                                                | **Use Case**                                                                                                                            |
| ------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------- |
| **Single Responsibility (SRP)** | A class should have **only one reason to change**, meaning it should only handle **one responsibility**.                                                       | Keeping separate classes for logging, data processing, and UI rendering.                                                                |
| **Open/Closed (OCP)**           | Software entities should be **open for extension but closed for modification**—meaning you should be able to add functionality without changing existing code. | Using inheritance or interfaces to extend behavior without modifying existing classes.                                                  |
| **Liskov Substitution (LSP)**   | Subclasses should be **substitutable for their base classes** without breaking functionality.                                                                  | Ensuring derived classes don’t remove behavior from parent classes (e.g., a Square class shouldn’t break a Rectangle class’s behavior). |
| **Interface Segregation (ISP)** | Clients should **not be forced to depend on interfaces they don’t use**—instead, split large interfaces into smaller, more specific ones.                      | Having multiple smaller interfaces instead of one large, bloated interface.                                                             |
| **Dependency Inversion (DIP)**  | High-level modules should **not depend on low-level modules**—both should depend on abstractions (interfaces).                                                 | Using dependency injection to allow flexibility and easier testing.                                                                     |

---

### Examples

### Null Object Pattern
```typescript
interface Logger {
  log(message: string): void;
}

class ConsoleLogger implements Logger {
  log(message: string) {
    console.log(message);
  }
}

class NullLogger implements Logger {
  log(message: string) {} // Does nothing
}

function getLogger(type: string): Logger {
  return type === "console" ? new ConsoleLogger() : new NullLogger();
}

const logger = getLogger(""); // Returns NullLogger
logger.log("This won't print anything.");
