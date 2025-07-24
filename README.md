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

## Question 8: Expalain Operation?
- Operation is Abstract class, designed for sub classing. Each sub task represents a specific task.
- main() is the method you override in operation subclasses to actually perform work.

**✅ Example: Custom WeatherFetchOperation**  

Let’s say you want to fetch weather data from an API. You can create a custom Operation subclass like this:  

```swift
import Foundation

class WeatherFetchOperation: Operation {
    private let url: URL
    private var task: URLSessionDataTask?

    init(url: URL) {
        self.url = url
    }

    override func main() {
        if isCancelled { return }

        let semaphore = DispatchSemaphore(value: 0)

        task = URLSession.shared.dataTask(with: url) { data, response, error in
            defer { semaphore.signal() }

            if self.isCancelled { return }

            if let data = data {
                print("✅ Weather data received: \(data.count) bytes")
                // You can parse JSON here or pass it to a delegate/closure
            } else if let error = error {
                print("❌ Error fetching weather: \(error)")
            }
        }

        task?.resume()
        semaphore.wait() // Wait for the task to complete before finishing the operation
    }

    override func cancel() {
        super.cancel()
        task?.cancel()
    }
}

```
**🧪 How to Use It**  

```swift
let weatherURL = URL(string: "https://api.weatherapi.com/v1/current.json?key=YOUR_KEY&q=London")!
let weatherOperation = WeatherFetchOperation(url: weatherURL)

let queue = OperationQueue()
queue.addOperation(weatherOperation)

```
**🧠 Why Use a Custom Operation?**  

You can **cancel** the operation cleanly.  
You can **track** its state (executing, finished).  
You can add **dependencies** between operations.  
You can **reuse** it in different parts of your app.  

**⚙️ How Does It Work?**  

- **Create an Operation**
      Use BlockOperation or subclass Operation.
- **Add it to an OperationQueue**
      The queue manages threads and runs the operation.
- **(Optional) Set dependencies or priorities**
      You can control the order and importance of tasks.

## Question 9: How to create test cases for UIViewcontroller life cycle methods?  

Testing UIViewController lifecycle methods directly can be tricky because they are tightly coupled with the UIKit framework and triggered by the system.  
However, you can still write meaningful tests by focusing on observable side effects and using test doubles or manual invocation.  

**✅ Approach to Test UIViewController Lifecycle Methods**  

**🔹 1. Override Lifecycle Methods**
Add logging or flags to detect method calls.  
```swift
class MyViewController: UIViewController {
    var didCallViewDidLoad = false

    override func viewDidLoad() {
        super.viewDidLoad()
        didCallViewDidLoad = true
    }
}
```
**🔹 2. Write Unit Test Using XCTest**  
```swift
func testViewDidLoadIsCalled() {
    let vc = MyViewController()
    _ = vc.view  // Triggers viewDidLoad
    XCTAssertTrue(vc.didCallViewDidLoad)
}
```
**🔹 3. Test Side Effects**  
Instead of testing the method directly, test what the method does (e.g., setting up views, data, observers).  
```swift
func testViewDidLoadSetsUpLabel() {
    let vc = MyViewController()
    _ = vc.view
    XCTAssertEqual(vc.titleLabel.text, "Welcome")
}
```
**🔹 4. Use loadViewIfNeeded()**  
This ensures the view is loaded and lifecycle methods are triggered.  
```swift
vc.loadViewIfNeeded()
```
**🔹 5. Test viewWillAppear, viewDidAppear, etc.**  

You can manually call these methods in tests to simulate behavior.  
```swift
vc.viewWillAppear(false)
XCTAssertTrue(vc.didSetupBindings)
```
# UIViewController Lifecycle Testing Guide

This table summarizes how to trigger common `UIViewController` lifecycle methods in unit tests using XCTest.

| 🔄 Lifecycle Method     | 🧪 How to Trigger in Test                |
|------------------------|------------------------------------------|
| `viewDidLoad`          | `_ = viewController.view`                |
| `viewWillAppear`       | `viewController.beginAppearanceTransition(true, animated: false)` |
| `viewDidAppear`        | `viewController.endAppearanceTransition()` |
| `viewWillDisappear`    | `viewController.beginAppearanceTransition(false, animated: false)` |
| `viewDidDisappear`     | `viewController.endAppearanceTransition()` |

Use these techniques to simulate lifecycle events and verify side effects in your view controllers.  

## Question 10: Explain Dispatch Barrier?  

A dispatch barrier is a synchronization mechanism used with concurrent dispatch queues in Swift (via Grand Central Dispatch)  
to ensure exclusive access to a resource during a specific task.  

**🚧 What it does:**  
It blocks the queue temporarily so that the barrier task runs alone, even on a concurrent queue.  

All tasks submitted before the barrier must finish before the barrier task starts.  

Tasks submitted after the barrier wait until the barrier task completes.  

**🧠 Why it's useful:**  

It allows safe modification of shared resources (like arrays or dictionaries) while still benefiting from concurrent reads.  

It’s like saying: “Everyone pause—let me do this one thing without interruption—then carry on.”  

```swift
import UIKit

let concurrentQueue = DispatchQueue(label: "com.example.concurrentQueue", attributes: .concurrent)
var sharedArray: [Int] = []

// Reading from the array concurrently
concurrentQueue.async {
    print("Read 1: \(sharedArray)")
}

concurrentQueue.async {
    print("Read 2: \(sharedArray)")
}

concurrentQueue.async {
    print("Read 3: \(sharedArray)")
}

concurrentQueue.async {
    print("Read 4: \(sharedArray)")
}

concurrentQueue.async {
    print("Read 5: \(sharedArray)")
}

concurrentQueue.async {
    print("Read 6: \(sharedArray)")
}

// Writing to the array using a barrier to ensure exclusive access
concurrentQueue.async(flags: .barrier) {
    sharedArray.append(42)
    print("Write: Added 42")
}


/*
concurrentQueue.async {
    sharedArray.append(42)
    print("Write: Added 42")
}
*/

// More reads after the barrier
concurrentQueue.async {
    print("Read 7: \(sharedArray)")
}


concurrentQueue.async {
    print("Read 8: \(sharedArray)")
}

concurrentQueue.async {
    print("Read 9: \(sharedArray)")
}
```


#### 🧪  Output – Without Dispatch Barrier  

| Run 1              | Run 2              | Run 3              | Run 4              | Run 5              | Run 6              |
|--------------------|--------------------|--------------------|--------------------|--------------------|--------------------|
| Read 1: []         | Read 1: []         | Read 1: []         | Read 1: []         | Read 1: []         | Read 1: []         |
| Read 5: []         | Read 2: []         | Read 2: []         | Read 2: []         | Read 2: []         | Read 2: []         |
| Read 6: []         | Read 3: []         | Read 3: []         | Read 3: []         | Read 3: []         | Read 3: []         |
| Write: Added 42    | Read 4: []         | Read 4: []         | Read 5: []         | Read 5: []         | Read 5: []         |
| Read 2: []         | Read 6: []         | Read 6: []         | Read 4: []         | Read 4: []         | Read 6: []         |
| Read 9: []         | Read 5: []         | Read 5: []         | Read 6: []         | Read 6: []         | Read 4: []         |
| Read 8: []         | Read 7: []         | Read 7: []         | Read 7: [42]       | Read 7: []         | Read 7: []         |
| Read 3: []         | Read 8: [42]       | Read 8: []         | Read 8: [42]       | Read 9: [42]       | Read 9: [42]       |
| Read 7: []         | Read 9: [42]       | Read 9: [42]       | Read 9: [42]       | Read 8: [42]       | Read 8: [42]       |
| Read 4: []         | Write: Added 42    | Write: Added 42    | Write: Added 42    | Write: Added 42    | Write: Added 42    |  

#### 🧪  Output – With Dispatch Barrier  

| Run 1              | Run 2              | Run 3              | Run 4              | Run 5              | Run 6              |
|--------------------|--------------------|--------------------|--------------------|--------------------|--------------------|
| Read 1: []         | Read 1: []         | Read 1: []         | Read 1: []         | Read 1: []         | Read 1: []         |
| Read 2: []         | Read 2: []         | Read 2: []         | Read 2: []         | Read 2: []         | Read 2: []         |
| Read 3: []         | Read 3: []         | Read 3: []         | Read 3: []         | Read 3: []         | Read 3: []         |
| Read 5: []         | Read 4: []         | Read 5: []         | Read 6: []         | Read 4: []         | Read 4: []         |
| Read 6: []         | Read 6: []         | Read 4: []         | Read 4: []         | Read 6: []         | Read 5: []         |
| Read 4: []         | Read 5: []         | Read 6: []         | Read 5: []         | Read 5: []         | Read 6: []         |
| Write: Added 42    | Write: Added 42    | Write: Added 42    | Write: Added 42    | Write: Added 42    | Write: Added 42    |
| Read 7: [42]       | Read 7: [42]       | Read 7: [42]       | Read 7: [42]       | Read 7: [42]       | Read 7: [42]       |
| Read 9: [42]       | Read 9: [42]       | Read 8: [42]       | Read 8: [42]       | Read 9: [42]       | Read 9: [42]       |
| Read 8: [42]       | Read 8: [42]       | Read 9: [42]       | Read 9: [42]       | Read 8: [42]       | Read 8: [42]       |  

## Question 11: Explain Dispatch Semaphore  

A Dispatch Semaphore in Swift is a powerful synchronization tool that helps manage access to shared resources across multiple threads.  
Think of it as a traffic light 🚦 for threads—it controls how many threads can enter a critical section at a time.  

**🧠 What It Is**  

A DispatchSemaphore is a wrapper around a traditional counting semaphore. It maintains an internal counter and a queue of waiting threads.  
Created with an initial value: This value represents how many threads can access the resource simultaneously.  
Two key operations:  
**wait():** Decrements the counter. If the counter is less than zero, the thread is blocked.  
**signal():** Increments the counter. If there are blocked threads, one is unblocked.  

```swift
let semaphore = DispatchSemaphore(value: 2) // Allow 2 threads at a time
let queue = DispatchQueue(label: "com.example.queue", attributes: .concurrent)

for i in 1...5 {
    queue.async {
        semaphore.wait()
        print("Task \(i) started")
        sleep(2) // Simulate work
        print("Task \(i) finished")
        semaphore.signal()
    }
}
```
**🔍 What Happens Here:**  
Only 2 tasks run concurrently.  
Others wait until a running task calls signal() to release the semaphore.  
This prevents race conditions and ensures controlled access.  

**✅ Use Cases**  

Limit concurrent access to a resource (e.g., downloading files).  
Throttle network requests.  
Coordinate thread execution when you need strict control.  

##### 🚦 DispatchSemaphore vs No Semaphore – Execution Comparison

This document showcases five runs of concurrent tasks executed with and without a semaphore. The semaphore limits concurrency, ensuring controlled task scheduling.

---

##### ⚙️ DispatchSemaphore Effect on Task Scheduling

This section compares the execution pattern of tasks when running **with** and **without** a `DispatchSemaphore`. The semaphore helps limit the number of concurrent tasks, leading to controlled execution.

---

#### 🚀 Execution Without Semaphore

| Run 1              | Run 2              | Run 3              | Run 4              | Run 5              |
|--------------------|--------------------|--------------------|--------------------|--------------------|
| Task 1 started     | Task 1 started     | Task 1 started     | Task 5 started     | Task 3 started     |
| Task 2 started     | Task 2 started     | Task 2 started     | Task 3 started     | Task 1 started     |
| Task 3 started     | Task 3 started     | Task 3 started     | Task 1 started     | Task 2 started     |
| Task 4 started     | Task 4 started     | Task 5 started     | Task 4 started     | Task 5 started     |
| Task 5 started     | Task 5 started     | Task 4 started     | Task 2 started     | Task 4 started     |
| Task 1 finished    | Task 3 finished    | Task 2 finished    | Task 3 finished    | Task 1 finished    |
| Task 4 finished    | Task 4 finished    | Task 4 finished    | Task 4 finished    | Task 2 finished    |
| Task 5 finished    | Task 2 finished    | Task 5 finished    | Task 5 finished    | Task 3 finished    |
| Task 2 finished    | Task 5 finished    | Task 1 finished    | Task 1 finished    | Task 4 finished    |
| Task 3 finished    | Task 1 finished    | Task 3 finished    | Task 2 finished    | Task 5 finished    |

---

#### 🔐 Execution With DispatchSemaphore

| Run 1              | Run 2              | Run 3              | Run 4              | Run 5              |
|--------------------|--------------------|--------------------|--------------------|--------------------|
| Task 1 started     | Task 1 started     | Task 1 started     | Task 1 started     | Task 1 started     |
| Task 2 started     | Task 2 started     | Task 2 started     | Task 2 started     | Task 2 started     |
| Task 2 finished    | Task 2 finished    | Task 2 finished    | Task 1 finished    | Task 1 finished    |
| Task 1 finished    | Task 1 finished    | Task 1 finished    | Task 2 finished    | Task 2 finished    |
| Task 3 started     | Task 3 started     | Task 3 started     | Task 3 started     | Task 4 started     |
| Task 4 started     | Task 4 started     | Task 4 started     | Task 4 started     | Task 3 started     |
| Task 3 finished    | Task 3 finished    | Task 4 finished    | Task 3 finished    | Task 4 finished    |
| Task 4 finished    | Task 4 finished    | Task 3 finished    | Task 4 finished    | Task 5 started     |
| Task 5 started     | Task 5 started     | Task 5 started     | Task 5 started     | Task 3 finished    |
| Task 5 finished    | Task 5 finished    | Task 5 finished    | Task 5 finished    | Task 5 finished    |


**📝 Observations:**  

Without semaphore: Tasks start and finish in unpredictable order, often overlapping.  
With semaphore: Tasks run in a controlled sequence (e.g., two at a time), improving predictability and stability.  





