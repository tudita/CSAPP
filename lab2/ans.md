反汇编函数`phase_1`

```
   0x08048b42 <+0>:     sub    $0x14,%esp
   0x08048b45 <+3>:     push   $0x804a0e4
   0x08048b4a <+8>:     push   0x1c(%esp)
   0x08048b4e <+12>:    call   0x80490cc <strings_not_equal>
   0x08048b53 <+17>:    add    $0x10,%esp
   0x08048b56 <+20>:    test   %eax,%eax
   0x08048b58 <+22>:    jne    0x8048b5e <phase_1+28>
   0x08048b5a <+24>:    add    $0xc,%esp
   0x08048b5d <+27>:    ret
   0x08048b5e <+28>:    call   0x80491c1 <explode_bomb>
   0x08048b63 <+33>:    jmp    0x8048b5a <phase_1+24>
```

查看`0x08048b45`处的地址`0x804a0e4`，利用gdb查看可知该地址存储了一个字符串：

` "I am for medical liability at the federal level."`

`phase_1`函数开头就调用了函数`string_not_equal`，传入了两个参数:

-  `0x804a02c`:字符串`"I am for medical liability at the federal level."`

-  `0x1c(%esp)`:根据`main`函数中`phase_1`附近的代码可知，此处传入了`read_line`函数的返回值，即用户输入的密码1。

```asm6502
   0x08048a3f <+100>:   call   0x8049221 <read_line>
   0x08048a44 <+105>:   mov    %eax,(%esp)
   0x08048a47 <+108>:   call   0x8048b42 <phase_1>
```

而在`phase_1`中，`string_not_equal`比较了上面两个字符串，若不相同则调用`explode_bomb`,相同则正常退出。进入`phase_2`。

进入`phase_2`函数。首先注意到函数调用了`read_six_number`

```asm6502
   0x080491e6 <+0>:     sub    $0xc,%esp
   0x080491e9 <+3>:     mov    0x14(%esp),%eax
   0x080491ed <+7>:     lea    0x14(%eax),%edx
   0x080491f0 <+10>:    push   %edx
   0x080491f1 <+11>:    lea    0x10(%eax),%edx
   0x080491f4 <+14>:    push   %edx
   0x080491f5 <+15>:    lea    0xc(%eax),%edx
   0x080491f8 <+18>:    push   %edx
   0x080491f9 <+19>:    lea    0x8(%eax),%edx
   0x080491fc <+22>:    push   %edx
   0x080491fd <+23>:    lea    0x4(%eax),%edx
   0x08049200 <+26>:    push   %edx
   0x08049201 <+27>:    push   %eax
   0x08049202 <+28>:    push   $0x804a2c3
   0x08049207 <+33>:    push   0x2c(%esp)
   0x0804920b <+37>:    call   0x8048810 <__isoc99_sscanf@plt>
   0x08049210 <+42>:    add    $0x20,%esp
   0x08049213 <+45>:    cmp    $0x5,%eax
   0x08049216 <+48>:    jle    0x804921c <read_six_numbers+54>
   0x08049218 <+50>:    add    $0xc,%esp
   0x0804921b <+53>:    ret
   0x0804921c <+54>:    call   0x80491c1 <explode_bomb>
```

对其进行反汇编可知，`phase_2`需要输入六个数字，否则将直接爆炸。

`read_six_numbers`执行完毕后栈指针+0x10，而在`phase_2`开头处，程序在栈帧中申请了0x2c字节的空间，如今还剩0x1c个字节。在`read_six_numbers`中读入的六个整数正储存在以`0x4(%esp)`为首地址的栈帧空间中。

```asm6502
   0x08048b87 <+34>:    cmpl   $0x0,0x4(%esp)
   0x08048b8c <+39>:    jne    0x8048b95 <phase_2+48>
   0x08048b8e <+41>:    cmpl   $0x1,0x8(%esp)
   0x08048b93 <+46>:    je     0x8048b9a <phase_2+53>
   0x08048b95 <+48>:    call   0x80491c1 <explode_bomb>
```

由上述代码可知：x1为0，x2为1。

```asm6502
   0x08048b9a <+53>:    lea    0x4(%esp),%ebx
   0x08048b9e <+57>:    lea    0x14(%esp),%esi
   0x08048ba2 <+61>:    jmp    0x8048bab <phase_2+70>
   0x08048ba4 <+63>:    add    $0x4,%ebx
   0x08048ba7 <+66>:    cmp    %esi,%ebx
   0x08048ba9 <+68>:    je     0x8048bbc <phase_2+87>
   0x08048bab <+70>:    mov    0x4(%ebx),%eax
   0x08048bae <+73>:    add    (%ebx),%eax
   0x08048bb0 <+75>:    cmp    %eax,0x8(%ebx)
   0x08048bb3 <+78>:    je     0x8048ba4 <phase_2+63>
   0x08048bb5 <+80>:    call   0x80491c1 <explode_bomb>
```

这段代码告诉我们，这六个数字组成斐波那契数列。

所以答案为：`"0 1 1 2 3 5"`

进入`phase_3`，查看`0x804a13e`的内容：`"%d %c %d"`可知读入的内容

```asm6502
   0x08048bfb <+39>:    call   0x8048810 <__isoc99_sscanf@plt>
   0x08048c00 <+44>:    add    $0x20,%esp
   0x08048c03 <+47>:    cmp    $0x2,%eax
   0x08048c06 <+50>:    jle    0x8048c1e <phase_3+74>
   ............
   0x08048c1e <+74>:    call   0x80491c1 <explode_bomb>
```

```asm6502
   0x08048c08 <+52>:    cmpl   $0x7,0x4(%esp)
   0x08048c0d <+57>:    ja     0x8048d09 <phase_3+309>
   .............
   0x08048d09 <+309>:   call   0x80491c1 <explode_bomb>
```

可知x1小于等于7

```asm6502
   0x08048c13 <+63>:    mov    0x4(%esp),%eax
   0x08048c17 <+67>:    jmp    *0x804a160(,%eax,4)
   0x08048c1e <+74>:    call   0x80491c1 <explode_bomb>
   0x08048c23 <+79>:    jmp    0x8048c08 <phase_3+52>
   0x08048c25 <+81>:    mov    $0x6b,%eax
   0x08048c2a <+86>:    cmpl   $0x65,0x8(%esp)
   0x08048c2f <+91>:    je     0x8048d13 <phase_3+319>
   .............
   0x08048d13 <+319>:   cmp    0x3(%esp),%al
   0x08048d17 <+323>:   je     0x8048d1e <phase_3+330>
   0x08048d19 <+325>:   call   0x80491c1 <explode_bomb>
   0x08048d1e <+330>:   mov    0xc(%esp),%eax
   0x08048d22 <+334>:   xor    %gs:0x14,%eax
   0x08048d29 <+341>:   jne    0x8048d2f <phase_3+347>
   0x08048d2b <+343>:   add    $0x1c,%esp
   0x08048d2e <+346>:   ret
   0x08048d2f <+347>:   call   0x8048790 <__stack_chk_fail@plt>
```

`0x08048c1`处提到了地址`0x804a160`，查看其中内容：

```asm6502
0x804a160:      0x25  0x8c  0x04  0x08  0x44  0x8c  0x04  0x08
```

恰为`0x08048c25 <+81>: mov $0x6b,%eax`

观察后续代码的地址，x1只能为0，且容易推断出x2为0x65，十进制为101。
