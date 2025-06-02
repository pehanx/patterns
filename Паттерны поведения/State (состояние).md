**Паттерн State (Состояние)** позволяет объекту изменять свое поведение при изменении внутреннего состояния, делая код чище и избавляясь от множества условных операторов. В iOS он особенно полезен для:

- Управления состоянием UI (загрузка, ошибка, контент)
- Реализации конечных автоматов
- Обработки сложных жизненных циклов

---

## **Паттерн State в Swift: Реальные примеры из iOS**

### **Пример 1: Плеер с разными состояниями**

#### **Задача:**
Реализовать аудиоплеер с состояниями:
- `Ready` (готов к воспроизведению)
- `Playing` (воспроизведение)
- `Paused` (на паузе)
- `Finished` (завершено)

---

#### **1. Реализация паттерна**

```swift
// Протокол состояния
protocol PlayerState: AnyObject {
    func play()
    func pause()
    func stop()
}

// Контекст (плеер)
class AudioPlayer {
    private var state: PlayerState
    
    init() {
        self.state = ReadyState(player: self)
    }
    
    func changeState(_ state: PlayerState) {
        self.state = state
    }
    
    func play() {
        state.play()
    }
    
    func pause() {
        state.pause()
    }
    
    func stop() {
        state.stop()
    }
}

// Конкретные состояния
class ReadyState: PlayerState {
    private weak var player: AudioPlayer?
    
    init(player: AudioPlayer) {
        self.player = player
    }
    
    func play() {
        print("Начинаем воспроизведение")
        player?.changeState(PlayingState(player: player!))
    }
    
    func pause() {
        print("Ошибка: еще не начали воспроизведение")
    }
    
    func stop() {
        print("Ошибка: еще не начали воспроизведение")
    }
}

class PlayingState: PlayerState {
    private weak var player: AudioPlayer?
    
    init(player: AudioPlayer) {
        self.player = player
    }
    
    func play() {
        print("Ошибка: уже воспроизводится")
    }
    
    func pause() {
        print("Ставим на паузу")
        player?.changeState(PausedState(player: player!))
    }
    
    func stop() {
        print("Останавливаем воспроизведение")
        player?.changeState(FinishedState(player: player!))
    }
}

// Аналогично реализуем PausedState и FinishedState...
```

#### **2. Использование**

```swift
let player = AudioPlayer()

player.play() // Начинаем воспроизведение
player.pause() // Ставим на паузу
player.play() // Возобновляем воспроизведение
player.stop() // Останавливаем

/* Вывод:
Начинаем воспроизведение
Ставим на паузу
Возобновляем воспроизведение
Останавливаем воспроизведение
*/
```

---

### **Пример 2: Состояния загрузки в UIViewController**

#### **Задача:**
Управлять состоянием экрана:
- `Loading` (загрузка)
- `Content` (контент)
- `Error` (ошибка)

---

#### **1. Реализация**

```swift
// Протокол состояния
protocol ViewState {
    func setupView()
}

// Контекст (ViewController)
class ProfileViewController: UIViewController {
    private var currentState: ViewState?
    
    func changeState(_ state: ViewState) {
        currentState = state
        state.setupView()
    }
}

// Конкретные состояния
struct LoadingState: ViewState {
    weak var controller: ProfileViewController?
    
    func setupView() {
        controller?.view.subviews.forEach { $0.removeFromSuperview() }
        let activity = UIActivityIndicatorView(style: .large)
        activity.startAnimating()
        controller?.view.addSubview(activity)
        // ...
    }
}

struct ContentState: ViewState {
    weak var controller: ProfileViewController?
    let user: User
    
    func setupView() {
        controller?.view.subviews.forEach { $0.removeFromSuperview() }
        let label = UILabel()
        label.text = user.name
        // ...
    }
}

struct ErrorState: ViewState {
    weak var controller: ProfileViewController?
    let error: Error
    
    func setupView() {
        controller?.view.subviews.forEach { $0.removeFromSuperview() }
        let label = UILabel()
        label.text = error.localizedDescription
        // ...
    }
}
```

#### **2. Использование**

```swift
class ProfileViewController: UIViewController {
    func loadData() {
        changeState(LoadingState(controller: self))
        
        API.loadProfile { [weak self] result in
            switch result {
            case .success(let user):
                self?.changeState(ContentState(controller: self!, user: user))
            case .failure(let error):
                self?.changeState(ErrorState(controller: self!, error: error))
            }
        }
    }
}
```

---

## **Где применяется в iOS?**

1. **UIViewController** жизненные циклы
2. **UIControl** состояния (selected, disabled)
3. **URLSessionTask** состояния (running, suspended, cancelled)
4. **SwiftUI** @State и @ObservedObject

---

## **Плюсы и минусы**

✅ **Плюсы:**
- Избавляет от больших switch/case
- Упрощает добавление новых состояний
- Делает код более читаемым

❌ **Минусы:**
- Может быть избыточным для простых случаев
- Требует создания множества классов

---

## **Сравнение с другими паттернами**

- **Strategy** - меняет алгоритмы, а **State** - поведение при изменении состояния
- **Observer** - уведомляет о изменениях, а **State** - меняет поведение
