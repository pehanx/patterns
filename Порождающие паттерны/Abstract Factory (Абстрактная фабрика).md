## Реализация Абстрактной фабрики для API-клиента
### 1. Абстрактные продукты (протоколы)
Определим, какие объекты будет создавать фабрика:

```swift
// Абстрактный сетевой клиент
protocol NetworkService {
    func fetchUsers(completion: @escaping ([User]) -> Void)
}

// Абстрактное хранилище (база данных/кеш)
protocol StorageService {
    func saveUsers(_ users: [User])
    func loadUsers() -> [User]
}
```

### 2. Конкретные продукты для Production
Реальные реализации для боевого режима:

```swift
// Реальный сетевой клиент (Alamofire/URLSession)
class ProductionNetworkService: NetworkService {
    func fetchUsers(completion: @escaping ([User]) -> Void) {
        print("Загрузка пользователей с реального сервера...")
        // Здесь мог бы быть URLSession или Alamofire запрос
        let users = [User(id: 1, name: "Иван"), User(id: 2, name: "Мария")]
        completion(users)
    }
}

// Реальное хранилище (CoreData/Realm)
class ProductionStorageService: StorageService {
    func saveUsers(_ users: [User]) {
        print("Сохранение пользователей в CoreData...")
    }
    
    func loadUsers() -> [User] {
        print("Загрузка пользователей из CoreData...")
        return [User(id: 1, name: "Иван (из БД)")]
    }
}
```

### 3. Конкретные продукты для Mock (тестов)
Заглушки для тестирования/разработки:

```swift
// Mock-сетевой клиент
class MockNetworkService: NetworkService {
    func fetchUsers(completion: @escaping ([User]) -> Void) {
        print("(MOCK) Загрузка пользователей без сети...")
        let users = [User(id: 99, name: "Тестовый пользователь")]
        completion(users)
    }
}

// Mock-хранилище (UserDefaults/In-Memory)
class MockStorageService: StorageService {
    func saveUsers(_ users: [User]) {
        print("(MOCK) Сохранение пользователей в память...")
    }
    
    func loadUsers() -> [User] {
        print("(MOCK) Загрузка тестовых пользователей...")
        return [User(id: 99, name: "Тестовый пользователь (из памяти)")]
    }
}
```

### 4. Абстрактная фабрика
Протокол, который создаёт связанные сервисы:

```swift
protocol ServiceFactory {
    func makeNetworkService() -> NetworkService
    func makeStorageService() -> StorageService
}
```

### 5. Конкретные фабрики
Реализации для Production и Mock:

```swift
// Фабрика для боевого режима
class ProductionFactory: ServiceFactory {
    func makeNetworkService() -> NetworkService {
        return ProductionNetworkService()
    }
    
    func makeStorageService() -> StorageService {
        return ProductionStorageService()
    }
}

// Фабрика для тестов
class MockFactory: ServiceFactory {
    func makeNetworkService() -> NetworkService {
        return MockNetworkService()
    }
    
    func makeStorageService() -> StorageService {
        return MockStorageService()
    }
}
```

### 6. Использование в коде

```swift
// Клиентский код (например, ViewModel)
class UserViewModel {
    private let networkService: NetworkService
    private let storageService: StorageService
    
    init(factory: ServiceFactory) {
        self.networkService = factory.makeNetworkService()
        self.storageService = factory.makeStorageService()
    }
    
    func loadData() {
        networkService.fetchUsers { [weak self] users in
            self?.storageService.saveUsers(users)
            let savedUsers = self?.storageService.loadUsers()
            print("Загружены пользователи: \(savedUsers ?? [])")
        }
    }
}

// Пример вызова
let isTesting = true // Может зависеть от конфига, схемы сборки и т.д.

let factory: ServiceFactory = isTesting ? MockFactory() : ProductionFactory()
let viewModel = UserViewModel(factory: factory)
viewModel.loadData()
```
