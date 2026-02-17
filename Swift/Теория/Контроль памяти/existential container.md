В Swift протоколы позволяют определять "контракты" для типов: что они должны уметь делать. Но когда ты используешь протокол как тип (например, переменная типа `ProtocolName` или массив `[ProtocolName]`), [[Swift]] не знает заранее, какой конкретный тип будет храниться — это может быть struct, class или enum, который conforms (соответствует) протоколу. Чтобы справиться с этим, Swift использует **existential container** (экзистенциальный контейнер). Это специальная внутренняя структура в памяти, которая "упаковывает" значение любого типа, соответствующего протоколу, вместе с необходимой информацией для работы с ним.

Простыми словами: existential container — это "коробка" фиксированного размера, в которую Swift кладёт объект, conforming протоколу, плюс "инструкции" по его использованию. Это позволяет хранить разные типы в одном месте (например, в массиве), не зная их заранее. Без этого ты не смог бы создать массив с разными объектами, которые просто реализуют один протокол.

Представь: у тебя протокол `Animal`, и типы `Dog` и `Cat`, которые его реализуют. Ты можешь создать массив `let animals: [any Animal] = [Dog(), Cat()]`. Под капотом каждый элемент — это existential container.

## Почему нужны Existential Containers?

Swift — статически типизированный язык, то есть компилятор хочет знать типы на этапе компиляции. Но протоколы дают динамику: тип может решаться в [[Runtime]] (во время выполнения). Например:

- Ты хочешь коллекцию объектов разных типов, но все они должны уметь `makeSound()`.
- Или функцию, которая принимает любой объект, conforming протоколу.

Без existential containers это было бы невозможно, потому что разные типы имеют разные размеры в памяти. Массив требует, чтобы все элементы были одного размера для эффективного доступа. Existential container решает это, делая все "коробки" одинакового размера.

В старых версиях Swift (до 5.6) ты просто писал `let value: ProtocolName`, и это подразумевало existential. Теперь (с Swift 5.6+) нужно явно писать `any ProtocolName`, чтобы подчеркнуть, что это existential type с потенциальными overhead'ами.

## Структура Existential Container

Теперь разберём, как это выглядит в памяти. На 64-битной системе (x64, как на большинстве Mac и iPhone) existential container занимает **5 машинных слов** (machine words), то есть 40 байт (5 * 8 байт). Он состоит из трёх частей:

1. **Value Buffer (Буфер значения)** — 3 слова (24 байта).
   - Здесь хранится само значение (инстанс типа, conforming протоколу).
   - Если значение маленькое (≤ 3 слов, то есть ≤ 24 байт), оно кладётся прямо в буфер. Это эффективно — нет аллокации на [[Heap]] (куче).
   - Если значение больше (например, [[struct]] с многими полями), то оно аллоцируется на heap, а в буфер кладётся только указатель на него.
   - Для классов всегда используется указатель, потому что классы — [[Reference Type]].

2. **Value Witness Table (VWT, Таблица свидетелей значения)** — 1 слово (указатель на таблицу).
   - Это "инструкции" по управлению жизненным циклом значения: как аллоцировать, копировать, уничтожать и деаллоцировать.
   - VWT уникальна для каждого конкретного типа. Она содержит 4 метода:
     - `allocate`: Решает, где хранить ([[Stack]] или heap).
     - `copy`: Копирует содержимое (важно для [[Value Type]] как struct).
     - `destruct`: Уменьшает счётчик ссылок (для [[ARC]]) и чистит.
     - `deallocate`: Освобождает память.
   - Благодаря VWT Swift может работать с любым типом, не зная его статически.

3. **Protocol Witness Table (PWT, Таблица свидетелей протокола)** — 1 слово (указатель на таблицу).
   - Здесь указатели на реальные реализации методов протокола для этого типа.
   - Например, если протокол требует `func doSomething()`, PWT скажет, где лежит код этой функции для конкретного типа.

Итого: контейнер всегда 40 байт, независимо от размера значения внутри. Это позволяет хранить их в массивах contiguous (непрерывно).

Пример: Если struct conforming протоколу занимает 12 байт (меньше 24), он влезет в буфер. Если 32 байта — на heap.

## Как это работает на примерах

Давай возьмём простой протокол и типы.

```swift
protocol Human {
    var name: String { get }
    func introduce()
}

struct Teacher: Human {
    let name: String
    let subject: String
    let yearsOfExperience: Int
    
    func introduce() {
        print("Я учитель \(name), преподаю \(subject) уже \(yearsOfExperience) лет.")
    }
}

struct Student: Human {
    let name: String
    let grade: Int
    let favoriteSubject: String
    
    func introduce() {
        print("Я студент \(name), в \(grade) классе, любимый предмет — \(favoriteSubject).")
    }
}

let people: [any Human] = [Teacher(name: "Алексей", subject: "Математика", yearsOfExperience: 10), Student(name: "Мария", grade: 9, favoriteSubject: "Литература")]

for person in people {
    person.introduce()
}
```

- Здесь `people` — массив existential containers.
- Для `Teacher` (предположим, размер ~24 байта или меньше) значение в буфере.
- Swift при вызове `introduce()` смотрит в PWT, находит нужную реализацию и вызывает.
- Если присвоить `var dynamicPerson: any Human = Teacher(...)`, потом `dynamicPerson = Student(...)` — контейнер перезапишется, VWT и PWT обновятся.

Это runtime polymorphism: тип решается во время выполнения.

## Existential Any: Синтаксис и когда использовать

С Swift 5.6 ввели ключевое слово `any` для explicit обозначения existential types. Раньше просто `ProtocolName` подразумевало existential, но теперь (и обязательно в Swift 6) нужно писать `any ProtocolName`.

Пример:

```swift
let content: any Equatable = 42  // Может быть Int, String и т.д., conforming Equatable
```

### Associated Types

Протоколы с associated types (например, `Collection<Element>`) тоже могут быть existential, но с ограничениями.

С Swift 5.7 добавили Primary Associated Types (PAT) для конкретизации:

```swift
protocol ImageFetching<Image> {
    associatedtype Image
    func fetchImage() -> Image
}

func useFetcher(_ fetcher: any ImageFetching<UIImage>) {  // Только с Image == UIImage
    let image = fetcher.fetchImage()
}
```

### Когда использовать any (existential)?

- Когда нужен [[Runtime]] выбор типа: например, фабрика, возвращающая разные типы в зависимости от условий.
- Для хранения heterogeneous коллекций.
- Если тип не известен на compile-time.

Но! Предпочитай:

1. Конкретные типы (лучшая производительность).
2. Opaque types (`some Protocol`): Когда тип известен, но скрыт (compile-time polymorphism, нет overhead).
3. Только потом [[any]].

Пример фабрики:

```swift
struct RemoteFetcher: ImageFetching { /* ... */ }
struct LocalFetcher: ImageFetching { /* ... */ }

func makeFetcher(for url: URL) -> any ImageFetching<UIImage> {
    if url.isFileURL {
        return LocalFetcher(url: url)
    } else {
        return RemoteFetcher(url: url)
    }
}
```

Здесь `any` нужно, потому что возвращаемые типы разные.

Если всегда один тип — используй `some`:

```swift
func makeFetcher(for url: URL) -> some ImageFetching<UIImage> {
    return RemoteFetcher(url: url)  // Всегда Remote, компилятор знает
}
```

## Performance Implications (Влияние на производительность)

Existential containers не бесплатны:

- **Динамическая аллокация**: Для больших значений — heap allocation.
- **Indirection (Опосредованный доступ)**: Доступ к методам через PWT (динамический dispatch, медленнее статического).
- **Копирование**: При присваивании весь контейнер копируется, плюс вызов copy из VWT.
- **Нельзя оптимизировать**: Компилятор не может inline или оптимизировать, как с generics.

Сравни с generics: `func process<T: Human>(t: T)` — статически, быстро, но не для heterogeneous.

Или `some`: Оpaque, компилятор знает тип, но скрывает его.

В Swift 6 `any` будет обязательно везде для existential, чтобы разработчики видели overhead и избегали его где возможно.

Совет: Используй existential только когда действительно нужна динамика. В остальных случаях — generics или some.
