# Pointers

Go is a pass by value language, meaning that when you pass a variable into a function, a copy of the variable is made, meaning that in some cases this can be memory inefficient.

To tackle this, Go uses pointers and m

This is where pointers and memory addresses come in, you can use pointers and memory addresses, which will allow you to declare a variable and update its value without creating a copy of it.

The `&` operator returns a memory address of a variable where as the `*` operator points to a memory address.

## Memory addresses & pointers

```Go
x := 42
fmt.Println("Value of x:", x)     // Prints: Value of x: 42
fmt.Println("Address of x:", &x)  // Prints: Address of x: <memory address>
```

## Dereferencing pointers

```Go
// Declare a variable 'x' of type int
var x int = 10

// Declare a pointer variable 'ptr' of type *int
// Assign the memory address of 'x' to 'ptr' using the '&' operator
ptr := &x

// Print the value of 'ptr' (which is the memory address of 'x')
fmt.Println("Value of ptr:", ptr)  // Output: Value of ptr: <memory address>

// Dereference 'ptr' to access the value stored at the memory address it points to
// This will print the value of 'x' indirectly through the pointer
fmt.Println("Value pointed to by ptr:", *ptr)  // Output: Value pointed to by ptr: 10

// Modify the value of 'x' indirectly through the pointer
*ptr = 20
fmt.Println("Updated value of x:", x)  // Output: Updated value of x: 20
```

## Example usage of functions using pointers

Below is an example of how a function can be created to use pointers to save on memory, as opposed to passing in plain variables which would end up meaning a copy is made in memory.

```Go
func double(x *int) {
    *x *= 2 // Modifying the value at the memory address pointed by x
}

func main() {
    x := 5
    fmt.Println("Before doubling:", x) // Prints: Before doubling: 5
    double(&x)                          // Passing the address of x
    fmt.Println("After doubling:", x)  // Prints: After doubling: 10
}
```
