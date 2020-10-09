# beginner-ios-realm
Assets and overview for Twitch.tv/mongodb session from Oct 9, 2020

In this session, we'll walk through the basics of creating an iOS app with Xcode, Swift and Realm. 

## Requirements

- Xcode (11.6 or higher) - get this from the app store.
- Swift - will come with Xcode
- Cocoapods - `brew install cocoapods`
- Realm Studio (optional) [Download](https://docs.realm.io/sync/realm-studio)

iOS apps are relatively easy to create. It's persisting data that many app developers overlook. In this session, let's walk through the basics
of creating an iOS app and then we'll add Realm for data persistence.

## Part 1: Setup and Basic UITableView App Creation

![Getting Started](./images/beginner-ios-swift-realm-1a.gif)

1. Start Xcode
2. File, New Project
   1. Select iOS App
   2. Swift Language
   3. Save the project somewhere
3. Open Main.storyboard
4. Add TableView
   1. Drag tableview from object library to upper left corner
   2. Click add constraints and set the top, left, upper, and lower to 0… add constraints.
   3. Increment the number of prototype cells (to 1)
   4. Configure the prototype cell identifier
   5. Click on the prototype cell to highlight it.
   6. In the attributes inspector, enter “cell” into the reusable cell field.
   7. Change the Style of the Prototype cell to “Subtitle” so we can display the name and the telephone number below it.
5. Add an IBOutlet to your code so you can control your tableView from code.
   1. Open  the Assistant View
   2. Add an IBOutlet to enable you to reference the table in your code.
   3. Right-click somewhere in the tableview and drag and drop inside the ViewController Class after the class declaration.
   4. Name the tableView
   5. Add tableview.delegate = self and tableview.dataSource = self to the viewDidLoad function after super.viewDidLoad()
6. Extend the ViewController class so it can manage the UITableView
   1. Insert the following two methods at th bottom of the ViewController class after the closing brace for the VC class:

```
extension ViewController: UITableViewDelegate {
    
}
 
extension ViewController: UITableViewDataSource {
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return people.count
    }
    
    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let cell = tableView.dequeueReusableCell(withIdentifier: "cell", for: indexPath)
        cell.textLabel?.text = people[indexPath.row].name
        cell.detailTextLabel?.text = people[indexPath.row].telephone
        return cell
    }
    
   
}
```
7. Add a `ContactItem` Class - put the following class defining just before the ViewController class definition. This will provide a way for us
to define what the data in our app will look like... the attributes of each contact item.

```
class ContactItem {
    var name = ""
    var telephone = ""
    var id = ""
    convenience init(name: String, telephone: String, id: String) {
        self.init()
        self.name = name
        self.telephone = telephone
        self.id = id
    }
}
```

8. Create an instance of ContactItem.
   1. Place the following inside the ViewController after the IBOutlet
```
   var person = ContactItem(name: "Mike", telephone: "214-123-1233", id: "")
   var people = [ContactItem]()
```
   1. Then append the person instance to people - place the following inside the ViewDidLoad Method.
   ```
        people.append(person)
   ```

9. Run the app and you should see one person in the table.

## Part 2: Enter Realm

Convert this to a Realm App. Because this will take a view minutes to build after importing we will open up a pre-built version ContactRealm-after-realm

1. Close out the Xcode project and open a terminal window. Ensure that you are in the project top directory.

   1. Add cocoapods to the project.
pod init.
   2. Edit the Podfile to look like the following:

```
# Uncomment the next line to define a global platform for your project
platform :ios, '13.0'

target 'ContactRealm9' do
  # Comment the next line if you don't want to use dynamic frameworks
  use_frameworks!

  # Pods for ContactRealm9
  pod 'RealmSwift','10.0.0-beta.3'

end
```
   3. From command line type: pod install

   This will create an xcworkspace file. From command line type open ./ContactRealm.xcworkspace.
   Open the ViewController.swift file and add the following import line below the import UIKit statement to bring in RealmSwift.

   ```
   import RealmSwift
   ```

   Press CMD+B to build the project - this will take several minutes.
   Once Realm has been added, we can modify our class to become a realm Object.

2. Convert the `ContactItem` class to a Realm Object. We do this using the Object prototype. Notice we replace the id default settings - removing it from the `convenience init`. This is because we're going to provide a real, generated _id - which will not necessarily be needed for this simple implementation. However, it will be needed when we convert this app to a sync'd app. This is because we'll sync our data to a MongoDB backend and MongoDB requires a unique _id for each document.

```
class ContactItem: Object {
    @objc dynamic var name = ""
    @objc dynamic var telephone = ""
    @objc dynamic var _id: ObjectId = ObjectId.generate()
    convenience init(name: String, telephone: String, id: String) {
        self.init()
        self.name = name
        self.telephone = telephone
    }
    override static func primaryKey() -> String? {
        return "_id"
    }
    
}
```

3. Convert the standard array storage mechanism (`people`) to become a Realm Query. Here's what your `ViewController` class should look like:

```
class ViewController: UIViewController {

    @IBOutlet weak var tableView: UITableView!
    
    var person = ContactItem(name: "Mike", telephone: "214-123-1233", id: "")

//    var people = [ContactItem]()
    var people = try! Realm().objects(ContactItem.self).sorted(byKeyPath: "name", ascending: true)
    let realm = try! Realm()

    override func viewDidLoad() {
        super.viewDidLoad()
        let path = realm.configuration.fileURL?.path
        print("Realm Path: \(String(describing: path))")
        tableView.delegate = self
        tableView.dataSource = self
        if realm.isEmpty {
            try! realm.write {
                realm.add(person)
            }
        }
    }
}
```

4. Build and run the app. You've just created a realm. To view the realm, I like to use RealmStudio. You'll need to obtain the path to the Realm file that gets created on your device. Since we're using the simulater, it's actually just creating a file on our local machine.  You can add the following lines to your `viewDidLoad()` class in order to get a debug printout of the path.

```
        let path = realm.configuration.fileURL?.path
        print("Realm Path: \(String(describing: path))")
```

This will print out (in Xcode debug console) something like the following when you run the app.

```
Realm Path: Optional("/Users/mlynn/Library/Developer/CoreSimulator/Devices/EF55DC4F-CC78-4318-8378-B1025C434A3D/data/Containers/Data/Application/832427F8-4ACA-4365-AF22-BFD6866C4C6A/Documents/default.realm")

```

5. View the realm in Realm Studio.
   1. Copy the path from the debug statement and copy it to your local directory. This is necessary because your app will still have a lock on the file. 

   ```
   cp /Users/mlynn/Library/Developer/CoreSimulator/...YOUR ACTUAL PATH.../default.realm ./default.realm
   open ./default.realm.

   You should now see RealmStudio opening up - and displaying your object.

## Part 3: Add a popup to accept new contacts.

Now that our app is leveraging realm we need a way to add Contacts to the app. We will add a navigation bar and a button which will open a dialog and prompt for additional contact details.

![Add Navigation Bar](./images/beginner-ios-swift-realm-2.gif)

1. Click the tableView and drag the top size indicator downward to make room for a navigation bar.
2. With the `main.storyboard` open, click to open the object library and drag a `Navigation Bar` item to the top of the view, positioning it against the top margin.
3. Now drag a `Button Bar Item` to the `Navigation Bar` dropping it in the right-most side of the bar. Add an image to the button bar by selecting the image field in the Attributes inspector. Select the + image.
4. Resize the `tableView` so that it touches the bottom of the `Navigation Bar`.
5. Click in the `Resolve Auto Layout Issues` button at the bottom of the Xcode screen and select `Reset to Suggested Constraints`.
6. Open the Assistant View so that you can see the Main Storyboard as well as the `ViewController.swift` file.
7. Right-click and drag from the + image in the navigation bar and drop it in the `ViewController` class below the closing brace of the `viewDidLoad()` function. Select `Action` and name it `addButtonTapped`. This function will be called when a user interacts with the plus button to add a contact.
8. Add the following code to the function you just created. This will create a dialog box and prompt the user for the contact details to be added.

```
@IBAction func addButtonTapped(_ sender: Any) {

   let alertController = UIAlertController(title: "Add Contact", message: "", preferredStyle: .alert)

   // When the user clicks the add button, present them with a dialog to enter the task name.
   alertController.addAction(UIAlertAction(title: "Save", style: .default, handler: {
      alert -> Void in
      let nameField = alertController.textFields![0] as UITextField
      let teleField = alertController.textFields![1] as UITextField

      // Create a new Task with the text that the user entered.
      let newPerson = ContactItem(name: nameField.text!, telephone: teleField.text!)

      // Any writes to the Realm must occur in a write block.
      try! self.realm.write {
            // Add the Task to the Realm. That's it!
            self.realm.add(newPerson)
            self.tableView.reloadData()

      }
   }))
   alertController.addAction(UIAlertAction(title: "Cancel", style: .cancel, handler: nil))
   alertController.addTextField(configurationHandler: { (nameField: UITextField!) -> Void in
      nameField.placeholder = "New Contact Name"
   })
   alertController.addTextField(configurationHandler: { (teleField: UITextField!) -> Void in
      teleField.placeholder = "+1 (111) 111-1111"
   })

   // Show the dialog.
   self.present(alertController, animated: true)
}
```

9. Build, and run the app. You should now be able to add contacts to the table, thereby adding them to the realm.
