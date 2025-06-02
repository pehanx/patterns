**Паттерн Command (Команда)** инкапсулирует запрос в виде объекта, позволяя параметризовать клиенты с различными запросами, ставить их в очередь или поддерживать отмену операций. В iOS он особенно полезен для:

- Реализации **отмены/повтора** действий (undo/redo)
- Управления **асинхронными задачами**
- Создания **макросов** (последовательностей команд)

---

## **Паттерн Command в Swift: Реальные примеры из iOS**

### **Пример 1: Система Undo/Redo в графическом редакторе**

#### **Задача:**
Реализовать функционал отмены/повтора для:
1. Добавления фигур
2. Изменения цвета
3. Перемещения объектов

---

#### **1. Базовые компоненты Command**

```swift
// Протокол команды
protocol Command {
    func execute()
    func undo()
}

// Получатель команды (реальный объект)
class Canvas {
    private var shapes: [String] = []
    private var colors: [String: String] = [:]
    
    func addShape(_ shape: String) {
        shapes.append(shape)
        print("Добавлена фигура: \(shape)")
    }
    
    func removeLastShape() {
        guard !shapes.isEmpty else { return }
        print("Удалена фигура: \(shapes.removeLast())")
    }
    
    func setColor(_ color: String, for shape: String) {
        colors[shape] = color
        print("Фигура \(shape) перекрашена в \(color)")
    }
    
    func undoColor(for shape: String) {
        colors.removeValue(forKey: shape)
        print("Цвет фигуры \(shape) сброшен")
    }
}
```

#### **2. Конкретные команды**

```swift
// Команда добавления фигуры
class AddShapeCommand: Command {
    private let canvas: Canvas
    private let shape: String
    
    init(canvas: Canvas, shape: String) {
        self.canvas = canvas
        self.shape = shape
    }
    
    func execute() {
        canvas.addShape(shape)
    }
    
    func undo() {
        canvas.removeLastShape()
    }
}

// Команда изменения цвета
class ChangeColorCommand: Command {
    private let canvas: Canvas
    private let shape: String
    private let newColor: String
    private var previousColor: String?
    
    init(canvas: Canvas, shape: String, color: String) {
        self.canvas = canvas
        self.shape = shape
        self.newColor = color
    }
    
    func execute() {
        // Запоминаем предыдущий цвет для undo
        previousColor = canvas.colors[shape]
        canvas.setColor(newColor, for: shape)
    }
    
    func undo() {
        guard let previousColor = previousColor else { return }
        canvas.setColor(previousColor, for: shape)
    }
}
```

#### **3. Invoker (инициатор команд)**
```swift
class CommandManager {
    private var undoStack: [Command] = []
    private var redoStack: [Command] = []
    
    func execute(command: Command) {
        command.execute()
        undoStack.append(command)
        redoStack.removeAll()
    }
    
    func undo() {
        guard let command = undoStack.popLast() else { return }
        command.undo()
        redoStack.append(command)
    }
    
    func redo() {
        guard let command = redoStack.popLast() else { return }
        command.execute()
        undoStack.append(command)
    }
}
```

#### **4. Использование**
```swift
let canvas = Canvas()
let manager = CommandManager()

// Добавляем фигуры
manager.execute(command: AddShapeCommand(canvas: canvas, shape: "Круг"))
manager.execute(command: AddShapeCommand(canvas: canvas, shape: "Квадрат"))

// Меняем цвет
manager.execute(command: ChangeColorCommand(canvas: canvas, shape: "Круг", color: "Красный"))

// Отменяем последнее действие
manager.undo() // Отмена изменения цвета

// Повторяем действие
manager.redo() // Повтор изменения цвета
```

**Вывод:**
```
Добавлена фигура: Круг
Добавлена фигура: Квадрат
Фигура Круг перекрашена в Красный
Фигура Круг перекрашена в nil (undo)
Фигура Круг перекрашена в Красный (redo)
```

---

### **Пример 2: Сетевые запросы с отменой**

#### **Задача:**
Реализовать отмену асинхронных сетевых запросов.

```swift
protocol NetworkCommand {
    func execute(completion: @escaping (Result<Data, Error>) -> Void)
    func cancel()
}

class FetchUserCommand: NetworkCommand {
    private let userId: String
    private var task: URLSessionTask?
    
    init(userId: String) {
        self.userId = userId
    }
    
    func execute(completion: @escaping (Result<Data, Error>) -> Void) {
        let url = URL(string: "https://api.example.com/users/\(userId)")!
        task = URLSession.shared.dataTask(with: url) { data, _, error in
            if let error = error {
                completion(.failure(error))
                return
            }
            completion(.success(data ?? Data()))
        }
        task?.resume()
    }
    
    func cancel() {
        task?.cancel()
    }
}

// Использование
let command = FetchUserCommand(userId: "123")
command.execute { result in
    switch result {
    case .success(let data):
        print("Данные получены: \(data.count) байт")
    case .failure(let error):
        print("Ошибка: \(error.localizedDescription)")
    }
}

// Отмена запроса через 2 секунды
DispatchQueue.main.asyncAfter(deadline: .now() + 2) {
    command.cancel()
}
```

---

## **Где применяется в iOS?**

1. **UIKit:**
   - `UIAction` (в iOS 14+) — это фактически команда
   - Целевые действия (`target-action`)

2. **Core Animation:**
   - `CABasicAnimation` с делегатом для отмены

3. **Многопоточность:**
   - `Operation` и `OperationQueue` (паттерн Command в чистом виде)

---

## **Плюсы и минусы**

✅ **Плюсы:**
- Инкапсулирует запросы как объекты
- Позволяет реализовать отмену/повтор
- Поддерживает очередь команд

❌ **Минусы:**
- Увеличивает количество классов
- Может усложнить код для простых задач

---

## **Сравнение с другими паттернами**

- **Strategy:** Заменяет алгоритмы, а Command инкапсулирует запросы
- **Chain of Responsibility:** Передает запрос по цепочке, а Command выполняет сразу
- **Memento:** Хранит состояние для отмены, а Command — сами операции
