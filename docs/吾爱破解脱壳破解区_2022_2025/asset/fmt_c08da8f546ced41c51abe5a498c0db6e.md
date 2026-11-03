
```cpp
    uint8_t uid[]{ 0x50, 0x04, 0x23, 0x00, 0x00, 0x00, 0x00, 0x00 };

    // 动调得到MD5四个字节，加上最后4个0x04填充

    uint8_t md5_and_pad[]{ 0xD2, 0x63, 0xE4, 0xE6, 0x04, 0x04, 0x04, 0x04 };

    // 半时整点时间戳计算

    FILETIME time{};

    DWORD64 timestamp{};

    GetSystemTimeAsFileTime(&time);

    memcpy((void*)(&timestamp), (void*)(&time), 8);

    timestamp = 1800 * ((timestamp / 0x989680 - 0x2B6109100LL) / 0x708);

    memcpy((void*)(timestamp_bytes), (void*)(&timestamp), 8);

    // 加密数据

    tea_encrypt((uint32_t*)timestamp_bytes, (uint32_t*)key1);

    tea_encrypt((uint32_t*)uid, (uint32_t*)key2);

    tea_encrypt((uint32_t*)md5_and_pad, (uint32_t*)key3);

    printf("flag{");

    for (int i = 0; i 
```

## 心得

动调调试分析程序主体流程很重要，要先了解大概执行框架才能逐步往下层分析，且上层一些代码条件有助于下层的分析。

然后踩了一个严重的坑就是IDA伪代码里面的变量值和实际值一些情况下是不一样的，之前写题没在意那么多，这次很多地方都发现有这种问题，卡了我分析好久。所以关键代码段最好用汇编逐步分析，看实际数据的变化。
