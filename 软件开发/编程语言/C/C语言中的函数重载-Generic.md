# \_Generic
\_Generic宏可以实现C++或其他OOP语言中函数重载的特性

```c
#include <stdio.h>
#include <stdlib.h>

void testi(int a) {
    printf("int : %d\n", a);
    return;
}

void testf(float a) {
    printf("float : %f\n", a);
    return;
}

#define test(a) _Generic(a, int: testi(a), float: testf(a))

int main(void)
{
    test(1);
    test(3.5f);
    return EXIT_SUCCESS;
}
```