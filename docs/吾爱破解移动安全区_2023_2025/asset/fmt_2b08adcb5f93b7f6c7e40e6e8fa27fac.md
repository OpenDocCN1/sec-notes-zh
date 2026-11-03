
```
# 合并 App Bundle 后的元气骑士
7977268000-7980000000 r--p 00000000 fc:0a 49763                          /data/app/~~6kC6XFUULhHulM01DwI6fA==/com.ChillyRoom.DungeonShooter-_E4Bf_EJNotbVBZxg5eWXQ==/lib/arm64/libil2cpp.so
# ... 此处省略无数行 ...
79cd789000-79cf000000 r--s 00000000 fc:0a 50772                          /data/media/0/Android/data/com.ChillyRoom.DungeonShooter/files/il2cpp/Metadata/global-metadata.dat
# ... 此处省略无数行 ... 下面就是我们要找的
7aa5415000-7aad76b000 r-xp 00000000 fc:0a 49763                          /data/app/~~6kC6XFUULhHulM01DwI6fA==/com.ChillyRoom.DungeonShooter-_E4Bf_EJNotbVBZxg5eWXQ==/lib/arm64/libil2cpp.so
7aad76b000-7aad76c000 ---p 00000000 00:00 0 
7aad76c000-7aae199000 rw-p 08356000 fc:0a 49763                          /data/app/~~6kC6XFUULhHulM01DwI6fA==/com.ChillyRoom.DungeonShooter-_E4Bf_EJNotbVBZxg5eWXQ==/lib/arm64/libil2cpp.so
7aae199000-7aae3d8000 rw-p 00000000 00:00 0                              [anon:.bss]
7aae3d8000-7aae3d9000 ---p 00000000 00:00 0 
7aae3d9000-7aae3dc000 r-xp 08d83000 fc:0a 49763                          /data/app/~~6kC6XFUULhHulM01DwI6fA==/com.ChillyRoom.DungeonShooter-_E4Bf_EJNotbVBZxg5eWXQ==/lib/arm64/libil2cpp.so
7aae3dc000-7aae3dd000 ---p 00000000 00:00 0 
7aae3dd000-7aae3df000 rw-p 08d86000 fc:0a 49763                          /data/app/~~6kC6XFUULhHulM01DwI6fA==/com.ChillyRoom.DungeonShooter-_E4Bf_EJNotbVBZxg5eWXQ==/lib/arm64/libil2cpp.so
7aae3df000-7aae3e1000 ---p 00000000 00:00 0 
7aae3e1000-7aae3f5000 rw-p 08d84000 fc:0a 49763                          /data/app/~~6kC6XFUULhHulM01DwI6fA==/com.ChillyRoom.DungeonShooter-_E4Bf_EJNotbVBZxg5eWXQ==/lib/arm64/libil2cpp.so
# ... 此处省略无数行 ...

# 原版元气骑士
# ... 此处省略无数行 ...
729caba000-729cd62000 r-xp 00142000 fc:0a 73347                          /data/app/~~wwI8fTI34usd9DBbBqpP4A==/com.ChillyRoom.DungeonShooter-olvDyD0tf9exDoyTM7Hwgg==/split_config.arm64_v8a.apk
729cd62000-729cd6a000 r--p 003e9000 fc:0a 73347                          /data/app/~~wwI8fTI34usd9DBbBqpP4A==/com.ChillyRoom.DungeonShooter-olvDyD0tf9exDoyTM7Hwgg==/split_config.arm64_v8a.apk
729cd6a000-729cdb2000 rw-p 003f1000 fc:0a 73347                          /data/app/~~wwI8fTI34usd9DBbBqpP4A==/com.ChillyRoom.DungeonShooter-olvDyD0tf9exDoyTM7Hwgg==/split_config.arm64_v8a.apk
# ... 此处省略无数行 ...
73c71c2000-73c8a39000 r--s 00000000 fc:0a 57632                          /data/media/0/Android/data/com.ChillyRoom.DungeonShooter/files/il2cpp/Metadata/global-metadata.dat
# ... 此处省略无数行 ... 下面就是我们要找的
73e3a0c000-73ebd62000 r-xp 00a00000 fc:0a 73347                          /data/app/~~wwI8fTI34usd9DBbBqpP4A==/com.ChillyRoom.DungeonShooter-olvDyD0tf9exDoyTM7Hwgg==/split_config.arm64_v8a.apk
73ebd62000-73ebd63000 ---p 00000000 00:00 0 
73ebd63000-73ec790000 rw-p 08d56000 fc:0a 73347                          /data/app/~~wwI8fTI34usd9DBbBqpP4A==/com.ChillyRoom.DungeonShooter-olvDyD0tf9exDoyTM7Hwgg==/split_config.arm64_v8a.apk
73ec790000-73ec9cf000 rw-p 00000000 00:00 0                              [anon:.bss]
73ec9cf000-73ec9d0000 ---p 00000000 00:00 0 
73ec9d0000-73ec9d3000 r-xp 09783000 fc:0a 73347                          /data/app/~~wwI8fTI34usd9DBbBqpP4A==/com.ChillyRoom.DungeonShooter-olvDyD0tf9exDoyTM7Hwgg==/split_config.arm64_v8a.apk
73ec9d3000-73ec9d4000 ---p 00000000 00:00 0 
73ec9d4000-73ec9d6000 rw-p 09786000 fc:0a 73347                          /data/app/~~wwI8fTI34usd9DBbBqpP4A==/com.ChillyRoom.DungeonShooter-olvDyD0tf9exDoyTM7Hwgg==/split_config.arm64_v8a.apk
73ec9d6000-73ec9d8000 ---p 00000000 00:00 0 
73ec9d8000-73ec9ec000 rw-p 09784000 fc:0a 73347                          /data/app/~~wwI8fTI34usd9DBbBqpP4A==/com.ChillyRoom.DungeonShooter-olvDyD0tf9exDoyTM7Hwgg==/split_config.arm64_v8a.apk
# ... 此处省略无数行 ...

# 创意糕点部
# ... 此处省略无数行 ...
731780a000-7319fb3000 r--p 00000000 fc:0a 68394                          /data/app/~~EeYFpmSrnO1pEm-dVmp9rQ==/net.kairosoft.android.okashi_en-AWdoF0iBkO1WbE3SF5a1qA==/lib/arm64/libil2cpp.so
# ... 此处省略无数行 ...
734a86e000-734b000000 r--s 00000000 fc:0a 69978                          /data/media/0/Android/data/net.kairosoft.android.okashi_en/files/il2cpp/Metadata/global-metadata.dat
# ... 此处省略无数行 ... 下面就是我们要找的
7410624000-7412b11000 r-xp 00000000 fc:0a 68394                          /data/app/~~EeYFpmSrnO1pEm-dVmp9rQ==/net.kairosoft.android.okashi_en-AWdoF0iBkO1WbE3SF5a1qA==/lib/arm64/libil2cpp.so
7412b11000-7412c5b000 r--p 024ec000 fc:0a 68394                          /data/app/~~EeYFpmSrnO1pEm-dVmp9rQ==/net.kairosoft.android.okashi_en-AWdoF0iBkO1WbE3SF5a1qA==/lib/arm64/libil2cpp.so
7412c5b000-7412dce000 rw-p 02636000 fc:0a 68394                          /data/app/~~EeYFpmSrnO1pEm-dVmp9rQ==/net.kairosoft.android.okashi_en-AWdoF0iBkO1WbE3SF5a1qA==/lib/arm64/libil2cpp.so
# ... 此处省略无数行 ...
```

找到了这个规律，我们就可以将其写入脚本中了。顺便呢，再完善一下输入输出，方便调试。

```bash
#!/system/bin/sh

echo "########################"

echo "# IL2Cpp Memory Dumper #"

echo "# by NekoYuzu neko.ink #"

echo "########################"

if [[ $1 == "" ]]; then

        echo "* Usage: $0  [out]"

        exit

fi

package=$1

if [[ $2 == "" ]]; then

        out=/sdcard/dump

else

        out=$2

fi

echo "- Target package: $package"

echo "- Output directory: $out"

mkdir -p "$out"

user=$(am get-current-user)

pid=$(ps -ef | grep $package | grep u$user | awk '{print $2}')

if [[ $pid == "" ]]; then

        echo "! Target package of current user ($user) not found, is process running?"

        exit

fi

echo "- Found target process: $pid"

targets="global-metadata.dat libil2cpp.so "$(getprop ro.product.cpu.abilist | awk -F',' '{for (i = 1; i /dev/null

        if [[ $(cat "${out}/tmp") == $(echo -ne "\x7F\x45\x4C\x46") ]]; then

                fileExt="so"

        else

                fileExt="dump"

        fi

        local fileOut="${out}/${offset}_${package}_${memName}.${fileExt}"

        if [[ $metadataOffset != "" ]] && [[ $(echo "ibase=16;(${metadataOffset}-${offset})/dev/null

        if [[ $fileExt == "so" ]]; then

                lastFile=$fileOut

        else

                if [[ $lastFile != "" ]]; then

                        echo "- Merging memory..."

                        skipMerge=false

                        if [[ $lastEnd != $offset ]]; then

                                gap_block=$(echo "ibase=16;(${offset}-${lastEnd})/${HEX_PAGESIZE}" | bc)

                                if [[ $gap_block -gt 32 ]]; then

                                        echo "- Gap block(s) $gap_block is too large, skipping merge..."

                                        skipMerge=true

                                        lastFile=$fileOut

                                else

                                        echo "- Adding $gap_block gap block(s)..."

                                        dd if="/proc/$pid/mem" bs=$SYS_PAGESIZE skip=$(echo "ibase=16;${lastEnd}/$HEX_PAGESIZE" | bc) count=$gap_block of="$out/tmp" 2>/dev/null

                                        cat "$out/tmp">>"$lastFile"

                                fi

                        fi

                        if [[ $skipMerge == "false" ]]; then

                                cat "$fileOut">>"$lastFile"

                                rm -f "$fileOut"

                        fi

                else

                        echo "- No ELF header found, but nothing to merge. Treating as a raw dump..."

                        lastFile=$fileOut

                fi

        fi

        lastEnd=$end

        rm -f "${out}/tmp"

done

echo "- Done!"
```

再进行最后一次测试，no problem~

![](https://attach.52pojie.cn/forum/202310/15/133853tbsg1uzsoxfgzbf1.png)

image-54.png (859.31 KB, 下载次数: 4)

[href="forum.php?mod=attachment&aid=MjY0OTMwN3w0MjYxNmNiNnwxNzkzNTY5MTE5fDE2ODA4NnwxODQ0NTg3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0OTMwN3w0MjYxNmNiNnwxNzkzNTY5MTE5fDE2ODA4NnwxODQ0NTg3&nothumb=yes)

2023-10-15 13:38 上传

后话：

脚本只能辅助，却不能代替我们去分析代码。
