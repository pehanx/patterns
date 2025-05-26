## 1. Реализация Builder для UIAlertController
### Шаг 1: Продукт (UIAlertController)
Это то, что мы будем строить.

### Шаг 2: Builder
Класс, который позволяет конфигурировать UIAlertController шаг за шагом.

```swift
import UIKit

final class AlertBuilder {
    private var title: String?
    private var message: String?
    private var tintColor: UIColor = .systemBlue
    private var icon: UIImage?
    private var actions: [UIAlertAction] = []
    private var preferredStyle: UIAlertController.Style = .alert
    
    // Устанавливаем заголовок
    func setTitle(_ title: String) -> Self {
        self.title = title
        return self
    }
    
    // Устанавливаем сообщение
    func setMessage(_ message: String) -> Self {
        self.message = message
        return self
    }
    
    // Устанавливаем цвет кнопок
    func setTintColor(_ color: UIColor) -> Self {
        self.tintColor = color
        return self
    }
    
    // Добавляем иконку
    func setIcon(_ image: UIImage?) -> Self {
        self.icon = image
        return self
    }
    
    // Добавляем действие (кнопку)
    func addAction(title: String, style: UIAlertAction.Style = .default, handler: (() -> Void)? = nil) -> Self {
        let action = UIAlertAction(title: title, style: style) { _ in
            handler?()
        }
        actions.append(action)
        return self
    }
    
    // Устанавливаем стиль (alert или actionSheet)
    func setPreferredStyle(_ style: UIAlertController.Style) -> Self {
        self.preferredStyle = style
        return self
    }
    
    // Собираем UIAlertController
    func build() -> UIAlertController {
        let alert = UIAlertController(
            title: title,
            message: message,
            preferredStyle: preferredStyle
        )
        
        alert.view.tintColor = tintColor
        
        if let icon = icon {
            let imageView = UIImageView(image: icon)
            alert.view.addSubview(imageView)
            // Здесь можно настроить констрейнты для иконки
        }
        
        actions.forEach { alert.addAction($0) }
        
        return alert
    }
}
```

## 2. Использование Builder в коде
### Пример 1: Простое уведомление

```swift
let alert = AlertBuilder()
    .setTitle("Внимание!")
    .setMessage("Вы уверены, что хотите удалить этот файл?")
    .addAction(title: "Отмена", style: .cancel)
    .addAction(title: "Удалить", style: .destructive) {
        print("Файл удалён!")
    }
    .build()

present(alert, animated: true)
```

### Пример 2: Кастомный Alert с иконкой

```swift
let customAlert = AlertBuilder()
    .setTitle("Успех!")
    .setMessage("Данные успешно сохранены.")
    .setTintColor(.systemGreen)
    .setIcon(UIImage(systemName: "checkmark.circle.fill"))
    .addAction(title: "OK") {
        print("OK нажата")
    }
    .build()

present(customAlert, animated: true)
```

### Пример 3: Action Sheet

```swift
let actionSheet = AlertBuilder()
    .setTitle("Выберите действие")
    .setPreferredStyle(.actionSheet)
    .addAction(title: "Фото", style: .default) {
        print("Выбрано фото")
    }
    .addAction(title: "Документ", style: .default) {
        print("Выбран документ")
    }
    .addAction(title: "Отмена", style: .cancel)
    .build()

present(actionSheet, animated: true)
```

## Где ещё применяется Builder в iOS?

### Конфигурация URLRequest:

```swift
let request = URLRequestBuilder()
    .setURL("https://api.example.com/users")
    .setMethod(.get)
    .addHeader("Authorization", "Bearer token")
    .build()
```
