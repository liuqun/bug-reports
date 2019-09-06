# Bug位置:
 1. 源文件xdp_host_table.c: 函数 new_xdp_host_table() (备注:修复补丁已经上传至主干版本, git提交日至位于commit ee3469)


# Bug描述：
 - 函数 new_xdp_host_table() 初始化所有结构体成员变量的过程中, 忘记初始化destroy函数指针. 导致主程序无法正常关闭.

# Bug补丁及其他信息
修复函数 new_xdp_host_table(), 将其中的destroy函数指针关联到API函数 xdp_host_table_destroy()上.

 - Bug发现日期: 2019-08-29
 - Bug补丁提交日期: 2019-08-29 (已合并到git主分支,git提交日至位于commit ee3469)

# 追加单元测试
追加一个单元测试用例, 伪代码片段如下:
```
TEST(XDtlsHostTest, check_all_public_and_private_functions)
{
    EXPECT_EQ(&xdp_host_table_destroy, p->xdp_host_table->destroy)
        << "API function pointer check failed! xdp_host_table->destroy != xdp_host_table_destroy";
}
```


# 【附件1：Bug补丁信息】
**git命令**: `git diff 7446d..ee3469`输出的补丁如下
```
diff --git a/src/xdtls_host/xdp_host_table.c b/src/xdtls_host/xdp_host_table.c
index 520c85d..9918ec7 100644
--- a/src/xdtls_host/xdp_host_table.c
+++ b/src/xdtls_host/xdp_host_table.c
@@ -334,6 +334,7 @@ xdp_host_table *new_xdp_host_table(void)
     table->configure = xdp_host_table_configure;
     table->startup = xdp_host_table_startup;
     table->shutdown = xdp_host_table_shutdown;
+    table->destroy = xdp_host_table_destroy;
     table->add = xdp_host_table_add;
     table->remove = xdp_host_table_remove;
     table->find = xdp_host_table_find;
```
