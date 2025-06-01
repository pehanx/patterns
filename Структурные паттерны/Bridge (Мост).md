**Паттерн Bridge (Мост)** разделяет абстракцию и реализацию, позволяя им изменяться независимо. В iOS это особенно полезно для:
- Поддержки разных версий API/UI
- Работы с несколькими поставщиками сервисов (например, платежные системы)
- Гибкого управления зависимостями

---

## **Паттерн Bridge в Swift: Реальный пример из iOS**

### **Задача:**
Разрабатываем систему обработки платежей, которая должна:
1. Работать с разными провайдерами (Stripe, PayPal)
2. Поддерживать разные типы платежей (кредитные карты, криптовалюта)
3. Легко добавлять новые провайдеры или типы платежей

---

## **1. Реализация Bridge**

### **Шаг 1: Определяем "Реализацию" (PaymentProcessor)**
```swift
// Протокол реализации (платежная система)
protocol PaymentProcessor {
    func processPayment(amount: Double, completion: @escaping (Bool) -> Void)
}
```

### **Шаг 2: Конкретные реализации (провайдеры)**
```swift
// Stripe реализация
class StripeProcessor: PaymentProcessor {
    func processPayment(amount: Double, completion: @escaping (Bool) -> Void) {
        print("Обработка платежа \(amount) через Stripe")
        completion(true)
    }
}

// PayPal реализация
class PayPalProcessor: PaymentProcessor {
    func processPayment(amount: Double, completion: @escaping (Bool) -> Void) {
        print("Обработка платежа \(amount) через PayPal")
        completion(true)
    }
}
```

### **Шаг 3: Определяем "Абстракцию" (PaymentMethod)**
```swift
// Абстракция (тип платежа)
class PaymentMethod {
    let processor: PaymentProcessor
    
    init(processor: PaymentProcessor) {
        self.processor = processor
    }
    
    func pay(amount: Double, completion: @escaping (Bool) -> Void) {
        fatalError("Метод должен быть переопределен")
    }
}
```

### **Шаг 4: Конкретные абстракции (типы платежей)**
```swift
// Кредитная карта
class CreditCardPayment: PaymentMethod {
    override func pay(amount: Double, completion: @escaping (Bool) -> Void) {
        print("Подготовка данных кредитной карты")
        processor.processPayment(amount: amount, completion: completion)
    }
}

// Криптовалюта
class CryptoPayment: PaymentMethod {
    override func pay(amount: Double, completion: @escaping (Bool) -> Void) {
        print("Конвертация в криптовалюту")
        processor.processPayment(amount: amount, completion: completion)
    }
}
```

---

## **2. Использование в коде**

### **Пример 1: Разные комбинации платежей**
```swift
// Stripe + Кредитная карта
let stripeCard = CreditCardPayment(processor: StripeProcessor())
stripeCard.pay(amount: 100) { success in
    print(success ? "Успешно!" : "Ошибка")
}

// PayPal + Криптовалюта
let paypalCrypto = CryptoPayment(processor: PayPalProcessor())
paypalCrypto.pay(amount: 0.5) { success in
    print(success ? "Успешно!" : "Ошибка")
}
```

**Вывод:**
```
Подготовка данных кредитной карты
Обработка платежа 100.0 через Stripe
Успешно!
Конвертация в криптовалюту
Обработка платежа 0.5 через PayPal
Успешно!
```

---

## **3. Дополнительные возможности**

### **Динамическое изменение реализации**
```swift
var payment = CreditCardPayment(processor: StripeProcessor())

// Позже можем изменить процессор
payment = CreditCardPayment(processor: PayPalProcessor())
```

---

## **Плюсы и минусы Bridge**

✅ **Плюсы:**
- Разделяет абстракцию и реализацию
- Позволяет изменять их независимо
- Соответствует Open-Closed Principle

❌ **Минусы:**
- Увеличивает сложность кода
- Может быть избыточным для простых случаев

---

## **Где применяется в iOS?**
1. **UI-фреймворки**:
   - Абстракция View + платформенная реализация (UIKit/SwiftUI)
2. **Базы данных**:
   - Общий интерфейс для CoreData/Realm/SQLite
3. **Сетевые слои**:
   - Единый API для разных транспортных протоколов

---

## **Сравнение с другими паттернами**
- **Adapter** - делает несовместимые интерфейсы совместимыми
- **Strategy** - меняет алгоритмы, а Bridge - всю реализацию
- **Abstract Factory** - создает семейства объектов
