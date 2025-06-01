**Паттерн Composite (Компоновщик)** позволяет работать с древовидными структурами, где и отдельные объекты, и их композиции обрабатываются единообразно. В iOS это особенно полезно для:
- Построения сложных UI-иерархий
- Организации файловых систем
- Обработки групп вью/виджетов

## Паттерн Composite в Swift: Реальный пример из iOS
**Задача**:
Разработаем систему отображения постов в соцсети, где пост может быть:
1. Простым (текст/картинка)
2. Составным (карусель из нескольких медиа)
3. Группой постов (лента/сторис)

## 1. Реализация Composite
### Шаг 1: Определяем базовый компонент (протокол)

```swift
protocol PostComponent {
    var id: UUID { get }
    func render() -> UIView
    func add(child: PostComponent)
    func remove(child: PostComponent)
}
```

### Шаг 2: Листовой компонент (простой пост)

```swift
final class SimplePost: PostComponent {
    let id = UUID()
    private let content: String
    
    init(content: String) {
        self.content = content
    }
    
    func render() -> UIView {
        let label = UILabel()
        label.text = content
        label.numberOfLines = 0
        return label
    }
    
    // Пустые методы (лист не может иметь детей)
    func add(child: PostComponent) {}
    func remove(child: PostComponent) {}
}
```

### Шаг 3: Составной компонент (группа постов)

```swift
final class PostGroup: PostComponent {
    let id = UUID()
    private var children = [PostComponent]()
    
    func render() -> UIView {
        let stack = UIStackView()
        stack.axis = .vertical
        stack.spacing = 8
        
        children.forEach {
            stack.addArrangedSubview($0.render())
        }
        
        return stack
    }
    
    func add(child: PostComponent) {
        children.append(child)
    }
    
    func remove(child: PostComponent) {
        children.removeAll { $0.id == child.id }
    }
}
```

## 2. Использование в коде
### Пример 1: Простая лента постов

```swift
let feed = PostGroup()

let post1 = SimplePost(content: "Привет, это мой первый пост!")
let post2 = SimplePost(content: "Сегодня красивая погода")

feed.add(child: post1)
feed.add(child: post2)

// Отображаем
let feedView = feed.render()
Пример 2: Сложная композиция (сторис + карусель)
swift
let story = PostGroup()
let story1 = SimplePost(content: "Story 1")
let story2 = SimplePost(content: "Story 2")
story.add(child: story1)
story.add(child: story2)

let carousel = PostGroup()
let slide1 = SimplePost(content: "Slide 1")
let slide2 = SimplePost(content: "Slide 2")
carousel.add(child: slide1)
carousel.add(child: slide2)

let mainFeed = PostGroup()
mainFeed.add(child: story)
mainFeed.add(child: carousel)

// Получаем готовую иерархию UIView
let contentView = mainFeed.render()
```

## 3. Оптимизация для iOS
Добавляем UI-специфичные методы

```swift
extension PostComponent {
    func animateAppear(duration: TimeInterval) {
        let view = self.render()
        view.alpha = 0
        UIView.animate(withDuration: duration) {
            view.alpha = 1
        }
    }
}

// Использование
post1.animateAppear(duration: 0.5)
```

## Плюсы и минусы Composite в iOS
✅ **Плюсы**:
- Единообразная работа с простыми и сложными элементами
- Простое добавление новых типов компонентов
- Рекурсивная обработка иерархий

❌ **Минусы**:
- Сложно ограничить типы дочерних элементов
- Может привести к излишней генерации объектов

## Где применяется в iOS?
1. **UIView иерархии**:
- UIStackView с вложенными стеками
- Кастомные коллекции вью
2. **CoreData Relationships**:
- Древовидные структуры данных
3. **Меню/навигация**:
- Вложенные разделы настроек

## Сравнение с другими паттернами
- **Decorator** добавляет поведение, а **Composite** структурирует
- **Chain of Responsibility** передает запросы, а **Composite** обрабатывает иерархию
