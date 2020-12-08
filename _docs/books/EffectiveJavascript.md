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

### Item 14: Beware of Unportable Scoping of Named Function Expressions

Things to Remember

- Use named function expressions to improve stack traces in Error objects and debuggers.
- Beware of pollution of function expression scope with Object.prototype in ES3 and buggy JavaScript environments.
- Beware of hoisting and duplicate allocation of named function expressions in buggy JavaScript environments.
- Consider avoiding named function expressions or removing them before shipping.
- If you are shipping in properly implemented ES5 environments, you’ve got nothing to worry about.

### Item 15: Beware of Unportable Scoping of Block-Local Function Declarations

Things to Remember

- Always keep function declarations at the outermost level of a program or a containing function to avoid unportable behavior.
- Use var declarations with conditional assignment instead of conditional function declarations.

### Item 16: Avoid Creating Local Variables with eval

Things to Remember

- Avoid creating variables with eval that pollute the caller’s scope.
- If eval code might create global variables, wrap the call in a nested function to prevent scope pollution.

### Item 17: Prefer Indirect eval to Direct eval

Things to Remember

- Wrap eval in a sequence expression with a useless literal to force the use of indirect eval.
- Prefer indirect eval to direct eval whenever possible.


## 3. Working with Functions

### Item 18: Understand the Difference between Function, Method, and Constructor Calls

Things to Remember

- Method calls provide the object in which the method property is looked up as their receiver.
- Function calls provide the global object (or undefined for strict functions) as their receiver. Calling methods with function call syntax is rarely useful.
- Constructors are called with new and receive a fresh object as their receiver.

### Item 19: Get Comfortable Using Higher-Order Functions

Things to Remember

- Higher-order functions are functions that take other functions as arguments or return functions as their result.
- Familiarize yourself with higher-order functions in existing libraries.
- Learn to detect common coding patterns that can be replaced by higher-order functions.

### Item 20: Use call to Call Methods with a Custom Receiver

Things to Remember

- Use the call method to call a function with a custom receiver.
- Use the call method for calling methods that may not exist on a given object.
- Use the call method for defining higher-order functions that allow clients to provide a receiver for the callback.

### Item 21: Use apply to Call Functions with Different Numbers of Arguments

Things to Remember

- Use the apply method to call variadic functions with a computed array of arguments.
- Use the first argument of apply to provide a receiver for variadic methods.

### Item 22: Use arguments to Create Variadic Functions

Things to Remember

- Use the implicit arguments object to implement variable-arity functions.
- Consider providing additional fixed-arity versions of the variadic functions you provide so that your consumers don’t need to use the apply method.

### Item 23: Never Modify the arguments Object

Things to Remember

- Never modify the arguments object.
- Copy the arguments object to a real array using [].slice.call(arguments) before modifying it.

### Item 24: Use a Variable to Save a Reference to arguments

Things to Remember

- Be aware of the function nesting level when referring to arguments.
- Bind an explicitly scoped reference to arguments in order to refer to it from nested functions.

### Item 25: Use bind to Extract Methods with a Fixed Receiver

``` js
var source = ["867", "-", "5309"];
source.forEach(function(s) {
    buffer.add(s);
});
buffer.join(); // "867-5309"
```

```js 
var source = ["867", "-", "5309"];
source.forEach(buffer.add.bind(buffer));
buffer.join(); // "867-5309"
```

Things to Remember

- Beware that extracting a method does not bind the method’s receiver to its object.
- When passing an object’s method to a higher-order function, use an anonymous function to call the method on the appropriate receiver.
- Use bind as a shorthand for creating a function bound to the appropriate receiver.

### Item 26: Use bind to Curry Functions

The technique of binding a function to a subset of its arguments is known as currying, named after the logician Haskell Curry, who popularized the technique in mathematics. Currying can be a succinct way to implement function delegation with less boilerplate than explicit wrapper functions.

Things to Remember

- Use bind to curry a function, that is, to create a delegating function with a fixed subset of the required arguments.
- Pass null or undefined as the receiver argument to curry a function that ignores its receiver.

### Item 27: Prefer Closures to Strings for Encapsulating Code

When in doubt, use a function. Strings are a much less flexible representation of code for one very important reason: They are not closures.

Things to Remember

- Never include local references in strings when sending them to APIs that execute them with eval.
- Prefer APIs that accept functions to call rather than strings to eval.

### Item 28: Avoid Relying on the toString Method of Functions

Things to Remember

- JavaScript engines are not required to produce accurate reflections of function source code via toString.
- Never rely on precise details of function source, since different engines may produce different results from toString.
- The results of toString do not expose the values of local variables stored in a closure.
- In general, avoid using toString on functions.

### Item 29: Avoid Nonstandard Stack Inspection Properties

The best policy is to avoid stack inspection altogether. If your reason for inspecting the stack is solely for debugging, it’s much more reliable to use an interactive debugger.

Things to Remember

- Avoid the nonstandard arguments.caller and arguments.callee, because they are not reliably portable.
- Avoid the nonstandard caller property of functions, because it does not reliably contain complete information about the stack.

## 4. Objects and Prototypes

### Item 30: Understand the Difference between prototype, getPrototypeOf, and __proto__

Things to Remember

- C.prototype determines the prototype of objects created by new C().
- Object.getPrototypeOf(obj) is the standard ES5 function for retrieving the prototype of an object.
- obj.__proto__ is a nonstandard mechanism for retrieving the prototype of an object.
- A class is a design pattern consisting of a constructor function and an associated prototype.

### Item 31: Prefer Object.getPrototypeOf to __proto__

Things to Remember

- Prefer the standards-compliant Object.getPrototypeOf to the nonstandard __proto__ property.
- Implement Object.getPrototypeOf in non-ES5 environments that support __proto__.

### Item 32: Never Modify __proto__

Things to Remember

- Never modify an object’s __proto__ property.
- Use Object.create to provide a custom prototype for new objects.

### Item 33: Make Your Constructors new-Agnostic

Things to Remember

- Make a constructor agnostic to its caller’s syntax by reinvoking itself with new or with Object.create.
- Document clearly when a function expects to be called with new.

### Item 34: Store Methods on Prototypes

Things to Remember

- Storing methods on instance objects creates multiple copies of the functions, one per instance object.
- Prefer storing methods on prototypes over storing them on instance objects.

### Item 35: Use Closures to Store Private Data

Things to Remember

- Closure variables are private, accessible only to local references.
- Use local variables as private data to enforce information hiding within methods.

### Item 36: Store Instance State Only on Instance Objects

Things to Remember

- Mutable data can be problematic when shared, and prototypes are shared between all their instances.

- Store mutable per-instance state on instance objects.
