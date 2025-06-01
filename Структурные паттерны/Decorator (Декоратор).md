**Паттерн Decorator** позволяет динамически добавлять объектам новую функциональность, оборачивая их в специальные классы-декораторы. В iOS это особенно полезно для:

- Расширения поведения UIView/UIViewController
- Добавления логирования/кеширования к сетевым запросам
- Реализации тем оформления

---

## **Паттерн Decorator в Swift: Реальный пример из iOS**

### **Задача 1: Декорирование сетевого запроса**
Добавим логирование и кеширование к базовому сетевому клиенту.

#### **1. Базовый протокол**
```swift
protocol NetworkService {
    func loadData(url: URL, completion: @escaping (Data?) -> Void)
}
```

#### **2. Конкретная реализация**
```swift
class BaseNetworkService: NetworkService {
    func loadData(url: URL, completion: @escaping (Data?) -> Void) {
        URLSession.shared.dataTask(with: url) { data, _, _ in
            completion(data)
        }.resume()
    }
}
```

#### **3. Декоратор логирования**
```swift
class LoggingNetworkDecorator: NetworkService {
    private let wrapped: NetworkService
    
    init(wrapped: NetworkService) {
        self.wrapped = wrapped
    }
    
    func loadData(url: URL, completion: @escaping (Data?) -> Void) {
        print("Начал загрузку: \(url.absoluteString)")
        let startTime = Date()
        
        wrapped.loadData(url: url) { data in
            print("Загрузка завершена за \(Date().timeIntervalSince(startTime)) сек")
            completion(data)
        }
    }
}
```

#### **4. Декоратор кеширования**
```swift
class CachingNetworkDecorator: NetworkService {
    private let wrapped: NetworkService
    private var cache = [URL: Data]()
    
    init(wrapped: NetworkService) {
        self.wrapped = wrapped
    }
    
    func loadData(url: URL, completion: @escaping (Data?) -> Void) {
        if let cached = cache[url] {
            print("Использую кеш для \(url.absoluteString)")
            completion(cached)
            return
        }
        
        wrapped.loadData(url: url) { [weak self] data in
            if let data = data {
                self?.cache[url] = data
            }
            completion(data)
        }
    }
}
```

#### **Использование:**
```swift
let service: NetworkService = CachingNetworkDecorator(
    wrapped: LoggingNetworkDecorator(
        wrapped: BaseNetworkService()
    )
)

let url = URL(string: "https://api.example.com/data")!
service.loadData(url: url) { data in
    // Обработка данных
}
```

---

### **Задача 2: Декорирование UIView**
Добавим тень и скругление к любой UIView.

#### **1. Базовый декоратор**
```swift
class ViewDecorator {
    let view: UIView
    
    init(view: UIView) {
        self.view = view
    }
    
    func decorate() -> UIView {
        return view
    }
}
```

#### **2. Конкретные декораторы**
```swift
class ShadowDecorator: ViewDecorator {
    override func decorate() -> UIView {
        view.layer.shadowColor = UIColor.black.cgColor
        view.layer.shadowOpacity = 0.5
        view.layer.shadowOffset = CGSize(width: 0, height: 2)
        view.layer.shadowRadius = 4
        return view
    }
}

class RoundCornersDecorator: ViewDecorator {
    let radius: CGFloat
    
    init(view: UIView, radius: CGFloat) {
        self.radius = radius
        super.init(view: view)
    }
    
    override func decorate() -> UIView {
        view.layer.cornerRadius = radius
        view.clipsToBounds = true
        return view
    }
}
```

#### **Использование:**
```swift
let button = UIButton(type: .system)
button.setTitle("Нажми меня", for: .normal)

let decoratedButton = ShadowDecorator(
    view: RoundCornersDecorator(
        view: button,
        radius: 8
    ).decorate()
).decorate()

// Добавляем на экран
view.addSubview(decoratedButton)
```

---

## **Ключевые преимущества Decorator в iOS**

✅ **Гибкость**:
- Можно комбинировать декораторы (кеширование + логирование)
- Не требует изменения исходного кода

✅ **Соответствует Open-Closed Principle**:
- Новая функциональность добавляется без модификации существующих классов

✅ **Упрощает тестирование**:
- Каждый декоратор можно тестировать изолированно

---

## **Где еще применять в iOS?**

1. **Темы оформления**:
   ```swift
   protocol Theme {
       func apply(to button: UIButton)
   }

   class DarkTheme: Theme { ... }
   class LightTheme: Theme { ... }

   // Декоратор для акцентной темы
   class AccentThemeDecorator: Theme {
       private let base: Theme
       init(base: Theme) { self.base = base }
       
       func apply(to button: UIButton) {
           base.apply(to: button)
           button.setTitleColor(.systemBlue, for: .normal)
       }
   }
   ```

2. **Аналитика**:
   ```swift
   // Декоратор для логирования событий аналитики
   class AnalyticsDecorator: ViewControllerDelegate {
       private let wrapped: ViewControllerDelegate
       init(wrapped: ViewControllerDelegate) { self.wrapped = wrapped }
       
       func viewDidAppear() {
           print("Отправляю событие в Firebase")
           wrapped.viewDidAppear()
       }
   }
   ```

---

## **Сравнение с другими паттернами**

🆚 **Decorator vs Adapter**:
- Adapter меняет интерфейс, Decorator расширяет функциональность

🆚 **Decorator vs Composite**:
- Composite работает с иерархиями, Decorator добавляет поведение

---

### **Что дальше?**
Хочешь разобрать:
1. **Декораторы для SwiftUI** (модификаторы как декораторы)?
2. **Реальный пример с CoreData** (кеширующий декоратор)?
3. **Разберем другой паттерн** (например, Facade)?
