---
tags:
  - АрхитектураПО
---

### Паттерн Visitor (Посетитель)
**Назначение:**  
Позволяет добавлять новые операции к объектам без изменения их классов. Инкапсулирует логику операций в отдельном классе-посетителе, что упрощает добавление новых операций и соблюдает принцип открытости/закрытости.

#### Ключевые аспекты:
1. **Двойная диспетчеризация:** Выбор метода зависит от типов посетителя и элемента
2. **Разделение логики:** Операции выносятся из классов объектов
3. **Обход структур:** Часто используется для работы со сложными структурами (деревья, AST)

---

### Пример на Python
**Сценарий:** Генерация отчетов для разных типов документов

```python
from abc import ABC, abstractmethod

# Интерфейс элемента (принимает посетителя)
class Document(ABC):
    @abstractmethod
    def accept(self, visitor):
        pass

# Конкретные элементы
class PDFDocument(Document):
    def accept(self, visitor):
        visitor.visit_pdf(self)

class WordDocument(Document):
    def accept(self, visitor):
        visitor.visit_word(self)

class MarkdownDocument(Document):
    def accept(self, visitor):
        visitor.visit_markdown(self)

# Интерфейс посетителя
class DocumentVisitor(ABC):
    @abstractmethod
    def visit_pdf(self, pdf):
        pass
    
    @abstractmethod
    def visit_word(self, word):
        pass
    
    @abstractmethod
    def visit_markdown(self, md):
        pass

# Конкретные посетители
class ExportVisitor(DocumentVisitor):
    def visit_pdf(self, pdf):
        print("Экспорт PDF в формат ISO 32000-1:2008")
    
    def visit_word(self, word):
        print("Конвертация DOCX в OOXML формат")
    
    def visit_markdown(self, md):
        print("Преобразование Markdown в HTML")

class MetadataVisitor(DocumentVisitor):
    def visit_pdf(self, pdf):
        print("Извлечение метаданных: автор, теги, дата создания")
    
    def visit_word(self, word):
        print("Получение свойств документа: редакции, комментарии")
    
    def visit_markdown(self, md):
        print("Анализ метаданных: заголовки, ссылки, версия")

# Клиентский код
documents = [
    PDFDocument(), 
    WordDocument(), 
    MarkdownDocument()
]

export_visitor = ExportVisitor()
meta_visitor = MetadataVisitor()

print("=== Экспорт документов ===")
for doc in documents:
    doc.accept(export_visitor)

print("\n=== Извлечение метаданных ===")
for doc in documents:
    doc.accept(meta_visitor)
```

**Вывод:**
```
=== Экспорт документов ===
Экспорт PDF в формат ISO 32000-1:2008
Конвертация DOCX в OOXML формат
Преобразование Markdown в HTML

=== Извлечение метаданных ===
Извлечение метаданных: автор, теги, дата создания
Получение свойств документа: редакции, комментарии
Анализ метаданных: заголовки, ссылки, версия
```

---

### Пример на C#
**Сценарий:** Анализ программного кода (AST обход)

```csharp
using System;
using System.Collections.Generic;

// Интерфейс элемента AST
interface IAstNode {
    void Accept(IAstVisitor visitor);
}

// Конкретные элементы
class VariableNode : IAstNode {
    public string Name { get; }
    public VariableNode(string name) => Name = name;
    public void Accept(IAstVisitor visitor) => visitor.Visit(this);
}

class AssignmentNode : IAstNode {
    public IAstNode Left { get; }
    public IAstNode Right { get; }
    public AssignmentNode(IAstNode left, IAstNode right) => 
        (Left, Right) = (left, right);
    public void Accept(IAstVisitor visitor) => visitor.Visit(this);
}

class BinaryOperationNode : IAstNode {
    public string Operator { get; }
    public IAstNode Left { get; }
    public IAstNode Right { get; }
    public BinaryOperationNode(string op, IAstNode left, IAstNode right) => 
        (Operator, Left, Right) = (op, left, right);
    public void Accept(IAstVisitor visitor) => visitor.Visit(this);
}

// Интерфейс посетителя
interface IAstVisitor {
    void Visit(VariableNode node);
    void Visit(AssignmentNode node);
    void Visit(BinaryOperationNode node);
}

// Конкретные посетители
class PrintVisitor : IAstVisitor {
    public void Visit(VariableNode node) => Console.Write(node.Name);
    
    public void Visit(AssignmentNode node) {
        node.Left.Accept(this);
        Console.Write(" = ");
        node.Right.Accept(this);
    }
    
    public void Visit(BinaryOperationNode node) {
        Console.Write("(");
        node.Left.Accept(this);
        Console.Write($" {node.Operator} ");
        node.Right.Accept(this);
        Console.Write(")");
    }
}

class TypeCheckVisitor : IAstVisitor {
    public void Visit(VariableNode node) => 
        Console.WriteLine($"Проверка типа переменной {node.Name}");
    
    public void Visit(AssignmentNode node) {
        Console.WriteLine("Проверка совместимости типов присваивания");
        node.Left.Accept(this);
        node.Right.Accept(this);
    }
    
    public void Visit(BinaryOperationNode node) {
        Console.WriteLine($"Проверка оператора {node.Operator}");
        node.Left.Accept(this);
        node.Right.Accept(this);
    }
}

// Клиентский код
class Program {
    static void Main() {
        // Строим AST: x = (a + 10) * b
        var ast = new AssignmentNode(
            new VariableNode("x"),
            new BinaryOperationNode("*",
                new BinaryOperationNode("+",
                    new VariableNode("a"),
                    new VariableNode("10")),
                new VariableNode("b"))
        );
        
        Console.WriteLine("=== Печать AST ===");
        ast.Accept(new PrintVisitor());
        
        Console.WriteLine("\n\n=== Проверка типов ===");
        ast.Accept(new TypeCheckVisitor());
    }
}
```

**Вывод:**
```
=== Печать AST ===
x = ((a + 10) * b)

=== Проверка типов ===
Проверка совместимости типов присваивания
Проверка типа переменной x
Проверка оператора *
Проверка оператора +
Проверка типа переменной a
Проверка типа переменной 10
Проверка типа переменной b
```

---

### Против каких антипаттернов борется Visitor?
1. **God Object (Божественный объект):**  
   **Проблема:** Один класс содержит слишком много несвязанной логики  
   **Решение:** Распределение операций по специализированным посетителям

2. **Violation of OCP (Нарушение принципа открытости/закрытости):**  
   **Проблема:** Добавление новых операций требует изменения существующих классов  
   **Решение:** Новые операции добавляются через новых посетителей

3. **Feature Envy (Зависть к функциональности):**  
   **Проблема:** Логика операций не принадлежит объектам, над которыми выполняется  
   **Решение:** Логика инкапсулируется в посетителях

4. **Shotgun Surgery (Выстрел дробью):**  
   **Проблема:** Изменение операции требует правок во многих классах  
   **Решение:** Все изменения операции локализуются в одном посетителе

5. **Poltergeists (Полтергейсты):**  
   **Проблема:** Временные объекты-координаторы, выполняющие минимальную работу  
   **Решение:** Посетители содержат значимую логику обработки

---

### Когда использовать Visitor?
1. При работе со сложными структурами объектов (AST, DOM, компиляторы)
2. Когда нужно выполнять различные операции над объектами с разными типами
3. Для отделения алгоритмов от структур данных
4. Когда новые операции добавляются чаще, чем новые типы элементов
5. Для реализации double dispatch (двойной диспетчеризации)

---

### Ограничения паттерна:
1. **Сложность добавления элементов:** Добавление нового типа элемента требует изменения всех посетителей
2. **Нарушение инкапсуляции:** Посетители часто требуют публичного доступа к внутреннему состоянию объектов
3. **Не для простых случаев:** Избыточен для систем с редкими изменениями операций
4. **Сложность отладки:** Поток выполнения становится менее очевидным

> **Важно:** Visitor особенно [[Паттерн Visitor в комбинации с Composite|эффективен]] в комбинации с паттерном [[паттерн composite|Composite]] для обработки древовидных структур. В языках с поддержкой pattern matching (C# 8.0+, Python 3.10+) некоторые сценарии Visitor можно реализовать [[Альтернатива Visitor с Pattern Matching|проще]].