

# Assembler x86-64

<table width=100%> <tr>
    <th width=20%> <b>Видеозапись семинара &rarr; </b> </th>
    <th>
    <a href="https://www.youtube.com/watch?v=i_eeouEiXnI&list=PLjzMm8llUm4AmU6i_hPU0NobgA4VsBowc&index=8">
        <img src="video.jpg" width="320"  height="160" align="left" alt="Видео с семинара"> 
    </a>
    </th>
    <th> </th>
 </table>

## Особенности
* Много регистров
* Много команд
* Много легаси
* Разные синтаксисы


[Ридинг Яковлева](https://github.com/victor-yacovlev/mipt-diht-caos/tree/master/practice/asm/x86_basics) 


Сегодня в программе:
* <a href="#regs" style="color:#856024"> Регистры </a>
* <a href="#syntax" style="color:#856024"> Синтаксисы </a>
* <a href="#clamp" style="color:#856024"> Функция clamp </a>
* <a href="#mem" style="color:#856024"> Работа с памятью </a>
* <a href="#call" style="color:#856024"> Вызов функций </a>
* <a href="#mul" style="color:#856024"> Интересные факты </a>
* <a href="#hw" style="color:#856024"> Комментарии к ДЗ </a>

#  <a name="regs"></a> Регистры

Немного истории

| Год  | Регистры           | Битность | Первый процессор | Комментарий |
|------|--------------------|----------|------------------|-------------|
| 1974 | a, b, c, d         | 8 bit    | Intel 8080       | |
| 1978 | ax, bx, cx, dx     | 16 bit   | Intel 8086       | X - eXtended ([совсем ненадежный источник](https://stackoverflow.com/a/892948))|
| 1985 | eax, ebx, exc, edx | 32 bit   | Intel 80386      | E - extended |
| 2003 | rax, rbx, rcx, rdx | 64 bit   | AMD Opteron      | R - (внезапно) register |


Как оно выглядит сейчас в 64-битных процессорах

<table width="800px" border="1" style="text-align:center; font-family: Courier New; font-size: 10pt">

<tbody><tr>
<td colspan="8" width="25%" style="background:lightgrey">RAX

<td colspan="8" width="25%" style="background:lightgrey">RCX

<td colspan="8" width="25%" style="background:lightgrey">RDX

<td colspan="8" width="25%" style="background:lightgrey">RBX

<tr>
<td colspan="4" width="12.5%">
<td colspan="4" width="12.5%" style="background:lightgrey">EAX

<td colspan="4" width="12.5%">
<td colspan="4" width="12.5%" style="background:lightgrey">ECX

<td colspan="4" width="12.5%">
<td colspan="4" width="12.5%" style="background:lightgrey">EDX

<td colspan="4" width="12.5%">
<td colspan="4" width="12.5%" style="background:lightgrey">EBX

<tr>
<td colspan="6" width="18.75%">
<td colspan="2" width="6.25%" style="background:lightgrey">AX

<td colspan="6" width="18.75%">
<td colspan="2" width="6.25%" style="background:lightgrey">CX

<td colspan="6" width="18.75%">
<td colspan="2" width="6.25%" style="background:lightgrey">DX

<td colspan="6" width="18.75%">
<td colspan="2" width="6.25%" style="background:lightgrey">BX

<tr>
<td colspan="6" width="18.75%">
<td width="3.125%" style="background:lightgrey">AH
<td width="3.125%" style="background:lightgrey">AL

<td colspan="6" width="18.75%">
<td width="3.125%" style="background:lightgrey">CH
<td width="3.125%" style="background:lightgrey">CL

<td colspan="6" width="18.75%">
<td width="3.125%" style="background:lightgrey">DH
<td width="3.125%" style="background:lightgrey">DL

<td colspan="6" width="18.75%">
<td width="3.125%" style="background:lightgrey">BH
<td width="3.125%" style="background:lightgrey">BL
</tbody></table>

(На самом деле все далеко не так просто устроено. [stackoverflow](https://stackoverflow.com/a/25456097))

Регистры x86 и их странные названия
* RAX - Accumulator Register
* RBX - Base Register
* RCX - Counter Register
* RDX - Data Register
* RSI - Source Index
* RDI - Destination Index
* RBP - Base Pointer
* RSP - Stack Pointer
* R8...R15 - дополнительные регистры общего назначения

Регистры в x86-64:
    
<br> `rax`, `rbx`, `rcx`, `rdx` - регистры общего назначения.
<br> `rsp` - указатель на вершину стека
<br> `rbp` - указатель на начало фрейма (но можно использовать аккуратно использовать как регистр общего назначения)
<br> `rsi`, `rdi` - странные регистры для копирования массива, по сути регистры общего назначения, но ограниченные в возможностях.
<br> `r8`...`r15`

Целочисленные аргументы передаются последовательно в регистрах: `rdi`, `rsi`, `rdx`, `rcx`, `r8`, `r9`. Если передается более 6 аргументов, то оставшиеся - через стек.
Вещественные аргументы передаются через регистры `xmm0`...`xmm7`.

Возвращаемое значение записывается в регистр `rax`.

Вызываемая функция **обязана сохранять на стеке значения регистров** общего назначения `rbx`, `rbp`, `r12`...`r15`. ([Почему `rbx` в этом ряду?](https://stackoverflow.com/questions/22214208/x86-assembly-why-is-ebx-preserved-in-calling-conventions))

Кроме того, при вызове функции для 64-разрядной архитектуры есть дополнительное требование - перед вызовом функции стек должен быть выровнен по границе 16 байт, то есть необходимо уменьшить значение rsp таким образом, оно было кратно 16. Если кроме регистров задействуется стек для передачи параметров, то они должны быть прижаты к нижней выровненной границе стека.

Для функций гарантируется 128-байтная "красная зона" в стеке ниже регистра rsp - область, которая не будет затронута внешним событием, например, обработчиком сигнала. Таким образом, можно задействовать для адресации локальных переменных память до rsp-128.


```python
%%save_file asm_filter_useless
%run chmod +x asm_filter_useless
#!/bin/bash
grep -v "^\s*\." | grep -v "^[0-9]"
```

#  <a name="syntax"></a> Syntaxes
### AT&T


```cpp
%%cpp att_example.c
%run gcc -m64 -masm=att -O3 att_example.c -S -o att_example.S
%run cat att_example.S | ./asm_filter_useless

#include <stdint.h>

//  rdi, rsi, rdx, rcx, r8, r9
int64_t sum(int32_t a, int32_t b, int32_t c, int32_t d, int32_t e, int32_t f, int32_t g, int64_t h) {
    return a + b + c + d + e + f + g + h;
}
```

### Intel

QWORD PTR — это переменная типа четверного слова, DWORD PTR — это переменная типа двойного слова. Слово — это 16 бит. Термин получил распространение в эпоху 16-ти битных процессоров, тогда в регистр помещалось ровно 16 бит. Такой объем информации стали называть словом (word). Т. е. в нашем случае dword (double word) 2*16 = 32 бита = 4 байта (обычный int). 

https://habr.com/ru/post/344896/


```cpp
%%cpp intel_example.c
%run gcc -m64 -masm=intel -O3 intel_example.c -S -o intel_example.S
%run cat intel_example.S | ./asm_filter_useless

#include <stdint.h>
    
//  rdi, rsi, rdx, rcx, r8, r9
int64_t sum(int32_t a, int32_t b, int32_t c, int32_t d, int32_t e, int32_t f, int32_t g, int64_t h) {
    return a + b + c + d + e + f + g +  h;
}
```

Про `endbr64` [Введение в аппаратную защиту стека / Хабр](https://habr.com/ru/post/494000/) и [control-flow-enforcement-technology](https://software.intel.com/sites/default/files/managed/4d/2a/control-flow-enforcement-technology-preview.pdf)

Про `cdqe` = `cltq` (это синонимы) - расширяет знаковое 32-битное до 64-битного знакового числа.

TLDR: чтобы хакерам было сложнее, есть особый режим процессора, в котором переход (jump) к инструкции не являющейся `endbr*` приводит к прерыванию и завершению программы.

#  <a name="clamp"></a> Пишем функцию clamp тремя способами


```python
%%asm clamp_disasm.S
.intel_syntax noprefix
.text
.globl clamp    
clamp:
    endbr64
    // x - edi
    // a - esi
    // b - edx
    mov eax, esi // eax = a
    cmp edi, esi // x ? a
    jl .clamp_return // if (x < a) goto .clamp_return;
    cmp edi, edx // x ? b
    mov eax, edx // eax = b
    cmovle eax, edi // if (x <= b) eax = x;
.clamp_return:
    ret // return eax;
```


```python
%%asm clamp_if.S
.intel_syntax noprefix
.text
.globl clamp
clamp:
    endbr64
    // x - edi
    // a - esi
    // b - edx
    mov eax, esi // eax = a
    cmp edi, esi // x ? a
    jl .clamp_return // if (x < a) goto .clamp_return;
    mov eax, edx
    cmp edi, edx // x ? b
    jg .clamp_return // if (x > b) goto .clamp_return;
    mov eax, edi // eax = x
.clamp_return:
    ret // return eax;
```


```python
%%asm clamp_cmov.S
.intel_syntax noprefix
.text
.globl clamp    
clamp:
    endbr64
    // x - edi
    // a - esi
    // b - edx
    mov eax, edi // eax = x
    cmp eax, esi // x ? a
    cmovl eax, esi // if (x < a) eax = a
    cmp eax, edx // x ? b
    cmovg eax, edx // if (x > b) eax = b;
    ret // return eax;
```


```cpp
%%cpp clamp.c
%run # gcc -S -m64 -masm=intel -O2 clamp.c -o /dev/stdout
#include <stdint.h>

int32_t clamp(int32_t x, int32_t a, int32_t b) {
    if (x < a) return a;
    if (x > b) return b;
    return x;
}
```


```cpp
%%cpp clamp_test.c
// compile and test using all three asm clamp implementations
%run gcc -m64 -masm=intel -O2 clamp_disasm.S clamp_test.c -o clamp_test.exe
%run ./clamp_test.exe
%run gcc -m64 -masm=intel -O2 clamp_if.S clamp_test.c -o clamp_if_test.exe
%run ./clamp_if_test.exe
%run gcc -m64 -masm=intel -O2 clamp_cmov.S clamp_test.c -o clamp_cmov_test.exe
%run ./clamp_cmov_test.exe
%run gcc -m64 -masm=intel -O2 clamp.c clamp_test.c -o clamp_cmov_c_test.exe
%run ./clamp_cmov_c_test.exe

#include <stdint.h>
#include <stdio.h>
#include <assert.h>
    

int32_t clamp(int32_t x, int32_t a, int32_t b);

int main() {
    assert(clamp(1, 10, 20) == 10);
    assert(clamp(100, 10, 20) == 20);
    assert(clamp(15, 10, 20) == 15);
    fprintf(stderr, "All is OK");
    return 0;
}
```

То же самое ассемблерной вставкой


```cpp
%%cpp clamp_inline_test.c
%run gcc -m64 -masm=intel -O2 clamp_inline_test.c -o clamp_inline_test.exe
%run ./clamp_inline_test.exe

#include <stdint.h>
#include <stdio.h>
#include <assert.h>
    
int32_t clamp(int32_t a, int32_t b, int32_t c);
__asm__(R"(
clamp:
    endbr64
    mov eax, esi
    cmp edi, esi
    jl .clamp_return
    cmp edi, edx
    mov eax, edx
    cmovle eax, edi
.clamp_return:
    ret
)");

int main() {
    assert(clamp(1, 10, 20) == 10);
    assert(clamp(100, 10, 20) == 20);
    assert(clamp(15, 10, 20) == 15);
    fprintf(stderr, "All is OK");
    return 0;
}
```

#  <a name="mem"></a> Поработаем с памятью

Даны n, x. Посчитаем $\sum_{i=0}^{n - 1} (-1)^i \cdot x[i]$


```python
%%asm my_sum.S
.intel_syntax noprefix
.text
.globl my_sum
my_sum:
    // n - edi
    // x - rsi
    mov eax, 0
    cmp edi, 0
start_loop:   
    jle return_eax
    add eax, DWORD PTR [rsi]
    add rsi, 4
    dec edi // and write compare with 0 flags
    
    jle return_eax
    sub eax, DWORD PTR [rsi]
    add rsi, 4
    dec edi // and write compare with 0 flags
    
    jmp start_loop
return_eax:
    ret
```


```cpp
%%cpp my_sum_test.c
%run gcc -g3 -m64 -masm=intel my_sum_test.c my_sum.S -o my_sum_test.exe
%run ./my_sum_test.exe

#include <stdint.h>
#include <stdio.h>
#include <assert.h>
    
int32_t my_sum(int32_t n, int32_t* x);

int main() {
    int32_t x[] = {100, 2, 200, 3};
    assert(my_sum(sizeof(x) / sizeof(int32_t), x) == 100 - 2 + 200 - 3);
    int32_t y[] = {100, 2, 200};
    assert(my_sum(sizeof(y) / sizeof(int32_t), y) == 100 - 2 + 200);
    int32_t z[] = {100};
    assert(my_sum(sizeof(z) / sizeof(int32_t), z) == 100);
    printf("SUCCESS");
    return 0;
}
```


```python

```


# <a name="call"></a> Вызов функций


```cpp
%%cpp example.c
%run gcc -m64 -masm=intel -Os example.c -S -o example.S
%run cat example.S 

#include <stdio.h>
    
int hello(int a, int b) {
    printf("Hello %d and %d\n", a, b);
    return a;
}
```


```cpp
%%cpp print.c
%run gcc -m64 -masm=intel -O3 print.c -o print.exe
%run ./print.exe

#include <stdio.h>
#include <stdint.h>
#include <assert.h>
    
// int hello(int a, int b) {
//     printf("Hello %d and %d\n", a, b);
//     return a;
// }
    
int hello(int a, int b);
__asm__(R"(
.format_s:
   .string "Hello %d and %d\n"
hello:
    /* rdi, rsi, rdx, rcx */
    push rdi /* сохраняем a и заодно обеспечиваем выравнивание по 16 байт для вызываемой функции */
    /* готовим аргументы для printf */
    mov rdx, rsi
    mov rsi, rdi
    lea rdi, .format_s[rip] /* вычисляем адрес форматной строки (он в данном случае относительный, а не абсолютный) */
    call printf@PLT /* вызываем функцию */
    pop rax /* восстанавливаем a и готовимся его возвращать */
    ret
)");


int main() {
    hello(1, 2);
    hello(10, 20);
    printf("SUCCESS\n");
}
```

#  <a name="mul"></a> Развлекательно-познавательная часть



```cpp
%%cpp mul.c
%run gcc -m64 -masm=intel -O1 mul.c -S -o mul.S
%run cat mul.S | ./asm_filter_useless

#include <stdint.h>
    
int32_t mul(int32_t a) { 
    return a * 5;
}
```


```cpp
%%cpp div_0.c
%run gcc -m64 -masm=intel -O3 div_0.c -S -o div_0.S
%run cat div_0.S | ./asm_filter_useless

#include <stdint.h>
    
uint32_t div(uint32_t a) { 
    return a / 12312423;
}

uint32_t div2(uint32_t a, uint32_t b) { 
    return a / b;
}
```


```cpp
%%cpp div.c
%run gcc -m64 -masm=intel -O3 div.c -S -o div.S
%run cat div.S | ./asm_filter_useless

#include <stdint.h>
    
int32_t div(int32_t a) { 
    return a / 4;
}

uint32_t udiv(uint32_t a) { 
    return a / 4;
}
```

```
-1: 1111 -> 1111
-2: 1110 -> 1111
-3: 1101 -> 1111
-4: 1100 -> 1111
-5: 1011 -> 1110
```

```
-1: 1111 -> 0010 -> 0
-2: 1110 -> 0001 -> 0
-3: 1101 -> 0000 -> 0
-4: 1100 -> 1111 -> -1
-5: 1011 -> 1110 -> -1
```



```python

```

# <a name="inline"></a> Inline ASM
http://asm.sourceforge.net/articles/linasm.html


```cpp
%%cpp simdiv.c
%run gcc -m64 -masm=intel -O3 simdiv.c -o simdiv.exe
%run ./simdiv.exe

#include <stdio.h>
#include <stdint.h>
#include <assert.h>
    
int32_t simdiv(int32_t a) { 
    uint32_t eax = ((uint32_t)a >> 31) + a;
    __asm__("sar %0" : "=a"(eax) : "a"(eax));
    return eax;
}

int main() {
    assert(simdiv(1) == 0);
    assert(simdiv(5) == 2);
    assert(simdiv(-1) == 0);
    assert(simdiv(-5) == -2);
    printf("SUCCESS\n");
}
```


```python

```


```python

```

##  <a name="hw"></a> Комментарии к дз





```python

```
