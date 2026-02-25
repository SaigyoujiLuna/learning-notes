“指针的 Nanbox” 是一个在编程语言实现（尤其是动态类型语言如 JavaScript、Lua、Python 等）中用于高效存储和区分不同类型数据的技巧，通常用于虚拟机或解释器的实现中。它的全称是 **NaN Boxing**（或 NaN Tagging），是一种利用 IEEE 754 浮点数标准中“NaN”（Not a Number）的特殊位模式来编码多种数据类型的技术。
# 为什么需要NaNBoxing
在动态类型语言中，变量可以在运行时改变类型（例如，一个变量可以是整数、浮点数、字符串、布尔值、指针等）。为了在内存中统一表示这些不同类型的数据，通常有两种方式：

1. **Tagged Union（标签联合体）**：用一个结构体包含一个类型标签和一个联合体（union），占用较多内存。
2. **Pointer + Heap Allocation**：所有非基本类型都分配在堆上，用指针引用，但频繁分配影响性能。

而 **Nanbox** 是一种更高效的替代方案：**用一个 64 位值（如 `uint64_t` 或 `double`）来同时编码类型信息和数据**，无需额外的标签字段或堆分配。
# IEEE 754双精度浮点数的结构
```gherkin
| 1 bit 符号 | 11 bits 指数 | 52 bits 尾数（有效数字） |
```
当指数全为 1（即 `0x7FF`）时，表示特殊值：
- 尾数为 0：表示 `+∞` 或 `-∞`
- 尾数非 0：表示 **NaN**（Not a Number）
IEEE 754 规定，**所有 NaN 值中，只有指数全为 1 且尾数非 0 的才是 NaN**。

更重要的是：**NaN 有很多很多种位模式**（2^52 - 1 种），而实际程序中使用的 NaN 很少。这为我们提供了“空闲”的位模式，可以用来编码其他数据。

**利用 NaN 的“空闲”位模式来编码非浮点数类型**，例如：

- 指针（32/64 位）
- 整数（32/52 位）
- 布尔值
- 空值（null）
- 类型标签

 关键技巧：
1. **所有非浮点数都编码为“安静的 NaN”（Quiet NaN）**。
2. 在 NaN 的尾数部分（52 位）中，**最高位（第 51 位）通常用于区分“信号 NaN”和“安静 NaN”**。我们使用安静 NaN。
3. 在尾数的剩余位中，**嵌入类型信息和数据**。
# 副作用
1. 使用NaNBox技术后，会导致可用地址空间减少，会导致软件可用内存降低到2^ 48
2. 为了避免系统分配高地址空间与高16位冲突，需要使用mmap显式请求低地址范围的内存或者预先申请内存，避免获得过高的地址
# 示例
```gherkin
[1][11 bits: 0x7FF][1][4 bits tag][48 bits data]
 ^    ^              ^    ^        ^
 |    |              |    |        |
 |    |              |    |        +-- 指针或整数
 |    |              |    +----------- 类型标签
 |    |              +---------------- 安静 NaN 标志
 |    +------------------------------- 指数（NaN）
 +------------------------------------ 符号位（可忽略）
```

C语言实现如下

```c
#include <stdbool.h>
#include <stdint.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define QNAN 0x7ff8000000000000

#define TYPE_NIL 0x0000
#define TYPE_BOOL 0x0001
#define TYPE_OBJECT 0x0002

typedef enum {
    NIL,
    BOOL,
    NUMBER,
    OBJECT,
} Type;

typedef enum { STRING, ARRAY } Obj_Type;

typedef struct {
    Obj_Type type;
    char *data;
} Obj;

Obj *create_string(const char *str) {
    Obj *obj = (Obj *)calloc(1, sizeof(Obj));
    obj->type = STRING;
    obj->data = strdup(str);
    return obj;
}
typedef struct {
    uintptr_t raw;
} Value;

uintptr_t create_number_raw(double num) {
    uintptr_t raw;
    memcpy(&raw, &num, sizeof(double));
    return raw;
}
#define NIL_VAL {.raw = QNAN}
#define NUMBER_VAL(a) {.raw = create_number_raw(a)}
#define BOOL_VAL(a)                                                            \
    {.raw =                                                                    \
         QNAN | ((uintptr_t)TYPE_BOOL << 48) | (uintptr_t)(a == true ? 1 : 0)}
#define OBJ_VAL(a) {.raw = QNAN | ((uintptr_t)TYPE_OBJECT << 48) | (uintptr_t)a}

void print_value(Value value) {
    if ((value.raw & QNAN) != QNAN) {
        double number;
        memcpy(&number, &value.raw, sizeof(double));
        printf("%lf\n", number);
        return;
    }
    uint64_t type = (value.raw & ~QNAN) >> 48;
    uint64_t origin = value.raw & ~QNAN & ~((uintptr_t)type << 48);
    switch (type) {
    case TYPE_NIL:
        printf("NIL\n");
        break;
    case TYPE_BOOL:
        printf("%s\n", (origin == 1) ? "true" : "false");
        break;
    case TYPE_OBJECT: {
        Obj *obj = (Obj *)origin;
        switch (obj->type) {
        case STRING:
            printf("%s\n", obj->data);
            break;
        case ARRAY:
            printf("Array: %s\n", obj->data);
            break;
        }
    }
    break;
    };
}
int main(int argc, char *argv[]) {
    Value v1 = NUMBER_VAL(3.14);
    Value v2 = BOOL_VAL(true);
    Value v3 = NIL_VAL;
    Obj *string = create_string("hello, world");
    Value v4 = OBJ_VAL(string);
    print_value(v1);
    print_value(v2);
    print_value(v3);
    print_value(v4);
    printf("Size of one Value: %zu bytes\n", sizeof(Value));
    printf("Size of array[100]: %zu bytes\n", sizeof(Value) * 100);
    return EXIT_SUCCESS;
}
```