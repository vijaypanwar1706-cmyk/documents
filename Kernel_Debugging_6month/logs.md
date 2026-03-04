# 📘 The C Programming Language — Chapter 1 Explanation

## **(Hinglish mein — Detailed Notes)**

------

## 🔰 Chapter 1: A Tutorial Introduction (Ek Aaramse Shuruat)

Chapter 1 ka goal yeh hai ki C language ke **core concepts** ko seedhe programs likhke samjhaya jaaye — bina zyada theory mein ghuse. Kernighan aur Ritchie ka approach hai: **pehle karo, phir samjho.**

------

## 1.1 — Getting Started (Hello, World! 🌍)

### Pehla Program

```c
#include <stdio.h>

main()
{
    printf("hello, world\n");
}
```

### Iska Matlab Kya Hai?

| Line                       | Matlab                                                       |
| -------------------------- | ------------------------------------------------------------ |
| `#include <stdio.h>`       | Standard Input/Output library ko include karo — bina iske `printf` nahi chalega |
| `main()`                   | Har C program ka **starting point** hota hai. Program yahi se shuru hota hai |
| `{ }`                      | Curly braces ke andar function ka **body** hota hai          |
| `printf("hello, world\n")` | Screen par text print karta hai                              |
| `\n`                       | **Newline character** — cursor next line par chala jaata hai |

### Important Baatein:

- Har C program mein ek `main()` function **zaroor** hota hai
- `printf` C ka built-in function nahi hai — yeh **standard library** ka function hai
- `\n` ek **escape sequence** hai. Iske alawa:
  - `\t` = Tab
  - `\b` = Backspace
  - `\\` = Backslash
  - `\"` = Double quote

> 💡 **Tip:** Agar `\n` bhool gaye toh output next line par nahi jaayega — sab ek hi line mein chipka rahega!

------

## 1.2 — Variables aur Arithmetic Expressions

### Fahrenheit to Celsius Table

```c
#include <stdio.h>

main()
{
    int fahr, celsius;
    int lower, upper, step;

    lower = 0;
    upper = 300;
    step = 20;

    fahr = lower;
    while (fahr <= upper) {
        celsius = 5 * (fahr - 32) / 9;
        printf("%d\t%d\n", fahr, celsius);
        fahr = fahr + step;
    }
}
```

### Naye Concepts:

**Comments:**

```c
/* Yeh ek comment hai — compiler ise ignore karta hai */
```

**Variables declare karna:**

- C mein har variable ko **pehle declare** karna padta hai
- `int` = integer (poora number, jaise 5, -3, 100)
- `float` = floating point (decimal number, jaise 3.14)

**Data Types ki List:**

| Type     | Kya hai                           |
| -------- | --------------------------------- |
| `char`   | Single character — ek byte        |
| `int`    | Integer — typically 16 ya 32 bit  |
| `short`  | Chhota integer                    |
| `long`   | Bada integer                      |
| `float`  | Decimal number — 32 bit           |
| `double` | Double precision decimal — 64 bit |

**While Loop:**

```c
while (fahr <= upper) {
    // body
}
```

- Condition check hoti hai
- Agar true hai toh body chalti hai
- Jab false ho jaaye toh loop band

**Integer Division ka Chakkar:** `5/9` likhoge toh answer **0** aayega kyunki dono integers hain — fractional part cut ho jaata hai! Isliye `5 * (fahr - 32) / 9` likha — pehle multiply karo, phir divide karo.

**printf ke Format Specifiers:**

| Specifier | Kya print karta hai                 |
| --------- | ----------------------------------- |
| `%d`      | Integer (decimal)                   |
| `%f`      | Float                               |
| `%3d`     | Kam se kam 3 character wide integer |
| `%6.1f`   | 6 wide, 1 decimal place wala float  |
| `%o`      | Octal                               |
| `%x`      | Hexadecimal                         |
| `%c`      | Character                           |
| `%s`      | String                              |

------

## 1.3 — The `for` Statement

```c
#include <stdio.h>

main()
{
    int fahr;
    for (fahr = 0; fahr <= 300; fahr = fahr + 20)
        printf("%3d %6.1f\n", fahr, (5.0/9.0)*(fahr-32));
}
```

### `for` Loop ka Structure:

```
for (initialization; condition; increment)
    statement;
```

1. **Initialization** — ek baar shuru mein hoti hai (`fahr = 0`)
2. **Condition** — har iteration se pehle check hoti hai (`fahr <= 300`)
3. **Increment** — har iteration ke baad hota hai (`fahr = fahr + 20`)

> 💡 `while` aur `for` mein fark: `for` tab zyada useful hai jab init, condition aur increment ek hi jagah rakhna ho. Code compact aur padhne mein asaan lagta hai.

**Float division ka use:** `5.0/9.0` likha taaki integer truncation na ho — `5.0` ek float hai, toh result bhi float aayega.

------

## 1.4 — Symbolic Constants (`#define`)

```c
#define LOWER 0
#define UPPER 300
#define STEP  20
```

### Kya Hota Hai?

- `#define` ek **preprocessor directive** hai
- Compile hone se pehle, compiler `LOWER` ko `0` se, `UPPER` ko `300` se replace kar deta hai
- Yeh **variable nahi** hain — sirf text replacement hai

### Kyun Use Karte Hain?

- **Magic numbers** (jaise 300, 20) directly likhna bura practice hai
- Agar badalna pade, toh sirf `#define` ki line change karo — poora program automatically update ho jaata hai
- Code zyada **readable** hota hai

### Rules:

- Naam convention: **UPPERCASE** mein likho
- Line ke end mein **semicolon nahi** lagate
- `#define` wali values ko quotes mein mat daalo

------

## 1.5 — Character Input aur Output

C mein input/output **character streams** ke through hoti hai. Library do basic functions deti hai:

| Function     | Kaam                                    |
| ------------ | --------------------------------------- |
| `getchar()`  | Keyboard se ek character padhta hai     |
| `putchar(c)` | Screen par ek character print karta hai |

### 1.5.1 — File Copying (Input ko Output mein copy karna)

```c
#include <stdio.h>

main()
{
    int c;
    while ((c = getchar()) != EOF)
        putchar(c);
}
```

**`EOF` kya hai?**

- "End of File" — jab input khatam ho jaaye
- `<stdio.h>` mein defined hai
- `c` ko `int` isliye declare kiya kyunki `EOF` koi bhi valid character nahi hai — isko store karne ke liye `char` kaafi nahi

**Concise version ka fayda:**

```c
while ((c = getchar()) != EOF)
```

Ek hi line mein: read karo → assign karo → check karo. Parentheses zaroori hain kyunki `!=` ki precedence `=` se zyada hai.

### 1.5.2 — Character Counting

```c
long nc = 0;
while (getchar() != EOF)
    ++nc;
printf("%ld\n", nc);
```

- `++nc` matlab `nc = nc + 1` (increment operator)
- `long` use kiya kyunki bahut bade numbers ho sakte hain
- `%ld` format specifier `long` ke liye

**`for` version:**

```c
for (nc = 0; getchar() != EOF; ++nc)
    ;
```

Semicolon akela — yeh **null statement** hai, body khaali hai.

### 1.5.3 — Line Counting

```c
int c, nl = 0;
while ((c = getchar()) != EOF)
    if (c == '\n')
        ++nl;
printf("%d\n", nl);
```

- `'\n'` ek **character constant** hai — single quotes mein
- Har newline character ek line ka end represent karta hai

### 1.5.4 — Word Counting

```c
#include <stdio.h>

#define IN  1  /* inside a word */
#define OUT 0  /* outside a word */

main()
{
    int c, nl, nw, nc, state;
    state = OUT;
    nl = nw = nc = 0;
    while ((c = getchar()) != EOF) {
        ++nc;
        if (c == '\n')
            ++nl;
        if (c == ' ' || c == '\n' || c == '\t')
            state = OUT;
        else if (state == OUT) {
            state = IN;
            ++nw;
        }
    }
    printf("%d %d %d\n", nl, nw, nc);
}
```

**Logic:**

- `state` track karta hai ki hum abhi word ke **andar** hain ya **bahar**
- Jab space/newline/tab mile → `state = OUT`
- Jab koi aur character mile aur state OUT ho → naya word shuru (`state = IN`, `++nw`)

**Operators:**

- `||` = OR (koi bhi ek true ho)
- `&&` = AND (dono true honge)
- `nl = nw = nc = 0` — multiple assignment ek saath (right se left hoti hai)

------

## 1.6 — Arrays

```c
#include <stdio.h>

main()
{
    int c, i, nwhite, nother;
    int ndigit[10];

    nwhite = nother = 0;
    for (i = 0; i < 10; ++i)
        ndigit[i] = 0;

    while ((c = getchar()) != EOF)
        if (c >= '0' && c <= '9')
            ++ndigit[c-'0'];
        else if (c == ' ' || c == '\n' || c == '\t')
            ++nwhite;
        else
            ++nother;

    printf("digits =");
    for (i = 0; i < 10; ++i)
        printf(" %d", ndigit[i]);
    printf(", white space = %d, other = %d\n", nwhite, nother);
}
```

### Array Kya Hai?

- Ek hi type ke elements ka collection
- `int ndigit[10]` — 10 integers ka array (index 0 se 9 tak)
- `ndigit[0]` pehla element, `ndigit[9]` aakhri element

### Character Arithmetic:

- `'0'`, `'1'` etc. ke **numeric values** hote hain (ASCII)
- `c - '0'` ek digit character ko uske integer value mein convert karta hai
- Example: agar `c = '5'`, toh `c - '0' = 5`

------

## 1.7 — Functions

Functions C ka ek core feature hai. Ek kaam ko baar baar karna ho toh function banao.

```c
#include <stdio.h>

int power(int m, int n);  /* Function prototype */

main()
{
    int i;
    for (i = 0; i < 10; ++i)
        printf("%d %d %d\n", i, power(2,i), power(-3,i));
    return 0;
}

/* power: m ko n-th power tak raise karta hai */
int power(int base, int n)
{
    int i, p;
    p = 1;
    for (i = 1; i <= n; ++i)
        p = p * base;
    return p;
}
```

### Function ke Parts:

```
return-type  function-name(parameter list)
{
    declarations
    statements
}
```

### Important Concepts:

**Function Prototype:**

```c
int power(int m, int n);
```

- `main` se pehle likhte hain taaki compiler ko pata ho function kaisa dikhega
- Parameter names optional hain — `int power(int, int);` bhi valid hai

**Return Statement:**

```c
return p;
```

- Function yeh value caller ko wapas deta hai
- `return 0;` main mein — program successfully khatam hua

**Local Variables:**

- Function ke andar ke variables sirf usi function mein visible hote hain
- `power` ka `i` aur `main` ka `i` **alag** hain — koi conflict nahi

------

## 1.8 — Arguments: Call by Value

C mein sab arguments **by value** pass hote hain — matlab function ko original variable nahi, uski ek **copy** milti hai.

```c
int power(int base, int n)
{
    int p;
    for (p = 1; n > 0; --n)
        p = p * base;
    return p;
}
```

Yahaan `n` ko seedha modify kiya — caller ka `n` affect nahi hoga. Function apni local copy use karta hai.

**Arrays alag hote hain:** Jab array pass karo toh sirf **address** jaata hai — elements ki copy nahi. Isliye function array ko modify kar sakta hai.

------

## 1.9 — Character Arrays (Strings)

C mein strings **character arrays** hote hain jo `'\0'` (null character) se end hoti hain.

### Longest Line dhundhne ka program:

```c
#include <stdio.h>
#define MAXLINE 1000

int getline(char line[], int maxline);
void copy(char to[], char from[]);

main()
{
    int len, max = 0;
    char line[MAXLINE];
    char longest[MAXLINE];

    while ((len = getline(line, MAXLINE)) > 0)
        if (len > max) {
            max = len;
            copy(longest, line);
        }
    if (max > 0)
        printf("%s", longest);
    return 0;
}
```

### `getline` function:

```c
int getline(char s[], int lim)
{
    int c, i;
    for (i = 0; i < lim-1 && (c=getchar()) != EOF && c != '\n'; ++i)
        s[i] = c;
    if (c == '\n') {
        s[i] = c;
        ++i;
    }
    s[i] = '\0';  // String ka end mark karo
    return i;
}
```

**Null Character `'\0'`:**

- String ka end mark karta hai
- `printf` ka `%s` isi ko dekhke jaanta hai ki string kahan khatam hui

**`void` return type:**

- `copy` function kuch return nahi karta
- `void` explicitly batata hai

------

## 1.10 — External Variables aur Scope

### Local (Automatic) Variables:

- Function ke andar declare hote hain
- Sirf usi function mein visible
- Function khatam hone par memory free ho jaati hai

### External Variables:

- Kisi bhi function ke bahar declare kiye jaate hain
- **Sab functions** inhe access kar sakte hain
- Program khatam hone tak memory mein rehte hain

```c
int max;               // External variable
char line[MAXLINE];    // External array
char longest[MAXLINE]; // External array

main() {
    extern int max;      // Declare karo ki external hai
    extern char longest[];
    ...
}
```

**Definition vs Declaration:**

| Term            | Matlab                                                       |
| --------------- | ------------------------------------------------------------ |
| **Definition**  | Variable banta hai, memory allocate hoti hai (`int max;` bahar) |
| **Declaration** | Bata do ki variable exist karta hai (`extern int max;` andar) |

### Warning ⚠️ — External Variables Ka Overuse:

- External variables se argument lists chhoti lagti hain, par yeh **bura practice** hai
- Isse program ka data flow unclear ho jaata hai
- Variables unexpected jagah modify ho sakte hain
- Code maintain karna mushkil ho jaata hai

> 💡 **Rule:** Jab tak zaroor na ho, local variables use karo!

------

## 📋 Chapter 1 — Summary (Saari Cheezein Ek Nazar Mein)

| Topic              | Key Concept                                     |
| ------------------ | ----------------------------------------------- |
| Hello World        | `#include`, `main()`, `printf()`                |
| Variables          | `int`, `float`, `char`, `long`, `double`        |
| Loops              | `while`, `for`                                  |
| Symbolic Constants | `#define NAME value`                            |
| I/O                | `getchar()`, `putchar()`, `printf()`, `scanf()` |
| Arrays             | `int arr[10]`, index 0 se shuru                 |
| Functions          | Return type, parameters, `return`               |
| Call by Value      | Copy jaati hai, original safe                   |
| Strings            | `char arr[]` + `'\0'` at end                    |
| Scope              | Local (auto) vs External (global)               |

------

## 🎯 Practice Exercises (Chapter 1 se)

1. **Ex 1-1:** `hello world` program run karo, kuch parts hatao — errors dekho
2. **Ex 1-3:** Temperature table mein heading add karo
3. **Ex 1-4:** Celsius to Fahrenheit table banao
4. **Ex 1-5:** Table reverse order mein print karo (300 se 0)
5. **Ex 1-8:** Spaces, tabs, newlines count karo input mein
6. **Ex 1-15:** Temperature conversion ke liye alag function banao
7. **Ex 1-17:** 80+ character wali lines print karo
8. **Ex 1-19:** `reverse(s)` function banao jo string reverse kare

------

*📚 Reference: "The C Programming Language" by Brian W. Kernighan & Dennis M. Ritchie (2nd Edition)*