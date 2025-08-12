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

## Question 18: What is fallthrough?  
In Swift, the fallthrough statement is used inside a switch statement to force execution to continue to the next case,  
even if the next case’s condition doesn’t match.  

**🔍 Why Use fallthrough?**  
Normally, Swift’s switch statements do not fall through like in C or JavaScript.  
Once a matching case is found and executed, the switch exits. But if you want to intentionally continue to the next case, you use fallthrough.  

**✅ Example:**  
```swift
let value = 2

switch value {
case 1:
    print("One")
case 2:
    print("Two")
    fallthrough
case 3:
    print("Three")
default:
    print("Default")
}
```

-------Output-----  
Two  
Three  

**⚠️ Important Notes:**  
fallthrough does not check the condition of the next case.  
It simply jumps to the next case’s code block.  
You can’t fall through to a case with a value binding (case let) or where clause unless it’s syntactically valid.  

**🧠 When to Use It?**  
Use fallthrough sparingly—only when you want to intentionally share logic between cases.  
In most Swift code, it's better to combine cases or use functions for shared logic.  

**🔐 Scenario: Role-Based Access Control**  

You have different user roles in an app: Guest, User, Moderator, and Admin.  
Each higher role inherits permissions from the lower ones.  

```swift
enum Role {
    case guest, user, moderator, admin
}

func printPermissions(for role: Role) {
    switch role {
    case .guest:
        print("Can view content")
    case .user:
        print("Can comment")
        fallthrough
    case .moderator:
        print("Can delete comments")
        fallthrough
    case .admin:
        print("Can manage users")
    }
}
```
**🔍 Explanation:**  
If the role is .user, it prints:  
Can comment  
Can delete comments  
Can manage users  
Because of fallthrough, it continues executing the next cases regardless of their condition.  

**✅ Why This Is Useful:**  

This pattern helps when permissions or behaviors are cumulative, and you want to avoid repeating logic or combining cases manually.  
Would you like me to convert this into a SwiftUI demo or playground snippet for hands-on testing?  

## Question 19: GCD & Operation Queue Architecture?  

Link: https://viewer.diagrams.net/?tags=%7B%7D&lightbox=1&highlight=0000ff&edit=_blank&layers=1&nav=1&dark=auto#R%3Cmxfile%3E%3Cdiagram%20name%3D%22Page-1%22%20id%3D%22pknFMyu1-kSTIrzhV8YT%22%3E7R1dc5u49rfcB89NH5rhG%2FyY2Em2M%2B3c7qbd7t2XO7KRbSYYvIBre3%2F9lQTC6MMBHAS1uzs7KRaSEOccnW8dRuZkvX9KwGb1KfZhODI0fz8ypyPDcF0P%2FcUNh7zBcrS8YZkEft6kHxueg79h0Ui7bQMfpkzHLI7DLNiwjfM4iuA8Y9pAksQ7ttsiDtmnbsASCg3PcxCKrd8CP1vlrZ6tHdt%2FgcFyRZ%2Bsa8WdNaCdi4Z0Bfx4V2kyH0bmJInjLL9a7ycwxLCjcMnHPZ64Wy4sgVHWZMDuz6fQcn7%2FVf98%2BN%2Fi48F7enn8830xy3cQbosXLhabHSgEkngb%2BRBPoo3M%2B90qyODzBszx3R1COWpbZesQ%2FdLRZTEdTDK4P7lOvXx7RDUwXsMsOaAuxQDDLAB2YClhdwS%2FSbusKqCnbaDA%2BLKc%2BQgUdFHApQWMDAFGkziBqGUSR3O4yVJ0GS%2FQn6fJ9AbtgsjH99DbJgDPPf2QbkA2X70TIItglLHgA2GwjND1HI2GCWrAkAwQNd4VN9aB7%2BPhUjywmOoCFVY9KvQ%2BUWHVkyvaZht8uV2HH4MFDIMIg2sDkwAtAcN0GhbNn49tdWSNuEsG0JCk%2BO0n8eYLSJYwK4A9j8MQbNJgRpaBWxI43yZp8B3%2BBtOcq%2BHWeJvhR09KboUbN3GSoZYUEUxAEANBmu1giokjgrsHH3Ek%2BoIuwqvz1xYzjXv0gO8I3eXvkXmHAD85NlCCYHu4U%2FxMgAkr2ADyQASshygLskM3dOOwZGOKZGNJqMb1VJGNW082MPLvsLjAmy8EaRrMWQqQ7S3oC7KjFkIJDEGGqIIVVJIXLoZ%2BjjFJlJD1uA3puOwMabxN5rAYVBUG3DzCzuYnygh1CxMR8Jevcz5GPAEjH8EMaw48WhCI8zt4R6GNBI47rCp22rBORmEo5juK6RrCP01fb8D1m4ibMmCGup0Qb%2FcZuljii2mQyyDU7dctRN2KDuiBZZ9XYX%2BWjOIRtsHURF7fvh%2FZ09d4zSnsnIuKYvb32q3mMIRPpfu5%2B5Kbho6IF4sUKtk5egOVrcLMopjIPwWcrMLNbQk3p21vZHiGZTL4MuyzOR470bhfhqeLWuRgMmgYzPEAb4o40xgWcebPjjjdNrrBnDCRatQ1MBeuG3WG7XSDOmEi1agTFUQBda0cE4sgDCdxGCdkrOnb0PMt1I6MrvgFVu54xsx0nG7sINOwGCi6xc8KMeiGhBosVYaQPhbA%2BgnZnEe1ULt5RlYxCC%2FQWWFyfiNPF4A9tkVYq3MbyfTyiydh3RTB2i8NG6IC%2FBTGM%2BJrI1SM%2FXE32De3TfBbXgEp66Y3MC030F4vkJbHomOqZ1oWlcvJNs3idYWWL554qcI%2BGPE2UAMvj3hLVW844rUFuE6R%2BnrADmftYQ%2Fn2yyIo8sn4FI5H4yAnXoCvnI7RvPYKc62Y%2FiJFNsxxhWFHgwGkuZ4fB5KDKtmItUoua7YgzFo8MEQDUoh%2BPAFpC%2BsRNC%2BHDZYtbnWIMQpnFxPFMJs51q7tiiExStV50YhzJ79aqaos%2F1kqoQA8XNVid5R99NrgWfvOh51wkSKUWddiSuQ2wCmaNj3a4Faoivw%2BRDNV0kUby%2FRc0K1KQpgS4wX9Gp4Uk390umW9ZxYxtBuP0t0%2B92lhHDjC6VczmViGQP7%2FKwfSFi%2B1e7WGdA6NKrdXvWsmUi1EBRdIZdsd%2BcUNpTdbcni45zd%2FTRBzdojBNk2IUExdAOs8f7P%2F6KWDyR5%2Fcqt8VOYuh5r3JJ5Ya7ZGufSKrSzWSI7kW31yxLtBnbBtZl0NRBvbtINjLp2abjXiLpzAwAC6noOANAsxJ8Xda5mdoM6YSLVqGuQjHsJBqnNbgA6xWAGqS268ytHRr7FyQtRFuFagPYFmKY6B%2ByBc6nsBi74C6RhVwRrzzQsmvwVGn5CIN1cAfV69sDU2yCh4fKo1xnclW2LBn2Fep%2FhGmxW5NT5pVOwM7RT225gLF8gBQ%2Bey%2BqIxmyFgu9BkgSIHC%2BffodOZ3V%2BesvToSrUW80XYSLF5ovTwHwpijrkeGqHR7gPsj%2FoSHT9X9x%2Baxe%2FpvtKt%2BmhJe7RGgmQX3m7%2FMzWUERi8acWdf08InEo16RE0ixXAOEJHCrdCs%2F26fWO5es9tSz%2B%2FRymO7rIF9AtvcqyvPIwgR98RzcIUZIbtMJIOk%2BCTRYn7xEFgSUuCVWpPlKGFfI5kHCI%2BLYZ37CICRSOO6SsfpIL1jtMOsvZjYH3MnpNrXrxLp%2FoxPNnCd%2BCoJQ%2FT2gW1oXaZOtHzQQ0bGsX0OoTMtU3o0KcLxwhe%2Ftq2%2B2anCxs3h%2FgEHiLASMS0Wrc222AuhbIvzIsY06C%2F9e%2BfkB%2FthsfZCRmuIwJu06aYV6yzBmYvyyJnHpfWXGWgCilQuaVJf7IqBqafVWfn8Jw0WavhXnkvfkAoqcr25sEMIrmLuH3lRC1%2Fy8Got1RWb%2FSQNF%2Bm0nnPN1fBXm7U3UI4O2kMAw26Slb8rSNDx1f8%2FEwhCU%2FQEBl7H%2FXHPsjif0%2FHju6Z5freFuJNU5LlZTmK4%2B3VrXtsj5l93Zpg5INNVAG6SavircI9tiq4cGuaR7QNBloNc12yR1ML5X2BfmvG5DbGg9yQ4S5IXEFlI3dw1z0ZX1NsZdF%2B4CdJ2Cekep%2FR66T74OyT5AFoNLlFXZV6IJwAbZh1rj%2F1ywIcVXBpv3vS%2BbVcAOrdR9xxPRI%2FuuKmNxb1oOny7awrLqmzqe7dFcnsUE%2Bylv8ExFa5B%2FVH7mHwrDp76OPgvxq76TIaFHO17nUcE4KDueOcZ6TwhrzxMNN1JmXQr7g014K%2BbqUuincBh5RWhZ2BfdgGSM%2BwJSELVrLirBGE40AiaiiXrVEQ3CQoCKC55RAEkSYiQTYZNKRdsCdsdUtSXTWkUUHuHyO7liLIeAorzQ0kuWlTra0estooEIYGG206K6uVhQIyLKtWzHKK0WXsiCDq9hTXUqCW9tyqtJAv9UMs0YckF%2BV6s2vOL%2Fd3rzfFEC1EsgdNJZicbWZDb7oclMJZOvcRDwtdiSBhOd42qvrEl7QfL2%2Fw1f7cJj%2BiiRWgxwgKrH8eL5d53Z7jUia5Xvs4%2ByE1arPgA4NufnkPNw9qjWfBEBLyqoYMiY3VsbkZIeEhnaA3y0wT2s%2B6qZFXx8CnxTEb%2BOau1PmmruNkLxQ86bvFPoT63xO589t3rbprfAda2MlEjOenaDOoTu4Xd%2BbcicXLwzXk33OxFbG9RrkP3WehNCjHmYNqV7Z3AcazHNPa4pqSbPzLW3VK%2F45dMEn1T5Pvi6l6hJ9ZrOUPb1eU%2BLd%2BbZuveo9FhSme0PTtI4yzGybQwE9W1T9zI4moVmTOzfTGYvwZA6VC81CWBa1YZWoGX%2FFaYveSnWpmcx13qUK0Ut%2BRu4BimbphtUV%2Bkg5aKKz1Icje1VkVHAfWjqhjvsoU1C8BrXFqHGMI%2BVhCMN4mYD1iHXqMvdafOyrzrNruY7nthEWSKaattURuvhKG7JvsvWa9%2B3JfBn%2FCIsfWlh0weiGTYLapjCRxa%2FfmC8yFMW2EsT%2FyOFrlMOco8DSxHMmPcvhBjWUFGQD9JgM4A0biuErU%2FInDZonA3AT6dxEio4smGyopP4F9R5CK16DA6qXe8aGhorrSXtQN5jlcszsXNK2XZedyOXW0lWU0ZAv%2BCRp8y%2Bo1%2FTn3Ga0BqHardAgI7NxBmxdbqbje65jtTKL7mzL7Oij4TZNpOIJrjaZjueV3YlP0c8%2BAdEcAzxaUp2lEgP5BpA2e7xTmys5WYEgemVA7xEVVZmSlIxpCQRNdI%2F2mhtD49GKJExTaTGc2sJWy7OtM0%2Fj2lymhq1MbZEv%2BLTawvanOZBKefW4RQ4jsqA28O8Yz8c4vMr27pxdCw2YGhA3uKYV8Q8xMjJ1O4uMOB6PCdE3acicXcq%2BJD82BDT9AsMNPimX4ZNyy7zMjLbehlmwCSFR1dIXefXNGb4dxVmwCMg3k3YriA1pQJaxQMw9XZH2m4c9GoOpANvPE1BIEPKQ%2FBl3nz%2F8Gz%2BDfMrx9wDupoH%2FMQa%2B9LH54b5RcdTvP0gkoX8eyeP6%2FlbeKbHBBujfTkguV%2BHTpmHZqnpAKxgxXlNVIbZxXwmW7Yzrgc0bWo%2Bh1rwZDyoC7TEruRxe12gc5eecPjbPuLqK8lvyBZ8UzWP5utq%2Bh1qR2SJOtI8byUJG9rmOo3kS2XfaorFMY2q7Hck%2BHqSyAj9jCcsqq%2F50z7PESM9vgEiPeRz5QfFtJ1ZaPcMst2o0oyILAaYdgBuQnkIkX0DkmB%2FvohDJrXwEwgccQiL1lRrGaTclF6limH5oqZdQ3viyPb7DWURcLS77zK8VOpyp6%2FC%2BfUWnuuiCm57qcjSmvyL23sCRW%2BbIgww8Z3EClg08WXUmD3A8WTLYaZNn4pp2VyaPy0leVxOZgpSWFbJ9SWlCBG%2FUknP%2Fa%2BXPnsOjQvziLv0oZj%2BOJ8WZuxdqMxRQqbcZTnyVcZiDVy6%2FX8%2BN9jmuKiFhShfcuECZ24MNUJazkGQfKUjdSGkd1tsdCLKbdzUZCRIHvoJF%2BeWiUsROQXhcVuMCZ%2F0sVOHUX1PiKSM%2BuAmaPokx6YD5HKalbw6QTVzwE0Ob4c1Q9dOtEog9ZlJApTUAlEdramM4CiHysM%2BH3YPohfwTAuLpa%2FUavYr2LkQ2HUJ5li6KbOlBaqpstpDZ6GcSY5Af%2BRku2fQp9vHnkR7%2BDw%3D%3D%3C%2Fdiagram%3E%3C%2Fmxfile%3E  

OperationQueue: https://viewer.diagrams.net/?tags=%7B%7D&lightbox=1&highlight=0000ff&edit=_blank&layers=1&nav=1&title=Untitled%20Diagram.drawio&dark=auto#R%3Cmxfile%3E%3Cdiagram%20name%3D%22Page-1%22%20id%3D%22bxCk7w9Eew4OVcTsMtd8%22%3E7V1bc6M4Fv4t%2B5Cq7oekuAseEzuZna3u7ZlJp2Zm32Qj21Rj8ADOpX%2F9SoAwumDLDoI43alUgmUQ4pxPRzrfkQ4X9mT9%2FEsGN6vPaYjiC8sIny%2Fs6YVlma7r4X%2Bk5KUqCSyjKlhmUViftCu4j76jupCeto1ClDMnFmkaF9GGLZynSYLmBVMGsyx9Yk9bpDF71w1cIqHgfg5jsfTPKCxWVanvGrvyf6NouaJ3No36mzWkJ9cF%2BQqG6VOryL69sCdZmhbV0fp5gmIiPCqX6rq7jm%2BbhmUoKVQu%2BJJ9%2F8%2FjgxVMHpf%2Fc18u%2F3bv736%2FdGr1PMJ4Wz%2FxheXFuMKbRYrrxc0uXmpZeP9sU%2FrFZV5q6hqfYPobrO2b3ff4aFn%2FLyua0YJJmiF8wSRN5mhT5PgwXeA%2FXzYog0WUJr9vEW5EfRV%2BkBlfEy6rWkWLLaaBVoGeSfmqWMe4wMSHMI6WCT6eYyGhDBc8oqyIsHqv6y%2FWURiSy2%2BeVlGB7jdwTup6wmDGZVm6TUJE5GeQ547ieJLGaVbezDYM10CLWh6t8kX5g8vzIku%2FIeYKc3ozaRpOmoKeO7VpNhjBnQula1RkL%2FiU%2BgLbq2FV9yuKsqcdSG2%2FLlu1AOr5dd%2Bo%2B8WyqXkHHXxQo%2BcYJAEJkjj94C6wIYfbdfwpWqA4Sojksfoj3AainmlcF%2F%2B2K5NqpqVi3PMLiC%2FJ6s9hlm6%2BwmyJilpv8zSO4SaPZmUzSEmG5tssjx7RH6jCcVmabgty60ljSUjhJs2wchOsS3wPUohgXjyhnOAsQU%2B3IbYW9AFBqxPgGzxi5Ow6hX2NtTHZFVBssWeAKbknJBiNNrC8IRbWbVJExUs%2FwDE54JiBiBxHAhxA0dQ%2FcvzDyEFJeE2sOenKMczzaM6CgO2p1eUoFEz7QSFlKMam6JG9TvbE9aW%2FpVFpJalwfVa4tgXYKvJ0m81RfVXbWHMV2c6BiooS4UJFpQKa53mFTgJBJ5%2FgjIztvGKwkKtvSLfCvQnuullLP0eZYmZEr%2BvbjaMH0L8HYq9Q96sATicLkkG2GeK%2BvmwQHRDLcTCXD4R7xH%2FSsMfrbEMQVQrAvblwp%2FtsTpeCTtZGXf2lcWV4DPova4Nwau%2FkqqFXpItFjrR0H9c8yqQlaTkQarBnLaPuSow6LXul2bMdm9GXY55s9tiKLHdYs%2Bdab2coGkd1gsRVVedYI6vO%2FtFVZ5tWP6oTKtKtOudHV51jev2oTqhIt%2Brcw6pjVXPAm%2BOc7NBFfujInGnfmtme149P5FgOa7vs%2BnMLDaYlgYOjyydyRV7mM3ZA6eQQ%2F%2F9wj11kGH8UxK2XBOlD2jZHXTimIO3AFYVtaxO2AnVxhiAORLEODGLRsZ%2BiBdzGRRvHhA7cZuQp3wOWA39kLIuO%2BzvAsu2KJNWwWPZEH36yzYt03fLXzx28NjWEY4HXU3CbXwNe5JqOYcjAe2MZRvlNF5mvAdSOYY8Napm7yxFTB8NB80paJBSULWcfLDIlxM0x2gcfpQGiKZ5EvxAK3Lh9RvMtCQHhr%2BCaKDGZ5Zt9kR8Fbuzs%2Bp9jilZu2P7304fm42Mn%2B9B8RZodMU%2F0ob%2BMprtXR1EsRpYundQcTyceqEi3UkTvuAl2s1bPIPy%2FOIU4j%2BhKhb2xoiuegvf2jrl2z%2BBGg1O5dndg6sh7Q2HfcVQnSPzUwWZw1Sk4me9bdSf3Ol51QkWaVQdksegz5Ac42jsQPddhXSkguq73L8l8lSXp9hypAStgLYwhMuKDuiZA5qqeIW5ZCsD1x%2Ba1gOjyXeclcNMzRS7nVLv%2ByKQWeEPRzVc7ZiYjW0Cf7fi554GKdI%2BC%2Bx2zhoWq%2FuKSX8vlyncIFtsMieo7CzcNjOqmgR%2FNTWMtvW%2Be7KaxFQFj4K7yA7ppBySuPtcfWXU%2FoJvGSfxU5lBQ3cDMoS%2B6E4LqzmG66zJS9OhjjTbdpcxyS643cTr%2FdtHahiQI%2BgzmvCYn55FXIfgKgaTzgy%2BwRbEODF%2FRl5jAZI6vjJJlFW19glFRfpDMYicrGCXll%2BeOcOC4oiqMYEiIv5OVjxzERyfSfHHlYwXx6zhuTPRZ8hIsgn0JozYwgt%2FJskcWwYHtjY1g0WP6vCUPaBmfyqnGuUM3cMSFS8NOL354x8Y3uSpOdWyEijQ7NnS38T7V1XunKz0dp0f0HBV%2F0Svx8d%2Bk%2FMqtP02fW6dNX47UPW5jKeQ9T1dthxgLJC6%2FlwucCBJgsRx6QzseAAlWFHxpnVZvJu1ucCBvcFe7hAe0mPPxQdWCfhErc8Wr5Yf5BiaSJYmy4jB6xHWUCC7roGsh83kWbYo0u8Rwg0uSQ0UhkYbGNZnso0mqnsH5t2XZ6S5bdykymOS0x0iq7czxIZNVz%2Fe%2BWpc7Zwa9JQzDFp8w5K0rx2%2FIG4JD2FdavNuUlT3lPXQeWnZX%2FhoPv%2BI%2F200IC7LQz1im5SCQDdwZzkFVsoaNpb4cxQslDdUdP66yZKhfUM7%2B1c9X7d51WSkYTXU38nsoQR3%2Bi5FofyjrqEEr%2BHrvbzNpnd3n64A3mOpTAO99xXG0ybs81D07dLzQCMllWEthhJi9OKEL7CC8kPAJQeCZvtu041XOLj%2F3NemKudYcXjqJN%2FkIb2%2FubiAGLQSBHxAzzDdVTqtF9EycJTGZmQ%2FlO6MMwwW6d0Y1iyDpvJ5uTGI4HAnD0BT2L3MxgvGQowyX%2FEo4GTgvytxdO7NTdYTmnKiIYOuUPfaK7oWqN8Yqnv9QRDHJCaZ6%2Fk1jvRR7sOZceiyY7sqfvsAErlhK0JJ04aa3MnDiA%2FT9wUlhEdhraI8EN%2FKv9oeK%2BLBc%2BnlHfZSfjuc%2BCppSb7%2BVGo%2F7YHVuUwEfy324AQ8erqLeuA95g7u5D3m79HIfCkEumtRxhZ7hEnu7bELHurTJ52ipTAnwEFVngpVMETw8UC2Oyrtp4wFs0lPeTUAH42bDvyx9oi%2Fi3NdmWhTCOL2YlivX8drmxbwyLPuAfSk%2FtZJ57iFpQT8s7XgGiEtZcOraWNMwsOG2fIP%2BmFy94Mrwgt2P2pqyY82TZ7JTMqdOidNpnri0ora%2F%2F3wg7H1mztdkzhTWMlJzFqbz7bry6g7Yq1nVXz7NOnwacwZNZMnn1t7t9Z3eubUoaIk%2F40n6RaDNnRGjd%2F9NCTSKig4z4ugb%2BffLZFp5tBFJivkES%2FgkqEwYUBD27CnDeiEOP03EMU9DcXG2%2FoksTQBs6p3YAi4nrCPJ%2BdzEXGUWrndNNsnOB43m9TRQKITzvFFHFI9LJuycOqIIJsDSM2YI9wH7xwDhAa0BxgDTkMXzOMzuYGgeT2HRJDPK89Mm%2BUwPRsJzWZm6NL7fTgxvSEBrcwucezQSMvpqXFb%2FH%2BUoZM3Sk1WVD0kRsavQrjN0FyVRvmKInkOVfTji3I%2F9MemqTP%2Bgw6cOxBviAjAp4jUOiwrLn%2Bk8k4Qk4hjF6TKDa855Zr474pUIhzxoB3g%2BOMqDNh3bdXrSF79BU8LO2YMu2Gtov3O2UEevOFAzRicbmOHEB8l%2B5ebZP57Q8vO3gZxr4HriMu6hbaACdaiB8R6Q8K4nNm9ltZ%2FLL9JTZ7y5ZBqAq0jTaj%2FaYNXVfrRdmt0D2RuxdLu0g3m0NJihAG57VHAD1qB5p4Lbo%2FvOaUU215a%2B%2BFJL3uBOcAP5LKjzfM5Xpp81dwaVTfuqSz0OrUHwQh94zlHT0muX7E3oZwg1WWsDTHGTgzxorG3dh2mIfHV7u58Q5P%2BT7v1TXBSw2w%2F4zpcEcNMjQF9v1dKtjALXtoGl2eile3ZE3qWE78VOkQL%2FwCRJGrVTH4UUBhcw6uDCMQbAO3Vw4YJenraZE%2Fs%2BLFBPrrtndNwDukMMFtQOqrAeBbYA6HtKKmQYj6a8P7ZjYUDbgKKBqYKrsmHlZgr642NNn4WII%2B4hl26e0%2BeZmSIBHpPXoBkzkgnhywYflaXGTfW5lRcJVMwhWdH%2BFeYka4JZFk8lg0tdGyEmmkS3NNeCpCKLrehNDkVssK9%2FKkwKDiAxhLzB6hEcR1CXz6lS72R6I8A205f0xu5JnmNbUxf00xv5ECqQcY%2BySR5v2HsUuMg9psxmqav56NvfBwxy%2Bw431AFJQgKa7Jyxl542DWlmssZlBPxRt6x6ZsAt1vVt%2B7TJmG86fFWGfeWZjuW7oPyrlsLpaM%2Ff63qEzmkjNz3znUGmZwrEVrP4CRbwvkgzuFTw6w%2FNv6Dny%2BLh3fOvCbDdvuZfPi9rT3TrLanF515f2aM9EUmVKZY3LvmDiPjdmvbAO6wKmp9rGC%2FcVFgPOEKMYuwRocbnYTe%2BMimjufE8nvg31J8aAPF5xGkKgPgshyuOaNy2qCA4LmBCn6Ozfs79CAbhlE0Z9SV4j3uKOndotkLIMUHbZQizbx%2F2RJPLI%2FI9MQTkePf%2FYxVrNk7b9fmQ71apbjL0iKqgd7TexgVMUJlY24DzOcpzehoeerPyogxVuM07o9P72N1R%2FeY%2BBgl6CcUwkNDwMq5W33htyXit95CYoL5%2Fpr7ld%2FzcAGNJq3rb7oU0q2LzAtOfAvwJt5%2FSGl5a1QYQ9dRC9ZK5NXze9d2GXJvgIe08M3KY%2BlpNk9hQM3gGWJT1nLOeGvkGS43Ktp7SbXfs1lNtDjT1l%2FtLxPgG1vX5HcHpnt9qGfCMtXoKRy7jqWspuau9eXCWGCNiCCzJHOnrKkOQeDc5XBToRQTJWbxBpcb7K6Dxyu6mN4uEah96M%2Bs2Tn0JnxB3VXxZ79GhAe4%2BoCYcVRf50XadSMjgj1lKBsfd6WTM%2FJyGpJ%2Fd%2Fh8%3D%3C%2Fdiagram%3E%3C%2Fmxfile%3E
