**Паттерн Одиночка (Singleton)** гарантирует, что у класса есть только **один экземпляр**, и предоставляет глобальную точку доступа к нему. В iOS он часто используется для сервисов, которые должны быть доступны везде (например, `URLSession.shared`, `UserDefaults.standard`).  

Но будь осторожен: Singleton — мощный инструмент, но его **неправильное применение** приводит к проблемам (глобальное состояние, сложность тестирования).  

---

## **Singleton в Swift: Реальный пример из iOS**  
### **Задача:**  
Создать **менеджер аутентификации**, который:  
- Хранит токен пользователя.  
- Управляет логином/логаутом.  
- Доступен из любого места приложения.  

### **Ключевые принципы:**  
1. **Закрытый `init`** — чтобы нельзя было создать экземпляр извне.  
2. **Статическое свойство `shared`** — глобальная точка доступа.  
3. **Потокобезопасность** — если Singleton используется в многопоточной среде.  

---

## **1. Базовая реализация Singleton**  
```swift
final class AuthManager {
    // Статический экземпляр
    static let shared = AuthManager()
    
    // Закрытый конструктор
    private init() {}
    
    // Данные пользователя
    private var authToken: String?
    
    // Методы
    func login(token: String) {
        authToken = token
        print("Пользователь авторизован. Токен сохранён.")
    }
    
    func logout() {
        authToken = nil
        print("Пользователь вышел.")
    }
    
    func isLoggedIn() -> Bool {
        return authToken != nil
    }
}
```

### **Использование:**  
```swift
// Где-то в LoginViewController
AuthManager.shared.login(token: "abc123")

// Где-то в ProfileViewController
if AuthManager.shared.isLoggedIn() {
    print("Показываем профиль")
} else {
    print("Показываем экран логина")
}
```

---

## **2. Улучшенная версия (потокобезопасность)**  
Если Singleton используется в многопоточной среде (например, сетевые запросы), добавляем **`DispatchQueue`** для синхронизации:  
```swift
final class ThreadSafeAuthManager {
    static let shared = ThreadSafeAuthManager()
    private init() {}
    
    private var authToken: String?
    private let queue = DispatchQueue(label: "auth.sync.queue", attributes: .concurrent)
    
    func login(token: String) {
        queue.async(flags: .barrier) {
            self.authToken = token
        }
    }
    
    func getToken() -> String? {
        queue.sync {
            return authToken
        }
    }
}
```

---

## **3. Singleton + Dependency Injection (для тестирования)**  
Проблема: Singleton трудно тестировать из-за глобального состояния.  
Решение: Используем **протокол** и внедряем зависимость.  

```swift
protocol AuthServiceProtocol {
    func login(token: String)
    func isLoggedIn() -> Bool
}

final class AuthManager: AuthServiceProtocol {
    static let shared = AuthManager()
    private init() {}
    private var authToken: String?
    
    func login(token: String) {
        authToken = token
    }
    
    func isLoggedIn() -> Bool {
        return authToken != nil
    }
}

// В тестах подменяем на Mock
final class MockAuthService: AuthServiceProtocol {
    func login(token: String) {}
    func isLoggedIn() -> Bool { return true }
}

// Пример внедрения в ViewModel
class ProfileViewModel {
    private let authService: AuthServiceProtocol
    
    init(authService: AuthServiceProtocol = AuthManager.shared) {
        self.authService = authService
    }
    
    func checkAuth() -> Bool {
        return authService.isLoggedIn()
    }
}
```

---

## **Где Singleton применяется в iOS?**  
1. **Системные классы Apple:**  
   - `FileManager.default`  
   - `URLSession.shared`  
   - `UserDefaults.standard`  
2. **Кастомные сервисы:**  
   - Логгер (`Logger.shared`).  
   - Кеш изображений (`ImageCacheManager.shared`).  
   - Аналитика (`AnalyticsTracker.shared`).  

---

## **Плюсы и минусы Singleton**  
✅ **Плюсы:**  
- Глобальный доступ к общему ресурсу.  
- Удобство (не нужно передавать объект между классами).  

❌ **Минусы:**  
- **Глобальное состояние** — усложняет отладку.  
- **Трудности с тестированием** — требует дополнительных ухищрений (как в примере с DI).  
- **Нарушает SRP** — если Singleton делает слишком много.  

---

## **Когда НЕ использовать Singleton?**  
- Если объект **не должен быть единственным** (например, каждая `UIViewController` должна создавать свой экземпляр сервиса).  
- Для **простых данных** — лучше использовать `struct` или `UserDefaults`.  

---

## **Альтернативы Singleton**  
1. **Dependency Injection (DI):**  
   ```swift
   let networkService = NetworkService() // Передаём явно, а не через shared
   let viewModel = ProfileViewModel(networkService: networkService)
   ```  
2. **SwiftUI `EnvironmentObject`:**  
   ```swift
   @EnvironmentObject var authManager: AuthManager // Аналог Singleton, но для SwiftUI
   ```
