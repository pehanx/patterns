**Factory Method (фабричный метод)** — это порождающий паттерн, который делегирует создание объектов подклассам, позволяя изменять тип создаваемых объектов на лету.  

### **Factory Method в Swift: Реальный пример из iOS**  
#### **Задача:**  
Допустим, у нас есть приложение с разными типами **документов** (текстовый, PDF, таблица), и каждый документ требует своей логики создания и отображения.  

Вместо жесткого связывания кода с конкретными классами, мы используем **Factory Method**, чтобы:  
- **Гибко создавать** документы.  
- **Легко добавлять** новые типы документов.  

---

## **1. Реализация Factory Method**  

### **Шаг 1: Абстрактный класс/протокол**  
Определяем интерфейс для создания документа.  

```swift
protocol Document {
    func open()
    func save()
}

// Абстрактный "создатель" (Creator)
protocol DocumentCreator {
    func createDocument() -> Document
    func setupDocument()  // Дополнительная логика
}

extension DocumentCreator {
    func setupDocument() {
        print("Настройка документа (базовая реализация)")
    }
}
```

---

### **Шаг 2: Конкретные продукты**  
Реализуем разные типы документов.  

```swift
// Текстовый документ
final class TextDocument: Document {
    func open() {
        print("Текстовый документ открыт (Rich Text)")
    }
    
    func save() {
        print("Текстовый документ сохранён в .rtf")
    }
}

// PDF-документ
final class PDFDocument: Document {
    func open() {
        print("PDF открыт в QLPreviewController")
    }
    
    func save() {
        print("PDF сохранён в Files.app")
    }
}

// Таблица (Numbers/Excel)
final class SpreadsheetDocument: Document {
    func open() {
        print("Таблица открыта в кастомном редакторе")
    }
    
    func save() {
        print("Таблица экспортирована в .xlsx")
    }
}
```

---

### **Шаг 3: Конкретные создатели**  
Каждый создатель возвращает свой тип документа.  

```swift
// Создатель текстовых документов
final class TextDocumentCreator: DocumentCreator {
    func createDocument() -> Document {
        return TextDocument()
    }
    
    func setupDocument() {
        print("Добавляем форматирование текста...")
    }
}

// Создатель PDF
final class PDFDocumentCreator: DocumentCreator {
    func createDocument() -> Document {
        return PDFDocument()
    }
}

// Создатель таблиц
final class SpreadsheetDocumentCreator: DocumentCreator {
    func createDocument() -> Document {
        return SpreadsheetDocument()
    }
}
```

---

## **2. Использование в коде**  

### **Пример 1: Создание документа**  
```swift
func createAndUseDocument(creator: DocumentCreator) {
    let document = creator.createDocument()
    creator.setupDocument()
    document.open()
    document.save()
}

// Текстовый документ
let textCreator = TextDocumentCreator()
createAndUseDocument(creator: textCreator)

// Вывод:
// Добавляем форматирование текста...
// Текстовый документ открыт (Rich Text)
// Текстовый документ сохранён в .rtf
```

### **Пример 2: Динамический выбор типа**  
```swift
enum DocumentType {
    case text, pdf, spreadsheet
}

func getCreator(for type: DocumentType) -> DocumentCreator {
    switch type {
    case .text: return TextDocumentCreator()
    case .pdf: return PDFDocumentCreator()
    case .spreadsheet: return SpreadsheetDocumentCreator()
    }
}

let userChoice: DocumentType = .pdf
let creator = getCreator(for: userChoice)
createAndUseDocument(creator: creator)

// Вывод:
// Настройка документа (базовая реализация)
// PDF открыт в QLPreviewController
// PDF сохранён в Files.app
```

---

## **Плюсы и минусы Factory Method в iOS**  

✅ **Плюсы:**  
- **Гибкость** – новые типы документов добавляются без изменения существующего кода.  
- **Инкапсуляция** – логика создания скрыта в подклассах.  
- **Тестируемость** – легко подменить реализацию (например, на `MockDocument`).  

❌ **Минусы:**  
- **Усложнение кода** – требуется создавать иерархию классов.  
- **Избыточность** – если типов объектов мало, можно обойтись простым `switch`.  

---

## **Где применяется в iOS?**  
1. **`UIKit`:**  
   - `UILabel`, `UIButton` и другие view могут создаваться через фабричные методы (например, для темной/светлой темы).  

2. **`CoreData`:**  
   - Разные `NSManagedObject` подклассы для различных сущностей.  

3. **Сетевые запросы:**  
   - Создание `URLRequest` для разных эндпоинтов API.  

---

## **Сравнение с Abstract Factory**  
- **Factory Method** – создаёт **один** продукт, но позволяет подклассам выбирать тип.  
- **Abstract Factory** – создаёт **семейства** связанных продуктов.  
