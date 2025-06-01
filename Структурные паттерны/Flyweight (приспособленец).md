**Паттерн Flyweight (Приспособленец)** помогает экономить память, разделяя общие части состояния между множеством объектов. В iOS он особенно полезен для:

- Оптимизации работы с тяжелыми ресурсами (изображения, шрифты)
- Управления большим количеством однотипных UI-компонентов
- Реализации пулов объектов

---

## **Паттерн Flyweight в Swift: Реальный пример из iOS**

### **Задача:**
Оптимизировать отображение 10,000 ячеек таблицы с:
- Одинаковыми иконками
- Разными текстовыми метками
- Разным цветом статуса

Без Flyweight каждая ячейка создавала бы свою копию иконки, что неэффективно.

---

## **1. Реализация Flyweight**

### **Шаг 1: Выделяем изменяемое и неизменяемое состояние**

```swift
// Неизменяемое разделяемое состояние (Flyweight)
struct IconAsset {
    let name: String
    let image: UIImage
    
    init(name: String) {
        self.name = name
        self.image = UIImage(named: name)!
    }
}

// Изменяемое внешнее состояние (контекст)
struct CellConfiguration {
    let title: String
    let statusColor: UIColor
}
```

### **Шаг 2: Фабрика Flyweight-объектов**

```swift
final class IconFactory {
    private static var cache: [String: IconAsset] = [:]
    
    static func icon(for name: String) -> IconAsset {
        if let cached = cache[name] {
            return cached
        }
        
        let newIcon = IconAsset(name: name)
        cache[name] = newIcon
        return newIcon
    }
}
```

### **Шаг 3: Применяем в UITableViewCell**

```swift
class OptimizedTableViewCell: UITableViewCell {
    private let iconView = UIImageView()
    private let titleLabel = UILabel()
    private let statusView = UIView()
    
    func configure(with config: CellConfiguration, iconName: String) {
        // Разделяемый Flyweight-объект
        let icon = IconFactory.icon(for: iconName)
        
        // Уникальное состояние
        titleLabel.text = config.title
        statusView.backgroundColor = config.statusColor
        
        // Настройка UI
        iconView.image = icon.image
    }
}
```

---

## **2. Использование в коде**

### **Пример: Отображение списка сообщений**

```swift
let messages = [
    (text: "Привет!", status: UIColor.green, icon: "read_icon"),
    (text: "Как дела?", status: UIColor.blue, icon: "unread_icon"),
    // ... 9998 других сообщений
]

func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
    let cell = tableView.dequeueReusableCell(withIdentifier: "cell", for: indexPath) as! OptimizedTableViewCell
    
    let message = messages[indexPath.row]
    cell.configure(
        with: CellConfiguration(
            title: message.text, 
            statusColor: message.status
        ),
        iconName: message.icon
    )
    
    return cell
}
```

---

## **3. Оптимизация памяти**

### **До применения Flyweight:**
- 10,000 ячеек × 3 КБ (иконка) = 30 МБ памяти

### **После применения Flyweight:**
- 2 уникальные иконки × 3 КБ = 6 КБ
- 10,000 ячеек × 50 байт (ссылка) = 500 КБ
- **Итого:** ~506 КБ вместо 30 МБ

---

## **Плюсы и минусы Flyweight в iOS**

✅ **Плюсы:**
- Экономит память при работе с множеством объектов
- Уменьшает время инициализации
- Упрощает работу с кешированием

❌ **Минусы:**
- Усложняет код (необходимость разделения состояния)
- Неэффективен при малом количестве объектов

---

## **Где применять в iOS?**

1. **Работа с изображениями:**
   ```swift
   // Вместо:
   UIImage(named: "icon")!
   
   // Используем:
   ImageCache.shared.image(for: "icon")
   ```

2. **Текстовые атрибуты:**
   ```swift
   // Разделяемое состояние
   struct TextAttributes {
       let font: UIFont
       let color: UIColor
   }
   
   class AttributesFactory {
       static let titleAttributes = TextAttributes(font: .boldSystemFont(ofSize: 16), color: .black)
       static let bodyAttributes = TextAttributes(font: .systemFont(ofSize: 14), color: .gray)
   }
   ```

3. **Ячейки коллекций/таблиц** с одинаковыми стилями.

---

## **Сравнение с другими паттернами**

🆚 **Flyweight vs Singleton**:
- Singleton гарантирует один экземпляр на всю программу
- Flyweight допускает несколько экземпляров, но разделяет общее состояние

🆚 **Flyweight vs Cache**:
- Cache хранит данные временно
- Flyweight - это архитектурный паттерн для разделения состояния
