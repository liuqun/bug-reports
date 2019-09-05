# Bug描述：
无锁队列内部断言xfs_assert_fatal()逻辑判断条件错误，enqueue/dequeue时直接触发断言异常导致程序崩溃...

# Bug位置:
 - 文件 xfs_ring_impl.c:
   1. 函数 _xfs_ring_do_enqueue_internal() 参见附件1
   2. 函数 _xfs_ring_do_dequeue_internal() 参见附件2
 - 文件 xfs_assert.c:
   1. 函数 xfs_assert_fatal()  参见附件3


# 【附件1： BUG代码片段1】
```
void _xfs_ring_do_enqueue_internal(fcall_exchanger* _exchanger)
{
     /* memory offset = pointer x cell_size. */
     uint32_t offset = (info->mask  & _exchanger->current_head) << info->alignment;
-    xfs_assert_fatal( (offset + info->increment >= info->size) );
+    xfs_assert_fatal( (offset + info->increment <= info->size) );
 
     /* store object into the cell. */
     xfs_memcopy(&(info->queue[offset]), _exchanger->object, info->increment);
}
```
# 【附件1： BUG代码片段2】
```
void _xfs_ring_do_dequeue_internal(fcall_exchanger* _exchanger)
 
     /* memory offset = pointer x cell_size*/
     uint32_t offset = (info->mask & _exchanger->current_head) << info->alignment;
-    xfs_assert_fatal( (offset + info->increment >= info->size) );
+    xfs_assert_fatal( (offset + info->increment <= info->size) );
 
     /* file object from the cell. */
     xfs_memcopy(_exchanger->object, &(info->queue[offset]), info->increment);
```

# 【附件3： 与当前BUG相关联的其他代码片段】

### 文件 xfs_assert.c: ###

```
#include "xfs_assert.h"

void xfs_assert( int bool_exp )
{
    assert(bool_exp);
}

void xfs_assert_fatal( int bool_exp )
{
    assert(bool_exp);
}

```

# 其他信息
 - Bug发现日期: 2019-09-04

# Bug补丁:
 - Git commit SHA1 ID: d12ea0c70125e17b9f512978001f61ae11095397 (发现并改正xfs_ring无锁队列实现代码的assert判断条件错误的Bug)
 - 提交日期: 2019-09-04
