# C Programming

## Pointers
Pointers are a sore point for many, many, MAAAAANNNYYYY people in C. Modern languages like Python, JavaScript, and C# handle memory allocation and freeing for you whereas in C, you must do this yourself or build your own custom garbage collector first. Pointers can be mildly confusing conceptually, but most of the struggle comes with the actual implementation of them. For example, most students will nod their head that they understand the concept of storing an address to another area in memory as "pointing" to that other area in memory.
However, simply understanding and actually putting the knowledge into practice are two different things and many people, myself *definitely included*, get massacred C when it comes to memory. Many of these errors either come from improper syntax or misunderstanding of how memory works "behind the scenes." This cheat sheet aims to address the syntax errors, which should not be underestimated in any way.

### Arrays
A pointer to an array is simply the memory address of the first (0th) element of the array. That's it.

A pointer to an array can come in several forms. Given the following:

`char* myReadOnlyArray = "This is a read-only area of memory!";
`char myStandardArray[] = {'H','e','l','l','o','\n','\0'};`
`char mySizedArray[50];`

And given this function:
`char* test_function(char* someArray);`
This function takes a pointer to the first element in a character array as an argument. This is also called a `char*`.

myReadOnlyArray is actually a character array, but it's placed in a *read only area in memory*, which means that it cannot be modified. **This means that you cannot do `myReadOnlyArray[2] = 'c';` to change the 'i' to a 'c' in the above example.** However, you *can* use the bracket syntax to *read*.

myStandardArray is *not* placed in a read-only area of memory. This means that `myStandardArray[2] = 'c'` **is a valid statement**.

In order to "move" a string into an array, which is not a read-only type, either each letter can be changed invidually using bracket or ptr arithmetic notations, or the function `strcpy(char* dest, char* src)` can be used.

In C, it is not okay to say `mySizedArray = "This is a test.\n";`
This won't work because there is no such thing as a string object in C, so strcpy was made to help solve this problem.

### Pointers, Arrays, and Functions

Passing a char array[] is the same as passing a char * to a function as long as the array[] is not initialized. If it is, then it's an array, no longer a pointer.

Also, functions cannot "return arrays" in C, which is why "output parameters" are often used; you can pass the address of a buffer into the function which will hold the new data. If you don't do this, you will try to pass the address of a locally scoped array which won't work because at the time of return, it will no longer be in memory. A static or global array is another possibility, but it will need to have a set length ahead of time.
