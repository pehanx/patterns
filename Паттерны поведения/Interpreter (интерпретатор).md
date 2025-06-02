**Паттерн Interpreter (Интерпретатор)** используется для определения грамматики языка и интерпретации его предложений. В iOS он особенно полезен для:

- Разбора математических выражений
- Обработки пользовательских запросов (например, поиск с фильтрами)
- Конвертации форматов данных (JSON → Swift-модели)

---

## **Паттерн Interpreter в Swift: Реальные примеры из iOS**

### **Пример 1: Калькулятор математических выражений**

#### **Задача:**
Реализовать интерпретатор для вычисления простых выражений:
- `"2 + 3 * 4"` → `14`
- `"(2 + 3) * 4"` → `20`

---

#### **1. Базовые компоненты Interpreter**

```swift
// Абстрактное выражение
protocol Expression {
    func interpret(_ context: inout [String: Int]) -> Int
}

// Терминальное выражение (число)
class NumberExpression: Expression {
    private let value: Int
    
    init(_ value: Int) {
        self.value = value
    }
    
    func interpret(_ context: inout [String: Int]) -> Int {
        return value
    }
}

// Терминальное выражение (переменная)
class VariableExpression: Expression {
    private let name: String
    
    init(_ name: String) {
        self.name = name
    }
    
    func interpret(_ context: inout [String: Int]) -> Int {
        return context[name] ?? 0
    }
}
```

#### **2. Нетерминальные выражения (операции)**

```swift
// Сложение
class AddExpression: Expression {
    private let left: Expression
    private let right: Expression
    
    init(_ left: Expression, _ right: Expression) {
        self.left = left
        self.right = right
    }
    
    func interpret(_ context: inout [String: Int]) -> Int {
        return left.interpret(&context) + right.interpret(&context)
    }
}

// Умножение
class MultiplyExpression: Expression {
    private let left: Expression
    private let right: Expression
    
    init(_ left: Expression, _ right: Expression) {
        self.left = left
        self.right = right
    }
    
    func interpret(_ context: inout [String: Int]) -> Int {
        return left.interpret(&context) * right.interpret(&context)
    }
}
```

#### **3. Парсер выражений**

```swift
class ExpressionParser {
    static func parse(_ input: String) -> Expression {
        // Упрощенный парсер (реализация может быть сложнее)
        let tokens = input.components(separatedBy: " ")
        var stack: [Expression] = []
        
        for token in tokens {
            switch token {
            case "+":
                let right = stack.removeLast()
                let left = stack.removeLast()
                stack.append(AddExpression(left, right))
            case "*":
                let right = stack.removeLast()
                let left = stack.removeLast()
                stack.append(MultiplyExpression(left, right))
            default:
                if let number = Int(token) {
                    stack.append(NumberExpression(number))
                } else {
                    stack.append(VariableExpression(token))
                }
            }
        }
        
        return stack.first ?? NumberExpression(0)
    }
}
```

#### **4. Использование**

```swift
// Контекст (переменные)
var context: [String: Int] = ["x": 5, "y": 10]

// Разбор и вычисление выражения
let expression = ExpressionParser.parse("2 x * y +") // 2 * x + y
let result = expression.interpret(&context)

print(result) // 20 (2*5 + 10)
```

---

### **Пример 2: Фильтрация данных**

#### **Задача:**
Интерпретировать пользовательские запросы для фильтрации массива:

```swift
// Протокол выражения для фильтрации
protocol FilterExpression {
    func filter(_ items: [String]) -> [String]
}

// Фильтр по ключевому слову
class ContainsFilter: FilterExpression {
    private let keyword: String
    
    init(_ keyword: String) {
        self.keyword = keyword
    }
    
    func filter(_ items: [String]) -> [String] {
        return items.filter { $0.contains(keyword) }
    }
}

// Комбинированный фильтр (AND)
class AndFilter: FilterExpression {
    private let left: FilterExpression
    private let right: FilterExpression
    
    init(_ left: FilterExpression, _ right: FilterExpression) {
        self.left = left
        self.right = right
    }
    
    func filter(_ items: [String]) -> [String] {
        return right.filter(left.filter(items))
    }
}

// Использование
let data = ["Apple", "Microsoft", "Amazon", "Alphabet"]
let filter = AndFilter(ContainsFilter("A"), ContainsFilter("e"))
let result = filter.filter(data) // ["Apple", "Amazon"]
```

---

## **Где применяется в iOS?**

1. **Core Data/NSPredicate:**
   ```swift
   // Фактически использует Interpreter для запросов
   let predicate = NSPredicate(format: "name CONTAINS %@ AND age > %d", "John", 25)
   ```

2. **Форматирование строк:**
   ```swift
   // NSDateFormatter интерпретирует строки формата
   let formatter = DateFormatter()
   formatter.dateFormat = "yyyy-MM-dd"
   ```

3. **JSON-парсеры:**
   - `JSONDecoder` интерпретирует JSON согласно моделям Swift

---

## **Плюсы и минусы**

✅ **Плюсы:**
- Гибкость в добавлении новых правил грамматики
- Отделяет логику разбора от бизнес-логики

❌ **Минусы:**
- Сложность для больших грамматик
- Неэффективность для простых задач

---

## **Сравнение с другими паттернами**

- **Composite:** Строит древовидные структуры, но без логики интерпретации
- **Visitor:** Отделяет алгоритмы от структуры объектов
- **Flyweight:** Оптимизирует память для терминальных символов
