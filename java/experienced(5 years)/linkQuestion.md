How can you sort a Map in Java?
A normal HashMap does not maintain ordering.
You can sort a map by:
•	key 
•	value 
Sort by key
Map<String, Integer> map = new HashMap<>();

map.put("C", 30);
map.put("A", 10);
map.put("B", 20);

Map<String, Integer> sorted =
        new TreeMap<>(map);

System.out.println(sorted);
Output:
{A=10, B=20, C=30}
TreeMap maintains keys in sorted order.
Sort by value
Map<String, Integer> sortedByValue =
        map.entrySet()
           .stream()
           .sorted(Map.Entry.comparingByValue())
           .collect(Collectors.toMap(
               Map.Entry::getKey,
               Map.Entry::getValue,
               (a, b) -> a,
               LinkedHashMap::new
           ));
We use LinkedHashMap so that the sorted iteration order is preserved.
Interview answer
HashMap itself does not guarantee ordering. For key sorting I normally use TreeMap. For value sorting I convert the entries to a stream, sort them using a comparator, and collect into LinkedHashMap to preserve the resulting order.
________________________________________
2. Design a Singleton class
A Singleton ensures that only one instance of a class exists and provides a global access point to it.
Best modern approach: Enum Singleton
public enum DatabaseConnection {
    INSTANCE;

    public void connect() {
        System.out.println("Connected");
    }
}
Usage:
DatabaseConnection.INSTANCE.connect();
Advantages:
•	thread-safe 
•	serialization-safe 
•	protects against reflection-based multiple instances more strongly than common constructor-based implementations 
________________________________________
Double-checked locking
public class Singleton {

    private static volatile Singleton instance;

    private Singleton() {
    }

    public static Singleton getInstance() {

        if (instance == null) {

            synchronized (Singleton.class) {

                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }

        return instance;
    }
}
Why volatile?
Because object creation is not necessarily one indivisible operation:
1. Allocate memory
2. Initialize object
3. Assign reference
Without volatile, instruction reordering could expose a partially initialized object to another thread.
Interview trap
Double-checked locking without volatile is unsafe under the Java Memory Model.
________________________________________
3. Comparable vs Comparator
Comparable
Used when the class has a natural ordering.
class Employee implements Comparable<Employee> {

    int salary;

    Employee(int salary) {
        this.salary = salary;
    }

    @Override
    public int compareTo(Employee other) {
        return Integer.compare(this.salary, other.salary);
    }
}
Sorting:
Collections.sort(employees);
Comparable defines:
compareTo()
________________________________________
Comparator
Used when you want an external/custom ordering.
employees.sort(
    Comparator.comparing(Employee::getName)
);
You can have multiple comparators:
Comparator<Employee> bySalary = 
    Comparator.comparing(Employee::getSalary);

Comparator<Employee> byName =
    Comparator.comparing(Employee::getName);
Difference
Comparable	Comparator
compareTo()	compare()
Natural ordering	Custom ordering
Class implements it	Separate object/lambda
Usually one natural order	Multiple sorting strategies
Important TreeSet point
If compareTo() says two objects are equal:
compareTo() == 0
TreeSet treats them as duplicates even if equals() says they are different.
________________________________________
4. Java 7, 8, 11 and 17 features
Java 7
Important features:
Diamond operator
List<String> names = new ArrayList<>();
Try-with-resources
try (BufferedReader br =
         new BufferedReader(new FileReader("test.txt"))) {
    System.out.println(br.readLine());
}
Multi-catch
try {
    // code
} catch (IOException | SQLException e) {
    e.printStackTrace();
}
Strings in switch
switch (status) {
    case "SUCCESS":
        break;
    case "FAILED":
        break;
}
________________________________________
Java 8
Java 8 was a major milestone.
Lambda
List<String> names = List.of("A", "B", "C");

names.forEach(name -> System.out.println(name));
Functional interfaces
@FunctionalInterface
interface Calculator {
    int add(int a, int b);
}
Streams
List<Integer> result =
    numbers.stream()
           .filter(x -> x % 2 == 0)
           .map(x -> x * 2)
           .toList();
Default methods
interface Vehicle {

    default void start() {
        System.out.println("Starting");
    }
}
Optional
Optional<String> name = Optional.ofNullable(value);
Method references
names.forEach(System.out::println);
New Date/Time API
LocalDate today = LocalDate.now();
LocalDateTime now = LocalDateTime.now();
________________________________________
Java 11
Important additions:
String methods
" ".isBlank();
"hello".lines();
"hello".repeat(3);
" hello ".strip();
HTTP Client
HttpClient client = HttpClient.newHttpClient();
Java 11 introduced the modern standard HTTP Client API.
var in lambda parameters
(var x, var y) -> x + y
________________________________________
Java 17
Java 17 is important because it is an LTS release.
Important language/platform features available by Java 17 include:
Records
public record Employee(
    long id,
    String name,
    double salary
) {}
The compiler generates components/accessors, constructor, equals, hashCode, and toString.
Usage:
Employee e = new Employee(1, "Aayush", 100000);

System.out.println(e.name());
Not:
e.getName();
unless you define such a method yourself.
Sealed classes
public sealed class Payment
    permits CardPayment, UpiPayment {
}
Only permitted subclasses may extend it.
Pattern matching for instanceof
if (obj instanceof String s) {
    System.out.println(s.length());
}
Text blocks
String json = """
        {
          "name": "John"
        }
        """;
Switch expressions
Modern switch expressions can return values:
String result = switch (status) {
    case SUCCESS -> "Completed";
    case FAILED -> "Failed";
    default -> "Unknown";
};
Important Java 11 → 17 interview point
Do not say Java 17 introduced everything above. Some features were introduced in earlier releases and were already final by Java 17. Interviewers often test whether you know the difference between introduced, preview, and finalized features.
________________________________________
5. Try-with-resources
Try-with-resources automatically closes resources implementing AutoCloseable.
try (BufferedReader reader =
         new BufferedReader(new FileReader("file.txt"))) {

    String line = reader.readLine();

} catch (IOException e) {
    e.printStackTrace();
}
Equivalent conceptually to manually calling close() in a finally, but safer.
Advantages
1.	Automatic resource closing 
2.	Cleaner code 
3.	Prevents resource leaks 
4.	Supports multiple resources 
5.	Correctly handles suppressed exceptions 
Example:
try (
    Connection con = dataSource.getConnection();
    PreparedStatement ps = con.prepareStatement(sql)
) {
    ...
}
________________________________________
6. Multi-catch
Java 7 introduced multi-catch.
try {
    // code
} catch (IOException | SQLException e) {
    log.error("Operation failed", e);
}
Both exception types share the same handling logic.
You cannot write:
catch (IOException | Exception e)
because IOException is already a subtype of Exception.
________________________________________
7. Runnable vs Callable
Runnable
Runnable task = () -> {
    System.out.println("Running");
};
Method:
void run()
Cannot directly return a result or throw checked exceptions.
________________________________________
Callable
Callable<Integer> task = () -> {
    return 100;
};
Method:
V call() throws Exception
Can:
•	return a result 
•	throw checked exceptions 
Example:
ExecutorService executor =
    Executors.newFixedThreadPool(5);

Future<Integer> future =
    executor.submit(() -> 10 + 20);

System.out.println(future.get());
________________________________________
8. Runnable vs Callable vs Future
Think of them differently:
Runnable  = task
Callable  = task + result
Future    = handle/reference to async result
Example:
Future<Integer> future =
    executor.submit(() -> 100);
future.get() waits for the result.
Problems with Future:
•	blocking get() 
•	difficult composition 
•	difficult error pipelines 
•	difficult combining of multiple async operations 
That's one reason CompletableFuture became important.
________________________________________
9. Exception hierarchy
At the top:
Object
  |
Throwable
  |
  +-- Error
  |
  +-- Exception
       |
       +-- RuntimeException
Error
Usually serious JVM/system-level conditions.
Examples:
OutOfMemoryError
StackOverflowError
NoClassDefFoundError
Applications generally don't try to recover from most Errors.
________________________________________
Exception
Conditions applications can potentially handle.
Checked exception
Compiler forces handling/declaration.
IOException
SQLException
Example:
void read() throws IOException {
}
Unchecked exception
Subclass of RuntimeException.
NullPointerException
IllegalArgumentException
IndexOutOfBoundsException
Compiler does not force explicit handling.
Interview answer
Checked exceptions are usually used for recoverable conditions that callers are expected to handle, while unchecked exceptions generally represent programming errors or invalid state. Modern Spring applications often prefer unchecked exceptions and centralized exception handling.
________________________________________
10. OOP principles
Encapsulation
Hide internal state.
class Account {

    private double balance;

    public void deposit(double amount) {
        if (amount > 0) {
            balance += amount;
        }
    }
}
________________________________________
Abstraction
Expose what something does, hide how it does it.
interface PaymentService {
    void pay();
}
________________________________________
Inheritance
Reuse/extend behavior.
class Animal {
    void sound() {}
}

class Dog extends Animal {
}
________________________________________
Polymorphism
Same interface/reference, different implementation.
Animal animal = new Dog();
animal.sound();
At runtime, overridden method resolution chooses Dog.sound().
________________________________________
11. Interface vs Abstract class
Interface
Best for a contract/capability.
interface Payment {
    void pay();
}
A modern interface can contain:
•	abstract methods 
•	default methods 
•	static methods 
•	private methods 
________________________________________
Abstract class
Best when classes share:
•	state 
•	common implementation 
•	protected methods 
•	constructor logic 
abstract class Vehicle {

    protected String brand;

    Vehicle(String brand) {
        this.brand = brand;
    }

    abstract void move();

    void stop() {
        System.out.println("Stopped");
    }
}
Interview answer
I use an interface when I primarily want a contract and multiple unrelated classes should implement it. I use an abstract class when I want to provide shared state or common implementation.
________________________________________
12. Can interfaces have private methods?
Yes.
Private interface methods were introduced in Java 9.
interface Payment {

    default void process() {
        validate();
        save();
    }

    private void validate() {
        System.out.println("Validation");
    }

    private void save() {
        System.out.println("Save");
    }
}
Private methods are useful for sharing implementation between default/static methods without exposing them as public API.
________________________________________
13. Two interfaces have the same default method
Example:
interface A {
    default void print() {
        System.out.println("A");
    }
}

interface B {
    default void print() {
        System.out.println("B");
    }
}

class Test implements A, B {
}
Compilation fails because Java cannot decide which default method to use.
Resolve explicitly:
class Test implements A, B {

    @Override
    public void print() {
        A.super.print();
    }
}
________________________________________
14. SOLID principles
S — Single Responsibility Principle
A class should have one primary reason to change.
Bad:
EmployeeService
    calculateSalary()
    generatePDF()
    sendEmail()
    saveToDatabase()
Better separation:
SalaryService
ReportService
EmailService
EmployeeRepository
________________________________________
O — Open/Closed Principle
Open for extension, closed for modification.
Instead of:
if (paymentType.equals("CARD")) ...
else if (paymentType.equals("UPI")) ...
design abstractions:
interface PaymentProcessor {
    void process();
}
Then create:
CardPaymentProcessor
UPIPaymentProcessor
WalletPaymentProcessor
________________________________________
L — Liskov Substitution Principle
Subclass should be usable wherever parent is expected without breaking expected behavior.
Classic bad example:
Bird
  |
  +-- Penguin
If base class requires:
bird.fly();
Penguin cannot satisfy that contract.
The abstraction itself may be wrong.
________________________________________
I — Interface Segregation Principle
Don't force clients to depend on methods they don't need.
Bad:
interface Worker {
    work();
    eat();
    sleep();
}
Separate contracts where appropriate.
________________________________________
D — Dependency Inversion Principle
High-level modules should depend on abstractions rather than concrete implementations.
Instead of:
class OrderService {
    private MySqlRepository repo = new MySqlRepository();
}
use:
class OrderService {

    private final OrderRepository repository;

    OrderService(OrderRepository repository) {
        this.repository = repository;
    }
}
Now Spring can inject different implementations.
________________________________________
15. Immutable class design
A truly immutable class should ensure its state cannot change after construction.
Example:
public final class Employee {

    private final String name;
    private final List<String> skills;

    public Employee(String name, List<String> skills) {
        this.name = name;
        this.skills = List.copyOf(skills);
    }

    public String getName() {
        return name;
    }

    public List<String> getSkills() {
        return skills;
    }
}
Important rules:
1.	Make class final when inheritance could compromise immutability. 
2.	Fields should normally be private final. 
3.	Initialize everything in constructor. 
4.	No setters. 
5.	Do defensive copying of mutable inputs. 
6.	Don't return internal mutable references. 
Bad:
this.skills = skills;
Then external code can do:
skills.clear();
and modify the object's internal state.
________________________________________
16. equals() and hashCode()
Contract:
If:
a.equals(b) == true
then:
a.hashCode() == b.hashCode()
The reverse is NOT required.
Two unequal objects can absolutely have the same hash code.
That is a hash collision.
________________________________________
Why it matters for HashMap
HashMap roughly works like:
key
 ↓
hashCode()
 ↓
bucket
 ↓
equals()
If you violate the contract, lookup can behave unexpectedly.
Example:
Map<Employee, String> map = new HashMap<>();
Suppose Employee is inserted using one state, then the fields involved in equality/hash are changed.
The object may effectively belong to a bucket based on its old hash but later generate a new hash.
Then:
map.get(employee)
may fail.
Best practice
Keys should generally be immutable.
________________________________________
17. Can HashMap keys be mutable?
Technically yes.
But it is dangerous if fields used in equals()/hashCode() change after insertion.
Bad:
class Employee {
    String id;

    // hashCode uses id
}
Then:
map.put(employee, "value");

employee.id = "NEW";

map.get(employee); // may return null
Interview answer
HashMap allows mutable objects as keys, but keys should ideally be immutable. If the state participating in hashCode changes after insertion, the object may no longer be found in the expected bucket.
________________________________________
18. How HashMap works internally
Conceptually:
HashMap
   |
   +-- array of buckets
          |
          +-- Node
          +-- Node
          +-- TreeNode
A hash is calculated from the key.
Then bucket index is determined from the hash and table size.
Collision happens when multiple keys map to the same bucket.
Java 8+ can convert a long collision chain into a tree under certain conditions, improving worst-case lookup behavior from approximately:
O(n)
toward:
O(log n)
for tree bins, subject to implementation thresholds and assumptions.
Average expected get() / put() is approximately:
O(1)
________________________________________
19. What is rehashing?
When HashMap becomes sufficiently full according to its load factor, it resizes the internal table.
Typical default load factor:
0.75
When resizing occurs, capacity usually grows approximately by 2×.
Example:
16
 ↓
32
 ↓
64
 ↓
128
Entries have to be redistributed into the new bucket arrangement.
That makes resizing expensive.
Important interview point
HashMap does not simply blindly calculate % newCapacity for every entry in the naive sense; modern Java implementations exploit the power-of-two table size and hash bits to efficiently determine whether an entry stays in the same bucket or moves by the old capacity.
________________________________________
20. HashSet internal working
HashSet is backed by a HashMap.
Conceptually:
HashMap<E, Object>
When you do:
set.add("A");
internally the value is stored in a backing map with a common dummy value.
Duplicate detection depends on:
hashCode()
+
equals()
Rule
If:
hashCode()
differs, objects go to different buckets.
If hash codes are the same, equals() is used to determine whether the elements are actually equal.
________________________________________
21. HashMap vs ConcurrentHashMap
HashMap
Not thread-safe.
Multiple threads can modify it concurrently and produce race conditions.
ConcurrentHashMap
Designed for concurrent access.
Modern implementations avoid a single global lock around the entire map.
They use techniques including:
•	volatile reads 
•	CAS 
•	fine-grained synchronization on specific bins when needed 
•	internal coordination during resizing 
This allows different threads to operate on different regions/bins concurrently.
Important correction
Don't say:
ConcurrentHashMap has one lock per bucket.
That is an oversimplification and not an accurate description of the modern implementation.
________________________________________
22. Why is HashMap unsafe even for concurrent reads?
If the map is never modified after safe publication, concurrent reads can be fine.
The issue is when writes/resizing occur concurrently with reads.
Without proper synchronization, the Java Memory Model does not provide the visibility/order guarantees you need.
Therefore this statement is better:
HashMap is not designed as a concurrent data structure. Concurrent reads of a safely published, never-modified map can be okay, but concurrent mutation requires synchronization or a concurrent collection.
________________________________________
23. Fail-fast vs fail-safe
Example:
List<Integer> list =
    new ArrayList<>(List.of(1, 2, 3));

for (Integer x : list) {
    list.remove(x);
}
This can cause:
ConcurrentModificationException
The iterator detects structural modification based on modification bookkeeping.
Fail-fast
Examples include standard collection iterators such as ArrayList iterators.
They attempt to detect concurrent structural modification and fail quickly.
CopyOnWriteArrayList
When writing:
list.add(...)
a new underlying array is created.
Readers can iterate over an immutable snapshot of the array they obtained.
Advantages:
•	excellent for many readers 
•	safe iteration 
Disadvantages:
•	expensive writes 
•	increased memory usage 
•	not suitable for write-heavy scenarios 
________________________________________
24. String vs StringBuilder vs StringBuffer
String
Immutable.
String s = "hello";
s = s + " world";
A new String object may be created.
________________________________________
StringBuilder
Mutable and generally faster for single-threaded string construction.
StringBuilder sb = new StringBuilder();

sb.append("Hello");
sb.append(" World");
Not synchronized.
________________________________________
StringBuffer
Mutable and synchronized.
Historically useful when thread-safe mutable character sequences were needed, but often has more synchronization overhead than StringBuilder.
Interview answer
I normally use String for immutable text, StringBuilder for repeated modifications in a single-threaded context, and StringBuffer only when its synchronization characteristics are actually needed.
________________________________________
25. Optional
Optional<T> represents a value that may or may not exist.
Instead of:
String name = null;
we can have:
Optional<String> name;
Example:
Optional<String> name =
    Optional.ofNullable(getName());

name.ifPresent(System.out::println);
Or:
String result =
    name.orElse("Unknown");
Do not abuse Optional
A common interview question is:
Why is Optional not recommended as a field?
Because it is primarily intended to represent optional return values, and using it everywhere as a field/parameter can add unnecessary complexity and integration overhead.
Typical good usage:
Optional<User> findUserById(Long id)
rather than blindly wrapping every object.
________________________________________
26. Java Memory Model
The Java Memory Model defines how threads interact through memory.
It answers questions such as:
•	when writes become visible 
•	what ordering guarantees exist 
•	what data races mean 
•	what happens-before relationships exist 
Main memory conceptually:
Thread 1
  |
  +-- local working state

Thread 2
  |
  +-- local working state

       ↓

   Shared memory
The JMM prevents us from assuming that every thread instantly observes every write.
________________________________________
27. What does volatile do?
private volatile boolean running = true;
A volatile variable provides important visibility and ordering guarantees.
When one thread writes:
running = false;
another thread reading it can reliably observe the latest volatile write under the JMM rules.
Important
volatile does NOT make arbitrary compound operations atomic.
This is not safely incremented merely because the variable is volatile:
volatile int count;

count++;
count++ is effectively:
read
+
add
+
write
Two threads can still lose updates.
Use:
AtomicInteger
or locking when needed.
________________________________________
28. volatile vs Atomic classes
volatile
Provides:
•	visibility 
•	ordering guarantees 
Doesn't make compound read-modify-write operations atomic.
AtomicInteger
AtomicInteger counter =
    new AtomicInteger();

counter.incrementAndGet();
Uses atomic operations such as CAS where appropriate.
Rule of thumb
Use volatile for simple state flags or publication scenarios.
Use atomic classes when you need atomic updates.
________________________________________
29. synchronized vs ReentrantLock
synchronized
synchronized void process() {
}
Advantages:
•	simple 
•	automatic lock release 
•	integrated language feature 
________________________________________
ReentrantLock
Lock lock = new ReentrantLock();

lock.lock();

try {
    process();
} finally {
    lock.unlock();
}
Provides additional features:
tryLock()
lockInterruptibly()
and optional fairness configuration.
Trade-off
synchronized is simpler and less error-prone.
ReentrantLock gives more control but requires explicit unlock handling.
________________________________________
30. What happens when exception occurs inside synchronized block?
Example:
synchronized (lock) {

    throw new RuntimeException();
}
The monitor is released when execution leaves the synchronized block, including because of an exception.
This is one major advantage over manually managed locks.
With ReentrantLock, you must ensure:
try {
   ...
} finally {
   lock.unlock();
}
________________________________________
31. wait() vs sleep()
sleep()
Thread.sleep(1000);
•	pauses current thread 
•	does not release a monitor it already owns 
•	used for timing/delay 
wait()
synchronized (lock) {
    lock.wait();
}
•	releases the monitor 
•	thread waits until notified/interrupted 
•	must be called while owning the object's monitor 
Interview statement
sleep is a timing mechanism; wait is a coordin
Why must wait() be called inside synchronized?
Because wait() operates on the object's monitor.
Before waiting, the thread must own that monitor:
synchronized (lock) {
    lock.wait();
}
Otherwise:
IllegalMonitorStateException
________________________________________
33. park/unpark vs wait/notify
LockSupport.park() blocks a thread.
LockSupport.unpark(thread) makes it eligible to continue.
These primitives are heavily used underneath higher-level concurrency utilities.
Compared with:
wait()/notify()
park/unpark is associated with a specific thread rather than relying on an object monitor's wait set.
________________________________________
34. Deadlock
Deadlock occurs when threads wait forever for resources held by each other.
Example:
Thread A holds Lock 1
Thread B holds Lock 2

A waits for Lock 2
B waits for Lock 1
Cycle:
A → B → A
Prevention
Use a consistent lock acquisition order.
For example:
Always acquire Account A before Account B
Other techniques:
•	reduce lock scope 
•	avoid nested locks 
•	use tryLock() with timeout 
•	use thread dumps to diagnose 
________________________________________
35. ThreadLocal and memory leaks
ThreadLocal associates data with a thread.
In a thread pool, threads live for a long time.
Suppose:
threadLocal.set(largeObject);
If you forget:
threadLocal.remove();
the worker thread may retain the value much longer than expected.
This can cause memory retention.
Best practice:
try {
    threadLocal.set(value);
    ...
} finally {
    threadLocal.remove();
}
________________________________________
36. What is a daemon thread?
Daemon threads are background threads that don't keep the JVM alive by themselves.
Example:
Thread t = new Thread(task);
t.setDaemon(true);
t.start();
Common conceptual uses:
•	background cleanup 
•	monitoring 
•	housekeeping 
Do not use daemon threads for critical business operations where losing work during JVM shutdown is unacceptable.
________________________________________
37. CompletableFuture
CompletableFuture supports asynchronous pipelines.
Example:
CompletableFuture
    .supplyAsync(() -> getUser())
    .thenApply(user -> user.getName())
    .thenAccept(System.out::println);
Important methods:
thenApply
Transforms a result.
future.thenApply(x -> x * 2);
Conceptually:
T → U
thenCompose
Flattens nested asynchronous operations.
future
    .thenCompose(user ->
        getOrders(user.id())
    );
Without compose:
CompletableFuture<CompletableFuture<List<Order>>>
With compose:
CompletableFuture<List<Order>>
thenCombine
Combines two independent futures.
userFuture.thenCombine(
    orderFuture,
    (user, orders) -> ...
);
exceptionally
Handles failure.
future.exceptionally(ex -> fallback());
handle
Handles both success and failure.
future.handle((result, ex) -> {
    if (ex != null) return fallback();
    return result;
});
________________________________________
38. Why CompletableFuture can hide exceptions
Consider:
CompletableFuture.runAsync(() -> {
    throw new RuntimeException("Failed");
});
If you never inspect or join the future, the exception may not be handled where you expect.
Always define an error path:
future.exceptionally(ex -> {
    log.error("Async failure", ex);
    return null;
});
or:
future.join();
and handle the resulting exception appropriately.
________________________________________
39. Common ForkJoinPool problem
Some async APIs use:
ForkJoinPool.commonPool()
If you perform blocking operations there:
CompletableFuture.supplyAsync(() -> {
    databaseCall();  // blocking
    return result;
});
you can occupy worker threads with blocking tasks.
That can reduce throughput.
For controlled workloads, use a dedicated executor:
ExecutorService executor =
    Executors.newFixedThreadPool(20);

CompletableFuture.supplyAsync(
    () -> databaseCall(),
    executor
);
But pool sizing should be based on workload characteristics, not an arbitrary number.
________________________________________
40. Garbage Collection
Java automatically manages memory.
Conceptually:
Heap
 |
 +-- Young Generation
 |     |
 |     +-- Eden
 |     +-- Survivor
 |
 +-- Old Generation
Objects are allocated primarily in young regions/generations depending on the collector.
Short-lived objects often die young.
Objects surviving collections may eventually be promoted/aged into older areas.
________________________________________
41. How GC knows an object is unreachable
GC does not simply ask:
"Was this object used recently?"
It determines reachability from GC roots.
Examples of GC roots can include:
•	active thread stacks 
•	static references 
•	JNI references 
•	other VM roots 
Conceptually:
GC Root
  ↓
Object A
  ↓
Object B
These are reachable.
But:
Object C → Object D
with no path from a root:
unreachable
They can be reclaimed.
________________________________________
42. Minor GC, Major GC, Full GC
Terminology varies by collector and Java version, so avoid presenting these terms as universal rigid categories.
Historically:
Minor GC
Young-generation collection.
Usually relatively frequent and focused on young objects.
Major GC
Often referred to as old-generation collection.
Full GC
Broader collection involving much more of the heap and potentially other memory areas/processing.
Important interview point
The exact behavior depends on the collector.
For modern Java, it is better to understand the specific collector, especially G1 or ZGC, rather than relying only on old terminology.
________________________________________
43. G1 GC
G1 stands for Garbage-First.
Instead of treating the heap as one giant contiguous young/old area in the traditional way, G1 divides the heap into regions.
It tracks information about garbage in regions and attempts to prioritize regions offering high reclamation value relative to pause-cost goals.
High-level flow:
Heap
 |
 +-- Region
 +-- Region
 +-- Region
 +-- Region
G1 aims for predictable pauses rather than simply maximizing raw throughput.
________________________________________
44. ZGC vs G1
G1
Good general-purpose collector with pause-time goals.
ZGC
Designed for very low pause times and large heaps.
It performs much of its work concurrently with application threads.
Interview answer
I choose a GC based on latency requirements, heap size, allocation rate, CPU overhead and application behavior. I don't choose ZGC merely because it is newer.
________________________________________
45. Metaspace
Metaspace stores class metadata in native memory rather than the old PermGen area.
Permanent Generation was removed in Java 8.
If metaspace fills
You can see:
OutOfMemoryError: Metaspace
Possible reasons:
•	excessive class generation 
•	classloader leaks 
•	dynamically generated classes 
•	frameworks generating many classes 
________________________________________
46. Escape analysis
The JIT compiler can analyze whether an object escapes a method/thread.
Example:
void test() {
    Point p = new Point(10, 20);
    calculate(p);
}
If p never escapes the method or thread in a relevant way, JIT optimizations may eliminate allocations or synchronization in appropriate situations.
Important:
Do not say "Java allocates objects on stack whenever escape analysis says they don't escape."
That is an oversimplification.
Escape analysis can enable optimizations such as scalar replacement, where an object may effectively disappear as an allocation and its fields are handled separately.
________________________________________
47. JIT compiler
JIT = Just-In-Time compiler.
Java bytecode initially runs through the JVM execution machinery, while frequently executed ("hot") code can be compiled into native machine code.
JIT can perform optimizations such as:
•	method inlining 
•	dead code elimination 
•	loop optimizations 
•	escape analysis 
•	devirtualization in appropriate cases 
This is why a Java application's performance can change after warm-up.
________________________________________
48. Safepoints
A safepoint is a JVM coordination point where threads can be brought into a state suitable for certain JVM operations.
They are important for operations requiring global coordination, including some GC phases and VM activities.
A common production scenario:
Application latency increases
        ↓
Long JVM pause / safepoint issue
        ↓
p99 latency spikes
Tools such as GC logs and JVM diagnostics help determine whether safepoint activity contributes to latency.
________________________________________
49. finalize()
finalize() is deprecated for removal and should not be used for resource management.
Problems include:
•	unpredictable execution time 
•	performance overhead 
•	resurrection-related complexities 
•	delayed reclamation semantics 
•	difficult debugging 
Use:
try-with-resources
AutoCloseable
explicit cleanup APIs
instead.
________________________________________
50. String immutability
Why is String immutable?
Important reasons include:
Security
Strings are frequently used for:
•	paths 
•	class names 
•	network addresses 
•	credentials/configuration 
Hashing
String hash codes can be cached.
String pool
Interned strings can safely be shared because they cannot be modified.
Thread safety
Immutable objects are naturally safe to share.
________________________________________
51. Why Java is pass-by-value
Java is always pass-by-value.
For objects, the value being passed is the reference value.
Example:
void change(Person p) {
    p.name = "Bob";
}
The object can be mutated.
But:
void change(Person p) {
    p = new Person("Bob");
}
doesn't make the caller's variable point to the new object.
Why?
Because the copy of the reference was passed.
Conceptually:
caller reference ─────→ Object A
       |
       +--- copied reference ---> method parameter
Both initially point to Object A.
Reassigning the parameter changes only the local copy.
________________________________________
52. Spring Framework
Spring is a framework for building loosely coupled applications using:
•	IoC 
•	Dependency Injection 
•	AOP 
•	transaction management 
•	MVC/Web 
•	data integration 
•	security integration 
Core idea:
Instead of:
OrderService service =
    new OrderService(
        new PaymentService()
    );
Spring creates and wires objects for you.
________________________________________
53. IoC vs Dependency Injection
IoC
Inversion of Control is the broader principle:
The framework controls object creation/lifecycle rather than application code doing everything manually.
DI
Dependency Injection is one way IoC is implemented.
Example:
@Service
class OrderService {

    private final PaymentService paymentService;

    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
Spring constructs OrderService and injects PaymentService.
________________________________________
54. Types of Dependency Injection
Three common forms:
Constructor injection
Preferred.
@Service
class UserService {

    private final UserRepository repo;

    UserService(UserRepository repo) {
        this.repo = repo;
    }
}
Advantages:
•	immutable dependency 
•	easier testing 
•	dependencies explicit 
•	object cannot be constructed without required dependency 
Setter injection
@Autowired
public void setRepo(UserRepository repo) {
    this.repo = repo;
}
Useful for optional dependencies in some cases.
Field injection
@Autowired
private UserRepository repo;
Works, but is generally less preferred because dependency requirements are hidden and testing becomes less convenient.
________________________________________
55. Spring Container internal working
At a high level:
Application starts
       ↓
Spring creates ApplicationContext
       ↓
Reads configuration/components
       ↓
Creates BeanDefinitions
       ↓
Resolves dependencies
       ↓
Instantiates beans
       ↓
Applies bean post-processors
       ↓
Runs initialization callbacks
       ↓
Bean ready
The key container interfaces/components include concepts such as:
•	BeanFactory 
•	ApplicationContext 
•	BeanDefinition 
•	BeanPostProcessor 
________________________________________
56. Spring Bean lifecycle
Conceptually:
BeanDefinition
    ↓
Instantiation
    ↓
Dependency Injection
    ↓
Aware callbacks
    ↓
BeanPostProcessor before initialization
    ↓
@PostConstruct
    ↓
InitializingBean / init-method
    ↓
BeanPostProcessor after initialization
    ↓
Ready
    ↓
@PreDestroy / destroy callback
Example:
@PostConstruct
public void init() {
    System.out.println("Initialized");
}

@PreDestroy
public void cleanup() {
    System.out.println("Cleanup");
}
________________________________________
57. @Component vs @Service vs @Repository vs @Controller
Technically these are stereotype annotations.
@Component
Generic Spring-managed component.
@Service
Semantically represents service/business logic.
@Repository
Represents persistence/data-access logic.
It also participates in persistence exception translation.
@Controller
Spring MVC controller.
Returns views unless otherwise configured.
@RestController
Effectively:
@Controller
@ResponseBody
Used primarily for REST APIs.
________________________________________
58. @Configuration
@Configuration
public class AppConfig {

    @Bean
    public PaymentClient paymentClient() {
        return new PaymentClient();
    }
}
It declares configuration and usually contains bean definitions.
Spring enhances configuration classes as necessary so that @Bean semantics are respected, including managed references between configuration methods in the standard configuration style.
________________________________________
59. @Bean vs @Component
@Component
Used on a class:
@Component
class PaymentService {
}
Spring discovers it through component scanning.
@Bean
Used on a method:
@Bean
public ObjectMapper objectMapper() {
    return new ObjectMapper();
}
Useful when:
•	integrating third-party classes 
•	creating objects with custom construction logic 
•	wanting explicit configuration 
________________________________________
60. @Primary vs @Qualifier
Suppose:
interface PaymentService {}
and:
CardPaymentService
UpiPaymentService
Both are beans.
@Primary
Marks one as the default.
@Primary
@Component
class CardPaymentService {}
@Qualifier
Explicitly identifies which bean you want.
public PaymentController(
    @Qualifier("upiPaymentService")
    PaymentService paymentService) {
}
Think:
@Primary   → default choice
@Qualifier → specific choice
________________________________________
61. Spring Boot auto-configuration
Spring Boot tries to configure the application based on:
•	classpath contents 
•	existing beans 
•	properties 
•	environment 
Example:
If Spring Boot detects web-related dependencies, it can configure web infrastructure.
@EnableAutoConfiguration enables the auto-configuration mechanism.
Spring Boot uses auto-configuration metadata to determine which configurations are candidates, and conditional annotations decide whether they should actually be applied.
Typical condition:
@ConditionalOnMissingBean
Meaning:
configure this default only when the application hasn't already provided a bean.
________________________________________
62. Can Spring Boot run without Tomcat?
Yes.
Spring Boot supports different web application types/stacks.
For traditional Spring MVC applications, Tomcat is the common default embedded servlet container.
But you can use alternatives such as:
•	Jetty 
•	Undertow 
or use a reactive stack such as WebFlux, commonly with Netty.
You can also build non-web applications.
________________________________________
63. Why does an executable JAR need Tomcat?
Executable JAR and Tomcat solve different problems.
The executable JAR packages:
Your application
+
dependencies
+
bootstrapping
For a servlet-based application, an embedded servlet container is still required to receive HTTP requests.
So:
Executable JAR
      +
Embedded Tomcat
      =
Standalone HTTP application
You don't need to install external Tomcat separately.
________________________________________
64. Spring Profiles
Profiles allow environment-specific configuration.
application.yml
application-dev.yml
application-qa.yml
application-prod.yml
Example:
spring:
  profiles:
    active: qa
You may also conditionally create beans:
@Profile("prod")
@Bean
public PaymentClient productionClient() {
    ...
}
Typical usage:
dev → local DB
qa → QA DB
prod → production DB
________________________________________
65. @Transactional
@Transactional defines transaction boundaries.
@Transactional
public void transferMoney() {
    debit();
    credit();
}
Conceptually:
Begin transaction
      ↓
Business operations
      ↓
Commit
If a suitable exception causes rollback:
Rollback
Spring generally uses proxies/interceptors around the target bean method.
________________________________________
66. Why @Transactional doesn't work on internal method calls
Example:
class PaymentService {

    public void outer() {
        inner();
    }

    @Transactional
    public void inner() {
    }
}
The call:
inner();
is an internal call on the same object, commonly called self-invocation.
Spring's proxy is bypassed.
Therefore the transactional interceptor may not run.
Solutions
•	move method to another bean 
•	call through a proxy where appropriate 
•	redesign transactional boundaries 
Don't casually recommend manually obtaining the current proxy unless there's a strong reason.
________________________________________
67. Transaction propagation
REQUIRED
Default.
If transaction exists:
join existing
Otherwise:
create new
________________________________________
REQUIRES_NEW
Suspends existing transaction and starts a new transaction.
Useful when a secondary operation must commit/rollback independently.
Example:
Main payment transaction
       ↓
Audit transaction REQUIRES_NEW
Audit can commit even when the main operation rolls back, depending on implementation and actual semantics.
________________________________________
NESTED
Creates a nested transaction-like boundary using savepoints when supported.
It is not equivalent to REQUIRES_NEW.
________________________________________
68. Checked vs unchecked rollback in Spring
By default, Spring transaction rollback rules historically apply to:
RuntimeException
Error
Checked exceptions do not automatically trigger rollback unless configured.
Example:
@Transactional(
    rollbackFor = IOException.class
)
Then an IOException can trigger rollback according to that rule.
________________________________________
69. Spring Security
Spring Security handles concerns such as:
•	authentication 
•	authorization 
•	security filters 
•	CSRF protection 
•	session security 
•	method security 
•	OAuth2 / JWT resource-server patterns 
Typical request flow:
Client
  ↓
Security Filter Chain
  ↓
Authentication
  ↓
Authorization
  ↓
Controller
________________________________________
70. JWT authentication
JWT commonly contains:
Header
Payload
Signature
Example conceptual payload:
{
  "sub": "12345",
  "roles": ["USER"],
  "exp": 1750000000
}
Typical flow:
Login
 ↓
Authentication
 ↓
JWT generated
 ↓
Client stores token
 ↓
Client sends Authorization: Bearer <token>
 ↓
Spring Security validates token
 ↓
Request authorized
Important
A JWT is generally signed, not inherently encrypted.
Do not store sensitive secrets in the JWT payload simply because it is encoded.
________________________________________
71. REST vs SOAP
REST
Architectural style based around resources and HTTP.
Commonly:
GET
POST
PUT
PATCH
DELETE
Usually JSON is used.
Advantages:
•	lightweight 
•	simple 
•	web-friendly 
•	easy integration 
________________________________________
SOAP
Protocol with XML messaging and formal service contracts.
Often associated with:
•	WSDL 
•	XML 
•	WS-* standards 
Can be valuable where strong formal contracts and enterprise WS-* features are important.
Interview answer
REST is generally simpler and more web-native for modern APIs, while SOAP provides a highly standardized protocol ecosystem that is still useful in some enterprise and legacy environments.
________________________________________
72. HTTP vs HTTPS
HTTP:
Client → Server
Data is normally transmitted without TLS encryption.
HTTPS:
HTTP + TLS
Provides:
•	confidentiality 
•	integrity 
•	server authentication through certificates 
Typical:
https://example.com
________________________________________
73. Path Variable vs Request Parameter
Path variable:
GET /users/123
@GetMapping("/users/{id}")
public User get(@PathVariable Long id)
Used to identify a resource.
Request parameter:
GET /users?status=ACTIVE&page=2
@GetMapping("/users")
public List<User> get(
    @RequestParam String status) {
}
Used for filtering/options/pagination.
________________________________________
74. Lazy vs Eager loading
In JPA/Hibernate:
Lazy
Associated data is loaded when accessed.
Eager
Associated data is loaded immediately according to the mapping/provider behavior.
Lazy loading can reduce unnecessary database access.
But careless lazy relationships can produce:
N+1 query problem
________________________________________
75. N+1 query problem
Suppose:
List<Order> orders = repository.findAll();
Then for each order:
order.getItems();
You might get:
1 query for orders
+
N queries for items
So total:
N + 1 queries
Possible solutions:
•	fetch joins 
•	entity graphs 
•	batch fetching 
•	carefully designed queries 
•	DTO projections 
Do not blindly make everything eager. That can create much worse problems.
________________________________________
76. SQL index
An index is a data structure that helps the database locate rows more efficiently.
Without index:
Scan many/all rows
With appropriate index:
Use index
 ↓
Locate matching entries
 ↓
Fetch rows
Example:
CREATE INDEX idx_user_email
ON users(email);
Then:
SELECT *
FROM users
WHERE email = 'a@example.com';
may become significantly faster.
Cost
Indexes also:
•	consume storage 
•	increase insert/update/delete overhead 
•	need maintenance 
So:
Don't index every column.
________________________________________
77. Detect duplicate records in SQL
Suppose:
users(email)
Find duplicate emails:
SELECT email, COUNT(*)
FROM users
GROUP BY email
HAVING COUNT(*) > 1;
To return complete duplicate rows:
SELECT *
FROM users
WHERE email IN (
    SELECT email
    FROM users
    GROUP BY email
    HAVING COUNT(*) > 1
);
________________________________________
78. JOIN performance
Example:
SELECT *
FROM orders o
JOIN customers c
  ON o.customer_id = c.id;
Indexes may help join performance, especially on relevant columns.
Usually:
Primary/unique key on customers.id
+
appropriate index on orders.customer_id
can help the optimizer.
But the actual plan depends on:
•	data distribution 
•	cardinality 
•	statistics 
•	indexes 
•	predicates 
•	join algorithm 
•	DB engine 
Always inspect:
EXPLAIN
or the database's equivalent.
________________________________________
79. UNION vs UNION ALL
UNION
Combines results and removes duplicates.
SELECT city FROM customers
UNION
SELECT city FROM suppliers;
UNION ALL
Combines results without duplicate removal.
SELECT city FROM customers
UNION ALL
SELECT city FROM suppliers;
UNION ALL is often faster because it doesn't perform duplicate elimination.
________________________________________
80. ACID
Atomicity
Transaction is all-or-nothing.
Debit
+
Credit
Either both succeed or transaction rolls back.
________________________________________
Consistency
Transaction moves DB from one valid state to another valid state according to defined constraints/rules.
________________________________________
Isolation
Concurrent transactions should not incorrectly interfere with one another.
________________________________________
Durability
After successful commit, data should survive failures according to the database's durability guarantees.
________________________________________
81. Optimistic vs Pessimistic locking
Optimistic
Assumes conflicts are relatively uncommon.
Common implementation:
version column
Example:
id | balance | version
1  | 1000    | 5
Update:
UPDATE account
SET balance = 900,
    version = 6
WHERE id = 1
  AND version = 5;
If zero rows update, someone else changed it.
Good for:
•	many reads 
•	relatively low conflicts 
________________________________________
Pessimistic
Locks data while operating.
Conceptually:
SELECT ...
FOR UPDATE;
Other transactions may have to wait.
Good where conflicts are expensive and strong coordination is necessary.
________________________________________
82. SQL vs NoSQL
SQL
Examples:
PostgreSQL
MySQL
Oracle
Strengths:
•	relational modeling 
•	joins 
•	ACID transactions 
•	complex queries 
•	strong constraints 
NoSQL
Examples:
MongoDB
DynamoDB
Cosmos DB
Cassandra
Redis
Strengths vary by database:
•	horizontal scalability 
•	flexible schemas 
•	high-throughput access patterns 
•	specialized data models 
Interview answer
Don't say:
SQL is for small data and NoSQL is for huge data.
That is incorrect.
Choose based on:
access patterns
consistency
transaction requirements
data relationships
scalability
latency
operational requirements
________________________________________
83. Longest Subarray Sum = K
Use prefix sum + HashMap.
Example:
arr = [1, 2, 3, -2, 5]
k = 3
Maintain:
prefix sum
If:
prefix - k
was seen before, then there is a subarray summing to k.
Java
public static int longestSubarray(int[] arr, int k) {

    Map<Integer, Integer> firstIndex = new HashMap<>();

    int prefix = 0;
    int maxLen = 0;

    firstIndex.put(0, -1);

    for (int i = 0; i < arr.length; i++) {

        prefix += arr[i];

        if (firstIndex.containsKey(prefix - k)) {
            maxLen = Math.max(
                maxLen,
                i - firstIndex.get(prefix - k)
            );
        }

        firstIndex.putIfAbsent(prefix, i);
    }

    return maxLen;
}
Complexity:
Time: O(n)
Space: O(n)
Why putIfAbsent?
We store the earliest index because the earliest index gives the longest possible subarray.
________________________________________
84. Move zeros to the end
Use two pointers.
public static void moveZeros(int[] arr) {

    int index = 0;

    for (int num : arr) {

        if (num != 0) {
            arr[index++] = num;
        }
    }

    while (index < arr.length) {
        arr[index++] = 0;
    }
}
Example:
[0,1,0,3,12]

→ [1,3,12,0,0]
Complexity:
O(n) time
O(1) extra space
________________________________________
85. Anagram
Two strings are anagrams if they contain the same characters with the same frequencies.
Example:
listen
silent
Use frequency counting.
public static boolean isAnagram(
        String a,
        String b) {

    if (a.length() != b.length()) {
        return false;
    }

    int[] freq = new int[256];

    for (int i = 0; i < a.length(); i++) {
        freq[a.charAt(i)]++;
        freq[b.charAt(i)]--;
    }

    for (int x : freq) {
        if (x != 0) return false;
    }

    return true;
}
Complexity:
O(n)
If full Unicode must be handled robustly, a map/code-point approach may be more appropriate.
________________________________________
86. Palindrome
Two-pointer approach:
public static boolean isPalindrome(String s) {

    int left = 0;
    int right = s.length() - 1;

    while (left < right) {

        if (s.charAt(left) != s.charAt(right)) {
            return false;
        }

        left++;
        right--;
    }

    return true;
}
Example:
madam → true
hello → false
Complexity:
O(n) time
O(1) extra space
________________________________________
87. Longest Common Prefix
Input:
flower
flow
flight
Output:
fl
Approach:
public static String longestCommonPrefix(
        String[] strs) {

    if (strs == null || strs.length == 0) {
        return "";
    }

    String prefix = strs[0];

    for (int i = 1; i < strs.length; i++) {

        while (!strs[i].startsWith(prefix)) {

            prefix =
                prefix.substring(
                    0,
                    prefix.length() - 1
                );

            if (prefix.isEmpty()) {
                return "";
            }
        }
    }

    return prefix;
}
________________________________________
88. Best Time to Buy and Sell Stock
Maintain the minimum price so far.
public static int maxProfit(int[] prices) {

    int minPrice = Integer.MAX_VALUE;
    int maxProfit = 0;

    for (int price : prices) {

        minPrice = Math.min(minPrice, price);

        maxProfit =
            Math.max(maxProfit,
                     price - minPrice);
    }

    return maxProfit;
}
Complexity:
O(n)
time
O(1)
space
________________________________________
89. LRU Cache
LRU = Least Recently Used.
Requirements:
get(key) → O(1)
put(key,value) → O(1)
Use:
HashMap + Doubly Linked List
Map gives direct lookup.
Linked list tracks recency.
Most Recent
   ↓
[A] ↔ [B] ↔ [C]
                ↑
             Least Recent
On get(A):
[A] moves to front
On capacity overflow:
remove tail
Java already provides:
LinkedHashMap
with access-order support, which can simplify an LRU implementation.
________________________________________
90. Merge Intervals
Input:
[1,3]
[2,6]
[8,10]
[15,18]
Output:
[1,6]
[8,10]
[15,18]
Sort by starting point.
Then:
if (current.start <= previous.end)
merge them.
Complexity:
O(n log n)
because of sorting.
________________________________________
91. Coin Change
Goal:
coins = [1,2,5]
amount = 11
Answer:
3
because:
5 + 5 + 1
Dynamic programming:
public static int coinChange(
        int[] coins,
        int amount) {

    int[] dp =
        new int[amount + 1];

    Arrays.fill(dp, amount + 1);

    dp[0] = 0;

    for (int i = 1; i <= amount; i++) {

        for (int coin : coins) {

            if (coin <= i) {
                dp[i] =
                    Math.min(
                        dp[i],
                        dp[i - coin] + 1
                    );
            }
        }
    }

    return dp[amount] > amount
        ? -1
        : dp[amount];
}
Complexity:
O(amount × numberOfCoins)
________________________________________
92. Dijkstra's algorithm
Used for shortest paths from one source when edge weights are non-negative.
Typical implementation:
Graph
  ↓
PriorityQueue
  ↓
Pick smallest known distance
  ↓
Relax neighbors
Example relaxation:
if dist[u] + weight < dist[v]
    update dist[v]
Using adjacency lists and a binary heap:
O((V + E) log V)
approximately, commonly expressed as:
O(E log V)
for connected sparse graphs.
Important interview trap
Dijkstra is not valid for negative edge weights.
________________________________________
93. Combination Sum II
This is backtracking + sorting + duplicate skipping.
Key ideas:
1.	Sort array. 
2.	Choose/not choose. 
3.	Don't reuse the same element index. 
4.	Skip duplicate values at the same recursion depth. 
Example logic:
if (i > start && candidates[i] == candidates[i - 1]) {
    continue;
}
That line is very important.
________________________________________
94. Fraud Detection System — high-level design
Suppose transactions arrive at:
10,000+ TPS
Architecture:
Client
   ↓
API Gateway
   ↓
Transaction Service
   ↓
Event Stream
   ↓
Fraud Detection
   ├── Rule Engine
   ├── ML Model
   ├── User Profile
   ├── Historical Features
   └── Risk Database
Possible processing:
Transaction
   ↓
Validate
   ↓
Idempotency
   ↓
Enrich
   ↓
Risk score
   ↓
Decision
   ├── APPROVE
   ├── DECLINE
   └── REVIEW
Important interview topics
You should discuss:
•	latency requirement 
•	false positives 
•	false negatives 
•	rule engine 
•	ML scoring 
•	feature store 
•	event streaming 
•	data retention 
•	auditability 
•	exactly-once misconceptions 
•	duplicate events 
•	retry 
•	idempotency 
•	observability 
________________________________________
95. Payment Processing System
This is one of the most important questions in your list.
Don't answer:
User → REST API → DB
at SDE-2 level.
Start with requirements.
Functional
•	create payment 
•	authorize 
•	capture 
•	refund 
•	status 
•	reconciliation 
Non-functional
•	high availability 
•	high throughput 
•	low latency 
•	no double charge 
•	durability 
•	auditability 
•	security 
________________________________________
Architecture
                ┌───────────────┐
Client ───────→ │ API Gateway   │
                └───────┬───────┘
                        ↓
                ┌───────────────┐
                │ Payment API   │
                └───────┬───────┘
                        ↓
               Idempotency Store
                 (Redis/DB)
                        ↓
                Payment Service
                        ↓
              ┌─────────┴─────────┐
              ↓                   ↓
        Transaction DB          Kafka
                                  ↓
                       Async consumers/events
                                  ↓
                       Notification / analytics
Duplicate payment
Client sends:
Idempotency-Key: abc123
Server stores:
abc123 → payment result
If same request arrives:
same key
   ↓
return existing result
rather than charging again.
External payment failure
You need a payment state machine:
CREATED
   ↓
PROCESSING
   ↓
AUTHORIZED
   ↓
CAPTURED
Failure states:
FAILED
CANCELLED
PENDING_RECONCILIATION
Important correction
Do not claim:
Saga automatically rolls back money.
A Saga provides a sequence of local transactions and compensating actions. Payment systems often require reconciliation, durable state, provider callbacks, and careful financial controls because a real external financial operation may not be perfectly reversible.
________________________________________
96. Saga vs 2PC
2PC
Two-phase commit:
Prepare
  ↓
Commit
Provides distributed transaction coordination but can be operationally expensive and introduce availability/performance concerns.
Saga
A business transaction becomes a sequence of local transactions.
Example:
Create Order
   ↓
Reserve Inventory
   ↓
Charge Payment
   ↓
Confirm Order
If payment fails:
Release Inventory
Cancel Order
That is a compensating action.
Interview answer
Saga is usually more suitable than 2PC for loosely coupled microservices because each service owns its local transaction, but it introduces eventual consistency and compensation complexity.
________________________________________
97. Kafka and event-driven architecture
Kafka is a distributed event streaming platform.
Basic concepts:
Producer
   ↓
Topic
   ↓
Partitions
   ↓
Consumer Group
Example:
Payment Service
      ↓
 payment-events
      ↓
 ├── Fraud Service
 ├── Notification Service
 └── Analytics Service
Advantages:
•	decoupling 
•	buffering 
•	replay 
•	scalable consumers 
•	asynchronous communication 
________________________________________
98. Kafka ordering
Kafka ordering is guaranteed within a partition, not globally across all partitions.
If all events for one payment need ordered processing, use a deterministic partition key such as:
paymentId
Then all events for that payment can be routed to the same partition.
________________________________________
99. Kafka exactly-once
Be careful here.
"Exactly once" is not a universal magical property meaning:
The entire financial business transaction can never execute twice.
Kafka provides exactly-once semantics in specific Kafka processing scenarios through transactional/idempotent mechanisms.
But an external side effect such as:
charge credit card
still requires application-level idempotency and provider-side controls.
That's a very good interview distinction.
________________________________________
100. Circuit breaker
Suppose:
Service A → Service B
Service B is failing.
Without protection:
A retries B
A retries B
A retries B
...
This can cause cascading failure.
Circuit breaker has states:
CLOSED
   ↓ too many failures
OPEN
   ↓ after timeout
HALF_OPEN
   ↓
CLOSED
OPEN
Fail fast instead of continuously calling broken service.
This protects resources and allows recovery.
________________________________________
101. Retry
Retries help with transient failures.
But careless retries can make outages worse.
Bad:
10 requests
→ each retries 5 times
→ 50 requests
Use:
•	exponential backoff 
•	jitter 
•	bounded attempts 
•	timeout 
•	circuit breaker 
And only retry operations where retrying is safe.
For example:
GET
is often easier to retry safely than:
chargeCard()
unless idempotency is explicitly handled.
________________________________________
102. Rate limiter
Suppose an API allows:
100 requests/minute
Common algorithms:
Token Bucket
Tokens are added at a controlled rate.
Request consumes token.
Leaky Bucket
Requests are processed at a controlled output rate.
Fixed Window
Count requests in fixed intervals.
Sliding Window
Tracks a rolling time range.
For distributed systems, Redis is often used for shared rate-limit state, with atomic operations/Lua scripts depending on the design.
________________________________________
103. Horizontal vs vertical scaling
Vertical
Increase machine capacity:
4 CPU → 16 CPU
8 GB → 64 GB
Simple, but has hardware limits.
Horizontal
Add instances:
1 server
 ↓
5 servers
 ↓
20 servers
Usually more resilient/scalable for stateless services.
Typical architecture:
Load Balancer
   ↓
Instance 1
Instance 2
Instance 3
Instance 4
________________________________________
104. How to debug high CPU in JVM
A good production approach:
CPU spike
  ↓
Confirm application/process CPU
  ↓
Identify hot threads
  ↓
Capture thread dump
  ↓
Map OS thread to Java thread
  ↓
Inspect stack trace
  ↓
Check application metrics
  ↓
Check GC/JIT behavior
Useful tools/concepts:
top
htop
jstack
jcmd
JFR
VisualVM
async-profiler
GC logs
Potential causes:
•	infinite loops 
•	excessive serialization 
•	regex problems 
•	high allocation 
•	busy waiting 
•	excessive locking/contention 
•	unexpected traffic 
•	GC overhead 
________________________________________
105. How to debug high memory
Start with:
Current heap usage
Old generation usage
Allocation rate
GC frequency
Metaspace
Native memory
Off-heap buffers
Then capture:
heap dump
Look for:
•	dominant objects 
•	retained size 
•	collection growth 
•	caches 
•	static collections 
•	ThreadLocal retention 
•	classloader leaks 
Important:
Not all memory used by a Java process is Java heap.
Process memory can include:
Heap
Metaspace
Thread stacks
Code cache
Direct buffers
Native libraries
JVM internals
________________________________________
106. Heap dump vs thread dump
Heap dump
Answers:
What objects are consuming memory?
Useful for:
•	memory leaks 
•	OutOfMemoryError 
•	unexpected object retention 
Thread dump
Answers:
What are threads doing right now?
Useful for:
•	deadlocks 
•	blocked threads 
•	CPU issues 
•	thread starvation 
•	stuck requests 
________________________________________
107. ConcurrentHashMap architecture — interview version
Modern ConcurrentHashMap is designed for high concurrency.
Conceptually:
Table
 ├── bin 0
 ├── bin 1
 ├── bin 2
 ├── ...
Reads can often proceed without locking.
Updates use a combination of:
•	CAS 
•	volatile memory semantics 
•	synchronized blocks around relevant bins when necessary 
During resize, threads can participate in the resizing process rather than relying on one thread doing everything in isolation.
Why faster than synchronized HashMap?
A naive synchronized map effectively serializes access:
Thread 1 ─┐
Thread 2 ─┤→ one global lock
Thread 3 ─┘
ConcurrentHashMap allows much more concurrent progress.
________________________________________
108. Producer-Consumer without BlockingQueue
Interviewers may ask this as a concurrency design exercise.
You need:
shared buffer
+
producer
+
consumer
+
thread synchronization
Conceptually:
Producer
   ↓
[Buffer]
   ↓
Consumer
When buffer is full:
Producer waits
When buffer is empty:
Consumer waits
You can implement using:
ReentrantLock
Condition
with:
notFull
notEmpty
This is actually a good reason to understand conditions deeply.
________________________________________
109. System design: News Aggregator
Architecture:
External Sources
       ↓
Feed Fetchers / Crawlers
       ↓
Kafka
       ↓
Parsing + Deduplication
       ↓
Content Store
       ↓
Ranking/Recommendation
       ↓
Search Index
       ↓
API
       ↓
Clients
Important problems:
Duplicate articles
Hash normalized content or use canonical URL/content fingerprinting.
Huge traffic
Use:
CDN
cache
Redis
horizontal scaling
Personalization
Store user preferences and ranking signals.
Freshness
Use event-driven ingestion.
Failure
One source being down shouldn't stop the entire pipeline.
________________________________________
110. Ride-sharing database design
Entities:
User
Driver
Vehicle
Ride
Payment
Location
Rating
Example:
users
drivers
vehicles
rides
payments
ratings
Ride:
ride_id
passenger_id
driver_id
pickup_lat
pickup_lon
drop_lat
drop_lon
status
fare
created_at
For location tracking at high frequency, don't assume a traditional relational table alone is always sufficient.
You may need:
streaming system
geospatial indexes
cache
time-series/event storage
depending on scale.
________________________________________
111. Database architecture for ride-sharing
Possible architecture:
Mobile Apps
    ↓
API Gateway
    ↓
Ride Service
    ↓
DB
For larger scale:
                 API Gateway
                      ↓
        ┌─────────────┼─────────────┐
        ↓             ↓             ↓
   Ride Service   Driver Service   Payment
        ↓             ↓
      SQL           Redis
        ↓
      Kafka
        ↓
 Analytics / Matching / Notifications
Matching drivers is often a high-throughput, location-sensitive problem and may need specialized data structures/storage rather than simply querying an SQL table every time.
________________________________________
112. Java Streams — map vs flatMap
map
One input produces one output.
List<String> names =
    employees.stream()
             .map(Employee::getName)
             .toList();
Conceptually:
Employee → String
________________________________________
flatMap
One input can produce multiple outputs and they are flattened.
List<List<Integer>> numbers =
    List.of(
        List.of(1, 2),
        List.of(3, 4)
    );

List<Integer> result =
    numbers.stream()
           .flatMap(List::stream)
           .toList();
Output:
[1,2,3,4]
Conceptually:
List<List<T>>
     ↓
flatMap
     ↓
List<T>
________________________________________
113. Stream vs Parallel Stream
Normal stream:
list.stream()
Parallel:
list.parallelStream()
Parallel streams use the ForkJoin common pool by default in typical usage.
Parallel execution can help when:
•	dataset is sufficiently large 
•	operations are CPU-intensive 
•	operations are independent 
•	splitting overhead is worthwhile 
It can hurt when:
•	data is small 
•	work is blocking 
•	ordering matters heavily 
•	shared mutable state exists 
•	task splitting/merging dominates 
Never say:
Parallel stream is always faster.
It absolutely is not.
________________________________________
114. Functional Interface
A functional interface has exactly one abstract method.
Examples:
Runnable
Callable
Comparator
Function
Predicate
Consumer
Supplier
Example:
@FunctionalInterface
interface Calculator {
    int calculate(int a, int b);
}
Lambda:
Calculator c =
    (a, b) -> a + b;
Why useful?
They allow behavior to be passed as data.
This is foundational to:
Streams
callbacks
functional composition
lambdas
________________________________________
115. Common Java functional interfaces
Predicate
Input → boolean
Predicate<Integer> p =
    x -> x > 10;
Function
Input → output
Function<String, Integer> f =
    String::length;
Consumer
Input → no result
Consumer<String> c =
    System.out::println;
Supplier
No input → output
Supplier<Double> s =
    Math::random;
________________________________________
116. Record with query results
A record is excellent for DTO-style immutable data.
public record EmployeeDto(
    Long id,
    String name,
    BigDecimal salary
) {}
A repository/query can conceptually return:
List<EmployeeDto>
Then:
List<EmployeeDto> employees =
    repository.findEmployees();
First record:
EmployeeDto first =
    employees.isEmpty()
        ? null
        : employees.get(0);
Or using Stream:
Optional<EmployeeDto> first =
    employees.stream().findFirst();
For SQL/JPA projections, the exact query syntax depends on how your repository and provider are configured.
________________________________________
117. Spring exception handling
Global exception handling:
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(UserNotFoundException.class)
    public ResponseEntity<String> handle(
            UserNotFoundException ex) {

        return ResponseEntity
            .status(HttpStatus.NOT_FOUND)
            .body(ex.getMessage());
    }
}
This prevents every controller from duplicating:
try {
   ...
} catch (...) {
   ...
}
________________________________________
118. @NotNull vs @NotEmpty vs @NotBlank
@NotNull
Value must not be null.
@NotNull
private String name;
"" is allowed.
@NotEmpty
Not null and not empty.
" " is still allowed
@NotBlank
Not null and contains at least one non-whitespace character.
For strings:
@NotNull
@NotEmpty
@NotBlank
@NotBlank is often preferable for text fields where whitespace-only input is invalid.
________________________________________
119. DTO validation vs Entity validation
Prefer validation at the API DTO boundary for API-specific rules.
Example:
public record CreateUserRequest(
    @NotBlank String name,
    @Email String email
) {}
Entity validation can still be valuable for domain invariants.
Why DTO validation?
Because:
API requirements
≠
database/entity requirements
A single entity may be used by:
•	create API 
•	update API 
•	internal jobs 
•	reports 
Each can have different validation needs.
________________________________________
120. How to answer "Explain your project architecture"
Use this structure:
1. Business problem
The application handles ...
2. Architecture
Client
 ↓
API Gateway
 ↓
Microservice
 ↓
DB
3. Technologies
Java
Spring Boot
Kafka
Redis
PostgreSQL
AWS
Docker/Kubernetes
4. Request flow
Explain one actual request end-to-end.
5. Reliability
Talk about:
•	retries 
•	timeout 
•	circuit breaker 
•	idempotency 
•	DB transactions 
6. Scalability
Talk about:
•	horizontal scaling 
•	caching 
•	Kafka 
•	read replicas 
•	partitioning where relevant 
7. Security
Talk about:
•	JWT/OAuth2 
•	TLS 
•	authorization 
•	secrets 
8. Observability
Talk about:
logs
metrics
tracing
alerts
This is much stronger than simply saying:
We used microservices and Spring Boot.
________________________________________
121. The 10 answers you should master first
For the interviews represented in your list, I would put these at the top:
1. HashMap internals
Be able to explain:
hash
→ bucket
→ collision
→ equals
→ tree bin
→ resizing
2. ConcurrentHashMap
Understand:
CAS
volatile
bin-level coordination
resizing
concurrency
3. Java Memory Model
Understand:
visibility
atomicity
ordering
happens-before
4. volatile
Know exactly what it solves and what it does not solve.
5. Garbage Collection
Know:
GC roots
reachability
young/old concepts
G1
ZGC
STW vs concurrent work
6. @Transactional
Know:
proxy
transaction boundary
rollback
propagation
self-invocation
7. Spring Boot auto-configuration
Understand:
classpath
conditions
@EnableAutoConfiguration
BeanDefinition
conditional beans
8. Kafka
Know:
topic
partition
offset
consumer group
ordering
rebalancing
delivery semantics
9. Idempotency
Especially for:
payments
orders
retries
distributed systems
10. Payment system design
Be ready to explain:
API Gateway
↓
Payment Service
↓
Idempotency
↓
DB
↓
Kafka
↓
external processor
↓
reconciliation
and more importantly:
What happens when it fails?
________________________________________
A very important interview mindset
For almost every SDE-2 question, answer in this pattern:
What → How → Why → Trade-off → Real project example
For example, don't answer only:
ConcurrentHashMap is thread-safe.
Answer:
ConcurrentHashMap is designed for concurrent access. Reads can generally proceed without locking, while updates use CAS and localized synchronization mechanisms. This avoids serializing the entire map behind one global lock. I would use it when multiple threads need concurrent access to shared map state, while remembering that compound business operations may still need additional synchronization or atomic APIs.
