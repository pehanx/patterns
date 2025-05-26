## 1. Реализация Prototype
### Шаг 1: Протокол NSCopying (для копирования)

```swift
import Foundation

// Объект, который можно клонировать
protocol RoomPrototype: NSCopying {
    var name: String { get set }
    var color: String { get set }
    var furniture: [String] { get set }
    
    func clone() -> Self
}

extension RoomPrototype {
    func clone() -> Self {
        return self.copy() as! Self
    }
}
```

### Шаг 2: Конкретный прототип

```swift
final class Room: RoomPrototype {
    var name: String
    var color: String
    var furniture: [String]
    
    init(name: String, color: String, furniture: [String]) {
        self.name = name
        self.color = color
        self.furniture = furniture
    }
    
    // Реализация NSCopying
    func copy(with zone: NSZone? = nil) -> Any {
        let copy = Room(name: name, color: color, furniture: furniture)
        return copy
    }
}
```

## 2. Использование в коде
### Пример 1: Клонирование комнаты

```swift
// Создаем прототип "Офис"
let officePrototype = Room(
    name: "Офис",
    color: "Белый",
    furniture: ["Стол", "Стул", "Шкаф"]
)

// Клонируем прототип и настраиваем
let customOffice = officePrototype.clone()
customOffice.name = "Мой офис"
customOffice.furniture.append("Диван")

print("Прототип:", officePrototype.name, officePrototype.furniture)
print("Клон:", customOffice.name, customOffice.furniture)

// Вывод:
// Прототип: Офис ["Стол", "Стул", "Шкаф"]
// Клон: Мой офис ["Стол", "Стул", "Шкаф", "Диван"]
```

### Пример 2: Клонирование UI-элементов

```swift
// Прототип кнопки
let baseButton = UIButton()
baseButton.backgroundColor = .blue
baseButton.setTitle("Кнопка", for: .normal)

// Клонируем (через архив/разархив)
if let clonedButton = try? NSKeyedUnarchiver.unarchivedObject(
    ofClass: UIButton.self,
    from: NSKeyedArchiver.archivedData(withRootObject: baseButton)
) {
    clonedButton.setTitle("Новая кнопка", for: .normal)
    // Добавляем на экран...
}
```

## Где применяется в iOS?
**1. Игры**: Клонирование персонажей/объектов с одинаковыми текстурами.
**2. Кеширование**: Быстрое восстановление сложных объектов (например, UIImage из кеша).
**3. UIKit**: Кастомные UIView, которые нужно дублировать (например, ячейки таблиц).
