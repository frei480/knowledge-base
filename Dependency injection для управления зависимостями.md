---
tags:
  - АрхитектураПО
---

# Подробный Разбор [[Dependency Injection основы и применение|Dependency Injection]] (DI) Для Управления Зависимостями

**Dependency Injection (DI)** — это архитектурный паттерн, реализующий принцип *Inversion of Control (IoC)*. Его суть: класс не создаёт свои зависимости самостоятельно, а получает их извне. Это фундаментальный подход для создания гибкого, тестируемого и поддерживаемого кода.

---

## 1. Основные Концепции
- **Зависимость (Dependency)**: Любой внешний объект, требуемый классу для работы (база данных, логгер, API-клиент).
- **Внедрение (Injection)**: Процесс передачи зависимости в класс извне.
- **Inversion of Control (IoC)**: Класс не управляет своими зависимостями — контроль "инвертируется" к внешнему контейнеру.

---

## 2. Зачем Это Нужно?
- **Устраняет жёсткую связанность**
  Классы зависят от *интерфейсов*, а не от конкретных реализаций.
- **Упрощает тестирование**
  Зависимости можно подменить моками/стабами.
- **Улучшает переиспользование кода**
  Компоненты становятся независимыми модулями.
- **Централизует конфигурацию**
  Управление зависимостями в одном месте (DI-контейнер).

---

## 3. Способы Внедрения Зависимостей
### a) Constructor Injection (Внедрение Через конструктор)
**Самый предпочтительный способ!**
```java
class OrderService {
  private final PaymentGateway gateway;
  private final Logger logger;

  // Зависимости передаются при создании объекта
  public OrderService(PaymentGateway gateway, Logger logger) {
    this.gateway = gateway;
    this.logger = logger;
  }
  
  public void processOrder(Order order) {
    gateway.charge(order.getAmount());
    logger.log("Order processed");
  }
}
```
**Преимущества**:
- Зависимости обязательны (объект не создаётся без них)
- Иммутабельность (`final` поля)
- Чёткая видимость зависимостей

### b) Setter Injection (Внедрение Через сеттеры)
```java
class UserService {
  private UserRepository repository;
  
  public void setRepository(UserRepository repository) {
    this.repository = repository;
  }
}
```
**Когда использовать**:
- Для опциональных зависимостей
- Когда объект должен быть переконфигурируемым

### c) Interface Injection
Редко используется. Требует реализации специального интерфейса для внедрения.

---

## 4. DI-контейнеры: Автоматизация Управления
**DI-контейнер** — фреймворк, который:
- Автоматически создаёт объекты
- Разрешает их зависимости
- Управляет жизненным циклом

**Популярные реализации**:
- **Java**: Spring Framework, Google Guice
- **C#**: .NET Core DI, Autofac
- **JavaScript/TypeScript**: InversifyJS, NestJS DI
- **Python**: Dependency Injector, FastAPI DI

---

## 5. Как Работает DI-контейнер (на Примере Spring)
**Шаг 1: Объявление зависимостей**
```java
public interface PaymentGateway {
  void charge(double amount);
}

@Component // Помечаем как компонент контейнера
public class StripeGateway implements PaymentGateway {
  public void charge(double amount) { /* ... */ }
}
```

**Шаг 2: Внедрение через конструктор**
```java
@Service
public class OrderService {
  private final PaymentGateway gateway;
  
  // Контейнер автоматически найдёт реализацию PaymentGateway
  public OrderService(PaymentGateway gateway) {
    this.gateway = gateway;
  }
}
```

**Шаг 3: Конфигурация контейнера** (опционально)
```java
@Configuration
public class AppConfig {
  @Bean
  public PaymentGateway paymentGateway() {
    return new StripeGateway(API_KEY); // Кастомизация
  }
}
```

---

## 6. Ключевые Преимущества DI
| Без DI | С DI |
|--------|------|
| `OrderService service = new OrderService(new StripeGateway(), new FileLogger());` | `OrderService service = container.get(OrderService.class);` |
| Жёсткая связанность | Слабая связанность |
| Тестирование требует реальных зависимостей | Легкая подмена зависимостей на моки |
| Изменение зависимости = перекомпиляция | Конфигурация изменяется в одном месте |

**Пример теста с DI**:
```java
class OrderServiceTest {
  @Test
  void processOrder_ShouldChargePayment() {
    // 1. Создаём мок зависимости
    PaymentGateway mockGateway = Mockito.mock(PaymentGateway.class);
    
    // 2. Внедряем мок в тестируемый сервис
    OrderService service = new OrderService(mockGateway, new TestLogger());
    
    // 3. Вызываем метод
    service.processOrder(new Order(100.0));
    
    // 4. Проверяем взаимодействие
    Mockito.verify(mockGateway).charge(100.0);
  }
}
```

---

## 7. Жизненные Циклы Объектов
DI-контейнеры управляют временем жизни объектов:
- **Singleton**: Один экземпляр на всё приложение (по умолчанию в Spring)
- **Prototype**: Новый экземпляр при каждом запросе
- **Request/Session**: Экземпляр на HTTP-запрос или сессию (в веб-приложениях)

---

## 8. Типичные Ошибки При Использовании DI
1. **Service Locator вместо DI**
   Антипаттерн: класс сам запрашивает зависимости из контейнера.
   ```java
   // ПЛОХО! Скрытая зависимость
   class OrderService {
     public void processOrder() {
       PaymentGateway gateway = Container.get(PaymentGateway.class);
     }
   }
   ```

2. **Циклические зависимости**
   `A → B → A`. Решения:
   - Рефакторинг (выделить общий функционал в третий класс)
   - Использование setter/field injection
   - `@Lazy` в Spring (создание прокси)

3. **Over-injection**
   Более 4-5 зависимостей в конструкторе → признак нарушения SRP.

---

## 9. Практические Рекомендации
1. **Используйте Constructor Injection везде где возможно**
2. **Программируйте против интерфейсов**
3. **Избегайте аннотаций, привязывающих код к конкретному DI-фреймворку** (где возможно)
4. **Для простых проектов достаточно "ручного DI"**:
   ```java
   public static void main(String[] args) {
     PaymentGateway gateway = new StripeGateway(API_KEY);
     Logger logger = new CloudLogger();
     OrderService service = new OrderService(gateway, logger);
     service.run();
   }
   ```

---

# Итог
**Dependency Injection** — это не просто "передача параметров в конструктор", а философия проектирования:
- 🛠️ Инструмент для борьбы с жёсткой связанностью
- 🧪 Основа для тестируемости
- 🔧 Механизм соблюдения [[SOLID принципы|SOLID-принципов]]
- 🌉 Мост между компонентами системы

Правильное применение DI делает код:
- **Гибким**: Замена реализаций без изменения клиентского кода
- **Понятным**: Явное объявление зависимостей
- **Надёжным**: Упрощение изоляции и тестирования компонентов