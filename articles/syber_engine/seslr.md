# Syber Engine
## The SyberScript Language Reference

**Valeriya Pudova**
**Janis Legzdinsh**

2003
Sybersoft

---

## Contents


- [Syber Engine](#syber-engine)
  - [The SyberScript Language Reference](#the-syberscript-language-reference)
  - [Contents](#contents)
  - [Classes](#classes)
    - [Dependson](#dependson)
    - [Attributes](#attributes)
    - [Create and Delete objects](#create-and-delete-objects)
    - [Duplicate objects](#duplicate-objects)
  - [Variables](#variables)
    - [Simple Variables](#simple-variables)
    - [Local variables](#local-variables)
    - [Arrays](#arrays)
    - [Dynamic Arrays](#dynamic-arrays)
    - [Reference variables](#reference-variables)
    - [Enumerations](#enumerations)
    - [Structs](#structs)
  - [Expressions](#expressions)
    - [Constants](#constants)
    - [Expressions](#expressions-1)
  - [Functions](#functions)
    - [Declaring Functions](#declaring-functions)
    - [Function overriding](#function-overriding)
    - [Optional arguments](#optional-arguments)
    - [Reference to argument](#reference-to-argument)
    - [Advanced function specifiers](#advanced-function-specifiers)
  - [Program Structure](#program-structure)
    - [For Loops](#for-loops)
    - [Do-While Loops](#do-while-loops)
    - [While Loops](#while-loops)
    - [Break](#break)
    - [Conditional Statements](#conditional-statements)
    - [Case Statements](#case-statements)
    - [Operator "with"](#operator-with)
  - [Language Functionality](#language-functionality)
    - [Built-in operators and their precedence](#built-in-operators-and-their-precedence)
    - [Functions set](#functions-set)
  - [Finite State Maсhines](#finite-state-maсhines)
    - [Declaring FSM](#declaring-fsm)
    - [State's line](#states-line)
    - [Predicates and Actions](#predicates-and-actions)
    - [Methods of FSM](#methods-of-fsm)
    - [Attributes of FSM](#attributes-of-fsm)
    - [Keywords of FSM](#keywords-of-fsm)
    - [Example of using fsm](#example-of-using-fsm)
  - [CodingStyle](#codingstyle)
  - [Keywords](#keywords)
    - [General keywords](#general-keywords)
    - [Fsm keywords](#fsm-keywords)


---

## Classes

A class is declared like this:

```c++
class MyClass:MyParentClass; // Class specifiers.

    // << Declaration of class variables and functions goes here

defaultproperties
{
    // << setup values of class variables goes here
}
```

Here I am declaring a new class named "MyClass", which inherits the functionality of "MyParentClass".
Object is the parent class of all objects in SELanguage. Object is an abstract base class, in that it doesn't do anything useful.
Each class inherits all of the variables and functions from its parent class. It can then add new variable declarations, add new functions (or override the existing functions).
The class declaration can take several optional specifiers that affect the class:
native: Says "this class uses behind-the-scenes C++ support". SELanguage expects native classes to contain a C++ implementation in the EXE file.
abstract: Declares the class as an "abstract base class". This prevents the user from spawning actors of this class, because the class isn't meaningful on its own.

### Dependson

The special keyword "dependson" can be used for class declaration. If one class (MyClass) uses any structure or enum values from another class (AnotherClassX, AnotherClassY), it should be marked.

```cpp
class MyClass:MyParentClass			// Class specifiers.
		 dependson(AnotherClassX)
		 dependson(AnotherClassY);
```

The word "dependson" changes the compiling order only. In this example AnotherClassX and AnotherClassY will compile first. But the class MyClass will compile last.

### Attributes

For the user's interface can be used groups of attributes. The special word "compound'' is used for one. Users will see any class's property which is marked by "compound". But the name of the compound attribute will be the name of the group, and this group contains all attributes.

```c++
compound Transformations
{
	Vector position;
	Vector rotation;
	Vector size;
}

compound engine
{
	float speed;
	float currentSpeed;
	float power;
	float fuel;
}
```

Another way is to use the expression "( compaundName )" after the type of variable.

```c++
float (engine) speed;
float (engine) currentSpeed;
float (engine) power;
float (engine) fuel;
```

The editor's inspector illustrated below.

```c++
Transformations
position        0.0, 1.0, 1.0
rotation        0.0, 0.0, 90.0
size            1.0, 1.0, 1.0

engine
speed           10.0
currentSpeed    9.9
power           200
fuel            1000
```

### Create and Delete objects

Object can be created by a "new" operator. This syntax can be used:

```c++
new ( owner, name ) Type;
```

Where "owner" is optional argument, it show; which object will be parent to the new object. The 'name' is optional name of ne object. But "Type" is type of new object. Basicly the part "(owner, name)" is optional to. Method return reference to new object.

```c++
// Create object NTexture with name 'TextureOne' for allTextures owner
new( allTextures, 'TextureOne') NTexture;

// Create object NTexture with default name and owner
new NTexture;

// Create object NTexture with name 'TextureOne' and default owner
new ( ,'TextureOne') NTexture ;
```

Objects can be deleted by the "delete" operator. This syntax can be used:

```c++
delete object;
```

### Duplicate objects

Object can be created by a "duplicate" operator. This syntax can be used:

```c++
duplicate ( owner, name ) object;
```

Where "owner" is optional argument, it show; which object will be parent to the new object. The 'name' is optional name of ne object. But "object" is an object which will be the source for a new object. New object has all values of attributes same like source object. Basicly the part '(owner, name)' is optional to. Method return reference to new object.

```c++
// Duplicate object 'TextureOne' set name for new object 'TextureTwo'
// for allTextures owner
NTexture testure_1 = new( allTextures, 'TextureOne') NTexture ;

NTexture testure_2 = duplicate ( allTextures, 'TextureTwo') texture_1;

// Duplicate object and for new object use default name and owner
duplicate texture_1;

// Duplicate object and set name for new object 'TextureTwo'
// but use default owner
duplicate ( ,'TextureTwo') texture_1;
```

---

## Variables

### Simple Variables

Here are some examples of instance variable declarations in SEScript:

```c++
bool b;                    // Declare a boolean variable named "b".
byte c;                    // Declare a byte variable named "c".
int a;                     // Declare an integer variable named "a".
float f;                   // Declare a floating-point variable named "f".
int Table[64];             // Declare an array of 64 integers named "Table".
string PlayerName;         // Declare a string pointer.
Actor Other;               // Declare a variable referencing an actor.
```

Variables can appear in two kinds of places in SEScript: Instance variables, which apply to an entire object, appear immediately after the class declarations. Local variables appear within a function, and are only active while that function executes.
Here are the basic variable types supported in SEScript:
byte: A 8-bit integer value, from 0 to 255.
int: A 32-bit integer value.
bool: A boolean value: either "true" or "false".
float: A 32-bit floating point number.
string: A string of characters.
enum: This value can be set to any from several named-values.
name: The name of an item in SELanguage (such as the name of a function, state, class, etc). Names are stored as a 16-bit index into the global name table. Names correspond to simple strings of 1-31 characters. Names are not like strings: strings can be modified dynamically, but names can only take on predefined name values.
Class references: A variable that refers to another object or actor in the world. Object and actor references are very powerful tools, because they enable you to access the variables and functions of another actor. Object references may also contain a special value called "none", which is the equivalent of the C "NULL" pointer: it says "this variable doesn't refer to any object".
Structs: Same as C structures.
The variable declaration can take several optional specifiers:
private: — Says "This value used only from this class".
native: — You can declare a SEScript variable as "native", which means that the variable is used from SEScript, but is actually written (elsewhere) in C++.
const: — Says "this value can not be changed".
editconst: — Says "This value can not be changed from interface".
transient: — Says "This value should not be saved to file".
mset: — When this value changes, method setAttribute() should be called.
mget: — When this value is read, the method getAttribute() should be called.

### Local variables

All local variables in SEScript should be declared by the "local" keyword. Just see example:

```c++
void OnMapSpawn(mthing_t mthing)
{
	// Declares all local variables.
	local int localA;
	local int localB;

	// Use variables
	localA = localA + localB;
}
```

### Arrays

Arrays are declared using the following syntax:

```c++
int MyArray[20]; 		// Declares an array of 20 ints.
```

SEScript supports only single dimension arrays.

### Dynamic Arrays

Arrays are declared using the following syntax:

```c++
array<type> name.
```

After declaration of an array, it is possible to read or write any element. If an array of elements contains more than array size, the array changes its own size to this element index. Another way is the redim operation. And it is possible to get count of elements by num attribute.

```c++
array<int> myArray;        // Declares an array.
myArray[0] = 10;           // Increase size to one element.
x = myArray[4];            // Increase size to five elements.
size = myArray.num;        // Get array size.
myArray.redim(2);          // Set the array size to two elements.
```

General thing is we can use the dynamic array method for array-attributes and array-plugs.

### Reference variables

You can declare a variable that refers to an object like this:

```c++
Actor A; 			// An actor reference.
```

The variable "A" above is a reference to an object in the Actor class. Such a variable can refer to any object that belongs to a subclass of Actor.
When you have a variable that refers to an actor, you can access that actor's variables, and call its functions.
Variables that refer to actors always either refer to a valid actor (any actor that actually exists in the level), or they contain the value "none". The value "none" is equivalent to the C/C++ "NULL" pointer.
Note that an object or actor reference "points to" another actor or object, it doesn't "contain" an actor or object. The C equivalent of an actor reference is a pointer to an object.

### Enumerations

Enumerations exist in SEScript as a convenient way to declare a bunch of keywords.
Here is sample code that declares enumerations.

```c++
// Declare an enumeration, with three values.
enum TreeLight
{
    CO_Red,
    CO_Green,
    CO_Blue
};

// Create enum variable
TreeLight tl;
```

### Structs

An SELanguage struct is a way of cramming a bunch of variables together into a new kind of super-variable called a struct. SEScript structs are just like C structs, in that they can contain any simple variables or arrays.
You can declare a struct as follows:

```c++
// A structure describing a plane
struct TPlane
{
    TVec normal;
    float dist;
    int __type;
    int __signbits;
    int __reserved1;
    int __reserved2;
};
```

In SEScript structures can have a parent structure, just like classes. For example:

```c++
struct sec_plane_t:TPlane
{
    float minz;
    float maxz;
    int pic;
    int __base_pic;
    float xoffs;
    float yoffs;
    int flags;
    int translucency;
};
```

---

## Expressions

### Constants

In SEScript, you can specify constant values of nearly all data types:
Integer constants are specified with simple numbers, for example: 123
If you must specify an integer constant in hexadecimal format, use i.e.: 0x123
Floating point constants are specified with decimal numbers like: 123.45 or 123
String constants must be enclosed in double quotes, for example: "MyString"
Name constants must be enclosed in single quotes, for example 'MyName'
Vector constants contain X, Y, and Z values like this: vec (1.0,2.0,4.0)
Color constants contain R, G, B and A values like this: col (1.0,2.0,4.0,5.0)
The "none" constant refers to "no object".
The "self" refers to "this object", i.e. the object whose script is executing.

### Expressions

To assign a value to a variable, use "=" like this:

```c++
void Test(void)
{
    int i;
    float f;
    string s;
    name n;
    TVec v, q;

    i = 10;         // Assign a value to integer variable i.
    f = 2.7;        // Assign a value to floating-point variable f.
    s = "Hello!";   // Assign a value to string variable s.
    n = 'John';     // Assign a value to name variable n.
    v = q;          // Copy value of vector q to v.
}
```

SEScript is strongly typed language, that means that attempts to assign a value of incompatible type will result in compiler error.

---

## Functions

### Declaring Functions

In SEScript, you can declare new functions and write new versions of existing functions. Functions can take one or more parameters, and can optionally return a value. The parameter and return value type must be of size 4 (i.e. integers, floats, pointers, references) or vectors. Some functions are implemented in C++, they are called builtin functions.
Here are some simple function declarations:

```c++
class MyClass:MyParentClass;

    int MyVariable;
    int GetMyVariable(void)
    {
        return MyVariable;
    }

defaultproperties
{
	MyVariable = 0;
}
```

When a function is called, the code within the brackets is executed. Inside the function, you can declare local variables, and execute any SEScript code. The optional "return" keyword causes the function to immediately return a value.
Function calls can be recursive. For example, the following function computes the factorial of a number:

```c++
// Function to compute the factorial of a number.
int Factorial(int Number)
{
    if (Number <= 0)
        return 1;
    else
        return Number * Factorial(Number - 1);
}
```

### Function overriding

"Function overriding" refers to writing a new version of a function in a subclass.
To override a function, just cut and paste the function definition from the parent class into your new class. For example, for OnMapSpawn, you could add this to your Demon class.

```c++
// New Demon class version of the OnMapSpawn function.
void OnMapSpawn(mthing_t mthing)
{
    // If monsters are disabled, then destroy this actor immediately
    if (nomonsters)
    {
        RemoveMobjThinker(self);
        return;
    }
    // Call parent class version of OnMapSpawn
    ::OnMapSpawn(mthing);
}
```

Function overriding is the key to creating new SEScript classes efficiently. You can create a new class that expands on an existing class. Then, all you need to do is override the functions which you want to be handled differently. This enables you to create new kinds of objects without writing gigantic amounts of code.

### Optional arguments

The argument of the function declared as "optional" can be skipped when the function is called. The tail of the function can check whether this argument or not. Script has "specified" keyword for it. This function returns a boolean value.

```c++
Int Function( optional int arg1, int arg2 )
{
    If (specified( arg1 ))
    {
	print("Argument is %d\n", arg1);
    }
    else
    {
      print("Argument was skipped\n");
    }
}

test()
{
    Function( , i);
}
```

The keyword specified can be used only for optional arguments.

### Reference to argument

Several arguments can be sended as references. There is a keyword 'out' for it.

```c++
function( int invar, out int outvar)
{
	outvar = invar;
}

test()
{
	int i;
	Function( 5, i);
	print("Completed with i=%d\n", i);
}
```

It finished with:

```c++
Completed with i=5
```

### Advanced function specifiers

The function declaration can take several optional specifiers:
static: Says "This function has no self pointer".
native: You can declare SEScript functions as "native", which means that the function is callable from SEScript, but is actually written (elsewhere) in C++. For example:

```c++
static native float sin(float angle);
```

---

## Program Structure

SEScript supports all the standard flow-control statements of C/C++/Java:

### For Loops

"for" loops let you cycle through a loop as long as some condition is met. For example:

```c++
// Example of "for" loop.
void ForExample(void)
{
    int i;
    print("Demonstrating the for loop");
    for (i = 0; i < 4; i++)
    {
        print("The value of i is %d\n", i);
    }
    print("Completed with i=%d\n", i);
}
```

The output of this loop is:

```c++
Demonstrating the for loop
The value of i is 0
The value of i is 1
The value of i is 2
The value of i is 3
Completed with i=4
```

In a for loop, you must specify three expressions separated by semicolons. The first expression is for initializing a variable to its starting value. The second expression gives a condition which is checked before each iteration of the loop executes; if this expression is true, the loop executes. If it's false, the loop terminates. The third condition gives an expression which increments the loop counter.
Though most "for" loop expressions just update a counter, you can also use "for" loops for more advanced things like traversing linked lists, by using the appropriate initialization, termination, and increment expressions.
In all of the flow control statements, you can either execute a single statement, without brackets, as follows:

```c++
for (i = 0; i < 4; i++)
    print("The value of i is %d", i);
```

Or you can execute multiple statements, surrounded by brackets, like this:

```c++
for (i = 0; i < 4; i++)
{
    print("The value of i is");
    print("%d\n", i);
}
```

### Do-While Loops

"do"-"while" loops let you cycle through a loop while some ending expression is true.

```c++
// Example of "do" loop.
void DoExample(void)
{
    int i;
    print("Demonstrating the do loop");
    i = 0;
    do
    {
        print("The value of i is %d\n", i);
        i = i + 1;
    } while (i < 4);
    print("Completed with i=%d\n", i);
}
```

The output of this loop is:

```c++
Demonstrating the do loop
The value of i is 0
The value of i is 1
The value of i is 2
The value of i is 3
Completed with i=4
```

### While Loops

"While" loops let you cycle through a loop while some starting expression is true.

```c++
// Example of "while" loop.
void WhileExample(void)
{
    int i = 0;

    print("Demonstrating the while loop");
    while (i < 4)
    {
        print( "The value of i is %d\n", i);
        i = i + 1;
    }
    print("Completed with i=%d\n", i);
}
```

The output of this loop is:

```c++
Demonstrating the do loop
The value of i is 0
The value of i is 1
The value of i is 2
The value of i is 3
Completed with i=4
```

### Break

The "break" command exits out of the nearest loop ("for", "do", or "while").

```c++
// Example of "while" loop.
void WhileExample(void)
{
    int i;
    
    print("Demonstrating break");
    for (i = 0; i < 10; i++)
    {
        if (i == 3)
            break;
        print("The value of i is %d\n", i);
    }
    print("Completed with i=%d\n", i);
}
```

The output of this loop is:

```c++
Demonstrating break
The value of i is 0
The value of i is 1
The value of i is 2
Completed with i=3
```

### Conditional Statements

"if" and "else" let you execute code if certain conditions are met.

```c++
// Example of simple "if".
if (LightBrightness < 20)
    print("My light is dim\n");

// Example of "if-else".
if (LightBrightness < 20)
    print("My light is dim\n");
else
    print("My light is bright\n");
```

### Case Statements

"switch", "case", "default", and "break" let you handle lists of conditions easily.

```c++
// Example of switch-case.
void TestSwitch(void)
{
    // Executed one of the case statements below, based on
    // the value in LightType.
    switch (LightType)
    {
    case LT_None:
        print("There is no lighting\n");
        break;
    case LT_Steady:
        print("There is steady lighting\n");
        break;
    case LT_Backdrop:
        print("There is backdrop lighting\n");
        break;
    default:
        print("There is dynamic\n");
        break;
    }
}
```

A "switch" statement consists of one or more "case" statements, and an optional "default" statement. After a switch statement, execution goes to the matching "case" statement if there is one; otherwise execution goes to the "default" statement; otherwise execution continues past the end of the "select" statement.
After you write code following a "case" label, you must use a "break" statement to cause execution to go past the end of the "switch" statement. If you don't use a "break", execution "falls through" to the next "case" handler.

### Operator "with"

The special word "with" can be used for the definition of a hidden reference pointer. The "with" operator receive any reference to object, this object's address now will be default reference. For example:

```c++
with nodeOne
{
      // default pointer to nodeOne
      with nodeTwo
      {
      // default pointer to nodeTwo
      }
      // default pointer to nodeOne
}
```

It is possible to use any expression after the "with" keyword. For example, creation of new nodes is very useful.

```c++
with new (,'nodeThree') Nvalue
```

Inside with brackets it is possible to use several features. Access to current reference by ".self" word, like this:

```c++
.self.do();
```

Access to any property or method of the current object, just write "." And name of this property, like this:

```c++
.color = vec(0.0, 0.0, 0.0);
```

Create new object as child for current "with" pointer:

```c++
with nodeOne
{
      // default pointer to nodeOne
      new Nvalue; // The nodeOne is owner for new node
}
```

Assign property-reference of current "with" as new this pointer:

```c++
with .children[0]
{
}
```

---

## Language Functionality

### Built-in operators and their precedence

SEScript provides a wide variety of C/C++/Java-style operators for such operations as adding numbers together, comparing values, and incrementing variables. Note that all of the operators have the same precedence as they do in C.

| Operator | Types it applies to | Meaning |
| :--- | :--- | :--- |
| `*=` | int, float, vector | Multiply and assign |
| `/=` | int, float, vector | Divide and assign |
| `+=` | int, float, vector | Add and assign |
| `-=` | int, float, vector | Subtract and assign |
| `||` | bool | Logical or |
| `&&` | bool | Logical and |
| `&` | int | Bitwise and |
| `\|` | int | Bitwise or |
| `^` | int | Bitwise exlusive or |
| `!=` | All | Compare for inequality |
| `==` | All | Compare for equality |
| `<` | int, float | Less than |
| `>` | int, float | Greater than |
| `<=` | int, float | Less than or equal to |
| `>=` | int, float | Greater than or equal to |
| `<<` | int | Left shift |
| `>>` | int | Right shift |
| `+` | int, float | Add |
| `-` | int, float | Subtract |
| `%` | int | Modulo (remainder after division) |
| `*` | int, float, vector | Multiply |
| `/` | int, float, vector | Divide |

The above table lists the operators in order of precedence (with operators of the same precedence grouped together). When you type in a complex expression like "1*2+3*4", SEScript automatically groups the operators by precedence. Since multiplication has a higher precedence than addition, the expression is evaluated as "(1*2)+(3*4)".
The "&&" (logical and) and "||" (logical or) operators are short-circuited: if the result of the expression can be determined solely from the first expression (for example, if the first argument of && is false), the second expression is not evaluated.
In addition, SEScript supports the following unary operators:
! (bool) Logical not.
- (int, float) negation.
~ (int) bitwise negation.
++, -- Decrement (either before or after a variable).

### Functions set

| Return value | Function | Meaning |
| :--- | :--- | :--- |
| float | sin(float angle) | Sine |
| float | cos(float angle) | Cosine |
| float | tan(float angle) | Tangent |
| float | atan(float angle) | Arctangent |
| float | atan2(float angle) | Sine |
| float | sqrt(float value) | Square root. |
| int | abs(int f) | Absolute value |
| int | min(int x, int y) | Minimum |
| int | max(int x, int y) | Maximum |
| float | fabs(float f) | Absolute value |
| float | fmin(float x, float y) | Minimum |
| float | fmax(float x, float y) | Maximum |
| float | fexp(float val) | Exponent |
| float | random() | Create random number |
| float | length(vector vec) | Vector length |
| vector | normalize(vector vec) | Normalize vector |
| float | dotproduct(vector v1, vector v2) | Dot product |
| vector | crossproduct(vector v1, vector v2) | Cross product (perpendicular vector) |
| | anglevectors(vector angle, out vector forward, out vector left, out vector up) | Create vectors from given angle vectors Source angle is a value of degrees. |
| vector | rottovec(vector angle) | Simplified version of angleVectors, create only forward vector Source angle is a value of degrees. |
| vector | vectorot(vector vec) | Create angle vector from a vector Destination angle is value of degrees. |
| float | anglemod360(float angle) | Normalize angle in range 0..360 |
| float | anglemod180(float angle) | Normalize angle in range 0..180 |
| vector | rotatormod360(vector angle) | Normalize rotator in range 0..360 Source and destination angles is value of degrees. |
| vector | rotatormod180(vector angle) | Normalize rotator in range 0..180 Source and destination angles is value of degrees. |
| char | strgetchar(string s, int i) | Get char number I from tith string |
| int | strlen(string s) | Get length of string |
| int | stricmp(string s1, string s2) | Compare strings ignoring case |
| string | strcat(string s1, string s2) | Append string to string |
| string | va(string format, …) | Print to temporary buffer |
| int | atoi(string str) | Convert string to integer |
| float | atof(string str) | Convert string to float |
| | print(string format, …) | Print to console |
| int | ftoi(float f) | Float to integer |
| float | itof(int i) | Integer to float |
| name | stringtoname(string s) | Convert string to name |

---

## Finite State Maсhines

### Declaring FSM

All built-in type fsm is the finite state machine. This is a special object which is always used to implementat behaviors. The fsm has some attributes, plugs and script's methods. The general attribute is State, it contains fname value of the current state. Second attribute is Enabled, it is equal to true if fsm is working now. And Time attribute is a float value internal timer of fsa. Two array plugs values and actions. It's possible to connect to Values only float value nodes, but to Actions plug can be connected to nodes of any type. Every state machine has a state table. There is an example declaration of state machine and one's table:

```c++
class ThreeLight :ParentClass;
fsm treeLight auto // FSM declaration
{
state S0:
	RED, , onRed;
state RED:
	R2YELLOW, , onYellow;
state R2YELLOW:
	GREEN, , onGreen;
state GREEN:
	G2YELLOW, , onYellow;
state G2YELLOW:
	RED, , onRed;
}
```

The optional keyword auto means that the fsm is started immediately after creation.

### State's line

Every state can have many lines, has 32 predicates and 32 actions max. Every line has the following syntax:

```c++
next state , [ list of predicates ], [ list of actions ];
```

Only the next state is an obligatory parameter. And only one line of state can execute its action, it's the first line where the result of predicates is true. That means that it will execute its actions and will change state.

### Predicates and Actions

A list of predicates and actions look like many keywords divided by whitespace:

```c++
list of predicates is:     predicate1 predicate2 predicate3
list of actions  is:       action1 action2 action3
```

Every predicate and action can be the function's name or special keyword. When we use the function's name as predicate or action, fsm is calling this function and predicate, and fsm is receiving Boolean results from predicate. General point is that this function should be declared in this class. Predicates can use invert '!' character before. Another thing is that the result of all predicates is the and operation between all predicates of one line. For example:

```c++
predicate1 !predicate2 predicate3
```

means the following expression:

```c++
predicate1 and !predicate2 and predicate3
```

It is possible write short expression in fsa table, like this:

```c++
predicate1 !predicate2 (speed > maxspeed) predicate3
```

It is possible to use function's calling, like this:

```c++
predicate1 !predicate2 (func(speed)) predicate3
```

It is possible create new short function, like this:

```c++
predicate1 !predicate2 {return speed/2;} predicate3
```

But don't use complicated expressions inside a state machine's table. It makes the table too hard to understand.

### Methods of FSM

The fsa has several methods such as:

```c++
void Start();                    // start fsm
void Stop();                     // stop fsm
void Call(fsa anotherFsa);       // call another fsm
void Return();                   // return to fsa caller
void GotoState(name state);      // goto to state
void TimeRes();                  // reset internal timer
bool IsInState(name state);      // check state
bool IsTime(float time);         // true if timer's value more that time
```

When one fsa calls another fsa, the first one is stopped and it waits for the second fsa to return. Second fsa should make return() to return control to the first fsm.

### Attributes of FSM

Another method to control fsa is access it's attributes from script.

```c++
st = treeLight.State;             // get fsa state

treeLight.Enabled = false;        // stop fsa

if( treeLight.IsCalling )         // true if fsa are calling another
{
};

treeLight.Values[0]  = valueX;    // Connect one predicate child
treeLight.Actions[0] = nodeY;     // Connect one action child
t = treeLight.Time;               // Get current time
```

### Keywords of FSM

In the state's line it's possible to use special keywords instead of predicates and methods. It is possible to use in predicates the following keywords:

```c++
bool time(float time)     // true after time
bool in(int n)            // true if values[n] is not equal zero
bool msg(int n)           // true if values[n] is not equal zero
                          // it reset values[n] to zero
```

In the actions part it is possible to use next keywords:

```c++
one(int n)          // values[n] = 1
zero(int n)         // values[n] = 0
do(int n)           // actions[n].call()
copy(int n)         // values[n] = values[n-1]
timeres             // fsa timer's time = 0
return              // return
stop                // stop
```

### Example of using fsm

Here is a classic example of fsm. The RS trigger is a simple finite state machine, one containing two small elements. Every element is the "TwoAndNot" finite state machine.

```c++
// Class AND-NOT element

class TwoAndNot : LogicElement

bool       in1;             // input signal 1
TwoAndNot  in2;             // input signal 2

fsm sm auto                 // state machine
{
      state S1:
          S0, x1 x2, ;      // go to 0 if (in1 and in2)
      state S0:
          S1, !x1, ;        // go to 1 if !in1
          S1,  x3, ;        // go to 1 if !in2
}

bool x1() { return in1; }

bool x2() { return in2.sm.State == 'S1'; }

bool x3() { return in2.sm.State == 'S0'; }

// Class RS trigger

class RSTrigger: LogicElement;
TwoAndNot q;
TwoAndNot nq;

void Init()
{
      q  = new TwoAndNot;  // create element
      nq = new TwoAndNot;  // create element
      q.in2  = nq;         // make connection
      nq.in2 = q;          // make connection
      q.in1  = false;
      nq.in1 = false;
}
```

It looks like a very simple task, but only a finite state machine decides this problem. Because the right decision is possible only when:
The source has lines only which display the state-graph.
All fsm work in one time, this signifies system has a real parallelism
The SELanguage lets us make the right decision, because it has a real fsm with states and state tables, and because all fsm are working at one time.

---

## CodingStyle

All built-in types, keywords and functions have all small characters.

```c++
float x;                         // float is built-in type
name  firstName;                 // name is built-in type
x = sin(x);                      // sin is built-in function (keyword)
```

All objects and variables begin from low character.

```c++
float x;                         // x is a variable
name  firstName;                 // firstName is a variable
```

All types, structures and classes begin from big characters.

```c++
class Example : FirstClass;      // Example is a class-name
```

All methods and attributes of an object begin from a big character.

```c++
obj.DoThis();                    // DoThis is a method
obj.Speed = 1.0;                 // Speed is an attribute
```


## Keywords

### General keywords

```
array 
abstract 
bool 
byte
break
class 
compound
case
const
default
delete
defaultproperties
dependson
do
duplicate
else
enum
editconst
float
for
false 
int
if
native
new
mset
mget
name
none 
optional
out
private
struct
self 
string
static
switch
specified
transient
true
while
whith
```

### Fsm keywords

```
auto 
copy
do
fsm 
in 
msg 
one 
return
state
time
timeres
zero
stop
```

---
