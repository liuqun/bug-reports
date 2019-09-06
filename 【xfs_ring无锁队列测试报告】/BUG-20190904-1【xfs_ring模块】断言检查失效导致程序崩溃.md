# Bug描述：
无锁队列内部断言xfs_assert_fatal()逻辑判断条件错误，enqueue/dequeue时直接触发断言异常导致程序崩溃...

# Bug位置:
 - 文件 xfs_ring_impl.c:
   1. 函数 _xfs_ring_do_enqueue_internal() 参见附件1
   2. 函数 _xfs_ring_do_dequeue_internal() 参见附件2
 - 文件 xfs_assert.c:
   1. 函数 xfs_assert_fatal()  参见附件3

# 其他信息
 - Bug发现日期: 2019-09-04
 - Bug编号（暂定） BUG-20190904-1

# Bug补丁
发现并改正xfs_ring无锁队列实现代码的assert判断条件错误的Bug
 - 补丁代码已经提交至git源代码管理系统, git提交日志: `git log 4bcb034d1b4ea805ae72d6277033d24a5877e0f8`
 - Bug补丁提交日期: 2019-09-06 已经合并至git版本管理系统主分支


# 附件: BUG-20190904-1 相关代码

## 【附件1代码片段：函数 _xfs_ring_do_enqueue_internal()】
```
/**
 * @internal Enqueue an new entry on the ring
 *
 * @param _exchanger
 *   A pointer to structured parameters
 * @return
 *   Noreturn.
 */
void _xfs_ring_do_enqueue_internal(fcall_exchanger* _exchanger)
{
    XFS_ASSERT( _exchanger);

    xfs_ring_info* info = _exchanger->info;

    /* make reservation for enqueue. */
    _xfs_ring_internal_move_product_head(_exchanger);
    if ( _exchanger->return_value == 0){
        //error: ring queue full,
        return;
    }

    /* memory offset = pointer x cell_size. */
    uint32_t offset = (info->mask  & _exchanger->current_head) << info->alignment;
    xfs_assert_fatal( (offset + info->increment >= info->size) );

    /* store object into the cell. */
    xfs_memcopy(&(info->queue[offset]), _exchanger->object, info->increment);

    _xfs_ring_internal_update_tail(_exchanger);
    _exchanger->availables--;
 }
```

## 【附件2代码片段：函数 _xfs_ring_do_dequeue_internal()】
```
/**
 * @internal Dequeue a objects from the ring
 *
 * @param _exchanger
 *   A pointer to structured parameters
 * @return
 *   Noreturn.
 */
void _xfs_ring_do_dequeue_internal(fcall_exchanger* _exchanger)
{
    XFS_ASSERT( _exchanger);

    xfs_ring_info* info = _exchanger->info;

    /* make reservation for enqueue. */
    _xfs_ring_internal_move_consume_head(_exchanger);
    if ( _exchanger->return_value == 0){
        //error,
        return;
    }

    /* memory offset = pointer x cell_size*/
    uint32_t offset = (info->mask & _exchanger->current_head) << info->alignment;
    xfs_assert_fatal( (offset + info->increment >= info->size) );

    /* file object from the cell. */
    xfs_memcopy(_exchanger->object, &(info->queue[offset]), info->increment);

    _xfs_ring_internal_update_tail(_exchanger);
    _exchanger->availables--;
}
```

## 【附件3代码片段： 文件 xfs_assert.c】

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

# 【附件4: Bug补丁详细内容】
```
From: liuqun <liuqun@192.168.1.77>
Date: Wed, 4 Sep 2019 16:24:26 +0800
Subject: [PATCH] 发现并改正xfs_ring无锁队列实现代码的assert判断条件错误的Bug
MIME-Version: 1.0
Content-Type: text/plain; charset=UTF-8
Content-Transfer-Encoding: 8bit

---
 lib/ring/lockfree/xfs_ring_impl.c | 4 ++--
 1 file changed, 2 insertions(+), 2 deletions(-)

diff --git a/lib/ring/lockfree/xfs_ring_impl.c b/lib/ring/lockfree/xfs_ring_impl.c
index d395cbf..9ac2a7d 100644
--- a/lib/ring/lockfree/xfs_ring_impl.c
+++ b/lib/ring/lockfree/xfs_ring_impl.c
@@ -267,7 +267,7 @@ void _xfs_ring_do_enqueue_internal(fcall_exchanger* _exchanger)

     /* memory offset = pointer x cell_size. */
     uint32_t offset = (info->mask  & _exchanger->current_head) << info->alignment;
-    xfs_assert_fatal( (offset + info->increment >= info->size) );
+    xfs_assert_fatal( (offset + info->increment <= info->size) );

     /* store object into the cell. */
     xfs_memcopy(&(info->queue[offset]), _exchanger->object, info->increment);
@@ -299,7 +299,7 @@ void _xfs_ring_do_dequeue_internal(fcall_exchanger* _exchanger)

     /* memory offset = pointer x cell_size*/
     uint32_t offset = (info->mask & _exchanger->current_head) << info->alignment;
-    xfs_assert_fatal( (offset + info->increment >= info->size) );
+    xfs_assert_fatal( (offset + info->increment <= info->size) );

     /* file object from the cell. */
     xfs_memcopy(_exchanger->object, &(info->queue[offset]), info->increment);
-- 
2.17.1
```