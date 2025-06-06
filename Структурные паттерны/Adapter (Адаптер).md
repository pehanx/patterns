**Паттерн Адаптер (Adapter)** позволяет объектам с несовместимыми интерфейсами работать вместе. В iOS это особенно полезно при интеграции сторонних библиотек, работе с разными API или адаптации legacy-кода под новые требования.

---

## **Паттерн Adapter в Swift: Реальный пример из iOS**

### **Задача:**
Допустим, у нас есть:
1. **Старый сервис** для оплаты через `XML` (например, legacy-банковская система).
2. **Новый сервис** для оплаты через `JSON` (например, Stripe API).

Мы хотим использовать **единый интерфейс** для оплаты, чтобы не переписывать весь код.

---

## **1. Реализация Adapter**

### **Шаг 1: Определяем целевой интерфейс (новый стандарт)**
```swift
// Новый протокол для оплаты (JSON-API)
protocol PaymentService {
    func pay(amount: Double, completion: (Bool) -> Void)
}
```

### **Шаг 2: Старая система (XML-API)**
```swift
// Legacy-класс, который нельзя изменить
class LegacyPaymentSystem {
    func makeXMLPayment(amount: Double, callback: (Bool) -> Void) {
        print("Отправка платежа через XML: \(amount) руб.")
        callback(true)
    }
}
```

### **Шаг 3: Адаптер для старой системы**
```swift
// Адаптер преобразует вызовы JSON → XML
class LegacyPaymentAdapter: PaymentService {
    private let legacySystem = LegacyPaymentSystem()
    
    func pay(amount: Double, completion: (Bool) -> Void) {
        legacySystem.makeXMLPayment(amount: amount) { isSuccess in
            completion(isSuccess)
        }
    }
}
```

### **Шаг 4: Новая система (JSON-API)**
```swift
// Современный сервис (например, Stripe)
class ModernPaymentService: PaymentService {
    func pay(amount: Double, completion: (Bool) -> Void) {
        print("Отправка платежа через JSON: \(amount) руб.")
        completion(true)
    }
}
```

---

## **2. Использование в коде**

### **Пример 1: Работа через единый интерфейс**
```swift
// Клиентский код (не знает про XML/JSON)
func processPayment(service: PaymentService, amount: Double) {
    service.pay(amount: amount) { isSuccess in
        print(isSuccess ? "Успешно!" : "Ошибка!")
    }
}

// Оплата через новый сервис (JSON)
let stripe = ModernPaymentService()
processPayment(service: stripe, amount: 1000)

// Оплата через старый сервис (через адаптер)
let legacyAdapter = LegacyPaymentAdapter()
processPayment(service: legacyAdapter, amount: 500)
```

**Вывод:**
```
Отправка платежа через JSON: 1000.0 руб.
Успешно!
Отправка платежа через XML: 500.0 руб.
Успешно!
```

---

## **3. Другие примеры Adapter в iOS**

### **Пример 2: Адаптер для `UITableView` и `UICollectionView`**
Если у вас есть данные для `UICollectionView`, но нужно отобразить их в `UITableView`:

```swift
protocol ListDisplayable {
    var title: String { get }
    var subtitle: String { get }
}

// Адаптер преобразует модель для UITableView
class CollectionToTableAdapter: ListDisplayable {
    private let collectionItem: CollectionItem
    
    init(item: CollectionItem) {
        self.collectionItem = item
    }
    
    var title: String { return collectionItem.name }
    var subtitle: String { return "ID: \(collectionItem.id)" }
}
```

---

## **Плюсы и минусы Adapter**

✅ **Плюсы:**
- **Интеграция несовместимых систем** без изменения их кода.
- **Соблюдение SRP** — логика адаптации вынесена в отдельный класс.
- **Упрощение тестирования** — можно мокать адаптеры.

❌ **Минусы:**
- **Усложнение кода** — появляются дополнительные классы.
- **Накладные расходы** — если адаптер делает сложные преобразования.

---

## **Где применяется в iOS?**
1. **Работа с разными API** (REST, SOAP, GraphQL).
2. **Адаптация данных** (например, `CoreData` → `Codable`).
3. **Сторонние SDK** (например, если Facebook SDK требует особого формата данных).

---

## **Сравнение с другими паттернами**
- **Facade** — упрощает интерфейс, но не преобразует данные.
- **Decorator** — добавляет поведение, но не меняет интерфейс.
