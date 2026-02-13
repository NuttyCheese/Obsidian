#Swift #uikit 
## 📘 Определение

**`Nib` (или `.xib`)** — это файл интерфейса в [[iOS]], который хранит **описание UI компонентов** ([[Swift/Теория/UIKit/UIView]], [[UIViewController]]) в сериализованном виде.  
Используется для **разделения визуального дизайна и кода**, позволяет создавать интерфейсы в Interface Builder и загружать их программно через `UINib`.  
Относится к **[[UIKit]] → UI**.

---

## 🔹 Примеры кода

### 1. Загрузка UIView из Nib

```swift
import UIKit

class CustomView: UIView {
    @IBOutlet weak var label: UILabel!
    
    required init?(coder: NSCoder) {
        super.init(coder: coder)
        commonInit()
    }
    
    override init(frame: CGRect) {
        super.init(frame: frame)
        commonInit()
    }
    
    private func commonInit() {
        let nib = UINib(nibName: "CustomView", bundle: nil)
        let view = nib.instantiate(withOwner: self, options: nil).first as! UIView
        view.frame = self.bounds
        addSubview(view)
    }
}
```

---

### 2. Регистрация Nib для [[UITableViewCell]]

```swift
let tableView = UITableView()
tableView.register(UINib(nibName: "CustomCell", bundle: nil), forCellReuseIdentifier: "CustomCell")

func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
    let cell = tableView.dequeueReusableCell(withIdentifier: "CustomCell", for: indexPath)
    return cell
}
```

---

### 3. Регистрация Nib для [[UICollectionViewCell]]

```swift
let collectionView = UICollectionView(frame: .zero, collectionViewLayout: UICollectionViewFlowLayout())
collectionView.register(UINib(nibName: "PhotoCell", bundle: nil), forCellWithReuseIdentifier: "PhotoCell")

func collectionView(_ collectionView: UICollectionView, cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {
    let cell = collectionView.dequeueReusableCell(withReuseIdentifier: "PhotoCell", for: indexPath)
    return cell
}
```

---

### 4. Загрузка UIViewController из Nib

```swift
let vc = MyViewController(nibName: "MyViewController", bundle: nil)
present(vc, animated: true)
```

---

### 5. Использование Nib с кастомной инициализацией

```swift
class BannerView: UIView {
    @IBOutlet private weak var titleLabel: UILabel!

    convenience init(title: String) {
        self.init(frame: .zero)
        let nib = UINib(nibName: "BannerView", bundle: nil)
        let view = nib.instantiate(withOwner: self, options: nil).first as! UIView
        view.frame = bounds
        addSubview(view)
        titleLabel.text = title
    }
}
```
