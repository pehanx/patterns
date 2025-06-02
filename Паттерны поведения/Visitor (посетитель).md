**Паттерн Visitor (Посетитель)** позволяет добавлять новые операции к объектам, не изменяя их классов. В iOS он особенно полезен для:

- Обработки разнородных объектов (например, элементов UI)
- Реализации операций "вне" классов объектов
- Добавления функциональности без модификации существующего кода

---

## **Паттерн Visitor в Swift: Реальные примеры из iOS**

### **Пример 1: Аналитика для разных UI-элементов**

#### **Задача:**
Отслеживать клики по разным элементам интерфейса:
- `UIButton`
- `UISwitch`
- `UITextField`

---

#### **1. Реализация паттерна**

```swift
// Протокол для элементов, принимающих посетителя
protocol AnalyticsElement {
    func accept(_ visitor: AnalyticsVisitor)
}

// Конкретные элементы
class AnalyticsButton: UIButton, AnalyticsElement {
    func accept(_ visitor: AnalyticsVisitor) {
        visitor.visit(button: self)
    }
}

class AnalyticsSwitch: UISwitch, AnalyticsElement {
    func accept(_ visitor: AnalyticsVisitor) {
        visitor.visit(switch: self)
    }
}

class AnalyticsTextField: UITextField, AnalyticsElement {
    func accept(_ visitor: AnalyticsVisitor) {
        visitor.visit(textField: self)
    }
}

// Протокол посетителя
protocol AnalyticsVisitor {
    func visit(button: AnalyticsButton)
    func visit(switch: AnalyticsSwitch)
    func visit(textField: AnalyticsTextField)
}

// Конкретный посетитель
class FirebaseAnalyticsVisitor: AnalyticsVisitor {
    func visit(button: AnalyticsButton) {
        print("Отправка события: button_click с id: \(button.accessibilityIdentifier ?? "")")
        // Firebase.Analytics.logEvent("button_click", parameters: [...])
    }
    
    func visit(switch: AnalyticsSwitch) {
        let state = `switch`.isOn ? "on" : "off"
        print("Отправка события: switch_toggle (\(state))")
    }
    
    func visit(textField: AnalyticsTextField) {
        print("Отправка события: text_edit в поле: \(textField.placeholder ?? "")")
    }
}
```

#### **2. Использование**

```swift
let elements: [AnalyticsElement] = [
    AnalyticsButton(),
    AnalyticsSwitch(),
    AnalyticsTextField()
]

let analyticsVisitor = FirebaseAnalyticsVisitor()

for element in elements {
    element.accept(analyticsVisitor)
}

/* Вывод:
Отправка события: button_click с id: 
Отправка события: switch_toggle (off)
Отправка события: text_edit в поле: 
*/
```

---

### **Пример 2: Генерация XML для модели данных**

#### **Задача:**
Преобразовать модель в XML:
- `User`
- `Product`
- `Order`

---

#### **1. Реализация**

```swift
protocol XMLExportable {
    func accept(_ visitor: XMLVisitor) -> String
}

class User: XMLExportable {
    let name: String
    let age: Int
    
    init(name: String, age: Int) {
        self.name = name
        self.age = age
    }
    
    func accept(_ visitor: XMLVisitor) -> String {
        return visitor.visit(user: self)
    }
}

protocol XMLVisitor {
    func visit(user: User) -> String
    func visit(product: Product) -> String
    func visit(order: Order) -> String
}

class XMLExportVisitor: XMLVisitor {
    func visit(user: User) -> String {
        return """
        <user>
            <name>\(user.name)</name>
            <age>\(user.age)</age>
        </user>
        """
    }
    
    // ... аналогичные реализации для Product и Order
}

// Использование
let user = User(name: "Иван", age: 30)
let visitor = XMLExportVisitor()
print(user.accept(visitor))
```

---

## **Где применяется в iOS?**

1. **Обработка разнотипных элементов коллекции**
2. **Генерация отчетов/экспорт данных**
3. **Аналитика и логирование**
4. **Реализация двойной диспетчеризации**

---

## **Плюсы и минусы**

✅ **Плюсы:**
- Добавляет операции без изменения классов
- Объединяет родственные операции в одном классе
- Упрощает добавление операций для сложных иерархий

❌ **Минусы:**
- Усложняет добавление новых классов элементов
- Нарушает инкапсуляцию (требует публичных методов)

---

## **Сравнение с другими паттернами**

- **Decorator:** Добавляет ответственности объекту, **Visitor** - внешние операции
- **Iterator:** Посещает элементы для обхода, **Visitor** - для выполнения операций
- **Composite:** Visitor часто применяется к Composite-структурам
