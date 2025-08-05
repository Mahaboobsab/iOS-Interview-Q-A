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

## Question 11: Call API where 2nd API is depends on 1st API responce?  

### Aproch 1: Async/Await  

```swift

// MARK: - Models
struct GeoLocation: Codable {
    let name: String
    let lat: Double
    let lon: Double
    let country: String
    let state: String?
}

struct WeatherResponse: Codable {
    struct Weather: Codable {
        let main: String
        let description: String
    }
    let weather: [Weather]
    let name: String
}

// MARK: - Networking Functions
func fetchGeoLocation(for city: String) async throws -> GeoLocation {
    let urlString = "https://api.openweathermap.org/geo/1.0/direct?q=\(city)&limit=100&appid=f32f36efd5c0cefa353f90cb87fa26d5&countrycode=IN"
    guard let url = URL(string: urlString) else {
        throw URLError(.badURL)
    }
    
    let (data, _) = try await URLSession.shared.data(from: url)
    let locations = try JSONDecoder().decode([GeoLocation].self, from: data)
    
    guard let first = locations.first else {
        throw NSError(domain: "No location found for \(city)", code: 0)
    }
    
    return first
}

func fetchWeather(lat: Double, lon: Double) async throws -> WeatherResponse {
    let urlString = "https://api.openweathermap.org/data/2.5/weather?lat=\(lat)&lon=\(lon)&appid=f32f36efd5c0cefa353f90cb87fa26d5"
    guard let url = URL(string: urlString) else {
        throw URLError(.badURL)
    }
    
    let (data, _) = try await URLSession.shared.data(from: url)
    return try JSONDecoder().decode(WeatherResponse.self, from: data)
}

// MARK: - Main Function
func fetchWeatherForCity(_ city: String) async {
    do {
        // Step 1: Get coordinates
        let location = try await fetchGeoLocation(for: city)
        print("📍 \(location.name), \(location.state ?? "") — lat: \(location.lat), lon: \(location.lon)")
        
        // Step 2: Use coordinates to get weather
        let weather = try await fetchWeather(lat: location.lat, lon: location.lon)
        if let info = weather.weather.first {
            print("🌤️ Weather in \(weather.name): \(info.main) — \(info.description)")
        } else {
            print("No weather info found.")
        }
        
    } catch {
        print("❌ Error: \(error.localizedDescription)")
    }
}

//MARK:  Method 1

//Task {
//    await fetchWeatherForCity("Ghataprabha")
//}


//

//MARK:  Method 2

/*
Task {
    let location = try await fetchGeoLocation(for: "Jath")     // ⏳ Waits for API 1
    print("📍 \(location.name), \(location.state ?? "") — lat: \(location.lat), lon: \(location.lon)")
    let weather = try await fetchWeather(lat: location.lat, lon: location.lon) // ⏳ Waits for API 2
    if let info = weather.weather.first {
        print("🌤️ Weather in \(weather.name): \(info.main) — \(info.description)")
    } else {
        print("No weather info found.")
    }
}
 */


//MARK:  Method 3


func loadWeather(for city: String) async {
    do {
        let location = try await fetchGeoLocation(for: city)
        
        do {
            let weather = try await fetchWeather(lat: location.lat, lon: location.lon)
            print("🌤️ Weather: \(weather)")
        } catch {
            print("❌ Failed to get weather info: \(error.localizedDescription)")
        }
        
    } catch {
        print("❌ Failed to get location info: \(error.localizedDescription)")
    }
}

Task {
    await loadWeather(for:"Jath")
}

//MARK: Method 4
Task {
    let location = try await fetchGeoLocation(for: "Jath")     // ⏳ Waits for API 1
    let weather = try await fetchWeather(lat: location.lat, lon: location.lon) // ⏳ Waits for API 2
}
```

### Operation Que  

```swift
import Foundation

// MARK: - Models

struct GeoLocation: Decodable {
    let lat: Double
    let lon: Double
    let name: String
}

struct WeatherInfo: Decodable {
    let name: String
    let main: Main
}

struct Main: Decodable {
    let temp: Double
}

// MARK: - Operation 1: GeoLocation Fetch

class GeoLocationOperation: Operation {
    let city: String
    var locationResult: GeoLocation?
    var error: Error?
    
    init(city: String) {
        self.city = city
    }
    
    override func main() {
        let semaphore = DispatchSemaphore(value: 0)
        let urlString = "https://api.openweathermap.org/geo/1.0/direct?q=\(city)&limit=100&appid=f32f36efd5c0cefa353f90cb87fa26d5&countrycode=IN"
        
        guard let url = URL(string: urlString) else {
            print("❌ Invalid URL")
            return
        }

        URLSession.shared.dataTask(with: url) { data, _, error in
            defer { semaphore.signal() }
            
            if let error = error {
                self.error = error
                return
            }
            
            guard let data = data else { return }

            do {
                let result = try JSONDecoder().decode([GeoLocation].self, from: data)
                self.locationResult = result.first
            } catch {
                self.error = error
            }
        }.resume()
        
        semaphore.wait() // Wait for network task to finish
    }
}

// MARK: - Operation 2: Weather Fetch

class WeatherOperation: Operation {
    let geoOp: GeoLocationOperation
    var weatherResult: WeatherInfo?
    var error: Error?
    
    init(dependentOn geoOp: GeoLocationOperation) {
        self.geoOp = geoOp
    }
    
    override func main() {
        guard let location = geoOp.locationResult else {
            print("❌ No location data found.")
            return
        }
        
        let lat = location.lat
        let lon = location.lon
        let semaphore = DispatchSemaphore(value: 0)
        
        let urlString = "https://api.openweathermap.org/data/2.5/weather?lat=\(lat)&lon=\(lon)&appid=f32f36efd5c0cefa353f90cb87fa26d5"
        
        guard let url = URL(string: urlString) else {
            print("❌ Invalid weather URL")
            return
        }

        URLSession.shared.dataTask(with: url) { data, _, error in
            defer { semaphore.signal() }
            
            if let error = error {
                self.error = error
                return
            }
            
            guard let data = data else { return }

            do {
                let result = try JSONDecoder().decode(WeatherInfo.self, from: data)
                self.weatherResult = result
            } catch {
                self.error = error
            }
        }.resume()
        
        semaphore.wait()
    }
}

// MARK: - Execution

func startWeatherFetch() {
    let geoOp = GeoLocationOperation(city: "Gokak")
    let weatherOp = WeatherOperation(dependentOn: geoOp)
    
    weatherOp.addDependency(geoOp)

    let queue = OperationQueue()
    queue.addOperations([geoOp, weatherOp], waitUntilFinished: true)
    
    if let weather = weatherOp.weatherResult {
        print("✅ City in \(geoOp.city)")
        print("✅ Weather in \(weather.name): \(weather.main.temp)°K")
    } else if let error = weatherOp.error ?? geoOp.error {
        print("❌ Error: \(error.localizedDescription)")
    } else {
        print("❌ Unknown error")
    }
}

// Call the function
startWeatherFetch()
```
## Question 12: What is CaseIterable?  

- CaseIterable is a protocol. It’s used to automatically generate a collection of all the cases of an enum.
- Swift automatically synthesizes the allCases property when you conform an enum to the CaseIterable protocol.
---
**🔍 What Happens When You Use CaseIterable**  

When you declare an enum like this:  
```swift
enum Direction: CaseIterable {
    case north, south, east, west
}
```
Swift automatically generates this for you behind the scenes:  
```swift
extension Direction {
    static var allCases: [Direction] {
        return [.north, .south, .east, .west]
    }
}
```
This means you can now do:  

```swift
for direction in Direction.allCases {
    print(direction)
}
```
---
**✅ Conditions for Automatic Synthesis**  

Swift will automatically synthesize allCases only if:  
- The enum has no associated values.
- All cases are explicitly listed.
- ❌ Not Synthesized Automatically

```swift
enum Status: CaseIterable {
    case success(code: Int)
    case failure(message: String)
}
```
This will cause a compiler error because Swift cannot infer all possible combinations of associated values.  

You would need to implement allCases manually:  
```swift
enum Status: CaseIterable {
    case success(code: Int)
    case failure(message: String)

    static var allCases: [Status] {
        return [.success(code: 200), .failure(message: "Error")]
    }
}
```

## Question 13: To which protocol optional is conforms to?  

Optional does conform to the **ExpressibleByNilLiteral** protocol.
This means you can initialize an optional value using nil directly, like this: 
```swift
let name: String? = nil
```
Under the hood  
```swift
extension Optional: ExpressibleByNilLiteral {
    public init(nilLiteral: ()) {
        self = .none
    }
}
```

Other example:  
```swift
struct MyNilType: ExpressibleByNilLiteral {
    init(nilLiteral: ()) {
        print("Initialized with nil")
    }
}

let value: MyNilType = nil
```


## Question 14: What are the Protocols that Optional Can Conforms To?  

|Protocol|Purpose|
|--------|------|
|**ExpressibleByNilLiteral**|Allows Optional to be initialized with nil.|
|Equatable|Enables comparison using == and != if the wrapped type is Equatable.|
|Hashable|Allows Optional to be used in sets or as dictionary keys if the wrapped type is Hashable.|
|Encodable / Decodable|Supports encoding and decoding with Codable if the wrapped type conforms.|
|CustomStringConvertible|Provides a human-readable description.|
CustomDebugStringConvertible|Provides a debug description.|
|Sendable|Ensures thread safety when the wrapped type is Sendable.|
|CaseIterable|**Not supported** — Optional does not conform to CaseIterable by default.|  

## Question 15: In Objective C only reference type can have nil?  
Yes, In Objective-C, the concept of nil is specific to **object (reference)** types.  
**✅ Reference Types (Objects)**  
- In Objective-C, objects (instances of classes) are reference types.
- These can be assigned nil, which means no object is being referenced.
```swift
NSString *name = nil; // Valid
```

**❌ Value Types (Primitives)**  

- Primitive types like int, float, double, BOOL, etc., are value types.
- These cannot be assigned nil. Doing so will result in a compiler error.
```swift
int age = nil; // ❌ Invalid

```

## Question 16: Explain Class & Static in swift?  

**🔹 static Keyword**  
- Used to define type-level properties or methods.  
- Cannot be overridden by subclasses.  
- Commonly used in structs, enums, and classes when you don’t want inheritance.
```swift
struct MathUtils {
    static func square(_ number: Int) -> Int {
        return number * number
    }
}
let result = MathUtils.square(5) // Output: 25
```
**🔹 class Keyword**
- Used only in classes to define type-level methods or properties.
- Can be overridden by subclasses.
- Enables polymorphism at the type level.

```swift
class Animal {
    class func sound() -> String {
        return "Some sound"
    }
}

class Dog: Animal {
    override class func sound() -> String {
        return "Bark"
    }
}

print(Dog.sound()) // Output: Bark
```
|Keyword|Applicable To|	Can Be Overridden|Use Case|
|-------|-------------|------------------|--------|
|static|struct, enum, class|❌ No|Fixed behavior, no inheritance|
|class|class only|✅ Yes|Allow subclass customization|  

**🔹 Type-Level Members in Swift**  

These are declared using either:  

**static** → for structs, enums, and classes (non-overridable)  
**class** → for classes only (overridable in subclasses)  

## Question 17: Explain Factory Design Pattern?  

The Factory Design Pattern in Swift is a creational design pattern that provides an interface for creating objects in a superclass,  
but allows subclasses to alter the type of objects that will be created. It helps encapsulate the object creation logic,  
making your code more modular, testable, and maintainable.  

**🧠 Core Idea**
Instead of using init() directly, you use a factory method to create and return instances. This is especially useful when:  

- Object creation is complex.
- You want to return different types based on conditions.
- You want to hide the creation logic from the caller.
```swift
protocol Notification {
    func send()
}

class EmailNotification: Notification {
    func send() {
        print("Sending Email Notification")
    }
}

class SMSNotification: Notification {
    func send() {
        print("Sending SMS Notification")
    }
}

class NotificationFactory {
    static func createNotification(type: String) -> Notification {
        switch type {
        case "email":
            return EmailNotification()
        case "sms":
            return SMSNotification()
        default:
            fatalError("Unknown notification type")
        }
    }
}

// Usage
let notification = NotificationFactory.createNotification(type: "email")
notification.send() // Output: Sending Email Notification
```
