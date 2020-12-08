# Effective Javascript Notes

## 1. Accustoming Yourself to JavaScript

### Item 1: Know Which JavaScript You Are Using

Things to Remember: 

- Decide which versions of JavaScript your application supports.
- Be sure that any JavaScript features you use are supported by all environments where your application runs.
- Always test strict code in environments that perform the strict-mode checks.
- Beware of concatenating scripts that differ in their expectations about strict mode.

### Item 2: Understand JavaScript’s Floating-Point Numbers

Things to Remember:

- JavaScript numbers are double-precision floating-point numbers.
- Integers in JavaScript are just a subset of doubles rather than a separate datatype.
- Bitwise operators treat numbers as if they were 32-bit signed integers.
- Be aware of limitations of precisions in floating-point arithmetic.

### Item 3: Beware of Implicit Coercions

Things to Remember:

- Type errors can be silently hidden by implicit coercions.
- The + operator is overloaded to do addition or string concatenation depending on its argument types.
- Objects are coerced to numbers via valueOf and to strings via toString.
- Objects with valueOf methods should implement a toString method that provides a string representation of the number produced by valueOf.
- Use typeof or comparison to undefined rather than truthiness to test for undefined values.

### Item 4: Prefer Primitives to Object Wrappers

Things to Remember:

- Object wrappers for primitive types do not have the same behavior as their primitive values when compared for equality.
- Getting and setting properties on primitives implicitly creates object wrappers.

### Item 5: Avoid using == with Mixed Types

Things to Remember:

- The == operator applies a confusing set of implicit coercions when its arguments are of different types.
- Use === to make it clear to your readers that your comparison does not involve any implicit coercions.
- Use your own explicit coercions when comparing values of different types to make your program’s behavior clearer.

### Item 6: Learn the Limits of Semicolon Insertion

Things to Remember:

- Semicolons are only ever inferred before a }, at the end of a line, or at the end of a program.
- Semicolons are only ever inferred when the next token cannot be parsed.
- Never omit a semicolon before a statement beginning with (, \[, +, -, or /.
- When concatenating scripts, insert semicolons explicitly between scripts.
- Never put a newline before the argument to return, throw, break, continue, ++, or --.
- Semicolons are never inferred as separators in the head of a for loop or as empty statements.

### Item 7: Think of Strings As Sequences of 16-Bit Code Units

Things to Remember:

- JavaScript strings consist of 16-bit code units, not Unicode code points.
- Unicode code points 216 and above are represented in JavaScript by two code units, known as a surrogate pair.
- Surrogate pairs throw off string element counts, affecting length, charAt, charCodeAt, and regular expression patterns such as “.”.
- Use third-party libraries for writing code point-aware string manipulation.
- Whenever you are using a library that works with strings, consult the documentation to see how it handles the full range of code points.


## 2. Variable Scope

### Item 8: Minimize Use of the Global Object

Things to Remember:

- Avoid declaring global variables.
- Declare variables as locally as possible.
- Avoid adding properties to the global object.
- Use the global object for platform feature detection.

### Item 9: Always Declare Local Variables

Things to Remember:

- Always declare new local variables with var.
- Consider using lint tools to help check for unbound variables.

### Item 10: Avoid with

Things to Remember:

- Avoid using with statements.
- Use short variable names for repeated access to an object.
- Explicitly bind local variables to object properties instead of implicitly binding them with a with statement.

### Item 11: Get Comfortable with Closures

Things to Remember

- Functions can refer to variables defined in outer scopes.
- Closures can outlive the function that creates them.
- Closures internally store references to their outer variables, and can both read and update their stored variables.

### Item 12: Understand Variable Hoisting

Things to Remember

- Variable declarations within a block are implicitly hoisted to the top of their enclosing function.
- Redeclarations of a variable are treated as a single variable.
- Consider manually hoisting local variable declarations to avoid confusion.

### Item 13: Use Immediately Invoked Function Expressions to Create Local Scopes

Things to Remember

- Understand the difference between binding and assignment.
- Closures capture their outer variables by reference, not by value.
- Use immediately invoked function expressions (IIFEs) to create local scopes.
- Be aware of the cases where wrapping a block in an IIFE can change its behavior.

