footer: © Feihong Hsu, 2015
slidenumbers: true

# Using Realm with Swift

## CocoaHeads

### March 10, 2015

### Feihong Hsu

### github.com/feihong


---

# What is Realm?

Realm is a mobile database for iOS and Android. On iOS, it's meant to be a replacement for Core Data, offering a similar API but with faster performance and a smaller footprint.

![inline](realm-icon.png)

---

# Installing in your project

Three methods:

- CocoaPods
- Carthage
- Manual installation (supports iOS 8 dynamic frameworks)

---

# Installing to a Swift project using CocoaPods

In your Podfile, you should force the Realm pod to be installed as a framework:

```ruby
platform :ios, '8.0'
use_frameworks!
pod 'Realm'
```

^ Source: https://groups.google.com/forum/#!topic/realm-cocoa/_fpQzHw6bxA

---

# Installing to a Swift project using CocoaPods (2)

If you don't add `use_frameworks!`, you must create a bridging header for Realm.

You also need to add the `RLMSupport.swift` file into your project. This file contains class extensions for RLMObject, RLMArray, and RLMResults.

^ The `RLMSupport.swift` file cannot be found in the Pods directory. It can be dragged into your project from the `Swift` directory of the Realm download, or you can get it directly from [Github](http://github.com/realm/realm-cocoa/Swift/RLMSupport.swift).

---

## Realm has a snazzy database browser

![fit](realm-browser.png)

^ You can use it to inspect and edit your database outside of your app.

---

# Creating a model

Just subclass RLMObject:

```swift
class Product: RLMObject {
    dynamic var name = ""
    dynamic var price = 0.0
    dynamic var rating = -1
    dynamic var startDate = NSDate()
}
```

---

# Gotcha: Custom initializers for RLMObject subclasses

You must first add a bunch of annoying boilerplate code:

```swift
override init() {
    super.init()
}
override init(object: AnyObject!) {
    super.init(object:object)
};
override init(object: AnyObject!, schema: RLMSchema!) {
    super.init(object: object, schema: schema)
}
override init(objectSchema: RLMObjectSchema) {
    super.init(objectSchema: objectSchema)
}
```

^ Source: https://github.com/realm/realm-cocoa/issues/1101

---

## Adding objects to the database

```swift
Product.createInRealm(realm, withObject: 
    ["Sais", 44.87, 5, NSDate()])

Product.createInDefaultRealmWithObject(
    ["name": "Sais", "price": 44.87, "rating": 5, "startDate": NSDate()])

realm.addObject(Product(
    name: "Sais", price: 44.87, rating: 5, startDate: "2013-02-28"))
```

^ If you use an Array to add an object, take care to order the values according to the order in which the corresponding properties were defined.

---

# All writes to the database must be inside a transaction

```swift
realm.beginWriteTransaction()
// write some data
realm.commitWriteTransaction()
```

```swift
realm.transactionBlock {
    // write some data
}
```

---

## Gotcha: one-liners in [RLMRealm transactionBlock:]

You might need to add a return statement inside the closure:

```swift
RLMRealm.defaultRealm().transactionWithBlock {
    Product.createInDefaultRealmWithObject(
        [name, 0.0, 3, NSDate()])
    return
}
```

^ In this case, it's because `[RLMRealm transactionBlock:]` expects the closure to return `Void`, but `[Product createInDefaultRealmWithObject:]` instead returns `Product!`.

^ It's also possible to solve this problem by changing the line to `let _ = Product.createInDefaultRealmWithObject([name, 0.0, 3, NSDate()])`, but I don't think that's as clean.

---

## Subscribing to database changes

```
let realm = RLMRealm.defaultRealm()
self.notificationToken = realm.addNotificationBlock {
    notification, realm in
    self.tableView.reloadData()
}
```

---

# Notifications

- There's no way to be notified of a particular type of database write.
- The notification is automatically deleted whenever the notification token goes out of scope. For this reason, you'll often want to make the notification token object a property of your view controller.

^ From the Realm website: "[It] is not currently possible to determine what was added/removed/moved/updated... We will be adding this feature in the near future."

---

# Gotcha: retain cyles in [RLMRealm addNotificationBlock:]

```swift
let realm = RLMRealm.defaultRealm()
self.notificationToken = realm.addNotificationBlock {
    [unowned self]      // avoid retain cycle
    notification, realm in
    self.tableView.reloadData()
}
```

^ In this case, the view controller (self) owns the notification token, which owns the closure, which would otherwise own self if you hadn't marked it as unowned.

---

# Pros

- Free, even for commercial projects
- Simpler API than Core Data
- Open source (partly)
- Nice database browser

^ The core C++ storage engine will also be open sourced in future.

---

# Cons

- Still beta (currently at version 0.90.5)
- Swift API is a work-in-progress
- No fine-grained notifications (also no support for KVO)
- NSDate is truncated to the second

---

# HelloRealm demo app

http://github.com/feihong/HelloRealm

---

![fit](HelloRealm.png)
![fit](HelloRealm-add.png)
![fit](HelloRealm-delete.png)
![fit](HelloRealm-dupe.png)

^ Screenshots of the app.

---

# Conclusion

You should give Realm a try!

# http://realm.io 

![left, inline](realm-icon.png) 

