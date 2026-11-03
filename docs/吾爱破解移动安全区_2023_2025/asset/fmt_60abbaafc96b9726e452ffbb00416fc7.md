
```
unsigned count = txn->offsets_size / sizeof(binder_size_t);

while (0 data.ptr.buffer + (int)(*offs), sizeof(binder_size_t))) {

        *offs += _off;

    }

    ++ offs;

}

}
```

### 3、内存释放。

需要保存原地址`A`和新的地址`AA`的映射关系到自定义的内存池中。

当Binder通信命令出现`BC_FREE_BUFFER`和`BR_FREE_BUFFER`时，则通过该命令要释放的`AA`地址，然后从内存池找到与之对应`A`的地址，并设置回去让上层继续释放，完成内存使用的闭环。

```
case BC_FREE_BUFFER:
{
    uintptr_t* buffPtr = (uintptr_t *)cmd;
    uintptr_t ptr = MemPool::detach(*buffPtr);
    if (__UNLIKELY(0 != ptr)) {
        *buffPtr = ptr;// set origin buffer
    }
    cmd += sizeof(uintptr_t);// move to next command
}   break;
```

### 4> 附：

如果你有需要，可以直接使用我们已经封装好的SDK来实现相应的功能，该项目已经开源，可以直接使用，参考[`//github.com/iofomo/binderceptor/blob/main/doc/sdk.md`](https://github.com/iofomo/binderceptor/blob/main/doc/sdk.md)。
