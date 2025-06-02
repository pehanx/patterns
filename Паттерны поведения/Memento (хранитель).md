**Паттерн Memento (Хранитель)** позволяет сохранять и восстанавливать предыдущие состояния объекта, не раскрывая деталей его реализации. В iOS он особенно полезен для:

1. Реализации undo/redo (отмена/повтор действий)
2. Сохранения и восстановления состояния приложения
3. Создания снимков (snapshots) сложных объектов

---

## **Паттерн Memento в Swift: Реальные примеры из iOS**

### **Пример 1: Система отмены для текстового редактора**

#### **Задача:**
Реализовать функционал:
- Сохранения состояния текста
- Отмены последнего изменения
- Возврата отмененного изменения

---

#### **1. Реализация паттерна**

```swift
// Объект, состояние которого нужно сохранять
class TextDocument {
    private var text: String
    private var cursorPosition: Int
    
    init(text: String = "", cursorPosition: Int = 0) {
        self.text = text
        self.cursorPosition = cursorPosition
    }
    
    func type(text newText: String) {
        text.insert(contentsOf: newText, at: text.index(text.startIndex, offsetBy: cursorPosition))
        cursorPosition += newText.count
    }
    
    func deleteBackward() {
        guard cursorPosition > 0 else { return }
        text.remove(at: text.index(text.startIndex, offsetBy: cursorPosition - 1))
        cursorPosition -= 1
    }
    
    // Создает снимок состояния
    func createMemento() -> TextMemento {
        return TextMemento(text: text, cursorPosition: cursorPosition)
    }
    
    // Восстанавливает состояние из снимка
    func restore(from memento: TextMemento) {
        self.text = memento.text
        self.cursorPosition = memento.cursorPosition
    }
    
    func printDocument() {
        print("Текст: '\(text)'")
        print("Курсор: \(cursorPosition)")
    }
}

// Memento - хранитель состояния
struct TextMemento {
    let text: String
    let cursorPosition: Int
    
    fileprivate init(text: String, cursorPosition: Int) {
        self.text = text
        self.cursorPosition = cursorPosition
    }
}

// Caretaker - управляет историей снимков
class History {
    private var mementos: [TextMemento] = []
    private var currentIndex = -1
    
    func save(_ memento: TextMemento) {
        // Удаляем все "переделанные" состояния при новом сохранении
        mementos = Array(mementos.prefix(currentIndex + 1))
        mementos.append(memento)
        currentIndex += 1
    }
    
    func undo() -> TextMemento? {
        guard currentIndex > 0 else { return nil }
        currentIndex -= 1
        return mementos[currentIndex]
    }
    
    func redo() -> TextMemento? {
        guard currentIndex < mementos.count - 1 else { return nil }
        currentIndex += 1
        return mementos[currentIndex]
    }
}
```

#### **2. Использование**

```swift
let document = TextDocument()
let history = History()

// Сохраняем начальное состояние
history.save(document.createMemento())

document.type(text: "Привет")
history.save(document.createMemento())
document.printDocument()

document.type(text: ", мир!")
history.save(document.createMemento())
document.printDocument()

// Отменяем последнее действие
if let memento = history.undo() {
    document.restore(from: memento)
    document.printDocument()
}

// Возвращаем отмененное действие
if let memento = history.redo() {
    document.restore(from: memento)
    document.printDocument()
}

/* Вывод:
Текст: 'Привет'
Курсор: 6
Текст: 'Привет, мир!'
Курсор: 11
Текст: 'Привет'
Курсор: 6
Текст: 'Привет, мир!'
Курсор: 11
*/
```

---

### **Пример 2: Сохранение состояния UIView**

#### **Задача:**
Сохранять и восстанавливать:
- Позицию view
- Размеры
- Альфа-канал

---

#### **1. Реализация**

```swift
// Memento для UIView
struct ViewMemento {
    let frame: CGRect
    let alpha: CGFloat
}

extension UIView {
    func createMemento() -> ViewMemento {
        return ViewMemento(frame: frame, alpha: alpha)
    }
    
    func restore(from memento: ViewMemento) {
        frame = memento.frame
        alpha = memento.alpha
    }
}

// Пример использования
let view = UIView(frame: CGRect(x: 0, y: 0, width: 100, height: 100))
view.alpha = 0.5

// Сохраняем состояние
let savedState = view.createMemento()

// Меняем view
view.frame = CGRect(x: 50, y: 50, width: 200, height: 200)
view.alpha = 1.0

// Восстанавливаем
view.restore(from: savedState)
```

---

## **Где применяется в iOS?**

1. **Core Data и UserDefaults** - по сути реализуют Memento для сохранения состояния
2. **UIStateRestoration** - восстановление состояния интерфейса
3. **UndoManager** - встроенная реализация undo/redo в UIKit

---

## **Плюсы и минусы**

✅ **Плюсы:**
- Сохраняет инкапсуляцию (не раскрывает детали состояния)
- Упрощает создание механизма отмены
- Позволяет сохранять историю состояний

❌ **Минусы:**
- Может потреблять много памяти для сложных объектов
- Усложняет код, если нужно сохранять только часть состояния

---

## **Сравнение с другими паттернами**

- **Command** - хранит операции, а **Memento** хранит состояния
- **Prototype** - копирует объекты, а **Memento** сохраняет состояние
- **Snapshot** в SwiftUI - аналогичная концепция для View
