**Паттерн Iterator (Итератор)** предоставляет способ последовательного доступа к элементам составного объекта, не раскрывая его внутренней структуры. В Swift и iOS этот паттерн встроен в стандартные коллекции через протокол `IteratorProtocol`.

---

## **Паттерн Iterator в Swift: Реальные примеры из iOS**

### **Пример 1: Кастомная коллекция для постраничной загрузки**

#### **Задача:**
Реализовать итератор для:
1. Постраничной загрузки данных из API
2. Ленивой подгрузки новых элементов
3. Обработки каждого элемента в цикле

---

#### **1. Реализация кастомного итератора**

```swift
struct PaginatedCollection<T> {
    private var currentPage = 0
    private let pageSize: Int
    private let loadPage: (Int) -> [T]?
    
    init(pageSize: Int, loadPage: @escaping (Int) -> [T]?) {
        self.pageSize = pageSize
        self.loadPage = loadPage
    }
}

// Поддержка Sequence для использования в for...in
extension PaginatedCollection: Sequence {
    func makeIterator() -> PaginatedIterator<T> {
        return PaginatedIterator(collection: self)
    }
}

// Кастомный итератор
struct PaginatedIterator<T>: IteratorProtocol {
    private var collection: PaginatedCollection<T>
    private var currentIndex = 0
    private var currentItems: [T] = []
    
    init(collection: PaginatedCollection<T>) {
        self.collection = collection
        loadNextPage()
    }
    
    mutating func next() -> T? {
        guard currentIndex < currentItems.count else {
            if loadNextPage() {
                return next()
            }
            return nil
        }
        
        let item = currentItems[currentIndex]
        currentIndex += 1
        return item
    }
    
    private mutating func loadNextPage() -> Bool {
        collection.currentPage += 1
        guard let newItems = collection.loadPage(collection.currentPage) else {
            return false
        }
        currentItems = newItems
        currentIndex = 0
        return !newItems.isEmpty
    }
}
```

#### **2. Использование**

```swift
// Имитация API
func mockAPI(page: Int) -> [String]? {
    let totalPages = 3
    guard page <= totalPages else { return nil }
    return Array(repeating: "Item", count: 5).map { "\($0) \(page * 5 + $0)" }
}

// Создаем коллекцию
let collection = PaginatedCollection(pageSize: 5, loadPage: mockAPI)

// Используем в цикле
for item in collection.prefix(8) { // Берем только первые 8 элементов
    print(item)
}

/* Вывод:
Item 1
Item 2
Item 3
Item 4
Item 5
Item 6
Item 7
Item 8
*/
```

---

### **Пример 2: Обход дерева каталогов (Composite + Iterator)**

#### **Задача:**
Реализовать:
1. Иерархическую структуру меню
2. Итератор для рекурсивного обхода
3. Поддержку `for...in`

---

#### **1. Базовые компоненты**

```swift
protocol MenuComponent {
    func makeIterator() -> AnyIterator<String>
}

// Лист (конечный элемент)
struct MenuItem: MenuComponent {
    let name: String
    
    func makeIterator() -> AnyIterator<String> {
        return AnyIterator([name].makeIterator())
    }
}

// Композит (узел с детьми)
class MenuCategory: MenuComponent {
    let name: String
    private var children: [MenuComponent] = []
    
    init(name: String) {
        self.name = name
    }
    
    func add(_ component: MenuComponent) {
        children.append(component)
    }
    
    func makeIterator() -> AnyIterator<String> {
        var stack: [AnyIterator<String>] = []
        
        // Добавляем текущее имя категории
        let selfIterator = AnyIterator([name].makeIterator())
        stack.append(selfIterator)
        
        // Добавляем итераторы детей
        stack.append(contentsOf: children.map { $0.makeIterator() })
        
        return AnyIterator {
            while !stack.isEmpty {
                if let next = stack.last?.next() {
                    return next
                }
                stack.removeLast()
            }
            return nil
        }
    }
}
```

#### **2. Использование**

```swift
// Строим меню
let drinks = MenuCategory(name: "Напитки")
drinks.add(MenuItem(name: "Кофе"))
drinks.add(MenuItem(name: "Чай"))

let food = MenuCategory(name: "Еда")
food.add(MenuItem(name: "Сэндвич"))
food.add(MenuItem(name: "Салат"))

let menu = MenuCategory(name: "Главное меню")
menu.add(drinks)
menu.add(food)

// Обход
for item in menu.makeIterator() {
    print(item)
}

/* Вывод:
Главное меню
Напитки
Кофе
Чай
Еда
Сэндвич
Салат
*/
```

---

## **Где применяется в iOS?**

1. **Стандартные коллекции:**
   ```swift
   let array = [1, 2, 3]
   var iterator = array.makeIterator()
   iterator.next() // 1
   ```

2. **Core Data:**
   ```swift
   let request: NSFetchRequest<Item> = Item.fetchRequest()
   let items = try context.fetch(request)
   for item in items { /* ... */ }
   ```

3. **Combine/SwiftUI:**
   ```swift
   Publishers.Sequence(sequence: [1, 2, 3])
       .sink { print($0) }
   ```

---

## **Плюсы и минусы**

✅ **Плюсы:**
- Единый интерфейс для разных коллекций
- Ленивая загрузка элементов
- Сокрытие внутренней структуры данных

❌ **Минусы:**
- Избыточен для простых массивов
- Сложность реализации для нелинейных структур

---

## **Сравнение с другими паттернами**

- **Composite:** Хранит древовидные структуры, а **Iterator** их обходит
- **Visitor:** Отделяет алгоритмы от структуры данных
- **Observer:** Уведомляет об изменениях, а **Iterator** просто перечисляет
