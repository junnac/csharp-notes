# Group Learning

- **2020-08-06** - Topics: Classes, Objects, Reference Types, Value Types,
  Throwing, Catching, Building Types
- **2020-08-13** - Topics: Video Topics: interface, abstract class, design
  patterns, interface segregation principle (SOL**I**D)

-----

## 1. Working with Classes and Objects

Compare and contrast C# and Ruby when it comes to methods, fields, and
constructors.

```rb
class Book
  def initialize(name)
    @grades = []
    @name = name
  end

  def add_grade(grade)
    @grades.push(grade)
  end
end
```

```c#
class Book
{
  public Book(string name)
  {
    grades = new List<double>();
    this.name = name;
  }

  public void AddGrade(double grade)
  {
    grades.Add(grade);
  }

  // Field definition (local state):
  // - can't use implicit typing (var)
  public List<double> grades;
  private string name;

  // List<double> grades = new List<double>();
  // ^ alternative to setting variable in constructor
}
```

| Ruby                   | C#                                    |
|------------------------|---------------------------------------|
| `Book.new()`           | `new Book()`                          |
| `def initialize(name)` | `public Book(string name)`            |
| `self`                 | `this`                                |
| protected variables    | private variables                     |
| attribute readers      | access modifiers                      |
| instance variable      | field definition                      |
| instance method        | `public` (for instance member access) |
| class method           | `static` (for object reference)       | 

## 2. Does C# pass parameters by reference or by value?

By default, C# always passes parameters by value. In the example below, the
value of `book1` is copied and set as an argument for `SetName`

```c#
var book1 = GetBook("Book 1");
SetName(book1, "New Name");
```

Although it's uncommon, C# can also pass parameters by reference by using the
`ref` keyword (or the `out` keyword if an incoming parameter has not been
initialized).

> **Note**: When you pass a parameter by reference and the referenced argument
> changes, any changes made in the argument are reflected. Using a `ref` for
> this reason is typically very frowned upon. Even in C#. This is referred to as
> a "side effect" of a function.
>
> Passing in a variable and having it changed out from under you without you
> knowing about it leads to weird bugs that take forever to figure out. This is
> where functional programming paradigms shine: many languages are starting to
> introduce functional concepts directly into their syntax or libraries (i.e.
> Immutable.js, Redux, stateless React components).

**Parameter passed by value (default)**

```c#
public void CSharpIsPassByValue()
{
    var book1 = GetBook("Book 1");
    GetBookSetName(book1, "New Name");
    // `book1` references `GetBook("Book 1")`
    Assert.Equal("Book 1", book1.Name);
}

private void GetBookSetName(Book book, string name)
{
    book = new Book(name);
    book.Name = name;
}
```

**Parameter passed by reference (`ref`)**

```c#
public void CSharpIsPassByRef()
{
    var book1 = GetBook("Book 1");
    GetBookSetNameByRef(ref book1, "New Name");
    
    Assert.Equal("New Name", book1.Name);
}

private void GetBookSetNameByRef(ref Book book, string name)
{
    book = new Book(name);
    book.Name = name;
}
```

**Parameter passed by reference (`out`)**

```c#
public void CSharpIsPassByOutRef()
{
    var book1 = GetBook("Book 1");
    GetBookSetNameByOutRef(out book1, "New Name");
    
    Assert.Equal("New Name", book1.Name);
}

private void GetBookSetNameByOutRef(out Book book, string name)
{
    book = new Book(name);
    book.Name = name;
}
```

Strings, however, are reference type variables (and are passed by reference) -
this is why they are immutable.

```c#
public void StringsBehaveLikeValueTypes()
{
    string name = "Korra";
    MakeReferenceUppercase(name);
    var upper = MakeUppercase(name);

    Assert.Equal("Korra", name);
    Assert.Equal("KORRA", upper);
}

private void MakeReferenceUppercase(string parameter)
{
    parameter.ToUpper();
}

private string MakeUppercase(string parameter)
{
    return parameter.ToUpper();
}
```

| Reference Type                   | Value Type                 |
|----------------------------------|----------------------------|
| `var b = new Book()`             | `var x = 3`                |
| pointer to book object is copied | stored `3` value is copied |

## 3. What is a property? What are they used for?

It's used to encapsulate data, similarly to a field. They have syntax similar to
fields, but more advantageous features. An example of an advantageous feature is
the ability to set the privacy of getters and setters.

```c#
// Public field
public string Name;

// Property member syntax (longhand)
private string name;
public string Name
{
    get
    {
        return name.ToUpper();
    }
    set
    {
        if(!String.IsNullOrEmpty(value))
        {
            name = value; // implicit variable `value` available in all setters
        }
    }
}

// Auto property (shorthand)
public string Name { get; set; }
```

> The shorthand syntax will _literally_ generate code that looks something like
this:
>
> ```c#
> private string _name;
> public string Name 
> {
>     get { return _name; }
>     set { _name = value; }
> }
> ```
>
> Back in the day _all_ the methods had to be written like that. A lot of the
> time this is where languages get the stigma of "it's so verbose to write code
> in." Luckily, C# realized it was super verbose and gave us the shorthand
> syntax.

## 4. What is a Member? What is it used for?

A [member] is used to represent the "data and behavior" of a class. All members
(except constructors) are inherited in child classes (aka "derived classes").

Some examples of members are:
- instances of a class (instance members)
- instance variables ([field] members)
- instance methods/constructors/destructors
- properties/attributes
- constants
- events
- nested types (more at [Stack Overflow 1] and [Stack Overflow 2])

Convention: public members (variables) are uppercase (i.e. `Name` instead of
`this.name`).

([Additional resource].)

## 5. What are Events and Delegates? What are they used for?

A delegate is not a method. It functions as a form of typing for methods - it
describes what a method's input/output types look like. A delegate definition is
different from class/struct definition although it looks similar. A delegate can
be initialized in two ways: longhand (`log = new
WriteLogDelegate(ReturnMessage)`) or shorthand (`log = ReturnMessage;`). When a
delegate is initialized, it references a method (`ReturnMessage` in this case).

**Delegate definition**

```c#
public delegate string WriteLogDelegate(string logMessage);
                // ^ describe what a method looks like and name of the delegate
                // ^ method output type   ^ method input type
```

**Delegate initialization**

```c#
public class TypeTests
{
    [Fact]
    public void WriteLogDelegateCanPointToMethod()
    {
        WriteLogDelegate log;

        // New `WriteLogDelegate` is initialized and references `ReturnMessage` method
        // log = new WriteLogDelegate(ReturnMessage);   // Longhand
        log = ReturnMessage;                            // Shorthand

        var result = log("Hello");
        Assert.Equal("Hello!", result);
    }

    // Only has to match return type and parameter type of delegate
    string ReturnMessage(string message)
    {
        return message;
    }
}
```

**Multicast delegate initialization**

```c#
int count = 0;

string ReturnMessage(string message)
{
    count++;
    return message;
}

string IncrementCount(string message)
{
    count++;
    return message;
}

[Fact]
public void WriteLogDelegateCanPointToMethodWithMulticast()
{
    count = 0;
    WriteLogDelegate log = ReturnMessage;
    log += ReturnMessage; // Confused about this and the line above
    log += IncrementCount;

    var result = log("Hello!");
    Assert.Equal("Hello!", result);
    Assert.Equal(3, count);
}
```

The `event` keyword when used in the setting of a delegate variable
(`GradeAdded`) sets an `event` member to the class. An `event` member of a class
can be used to set some sort of event listener in a method, that can later be
handled in an external event handler.

Steps to use events:
1. Define a `delegate` that takes in an event sender (generic `object` type) and
   event arguments (`EventArgs` type)
    ```c#
    public delegate void GradeAddedDelegate(object sender, EventArgs args);
    ```
2. Set the delegate as a named `event`
    ```c#
    public event GradeAddedDelegate GradeAdded;
    ```
3. Raise the event (invoke the delegate)
    ```c#
    GradeAdded(this, new EventArgs());
    ```
4. Define an event handler
    ```c#
    static void OnGradeAdded(object sender, EventArgs e)
    {
        Console.WriteLine("A grade was added");
    }
    ```
5. Handle the event (use the `+=` or `-=` operator to add/subtract methods
   into/from the "invocation list")
    ```c#
    book.GradeAdded += OnGradeAdded;
    ```

**Set the delegate and event**

```c#
public delegate void GradeAddedDelegate(object sender, EventArgs args);
    
public class Book
{
    public Book(string name)
    {
        grades = new List<double>();
        Name = name;
    }
    public void AddGrade(double grade)
    {
        if (grade <= 100 && grade >= 0)
        {
            grades.Add(grade);
            if (GradeAdded != null)
            {
                // "Raising the event" = invoking the delegate
                // `this` = I am the sender
                GradeAdded(this, new EventArgs());
            }
        }
        else
        {
            throw new ArgumentException($"Invalid {nameof(grade)}");
        }
    }

    public event GradeAddedDelegate GradeAdded;
}
```

**Handle the event**

```c#
static void Main(string[] args)
{
    // String `name` is "stored and encapsulated" in `Book` class
    var book = new Book("Jo's grade book");

    // Handle GradeAdd event:
    book.GradeAdded += OnGradeAdded;

    // CODE SHORTENED FOR BREVITY
}

static void OnGradeAdded(object sender, EventArgs e)
    {
        Console.WriteLine("A grade was added");
    }
```

-----

## 1. What is an interface and its purpose?

An interface is a _contract_ ensuring the input/output typing of an entity's
properties and methods. It can be used as a sort of _wrapper_ type. An entity's
properties and methods are known as _signature lines_. An interface includes the
_minimum_ signature lines of a model. A model using the interface can have
additional signature lines, but must have the minimum signature lines listed in
the interface. Each signature line defines an output type and input type.

**Shortcut: generate interface with `cmd` + `.` + `Extract interface`**

Parent classes inheritance must preface interface assignment.

```c#
public class PhysicalProductModel : PhysicalProductBase, IProductModel
```

Although a class can only inherit from one parent class, it can implement
multiple interfaces because interfaces are simply _contracts_.

```c#
public class DigitalProductModel : IProductModel, IDigitalProductModel
```

Alternatively, an interface can build on top of another interface.

```c#
public interface IDigitalProductModel : IProductModel
```

The `DigitalProductModel` below will have all the signature lines of both the
`IDigitalProductModel` and the `IProductModel`.

```c#
public class DigitalProductModel : IDigitalProductModelIDigitalProductModel
```

> **Side note**: the `digital` variable below is set to the `prod` object if
> `prod is IDigitalProductModel`.

```c#
if (prod is IDigitalProductModel digital)
{
    Console.WriteLine($"For the { digital.Title } you have { digital.TotalDownloadsLeft } downloads left");
}
```

## 2. What is an abstract class and what is it used for?

Abstract classes are parent classes that _abstract_ methods for method
inheritance/overriding, DRYer code, and reusable code. They are a blend of
interfaces and a base parent class. You define an abstract class with the
`abstract` keyword, like so:

```c#
public abstract class DataAccess
{
    public string LoadConnectionString(string name)
    {
        Console.WriteLine("Load Connection String");
        return "testConnectionString";
    }
}
```

You cannot instantiate an abstract class. When inheriting an abstract class, the
typings of the child classes are then _blended together_. Abstract class methods
define the typings of a child class' methods with syntax similar to interfaces.
For example, the `abstract` keyword is used and the example below defines the
input/output typing of the `LoadData` and `SaveData` methods:

```c#
public abstract class DataAccess
{
    public string LoadConnectionString(string name)
    {
        Console.WriteLine("Load Connection String");
        return "testConnectionString";
    }

    // Similar syntax to interface
    public abstract void LoadData(string sql);
    public abstract void SaveData(string sql);
}
```

The `SqliteDataAccess` child class that inherits from the abstract `DataAccess`
class would then need to use the `override` keyword to actually define the
`LoadData` and `SaveData` methods.

> **Note**: you can only `override` a method if the parent method is defined
> with `virtual` or `abstract`. Defining a method as `virtual` means that you
> don't have to override it, but you can.

```c#
public class SqliteDataAccess : DataAccess
{
    // Can override (virtual) DataAccess.LoadConnectionString
    public override string LoadConnectionString(string name)
    {
        string output = base.LoadConnectionString(name);
        output += " (from SQLite)";
        return output;
    }

    // Needs to override (abstract) DataAccess.LoadData
    public override void LoadData(string sql)
    {
        Console.WriteLine("Loading SQLite Data");
    }

    // Needs to override (abstract) DataAccess.SaveData
    public override void SaveData(string sql)
    {
        Console.WriteLine("Saving data to SQLite");
    }
}
```

Just like an interface, an instance of the `SqliteDataAccess` class can be typed
as `DataAccess`. The `db` instance below would inherit the
`DataAccess.LoadConnectionString` method and reference its internal
`SqliteDataAccess.LoadData` and `SqliteDataAccess.SaveData` methods.

```c#
DataAccess db = new SqliteDataAccess();
```

## 3. Find an example of an interface in Titan.

An example of an interface in Titan is `IValidationError`.

```c#
namespace Titan.Core.Errors
{
    public interface IValidationError
    {
        string PropertyName { get; }

        string Message { get; }

        int? Status { get; set; }
    }
}
```

### How it works

It's used to define the typings of the `ValidationError` class's properties
(`PropertyName`, `Message`, and `Status`):

```c#
namespace Titan.Core.Errors
{
    public class ValidationError : IValidationError
    {
        public ValidationError(string propertyName, string message) : this(message)
        {
            PropertyName = propertyName;
        }

        public ValidationError(string message)
        {
            PropertyName = string.Empty;
            Message = message;
        }

        public string PropertyName { get; }

        public string Message { get; }

        public int? Status { get; set; }
    }
}
```

### Why we used it in that specific scenario

We used an interface to ensure that every `ValidationError` instance has a
`PropertyName` of type string, a `Message` of type string, and a nullable
`Status` of type integer. This way, we can also reference the interface to type
collections of validation errors.

```c#
public IEnumerable<IValidationError>? Errors { get; private set; }
```

## 4. Find an example in Titan of where we might want to implement an interface.

We might want to implement an `IPractice`. We could then use ISP to implement
`ICode`, `IProject`, `IReading`, and `IVideo` interfaces that build on top of
the an `IPractice` interface.

### Why would an interface be beneficial in that scenario?

It could be beneficial to implement an interface in that scenario because the
`Code`, `Project`, `Reading`, and `Video` classes all have shared and unique
fields. The `IPractice` interface could keep all the generic `Practice` fields
separate from the unique `Code`, `Project`, `Reading`, and `Video` fields. We're
currently using inheritance instead of setting specific interfaces that specify
the typings for each child class.

## 5. What's the Interface Segregation Principle (ISP)?

The Interface Segregation Principle is the practice of not having one general
monolothic interface, and having modular interfaces that are combined instead.

For example, a `Book` class can implement an `IBorrowableBook` interface, which
is actually a combination of the `IBorrowable` and `IBook` interfaces (with the
`IBook` interface building on the `ILibraryItem` interface):

```
IBorrowableBook
  ├── IBorrowable
  └── IBook
        └── ILibraryItem
```

**`Book` class**

```c#
public class Book : IBorrowableBook
{
    public string LibraryId { get; set; }
    public string Title { get; set; }
    public string Author { get; set; }
    public int Pages { get; set; }
    public int CheckOutDurationInDays { get; set; } = 14;
    public string Borrower { get; set; }
    public DateTime BorrowDate { get; set; }

    public void CheckOut(string borrower)
    {
        Borrower = borrower;
        BorrowDate = DateTime.Now;
    }

    public void CheckIn()
    {
        Borrower = "";
    }

    public DateTime GetDueDate()
    {
        return BorrowDate.AddDays(CheckOutDurationInDays);
    }
}
```

**`IBorrowableBook` interface**

```c#
public interface IBorrowableBook : IBorrowable, IBook
{
}
```

**`IBorrowable` interface**

```c#
public interface IBorrowable
{
    DateTime BorrowDate { get; set; }
    string Borrower { get; set; }
    int CheckOutDurationInDays { get; set; }

    void CheckIn();
    void CheckOut(string borrower);
    DateTime GetDueDate();
}
```

**`IBook` interface**

```c#
public interface IBook : ILibraryItem
{
    string Author { get; set; }
    int Pages { get; set; }
}
```

**`ILibraryItem` interface**

```c#
public interface ILibraryItem
{
    string LibraryId { get; set; }
    string Title { get; set; }
}
```

### Is there an example of that principle in practice in Titan?

I don't think so. The two most closely tied interfaces are
`IGroupDayAssignments` and `IGroupCalendarDayAssignments`. I think they are an
implementation of interfaces that are similar and segregated, but aren't an
example of ISP because they don't follow the pattern of combining with other
interfaces in a modular way.


[member]:
https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/members
[Stack Overflow 1]: https://stackoverflow.com/a/1084255/124399 [Stack Overflow
2]: https://stackoverflow.com/a/1083330/124399 [field]:
https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/fields
[Additional resource]:
https://www.techopedia.com/definition/25589/class-members-c-sharp
