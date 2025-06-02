**Паттерн Observer (Наблюдатель)** позволяет объектам подписываться на изменения состояния других объектов и автоматически получать уведомления. В iOS он реализован через несколько механизмов:

### 1. Основные реализации Observer в iOS:
1. **NotificationCenter** (децентрализованные уведомления)
2. **KVO (Key-Value Observing)** (наблюдение за свойствами)
3. **Combine Framework** (современный реактивный подход)
4. **Собственные реализации через протоколы**

---

## **Реальный пример 1: Собственный Observer для модели пользователя**

### Задача:
Уведомлять экраны при изменении данных пользователя.

#### 1. Определяем протоколы:
```swift
protocol UserObserver: AnyObject {
    func userDidUpdate(_ user: User)
}

class User {
    private var observers = [UserObserver]()
    
    var name: String {
        didSet { notifyObservers() }
    }
    
    var age: Int {
        didSet { notifyObservers() }
    }
    
    init(name: String, age: Int) {
        self.name = name
        self.age = age
    }
    
    func addObserver(_ observer: UserObserver) {
        observers.append(observer)
    }
    
    func removeObserver(_ observer: UserObserver) {
        observers.removeAll { $0 === observer }
    }
    
    private func notifyObservers() {
        observers.forEach { $0.userDidUpdate(self) }
    }
}
```

#### 2. Использование:
```swift
class ProfileViewController: UIViewController, UserObserver {
    private let user: User
    
    init(user: User) {
        self.user = user
        super.init(nibName: nil, bundle: nil)
        user.addObserver(self)
    }
    
    func userDidUpdate(_ user: User) {
        print("Обновление интерфейса: \(user.name), \(user.age)")
        // updateUI()
    }
    
    deinit {
        user.removeObserver(self)
    }
}
```

---

## **Реальный пример 2: Combine для наблюдения за данными**

### Задача:
Реактивно обновлять UI при изменении данных.

#### 1. Модель с Combine:
```swift
import Combine

class DataModel {
    @Published var items: [String] = []
    @Published var isLoading = false
}
```

#### 2. Подписка во ViewController:
```swift
class ItemsViewController: UIViewController {
    private let model = DataModel()
    private var cancellables = Set<AnyCancellable>()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        model.$items
            .receive(on: DispatchQueue.main)
            .sink { [weak self] items in
                self?.updateList(with: items)
            }
            .store(in: &cancellables)
        
        model.$isLoading
            .sink { [weak self] isLoading in
                isLoading ? self?.showLoader() : self?.hideLoader()
            }
            .store(in: &cancellables)
    }
}
```

---

## **Плюсы и минусы**

✅ **Плюсы:**
- Гибкая связь между объектами
- Автоматическое обновление
- Поддержка множества подписчиков

❌ **Минусы:**
- Может привести к утечкам памяти
- Сложнее отлаживать
- Неочевидный поток данных

---

## **Сравнение подходов в iOS**

| Способ          | Плюсы                          | Минусы                       |
|-----------------|--------------------------------|------------------------------|
| NotificationCenter | Простота, глобальность        | Нетипизированность, строкаи |
| KVO             | Нативное решение               | Сложный API, ручное управление |
| Combine         | Мощный, типобезопасный        | Только iOS 13+               |
| Делегаты        | Четкие контракты               | 1:1 связь                    |

---

## **Оптимальное использование**
1. **Для простых уведомлений** → NotificationCenter
2. **Для реактивного UI** → Combine
3. **Для наблюдения за свойствами** → KVO
4. **Для четких контрактов** → Собственные протоколы
