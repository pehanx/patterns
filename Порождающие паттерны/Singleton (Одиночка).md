## 1. Базовая реализация Singleton

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

### Использование:

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

## 2. Улучшенная версия (потокобезопасность)
Если Singleton используется в многопоточной среде (например, сетевые запросы), добавляем DispatchQueue для синхронизации:

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

## 3. Singleton + Dependency Injection (для тестирования)
Проблема: Singleton трудно тестировать из-за глобального состояния.
Решение: Используем протокол и внедряем зависимость.

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

## Где Singleton применяется в iOS?
1. **Системные классы Apple**:
- ```FileManager.default```
- ```URLSession.shared```
- ```UserDefaults.standard```
2. **Кастомные сервисы**:
- Логгер (```Logger.shared```).
- Кеш изображений (```ImageCacheManager.shared```).
- Аналитика (```AnalyticsTracker.shared```).
