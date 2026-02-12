#swift #ios #struct #class #stack #heap #value_type #reference_type #memory_management #object_lifecycle
# Когда [[struct]] ведёт себя как [[reference type]]

**Короткий ответ:**  
**Никогда** семантически (struct всегда [[value type]]),  
но **может казаться** так из-за вложенных reference types или COW.

### 1. Struct + [[class]] внутри (самый частый случай)

```swift
struct Box {
    var obj: MyClass   // ← reference type внутри
}

var a = Box(obj: MyClass())
var b = a             // копия Box
b.obj.value = 100

print(a.obj.value)    // 100 — изменилось!
```

**Почему кажется reference?**  
`Box` скопирован, но `MyClass` — **один и тот же** объект в куче.

**Правда:**  
Это **не** reference semantics структуры,  
а **reference semantics** вложенного `class`.

### 2. [[Copy-on-Write]] коллекции ([[Array]], [[Dictionary]], [[String]], [[Set]])

```swift
var a = [1, 2, 3]
var b = a
b.append(4)

print(a)  // [1, 2, 3]
print(b)  // [1, 2, 3, 4]
```

**Почему кажется reference?**  
До мутации — общий буфер в куче.

**Правда:**  
Это **value semantics с ленивым копированием** (COW).  
При мутации создаётся копия — классический value type.

### 3. Struct в замыкании (escape → [[heap]])

```swift
var counter = 0
let inc = {
    counter += 1   // counter "поднят" в кучу
}
inc()
inc()
print(counter)     // 2
```

**Почему кажется reference?**  
Переменная `counter` живёт дольше функции → heap.

**Правда:**  
Это **escape analysis**, а не reference semantics.  
Struct всё равно копируется при захвате.

### 4. Struct внутри class — [[inline]]

```swift
class Container {
    var point = Point(x: 0, y: 0)   // struct inline внутри объекта
}
```

**Правда:**  
`Point` **не** отдельный объект в куче,  
а **встроен** в память `Container` → value semantics.

### Короткое резюме

| Ситуация                                 | Кажется reference? | На самом деле                         |
| ---------------------------------------- | ------------------ | ------------------------------------- |
| Struct + class внутри                    | Да                 | reference semantics от class          |
| COW-коллекции (Array, [[String]] и т.д.) | Да (до мутации)    | value semantics + ленивое копирование |
| Struct в замыкании (escape)              | Да                 | escape analysis → heap                |
| Struct внутри class                      | Нет                | inline storage                        |
| Чистый struct без reference внутри       | Нет                | чистый value type                     |

**Главное правило 2026**  
> Struct **никогда** не становится reference type.  
> Если кажется, что он ведёт себя как reference — внутри почти всегда спрятано `class`, замыкание или COW-буфер.
