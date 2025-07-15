# iOS-Interview-Q-A
Contains interview question &amp; Answers on Objective C, swift &amp; Swift UI

## Question 1: Explain iOS Property Attributes  

This table provides a concise explanation of common property attributes used in iOS development (Objective-C), including their default values.

| Attribute     | One-Line Explanation                                                                 | Default Value               |
|---------------|----------------------------------------------------------------------------------------|-----------------------------|
| `strong`      | Keeps a strong reference to the object, increasing its retain count.                  | ✅ Yes (for object types)   |
| `weak`        | Holds a non-owning reference that becomes `nil` when the object is deallocated.       | ❌ No                       |
| `copy`        | Creates a copy of the object instead of referencing the original (used for immutability). | ❌ No                   |
| `assign`      | Assigns the value directly without retaining it (used for primitives).                | ✅ Yes (for non-objects)    |
| `nonatomic`   | Access to the property is not thread-safe but faster.                                 | ✅ Yes                      |
| `atomic`      | Ensures thread-safe access to the property (slower).                                 | ❌ No                       |
| `readwrite`   | Allows both getter and setter methods.                                                | ✅ Yes                      |
| `readonly`    | Only allows a getter; no setter is generated.                                         | ❌ No                       |


## Question 2: Difference between strong & retain?  

- **strong** - Swift (modern syntax)
- **retain** - Objective C (older terminology)
- 
  ```swift
  var person: Person // (strong by defalult)  
  ```
  
  ```Objective C
  @property(retain) Person *person;
  ```
  
## Question 3: Explain what is the difference between Weak & Unowned?  

Ans:  
| Feature           |  weak                                   | unowned                                       |
|------------------|-----------------------------------------|-----------------------------------------------|
| **Ownership**    | Does **not** increase reference count   | Does **not** increase reference count         |
| **Optionality**  | Must be declared as **optional (?)**    | Declared as **non-optional**                  |
| **Nil after dealloc**    | Becomes **nil** when object is deallocated  | **Crashes** if accessed after deallocation        |
| **Use Case**  |When the reference can **become nil**  | When the reference should **never be nil**                 |
| **Safety**  |Safe (ARC sets it to nil)  | Unsafe (can lead to runtime crash)                |  

**✅ When to Use Which?**  

**Use weak when:**  

  You expect the referenced object might be deallocated.  
  **Example**: Delegates (delegate pattern), closures capturing self.  

**Use unowned when**:  

  You know the referenced object will always exist as long as the referencing object exists.  
  Example: In closures where self is guaranteed to outlive the closure.  

  **🧠 First, What Happens in a Network Call?**  
  
      When you make a network request (e.g., using URLSession or a custom API manager), you often pass a closure to handle the response.  
      That closure might capture self (usually a ViewController) to update the UI once data is received.  

**⚠️ The Problem: Retain Cycle**  

    If you capture self strongly inside the closure, and the closure is retained by the network manager, you create a retain cycle:  
    

**ViewController** → **retains** → **NetworkManager**  
**NetworkManager** → **retains** → **closure**  
**closure** → **retains** → **ViewController**  
This means the ViewController won’t be deallocated, even if the user navigates away  


**✅ Scenario 1: Using weak self — When View Can Be Deallocated**  

**🧠 Example: Network Call in a ViewController**  
```swift
class ProfileViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
        fetchProfile()
    }

    func fetchProfile() {
        NetworkManager.shared.getProfile { [weak self] profile in
            self?.updateUI(with: profile)
        }
    }

    func updateUI(with profile: Profile) {
        // update labels, images, etc.
    }
}
```
**🔍 What Happens Here?**  

      User opens ProfileViewController.  
      A network call starts.  
      Before the network call finishes, the user navigates away (e.g., presses back).  
      ProfileViewController is deallocated.  
      The closure still runs, but self is now nil.  
      Because we used [weak self], it safely does nothing.  

**✅ Use weak when:**  

      The closure might run after the view is gone.  
      You want to avoid crashes and handle self being nil.  

**✅ Scenario 2: Using unowned self — When View Will Not Be Deallocated**  

**🧠 Example: Animation or Short-lived Closure**  
```swift
class WelcomeViewController: UIViewController {
    override func viewDidAppear(_ animated: Bool) {
        super.viewDidAppear(animated)
        animateLogo()
    }

    func animateLogo() {
        UIView.animate(withDuration: 1.0) { [unowned self] in
            self.logo.alpha = 1.0
        }
    }
}
```
**🔍 What Happens Here?**  

    The animation runs immediately after the view appears.  
    The view is guaranteed to be alive during the animation.  
    self will not be deallocated before the closure runs.  

**✅ Use unowned when:**  

    The closure runs immediately or very soon.  
    You're sure self will still exist.  
    You want to avoid optional unwrapping (self?).  

## Question 4: Explain why swift is called protocol oriented language?  
  - They are not limited to **classes**, **structs** & **enums** can conform too.
  - Protocol extenstion allow default behaviour
  - Encourgages **composition** over inheritance.
    
    **Example**:
    ```swift
    protocol Eater {
    func eat()
    }
    
    protocol Barker {
    func bark()
    }
    
    struct Dog: Eater, Barker {
    
    func eat() {
    print("Eating")
    }
    
    func bark() {
    print("Barking")
      }
  }
    ```  
## Question 5: Explain Core Data DELETE Rules?  

    |-> No Action  
    |-> Nullify  
    |-> Cascade - (When deletes source object, it also deletes any object in the relationship)  
    |-> Deny - (Prevents deletion, if there are any related objects)  

    
## Question 6: What is an Enum with Associated Type?  

- Sometimes, you want each case to carry extra information. That’s where associated types come in.
  
- **Enum**: A list of named values.
- **Associated** type: Extra data attached to each value.
- **Why use i**t: To handle different types of data in a clean and organized way.
  

  Imagine you’re building a media app. You want to represent different types of media:
 ```swift  
  
   enum Media {
    case photo(name: String)
    case video(name: String, duration: Int)
    case text(message: String)
}

func show(media: Media) {
    switch media {
    case .photo(let name):
        print("Photo: \(name)")
    case .video(let name, let duration):
        print("Video: \(name), Duration: \(duration) seconds")
    case .text(let message):
        print("Text: \(message)")
    }
}
```
## Question 6: What is defer in swift?
- In Swift, **defer** is a keyword used to schedule code to **run later**, specifically just before the current scope (like a function or loop) exits.

**Think of defer like saying:**  

"Do this last, no matter what happens before."  
 Even if your function returns early or throws an error, the code inside defer will still run before the function finishe  
 ```swift
func readFile() {
    print("Opening file")

    defer {
        print("Closing file")
    }

    print("Reading file")
}
```

**Output**  
Opening file  
Reading file  
Closing file  

Even though defer is written in the middle, it runs last.  

**Why Use defer?**
To clean up resources (like closing files, releasing memory, stopping timers).  
To make sure something always happens, even if there's an error or early return.  

**Multiple defer Blocks**  
If you use more than one defer, they run in reverse order (like a stack):  

```swift
func test() {
    defer { print("First") }
    defer { print("Second") }
    defer { print("Third") }
}
```

**Output**  
Third  
Second  
First  

**Example**  

```swift
func readFileContents(path: String) {
    let file = openFile(path) // pretend this opens a file
    print("File opened")

    defer {
        closeFile(file) // this will always run before the function ends
        print("File closed")
    }

    // Simulate reading
    guard let contents = try? read(file) else {
        print("Failed to read file")
        return
    }

    print("File contents: \(contents)")
}

```

## Question 7: Expalain Process, Task & Thread?  

**Summary Table**

		
| Concept     | Description  |iOS Example|
|---------------|------------|-----------|
|**Process**|Running instance of an app|Each app runs in its own process |
|**Thread**|Unit of execution within a process| Main thread for UI, background threads for tasks |
|**Task**|Unit of work scheduled on a thread | GCD block, Swift Task, URLSessionTask|  

**Example of App**

| Step     | What it is  |
|---------------|------------|
|**App running**|Process|
|**API call**|Task (runs on a background thread)|
|**Updating UI**|Task (runs on the main thread)|
|**Main thread**|Thread (dedicated to UI updates)|




