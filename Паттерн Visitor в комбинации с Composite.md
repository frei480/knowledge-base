---
tags:
  - АрхитектураПО
---

# Паттерн [[паттерн visitor|Visitor]] в комбинации с [[паттерн composite|Composite]]

Комбинация паттернов **Visitor** и **Composite** образует мощный инструмент для обработки сложных иерархических структур данных. Эта комбинация особенно эффективна, когда нужно выполнять операции над древовидными структурами, где узлы могут быть как простыми элементами, так и составными объектами.

## Почему эта комбинация так эффективна?

1. **Composite**:
   - Организует объекты в древовидные структуры
   - Позволяет единообразно работать с отдельными объектами и их композициями
   - Создает иерархию "часть-целое"

2. **Visitor**:
   - Добавляет новые операции без изменения классов элементов
   - Централизует логику обработки в одном месте
   - Обеспечивает двойную диспетчеризацию

Вместе они позволяют:
- Единообразно обрабатывать сложные иерархии
- Легко добавлять новые операции
- Сохранять чистоту кода элементов иерархии
- Эффективно обходить древовидные структуры

## Пример: Обработка AST (Abstract Syntax Tree)

Рассмотрим пример обработки абстрактного синтаксического дерева для простого языка программирования.

### Реализация на Python

```python
from abc import ABC, abstractmethod

# Базовый класс для узлов AST (Composite)
class AstNode(ABC):
    @abstractmethod
    def accept(self, visitor):
        pass

# Листовые узлы
class NumberNode(AstNode):
    def __init__(self, value):
        self.value = value
        
    def accept(self, visitor):
        return visitor.visit_number(self)

class VariableNode(AstNode):
    def __init__(self, name):
        self.name = name
        
    def accept(self, visitor):
        return visitor.visit_variable(self)

# Составные узлы
class BinaryOpNode(AstNode):
    def __init__(self, left, op, right):
        self.left = left
        self.op = op
        self.right = right
        
    def accept(self, visitor):
        return visitor.visit_binary_op(self)

class FunctionCallNode(AstNode):
    def __init__(self, name, args):
        self.name = name
        self.args = args
        
    def accept(self, visitor):
        return visitor.visit_function_call(self)

# Базовый посетитель
class AstVisitor(ABC):
    @abstractmethod
    def visit_number(self, node):
        pass
    
    @abstractmethod
    def visit_variable(self, node):
        pass
    
    @abstractmethod
    def visit_binary_op(self, node):
        pass
    
    @abstractmethod
    def visit_function_call(self, node):
        pass

# Конкретные посетители
class PrintVisitor(AstVisitor):
    def visit_number(self, node):
        return str(node.value)
    
    def visit_variable(self, node):
        return node.name
    
    def visit_binary_op(self, node):
        left = node.left.accept(self)
        right = node.right.accept(self)
        return f"({left} {node.op} {right})"
    
    def visit_function_call(self, node):
        args = ", ".join(arg.accept(self) for arg in node.args)
        return f"{node.name}({args})"

class EvalVisitor(AstVisitor):
    def __init__(self, variables=None):
        self.variables = variables or {}
    
    def visit_number(self, node):
        return node.value
    
    def visit_variable(self, node):
        return self.variables.get(node.name, 0)
    
    def visit_binary_op(self, node):
        left = node.left.accept(self)
        right = node.right.accept(self)
        
        if node.op == '+': return left + right
        if node.op == '-': return left - right
        if node.op == '*': return left * right
        if node.op == '/': return left / right
        raise ValueError(f"Unknown operator: {node.op}")
    
    def visit_function_call(self, node):
        if node.name == 'pow':
            if len(node.args) != 2:
                raise ValueError("pow() requires exactly 2 arguments")
            base = node.args[0].accept(self)
            exponent = node.args[1].accept(self)
            return pow(base, exponent)
        raise ValueError(f"Unknown function: {node.name}")

# Клиентский код
if __name__ == "__main__":
    # Строим AST для выражения: (2 * x) + pow(3, y)
    expr = BinaryOpNode(
        BinaryOpNode(NumberNode(2), '*', VariableNode('x')),
        '+',
        FunctionCallNode('pow', [NumberNode(3), VariableNode('y')])
    )
    
    # Печать выражения
    printer = PrintVisitor()
    print("AST representation:", expr.accept(printer))
    
    # Вычисление выражения
    evaluator = EvalVisitor({'x': 5, 'y': 2})
    result = expr.accept(evaluator)
    print("Result:", result)  # (2 * 5) + pow(3, 2) = 10 + 9 = 19
```

### Реализация на C#

```csharp
using System;
using System.Collections.Generic;
using System.Text;

// Базовый класс для узлов AST (Composite)
public abstract class AstNode
{
    public abstract T Accept<T>(IAstVisitor<T> visitor);
}

// Листовые узлы
public class NumberNode : AstNode
{
    public double Value { get; }
    
    public NumberNode(double value) => Value = value;
    
    public override T Accept<T>(IAstVisitor<T> visitor) 
        => visitor.Visit(this);
}

public class VariableNode : AstNode
{
    public string Name { get; }
    
    public VariableNode(string name) => Name = name;
    
    public override T Accept<T>(IAstVisitor<T> visitor) 
        => visitor.Visit(this);
}

// Составные узлы
public class BinaryOpNode : AstNode
{
    public AstNode Left { get; }
    public string Op { get; }
    public AstNode Right { get; }
    
    public BinaryOpNode(AstNode left, string op, AstNode right)
        => (Left, Op, Right) = (left, op, right);
    
    public override T Accept<T>(IAstVisitor<T> visitor) 
        => visitor.Visit(this);
}

public class FunctionCallNode : AstNode
{
    public string Name { get; }
    public List<AstNode> Args { get; }
    
    public FunctionCallNode(string name, List<AstNode> args)
        => (Name, Args) = (name, args);
    
    public override T Accept<T>(IAstVisitor<T> visitor) 
        => visitor.Visit(this);
}

// Интерфейс посетителя
public interface IAstVisitor<T>
{
    T Visit(NumberNode node);
    T Visit(VariableNode node);
    T Visit(BinaryOpNode node);
    T Visit(FunctionCallNode node);
}

// Конкретные посетители
public class PrintVisitor : IAstVisitor<string>
{
    public string Visit(NumberNode node) => node.Value.ToString();
    
    public string Visit(VariableNode node) => node.Name;
    
    public string Visit(BinaryOpNode node)
    {
        var left = node.Left.Accept(this);
        var right = node.Right.Accept(this);
        return $"({left} {node.Op} {right})";
    }
    
    public string Visit(FunctionCallNode node)
    {
        var args = new StringBuilder();
        foreach (var arg in node.Args)
        {
            if (args.Length > 0) args.Append(", ");
            args.Append(arg.Accept(this));
        }
        return $"{node.Name}({args})";
    }
}

public class EvalVisitor : IAstVisitor<double>
{
    private readonly Dictionary<string, double> _variables;
    
    public EvalVisitor(Dictionary<string, double> variables = null)
    {
        _variables = variables ?? new Dictionary<string, double>();
    }
    
    public double Visit(NumberNode node) => node.Value;
    
    public double Visit(VariableNode node)
    {
        if (_variables.TryGetValue(node.Name, out var value))
            return value;
        return 0;
    }
    
    public double Visit(BinaryOpNode node)
    {
        var left = node.Left.Accept(this);
        var right = node.Right.Accept(this);
        
        return node.Op switch
        {
            "+" => left + right,
            "-" => left - right,
            "*" => left * right,
            "/" => left / right,
            _ => throw new ArgumentException($"Unknown operator: {node.Op}")
        };
    }
    
    public double Visit(FunctionCallNode node)
    {
        if (node.Name == "pow" && node.Args.Count == 2)
        {
            var baseVal = node.Args[0].Accept(this);
            var exponent = node.Args[1].Accept(this);
            return Math.Pow(baseVal, exponent);
        }
        throw new ArgumentException($"Unknown function: {node.Name}");
    }
}

// Клиентский код
class Program
{
    static void Main()
    {
        // Строим AST для выражения: (2 * x) + pow(3, y)
        var expr = new BinaryOpNode(
            new BinaryOpNode(
                new NumberNode(2), "*", new VariableNode("x")),
            "+",
            new FunctionCallNode("pow", new List<AstNode>
            {
                new NumberNode(3),
                new VariableNode("y")
            })
        );
        
        // Печать выражения
        var printer = new PrintVisitor();
        Console.WriteLine($"AST representation: {expr.Accept(printer)}");
        
        // Вычисление выражения
        var variables = new Dictionary<string, double>
        {
            ["x"] = 5,
            ["y"] = 2
        };
        var evaluator = new EvalVisitor(variables);
        var result = expr.Accept(evaluator);
        Console.WriteLine($"Result: {result}"); // (2 * 5) + pow(3, 2) = 10 + 9 = 19
    }
}
```

## Преимущества комбинации Composite + Visitor

1. **Единообразие обработки**:
   - Позволяет обрабатывать как отдельные элементы, так и сложные композиции одинаковым образом
   - Упрощает добавление новых операций для всей иерархии

2. **Расширяемость**:
   - Новые операции добавляются через создание новых посетителей
   - Новые типы узлов добавляются без изменения существующих посетителей

3. **Разделение обязанностей**:
   - Структура данных (Composite) отделена от алгоритмов (Visitor)
   - Каждый класс фокусируется на своей области ответственности

4. **Эффективный обход**:
   - Посетитель контролирует порядок обхода дерева
   - Можно реализовать различные стратегии обхода (DFS, BFS)

## Практические применения

1. **Компиляторы и интерпретаторы**:
   - Обход AST для генерации кода
   - Статический анализ кода
   - Оптимизации

2. **UI фреймворки**:
   - Отрисовка компонентов
   - Обработка событий
   - Валидация форм

3. **Документо-ориентированные системы**:
   - Экспорт документов в разные форматы
   - Поиск по сложным документам
   - Преобразование структуры документа

4. **Игровые движки**:
   - Обработка сцены (рендеринг, физика)
   - Сериализация игровых объектов
   - Поиск путей в сложных средах

## Антипаттерны, против которых борется эта комбинация

1. **Switch Statements (Код с большим количеством switch/case)**:
   - Проблема: Разрозненная логика обработки разных типов
   - Решение: Логика сгруппирована в посетителях

2. **God Object (Божественный объект)**:
   - Проблема: Один класс содержит всю логику обработки
   - Решение: Разделение на узлы и посетителей

3. **Feature Envy (Зависть к функциональности)**:
   - Проблема: Методы одного класса чрезмерно используют данные другого
   - Решение: Операции над данными инкапсулированы в посетителях

4. **Violation of OCP (Нарушение принципа открытости/закрытости)**:
   - Проблема: Добавление нового типа требует изменения существующего кода
   - Решение: Новые типы добавляются без изменения посетителей

## Когда использовать комбинацию Composite + Visitor

1. **Сложные иерархии объектов**:
   - Когда у вас есть древовидная структура с различными типами узлов

2. **Множество операций над структурой**:
   - Когда нужно выполнять различные операции над одними и теми же данными

3. **Частые изменения операций**:
   - Когда новые операции добавляются чаще, чем изменяется структура данных

4. **Разделение структуры и поведения**:
   - Когда важно сохранить чистоту классов данных

## Ограничения

1. **Сложность добавления новых типов узлов**:
   - Добавление нового типа узла требует обновления всех посетителей

2. **Нарушение инкапсуляции**:
   - Посетителям часто требуется доступ к внутреннему состоянию узлов

3. **Сложность отладки**:
   - Поток выполнения может быть неочевидным из-за двойной диспетчеризации

4. **Производительность**:
   - Виртуальные вызовы и рекурсия могут влиять на производительность

Комбинация паттернов Composite и Visitor особенно мощна при работе с абстрактными синтаксическими деревьями (AST), документами XML/JSON, UI компонентами и другими сложными иерархическими структурами. Она обеспечивает гибкость и расширяемость, позволяя добавлять новые операции без изменения существующей структуры данных.