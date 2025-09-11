# iOS-Interview-QA
Contains interview question &amp; Answers on **Objective C & Swift**

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

![gcd 2](https://github.com/user-attachments/assets/a9b4e7b5-7d7d-41cb-9b05-68da9c37eba6)


![OperationQueue](https://github.com/user-attachments/assets/95015678-3a76-4e07-8448-49e17186c7d7)  

## Question 20: What is the purpose of SceneDelegate in iOS?  
SceneDelegate manages the lifecycle of individual UI scenes (windows). It was introduced in iOS 13 to support multi-window capabilities, especially on iPadOS. It handles scene-specific events like connecting, disconnecting, entering foreground/background, and setting up the UI.  
<img width="744" height="304" alt="Screenshot 2025-08-13 at 9 55 35 AM" src="https://github.com/user-attachments/assets/8d737f02-d2c6-48c4-b06f-38021fa36829" />

## Question 21: What is Closure?  

- A self-contained block of functionality.
- Can be assigned to variables, passed as parameters, or executed inline.

<img width="756" height="430" alt="Screenshot 2025-08-14 at 10 06 17 PM" src="https://github.com/user-attachments/assets/4b648170-c272-4bed-9910-96c7dee0f32d" />

### Basic Syntax
---  
<img width="751" height="515" alt="Screenshot 2025-08-14 at 10 18 30 PM" src="https://github.com/user-attachments/assets/0b9c0586-cec5-4061-b05b-025673893e01" />


## Question 21: Explain SCRUM?  

**🧩 Slide 1: Requirement Breakdown**  

**🔹 Roles Involved:**  

**Business Analyst / Product Owner:** Gathers and defines requirements.

**Development Team:** Builds the software based on those requirements.

**🔹 Flow of Work:**  

**Feature**: A high-level capability (e.g., Attendance Module).

**Epic**: A large chunk of work under that feature (e.g., Student Attendance).

**User Story**: A specific task or functionality (e.g., “Add Student Attendance”).

✅ This breakdown ensures clarity, traceability, and focus during development.


<img width="589" height="636" alt="SCRUM" src="https://github.com/user-attachments/assets/f66237c4-f6de-4539-bb45-6911df74355a" />

## Question 21: Explain Core data stack?  
![Core data stack](https://github.com/user-attachments/assets/c59395a4-62ed-458d-8296-da5f1472214b)


<img width="341" height="512" alt="ccc" src="https://github.com/user-attachments/assets/90164a0a-a699-4ea5-91cd-62e21a235ba3" />  

**🔹 Lightweight Migration**  

**✅ Use When:**
- Changes to the data model are simple and non-destructive.
- You’re adding or renaming attributes, entities, or relationships.
- You’re not changing the underlying structure in a way that requires custom logic.
**💡 Examples:**
  
- Adding a new attribute to an existing entity (e.g., isFavorite: Bool).
- Renaming an attribute using renamingIdentifier.
- Adding a new entity without affecting existing ones.
- Making a relationship optional or changing its delete rule.

**🔸 Heavyweight Migration**  
**✅ Use When:**  

- Changes are complex or destructive.
- You’re removing entities or attributes.
- You’re changing attribute types (e.g., String to Int).
- You need to transform data during migration.
- You’re merging or splitting entities.

**💡 Examples:**  

- Splitting one entity into two (e.g., User → Profile + Account).
- Changing a relationship from one-to-many to many-to-many.
- Converting a String attribute to a Date.
- Removing an attribute that’s no longer needed.

**🔄 Summary Table**  

|Feature|	Lightweight Migration|	Heavyweight Migration|
|-------|------------------------|-----------------------|
|Complexity|	Simple|	Complex|
|Setup|	Automatic|	Manual|
|Mapping Model Required|	No|	Yes|
|Custom Logic|	No|	Often Yes|
|Typical Use Case|	Add/rename attributes|	Merge/split entities, type change|


## Question 22: Explain Repository Pattern?  

"The Repository Pattern is a design pattern that separates the business logic from the data access logic by providing a clean interface to interact with data sources like databases or APIs."  

**🔑 Key Points to Remember**
- It acts like a middleman between your app and the data.
- You interact with the repository, not directly with the database or API.
- Makes your code cleaner, testable, and easier to maintain.

**🎯 Quick Analogy (Optional in Interview)**  

**"Think of it like a library. You don’t go to the storage room to find a book—you ask the librarian (repository), who knows where to get it."**  

**📦 Example in Swift (iOS Context)**  

Let’s say you’re building a ToDo app. You have a Task model and want to fetch tasks from a local database or a remote API.  

**1. Model**
```swift
struct Task {
    let id: Int
    let title: String
    let isCompleted: Bool
}
```
**2. Repository Protocol**  
```swift
protocol TaskRepository {
    func fetchTasks() -> [Task]
    func addTask(_ task: Task)
    func deleteTask(by id: Int)
}
```
**3. Concrete Implementation (e.g., Local DB)**  
```swift
class LocalTaskRepository: TaskRepository {
    private var tasks: [Task] = []

    func fetchTasks() -> [Task] {
        return tasks
    }

    func addTask(_ task: Task) {
        tasks.append(task)
    }

    func deleteTask(by id: Int) {
        tasks.removeAll { $0.id == id }
    }
}
```

**4. Usage in ViewModel**  
```swift
class TaskViewModel {
    private let repository: TaskRepository

    init(repository: TaskRepository) {
        self.repository = repository
    }

    func getAllTasks() -> [Task] {
        return repository.fetchTasks()
    }
}
```
**🧠 How to Explain in an Interview**
"I use the Repository Pattern to abstract the data layer from my business logic.  
For example, in a ToDo app, I define a TaskRepository protocol and implement it with a LocalTaskRepository.  
This allows me to swap data sources easily and write unit tests for my ViewModel without depending on actual data storage.  

**🧪 Unit Test Example in Swift (Using XCTest)**  

Let’s write test cases for the TaskViewModel using a mock repository.  

**1. Mock Repository**  
```swift
class MockTaskRepository: TaskRepository {
    var tasks: [Task] = [
        Task(id: 1, title: "Buy groceries", isCompleted: false),
        Task(id: 2, title: "Walk the dog", isCompleted: true)
    ]

    func fetchTasks() -> [Task] {
        return tasks
    }

    func addTask(_ task: Task) {
        tasks.append(task)
    }

    func deleteTask(by id: Int) {
        tasks.removeAll { $0.id == id }
    }
}
```
**2. Test Case for ViewModel**  
```swift
import XCTest

class TaskViewModelTests: XCTestCase {
    var viewModel: TaskViewModel!
    var mockRepository: MockTaskRepository!

    override func setUp() {
        super.setUp()
        mockRepository = MockTaskRepository()
        viewModel = TaskViewModel(repository: mockRepository)
    }

    func testFetchTasksReturnsCorrectCount() {
        let tasks = viewModel.getAllTasks()
        XCTAssertEqual(tasks.count, 2)
    }

    func testAddTaskIncreasesCount() {
        let newTask = Task(id: 3, title: "Read book", isCompleted: false)
        mockRepository.addTask(newTask)
        let tasks = viewModel.getAllTasks()
        XCTAssertEqual(tasks.count, 3)
    }

    func testDeleteTaskDecreasesCount() {
        mockRepository.deleteTask(by: 1)
        let tasks = viewModel.getAllTasks()
        XCTAssertEqual(tasks.count, 1)
    }
}
```
**🧠 How to Explain in an Interview**  

"The Repository Pattern allows me to mock data sources during unit testing.  
For example, I create a MockTaskRepository to simulate task data and test my TaskViewModel independently.  
This ensures my business logic is correct without relying on actual database or network calls."


## Question 23: 🧠 What Is a Use Case?  

A Use Case is a class or struct that encapsulates a specific task your app performs.  
It’s often called an Interactor in VIPER or UseCase in Clean Architecture. It defines what the app does, not how the UI looks.  

## Question 24: NSURLSession  
# NSURLSession Interview Questions & Answers

## 🟢 Basic Level

### 1. What is `NSURLSession` and why is it used?
`NSURLSession` is a class used to perform network data tasks like downloading, uploading, and fetching data from web services. It replaces older APIs like `NSURLConnection` and provides better support for background tasks, caching, and configuration.

### 2. What are the different types of `NSURLSession` configurations?
- `default`: Stores cookies, credentials, and caches.
- `ephemeral`: Doesn’t store cookies, credentials, or caches—good for privacy.
- `background`: Allows tasks to continue even if the app is suspended or terminated.

### 3. How do you make a simple GET request using `NSURLSession`?
```swift
let url = URL(string: "https://api.example.com/data")!
let task = URLSession.shared.dataTask(with: url) { data, response, error in
    if let data = data {
        // Handle response
    }
}
task.resume()
```

### 4. What is the difference between `dataTask`, `downloadTask`, and `uploadTask`?
- `dataTask`: For small data transfers (e.g., JSON APIs).
- `downloadTask`: For downloading files to disk.
- `uploadTask`: For uploading files or data to a server.

### 5. How do you handle errors in a `NSURLSession` request?
Check the `error` object in the completion handler. Also validate the `HTTPURLResponse` status code and ensure `data` is not nil.

## 🟡 Intermediate Level

### 6. How does `NSURLSession` handle caching?
It uses `URLCache` and respects HTTP cache headers. You can customize caching behavior using `URLSessionConfiguration`.

### 7. What is the role of `URLSessionDelegate`?
It allows handling authentication challenges, redirection, and background task completion. You can also monitor task-level events like progress and errors.

### 8. How do you handle authentication challenges in `NSURLSession`?
Implement `urlSession(_:didReceive:completionHandler:)` in the delegate to respond to server trust or basic auth challenges.

### 9. How do you cancel a running task in `NSURLSession`?
Call `.cancel()` on the task:
```swift
task.cancel()
```

### 10. How do you track progress for a file download or upload?
Use `URLSessionDownloadDelegate` or `URLSessionTaskDelegate` methods like:
- `urlSession(_:downloadTask:didWriteData:totalBytesWritten:totalBytesExpectedToWrite:)`
- `urlSession(_:task:didSendBodyData:totalBytesSent:totalBytesExpectedToSend:)`

## 🔴 Advanced Level

### 11. How does `NSURLSession` work in the background with `backgroundSessionConfiguration`?
It creates a session that continues tasks even if the app is suspended. The system wakes the app when the task completes and calls the delegate method `urlSessionDidFinishEvents(forBackgroundURLSession:)`.

### 12. Explain how to resume a download after the app is terminated.
Use `downloadTask(withResumeData:)` with resume data captured from `cancel(byProducingResumeData:)`. Store the resume data persistently.

### 13. How do you implement retry logic for failed requests using `NSURLSession`?
Wrap the request in a retry mechanism using exponential backoff. Retry based on error type or status code (e.g., 500, 503).

### 14. What are the security considerations when using `NSURLSession` (e.g., SSL pinning)?
Use `urlSession(_:didReceive:completionHandler:)` to validate server certificates manually. This helps prevent man-in-the-middle attacks.

### 15. How would you design a network layer using `NSURLSession` that supports dependency injection and testability?
- Abstract `URLSession` behind a protocol.
- Inject mock sessions for unit testing.
- Use a service layer to encapsulate API logic.
- Handle parsing, error handling, and retries in a centralized way.


### Question 25: Q: What’s the difference between Git rebase and merge? Can you explain with an example?


"Both git **merge** and git **rebase** are used to integrate changes from one branch into another, but they handle history differently.  

When I use git merge, Git creates a new merge commit that ties together the histories of both branches.  
This is useful in collaborative environments where preserving the exact timeline of contributions is important.   
However, it can lead to a cluttered commit history, especially with frequent merges.

On the other hand, git rebase rewrites the commit history by replaying my feature branch commits on top of the latest main.  
This results in a clean, linear history — which is ideal for solo development or preparing a branch before merging.   
I recently used rebase in my SwiftUI project to move feature branch changes onto main.  
Instead of merging, I rebased main onto the feature branch, resolved a few conflicts, and then force-pushed main. This made the commit history easier to read and avoided unnecessary merge commits.  


**In short:**  


**Merge** preserves history and shows when branches were combined.  

**Rebase** rewrites history to make it look like changes were made sequentially. I choose based on context — merge for collaboration, rebase for clarity."

## Question 26: What is the main difference between Generics and Associated Types?  

| **Generics**                           | **Associated Types**                |
| -------------------------------------- | ----------------------------------- |
| Defined on functions, structs, classes | Defined inside protocols            |
| Type decided by **caller**             | Type decided by **conforming type** |
| Best for algorithms & containers       | Best for protocol abstraction       |

```swift
// Using generics
func swapValues<T>(_ a: inout T, _ b: inout T) {
    let temp = a
    a = b
    b = temp
}

// Using associated type
protocol Swappable {
    associatedtype Value
    mutating func swap(_ a: inout Value, _ b: inout Value)
}

struct IntSwapper: Swappable {
    func swap(_ a: inout Int, _ b: inout Int) {
        let temp = a
        a = b
        b = temp
    }
}
```
## Question 27:Difference between Architecture vs Design Patterns in iOS?

Understanding the difference between **architecture** and **design patterns** is crucial for building scalable and maintainable iOS applications.

---

## 🏛️ Architecture (High-Level Structure)

**Definition**: Architecture defines the overall structure of your app — how components like UI, business logic, and data interact.

### Common iOS Architectures:
- **MVC (Model-View-Controller)** – Apple's default, but can lead to "Massive View Controllers".
- **MVVM (Model-View-ViewModel)** – Separates UI logic from business logic.
- **VIPER** – Highly modular, used in large-scale apps.
- **Clean Architecture** – Focuses on separation of concerns and testability.

### Purpose:
- Organize codebase
- Improve scalability and testability
- Define how layers communicate

---

## 🧩 Design Patterns (Low-Level Solutions)

**Definition**: Design patterns are reusable solutions to common problems in software design. They are building blocks used within an architecture.

### Common iOS Design Patterns:
- **Singleton** – Shared instance (e.g., `UserDefaults`, `URLSession`)
- **Observer** – Used in `NotificationCenter`, Combine, KVO
- **Delegate** – Used in `UITableViewDelegate`, etc.
- **Facade** – Simplifies complex subsystems
- **Factory**, **Builder**, **Strategy**, etc.

### Purpose:
- Solve specific design problems
- Improve code reusability and flexibility
- Used within architectural layers

---

## 🧠 Analogy

Think of **architecture** as the **blueprint of a house** (how rooms are laid out), and **design patterns** as the **furniture and tools** you use inside each room to make it functional.

---

## ✅ Summary Table

| Aspect              | Architecture                     | Design Pattern                        |
|---------------------|----------------------------------|----------------------------------------|
| **Scope**           | High-level structure             | Low-level reusable solutions           |
| **Examples**        | MVC, MVVM, VIPER, Clean Arch     | Singleton, Observer, Facade, Delegate  |
| **Purpose**         | Organize app layers              | Solve specific design problems         |
| **Usage**           | App-wide                         | Within classes/modules                 |

---

| Design Pattern       | Description                                                                 | Common Usage in iOS                                      |
|----------------------|-----------------------------------------------------------------------------|-----------------------------------------------------------|
| **Singleton**         | Ensures a class has only one instance and provides a global access point.  | `UserDefaults`, `UIApplication.shared`, `URLSession.shared` |
| **Observer**          | Allows objects to be notified of changes in other objects.                 | `NotificationCenter`, `KVO`, `Combine`, `@Published`       |
| **Delegate**          | Enables one object to act on behalf of another.                            | `UITableViewDelegate`, `UICollectionViewDelegate`          |
| **Facade**            | Provides a simplified interface to a complex subsystem.                    | `AVFoundation`, `CoreData` wrappers                        |
| **Factory**           | Creates objects without specifying the exact class.                        | `UIViewController.instantiate()` from Storyboard           |
| **MVC (Model-View-Controller)** | Separates concerns into model, view, and controller.                     | UIKit architecture                                        |
| **MVVM (Model-View-ViewModel)** | Separates UI logic from business logic using ViewModel.                  | SwiftUI, Combine-based apps                               |
| **Builder**           | Constructs complex objects step-by-step.                                   | `URLRequest` configuration, custom UI components           |
| **Strategy**          | Enables selecting an algorithm at runtime.                                 | Custom layout strategies, sorting/filtering logic          |
| **Adapter**           | Converts one interface to another.                                         | Bridging Objective-C and Swift, protocol extensions        |
| **Command**           | Encapsulates a request as an object.                                       | Undo/Redo operations, UI actions                          |
| **Composite**         | Treats individual objects and compositions uniformly.                      | View hierarchies (`UIView` containing subviews)            |
| **Template Method**   | Defines the skeleton of an algorithm, deferring steps to subclasses.       | `UIViewController` lifecycle methods (`viewDidLoad`, etc.) |


## 📌 Conclusion

- Use **architecture** to define how your app is structured.
- Use **design patterns** to solve specific problems within that structure.
- Both are essential for writing clean, maintainable, and scalable iOS code.

## Question 28: What is Facade design pattern?  

- The Facade pattern provides a single, simplified interface to a complex system, hiding its internal details and making it easier for the client to use.
- **Facade = One easy door to many complex rooms.**

**Example**:  

**UIKit → UIImage:** Internally deals with Core Graphics, file handling, and memory, but provides a simple interface (UIImage(named:)).

**URLSession**: Acts as a facade over lower-level networking APIs like CFNetwork and sockets.

**AVPlayer**: Provides a high-level interface to play media, hiding the complexity of AVFoundation.  

**Example:**  
```swift
// Subsystems
class AudioPlayer {
    func playAudio() { print("Playing audio...") }
}

class VideoPlayer {
    func playVideo() { print("Playing video...") }
}

class Subtitles {
    func loadSubtitles() { print("Loading subtitles...") }
}

// Facade
class MediaFacade {
    private let audio = AudioPlayer()
    private let video = VideoPlayer()
    private let subs = Subtitles()
    
    func playMovie() {
        audio.playAudio()
        video.playVideo()
        subs.loadSubtitles()
        print("Movie started 🎬")
    }
}

// Client Code
let media = MediaFacade()
media.playMovie()
```
**1️⃣ Without Facade – Raw URLSession**  
If you use URLSession directly, the client code has to deal with:
- Building requests (URLRequest).
- Handling URLSession.dataTask.
- Checking errors.
- Checking HTTP status codes.
- Parsing JSON.
- Cancel/resume tasks manually.
- Example (raw usage):

```swift
let url = URL(string: "https://jsonplaceholder.typicode.com/posts")!

let task = URLSession.shared.dataTask(with: url) { data, response, error in
    if let error = error {
        print("Error: \(error)")
        return
    }

    guard let httpResponse = response as? HTTPURLResponse,
          httpResponse.statusCode == 200 else {
        print("Invalid response")
        return
    }

    guard let data = data else {
        print("No data received")
        return
    }

    // Parse JSON
    do {
        let posts = try JSONDecoder().decode([Post].self, from: data)
        print(posts)
    } catch {
        print("Decoding error: \(error)")
    }
}
task.resume()
```
**2️⃣ With Facade – NetworkManager**  

Now, let’s see what happens when we add a Facade (NetworkManager):  
```swift
struct Post: Codable {
    let id: Int
    let title: String
}

// Facade
class NetworkManager {
    static let shared = NetworkManager()
    private init() {}
    
    func fetch<T: Decodable>(_ type: T.Type, from url: URL, completion: @escaping (Result<T, Error>) -> Void) {
        let task = URLSession.shared.dataTask(with: url) { data, response, error in
            if let error = error {
                completion(.failure(error))
                return
            }
            
            guard let httpResponse = response as? HTTPURLResponse,
                  (200...299).contains(httpResponse.statusCode),
                  let data = data else {
                completion(.failure(NSError(domain: "InvalidResponse", code: 0)))
                return
            }
            
            do {
                let decoded = try JSONDecoder().decode(T.self, from: data)
                completion(.success(decoded))
            } catch {
                completion(.failure(error))
            }
        }
        task.resume()
    }
}
```
Usage (client side):
```swift
let url = URL(string: "https://jsonplaceholder.typicode.com/posts")!

NetworkManager.shared.fetch([Post].self, from: url) { result in
    switch result {
    case .success(let posts):
        print("✅ Got \(posts.count) posts")
    case .failure(let error):
        print("❌ Error: \(error)")
    }
}
```
**3️⃣ Why this is Facade?**

- Client only calls one method (fetch) instead of dealing with URLSession, error checks, and decoding.
- Complex subsystems hidden:
- URLSession.dataTask → hidden
- JSONDecoder → hidden
- HTTPURLResponse validation → hidden
- Unified & clean API: just pass a URL + expected type.
- Reusability: Any screen needing networking can use the same facade without duplicating boilerplate code.

**4️⃣ Key Idea**  

**Subsystem** = URLSession, JSONDecoder, Error handling, etc.
**Facade** = NetworkManager which exposes one simple entry point.
**Client** = Your ViewModel or ViewController which consumes a clean API without worrying about internals.  

**NetworkManager acts as a Facade because it simplifies the complex networking workflow (URLSession, response validation, JSON decoding) into a single clean API for clients to use. Clients don’t need to know how URLSession or JSONDecoder works — they just call one method**  

## Question 29: What is KVC & KVO?  

**✅ KVC (Key-Value Coding)**  

KVC is a mechanism in iOS that allows you to **access and modify object properties** using string keys instead of direct property access.  
It’s useful for dynamic data handling, like updating properties from a dictionary or JSON.  
```swift
person.setValue("Mahaboob", forKey: "name")
```

**✅ KVO (Key-Value Observing)**  

KVO lets you observe changes to property values and respond when they change.  
It’s commonly used for UI updates or reactive behavior when a model changes.  

```swift
person.observe(\.name, options: [.new]) { _, change in
    print("Name changed to \(change.newValue!)")
}
```
**KVC (Key-Value Coding) and KVO (Key-Value Observing) are not design patterns in the traditional sense like MVC, MVVM, or Singleton.  
Instead, they are mechanisms or features provided by the Objective-C runtime (and available in Swift via bridging) that enable dynamic behavior in your code.**  

In Swift, especially with SwiftUI, these are largely replaced by Combine and property wrappers like **@Published, @ObservedObject**, etc.  

**🎯 Scenario: Profile Update in a Banking App**  

Imagine you're building a banking app. There's a user profile screen where the user's name, email, and balance are displayed. You want to:  
- Dynamically update the UI when the user's balance changes.  
- Access user properties using strings for flexible data handling (e.g., from JSON or form inputs).

```swift
//MARK: 🔹 Step 1: Define the Model
class User: NSObject {
    @objc dynamic var name: String
    @objc dynamic var email: String
    @objc dynamic var balance: Double

    init(name: String, email: String, balance: Double) {
        self.name = name
        self.email = email
        self.balance = balance
    }
}

//MARK: 🔹 Step 2: Use KVC to Update Properties from a Dictionary

let user = User(name: "Mahaboob", email: "mahaboob@example.com", balance: 1000.0)

let updateDict: [String: Any] = [
    "name": "Mahaboobsab Nadaf",
    "balance": 1500.0
]

for (key, value) in updateDict {
    user.setValue(value, forKey: key) // 🔥 KVC in action
}

//MARK: 🔹 Step 3: Use KVO to Observe Balance Changes

class BalanceObserver {
    var observation: NSKeyValueObservation?

    init(user: User) {
        observation = user.observe(\.balance, options: [.new]) { _, change in
            print("💰 Balance updated to: \(change.newValue ?? 0.0)")
            // You can trigger a UI update here
        }
    }
}

let observer = BalanceObserver(user: user)
user.balance = 2000.0 // 🔥 KVO triggers observer
```

**🧠 Summary**  

**KVC/KVO:** Powerful but legacy. Still useful when working with Objective-C APIs or dynamic behavior.
**Property Observers (didSet):** Great for simple, local change tracking.
**Property Wrappers (@Published)**: Modern, reactive, and preferred in SwiftUI/Combine.  

**Compare between KVC/KVO vs Property Observers vs Property Wrappers**  

| Feature                | KVC (Key-Value Coding)                            | KVO (Key-Value Observing)                          | Property Observers (`willSet` / `didSet`)         | Property Wrappers (`@Published`, etc.)            |
|------------------------|---------------------------------------------------|----------------------------------------------------|---------------------------------------------------|---------------------------------------------------|
| **Purpose**            | Access/set properties using string keys          | Observe property changes at runtime                | React to property changes locally                 | Add behavior to properties (e.g., observation)    |
| **Requires**           | `NSObject` + `@objc`                             | `NSObject` + `@objc dynamic`                       | Native Swift                                      | Native Swift + Combine/SwiftUI                   |
| **Scope**              | Dynamic access from anywhere                     | External observation supported                     | Internal to the class                             | External observation via Combine/SwiftUI         |
| **Use Case**           | JSON parsing, dynamic forms                      | UI updates, bindings (legacy)                      | Logging, validation, internal triggers            | Reactive UI updates, state management            |
| **SwiftUI Friendly**   | ❌ Not recommended                                | ❌ Legacy approach                                 | ❌ Not reactive                                    | ✅ Fully reactive                                 |
| **Combine Support**    | ❌ No                                             | ❌ No                                               | ❌ No                                              | ✅ Yes                                             |
| **Customization**      | Limited                                           | Limited                                            | Limited                                           | Highly customizable                              |
| **Example**            | `setValue(_:forKey:)`                            | `observe(_:options:changeHandler:)`                | `didSet { ... }`                                  | `@Published var name: String`                    |

## Question 29: What is adapter design pattern?  

Adapter helps us reuse existing classes/libraries without modifying them, by providing a middle layer that translates one interface into another.  

**🔧 Real-Life Analogy**

- Think of a charging adapter:
- Your laptop charger plug doesn’t fit the wall socket.
- You use an adapter to make them compatible.
- Neither the socket nor the charger changes — only the adapter does the work

```swift
import Foundation
import CoreLocation
import UIKit

// MARK: - Target Protocol
protocol LocationProvider {
    func getLocation(completion: @escaping (String) -> Void)
}

// MARK: - Adaptee
// Apple service provides location using CLLocationManager (delegate-based API)
class AppleLocationService: NSObject, CLLocationManagerDelegate {
    private let manager = CLLocationManager()
    private var completion: ((CLLocation) -> Void)?
    
    override init() {
        super.init()
        manager.delegate = self
        manager.requestWhenInUseAuthorization()
    }
    
    func requestLocation(completion: @escaping (CLLocation) -> Void) {
        self.completion = completion
        manager.requestLocation()
    }
    
    // Delegate method
    func locationManager(_ manager: CLLocationManager, didUpdateLocations locations: [CLLocation]) {
        if let location = locations.first {
            completion?(location)
        }
    }
    
    func locationManager(_ manager: CLLocationManager, didFailWithError error: Error) {
        print("❌ Location error: \(error.localizedDescription)")
    }
}

// MARK: - Adapter
class LocationAdapter: LocationProvider {
    private let appleService: AppleLocationService
    
    init(service: AppleLocationService) {
        self.appleService = service
    }
    
    func getLocation(completion: @escaping (String) -> Void) {
        appleService.requestLocation { location in
            let formatted = "Lat: \(location.coordinate.latitude), Lng: \(location.coordinate.longitude)"
            completion(formatted)
        }
    }
}

// MARK: - Client
class LocationApp {
    private let provider: LocationProvider
    
    init(provider: LocationProvider) {
        self.provider = provider
    }
    
    func showLocation() {
        provider.getLocation { location in
            print("📍 Current Location -> \(location)")
        }
    }
}

// MARK: - Usage
let service = AppleLocationService()               // Adaptee
let adapter = LocationAdapter(service: service)    // Adapter
let app = LocationApp(provider: adapter)           // Client
app.showLocation()

```
**🖥 Output**  
```yml
📍 Current Location -> Lat: 12.9716, Lng: 77.5946
```
✅ Key Points

- **AppleLocationService** (Adaptee) internally uses CLLocationManagerDelegate to fetch the actual location.
- **LocationAdapter** converts it into the LocationProvider interface our app expects.
- **The client (LocationApp)** only sees the getLocation method with a nice, simple String.

## Question 30: What is Builder Pattern?  

**Builder is a creational design pattern that lets you construct complex objects step by step.**  

Builder pattern is used when we want to construct complex objects step by step instead of passing everything in one initializer.  
In iOS, we often use it for objects like UIAlertController, URLRequest, or custom models that require many optional configurations.  

**🔧 Real-Life Analogy**  

- Think of building a Burger 🍔:
- You don’t pass all ingredients at once.
- You add bun → patty → cheese → sauce → lettuce step by step.
- Finally, you get a burger ready to eat.
- That’s exactly what the Builder pattern does — builds an object step by step instead of one giant initializer.

**🧑‍💻 Example in iOS (Swift)**
Let’s say we want to build a complex UIAlertController step by step.  

**Without Builder (messy)**  

```swift
let alert = UIAlertController(title: "Error",
                              message: "Something went wrong",
                              preferredStyle: .alert)

alert.addAction(UIAlertAction(title: "Retry", style: .default, handler: nil))
alert.addAction(UIAlertAction(title: "Cancel", style: .cancel, handler: nil))
```
This works, but if the alert becomes complex (lots of customization), the initializer becomes hard to read & maintain.  

**With Builder Pattern**  

```swift
import UIKit

// MARK: - Builder
class AlertBuilder {
    private var title: String?
    private var message: String?
    private var style: UIAlertController.Style = .alert
    private var actions: [UIAlertAction] = []
    
    func setTitle(_ title: String) -> AlertBuilder {
        self.title = title
        return self
    }
    
    func setMessage(_ message: String) -> AlertBuilder {
        self.message = message
        return self
    }
    
    func setStyle(_ style: UIAlertController.Style) -> AlertBuilder {
        self.style = style
        return self
    }
    
    func addAction(title: String, style: UIAlertAction.Style, handler: ((UIAlertAction) -> Void)? = nil) -> AlertBuilder {
        let action = UIAlertAction(title: title, style: style, handler: handler)
        actions.append(action)
        return self
    }
    
    func build() -> UIAlertController {
        let alert = UIAlertController(title: title, message: message, preferredStyle: style)
        actions.forEach { alert.addAction($0) }
        return alert
    }
}

// MARK: - Usage
let alert = AlertBuilder()
    .setTitle("Error")
    .setMessage("Something went wrong")
    .addAction(title: "Retry", style: .default, handler: nil)
    .addAction(title: "Cancel", style: .cancel, handler: nil)
    .build()
```
**✅ Benefits of Builder Pattern**  
- Makes object creation readable & flexible.
- Avoids large initializers with too many parameters.
- Allows step-by-step construction.
- Same builder can be reused for different objects (e.g., different types of alerts).

## Question 31:  Explain Strategy Design Pattern?  

Strategy is a behavioral design pattern that allows you to define a family of algorithms, put each one in a separate class, and make them interchangeable.  
The client can switch strategies at runtime without changing its code.  

**"Strategy pattern is used when we have multiple algorithms for a task, and we want to choose one at runtime.  
In iOS, a real example is multiple payment methods or login methods — each strategy has the same interface but different implementation."**  

**📖 Problem**  

- When you load an image in an app, there can be multiple strategies:
- Load from Cache (fastest).
- Load from Disk (local file storage).
- Load from Network (API or CDN).

👉 Instead of hardcoding one way, we use the Strategy Pattern so that the client can switch at runtime.  

**🧑‍💻 Example – Image Loading with Strategy**  

```swift
import UIKit

// MARK: - Strategy Protocol
protocol ImageLoadingStrategy {
    func loadImage(name: String, completion: @escaping (UIImage?) -> Void)
}

// MARK: - Concrete Strategies

// 1. Load from Cache
class CacheImageLoader: ImageLoadingStrategy {
    private var cache: [String: UIImage] = [
        "logo": UIImage(systemName: "star.fill")! // mock image in cache
    ]
    
    func loadImage(name: String, completion: @escaping (UIImage?) -> Void) {
        print("📦 Loading from Cache...")
        completion(cache[name])
    }
}

// 2. Load from Disk
class DiskImageLoader: ImageLoadingStrategy {
    func loadImage(name: String, completion: @escaping (UIImage?) -> Void) {
        print("💽 Loading from Disk...")
        // Simulating disk read
        let image = UIImage(systemName: "folder.fill")
        completion(image)
    }
}

// 3. Load from Network
class NetworkImageLoader: ImageLoadingStrategy {
    func loadImage(name: String, completion: @escaping (UIImage?) -> Void) {
        print("🌐 Loading from Network...")
        // Simulating network call with delay
        DispatchQueue.global().asyncAfter(deadline: .now() + 1) {
            let image = UIImage(systemName: "wifi")
            completion(image)
        }
    }
}

// MARK: - Context
class ImageLoaderContext {
    private var strategy: ImageLoadingStrategy
    
    init(strategy: ImageLoadingStrategy) {
        self.strategy = strategy
    }
    
    func setStrategy(_ strategy: ImageLoadingStrategy) {
        self.strategy = strategy
    }
    
    func load(name: String, completion: @escaping (UIImage?) -> Void) {
        strategy.loadImage(name: name, completion: completion)
    }
}

// MARK: - Usage
let context = ImageLoaderContext(strategy: CacheImageLoader())

// Try cache first
context.load(name: "logo") { image in
    print("✅ Got image from Cache:", image ?? UIImage())
}

// Switch to Disk
context.setStrategy(DiskImageLoader())
context.load(name: "logo") { image in
    print("✅ Got image from Disk:", image ?? UIImage())
}

// Switch to Network
context.setStrategy(NetworkImageLoader())
context.load(name: "logo") { image in
    print("✅ Got image from Network:", image ?? UIImage())
}
```

**🖥 Sample Output**  
```yml
📦 Loading from Cache...
✅ Got image from Cache: <UIImage: star.fill>
💽 Loading from Disk...
✅ Got image from Disk: <UIImage: folder.fill>
🌐 Loading from Network...
✅ Got image from Network: <UIImage: wifi>
```

**✅ Why This is Strategy Pattern**

- Context (ImageLoaderContext) → Uses ImageLoadingStrategy.
- Strategies (CacheImageLoader, DiskImageLoader, NetworkImageLoader) → Different ways of fetching images.
- Client → Can switch strategies dynamically (setStrategy).

**⚡ Real-World Note:**  

This is how libraries like SDWebImage or Kingfisher internally work — they try cache → disk → network.
Each step is basically a strategy.

## Question 32: What is Command Design Pattern?  

**In iOS, I can use Command pattern to wrap actions like ‘Turn Light On’ or ‘Fetch Data’ into objects, so the invoker (UI) doesn’t need to know how the action is executed. It also allows undo and scheduling of actions.”**  

Command is a behavioral design pattern that turns a request into a standalone object containing all the information about the request.
This allows you to parameterize methods with different requests, delay or queue execution, and support undo/redo operations.

**🔧 Real-Life Analogy**
- Think of a TV Remote Control 📺:
- Each button (Command) knows how to execute a specific action on the TV (Receiver).
- The remote (Invoker) doesn’t know how the TV works — it just sends commands.
- You can also undo (go back to previous channel, turn volume down, etc.).

**Example**: Note app, Any kind of design app or IOT projects.

```swift
import UIKit

// Receiver
class Light {
    func on() { print("💡 ON") }
    func off() { print("💡 OFF") }
}

// Command
protocol Command {
    func execute()
    func undo()
}

class LightOnCommand: Command {
    let light: Light
    init(_ light: Light) { self.light = light }
    func execute() { light.on() }
    func undo() { light.off() }
}

// Invoker
class Remote {
    var command: Command?
    func pressButton() { command?.execute() }
    func pressUndo() { command?.undo() }
}

// Usage
let light = Light()
let onCommand = LightOnCommand(light)
let remote = Remote()

remote.command = onCommand
remote.pressButton()   // 💡 ON
remote.pressUndo()     // 💡 OFF
```

## Question 33: Explain Composite Design Pattern?  

Composite pattern lets you treat individual objects and compositions of objects uniformly.  
It allows you to build a tree structure of objects where clients can interact with single objects or groups of objects in the same way.  

**Key Points**

- **Component** → Common interface for objects and groups.
- **Leaf** → Individual objects (cannot have children).
- **Composite** → Groups of objects (can contain Leafs or other Composites).
- **Client** → Works with Component interface, doesn’t care if it’s a Leaf or Composite.

**Real-world iOS Example: UIViews**  

- UIView is a **Composite**.
- UILabel, UIButton are **Leaf** objects.
- A UIView can contain multiple subviews (other Leaf or Composite views).
- You can call addSubview() on a UIView and treat it the same way as a single UILabel.

```swift
import Foundation

// Component
protocol FileSystemItem {
    var name: String { get }
    func display(indent: String)
}

// Leaf
class File: FileSystemItem {
    let name: String
    init(name: String) { self.name = name }
    
    func display(indent: String = "") {
        print(indent + "📄 \(name)")
    }
}

// Composite
class Folder: FileSystemItem {
    let name: String
    private var items: [FileSystemItem] = []
    
    init(name: String) { self.name = name }
    
    func add(_ item: FileSystemItem) {
        items.append(item)
    }
    
    func display(indent: String = "") {
        print(indent + "📁 \(name)")
        for item in items {
            item.display(indent: indent + "   ")
        }
    }
}

// Usage
let file1 = File(name: "Resume.pdf")
let file2 = File(name: "Photo.png")

let folder1 = Folder(name: "Documents")
folder1.add(file1)
folder1.add(file2)

let file3 = File(name: "Song.mp3")
let folder2 = Folder(name: "Music")
folder2.add(file3)

let rootFolder = Folder(name: "Root")
rootFolder.add(folder1)
rootFolder.add(folder2)

rootFolder.display()
```

**Sample Output**  

```markdown
📁 Root
   📁 Documents
      📄 Resume.pdf
      📄 Photo.png
   📁 Music
      📄 Song.mp3

```
