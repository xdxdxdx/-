# 漏洞点
 UAF

# 利用
  1、利用UFA泄露堆地址
  2、利用fastbin attack，修改fd，实现任意写
  3、修改一个chunk的size大小，执行free操作，chunk进入unsorted bin中，此时该chunk的fd和bk指针指向main_arena，可以泄露出main_arena的地址，从而泄露libc的地址
  4、利用fastbin attack修改malloc_hook的地址，从而get shell,malloc的部分源码如下，首先会检查_malloc_hook是否为0，如果不为0，就执行_malloc_hook函数
 ```
 __int64 __fastcall malloc(__int64 a1)
{
  __int64 v2; // rsi@5
  __int64 v3; // rdx@5
  __int64 v4; // rax@12
  int *v5; // rcx@13
  bool v8; // zf@17
  void *retaddr; // [rsp+18h] [rbp+0h]@28

  if ( _malloc_hook )
    return _malloc_hook(a1, retaddr);
  _RBX = *MK_FP(__FS__, 64LL);
 ```

# 通过main_arena泄露libc基址
在fastbin为空时，unsortbin的fd和bk指向自身main_arena，而main_arena存储在libc.so.6文件的.data段，通过这个偏移我们就可以获取libc的基址
## 如何查找main_arena在libc中的偏移
首先使用IDA打开libc文件，然后搜索函数malloc_trim()，具体如下所示,dword_39DB00就为main_arena在libc中的偏移，可以参照malloc.c的源代码来验证。
```
__int64 __fastcall malloc_trim(__int64 a1)
{
  bool v3; // zf@4
  int v4; // er10@9
  volatile signed __int32 *v5; // rdx@10
  unsigned __int64 v7; // r8@17
  unsigned __int64 v8; // r15@19
  signed __int64 v9; // rcx@19
  signed int v10; // ebp@19
  signed __int64 v11; // r13@19
  unsigned __int64 v12; // r12@19
  signed __int64 v13; // r14@19
  __int64 i; // rbx@23
  unsigned __int64 v15; // rdi@27
  int v16; // eax@31
  unsigned __int64 v17; // r12@36
  unsigned __int64 v18; // r12@37
  unsigned __int64 v19; // r12@42
  int v20; // ST90_4@32
  signed int v21; // er12@44
  volatile signed __int32 *v22; // [rsp+8h] [rbp-50h]@3
  signed int v23; // [rsp+10h] [rbp-48h]@18
  unsigned int v24; // [rsp+14h] [rbp-44h]@3
  __int64 v25; // [rsp+18h] [rbp-40h]@1

  v25 = a1;
  if ( dword_39D144 < 0 )
    sub_7D6B0();
  v24 = 0;
  v22 = (volatile signed __int32 *)&dword_39DB00;
  do
  {
```

## 怎么搜索one gadget
  搜索字符串‘/bin/sh’

# 参考资料
0ctf 2017 babyheap
