**Паттерн Proxy (Заместитель)** — это структурный паттерн, который подставляет вместо реальных объектов специальные объекты-заменители. Эти объекты перехватывают вызовы к оригинальному объекту, позволяя выполнить дополнительные действия до или после обращения к нему.  

В iOS Proxy особенно полезен для:  
- **Ленивой загрузки** (lazy loading)  
- **Кеширования** результатов операций  
- **Контроля доступа** к敏感ным объектам  
- **Логирования** вызовов методов  

---

## **Паттерн Proxy в Swift: Реальные примеры из iOS**  

### **Пример 1: Ленивая загрузка изображения (Lazy Loading Proxy)**  

#### **Задача:**  
Загружать тяжелое изображение только тогда, когда оно действительно нужно (например, при отображении на экране).  

#### **Реализация:**  
```swift
// 1. Общий протокол для изображения
protocol Image {
    func display()
}

// 2. Реальный объект (тяжелое изображение)
class RealImage: Image {
    private let filename: String
    
    init(filename: String) {
        self.filename = filename
        loadFromDisk()
    }
    
    private func loadFromDisk() {
        print("Загрузка изображения \(filename) с диска...")
    }
    
    func display() {
        print("Отображение изображения \(filename)")
    }
}

// 3. Прокси (ленивая загрузка)
class ImageProxy: Image {
    private let filename: String
    private var realImage: RealImage?
    
    init(filename: String) {
        self.filename = filename
    }
    
    func display() {
        // Загружаем изображение только при первом вызове
        if realImage == nil {
            realImage = RealImage(filename: filename)
        }
        realImage?.display()
    }
}
```

#### **Использование:**  
```swift
let image = ImageProxy(filename: "photo.jpg")

// Изображение не загружается сразу
print("Изображение создано, но еще не загружено")

// Загрузка происходит только здесь
image.display()

// Вывод:
// Изображение создано, но еще не загружено
// Загрузка изображения photo.jpg с диска...
// Отображение изображения photo.jpg
```

---

### **Пример 2: Защитный Proxy (Access Control)**  

#### **Задача:**  
Ограничить доступ к сервису оплаты, если пользователь не авторизован.  

#### **Реализация:**  
```swift
// 1. Протокол платежного сервиса
protocol PaymentService {
    func processPayment(amount: Double)
}

// 2. Реальный сервис
class RealPaymentService: PaymentService {
    func processPayment(amount: Double) {
        print("Обработка платежа на сумму \(amount) руб.")
    }
}

// 3. Защитный прокси
class PaymentProxy: PaymentService {
    private let realService = RealPaymentService()
    private let isAuthenticated: Bool
    
    init(isAuthenticated: Bool) {
        self.isAuthenticated = isAuthenticated
    }
    
    func processPayment(amount: Double) {
        guard isAuthenticated else {
            print("Ошибка: Пользователь не авторизован!")
            return
        }
        realService.processPayment(amount: amount)
    }
}
```

#### **Использование:**  
```swift
let authUser = PaymentProxy(isAuthenticated: true)
authUser.processPayment(amount: 1000) // Успешно

let guestUser = PaymentProxy(isAuthenticated: false)
guestUser.processPayment(amount: 1000) // Ошибка

// Вывод:
// Обработка платежа на сумму 1000.0 руб.
// Ошибка: Пользователь не авторизован!
```

---

### **Пример 3: Кеширующий Proxy (Caching)**  

#### **Задача:**  
Кешировать результаты сетевых запросов, чтобы избежать повторных загрузок.  

#### **Реализация:**  
```swift
// 1. Протокол сервиса данных
protocol DataService {
    func fetchData() -> String
}

// 2. Реальный сервис (например, сетевой запрос)
class RealDataService: DataService {
    func fetchData() -> String {
        print("Загрузка данных из сети...")
        return "Данные"
    }
}

// 3. Прокси с кешированием
class CachingProxy: DataService {
    private let realService = RealDataService()
    private var cachedData: String?
    
    func fetchData() -> String {
        if let cachedData = cachedData {
            print("Данные взяты из кеша")
            return cachedData
        }
        let data = realService.fetchData()
        cachedData = data
        return data
    }
}
```

#### **Использование:**  
```swift
let service = CachingProxy()

// Первый вызов — загрузка из сети
print(service.fetchData())

// Второй вызов — данные из кеша
print(service.fetchData())

// Вывод:
// Загрузка данных из сети...
// Данные
// Данные взяты из кеша
// Данные
```

---

## **Где применяется Proxy в iOS?**  
1. **`NSProxy`** в Objective-C — для перехвата сообщений.  
2. **Ленивая инициализация** (`lazy var` в Swift — это тоже вид Proxy).  
3. **Кеширование** (например, `NSCache` или `URLCache`).  
4. **Mock-объекты** в тестах.  

---

## **Плюсы и минусы Proxy**  

✅ **Плюсы:**  
- Контроль над жизненным циклом реального объекта.  
- Дополнительная логика без изменения основного класса.  
- Экономия ресурсов (кеширование, ленивая загрузка).  

❌ **Минусы:**  
- Усложнение кода (появляются дополнительные классы).  
- Возможные накладные расходы на проверки.  

---

## **Сравнение с другими паттернами**  
- **Decorator** — добавляет новую функциональность, а Proxy контролирует доступ.  
- **Adapter** — меняет интерфейс объекта, а Proxy сохраняет его.  
