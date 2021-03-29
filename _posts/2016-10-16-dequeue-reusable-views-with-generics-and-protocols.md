---
layout: post
title: "Dequeue Reusable Views With Generics and Protocols"
author: "Linh Vo"
tags: "UIKit"
---

A Swift mixin for reusing views easily and in a type-safe way (UITableViewCells, UICollectionViewCells, custom UIViews, ViewControllers, Storyboards…)

# NibLoadable protocol

```swift
protocol NibLoadable: class {
  static var nib: UINib { get }
}

extension NibLoadable {
    static var nib: UINib {
        return UINib(nibName: String(describing: self), bundle: Bundle(for: self))
    }

    static func loadFromNib() -> Self {
        guard let view = nib.instantiate(withOwner: nil, options: nil).first as? Self else {
            fatalError("The nib \(nib) expected its root view to be of type \(self)")
        }
        return view
    }
}
```

# NibOwnerLoadable protocol

```swift
protocol NibOwnerLoadable: class {
    static var nib: UINib { get }
}

extension NibOwnerLoadable {
    static var nib: UINib {
        return UINib(nibName: String(describing: self), bundle: Bundle(for: self))
    }
}

extension NibOwnerLoadable where Self: UIView {
    func loadNibContent() {
        let layoutAttributes: [NSLayoutConstraint.Attribute] = [.top, .leading, .bottom, .trailing]
        for case let view as UIView in type(of: self).nib.instantiate(withOwner: self, options: nil) {
            view.translatesAutoresizingMaskIntoConstraints = false
            self.addSubview(view)
            NSLayoutConstraint.activate(layoutAttributes.map { attribute in
                NSLayoutConstraint(
                    item: view, attribute: attribute,
                    relatedBy: .equal,
                    toItem: self, attribute: attribute,
                    multiplier: 1, constant: 0.0
                )
            })
        }
    }
}
```

# Reusable protocol

```swift
protocol Reusable: class {
  static var reuseIdentifier: String { get }
}

extension Reusable {
  static var reuseIdentifier: String {
    return String(describing: self)
  }
}
```

# UITableView extension

```swift
extension UITableView {

    func register<T: UITableViewCell>(cellType: T.Type) where T: Reusable & NibLoadable {
        self.register(cellType.nib, forCellReuseIdentifier: cellType.reuseIdentifier)
    }

    func register<T: UITableViewCell>(cellType: T.Type) where T: Reusable {
        self.register(cellType.self, forCellReuseIdentifier: cellType.reuseIdentifier)
    }

    func dequeueReusableCell<T: UITableViewCell>(for indexPath: IndexPath, cellType: T.Type = T.self) -> T where T: Reusable {
        guard let cell = self.dequeueReusableCell(withIdentifier: cellType.reuseIdentifier, for: indexPath) as? T else {
            fatalError(
                "Failed to dequeue a cell with identifier \(cellType.reuseIdentifier) matching type \(cellType.self). "
                    + "Check that the reuseIdentifier is set properly in your XIB/Storyboard "
                    + "and that you registered the cell beforehand"
            )
        }
        return cell
    }

    func register<T: UITableViewHeaderFooterView>(headerFooterViewType: T.Type) where T: Reusable & NibLoadable {
        self.register(headerFooterViewType.nib, forHeaderFooterViewReuseIdentifier: headerFooterViewType.reuseIdentifier)
    }

    func register<T: UITableViewHeaderFooterView>(headerFooterViewType: T.Type) where T: Reusable {
        self.register(headerFooterViewType.self, forHeaderFooterViewReuseIdentifier: headerFooterViewType.reuseIdentifier)
    }

    func dequeueReusableHeaderFooterView<T: UITableViewHeaderFooterView>(_ viewType: T.Type = T.self) -> T? where T: Reusable {
        guard let view = self.dequeueReusableHeaderFooterView(withIdentifier: viewType.reuseIdentifier) as? T? else {
            fatalError(
                "Failed to dequeue a header/footer with identifier \(viewType.reuseIdentifier) "
                    + "matching type \(viewType.self). "
                    + "Check that the reuseIdentifier is set properly in your XIB/Storyboard "
                    + "and that you registered the header/footer beforehand"
            )
        }
        return view
    }
}
```

# UICollectionView extension

```swift
extension UICollectionView {
    func register<T: UICollectionViewCell>(cellType: T.Type) where T: Reusable & NibLoadable {
        self.register(cellType.nib, forCellWithReuseIdentifier: cellType.reuseIdentifier)
    }

    func register<T: UICollectionViewCell>(cellType: T.Type) where T: Reusable {
        self.register(cellType.self, forCellWithReuseIdentifier: cellType.reuseIdentifier)
    }

    func dequeueReusableCell<T: UICollectionViewCell>(for indexPath: IndexPath, cellType: T.Type = T.self) -> T where T: Reusable {
        let bareCell = self.dequeueReusableCell(withReuseIdentifier: cellType.reuseIdentifier, for: indexPath)
        guard let cell = bareCell as? T else {
            fatalError(
                "Failed to dequeue a cell with identifier \(cellType.reuseIdentifier) matching type \(cellType.self). "
                    + "Check that the reuseIdentifier is set properly in your XIB/Storyboard "
                    + "and that you registered the cell beforehand"
            )
        }
        return cell
    }

    func register<T: UICollectionReusableView>(supplementaryViewType: T.Type, ofKind elementKind: String) where T: Reusable & NibLoadable {
        self.register(
            supplementaryViewType.nib,
            forSupplementaryViewOfKind: elementKind,
            withReuseIdentifier: supplementaryViewType.reuseIdentifier
        )
    }

    func register<T: UICollectionReusableView>(supplementaryViewType: T.Type, ofKind elementKind: String) where T: Reusable {
        self.register(
            supplementaryViewType.self,
            forSupplementaryViewOfKind: elementKind,
            withReuseIdentifier: supplementaryViewType.reuseIdentifier
        )
    }

    func dequeueReusableSupplementaryView<T: UICollectionReusableView>
    (ofKind elementKind: String, for indexPath: IndexPath, viewType: T.Type = T.self) -> T where T: Reusable {
        let view = self.dequeueReusableSupplementaryView(
            ofKind: elementKind,
            withReuseIdentifier: viewType.reuseIdentifier,
            for: indexPath
        )
        guard let typedView = view as? T else {
            fatalError(
                "Failed to dequeue a supplementary view with identifier \(viewType.reuseIdentifier) "
                    + "matching type \(viewType.self). "
                    + "Check that the reuseIdentifier is set properly in your XIB/Storyboard "
                    + "and that you registered the supplementary view beforehand"
            )
        }
        return typedView
    }
}
```

# Usage

## 1. Declare your cells to conform to Reusable or NibReusable

- Use the `Reusable` protocol if they don't depend on a NIB (this will use `registerClass(…)` to register the cell)

- Use the `NibReusable` typealias (= `Reusable & NibLoadable`) if they use a XIB file for their content (this will use `registerNib(…)` to register the cell)

```swift
class CustomCell: UITableViewCell, Reusable {
    /* And that's it! */
}
```

✍️ Notes

- For cells embedded in a Storyboard's tableView, either one of those two protocols will work (as you won't need to register the cell manually anyway, since registration is handled by the storyboard automatically)

- If you create a XIB-based cell, don't forget to set its Reuse Identifier field in Interface Builder to the same string as the name of the cell class itself.

- 💡 NibReusable is a typealias, so you could still use two protocols conformance Reusable, NibLoadable instead of NibReusable.

## 2. Register your cells

Unless you've prototyped your cell in a Storyboard, you'll have to register the cell class or Nib by code.

To do this, instead of calling `registerClass(…)` or `registerNib(…)` using a String-based `reuseIdentifier`, just call:

```swift
tableView.register(cellType: theCellClass.self)
```

## 3. Dequeue your cells

To dequeue a cell (typically in your `cellForRowAtIndexPath` implementation), simply call `dequeueReusableCell(indexPath:)`

```swift
// Either
let cell = tableView.dequeueReusableCell(for: indexPath) as MyCustomCell
// Or
let cell: MyCustomCell = tableView.dequeueReusableCell(for: indexPath)
```

As long as **Swift can use type-inference to understand that you'll want a cell of type `MyCustomCell`** (either using `as MyCustomCell` or explicitly typing the receiving variable `cell: MyCustomCell`), it will magically infer both the cell class to use and thus its `reuseIdentifier` needed to dequeue the cell, and which exact type to return to save you a type-cast.

- No need for you to manipulate `reuseIdentifiers` Strings manually anymore!

- No need to force-cast the returned `UITableViewCell` instance down to your `MyCustomCell` class either!

Now all you have is **a beautiful code and type-safe cells**, with compile-type checking, and no more String-based API!

# Type-safe XIB-based reusable views

Reusable also allows you to create reusable custom views designed in Interface Builder to reuse them in other XIBs or Storyboards, or by code. This allows you to treat those views like custom UI widgets that can be used in multiple places in your app.

## 1. Declare your views to conform to `NibLoadable` or `NibOwnerLoadable`

In your swift source declaring your custom view class:

- Use the `NibLoadable` protocol if the XIB you're using don't use its "File's Owner" and the reusable view you're designing is the root view of the XIB

- Use the `NibOwnerLoadable` protocol if you used a "File's Owner" of the XIB being of the class of your reusable view, and the root view(s) of the XIB is to be set as a subview providing its content.

```swift
// a XIB-based custom UIView, used as root of the XIB
final class NibBasedRootView: UIView, NibLoadable { /* and that's it! */ }

// a XIB-based custom UIView, used as the XIB's "File's Owner"
final class NibBasedFileOwnerView: UIView, NibOwnerLoadable { /* and that's it! */ }
```

> 💡 You should use the second approach if you plan to use your custom view in another XIB or Storyboard.
> This will allow you to just drop a UIView in a XIB/Storyboard and change its class in IB's inspector to the class of your custom XIB-based view to use it. That custom view will then automagically load its own content from the associated XIB when instantiated by the storyboard containing it, without having to write additional code to load the content of the custom view manually every time.

## 2. Design your view in Interface Builder

For example if you named your class `MyCustomWidget` and made it `NibOwnerLoadable`:

- Set the File's Owner's class to `MyCustomWidget`

- Design the content of the view via the root view of that XIB (which is a standard `UIView` with no custom class) and its subviews
- Connect any `@IBOutlets` and `@IBActions` between the File's Owner (the `MyCustomWidget`) and its content

```swift
final class MyCustomWidget: UIView, NibOwnerLoadable {
    @IBOutlet private var rectView: UIView!
    @IBOutlet private var textLabel: UILabel!

    @IBInspectable var rectColor: UIColor? {
        didSet {
        self.rectView.backgroundColor = self.rectColor
        }
    }
    @IBInspectable var text: String? {
        didSet {
        self.textLabel.text = self.text
        }
    }
}
```

Then that widget can be integrated in a Storyboard Scene (or any other XIB) by simply dropping a `UIView` on the Storyboard, and changing its class to `MyCustomWidget` in IB's inspector.

## 3a. Auto-loading the content of a `NibOwnerLoadable` view

If you used `NibOwnerLoadable` and made your custom view the File's Owner of your XIB, you should then override `init?(coder:)` so that it loads it's associated XIB as subviews and add constraints automatically:

```swift
final class MyCustomWidget: UIView, NibOwnerLoadable {
  …
  required init?(coder aDecoder: NSCoder) {
    super.init(coder: aDecoder)
    self.loadNibContent()
  }
}
```

`self.loadNibContent()` is a method provided by the NibOwnerLoadable mixin. It basically loads the content from the associated `MyCustomWidget.xib`, then add all the root views in that XIB as subviews of your `MyCustomWidget`, with appropriate layout constraints to make them the same size as your `MyCustomWidget` container view.

Overriding `init?(coder:)` and calling `self.loadNibContent()` thus allows you to have that content automatically loaded by the system when that `MyCustomWidget` in included in another XIB or in a Storyboard (as `init?(coder:)` is the `init` that is called by iOS to create those instances in a XIB or Storyboard)

> 💡 Note: it is also possible to override `init(frame:)` similarly, in order to be able to also create an instance of that view manually via code if needed.

## 3b. Instantiating a `NibLoadable` view

If you used `NibLoadable` and made your custom view the root view of your XIB (not using the File's Owner at all), these are not designed to be used in other Storyboards or XIBs like `NibOwnerLoadable` is, as they won't be able to auto-load their content.

Instead, you will instantiate those `NibLoadable` views by code, which is as simple as calling `loadFromNib()` on your custom class:

```swift
let view1 = NibBasedRootView.loadFromNib() // Create one instance
let view2 = NibBasedRootView.loadFromNib() // Create another one
let view3 = NibBasedRootView.loadFromNib() // and another one
```
