# Bug位置:
 1. 源文件xudp_impl.c: 函数 _xudp_impl_destroy() 参见附件1
 2. 源文件xudp.c:      函数 _xudp_destroy() 参见附件2

# Bug描述：
 - 上述两个函数，各自最后3行代码中，先调用free()函数释放内存, 之后又调用子函数 _xxxx_error() 访问已经被释放的内存...
 - 当前版本中暂时不会出问题，后期实现日志函数后会可能导致数据错乱或主程序崩溃.







# 【附件1： BUG代码片段】
```
/**
 * 销毁xudp_impl
 *
 * @param impl
 *  指向xudp 的实现结构体指针
 *
 * @return
 *  - XUDP_IMPL_RETURN_SUCCESS 成功
 *  - XUDP_IMPL_RETURN_ERROR 失败
 */
int _xudp_impl_destroy(xudp_impl *_impl)
{
    if (0 == _impl){
        return _xudp_impl_error(_impl, XERR_NULL_XUDP_IMPL_PTR);
    }

    xfs_free(_impl);
    return _xudp_impl_error(_impl, XERR_NO_ERROR);
}
```

# 【附件2： BUG代码片段】
```
/**
 * 销毁xudp
 *
 * @param _xudp
 * 指向xudp 的结构体指针
 *
 * @return
 *  - 成功返回
 *  - 失败返回XUDP_RETURN_ERROR
 */
int _xudp_destroy(xudp *_xudp)
{
    if (0 == _xudp) {
        return _xudp_error(_xudp, XERR_NULL_XUDP_PTR);
    }

    xudp_internals* internals = (xudp_internals*)_xudp->internals;
    if ( internals->xudp_state_machine != XUDP_T_SHUTDOWN ) {
        return _xudp_error(_xudp, XERR_BAD_STATE);
    }

    internals->impl->destroy(internals->impl);
    xfs_free(internals);

    xfs_free(_xudp);
    return _xudp_error(_xudp, XERR_NO_ERROR);
}
```



# Bug解决方案
 - 将 _xudp_impl_destroy()函数最后3行修改为

		int ret = _impl_error(_impl, XERR_NO_ERROR);
		xfs_free(_impl);
		return ret;

 - 将 _xudp_destroy()函数最后3行修改为:

		int ret = _xudp_error(_xudp, XERR_NO_ERROR);
		xfs_free(_xudp);
		return ret;

# 其他信息
 - Bug发现日期: 2019-08-29
 - Bug补丁提交日期: 2019-08-29 (已合并到git主分支)
