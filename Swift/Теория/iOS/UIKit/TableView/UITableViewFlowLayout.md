**`UITableViewFlowLayout`** — это **устаревший** и **не рекомендуемый** в 2025–2026 годах способ создания макета для [[UITableView]].

Начиная примерно с 2019–2020 годов (iOS 13+), Apple официально рекомендует **полностью отказаться** от `UITableViewFlowLayout` в пользу двух современных подходов:

| Подход                                                             | Когда использовать в 2025–2026 годах                                                                                      | Почему именно этот подход сейчас                         |
| ------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------- |
| **[[UICollectionView]] + [[UICollectionViewCompositionalLayout]]** | Почти во всех новых проектах, где нужна гибкость, адаптивность, разные размеры ячеек в одной секции, orthogonal scrolling | Самый мощный, современный и поддерживаемый инструмент    |
| **Обычный UITableView + self-sizing cells + estimatedRowHeight**   | Когда действительно нужна именно таблица (очень длинные списки, простая линейная структура, производительность критична)  | Самый простой и быстрый вариант для классических списков |

### Почему UITableViewFlowLayout считается устаревшим

| Проблема / ограничение                        | Последствия в 2025–2026 годах                                                                   |
| --------------------------------------------- | ----------------------------------------------------------------------------------------------- |
| Нет поддержки compositional layout            | Невозможно сделать горизонтальные секции, nested группы, кастомные layouts внутри одной таблицы |
| Сложно делать адаптивные размеры ячеек        | Требуется много ручного кода (estimatedHeight, systemLayoutSizeFitting и т.д.)                  |
| Нет orthogonal scrolling                      | Нельзя сделать горизонтальную прокрутку внутри вертикальной таблицы                             |
| Нет встроенной поддержки sticky headers       | Нужно писать кастомные решения или использовать viewForHeaderInSection с хакками                |
| Производительность хуже на сложных списках    | [[UICollectionViewCompositionalLayout]] лучше оптимизирован под современные устройства          |
| Apple активно продвигает compositional layout | Все WWDC 2020–2025 сессии по коллекциям и спискам показывают именно compositional подход        |

### Когда всё ещё можно встретить UITableViewFlowLayout в 2026 году

- Очень старые проекты (2015–2019 годы)
- Код, который поддерживает [[iOS]] 12 и ниже
- Супер-простые списки, где не нужна никакая кастомизация (однотипные ячейки, фиксированная высота)
- Команды, которые боятся миграции (очень редко, но бывает)

### Рекомендуемый современный путь в 2026 году

#### Вариант 1 — Самый рекомендуемый (почти всегда)

Перейти на **UICollectionView + UICollectionViewCompositionalLayout**

```swift
func createListLayout() -> UICollectionViewCompositionalLayout {
    let config = UICollectionLayoutListConfiguration(appearance: .insetGrouped)
    return UICollectionViewCompositionalLayout.list(using: config)
}

// или полноценный compositional layout
let layout = UICollectionViewCompositionalLayout { sectionIndex, environment in
    let itemSize = NSCollectionLayoutSize(
        widthDimension: .fractionalWidth(1.0),
        heightDimension: .estimated(44)
    )
    let item = NSCollectionLayoutItem(layoutSize: itemSize)
    
    let groupSize = NSCollectionLayoutSize(
        widthDimension: .fractionalWidth(1.0),
        heightDimension: .estimated(44)
    )
    let group = NSCollectionLayoutGroup.horizontal(layoutSize: groupSize, subitems: [item])
    
    let section = NSCollectionLayoutSection(group: group)
    section.interGroupSpacing = 0
    
    return section
}
```

#### Вариант 2 — Если очень хочется оставить [[UITableView]]

Используйте **self-sizing + estimatedRowHeight** (без FlowLayout)

```swift
tableView.rowHeight = UITableView.automaticDimension
tableView.estimatedRowHeight = 44  // или средняя высота вашей ячейки

// В ячейке обязательно используйте Auto Layout (constraints от contentView)
class ModernTableViewCell: UITableViewCell {
    // все subviews добавляются в contentView
    // constraints от contentView.leading/trailing/top/bottom
}
```

### Итог 2026 года

| Вопрос                                       | Ответ в 2025–2026 годах                                                                                |
| -------------------------------------------- | ------------------------------------------------------------------------------------------------------ |
| Можно ли использовать UITableViewFlowLayout? | Технически да, но **не рекомендуется**                                                                 |
| Что использовать вместо него?                | **UICollectionViewCompositionalLayout** (99% случаев)                                                  |
| Когда оставить UITableView?                  | Только очень простые списки + self-sizing + iOS 12 поддержка                                           |
| Что Apple показывает на WWDC?                | Только compositional layout, [[UITableViewDiffableDataSource\|DiffableDataSource]], List в [[SwiftUI]] |

**Короткий девиз 2026**:
> Если вы пишете новый код или обновляете проект в 2025–2026 годах и видите `UITableViewFlowLayout` —  
> **сразу заменяйте** на `UICollectionViewCompositionalLayout`.  
> Это **не модный тренд**, это **официальный современный стандарт** Apple для всех списков и коллекций.
