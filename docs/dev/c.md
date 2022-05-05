# C

## Strings

To declare a string, we need to use an array of characters:

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int main() {
  char c[] = { 'H', 'e', 'l', 'l', 'o', '\0'};
  printf("%s\n", c);
}
```

However, we can the C compiler (gcc) already handle this type of expressions, so you can simply declare a string as follows:

```c
char c[] = "Hello";
```

We can also declare a pointer to reserve memory on the heap. For example, for the string "Hello" we need 6 bytes (counting the "`\0`") and use the `string.h` library with the `strcpy()` function:

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int main() {
  char *c = malloc(sizeof(char) * 6);
  strcpy(c, "Hello"); // Pointer + String
  printf("%s\n", c);
  free(c);
}
```

Obviously, we can also use `strcpy()` without pointers:

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int main() {
  char s[64];
  strcpy(s, "Hello"); // Pointer + String
  printf("%s\n", s);
}
```

### Arrays of strings

If we need an array with several strings, we can do the following:

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int main() {
  char c[2][8] = { "Hello", "Message" };
  printf("%s\n", c[0]);
  printf("%s\n", c[1]);
}
```

{% hint style="info" %}
**2** is the **number of strings** and **8** is the **maximum size** of a string in **bytes**. In this case, the string "**Message**" occupies 8 bytes (counting the "`\0`").
{% endhint %}

To play with pointers and the heap, we can do the following:

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int main() {
  // Array "strings" = 2 pointers of type char:
  char **strings = malloc(sizeof(char*) * 2);
  char *msg1 = malloc(sizeof(char) * 8);
  char *msg2 = malloc(sizeof(char) * 8);

  strcpy(msg1, "Hello");
  strcpy(msg2, "Message");

  strings[0] = msg1;
  strings[1] = msg2;

  printf("%s\n", strings[0]);
  printf("%s\n", strings[1]);
}
```

## Structs

```c
#include <stdio.h>
#include <stdlib.h>

struct Coordinate {
  double x;
  double y;
};

int main() {
  //struct Coordinate coordinate = { 0.5, 5.9 }; // Another method
  struct Coordinate coordinate = { .y = 5.9, .x = 0.5 };
  printf("x: %f - y: %f\n", coordinate.x, coordinate.y);
}
```

With pointers, we need to do the following:

```c
#include <stdio.h>
#include <stdlib.h>

struct Coordinate {
  double x;
  double y;
};

int main() {
  struct Coordinate *coordinate = malloc(sizeof(struct Coordinate));
  coordinate->x = 6.4;
  coordinate->y = 9.7;
  printf("x: %f - y: %f\n", coordinate->x, coordinate->y);
  free(coordinate);
}
```

### Linked List Data Structure

A linked list is a linear data structure, in which the elements are not stored at contiguous memory locations. The elements in a linked list are linked using pointers as shown in the below image:

{% embed url="https://media.geeksforgeeks.org/wp-content/cdn-uploads/gq/2013/03/Linkedlist.png" %}

In simple words, a linked list consists of nodes where each node contains a data field and a reference(link) to the next node in the list.

```c
#include <stdio.h>
#include <stdlib.h>

struct Node {
  int value;
  struct Node *next;
};

int main() {
  struct Node *head = malloc(sizeof(struct Node));
  head->value = 1;
  head->next = malloc(sizeof(struct Node));
  head->next->value = 2;
  head->next->next = malloc(sizeof(struct Node));
  head->next->next->value = 3;

  printf("Memory address of the first node: %p\n", head);
  printf("Value of the first node: %i\n", head->value);

  printf("Memory address of the second node: %p\n", head->next);
  printf("Value of the second node: %i\n", head->next->value);

  printf("Memory address of the third node: %p\n", head->next->next);
  printf("Value of the third node: %i\n", head->next->next->value);
  
  free(head);
}

```

## Arguments

```c
#include <stdio.h>

int main (int argc, char *argv[]) {
  printf("Count: %i\n", argc);
  for (int i = 0; i < argc; i++) {
    printf("Argument %i: %s\n", i, argv[i]);
  }
}
```

{% hint style="info" %}
**argc** is the number of arguments and `*argv[]` is an [array of strings](c.md#arrays-of-strings) (`**argv` can also be declared).
{% endhint %}

We can also check the number of arguments:

```c
#include <stdio.h>
#include <stdlib.h>

int main (int argc, char *argv[]) {
  if (argc != 2) {
    printf("Use: ./rgb <FILE>\n");
    exit(-1);
  }
}
```

## RGB

```c
#include <stdio.h>
#include <stdlib.h>

struct RGB {
  int r;
  int g;
  int b;
};

void set(struct RGB *rgb, int r, int g, int b){
  if (r + g + b <= 255 * 3 && (r >= 0 && g >= 0 && b >= 0)){
    rgb->r = r;
    rgb->g = g;
    rgb->b = b;
  }
}

void invert(struct RGB *rgb){
  set(rgb, 255 - rgb->r, 255 - rgb->g, 255 - rgb->b);
}

// Pointer to string:
char *rgb_str(struct RGB *rgb) {
  char *str = malloc(sizeof(char) * 64);
  sprintf(
      str, "RGB: (%i, %i, %i)\nHEX: 0x%02x%02x%02x",
      rgb->r, rgb->g, rgb->b,
      rgb->r, rgb->g, rgb->b
  );
  return str;
  // 
}

int main (int argc, char *argv[]) {
  struct RGB *rgb = malloc(sizeof(struct RGB));
  rgb->r = 0;
  rgb->g = 0;
  rgb->b = 0;
  printf("%s\n", rgb_str(rgb));
  invert(rgb);
  printf("%s\n", rgb_str(rgb));
}

```

> `sprintf()` allows us to store a formatted string in a pointer.

### RGB (with file management)

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

struct RGB {
  int r;
  int g;
  int b;
  int last; // It is used to indicate the last color, so the main() function doesn't continue the loop.
};

void set(struct RGB *rgb, int r, int g, int b){
  if (r + g + b <= 255 * 3 && (r >= 0 && g >= 0 && b >= 0)){
    rgb->r = r;
    rgb->g = g;
    rgb->b = b;
  }
}

void invert(struct RGB *rgb){
  set(rgb, 255 - rgb->r, 255 - rgb->g, 255 - rgb->b);
}

char *rgb_str(struct RGB *rgb) {
  char *str = malloc(sizeof(char) * 64);
  sprintf(
      str, "RGB: (%i, %i, %i)\nHEX: 0x%02x%02x%02x",
      rgb->r, rgb->g, rgb->b,
      rgb->r, rgb->g, rgb->b
  );
  return str;
}


// File management:
struct RGB *get_inverted_colors(char *file) {
  int limit = 4;
  struct RGB *inverted = malloc(sizeof(struct RGB) * limit);

  int line_size = 16; // How many bytes does a line occupy at most? 16
  char *line = malloc(sizeof(char) * line_size);

  FILE *f = fopen(file, "r"); // Open file

  int i = 0;
  // Loop to go through each line of the file:
  while (fgets(line, line_size, f)) {
    if (i >= limit - 1) {
      limit *= 2;
      // Increase the memory space in the HEAP:
      inverted = realloc(inverted, sizeof(struct RGB) * limit);
    }

    int values[3]; // Only 3 values (Ex: [0, 255, 100]) per line
    char *split = strdup(line); // Pointer to duplicate line

    for (int i = 0; i < 3; i++){
      values[i] = atoi(strsep(&split, " ")); // Split and add integer to array
    }

    free(split);

    set(&inverted[i], values[0], values[1], values[2]);
    inverted[i].last = 0;
    invert(&inverted[i++]);
  }

  inverted[i].last = 1;
  free(line); // Free up memory space.
  fclose(f); // Close file
  return inverted;
}

int main (int argc, char *argv[]) {
  if (argc != 2) {
    printf("Use: ./rgb <FILE>\n");
    exit(-1);
  }
  struct RGB *inverted = get_inverted_colors(argv[1]);
  for (int i = 0; !inverted[i].last; i++) {
    char *s = rgb_str(&inverted[i]);
    printf("%s\n\n", s);
    free(s);
  }
  free(inverted);
}

```

`strdup()`: The **strdup()** function is used to **duplicate a string**. In this case, we use it to prevent the **strsep()** function from modifying the original string in the file.

`atoi()`: The C library function `int atoi(const char *str)` converts the string argument str to an integer (type int).
