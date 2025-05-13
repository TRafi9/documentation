# Common Type System

The **Common Type System** refers to a set of types that include:

- `class`
- `interface`
- `structure`
- `enumeration`
- `delegate`

---

## Interface

An **interface** is a blueprint for its inheritors. For example, a class that inherits from an interface must implement its methods with the correct signature.

---

## Structure

A **structure** is a lightweight class type with **value-based semantics**. It is typically best suited for modeling geometric or mathematical data.

### Example: A C# Structure Type

```csharp
struct Point
{
    // Structures can contain fields.
    public int xPos, yPos;

    // Structures can contain parameterized constructors.
    public Point(int x, int y)
    {
        xPos = x;
        yPos = y;
    }

    // Structures may define methods.
    public void PrintPosition()
    {
        Console.WriteLine("({0}, {1})", xPos, yPos);
    }
}
```

## Enumeration

An **enumeration** is a construct that allows you to group name-value pairs. It is defined using the `enum` keyword.

You can use it to enumerate a fixed set of values. By default, the values are assigned incrementally starting from zero.

### Example: Implicit Values

```csharp
enum DaysOfWeek
{
    Mon,   // 0
    Tues,  // 1
    Wed    // 2
}
```

Or hard code them to point to something specifically e.g.

```csharp
enum CharacterTypeEnum
{
    Wizard = 100,
    Fighter = 200,
    Thief = 300
}
```

## Delegate

A **delegate** is the .NET equivalent of a function pointer, but implemented as a class.

Delegates are especially useful when you want to:

- Allow one object to forward a function call to another object.
- Support **multicasting**, i.e. forwarding a function call to multiple recipients.
- Enable **asynchronous method invocation**, i.e. invoking a method on another thread.

They provide type safety and flexibility in scenarios like event handling and callbacks.
