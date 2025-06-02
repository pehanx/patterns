**Паттерн Chain of Responsibility (Цепочка обязанностей)** позволяет передавать запросы последовательно через цепочку обработчиков, пока один из них не обработает запрос. В iOS он часто применяется для:

- Обработки событий (например, тапы в сложных view-иерархиях)
- Валидации данных (цепочка проверок)
- Middleware в сетевых запросах

---

## **Chain of Responsibility в Swift: Реальный пример из iOS**

### **Пример 1: Валидация формы регистрации**

#### **Задача:**
Проверить данные пользователя при регистрации:
1. Логин не пустой
2. Пароль содержит >= 8 символов
3. Email валидный

Каждая проверка должна быть отдельным обработчиком.

---

#### **1. Реализация паттерна**

```swift
// Протокол обработчика
protocol Handler: AnyObject {
    var nextHandler: Handler? { get set }
    func handle(_ request: String) -> Bool
}

// Базовый класс обработчика
class BaseHandler: Handler {
    var nextHandler: Handler?
    
    func handle(_ request: String) -> Bool {
        // Передаем запрос следующему обработчику
        return nextHandler?.handle(request) ?? true
    }
}

// Конкретные обработчики
class EmptyCheckHandler: BaseHandler {
    override func handle(_ request: String) -> Bool {
        guard !request.isEmpty else {
            print("Ошибка: Поле не может быть пустым")
            return false
        }
        return super.handle(request)
    }
}

class LengthCheckHandler: BaseHandler {
    private let minLength: Int
    
    init(minLength: Int) {
        self.minLength = minLength
    }
    
    override func handle(_ request: String) -> Bool {
        guard request.count >= minLength else {
            print("Ошибка: Минимальная длина \(minLength) символов")
            return false
        }
        return super.handle(request)
    }
}

class EmailValidationHandler: BaseHandler {
    override func handle(_ request: String) -> Bool {
        let emailRegex = "[A-Z0-9a-z._%+-]+@[A-Za-z0-9.-]+\\.[A-Za-z]{2,64}"
        let predicate = NSPredicate(format: "SELF MATCHES %@", emailRegex)
        
        guard predicate.evaluate(with: request) else {
            print("Ошибка: Некорректный email")
            return false
        }
        return super.handle(request)
    }
}
```

#### **2. Использование**

```swift
// Создаем цепочку
let emptyCheck = EmptyCheckHandler()
let lengthCheck = LengthCheckHandler(minLength: 8)
let emailCheck = EmailValidationHandler()

emptyCheck.nextHandler = lengthCheck
lengthCheck.nextHandler = emailCheck

// Тестируем
func validate(_ input: String) {
    if emptyCheck.handle(input) {
        print("✅ Валидация успешна для: '\(input)'")
    }
}

validate("")          // Ошибка: Поле не может быть пустым
validate("short")     // Ошибка: Минимальная длина 8 символов
validate("test@")     // Ошибка: Некорректный email
validate("good.email@example.com") // ✅ Валидация успешна
```

---

### **Пример 2: Обработка touch-событий в UIKit**

#### **Задача:**
Обрабатывать тапы в иерархии view:
1. Сначала обрабатывает кнопка
2. Потом контейнер
3. В конце родительский view

```swift
class TouchHandler {
    var nextHandler: TouchHandler?
    
    func handleTouch() -> Bool {
        return nextHandler?.handleTouch() ?? false
    }
}

class ButtonHandler: TouchHandler {
    override func handleTouch() -> Bool {
        if /* кнопка активна */ {
            print("Кнопка обработала тап")
            return true
        }
        return super.handleTouch()
    }
}

class ContainerHandler: TouchHandler {
    override func handleTouch() -> Bool {
        if /* тап в области контейнера */ {
            print("Контейнер обработал тап")
            return true
        }
        return super.handleTouch()
    }
}

// Использование
let button = ButtonHandler()
let container = ContainerHandler()
button.nextHandler = container

button.handleTouch() // Вызовет обработку по цепочке
```

---

## **Где применяется в iOS?**

1. **Обработка событий:**
   - `UIResponder`-цепочка (по сути реализует этот паттерн)
   - Жесты в сложных view-иерархиях

2. **Сетевые запросы:**
   ```swift
   // Цепочка middleware для обработки запроса:
   // Auth → Logging → Caching → API
   ```

3. **Валидация данных:**
   - Проверка форм регистрации
   - Фильтрация введенных данных

---

## **Плюсы и минусы**

✅ **Плюсы:**
- Уменьшает зависимость между отправителем и получателем
- Позволяет динамически менять цепочку
- Реализует принцип единственной ответственности

❌ **Минусы:**
- Нет гарантии обработки запроса
- Может усложнить отладку

---

## **Сравнение с другими паттернами**

- **Decorator** — добавляет функциональность, а **Chain** передает запрос
- **Command** — инкапсулирует запрос, а **Chain** его обрабатывает
- **Composite** — работает с древовидными структурами
