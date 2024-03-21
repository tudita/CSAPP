```
I am for medical liability at the federal level.
0 1 1 2 3 5
0 k 101
14 7
5 115
3 4 6 5 2 1
```

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

到`0x08048c03`处，三个变量的地址分别为：

`0x4(%esp)` `0x3(%esp)` `0x8(%esp)`   分别设为x1,c,x2

```asm6502
   0x08048c08 <+52>:    cmpl   $0x7,0x4(%esp)
   0x08048c0d <+57>:    ja     0x8048d09 <phase_3+309>
   .............
   0x08048d09 <+309>:   call   0x80491c1 <explode_bomb>
```

由上述代码可知x1小于等于0x7

```asm6502
   0x08048c13 <+63>:    mov    0x4(%esp),%eax
   0x08048c17 <+67>:    jmp    *0x804a160(,%eax,4)
   0x08048c1e <+74>:    call   0x80491c1 <explode_bomb>
   0x08048c23 <+79>:    jmp    0x8048c08 <phase_3+52>
   0x08048c25 <+81>:    mov    $0x6b,%eax
   0x08048c2a <+86>:    cmpl   $0x65,0x8(%esp)
   0x08048c2f <+91>:    je     0x8048d13 <phase_3+319>
   0x08048c35 <+97>:    call   0x80491c1 <explode_bomb>
```

`0x804a160`指向地址`0x08048c25`——`0x08048c25 <+81>: mov $0x6b,%eax`

取x1为0，则跳转到上述地址。可知x2应为0x65(101)。

```asm6502
   0x08048d13 <+319>:   cmp    0x3(%esp),%al
   0x08048d17 <+323>:   je     0x8048d1e <phase_3+330>
   0x08048d19 <+325>:   call   0x80491c1 <explode_bomb>
```

可知c的ascii码与%al中的值相同，为0x6b(107) `c = 'k'`

综上phase_3答案为`"0 k 101"`

进入`phase_4`开头读入了一些数据，查看地址`0x804a2cf`的内容，为`"%d %d"`可见读入了两个整数，同时不难分析出，到`0x08048dba <+39>`处时，两个整数储存在`0x4(%esp)` 和`0x8(%esp)`。分别设为x1,x2。

```asm6502
   0x08048df5 <+98>:    cmpl   $0xe,0x4(%esp)
   0x08048dfa <+103>:   jbe    0x8048dc7 <phase_4+52>
   0x08048dfc <+105>:   jmp    0x8048dc2 <phase_4+47>
```

函数随后跳转到这段代码，可知x1应当小于等于0xe。随后pc继续跳转——

```asm6502
   0x08048dc7 <+52>:    sub    $0x4,%esp
   0x08048dca <+55>:    push   $0xe
   0x08048dcc <+57>:    push   $0x0
   0x08048dce <+59>:    push   0x10(%esp)
   0x08048dd2 <+63>:    call   0x8048d34 <func4>
   0x08048dd7 <+68>:    add    $0x10,%esp
   0x08048dda <+71>:    cmp    $0x7,%eax
   0x08048ddd <+74>:    je     0x8048dfe <phase_4+107>
```

接着调用了`func4`，根据后续代码可知返回值应等于0x7，且x2也应为0x7

此时栈中：

```
0xe     12
0x0     8
x1      4 
返回地址 0
---------
%ecx = x1
%eax = 0
%ebx = 14
```

将`func4`逆向为C代码

```c
int func4(int x, int y, int z) {
    int v = ((z - y) / 2) + y;
    if (v > x)
        return 2 * func4(x, y, v - 1);
    y = 0;
    if (v < x)
        return func4(x, v + 1, z) * 2 + 1;
    return y;
}
```

```c
for (int x1 = 0; x1 <= 0xe; x1++) {
        int val = func4(x1, 0, 14);
        cout << x1 << ":" << val << "  ";
}
```

使用上面的循环输出x1各个取值时`func4`的返回值：

`0:0  1:0  2:4  3:0  4:2  5:2  6:6  7:0  8:1  9:1  10:5  11:1  12:3  13:3  14:7`

不难看出`x1 = 14`

所以答案为`14 7`

进入`phase_5`函数，阅读前面的部分，可知`phase_5`要求读入两个整数x1,x2。

```asm
   0x08048e3b <+47>:    mov    0x4(%esp),%eax
   0x08048e3f <+51>:    and    $0xf,%eax
   0x08048e42 <+54>:    mov    %eax,0x4(%esp)
   0x08048e46 <+58>:    cmp    $0xf,%eax
   0x08048e49 <+61>:    je     0x8048e79 <phase_5+109>  
   .............
   0x08048e79 <+109>:   call   0x80491c1 <explode_bomb>
```

可知

链表如图所示：

    <img title="" src="file:///C:/Users/abc/AppData/Roaming/marktext/images/2024-03-20-19-33-41-image.png" alt="" width="441">

要满足要求，`x1 = 5`

`x2`为链表上所有节点数字之和（除第一个节点外），即`115`

答案为`5 115`

进入`phase_6`：开头读入六个整数，到`0x08048ebd`时，六个数字存储在以` 0xc(%esp)`为首地址的24个字节中。

```asm6502
   0x08048ec4 <+41>:    add    $0x1,%esi
   0x08048ec7 <+44>:    cmp    $0x6,%esi
   0x08048eca <+47>:    je     0x8048efa <phase_6+95>
   0x08048ecc <+49>:    mov    %esi,%ebx
   0x08048ece <+51>:    mov    0xc(%esp,%ebx,4),%eax
   0x08048ed2 <+55>:    cmp    %eax,0x8(%esp,%esi,4)
   0x08048ed6 <+59>:    je     0x8048ef3 <phase_6+88>
   0x08048ed8 <+61>:    add    $0x1,%ebx
   0x08048edb <+64>:    cmp    $0x5,%ebx
   0x08048ede <+67>:    jle    0x8048ece <phase_6+51>
   0x08048ee0 <+69>:    mov    0xc(%esp,%esi,4),%eax
   0x08048ee4 <+73>:    sub    $0x1,%eax
   0x08048ee7 <+76>:    cmp    $0x5,%eax
   0x08048eea <+79>:    jbe    0x8048ec4 <phase_6+41>
   0x08048eec <+81>:    call   0x80491c1 <explode_bomb>
   0x08048ef1 <+86>:    jmp    0x8048ec4 <phase_6+41>
   0x08048ef3 <+88>:    call   0x80491c1 <explode_bomb>
   0x08048ef8 <+93>:    jmp    0x8048ed8 <phase_6+61>
```

`%esi`作为访问六个数字的索引值，值在1~6中变化，由上述代码可知，输入的六个数字全都在1~6之间，等到六个数字被全部访问过一遍后，跳出循环。

此外，根据<+49>到<+59>可知，数列中没有两个连续相同的数字。

```cpp
//esi- i ebx-i eax-cnt edx-p ecx-a[i]
for(int i = 0; i <= 5; i++) {
    cnt = 1
    ecx = a[i]
    p = 0x804c13c
    if(a[0] > 1) { //将p移动到第i个节点
        while(a[0]!=cnt) {
            p = *(p + 1);//p = (*p).next;
            cnt++;
        }
    }
//1->2->3->4->5->6
    a[5 + i] = p    
}
b = a+6; //ebx-b()
ptr = b[0];
//eax-i   ecx-ptr
for(int i = 1; i <= 5; i++) {
    ptr->next = b[i];
    ptr = b[i];
}
b[5]->next = NULL;
for(int i = 5; i > 0;i--){
    b[0] = b[0]->next;
       pp = b[0]->next;
    val = *pp
    if(val < *b[0]) bomb();

}
```

根据逆向代码可知，要输入的数列应该是链表上节点降序排序后的编号所组成的数列。

```
0x804c13c <node1>:    0x00000065   0x00000001   0x0804c148  
                      0x00000109   0x00000002   0x0804c154   
                      0x000003ab   0x00000003   0x0804c160   
                      0x00000307   0x00000004   0x0804c16c
                      0x00000121   0x00000005   0x0804c178  
                      0x00000233   0x00000006   0x00000000 
```

则答案为：`3 4 6 5 2 1`

`secret_phase`：

在`bomb.c`的最后有这样一段注释：

```c
    /* Wow, they got it!  But isn't something... missing?  Perhaps
     * something they overlooked?  Mua ha ha ha ha! */
```

这暗示我们有隐藏关卡。我们查看反汇编文件`bomb.s`，发现其中有两个函数`fun7`与`secret_phase`在前面的lab中从未出现过。

现在问题在于：如何找到`secret_phase`的入口？

搜索函数`secret_phase`的地址，可以发现在`phase_defused`中出现了转向`secret_phase`的跳转语句。
