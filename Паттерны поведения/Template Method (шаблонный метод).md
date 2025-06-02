**Паттерн Template Method (Шаблонный метод)** определяет скелет алгоритма, перекладывая ответственность за некоторые его шаги на подклассы. Это позволяет подклассам переопределять отдельные шаги алгоритма, не меняя его структуры. В iOS он часто используется для:

- Стандартизации процессов (жизненный цикл ViewController)
- Реализации переиспользуемых алгоритмов
- Создания расширяемых компонентов

---

## **Паттерн Template Method в Swift: Реальные примеры из iOS**

### **Пример 1: Жизненный цикл сетевого запроса**

#### **Задача:**
Создать базовый алгоритм выполнения запроса с возможностью кастомизации:
1. Подготовка запроса
2. Выполнение запроса
3. Обработка ошибок
4. Парсинг ответа

---

#### **1. Реализация паттерна**

```swift
// Абстрактный класс с шаблонным методом
abstract class NetworkOperation<T> {
    
    // Шаблонный метод (не может быть переопределен)
    final func execute() -> Result<T, Error> {
        let request = prepareRequest()
        let data = performRequest(request)
        return handleResult(data)
    }
    
    // Шаги, которые должны быть реализованы подклассами
    func prepareRequest() -> URLRequest {
        fatalError("Must be overridden")
    }
    
    func parseResponse(_ data: Data) -> T {
        fatalError("Must be overridden")
    }
    
    // Методы с реализацией по умолчанию
    func performRequest(_ request: URLRequest) -> Data {
        // Здесь реальная логика выполнения запроса
        print("Выполняем запрос: \(request.url?.absoluteString ?? "")")
        return Data() // Заглушка
    }
    
    func handleError(_ error: Error) -> Result<T, Error> {
        print("Ошибка: \(error.localizedDescription)")
        return .failure(error)
    }
    
    // Финальный метод обработки
    private func handleResult(_ data: Data) -> Result<T, Error> {
        do {
            let result = try parseResponse(data)
            return .success(result)
        } catch {
            return handleError(error)
        }
    }
}
```

#### **2. Конкретная реализация**

```swift
class UserFetchOperation: NetworkOperation<[User]> {
    private let endpoint = URL(string: "https://api.example.com/users")!
    
    override func prepareRequest() -> URLRequest {
        var request = URLRequest(url: endpoint)
        request.httpMethod = "GET"
        return request
    }
    
    override func parseResponse(_ data: Data) -> [User] {
        let decoder = JSONDecoder()
        return try decoder.decode([User].self, from: data)
    }
    
    override func handleError(_ error: Error) -> Result<[User], Error> {
        print("Специфичная обработка ошибок для UserFetchOperation")
        return .failure(error)
    }
}
```

#### **3. Использование**

```swift
let operation = UserFetchOperation()
let result = operation.execute()

switch result {
case .success(let users):
    print("Получено пользователей: \(users.count)")
case .failure(let error):
    print("Ошибка получения пользователей: \(error)")
}
```

---

### **Пример 2: Базовый UIViewController с шаблонными методами**

#### **Задача:**
Создать базовый ViewController с общим жизненным циклом:

```swift
class BaseViewController: UIViewController {
    
    // Шаблонный метод
    final override func viewDidLoad() {
        super.viewDidLoad()
        setupViews()
        setupConstraints()
        configureAppearance()
        bindViewModel()
    }
    
    // Методы для переопределения
    func setupViews() {
        // Базовая реализация (может быть пустой)
    }
    
    func setupConstraints() {
        // Базовая реализация
    }
    
    func configureAppearance() {
        // Базовая реализация
    }
    
    func bindViewModel() {
        fatalError("Must be overridden")
    }
}

// Конкретная реализация
class ProfileViewController: BaseViewController {
    override func setupViews() {
        view.addSubview(avatarImageView)
        view.addSubview(nameLabel)
    }
    
    override func bindViewModel() {
        viewModel.$user
            .receive(on: DispatchQueue.main)
            .sink { [weak self] user in
                self?.nameLabel.text = user.name
            }
            .store(in: &cancellables)
    }
}
```

---

## **Где применяется в iOS?**

1. **UIKit:** 
   - `UIView.drawHierarchy(in:afterScreenUpdates:)` вызывает `draw(_:)`
   - `UICollectionViewLayout` методы подготовки атрибутов

2. **SwiftUI:**
   - `View.body` - шаблонный метод, который вы реализуете

3. **Foundation:**
   - `JSONEncoder/JSONDecoder` с кастомными стратегиями кодирования

---

## **Плюсы и минусы**

✅ **Плюсы:**
- Исключает дублирование кода
- Контролирует точки расширения
- Сохраняет структуру алгоритма

❌ **Минусы:**
- Ограничивает гибкость через наследование
- Может привести к сложным иерархиям классов

---

## **Сравнение с другими паттернами**

- **Strategy:** Использует композицию, **Template Method** - наследование
- **Factory Method:** Часто реализуется через Template Method
- **Decorator:** Добавляет поведение, **Template Method** фиксирует структуру
