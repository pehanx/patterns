**Паттерн Strategy (Стратегия)** позволяет определять семейство алгоритмов, инкапсулировать каждый из них и делать их взаимозаменяемыми. В iOS он особенно полезен для:

- Реализации различных вариантов сортировки/фильтрации
- Выбора способа оплаты
- Адаптации поведения UI под разные платформы (iPhone/iPad)

---

## **Паттерн Strategy в Swift: Реальные примеры из iOS**

### **Пример 1: Система оплаты с разными провайдерами**

#### **Задача:**
Реализовать выбор между:
- Apple Pay
- Credit Card
- PayPal

---

#### **1. Реализация паттерна**

```swift
// Протокол стратегии
protocol PaymentStrategy {
    func processPayment(amount: Double)
}

// Конкретные стратегии
class ApplePayStrategy: PaymentStrategy {
    func processPayment(amount: Double) {
        print("Оплата через Apple Pay: \(amount) руб.")
        // Вызов PKPaymentAuthorizationController
    }
}

class CreditCardStrategy: PaymentStrategy {
    private let cardNumber: String
    
    init(cardNumber: String) {
        self.cardNumber = cardNumber
    }
    
    func processPayment(amount: Double) {
        print("Оплата картой \(cardNumber.suffix(4)): \(amount) руб.")
        // Логика обработки карты
    }
}

class PayPalStrategy: PaymentStrategy {
    private let email: String
    
    init(email: String) {
        self.email = email
    }
    
    func processPayment(amount: Double) {
        print("Оплата через PayPal (\(email)): \(amount) руб.")
        // Интеграция с PayPal SDK
    }
}

// Контекст
class PaymentService {
    private var strategy: PaymentStrategy
    
    init(strategy: PaymentStrategy) {
        self.strategy = strategy
    }
    
    func changeStrategy(_ strategy: PaymentStrategy) {
        self.strategy = strategy
    }
    
    func makePayment(amount: Double) {
        strategy.processPayment(amount: amount)
    }
}
```

#### **2. Использование**

```swift
let paymentService = PaymentService(strategy: ApplePayStrategy())
paymentService.makePayment(amount: 1000) // Оплата через Apple Pay: 1000.0 руб.

paymentService.changeStrategy(CreditCardStrategy(cardNumber: "1234567812345678"))
paymentService.makePayment(amount: 1500) // Оплата картой 5678: 1500.0 руб.

paymentService.changeStrategy(PayPalStrategy(email: "user@example.com"))
paymentService.makePayment(amount: 2000) // Оплата через PayPal (user@example.com): 2000.0 руб.
```

---

### **Пример 2: Разные алгоритмы сортировки в UITableView**

#### **Задача:**
Реализовать сортировку:
- По дате
- По имени
- По рейтингу

---

#### **1. Реализация**

```swift
protocol SortStrategy {
    func sort(_ items: [Product]) -> [Product]
}

class DateSortStrategy: SortStrategy {
    func sort(_ items: [Product]) -> [Product] {
        return items.sorted { $0.date > $1.date }
    }
}

class NameSortStrategy: SortStrategy {
    func sort(_ items: [Product]) -> [Product] {
        return items.sorted { $0.name < $1.name }
    }
}

class RatingSortStrategy: SortStrategy {
    func sort(_ items: [Product]) -> [Product] {
        return items.sorted { $0.rating > $1.rating }
    }
}

class ProductListViewController: UIViewController {
    private var sortStrategy: SortStrategy = DateSortStrategy()
    private var products: [Product] = []
    
    func changeSortStrategy(_ strategy: SortStrategy) {
        self.sortStrategy = strategy
        reloadData()
    }
    
    private func reloadData() {
        let sortedProducts = sortStrategy.sort(products)
        // Обновляем UITableView
    }
}
```

#### **2. Использование**

```swift
let vc = ProductListViewController()

// Пользователь выбрал сортировку по имени
vc.changeSortStrategy(NameSortStrategy())

// Пользователь выбрал сортировку по рейтингу
vc.changeSortStrategy(RatingSortStrategy())
```

---

## **Где применяется в iOS?**

1. **Вычисление цены с разными системами скидок**
2. **Аналитика (разные сервисы: Firebase, Amplitude)**
3. **Кеширование (разные стратегии: RAM, Disk, Hybrid)**
4. **Валидация форм (разные правила для разных полей)**

---

## **Плюсы и минусы**

✅ **Плюсы:**
- Замена наследования на композицию
- Возможность менять алгоритмы на лету
- Изоляция кода алгоритмов

❌ **Минусы:**
- Увеличивает количество классов
- Клиент должен знать о разных стратегиях

---

## **Сравнение с другими паттернами**

- **State** - меняет поведение при изменении состояния, **Strategy** - явный выбор алгоритма
- **Factory Method** - создает объекты, **Strategy** - инкапсулирует алгоритмы
- **Command** - инкапсулирует запросы, **Strategy** - инкапсулирует алгоритмы
