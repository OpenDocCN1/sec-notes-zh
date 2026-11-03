
# 代码解密与运行结果

## 解密代码

```python
import base64

# 第一步：Base64解码
b64_data = ("YHM2HhUMABFULBIWdFVQWUVZSkYRDxgsFlNXf28SLQQIWTtYQFBCUV9UTRAyAwoRBkUQPh8ADTZUUTg8NThuLjAPOyA"
            "xUlhUeD4OPxJjBllTWXVnWyE1Nh8uWTwmEQIBNQ0LWFFXf2QeNT0aBCg3MwwhGjkqIzIAUlQEYgUWYHMAOCA6UlhUeD"
            "0gPw9jBgxQWQBnWCElOhIuGDAceGZvJgxtZnx4f2ERUFkEVJX86OVUuvvKnNWR0Zuo3Iy6TTgDHUJPUkKEwP/1WbeWg"
            "92Iq9e0yJzw9YLD84rIxUxJWXjbqLAW0Zi9hdbele3zl+/rsNfkJTEeGBURyK2i6Fm6zvaG++iSyN2M7usDFBJrNTg8"
            "ZzYPLDU2IS1UYktVAW8IOT95aG19IjgbLDY3IEVJf1sdSWk0PnpmZ3FwITV/TkVTClVHUmEqKQBzeWVpcXRuIzx/TkV"
            "TClVAUmEqKQBxdXliGA8RXQFvRmhpf28QOg1FJjtcV0dPSEYZCRgrEklDGUxOUmFFWX8ZRlBCTUBfTRsmBwAQWgdUAU"
            "sOIjYZERVaXVwZBlACUwMMAEUdc0sHWTZXFFBYTV9UHxgrFk0HExEVdkJoc1IzUFBQGG1VBxttWxZKSGh+f0tFWTcZC"
            "RUDCwoAYHN/U0VDFAoGfwhFEDEZRw87MhIRTVl/U0VDGkVJf0NNEX8FCBUDERIaTRF2UztDEWh+f0tFWS1cQEBEVhJZ"
            "TV9/Qx0lNCMyGS0jP1IzOT9VVFNCHlkJPl9ueEVUf0sBHDkZa2pfVltFMiZ3AAAPFElUPAQBHHYDOT8WGBIRTVl/UxY"
            "GHgNaPAQBHH8EFFZZXFc8Z1l/U0VDUkVULA4JH3FJVxULGAI8Z1l/U0VDUkVULA4JH3FKQFRVUxIMTSICfm9DUkVUf0"
            "tFWSxcWFMYSldWTUR/KFVPUlUpUmFFWX8ZFBUWGEFUAR9xGwQNFgkRLEtYWQRkOT87MhIRTVk7FgNDABAadxgAFTkQD"
            "jg8GBIRTVl/U0UXABxOUmFFWX8ZFBUWGBIRTVkoGwwPF0UHOgcDVy9aFAkWVFdfRQo6HwNNEQoQOkJfdFUZFBUWGBIR"
            "TVl/U0VDUkVUMBtFRH9KUVlQFlFeCRwEAAAPFEsEPDZoc38ZFBUWGBIRTVl/U0VDUkUHOgcDVy9aFB4LGAM8Z1l/U0V"
            "DUkVUf0tFWX8ZFBVfXhJeHVliTkUsIjokCjgtQ1IzFBUWGBIRTVl/U0VDUkVUf0tFWX9QUhVFXV5XQwk8U05DRkVKfw"
            "cAF3dKUVlQFlFeCRx2SWhpUkVUf0tFWX8ZFBUWGBIRTVl/U0VDUkVUPRkAGDQ0PhUWGBIRTVl/U0VDUkVUf0tFWX8ZQ"
            "hULGFtfGVc5AQoOLQcNKw4WUSxcWFMYW11VCCIsFgkFXBUXZRgAFTkXRFYdDG8dTV4zGhEXHgBTdmZvWX8ZFBUWGBIR"
            "TVl/U0VDUkVUf0sWHDNfGkZCWVFaQxgvAwANFk0CdmZvWX8ZFBUWGBIRTVl/U0VDUkVUf0sWHDNfGkVVGBkMTU1SeUV"
            "DUkVUf0tFWX8ZFBUWGBJUARA5UwoTUlhJfyQ1JhN2dXFpa2ZjV3RVU0VDUkVUf0tFWX8ZFBUWGBIRTVk2FUUQFwkScR"
            "sGWWEEFFlTVhpCCBU5XQYMFgBdZWZvWX8ZFBUWGBIRTVl/U0VDUkVUf0tFWX8ZVkdTWVk8Z1l/U0VDUkVUf0tFWX8ZF"
            "BUWGBIRHhwzFUsRFwIvbzZFRH9KUVlQFlFeCRwEAAAPFEsEPDZoc38ZFBUWGBIRTVl/U0VDUkVUf0tFCjpVUhtGWxIa"
            "UFlufm9DUkVUf0tFWX8ZFBUWGBIRCBU2FUUMAkVJYksqKQB6dXl6Aj87TVl/U0VDUkVUf0tFWX8ZFBUWGBJYC1ksFgk"
            "FXBUXf1VYWTNcWh1FXV5XQxowFwBKSGh+f0tFWX8ZFBUWGBIRTVl/U0VDUkVUf0tFGy1cVV47MhIRTVl/U0VDUkVUf0"
            "tFWX8ZFBUWS1ERUFksFgkFXAYbOw4+CjpVUhtGW288Z1l/U0VDUkVUf0tFWX8ZFBUWGBIRHhwzFUsTEUVfYktUdFUZF"
            "BUWGBIRTVl/U0VDUkVUf0tFWTZfFEZVGA8MTUllfm9DUkVUf0tFWX8ZFBUWGBIRTVl/U0VDUkUELQILDXdmZ2FkcXx2"
            "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"
            "+d/cs48YFhwTRHRV")

decoded_b64 = base64.b64decode(b64_data)

# 第二步：XOR解密
key = b"my_secret_key_9456821"
decrypted_bytes = bytearray()
for i in range(len(decoded_b64)):
    decrypted_byte = decoded_b64[i] ^ key[i % len(key)]
    decrypted_bytes.append(decrypted_byte)

# 结果是原始的Python源代码
original_code = decrypted_bytes.decode('utf-8')
print(original_code)
```

## 运行结果

```python
import sys
import base64
from datetime import datetime

_CIPHER = 'UkFMZ2leaGV6XjEzM+YRNidLR2ldaGV/XDEwMTAiUEROZm9fa2Z7'
_KEY = 'VEFPZ29fa2V5XzIwMjUh'
_STRINGS = ['&#128640; 启动实例 A\n', '&#128272; 请输入密码：', '&#9989; 验证成功！\n', '&#128197; 当前时间: ']
OP_PUSH = 0x01
OP_LOAD_STR = 0x06
OP_CALL = 0x03
OP_JMP_IF_NE = 0x04
OP_HALT = 0x05

def _decrypt(data, k):
    return bytes(b ^ k[i % len(k)] for i, b in enumerate(data))

def _djb2(s):
    h = 5381
    for c in s:
        h = ((h  len(self.code):
                        break
                    v = int.from_bytes(self.code[self.pc:self.pc+4], 'little')
                    self.stack.append(v)
                    self.pc += 4
                elif op == OP_LOAD_STR:
                    if self.pc >= len(self.code):
                        break
                    self.reg[0] = self.code[self.pc]
                    self.pc += 1
                elif op == OP_CALL:
                    if self.pc >= len(self.code):
                        break
                    sc = self.code[self.pc]
                    self.pc += 1
                    if sc == 0:
                        print(_STRINGS[self.reg[0]], end='', flush=True)
                    elif sc == 1:
                        s = input().strip().encode()
                        self.handles.append(s)
                        self.reg[0] = len(self.handles) - 1
                    elif sc == 2:
                        code = self.stack.pop() if self.stack else 1
                        sys.exit(code)
                    elif sc == 3:
                        h = self.reg[0]
                        self.reg[1] = _djb2(self.handles[h]) if 0 = len(self.code):
                        break
                    target = self.code[self.pc]
                    self.pc += 1
                    actual = self.reg[1]
                    expected = self.stack.pop() if self.stack else 0
                    if actual != expected:
                        if 0
```
