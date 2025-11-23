---
tags:
  - python
---
Чтобы запретить изменение атрибута после его первой установки, мы можем создать перезаписываемый дескриптор, который в методе `__set__` будет проверять, было ли значение уже установлено.
```python
class ReadOnlyAttribute:
    """
    Дескриптор, который позволяет установить значение атрибута только один раз.
    После первой установки атрибут становится доступным только для чтения.
    """
    def __init__(self, name=None):
        self.public_name = name
        self.private_name = f'_{name}' if name else '_value' # Внутреннее имя для хранения значения

    # Метод __set_name__ вызывается автоматически Python'ом при создании класса.
    # Он передает имя атрибута, которым дескриптор будет известен в классе.
    def __set_name__(self, owner, name):
        if self.public_name is None:
            self.public_name = name
            self.private_name = f'_{name}'

    def __get__(self, instance, owner):
        if instance is None:
            return self
        # Возвращаем значение из внутреннего хранилища экземпляра
        return getattr(instance, self.private_name, None)

    def __set__(self, instance, value):
        # Проверяем, установлено ли значение уже
        if hasattr(instance, self.private_name) and getattr(instance, self.private_name) is not None:
            raise AttributeError(f"Атрибут '{self.public_name}' может быть установлен только один раз.")
        
        # Устанавливаем значение во внутреннее хранилище экземпляра
        setattr(instance, self.private_name, value)
        print(f"Атрибут '{self.public_name}' установлен на: {value}")

    def __delete__(self, instance):
        raise AttributeError(f"Атрибут '{self.public_name}' не может быть удален.")

class MyImmutableObject:
    # Используем дескриптор для атрибутов, которые должны быть неизменяемыми
    id = ReadOnlyAttribute()
    name = ReadOnlyAttribute('name') # Можно указать имя явно, хотя __set_name__ его установит

    def __init__(self, id_val, name_val):
        self.id = id_val  # Первая установка значения
        self.name = name_val # Первая установка значения

print("Создание объекта 1:")
obj1 = MyImmutableObject(123, "Immutable Item A")
print(f"ID: {obj1.id}, Name: {obj1.name}")

print("\nПопытка изменить id:")
try:
    obj1.id = 456
except AttributeError as e:
    print(f"Ошибка: {e}")

print("\nПопытка изменить name:")
try:
    obj1.name = "New Name B"
except AttributeError as e:
    print(f"Ошибка: {e}")

print(f"\nТекущие значения: ID: {obj1.id}, Name: {obj1.name}")

print("\nСоздание объекта 2:")
obj2 = MyImmutableObject(789, "Immutable Item C")
print(f"ID: {obj2.id}, Name: {obj2.name}")

# Проверка, что дескриптор работает для каждого экземпляра отдельно
print(f"\nЗначения obj1 после создания obj2: ID: {obj1.id}, Name: {obj1.name}")
```
**Объяснение:**
1. **ReadOnlyAttribute Класс Дескриптора:**
    - **__init__(self, name=None):** Инициализирует дескриптор. Мы используем `public_name` для имени атрибута в классе, который использует дескриптор, и `private_name` для имени, под которым фактическое значение будет храниться в словаре экземпляра (`instance.__dict__`). Это важно, чтобы избежать рекурсивных вызовов `__set__` и коллизий имен.
        
    - **__set_name__(self, owner, name) (Python 3.6+):** Это очень полезный "магический" метод для дескрипторов. Python вызывает его после того, как класс, содержащий дескриптор, был создан. Он автоматически передает имя атрибута, которое было использовано для назначения дескриптора в классе (например, `id` или `name` в `MyImmutableObject`). Это позволяет дескриптору знать свое "публичное" имя без необходимости передавать его вручную в `__init__`.
        
    - **__get__(self, instance, owner):**
        - Когда мы обращаемся к атрибуту (например, obj1.id), этот метод вызывается.
        - `if instance is None: return self` обрабатывает случай, когда мы обращаемся к дескриптору через класс (MyImmutableObject.id). В этом случае возвращается сам объект дескриптора.
        - `return getattr(instance, self.private_name, None)`: Возвращает значение, хранящееся во внутреннем атрибуте (`_id` или `_name`) экземпляра. Если его еще нет, возвращает `None`.
            
    - **__set__(self, instance, value):**
        - Это сердце функциональности. Когда мы пытаемся установить атрибут (obj1.id = 123), этот метод вызывается.
        - `if hasattr(instance, self.private_name) and getattr(instance, self.private_name) is not None:` Здесь мы проверяем, существует ли уже внутренний атрибут `_id` (или `_name`) в экземпляре и не является ли его значение None.
            - Если он существует и не None, это означает, что атрибут был установлен ранее. Мы генерируем AttributeError, запрещая повторное изменение.
        - `setattr(instance, self.private_name, value)`: Если атрибут еще не был установлен, мы сохраняем новое значение в приватном атрибуте экземпляра (например, `obj1._id = 123`). Важно использовать setattr здесь, чтобы установить значение непосредственно в __dict__ экземпляра, а не вызывать __set__ дескриптора рекурсивно.
            
    - **__delete__(self, instance):**
        - Мы также перехватываем попытки удаления атрибута (например, del obj1.id) и генерируем AttributeError, так как атрибут должен быть неизменяемым.
            

**Как это работает с MyImmutableObject:**

- Когда вы определяете id = ReadOnlyAttribute() в MyImmutableObject, Python видит, что ReadOnlyAttribute является дескриптором.
    
- При создании obj1 = MyImmutableObject(123, "Immutable Item A"):
    - self.id = 123 вызывает ReadOnlyAttribute.__set__(obj1, 123). Поскольку `obj1._id` еще не существует, значение 123 сохраняется в obj1._id.
    - self.name = "Immutable Item A" вызывает ReadOnlyAttribute.__set__(obj1, "Immutable Item A"). Значение сохраняется в obj1._name.
        
- Когда вы пытаетесь `obj1.id = 456:`
    - Это снова вызывает `ReadOnlyAttribute.__set__(obj1, 456)`.
    - На этот раз `hasattr(obj1, '_id')` будет `True`, и `getattr(obj1, '_id')` будет `123` (не `None`).
    - Будет сгенерировано `AttributeError`, эффективно запрещая изменение.
        

Этот подход с использованием дескрипторов позволяет вам создавать атрибуты "только для чтения" в ваших классах, обеспечивая инкапсуляцию и предсказуемое поведение.