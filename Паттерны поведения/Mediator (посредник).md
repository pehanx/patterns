**Паттерн Mediator (Посредник)** упрощает взаимодействие между компонентами системы, перенося логику их общения в отдельный объект-посредник. В iOS он особенно полезен для:

- Снижения связанности между ViewController'ами
- Организации сложных взаимодействий в UI
- Управления состоянием приложения

---

## **Паттерн Mediator в Swift: Реальные примеры из iOS**

### **Пример 1: Чат-приложение с несколькими участниками**

#### **Задача:**
Реализовать чат, где:
1. Пользователи могут отправлять сообщения
2. Сообщения рассылаются всем участникам
3. Нет прямой связи между пользователями

---

#### **1. Реализация паттерна**

```swift
// Протокол посредника
protocol ChatMediator: AnyObject {
    func send(message: String, from sender: User)
    func addUser(_ user: User)
}

// Конкретный посредник
class ChatRoom: ChatMediator {
    private var users: [User] = []
    
    func addUser(_ user: User) {
        users.append(user)
        print("\(user.name) присоединился к чату")
    }
    
    func send(message: String, from sender: User) {
        for user in users {
            // Не отправляем сообщение отправителю
            if user !== sender {
                user.receive(message: message)
            }
        }
    }
}

// Класс пользователя
class User {
    let name: String
    private weak var mediator: ChatMediator?
    
    init(name: String, mediator: ChatMediator) {
        self.name = name
        self.mediator = mediator
        mediator.addUser(self)
    }
    
    func send(message: String) {
        print("\(name) отправляет: \(message)")
        mediator?.send(message: message, from: self)
    }
    
    func receive(message: String) {
        print("\(name) получает: \(message)")
    }
}
```

#### **2. Использование**

```swift
let chatRoom = ChatRoom()

let alice = User(name: "Алиса", mediator: chatRoom)
let bob = User(name: "Боб", mediator: chatRoom)
let charlie = User(name: "Чарли", mediator: chatRoom)

alice.send(message: "Привет всем!")
bob.send(message: "Привет, Алиса!")

/* Вывод:
Алиса присоединился к чату
Боб присоединился к чату
Чарли присоединился к чату
Алиса отправляет: Привет всем!
Боб получает: Привет всем!
Чарли получает: Привет всем!
Боб отправляет: Привет, Алиса!
Алиса получает: Привет, Алиса!
Чарли получает: Привет, Алиса!
*/
```

---

### **Пример 2: Координация UI в iOS (без Storyboard Segues)**

#### **Задача:**
Организовать навигацию между экранами так, чтобы:
1. ViewController'ы не знали друг о друге
2. Вся навигация управлялась централизованно

---

#### **1. Реализация**

```swift
// Протокол посредника
protocol AppNavigationMediator: AnyObject {
    func showLoginScreen()
    func showMainScreen(user: String)
    func showProfileScreen(user: String)
}

// Реализация посредника
class AppCoordinator: AppNavigationMediator {
    private weak var window: UIWindow?
    private var navigationController: UINavigationController?
    
    init(window: UIWindow?) {
        self.window = window
    }
    
    func start() {
        showLoginScreen()
    }
    
    func showLoginScreen() {
        let loginVC = LoginViewController()
        loginVC.mediator = self
        navigationController = UINavigationController(rootViewController: loginVC)
        window?.rootViewController = navigationController
    }
    
    func showMainScreen(user: String) {
        let mainVC = MainViewController()
        mainVC.mediator = self
        mainVC.user = user
        navigationController?.pushViewController(mainVC, animated: true)
    }
    
    func showProfileScreen(user: String) {
        let profileVC = ProfileViewController()
        profileVC.user = user
        navigationController?.pushViewController(profileVC, animated: true)
    }
}

// Базовый класс для ViewController'ов
class BaseViewController: UIViewController {
    weak var mediator: AppNavigationMediator?
}

// Конкретные ViewController'ы
class LoginViewController: BaseViewController {
    @IBAction func loginButtonTapped() {
        mediator?.showMainScreen(user: "Текущий пользователь")
    }
}

class MainViewController: BaseViewController {
    var user: String!
    
    @IBAction func profileButtonTapped() {
        mediator?.showProfileScreen(user: user)
    }
}
```

#### **2. Инициализация в AppDelegate**

```swift
func application(_ application: UIApplication, 
               didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
    
    let window = UIWindow(frame: UIScreen.main.bounds)
    let coordinator = AppCoordinator(window: window)
    coordinator.start()
    
    window.makeKeyAndVisible()
    self.window = window
    
    return true
}
```

---

## **Где применяется в iOS?**

1. **Координаторы (Coordinators)** - популярная архитектура для навигации
2. **Централизованные системы событий** (аналог NotificationCenter, но типизированный)
3. **Сложные UI-компоненты** (например, кастомные контролы с множеством взаимодействий)

---

## **Плюсы и минусы**

✅ **Плюсы:**
- Уменьшает связанность между компонентами
- Централизует управление взаимодействиями
- Упрощает добавление новых компонентов

❌ **Минусы:**
- Посредник может превратиться в "божественный объект"
- Усложняет отслеживание потока событий

---

## **Сравнение с другими паттернами**

- **Observer:** Рассылает события всем подписчикам, а **Mediator** управляет направленными взаимодействиями
- **Facade:** Упрощает интерфейс к подсистеме, а **Mediator** добавляет новую логику взаимодействий
- **Command:** Инкапсулирует запросы, а **Mediator** определяет как они будут обрабатываться
