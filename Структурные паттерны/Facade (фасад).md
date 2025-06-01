**Паттерн Facade (Фасад)** предоставляет простой интерфейс для работы со сложной подсистемой, скрывая её детали реализации. В iOS это особенно полезно для:

- Упрощения работы с многослойными системами
- Инкапсуляции сложных зависимостей
- Предоставления удобного API для часто используемых операций

---

## **Паттерн Facade в Swift: Реальный пример из iOS**

### **Задача:**
Создадим унифицированный API для работы с:
1. Сетевыми запросами
2. Локальным кешированием
3. Парсингом JSON
4. Обновлением UI

Вместо того чтобы вызывать все эти компоненты по отдельности, мы создадим **Facade** - `DataManager`.

---

## **1. Реализация Facade**

### **Шаг 1: Создаем сложную подсистему**

```swift
// 1. Сетевой слой
class NetworkService {
    func fetchData(from url: URL, completion: @escaping (Data?) -> Void) {
        print("Загрузка данных из сети...")
        URLSession.shared.dataTask(with: url) { data, _, _ in
            completion(data)
        }.resume()
    }
}

// 2. Кеширование
class CacheManager {
    private var cache = [URL: Data]()
    
    func cachedData(for url: URL) -> Data? {
        print("Проверка кеша...")
        return cache[url]
    }
    
    func save(_ data: Data, for url: URL) {
        print("Сохранение в кеш...")
        cache[url] = data
    }
}

// 3. Парсинг JSON
class Parser {
    func parse<T: Decodable>(_ data: Data) throws -> T {
        print("Парсинг JSON...")
        return try JSONDecoder().decode(T.self, from: data)
    }
}

// 4. Обновление UI (для примера)
class UIManager {
    func updateUI(onMainThread work: @escaping () -> Void) {
        DispatchQueue.main.async {
            print("Обновление UI...")
            work()
        }
    }
}
```

### **Шаг 2: Создаем Facade**

```swift
class DataManager {
    private let network = NetworkService()
    private let cache = CacheManager()
    private let parser = Parser()
    private let uiManager = UIManager()
    
    func load<T: Decodable>(_ type: T.Type, from url: URL, completion: @escaping (T?) -> Void) {
        // 1. Проверяем кеш
        if let cachedData = cache.cachedData(for: url) {
            uiManager.updateUI {
                let result = try? parser.parse(cachedData) as T
                completion(result)
            }
            return
        }
        
        // 2. Загружаем из сети
        network.fetchData(from: url) { [weak self] data in
            guard let self = self, let data = data else {
                completion(nil)
                return
            }
            
            // 3. Сохраняем в кеш
            self.cache.save(data, for: url)
            
            // 4. Парсим и возвращаем результат
            self.uiManager.updateUI {
                let result = try? self.parser.parse(data) as T
                completion(result)
            }
        }
    }
}
```

---

## **2. Использование Facade**

### **Пример 1: Загрузка данных**

```swift
struct User: Decodable {
    let id: Int
    let name: String
}

let dataManager = DataManager()
let url = URL(string: "https://api.example.com/user/1")!

dataManager.load(User.self, from: url) { user in
    guard let user = user else { return }
    print("Получен пользователь: \(user.name)")
}
```

**Вывод:**
```
Проверка кеша...
Загрузка данных из сети...
Сохранение в кеш...
Парсинг JSON...
Обновление UI...
Получен пользователь: Иван Иванов
```

### **Пример 2: Повторная загрузка (из кеша)**

```swift
dataManager.load(User.self, from: url) { user in
    // ...
}
```

**Вывод:**
```
Проверка кеша...
Парсинг JSON...
Обновление UI...
Получен пользователь: Иван Иванов
```

---

## **3. Дополнительные возможности Facade**

### **Упрощение сложных операций**

```swift
extension DataManager {
    func loadUserProfile(userId: Int, completion: @escaping (UserProfile?) -> Void) {
        let userUrl = URL(string: "https://api.example.com/user/\(userId)")!
        let friendsUrl = URL(string: "https://api.example.com/user/\(userId)/friends")!
        
        var user: User?
        var friends: [Friend]?
        
        let group = DispatchGroup()
        
        group.enter()
        load(User.self, from: userUrl) { result in
            user = result
            group.leave()
        }
        
        group.enter()
        load([Friend].self, from: friendsUrl) { result in
            friends = result
            group.leave()
        }
        
        group.notify(queue: .main) {
            guard let user = user, let friends = friends else {
                completion(nil)
                return
            }
            completion(UserProfile(user: user, friends: friends))
        }
    }
}
```

---

## **Плюсы и минусы Facade в iOS**

✅ **Плюсы:**
- Упрощает работу со сложными системами
- Уменьшает связанность между компонентами
- Делает код более читаемым

❌ **Минусы:**
- Может стать "божественным объектом" (God Object), если не контролировать размер
- Дополнительный слой абстракции

---

## **Где применять в iOS?**

1. **Работа с CoreData/Realm**:
   ```swift
   class StorageFacade {
       private let coreDataStack: CoreDataStack
       private let realmInstance: Realm
       
       func saveUser(_ user: User) { ... }
       func fetchUsers() -> [User] { ... }
   }
   ```

2. **Комбинация сервисов аналитики**:
   ```swift
   class AnalyticsFacade {
       private let firebaseAnalytics
       private let amplitudeAnalytics
       
       func trackEvent(_ event: AnalyticsEvent) {
           firebaseAnalytics.track(event)
           amplitudeAnalytics.track(event)
       }
   }
   ```

3. **Управление зависимостями**:
   ```swift
   class Dependencies {
       static let shared = Dependencies()
       
       let network: NetworkFacade
       let storage: StorageFacade
       let analytics: AnalyticsFacade
       
       private init() {
           self.network = NetworkFacade()
           self.storage = StorageFacade()
           self.analytics = AnalyticsFacade()
       }
   }
   ```

---

## **Сравнение с другими паттернами**

🆚 **Facade vs Decorator**:
- Facade упрощает интерфейс, Decorator добавляет функциональность

🆚 **Facade vs Mediator**:
- Facade предоставляет простой доступ, Mediator управляет взаимодействием компонентов
