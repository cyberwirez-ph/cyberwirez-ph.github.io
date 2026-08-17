---
title: "C, Condensed: Easy Notes & References for Learning C"
date: 2026-06-27
description: "Beginner-friendly notes and references covering the fundamentals of C."
author: "grit"
draft: false
---

*Originally published on [grit8086.github.io](https://grit8086.github.io/posts/condensed-c/)*

Notes and materials adapted from Code Academy & learn-c, reorganized here in text form since it’s easier for me to recall this way.

### Introduction

C is a low level language that allows you to directly interact with the CPU and memory, that’s why it’s mainly used for developing kernels, malware, OS, and etc.

### Fundamentals

Below is a basic hello world:


```c
// Single line comment

/*

  Multi
  line
  comment

*/

#include <stdio.h> // Includes the C standard I/O library so we can use printf()

int main() // main() is the main entry point of the program, recognized by C compilers.
           // It can't be Main(), only main(). Code inside the braces runs first.
{

  printf("Hello World!");   // Prints "Hello World" on the console
  printf("Hello World!\n"); // Prints "Hello World" then a new line
  printf("Hello World\t");  // Equivalent to a tab

  return 0;                 // Return code that tells your OS the program ran successfully

}
```


In order to run this, we have to compile it using a GCC compiler (it translates C code into an executable program that your computer can run, turning human readable code into machine code that the processor can understand).


```bash
gcc main.c -o main
gcc [program.c] -o [outputName]

./main
```


#### Variables and its data types

A variable is a container that can be used to store data, which can be used or modified later in the program.


```c
// Uninitialized variable
int scoreUninitialized;

int scoreUninitialized = 1; // Initialize variable

// Declaring multiple variables
int a, b, c, d, e;

// Declaring an initialized variable
int scoreInitialize = 1;

// <====Different types====>
char    name[]        = "grit"; // Behind the scenes it's stored as g r i t \0 (\0 is a null terminator added to mark the end of the string)
char    manualName[]  = {'g', 'r', 'i', 't', '\0'}; // You can also declare a string manually
int     age           = 10;
float   price         = 1.0;
double  precisePrice  = 3.14; // More precise than float
char    grade         = 'A';

// You can also assign a variable's value from another variable
int iHaveGritsAge = age;

// If you don't want a variable's value to ever change, declare a constant
const int I_DONT_CHANGE = 3;

// In order to print them you need specific specifiers
printf("Name:         \n%s", name         );
printf("Age:          \n%d", age          );
printf("Price:        \n%f", price        );
printf("Precise Price \n%f", precisePrice );
printf("Grade:        \n%c", grade        );
```


> **float vs double:** **double** is the default floating point type in C and is more precise. Use **float** only when you specifically need to save memory, for example in large arrays or graphics code. When in doubt, use **double**.

We can do many things with variables like storing input, calculations, decisions, etc. Below is an example:


```c
#include <stdio.h>

int main() {
    int age; // Stores the user's age
    age = 25; // Assigns a value to the variable

    if (age >= 18) {
        printf("You are an adult.\n");
    } else {
        printf("You are a minor.\n");
    }

    return 0;
}
```


##### Implicit Type Conversion

Implicit conversion is when the compiler automatically converts a value to match the type of the variable it’s being assigned to.


```c
double a = 1.0;
int    b = a; // Implicit conversion (double -> int), this'll be 1
```


##### Explicit Conversion

Explicit conversion, or type casting, is when you manually specify the type a value should be converted to. This is the standard and reliable way.


```c
double a = 1.0;
int    b = (int)a; // Explicit conversion using type casting
```


Also, a **char** doesn’t store a letter directly, it stores a numeric value representing that letter.


```c
char sourceChar = 'a'; 
int targetInt = (int)sourceChar; // targetInt is now 97

int sourceInt = 97;
char targetChar = (char)sourceInt; // targetChar is now 'a'
```


### Operators

Basic arithmetic operations:


```c
int a = 1 + 2;

int b = 1;
int c = 2;
int d = b + c; // = 3

// You can also do -, *, and /

int modulo = 10 % 3; // Returns the remainder of a division instead of the quotient
```


#### Increment and Decrement


```c
int a = 1; 
a++;  // Returns the value (1) then increments -> 2
++a;  // Increments first (1->2) then returns the value -> 2

int b = 2;
b--; // b = 1;
```


#### Shorthand Assignment Operators

Instead of manually typing full expressions, you can use shorthand expressions in C:


```c
int a = 2, b = 3;

a += b;  // Same as a = a + b;   a is now 5
a -= b;  // Same as a = a - b;   a is now 2
a *= b;  // Same as a = a * b;   a is now 6
a /= b;  // Same as a = a / b;   a is now 2
a %= b;  // Same as a = a % b;   a is now 2
```


#### Bitwise Operators

Bitwise operators work directly on the individual **bits** of a value, not the value as a whole. They come up constantly when working with flags, masks, and low-level data.

Before the table: each of these operators works on the individual bits of a number, not the number as a whole. So instead of asking “is 5 greater than 3,” you are asking “which specific binary digits are set.”

Operator| Name| Description  
---|---|---  
`&`| AND| Bit is 1 only if both corresponding bits are 1  
|| OR| Bit is 1 if either corresponding bit is 1  
`^`| XOR| Bit is 1 only if exactly one of the two bits is 1  
`~`| NOT| Flips all bits (0 becomes 1, 1 becomes 0)  
`<<`| Left shift| Shifts bits left, zero-fills from the right  
`>>`| Right shift| Shifts bits right, zero-fills from the left  
  
**Binary operation diagrams**


```
   AND (&)                   OR (|)                    XOR (^)
   both bits must be 1       either bit can be 1       bits must differ

   1010                      1010                      1010
 & 1100                    | 1100                    ^ 1100
   ----                      ----                      ----
   1000                      1110                      0110

   result: only where        result: everywhere        result: only where
   both had a 1              at least one had a 1      they disagreed
```


**AND: checking and clearing bits**


```c
int flags = 0b10101100;
int mask  = 0b00001000;  // Check bit 3

if (flags & mask) {
    printf("Bit 3 is set\n");
}
```


**OR: setting bits**


```c
DWORD flAllocationType = MEM_COMMIT | MEM_RESERVE;  // Two flags set at once
```


**XOR: toggling bits**


```c
int a = 0b1100;
int b = 0b1010;
int c = a ^ b;  // 0b0110

// XOR a value with itself always gives 0
int x = 42;
x = x ^ x;  // x = 0

// Swap two values without a temp variable
int p = 5, q = 9;
p ^= q;
q ^= p;
p ^= q;
// p = 9, q = 5
```


One caveat: the XOR swap breaks silently if **p** and **q** point to the same memory location. Both become zero. Use a temp variable when you are not certain they are distinct.

**NOT: flipping all bits**


```c
unsigned char a = 0b00001111;
unsigned char b = ~a;  // 0b11110000
```


**Left and Right Shift**

Shifting left by N is equivalent to multiplying by 2^N. Shifting right by N is equivalent to dividing by 2^N.


```c
int a = 1;
int b = a << 3;  // 1 * 8 = 8
int c = b >> 1;  // 8 / 2 = 4
```


Shifts are commonly used to construct bitmasks or pack and unpack data:


```c
uint16_t packed    = (high << 8) | low;
uint8_t  high_back = (packed >> 8) & 0xFF;
```


#### Operator Precedence

C doesn’t evaluate strictly left to right, it follows a precedence order: some operators run before others, and operators of equal priority run left to right.

Priority| Operators| Description  
---|---|---  
1| `++, --, ()`| Increment, decrement, grouping  
2| `!, (typecast)`| Logical NOT, type conversion  
3| `*, /, %`| Multiplication, division, modulo  
4| `+, -`| Addition, subtraction  
5| `<, <=, >, >=`| Comparison  
6| `==, !=`| Equality  
7| `&&`| Logical AND  
8| ||| Logical OR  
9| `=, +=, -=, *=, etc.`| Assignment  

```c
int a = 5, b = 10, c = 2;
int result = a + b * c; // b * c runs first (20), then + a -> 25
```


You may use **()** to override the order:


```c
int result = (a + b) * c; // now a + b runs first -> 30
```


### Control Flow

Programs often need to make decisions based on different conditions. This is where conditional logic comes in.

An **if** statement lets you check a condition, if it’s true, the code block runs, if it’s false, the block is skipped.


```
if (condition) 
{
    // Code runs if the condition is true
}
```


You can also use relational and logical operators inside an **if** statement, for example **if (grade >= 60)**.

#### Relational and Logical Operators in if Statements

We’ve already used relational operators like **==** , **!=** , **<** , **>** , **< =**, and **> =** in if statements. These operators compare values and return true or false, making them perfect for conditionals.


```c
if (grade >= 60) {
    printf("You passed!\n");
}
```


If **grade** is 60 or higher, the message is printed. Otherwise, nothing happens.

**Logical Operators in Conditionals**

Sometimes we need to check multiple conditions at once. This is where logical operators come in:

  * **& &** (AND) - true only if both conditions are true
  * **||** (OR) - true if at least one condition is true
  * **!** (NOT) - reverses the condition



Checking two conditions with **& &** (AND):


```c
if (a > 0 && b > 0) {
    printf("Both numbers are positive\n");
}
```


Here, both **a** and **b** must be positive for the message to print.

Checking either condition with **||** (OR):


```c
if (a > 0 || b > 0) {
    printf("At least one number is positive\n");
}
```


This prints if either **a** or **b** is positive.

Using **!** (NOT):


```c
if (!(a > 0)) {
    printf("a is not positive\n");
}
```


This prints if **a** is NOT positive (i.e. **a <= 0**).

Sample code putting it all together:


```c
#include <stdio.h>

int main() {

  int a = 10;
  int b = -5;

  if (a > 0 || b > 0) {
    printf("Positive\n");
  }
  if (a > 0 && !(b > 0)) {
    printf("Positive too\n");
  }

  return 0;
}
```


Here’s a full example using **if / else** with a coin flip:


```c
#include <stdio.h>
#include <stdlib.h>
#include <time.h>

int main() 
{
  // Create a number that's 0 or 1
  srand(time(NULL));
  int coin = rand() % 2;

  // If number is 0: Heads
  // If it is not 0: Tails

  if (coin == 0) 
  {
    printf("Heads\n");
  } 
  else // This runs if the condition is false
  {
    printf("Tails\n");
  }

  return 0;
}
```


If there are more than two possible outcomes, you can use **else if** :


```
if (condition1) {
    // runs if condition1 is true
}
else if (condition2) {
    // runs if condition2 is true (condition1 was false)
}
else if (condition3) {
    // runs if condition3 is true (condition1 and condition2 were false)
}
else {
    // runs if none of the above conditions are true
}
```


#### switch Statement

When checking one variable against multiple values, a **switch** statement provides a cleaner and more readable alternative to a long **if / else if** chain.


```c
int number = 7;

switch(number) {
  case 1:
    printf("Bulbasaur\n");
    break;
  case 2:
    printf("Ivysaur\n");
    break;
  case 3:
    printf("Venusaur\n");
    break;
  case 4:
    printf("Charmander\n");
    break;
  case 5:
    printf("Charmeleon\n");
    break;
  case 6:
    printf("Charizard\n");
    break;
  case 7:
    printf("Squirtle\n");
    break;
  case 8:
    printf("Wartortle\n");
    break;
  case 9:
    printf("Blastoise\n");
    break;
  default:
    printf("Unknown\n");
    break;
}
```


#### Ternary Operator

A shortcut for writing simple **if / else** statements in one line.


```
condition ? value_if_true : value_if_false;

// is equivalent to:

if (condition)
{
    value_if_true;
}
else
{
    value_if_false;
}
```


**Finding the minimum of two numbers**

With **if / else** :


```c
if (a < b) {
    min = a;
} else {
    min = b;
}
```


With the ternary operator:


```c
min = (a < b) ? a : b;
```


**Deciding what to print**


```c
printf("%s\n", (score >= 60) ? "Pass" : "Fail");
```


### Loops & Errors

A loop is a way to repeat a block of code until a certain condition is met.

#### while loop

A loop that keeps running over and over until the condition becomes false.


```c
#include <stdio.h>

int main(void) {
    char password[20] = "sesame";
    char guess[20];

    printf("Enter password: ");
    scanf("%s", guess);

    while (strcmp(guess, password) != 0) { // Keeps looping while the guess is wrong
        printf("Wrong password, try again: ");
        scanf("%s", guess);
    }

    printf("Access granted.\n");
    return 0;
}
```


#### for loop

This is used when we know exactly how many times we want to iterate.


```c
/* for (initialization; condition; update)
   // Statements
*/

for (int i = 0; i < 10; i++)
{
   printf("%d\n", i);
}
```


#### break

This statement allows us to exit a loop immediately, regardless of the loop’s condition.


```c
#include <stdio.h>

int main(void)
{
    int number;

    // Loop 1: exits using break
    while (1) { // Infinite loop, will only stop via break
        printf("Loop 1 - Enter a number (0 to stop): ");
        scanf("%d", &number);

        if (number == 0) {
            break; // Exits the loop immediately when 0 is entered
        }

        printf("You entered: %d\n", number);
    }

    printf("Loop 1 ended.\n\n");

    // Loop 2: exits naturally through the condition, no break needed
    number = 1; // Start positive so the loop runs at least once
    while (number > 0) {
        printf("Loop 2 - Enter a number (0 or negative to stop): ");
        scanf("%d", &number);

        if (number > 0) {
            printf("You entered: %d\n", number);
        }
    }

    printf("Loop 2 ended.\n");

    return 0;
}
```


**Loop 1** uses **while (1)** , which has no natural stopping condition on its own, so **break** is the only way out.

**Loop 2** has no **break** at all, it relies entirely on the condition **while (number > 0)**. Each time through, **scanf()** updates **number** , and once the user enters **0** or a negative value, that condition becomes false on its own, ending the loop naturally without any manual exit.

#### continue

This statement skips only the remaining code inside the loop body for the current iteration, unlike **break** , which exits the loop entirely. Instead, it jumps back to the start of the loop and moves on to the next iteration.


```c
for (int i = 0; i < 10; i++) {

    if (i % 2 == 0)
    {

        continue;  // Skips the printf below and goes straight to i++

    }

    printf("%d is odd\n", i);
}
```


### Arrays

Let’s talk about arrays, a collection of variables of the same type, stored in contiguous memory. It helps manage multiple related variables efficiently.


```c
// Uninitialized array (the compiler reserves memory but doesn't set initial values)
// The size must be specified so the compiler can allocate memory. It can also be filled later in the program
int age[4];

// Initialized array (array with initial values, the compiler automatically determines the size)
// You can also specify the size explicitly if you want to
int age2[] = {1, 2, 3, 4};

// Also, array elements are zero indexed, meaning the first element is at index 0,
// because computers count from 0. Computers store arrays in contiguous memory locations.
// The index represents an offset from the starting memory address.
// Since the first element is at the starting address, its offset is 0,
// making zero-based indexing more efficient.

//                       0  1  2  3
int exampleArray[] = {1, 3, 6, 7};

printf("%d\n", exampleArray[2]); // Prints 6

// Modifying arrays
exampleArray[2] = 5;

printf("%d\n", exampleArray[2]); // Prints 5
```


#### Looping Through Arrays

Arrays let you store multiple values in a single variable. Instead of handling each value individually, you can use a loop to go through all of them at once.

**Using a while loop**


```c
#include <stdio.h>

int main(void) {
    int arr[] = {6, 9, 18, 37, 4};
    int i = 0;

    while (i < 5) {
        printf("%d\n", arr[i]);
        i++;
    }

    return 0;
}
```


**Using a for loop**


```c
#include <stdio.h>

int main(void) {
    int arr[] = {6, 9, 18, 37, 4};

    for (int i = 0; i < 5; i++) {
        printf("%d\n", arr[i]);
    }

    return 0;
}
```


Same result, but the initialization, condition, and increment are all on one line, this is why **for** is the more common choice for looping through arrays.

**The problem: hardcoded size**

Both examples above hardcode **5** as the array length. That’s bad practice, if the array’s size ever changes, the loop breaks unless you remember to update the number too.

#### Determining Array Size with sizeof()

**sizeof()** returns the total memory (in bytes) occupied by a variable or data type.


```c
int arr[] = {3, 8, 4, 0, 9};
int len = sizeof(arr);
printf("%d\n", len); // Outputs 20, not 5
```


The array holds 5 integers, and each **int** takes 4 bytes, so the total size is 5 x 4 = 20 bytes. To get the actual number of _elements_ , divide the total size by the size of one element:


```c
int len = sizeof(arr) / sizeof(int); // len = 5
```


Now we can rewrite the loop so it adapts to any array length automatically:


```c
#include <stdio.h>

int main(void) {
    int arr[] = {3, 2, 10, 6, 18, 5, 8, 4, 0, 9};
    int len = sizeof(arr) / sizeof(int);

    for (int i = 0; i < len; i++) {
        printf("%d\n", arr[i]);
    }

    return 0;
}
```


You can even skip the **len** variable and calculate the size directly inside the loop condition:


```c
for (int i = 0; i < sizeof(arr) / sizeof(int); i++) {
    printf("%d\n", arr[i]);
}
```


This only works because the array’s declaration is still visible nearby. Once arrays get passed into other functions, **sizeof()** stops working this way, that’s covered later once we get to pointers.

#### Multidimensional Arrays

A multidimensional array is an array of arrays, most commonly a 2D grid of rows and columns, used for things like matrices, grids, and lookup tables.

**Declaring**


```c
int mat[3][4]; // 3 rows, 4 columns, uninitialized
```


Visually:


```
[ [ ?, ?, ?, ? ],   // Row 0
  [ ?, ?, ?, ? ],   // Row 1
  [ ?, ?, ?, ? ] ]  // Row 2
```


**Initializing**


```c
int mat2[][3] = { {1, 6, 3}, {5, 9, 2} }; // 2 rows, 3 columns
```

```
[ [ 1, 6, 3 ],
  [ 5, 9, 2 ] ]
```


The row count can be left out when the array is initialized, since it can be inferred from how many **{ }** groups you wrote, but the column count must always be specified. Arrays with more than 2 dimensions exist, but are rarely used in practice.

**Accessing elements**

Elements are accessed with **array[row][column]** , and just like regular arrays, indexing starts at **0**.


```c
int mat[][3] = {
    {19, 6, 7},   // row 0
    {20, 3, 17},  // row 1
    {16, 13, 10}  // row 2
};

mat[1][2]; // 17, second row, third column
```


**Looping through it**

A single loop isn’t enough since there are two dimensions to walk through, one loop for the rows, and a nested loop for the columns within each row:


```c
int mat[3][3] = {{12, 8, 2}, {17, 19, 5}, {6, 11, 2}};

for (int i = 0; i < 3; i++) {
    for (int j = 0; j < 3; j++) {
        printf("%d\n", mat[i][j]);
    }
}
```


**Avoiding hardcoded dimensions**

Same idea as before, use **sizeof()** instead of hardcoding the row/column counts:


```c
int rowDimension = sizeof(mat) / sizeof(mat[0]);
int columnDimension = sizeof(mat[0]) / sizeof(int);

for (int i = 0; i < rowDimension; i++) {
    for (int j = 0; j < columnDimension; j++) {
        printf("%d\n", mat[i][j]);
    }
}
```


**Where this actually shows up:** anywhere there is a natural row/column or lookup relationship. The pattern **transition[state][input]** replacing a long **if/else** chain shows up in parsers, lexers, and cipher substitution tables. A grid of pixel values or a game board are the obvious visual examples.

### Strings

A string is simply a sequence of characters. In C, strings are represented as arrays of **char** , so they follow the same rules as arrays.

#### Creating a String

There are two ways to create one:


```c
// Manually, with the null terminator included
char str[] = {'H', 'e', 'l', 'l', 'o', '\0'};

// Using a string literal (simpler, terminator added automatically)
char str2[] = "Hello";
```


Both produce the same result. The null terminator (**\0**) marks the end of the string, so **“Hello”** takes up 6 bytes in memory, not 5. Note also that C is case-sensitive, **‘A’** and **‘a’** are different characters.

**Printing strings**


```c
char str[] = "Hello World";
printf("%s\n", str); // %s is the string specifier
```


#### Accessing and Modifying Characters

Since a string is just an array, you access and modify characters the same way, by index, starting at **0** :


```c
char str[] = "Hello Yorld";
printf("%c\n", str[6]); // Prints 'Y'

str[6] = 'W'; // Modify in place
printf("%s\n", str); // Prints "Hello World"
```


Important: you can only overwrite existing characters, you can’t add or remove characters from a string this way, the array’s size is fixed once declared.

#### Looping Through a String


```c
#include <stdio.h>
#include <string.h>

int main(void) {
    char str[] = "Hello World";

    int len = strlen(str);  // cache it, do not call strlen on every iteration
    for (int i = 0; i < len; i++) {
        printf("%c", str[i]);
    }

    return 0;
}
```


Use **strlen()** instead of hardcoding the length, it calculates the string’s length dynamically, so the loop works for any string. Note that **strlen()** doesn’t count the null terminator.

#### strcat(), Joining Strings

**strcat(destination, source)** appends **source** onto the end of **destination** , modifying **destination** directly.


```c
#include <stdio.h>
#include <string.h>

int main(void) {
    char s1[9] = "abcd";  // Needs enough room for "abcdefgh\0"
    char s2[] = "efgh";

    strcat(s1, s2);
    printf("%s\n", s1); // "abcdefgh"

    return 0;
}
```


**destination** must have enough space for both strings combined, plus the null terminator. If it doesn’t, you get a **buffer overflow** , where the extra data spills into memory it shouldn’t, corrupting adjacent data and causing crashes or security vulnerabilities:


```c
char s1[5] = "abcd";  // Only room for "abcd\0", no extra space
char s2[] = "efgh";

strcat(s1, s2); // Overflow: tries to write 9 bytes into 5
```


To avoid this, either size the destination buffer generously, or use **strncat()** , which limits how many characters get copied:


```c
strncat(s1, s2, sizeof(s1) - strlen(s1) - 1);
```


#### strcpy(), Copying Strings

**strcpy(destination, source)** copies **source** into **destination** , replacing whatever was there (unlike **strcat()** , which appends).


```c
#include <stdio.h>
#include <string.h>

int main(void) {
    char s1[] = "ABCD";
    char s2[5]; // Needs at least 5 bytes: 4 chars + '\0'

    strcpy(s2, s1);
    printf("%s\n", s2); // "ABCD"

    return 0;
}
```


Same overflow risk applies here: if **destination** is too small, **strcpy()** overwrites memory it shouldn’t. The safer alternative is **strncpy()** , which caps how many characters get copied, but you have to manually null-terminate the result yourself:


```c
char s1[] = "ABCD";
char s2[3];

strncpy(s2, s1, sizeof(s2) - 1);
s2[sizeof(s2) - 1] = '\0'; // Manually ensure it's terminated

printf("%s\n", s2); // "AB", safely truncated
```


### Pointers

**A quick mental model of memory**

When a program runs, the OS gives it access to RAM to store data while it’s running. Think of RAM like a giant row of numbered mailboxes, each box holds one byte, and each byte has its own unique address (written in hexadecimal, like **0x1000**).

Memory is roughly divided into a few regions:

  * **Code segment** \- the compiled instructions themselves (read-only)
  * **Data segment** \- global/static variables
  * **Heap** \- memory you manually allocate and free (**malloc()** , **free()**), grows as needed
  * **Stack** \- local variables and function call info, automatically allocated and freed as functions are called and return



You don’t need to memorize this yet, just know that every variable you declare lives _somewhere_ in this memory, at a specific address.

#### What is a Pointer?

A pointer is a variable that stores a memory address instead of storing a value directly. Instead of holding data like **4** or **‘A’** , it holds _where_ that data lives in memory.


```c
int x = 4; // x's value (4) is stored somewhere in RAM, say address 0x1000
```


**Declaring a pointer**


```c
int* ptr;  // Pointer to an int
int *ptr;  // Same thing, just a different style, both are valid
```


**The address-of operator ( &)**

**&** retrieves a variable’s memory address:


```c
int x = 42;
printf("%p\n", &x); // Prints x's address in hex, e.g. 0x1000
```


Note that **%p** is the format specifier for printing addresses, and the actual address will differ every time you run the program.

**Assigning an address to a pointer**


```c
int x = 42;
int* ptr = &x; // ptr now stores the address of x, not 42 itself

printf("%p\n", &x);  // Address of x
printf("%p\n", ptr);  // Same address, since ptr points to x
```


#### Dereferencing

Once a pointer holds an address, ***** lets you reach into that address and read or modify the value stored there:


```c
int x = 4;
int* ptr = &x;
int y = *ptr; // Dereference ptr: "go to the address ptr holds, and get the value there" -> y = 4
```


You can also modify the original variable through the pointer:


```c
int x = 4;
int* ptr = &x;
*ptr = 200; // Changes x directly, since ptr points to x's address
printf("%d\n", x); // 200
```


Don’t confuse ***** here with multiplication, context (whether it’s next to a pointer declaration/variable vs. two numbers) tells them apart.

**Reassigning a pointer**

A pointer isn’t locked to one variable, it can be pointed at a different variable of the same type later:


```c
int x = 3, y = 14;
int* ptr = &x; // ptr points to x
ptr = &y;      // ptr now points to y instead
```


**Uninitialized pointers**

A pointer that hasn’t been assigned an address holds garbage, or in some cases prints as **(nil)**. Always initialize pointers, even if just to **NULL** :


```c
int* ptr = NULL; // Explicitly "points to nothing" for now
```


#### Pointer Arithmetic

Because a pointer holds an address, you can shift it forward or backward, but only addition and subtraction are allowed (multiplying or dividing an address makes no sense):


```c
int arr[5] = {10, 20, 30, 40, 50};
int* ptr = arr; // Points to arr[0]

ptr = ptr + 2; // Moves 2 elements forward, to arr[2]
printf("%d\n", *ptr); // 30
```


The key detail: **ptr + 1** doesn’t move 1 byte, it moves 1 _element’s worth_ of bytes, so for **int** (4 bytes), **ptr + 2** actually shifts the address by 8 bytes. **ptr++** and **ptr–** work the same way, one element at a time.

Going out of bounds (moving a pointer past the memory it’s actually allowed to access) can crash your program or corrupt other data, so pointer arithmetic needs to stay within the array you’re working with.

#### Pointers and Arrays

An array’s name is itself a pointer to its first element, **arr** is equivalent to **& arr[0]**. This means you can walk through an array with a pointer instead of indices:


```c
int arr[5] = {2, 4, 6, 8, 10};
int* ptr = arr;

for (int i = 0; i < 5; i++) {
    printf("%d\n", *ptr); // Read current element
    ptr++;                // Move to the next one
}
```


The same works for modifying values:


```c
int arr[5] = {1, 2, 3, 4, 5};
int* ptr = arr;

for (int i = 0; i < 5; i++) {
    *ptr = 3; // Overwrite each element with 3
    ptr++;
}
```


#### Pointers and Strings

Since a string is just a **char** array, the same pattern applies. A common approach is to loop until you hit the null terminator instead of using a fixed count:


```c
char str[] = "Hello";
char* ptr = str;

while (*ptr != '\0') {
    printf("%c", *ptr);
    ptr++;
}
```


**Quick recap**

  * A pointer stores an address, not a value
  * **&** gets a variable’s address
  * ***** dereferences a pointer, getting (or setting) the value at that address
  * **%p** prints an address
  * Pointer arithmetic moves in steps of the data type’s size, not raw bytes
  * Always initialize pointers, uninitialized ones hold garbage



### Memory Management

Unlike languages like Java or Python, C doesn’t manage memory for you automatically, there’s no garbage collector cleaning up after you. This gives you direct control over memory, which is powerful, but it also means mistakes (forgetting to free memory, using memory after it’s freed) can cause crashes or unpredictable behavior.

#### Stack vs Heap

  * **Stack** \- stores local variables. Memory is automatically allocated when a variable comes into scope, and automatically freed when it goes out of scope (e.g. when a function returns). You don’t manage this yourself.
  * **Heap** \- memory you manually request and release. It stays reserved for as long as you want, even after the function that created it returns, until you explicitly free it.


```
   STACK                              HEAP
   (managed automatically)            (managed by you)

   +---------------------+            +---------------------+
   |   main()            |            |                     |
   |   int x = 10;  <----+-- lives    |  malloc(100) -------+--> stays alive
   |                     |   here     |  until free()       |    until free()
   +---------------------+            |                     |
   |   foo() called      |            +---------------------+
   |   int y = 5;        |
   |                     |    foo() returns -> y is GONE
   +---------------------+    main() returns -> x is GONE
   |   foo() returns     |
   |   y no longer exists|
   +---------------------+

   local variables die with their function
   heap memory outlives the function that created it
```


The heap is what we’re managing manually in this section, using four functions from **< stdlib.h>**:

Function| Purpose  
---|---  
`malloc()`| Allocates a block of memory  
`calloc()`| Allocates memory and zeroes it out  
`realloc()`| Resizes a previously allocated block  
`free()`| Releases allocated memory  
  
#### malloc() and free()


```c
#include <stdio.h>
#include <stdlib.h>

int main(void) {
    int* p = malloc(sizeof(int)); // Reserve enough space for one int
    *p = 12;

    printf("%d\n", *p); // 12

    free(p); // Release the memory once you're done with it
    return 0;
}
```


A couple of rules that matter here:

  * Always **free()** memory you no longer need, forgetting to is a **memory leak** , memory that stays reserved for nothing.
  * Never use a pointer after you’ve freed it, that’s undefined behavior (this is called a **use-after-free**).



Shorthand for **sizeof** , useful when the type might change later:


```c
int* p = malloc(sizeof *p); // *p is an int, so this is the same as sizeof(int)
```


**Checking for allocation failure**

**malloc()** returns **NULL** if it fails to get memory (e.g. system is out of memory). Always check before using the pointer:


```c
int* x = malloc(sizeof(int) * 10);
if (x == NULL) {
    printf("Memory allocation failed!\n");
    return 1;
}
```


Or combined into one line:


```c
if ((x = malloc(sizeof(int) * 10)) == NULL) {
    printf("Memory allocation failed!\n");
}
```


**Allocating an array on the heap**


```c
int* arr = malloc(sizeof(int) * 10); // Space for 10 ints

for (int i = 0; i < 10; i++) {
    arr[i] = i * 5;
}

for (int i = 0; i < 10; i++) {
    printf("%d\n", arr[i]);
}

free(arr);
```


Once allocated, you can index into **arr** exactly like a regular array.

**Why malloc() memory is dangerous by default**

**malloc()** reserves memory but doesn’t clean it, whatever was left over from the previous program that used that memory is still sitting there. This is called **garbage data** :


```c
int* arr = malloc(5 * sizeof(int));

for (int i = 0; i < 5; i++) {
    printf("%d ", arr[i]); // Unpredictable, e.g. "32210 0 4200123 -858993460 12"
}

free(arr);
```


If your code assumes those values start at **0** (like a counter or accumulator), this will silently produce wrong results instead of crashing, which makes it a nasty bug to track down.

#### calloc(), Allocating Pre-Zeroed Memory

**calloc()** does the same job as **malloc()** , but guarantees every byte starts at **0** :


```c
int* arr = calloc(5, sizeof(int)); // 5 ints, all zeroed

for (int i = 0; i < 5; i++) {
    printf("%d ", arr[i]); // 0 0 0 0 0
}

free(arr);
```


Note the arguments are different: **calloc(count, size)** instead of **malloc(count * size)**.

You could zero out **malloc()** ’d memory manually with **memset(arr, 0, 5 * sizeof(int))** , **calloc()** just does that for you in one step.

**When to use which:**

  * Use **calloc()** when you need memory to start at zero, e.g. a counter array, or you want to be safe against reading uninitialized values.
  * Use **malloc()** when you’re about to overwrite the memory immediately anyway (e.g. reading data straight into it), since zeroing it first would just be wasted work.



#### realloc(), Resizing Memory

If you allocated memory and later need more (or less) space, **realloc()** resizes the existing block instead of you manually creating a new one and copying everything over:


```c
#include <stdio.h>
#include <stdlib.h>

int main(void) {
    int* nums = malloc(2 * sizeof(int));
    if (nums == NULL) {
        printf("Allocation failed!\n");
        return 1;
    }

    nums[0] = 1;
    nums[1] = 2;

    int* resized = realloc(nums, 4 * sizeof(int)); // Grow to hold 4 ints
    if (resized == NULL) {
        printf("Reallocation failed!\n");
        free(nums); // Original block is still valid, free it before exiting
        return 1;
    }

    nums = resized; // Always reassign, realloc may move the block to a new address

    nums[2] = 3;
    nums[3] = 4;

    printf("%d, %d, %d, %d\n", nums[0], nums[1], nums[2], nums[3]);

    free(nums);
    return 0;
}
```


**realloc()** might move the block to a completely different address if it cannot expand in place. Always treat the return value as a potentially new address, never assume it stayed the same.

Always assign the result to a separate pointer first and check for **NULL** before overwriting your original. If you assign straight back and **realloc()** returns **NULL** , you lose your only reference to the original block and it leaks.

### memset() and memcpy()

Two functions from **< string.h>** that appear in almost every loader, shellcode stub, and Windows API call sequence.

#### memset(), Filling Memory

**memset(ptr, value, size)** fills a block of memory with a single byte value.


```c
#include <string.h>

char buf[64];
memset(buf, 0, sizeof(buf));    // Zero out the buffer
memset(buf, 0x41, sizeof(buf)); // Fill with 'A'
```


The most common use is zeroing a buffer or struct before use:


```c
STARTUPINFOW si;
memset(&si, 0, sizeof(si));
si.cb = sizeof(si);
```


#### memcpy(), Copying Memory

**memcpy(destination, source, size)** copies **size** bytes from **source** to **destination**. No null terminator logic. It copies raw bytes.


```c
#include <string.h>

unsigned char shellcode[] = { 0x90, 0x90, 0xC3 };
unsigned char buf[3];

memcpy(buf, shellcode, sizeof(shellcode));
```


In a loader, **memcpy** is how you copy sections from a file buffer into allocated memory:


```c
memcpy(pDest, pSrc, sectionHeader.SizeOfRawData);
```


Two things to remember. **memcpy** does not handle overlapping regions. If source and destination overlap, use **memmove** instead. And there is no bounds checking. Writing more bytes than the destination can hold is a buffer overflow.

### Functions

A function is a reusable block of code that performs a specific task. Instead of writing the same logic over and over, you write it once and call it whenever you need it.


```c
#include <stdio.h>

int add(int a, int b) {
    return a + b;
}

int main(void) {
    int result = add(5, 3);
    printf("Sum: %d\n", result); // 8
    return 0;
}
```


Why use functions:

  * Avoid repeating code
  * Make code easier to read and manage
  * Break big problems into smaller, manageable pieces



**Calling functions and arguments**

To use a function, you “call” it by writing its name followed by **()**. If it needs input, you pass that input inside the parentheses, these are called **arguments**. Different functions expect different numbers of arguments:


```c
printf("Hello, World!"); // One argument

int num = 9;
printf("My number is %d", num); // Two arguments, %d is a placeholder for num
```


**Library functions**

C comes with built-in functions grouped into libraries (headers), so you don’t have to write everything from scratch. To use them, include the relevant header at the top of your file:

Header| Provides  
---|---  
`<stdio.h>`| Input/output, e.g. `printf()`  
`<stdlib.h>`| Utilities, e.g. `abs()`, `malloc()`, `rand()`  
`<math.h>`| Math functions, e.g. `ceil()`, `log()`  
`<ctype.h>`| Character functions, e.g. `toupper()`  

```c
#include <stdio.h>
#include <math.h>
#include <ctype.h>

int main(void) {
    float number = 4.5;
    char letter = 'a';

    printf("%f\n", ceil(number));    // 5.000000
    printf("%d\n", isupper(letter)); // 0 (false)

    letter = toupper(letter);
    printf("%d\n", isupper(letter)); // 1 (true)

    return 0;
}
```


#### Defining Your Own Functions

A function’s signature tells you three things: its name, what inputs (parameters) it needs, and what type of value it returns (or **void** if it returns nothing).


```c
returnType functionName(type1 parameter1, type2 parameter2) {
    // function body
    return output; // omit if returnType is void
}
```


No parameters, no return value:


```c
void makeCookie(void) {
    printf("Milk\nFlour\nChocolate Chips\nButter\n");
    printf("Here's a cookie!\n");
}

int main(void) {
    makeCookie();
    return 0;
}
```


With parameters and a return value:


```c
int addNumbers(int a, int b) {
    return a + b;
}

int main(void) {
    int result = addNumbers(5, 7);
    printf("Sum: %d\n", result); // 12
    return 0;
}
```


Use **void** inside the parentheses when a function takes no parameters (**makeCookie(void)**), it’s more explicit than leaving them empty, even though both compile.

#### Return Values

A function’s return type must match the type it actually returns, **int** , **double** , **char** , and pointers are all valid return types.


```c
int getSecretNumber(void) {
    int secretNumber = 7;
    return secretNumber;
}

int main(void) {
    int number = getSecretNumber();
    printf("The secret number is: %d\n", number);
    return 0;
}
```


Once **return** executes, the function exits immediately, any code written after it inside that function never runs:


```c
int getSecretNumber(void) {
    int secretNumber = 7;
    return secretNumber;
    printf("This never runs\n"); // Unreachable
}
```


**Type mismatches**

Arguments you pass in must match the parameter types the function expects:


```c
void sayIt(int number) {
    printf("%d\n", number);
}

int main(void) {
    char* string = "Hi!";
    sayIt(string); // Wrong: passing a string where an int is expected
}
```


This compiles with a warning at best, but causes unpredictable behavior, always match your argument types to the parameters.

#### Passing by Value vs. Passing by Pointer

By default, C passes a _copy_ of a variable into a function, so changes made inside the function don’t affect the original:


```c
void myFunc(int a) {
    a = a + 2;
    printf("Inside: %d\n", a); // 12
}

int main(void) {
    int a = 10;
    myFunc(a);
    printf("Outside: %d\n", a); // Still 10, unaffected
    return 0;
}
```


To actually modify the original variable, pass a pointer to it instead, then dereference it inside the function:


```c
void myFunc(int* a) {
    *a = *a + 2;
}

int main(void) {
    int a = 10;
    myFunc(&a);
    printf("%d\n", a); // 12, original variable changed
    return 0;
}
```


This pattern is also how you can get a function to “return” more than one value, since a function can only return a single value directly:


```c
void swap(int* x, int* y) {
    int temp = *x;
    *x = *y;
    *y = temp;
}

int main(void) {
    int num1 = 5, num2 = 10;
    swap(&num1, &num2);
    printf("num1 = %d, num2 = %d\n", num1, num2); // num1 = 10, num2 = 5
    return 0;
}
```


You’ll reach for pointer parameters when you want to: modify a variable without returning it, avoid copying large data (arrays, structs), return multiple values, or work with dynamically allocated memory.

#### Function Prototypes

A function must be declared before it’s used, if you call a function before the compiler has seen its definition, you’ll get an error. A **function prototype** solves this: it declares the function’s name, return type, and parameter types up front, so the full definition can come later in the file.


```c
#include <stdio.h>

// Prototypes
int upperFunction(int, int);
void lowerFunction(char*);

int main(void) {
    int sum = upperFunction(4, 2);
    printf("Sum: %d\n", sum);
    return 0;
}

// Definitions
int upperFunction(int number1, int number2) {
    int sum = number1 + number2;
    lowerFunction("The numbers have been added.");
    return sum;
}

void lowerFunction(char* string1) {
    printf("%s\n", string1);
}
```


#### Scope

Scope determines where in your code a variable can be seen and used.


```c
int someFunction(void) {
    int myVariable = 20; // Only exists inside someFunction()
    return 0;
}

int main(void) {
    char myVariable[] = "10"; // A completely separate variable, only exists inside main()
    return 0;
}
```


Even with the same name, these two **myVariable** s don’t conflict, they live in different scopes.

_Local scope_ \- a variable declared inside a function or block only exists within that function or block:


```c
void myFunc(void) {
    int b = 10; // Local to myFunc()
    if (b > 5) {
        int c = 4; // Local to this if block only
    }
    // c is not accessible here anymore
}
```


_Global scope_ \- a variable declared outside all functions is accessible everywhere in the file:


```c
int b = 10; // Global

void myFunc(void) {
    printf("%d\n", b); // Allowed
}

int main(void) {
    printf("%d\n", b); // Allowed
    return 0;
}
```


Global variables are generally best avoided where possible, they’re harder to track (any function can change them), can cause naming conflicts, and make debugging harder.

_Nested scope_ \- a block inside another block (child scope) can access variables from the scope it’s nested in (parent scope), but not the reverse:


```c
int f = 10; // Global

int main(void) {
    int a = 20; // Local to main()

    if (a > 10) {
        int b = 30; // Local to this if block
        printf("%d\n", b + a + f); // Allowed: b from here, a from main(), f from global
    }
    // b is not accessible here

    return 0;
}
```


When C looks up a variable, it checks the current scope first, then works outward through parent scopes, all the way up to global, if it’s not found anywhere, that’s a compile error.

### Function Pointers

In offensive development you will constantly need to call functions without importing them, resolve APIs at runtime, and execute code from a buffer. All three of those rely on function pointers.

A function pointer stores the address of a function rather than data. This lets you call a function indirectly, pass functions as arguments, or jump to code you wrote at runtime.

**How to declare one**

The declaration wraps the pointer name in parentheses to separate it from the return type:


```c
int add(int a, int b) {
    return a + b;
}

int main(void) {
    // int (*funcPtr)(int, int) reads as:
    // funcPtr is a pointer to a function that takes two ints and returns an int
    int (*funcPtr)(int, int) = add;

    int result = funcPtr(3, 4);  // 7
    printf("%d\n", result);
    return 0;
}
```


**typedef** makes the declaration reusable and readable. Without it, every use of the function pointer type has to repeat the full syntax:


```c
typedef int (*MathFunc)(int, int);  // give the type a name
MathFunc funcPtr = add;             // now it reads like a normal variable
```


**Dynamic API resolution**

Resolving and calling a Windows API without importing it:


```c
typedef HANDLE (WINAPI* pOpenProcess)(DWORD, BOOL, DWORD);

HMODULE hKernel32 = GetModuleHandleW(L"kernel32.dll");
pOpenProcess OpenProcess = (pOpenProcess)GetProcAddress(hKernel32, "OpenProcess");

HANDLE hProcess = OpenProcess(PROCESS_ALL_ACCESS, FALSE, targetPid);
```


The **typedef** here defines the function signature once. **GetProcAddress** returns a raw address. The cast tells the compiler how to interpret that address as a callable function.

**Executing shellcode from a buffer**


```c
PVOID pBuffer = VirtualAlloc(NULL, shellcodeSize, MEM_COMMIT | MEM_RESERVE, PAGE_EXECUTE_READWRITE);
memcpy(pBuffer, shellcode, shellcodeSize);

// Cast the buffer to a function pointer and call it
((void(*)())pBuffer)();
```


**void(*)()** means: a pointer to a function that takes no arguments and returns nothing. Wrapping it in another set of parentheses and adding **()** calls it immediately.

### Structures

So far we’ve worked with basic types (**int** , **char**) and derived types built from them (arrays, pointers). A **structure** is another derived type, it lets you group different types of variables into a single unit.

Unlike an array, which holds multiple values of the _same_ type, a structure can hold multiple _different_ types together, useful for representing one real-world “thing” made of several pieces of data.

**Defining a structure**


```c
struct Bottle {
    char* name;
    int maxCapacity;
    int currentCapacity;
};
```


  * **struct** defines a structure.
  * **Bottle** is the structure’s name.
  * The variables inside (**name** , **maxCapacity** , **currentCapacity**) are called **member variables**.
  * Members are only _declared_ here, not initialized, giving them a value at this stage is an error.



**Why bother?** Compare storing two bottles without a structure:


```c
char bottleName1[] = "Medium Bottle";
int maxCapacity1 = 24;
int currentCapacity1 = 0;

char bottleName2[] = "Large Bottle";
int maxCapacity2 = 48;
int currentCapacity2 = 20;
```


Six separate variables for two bottles, and it only gets worse as you add more. With a structure, each bottle is a single variable:


```c
struct Bottle bottle1 = {"Medium Bottle", 24, 0};
struct Bottle bottle2 = {"Large Bottle", 48, 20};
```


**Initializing a structure**

Values are assigned in the same order the members were declared:


```c
struct Bottle bottle1 = {"superBottle", 24, 0};
```


If you don’t want to worry about order, use named (designated) initializers instead:


```c
struct Bottle bottle1 = {
    .name = "superBottle",
    .maxCapacity = 24,
    .currentCapacity = 0
};
```


You can also declare first and assign values later:


```c
struct Bottle myBottle;
myBottle.name = "Medium Bottle";
myBottle.maxCapacity = 24;
myBottle.currentCapacity = 0;
```


**Dot notation**

Use **.** to access or modify a structure’s members:


```c
struct Bottle myBottle = {"Medium Bottle", 24, 0};

myBottle.currentCapacity = 10;
printf("The bottle is now filled to %d\n", myBottle.currentCapacity);
```


**Structure pointers**

Structures can take up a fair amount of memory, especially with several fields or large strings, so it’s common to work with a pointer to a structure instead of copying the whole thing around.


```c
struct Bottle myBottle = {"Medium Bottle", 24, 0};
struct Bottle* bottlePointer = &myBottle; // Points to myBottle
```


There are two ways to access members through a pointer:


```c
// Dereference then dot, needs parentheses since . binds tighter than *
(*bottlePointer).maxCapacity;

// Arrow notation, cleaner and far more common in practice
bottlePointer->maxCapacity;
```


*_(_bottlePointer).maxCapacity__ and **bottlePointer- >maxCapacity** do exactly the same thing, arrow notation is just shorthand that’s easier to read, and what you’ll see in most real C code.

**Why use structure pointers:**


```
   passing by VALUE                   passing by POINTER

   bottleFunction(b1)                 bottleFunction(&b1)

   +------------------+               +------------------+
   |  COPY of b1      |               |  address of b1   |  (8 bytes)
   |  name ptr  (8B)  |               +------------------+
   |  maxCap    (4B)  |                        |
   |  currCap   (4B)  |               points to original
   +------------------+               +------------------+
   (16+ bytes copied)                 |  actual b1       |
                                      |  name ptr  (8B)  |
   changes stay inside                |  maxCap    (4B)  |
   the function                       |  currCap   (4B)  |
                                      +------------------+
                                      changes affect original
```


  * Avoids copying large structures unnecessarily, saving memory
  * Cheaper to pass into functions, since only an address is passed instead of the entire structure
  * Matters more as your structures grow bigger or more complex



**Structures and functions**

Passing a structure **by value** gives the function a full copy, changes made inside don’t affect the original:


```c
void bottleFunction(struct Bottle b) {
    b.maxCapacity = 100; // Only changes the copy, original is untouched
}
```


Passing a **pointer** to the structure instead lets the function modify the original:


```c
void bottleFunction(struct Bottle* bPointer) {
    bPointer->maxCapacity = 4; // Modifies the original structure
}
```


Both in the same function, to see the contrast directly:


```c
void bottleFunction(struct Bottle b, struct Bottle* bPointer) {
    b.maxCapacity = 100;        // Only affects the local copy of b
    bPointer->maxCapacity = 4;  // Affects the original structure bPointer points to
}

int main(void) {
    struct Bottle b1 = {"Medium", 24, 9};
    struct Bottle b2 = {"Large", 35, 9};

    bottleFunction(b1, &b2); // b1 passed by value, b2 passed by reference

    return 0;
}
```


A function can also **return** a structure, using **struct** as the return type:


```c
struct Bottle getEmptyBottle(void) {
    struct Bottle b = {"My Bottle", 24, 0};
    return b; // Returns a copy of b
}
```


Always use the **struct** keyword when referring to a structure type in a parameter list or return type.

### typedef Structs

Typing **struct Bottle** every time is verbose. **typedef** gives the structure an alias.


```c
typedef struct _BOTTLE {
    char* name;
    int   maxCapacity;
    int   currentCapacity;
} BOTTLE, *PBOTTLE;
```


Three names are now defined:

Name| Meaning  
---|---  
`struct _BOTTLE`| The original struct name  
`BOTTLE`| Alias for `struct _BOTTLE`  
`PBOTTLE`| Pointer to a BOTTLE (`BOTTLE*`)  

```c
BOTTLE  b1 = {"Medium", 24, 0};  // Direct struct
PBOTTLE b2 = &b1;                // Pointer to the struct
```


This exact pattern is used throughout the Windows API. Every Windows type that starts with **P** is a pointer to the base type. So **PHANDLE** is **HANDLE*** , **PDWORD** is **DWORD*** , **PSYSTEM_INFO** is **SYSTEM_INFO***. The **P** just means pointer-to.

**{ 0 }** zero-initializes the entire struct. You will see this constantly in Windows API code. **STARTUPINFOW si = { 0 };** before passing it to **CreateProcessW** is a standard pattern.

### Struct Alignment and sizeof()

Structs are not always the size you expect. Here is why.

The CPU reads memory most efficiently when values sit at addresses that are multiples of their size. A 4-byte **int** prefers to start at an address divisible by 4. To enforce this, the compiler silently inserts **padding** , unused filler bytes between members, so each one lands at the right boundary.


```c
struct Example {
    char  a;   // 1 byte
    int   b;   // 4 bytes
    char  c;   // 1 byte
};
```


You might expect 6 bytes. It is actually 12.


```
   struct Example in memory (12 bytes, not 6)

   byte:  0     1     2     3     4     5     6     7     8     9    10    11
          +-----+-----+-----+-----+-----+-----+-----+-----+-----+-----+-----+-----+
          |  a  | pad | pad | pad |        b (int)        |  c  | pad | pad | pad |
          | 1 B |  -  |  -  |  - |  4 bytes              | 1 B |  -  |  -  |  -  |
          +-----+-----+-----+-----+-----+-----+-----+-----+-----+-----+-----+-----+
          ^                       ^                       ^
          char a                  int b starts at 4       char c
          (1 byte)                (must align to 4)       (1 byte)
```


Always use **sizeof()** on a struct rather than adding up the members:


```c
printf("%zu\n", sizeof(struct Example));  // 12, not 6
```


**Why this matters when parsing binary formats**

If you cast a raw pointer to a struct to parse a PE header, network packet, or binary file format, padding will silently break it. The struct layout will not match the on-disk layout.

The fix is **#pragma pack** :


```c
#pragma pack(push, 1)   // Save current alignment, set to 1 byte

struct RawHeader {
    char  magic[2];
    int   offset;
    char  flags;
};                      // 7 bytes exactly, no padding

#pragma pack(pop)       // Restore previous alignment
```


Use **#pragma pack(push, 1)** before any struct that maps directly to a binary format, and **#pragma pack(pop)** after it. Forgetting this is one of the most common causes of silent data corruption when writing PE parsers.

### Enumerations

An **enum** defines a set of named integer constants. It is used to represent a fixed set of states, options, or return values, anywhere you would otherwise use magic numbers.


```c
enum Weekdays {
    Monday,     // 0
    Tuesday,    // 1
    Wednesday,  // 2
    Thursday,   // 3
    Friday,     // 4
    Saturday,   // 5
    Sunday      // 6
};
```


The compiler assigns values starting from 0. You can override the starting value:


```c
enum HttpStatus {
    OK           = 200,
    NOT_FOUND    = 404,
    SERVER_ERROR = 500
};
```

```c
enum Weekdays today = Friday;

switch (today) {
    case Monday:  printf("Monday\n");  break;
    case Friday:  printf("Friday\n");  break;
    default:      printf("Other\n");   break;
}
```


Enum values are accessed directly by name. They are just named integers. In Windows API and offensive dev contexts, enums represent memory protection states, process access rights, and NTSTATUS codes:


```c
// Instead of: if (status == 0xC0000005)
if (status == STATUS_ACCESS_VIOLATION)
```


### Unions

A **union** stores different data types in the same memory location. Only one member is active at a time. Writing to one overwrites all the others because they all share the same space.


```c
union ExampleUnion {
    int   IntegerVar;
    char  CharVar;
    float FloatVar;
};
```


The memory allocated equals the size of the **largest** member.


```c
union ExampleUnion u;

u.IntegerVar = 65;
printf("%d\n", u.IntegerVar);  // 65
printf("%c\n", u.CharVar);     // 'A'  (65 in ASCII)
```


Setting **IntegerVar** to 65 also changes **CharVar** to **‘A’** because they occupy the same memory.

You will encounter unions inside Windows-defined structures. A common pattern is a union inside a struct:


```c
typedef struct _LARGE_INTEGER {
    union {
        struct {
            DWORD LowPart;
            LONG  HighPart;
        };
        LONGLONG QuadPart;
    };
} LARGE_INTEGER;
```


This lets you access the same 64-bit value either as two 32-bit halves or as a single 64-bit integer. You will see **LARGE_INTEGER** used in file size and timestamp APIs.

### Introduction to the Windows API

The **Windows API** provides a way for applications to interact with the Windows operating system. Displaying something on screen, modifying a file, querying the registry, allocating memory, all of these go through the Windows API.

#### Windows Data Types

The Windows API defines its own data types on top of standard C. Most are just typedefs for types you already know.

Type| Description| Equivalent  
---|---|---  
`DWORD`| 32-bit unsigned integer, always 32-bit| `unsigned long`  
`SIZE_T`| Unsigned integer the size of a pointer| `size_t`  
`VOID`| Absence of a specific type| `void`  
`PVOID`| Pointer to any data type| `void*`  
`HANDLE`| Identifies an OS-managed object (file, process, thread)| `void*`  
`HMODULE`| Handle to a loaded module, base address of a DLL or EXE| `void*`  
`BOOL`| Boolean. TRUE (nonzero) or FALSE (0)| `int` (4 bytes)  
`BOOLEAN`| Smaller boolean| `unsigned char`  
`ULONG_PTR`| Unsigned integer the same size as a pointer| pointer-sized  
  
**ULONG_PTR** is needed for pointer arithmetic on **PVOID** , since direct arithmetic on **void** * does not compile:


```c
PVOID pBuffer = VirtualAlloc(NULL, 0x1000, MEM_COMMIT | MEM_RESERVE, PAGE_READWRITE);
// pBuffer = pBuffer + 10;              // Error
pBuffer = (ULONG_PTR)pBuffer + 10;     // Correct
```


**String types**

Type| Description| Equivalent  
---|---|---  
`LPCSTR`| Pointer to a constant ANSI string (read-only)| `const char*`  
`LPSTR`| Pointer to a writable ANSI string| `char*`  
`LPCWSTR`| Pointer to a constant wide string (read-only)| `const wchar_t*`  
`LPWSTR`| Pointer to a writable wide string| `wchar_t*`  
  
The **L** prefix is a leftover from 16-bit Windows. The **C** means const. The **W** means wide (UTF-16). **LPCWSTR** = Long Pointer to Const Wide String = **const wchar_t** *.

#### Data Type Pointers

For most Windows types, a **P** -prefixed pointer version exists:

Pointer type| Same as  
---|---  
`PHANDLE`| `HANDLE*`  
`PSIZE_T`| `SIZE_T*`  
`PDWORD`| `DWORD*`  
`PBYTE`| `BYTE*`  
  
#### ANSI and Unicode Functions

Most Windows API functions come in two versions. **CreateFileA** is ANSI, **CreateFileW** is Unicode. Always use the W variants. The A variants convert your string and call W anyway.


```c
// Correct
HANDLE hFile = CreateFileW(L"C:\\file.txt", GENERIC_READ, 0, NULL, OPEN_EXISTING, 0, NULL);
```


#### IN and OUT Parameters

Windows API parameters are annotated with **IN** and **OUT** as documentation hints, not keywords.

  * **IN** parameter: you provide the value.
  * **OUT** parameter: the function writes a result back. These are always pointers.


```c
BOOL HackTheWorld(OUT int* num) {
    *num = 123;
    return TRUE;
}

int main() {
    int a = 0;
    HackTheWorld(&a);
    // a is now 123
}
```


#### Using a Windows API: CreateFileW


```c
HANDLE CreateFileW(
  [in]           LPCWSTR               lpFileName,
  [in]           DWORD                 dwDesiredAccess,
  [in]           DWORD                 dwShareMode,
  [in, optional] LPSECURITY_ATTRIBUTES lpSecurityAttributes,
  [in]           DWORD                 dwCreationDisposition,
  [in]           DWORD                 dwFlagsAndAttributes,
  [in, optional] HANDLE                hTemplateFile
);
```

```c
HANDLE  hFile    = INVALID_HANDLE_VALUE;
LPCWSTR filePath = L"C:\\Users\\yourname\\Desktop\\test.txt";

hFile = CreateFileW(
    filePath,
    GENERIC_ALL,
    0,
    NULL,
    CREATE_ALWAYS,
    FILE_ATTRIBUTE_NORMAL,
    NULL
);

if (hFile == INVALID_HANDLE_VALUE) {
    printf("[-] CreateFileW failed with error: %d\n", GetLastError());
    return -1;
}

// Do work with hFile...

CloseHandle(hFile);
```


**CloseHandle** is not optional. Handles are kernel objects. Leaking them in a long-running process or implant means the OS keeps the object alive for the process lifetime. Call **CloseHandle** on every handle you open, including on error paths.

Note that **CreateFileW** returns **INVALID_HANDLE_VALUE** on failure, not **NULL**. **INVALID_HANDLE_VALUE** is **-1** cast to a **HANDLE** , which is **0xFFFFFFFFFFFFFFFF** on x64. Not all handles use the same failure value. Read the docs per function.

#### HeapAlloc and the Windows Heap APIs

In Windows malware development you will sometimes want to allocate memory without depending on the C runtime. **malloc** internally calls **HeapAlloc** via the CRT. Going directly removes that dependency.


```c
#include <windows.h>

HANDLE hHeap = GetProcessHeap();
PVOID pBuffer = HeapAlloc(hHeap, HEAP_ZERO_MEMORY, 256);
if (pBuffer == NULL) {
    // allocation failed
}

// Use the buffer...

HeapFree(hHeap, 0, pBuffer);
pBuffer = NULL;
```


**HEAP_ZERO_MEMORY** zeroes the allocation. Without it the memory contains whatever was there before.

At the lower level, **HeapAlloc** calls **RtlAllocateHeap** from **ntdll.dll** (the core Windows runtime library that sits below the Win32 API layer). Some shellcode loaders avoid importing **malloc** or linking the CRT entirely, using **HeapAlloc** or **VirtualAlloc** directly. Knowing both lets you read and write either style.

#### Error Handling: Win32 vs Native API

**Win32 API errors**

When a Win32 function fails, it sets a thread-local error code. Retrieve it with **GetLastError()**.


```c
if (hFile == INVALID_HANDLE_VALUE) {
    printf("Error: %d\n", GetLastError());
}
```


Common codes:

Code| Meaning  
---|---  
2| `ERROR_FILE_NOT_FOUND`  
5| `ERROR_ACCESS_DENIED`  
87| `ERROR_INVALID_PARAMETER`  
  
**Native API errors (NTSTATUS)**

Functions from **ntdll.dll** (**Nt** /**Zw** prefixed) return the error code directly as an **NTSTATUS** value. Zero means success (**STATUS_SUCCESS**).


```c
NTSTATUS status = NtSomething(...);

if (!NT_SUCCESS(status)) {
    printf("[!] Failed with status: 0x%0.8X\n", status);
}
```


**NT_SUCCESS** is a macro: **#define NT_SUCCESS(Status) (((NTSTATUS)(Status)) >= 0)**.

Win32 functions return **BOOL** or a handle, and you call **GetLastError()** to find the error. Native API functions return **NTSTATUS** directly, so you check the return value with **NT_SUCCESS**.
