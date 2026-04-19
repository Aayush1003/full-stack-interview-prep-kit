# Java Interview Questions for Freshers

**Target Profile:** Entry-Level / Fresher
**Focus Areas:** Core Java, Object-Oriented Programming (OOP), Collections, and Basic Memory Concepts.

---

### 1. The 4 Pillars of OOP
**Question:** Explain the four pillars of Object-Oriented Programming in Java.
**Answer:**
- **Encapsulation:** Wrapping variables and methods together in a single class, and hiding details using `private` modifiers (using getters/setters).
- **Inheritance:** A class can acquire properties of another class utilizing the `extends` keyword.
- **Polymorphism:** "Many forms." Achieved via Method Overloading (same method name, different parameters) and Method Overriding (rewriting a parent class method in the child class).
- **Abstraction:** Hiding complex implementation details and showing only the necessary features, achieved via `abstract` classes and `interface`.

### 2. String, StringBuilder, and StringBuffer
**Question:** Is a String mutable? Why do we use `StringBuilder`?
**Answer:**
Strings in Java are **Immutable**, meaning once created, they cannot be changed. Whenever you modify a String, Java creates a completely new String object in the String Constant Pool, which is bad for memory if done in a loop.
We use `StringBuilder` (or `StringBuffer` for thread-safety) because they are mutable. You can append and modify them without generating new object allocations.

### 3. Collection Framework: List vs Set vs Map
**Question:** What is the difference between an ArrayList, a HashSet, and a HashMap?
**Answer:**
- **List (ArrayList):** Ordered, allows duplicate values, and items are accessed via index.
- **Set (HashSet):** Unordered, strictly does NOT allow duplicate values.
- **Map (HashMap):** Stores data in Key-Value pairs. Keys must be unique, but values can be duplicated.

### 4. Equals (`==`) vs `.equals()`
**Question:** What is the difference between `==` and `.equals()` when comparing Strings?
**Answer:**
- `==` checks for **Reference Equality**. It checks if both objects point to the exact same memory location.
- `.equals()` checks for **Value Equality**. It physically compares the characters inside the String object to see if they match.

### 5. Exception Handling
**Question:** What is the difference between Checked and Unchecked Exceptions in Java?
**Answer:**
- **Checked Exceptions (Compile-Time):** Exceptions that the Java compiler forces you to handle using `try/catch` or the `throws` keyword (e.g., `IOException`, `SQLException`).
- **Unchecked Exceptions (Run-Time):** Exceptions that occur during the execution of the program. The compiler does not verify them. They happen due to programming errors (e.g., `NullPointerException`, `ArrayIndexOutOfBoundsException`).
