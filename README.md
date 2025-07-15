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
    
