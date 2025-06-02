**Абстрактная фабрика (Abstract Factory)** — это порождающий паттерн, который создаёт **семейства связанных объектов**, не привязываясь к конкретным классам. В iOS он особенно полезен для:

- Кроссплатформенных UI (iOS/macOS/tvOS)
- Работы с разными API (Production/Mock)
- Создания совместимых компонентов (например, тем оформления)

---

## **Abstract Factory в Swift: Реальный пример из iOS**

### **Задача:**
Разработать систему UI-компонентов, которая:
1. Поддерживает **разные стили** (Apple Human Interface Guidelines для iOS и macOS)
2. Автоматически создает **согласованные компоненты** (кнопки, текстовые поля, переключатели)
3. Легко расширяется для новых платформ (например, watchOS)

---

## **1. Реализация Abstract Factory**

### **Шаг 1: Определяем абстрактные продукты (протоколы)**
```swift
// Абстрактная кнопка
protocol Button {
    func render()
    func onTap(action: @escaping () -> Void)
}

// Абстрактный текстовый поле
protocol TextField {
    func render()
    func setPlaceholder(_ text: String)
}

// Абстрактный переключатель
protocol Switch {
    func render()
    func setOn(_ isOn: Bool)
}
```

### **Шаг 2: Конкретные продукты для iOS**
```swift
// iOS-кнопка (UIKit)
final class iOSButton: Button {
    private var tapAction: (() -> Void)?
    
    func render() {
        print("Отрисована iOS-кнопка (UIButton)")
    }
    
    func onTap(action: @escaping () -> Void) {
        self.tapAction = action
        print("iOS: Добавлен обработчик нажатия")
    }
}

// iOS-текстовое поле
final class iOSTextField: TextField {
    func render() {
        print("Отрисовано iOS-текстовое поле (UITextField)")
    }
    
    func setPlaceholder(_ text: String) {
        print("iOS: Установлен плейсхолдер '\(text)'")
    }
}
```

### **Шаг 3: Конкретные продукты для macOS**
```swift
// macOS-кнопка (AppKit)
final class MacButton: Button {
    func render() {
        print("Отрисована macOS-кнопка (NSButton)")
    }
    
    func onTap(action: @escaping () -> Void) {
        print("macOS: Добавлен обработчик нажатия (target-action)")
    }
}

// macOS-текстовое поле
final class MacTextField: TextField {
    func render() {
        print("Отрисовано macOS-текстовое поле (NSTextField)")
    }
    
    func setPlaceholder(_ text: String) {
        print("macOS: Установлен placeholderString '\(text)'")
    }
}
```

### **Шаг 4: Абстрактная фабрика (протокол)**
```swift
protocol UIFactory {
    func createButton() -> Button
    func createTextField() -> TextField
    func createSwitch() -> Switch
}
```

### **Шаг 5: Конкретные фабрики**
```swift
// Фабрика для iOS
final class iOSUIFactory: UIFactory {
    func createButton() -> Button {
        return iOSButton()
    }
    
    func createTextField() -> TextField {
        return iOSTextField()
    }
    
    func createSwitch() -> Switch {
        return iOSSwitch() // Реализация аналогична другим компонентам
    }
}

// Фабрика для macOS
final class MacUIFactory: UIFactory {
    func createButton() -> Button {
        return MacButton()
    }
    
    func createTextField() -> TextField {
        return MacTextField()
    }
    
    func createSwitch() -> Switch {
        return MacSwitch()
    }
}
```

---

## **2. Использование в коде**

### **Пример 1: Создание кроссплатформенного UI**
```swift
class SettingsScreen {
    private let factory: UIFactory
    
    init(factory: UIFactory) {
        self.factory = factory
    }
    
    func setupUI() {
        let button = factory.createButton()
        let textField = factory.createTextField()
        
        button.onTap {
            print("Кнопка нажата!")
        }
        
        textField.setPlaceholder("Введите имя")
        
        button.render()
        textField.render()
    }
}

// Для iOS
let iOSSettings = SettingsScreen(factory: iOSUIFactory())
iOSSettings.setupUI()

// Для macOS
let macSettings = SettingsScreen(factory: MacUIFactory())
macSettings.setupUI()
```

**Вывод для iOS:**
```
iOS: Добавлен обработчик нажатия
iOS: Установлен плейсхолдер 'Введите имя'
Отрисована iOS-кнопка (UIButton)
Отрисовано iOS-текстовое поле (UITextField)
```

**Вывод для macOS:**
```
macOS: Добавлен обработчик нажатия (target-action)
macOS: Установлен placeholderString 'Введите имя'
Отрисована macOS-кнопка (NSButton)
Отрисовано macOS-текстовое поле (NSTextField)
```

---

## **3. Где применяется в iOS?**

1. **Темы оформления**:
   ```swift
   protocol ThemeFactory {
       func createPrimaryButton() -> Button
       func createSecondaryButton() -> Button
   }
   
   class LightTheme: ThemeFactory { ... }
   class DarkTheme: ThemeFactory { ... }
   ```

2. **Работа с разными API**:
   ```swift
   protocol NetworkFactory {
       func createAuthService() -> AuthService
       func createDataService() -> DataService
   }
   
   class ProductionFactory: NetworkFactory { ... }
   class MockFactory: NetworkFactory { ... }
   ```

3. **Поддержка разных версий ОС**:
   ```swift
   func makeFactory() -> UIFactory {
       if #available(iOS 15, *) {
           return iOS15UIFactory()
       } else {
           return LegacyUIFactory()
       }
   }
   ```

---

## **Плюсы и минусы**

✅ **Плюсы:**
- Гарантирует совместимость компонентов
- Упрощает поддержку новых вариаций
- Изолирует конкретные классы

❌ **Минусы:**
- Сложность добавления новых типов продуктов
- Может привести к созданию многих классов

---

## **Сравнение с другими паттернами**

- **Factory Method** — создаёт один продукт, Abstract Factory — целое семейство
- **Builder** — концентрируется на пошаговом создании, Abstract Factory — на совместимости
