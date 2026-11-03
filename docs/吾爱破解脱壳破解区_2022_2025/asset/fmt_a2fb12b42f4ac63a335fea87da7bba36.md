
```python
import base64, gzip, hashlib, binascii
from typing import Optional, List, Tuple

# --------------- CONFIG ---------------
RESPONSE = (
    "2abd2b6d922769cceu1uNDRjMzYyYXr4Ly2dfisoK359UYTmBysDfrBUAwJQtCsGBivnAwJSh4dSVXwGKNBSnTFjijiPchszZmE=bb57d9b8561ce2a5"

)
KEYS_FILE = "keys.txt"
PASSWORD = "pass"
MIN_RATIO = 0.7
# --------------- helpers ---------------
def printable_ratio(b: bytes) -> float:
    ifnot b:
        return0.0
    good = sum(1for x in b if32  Optional[bytes]:
    if isinstance(s, str):
        s = s.encode("latin1")
    try:
        return base64.b64decode(s, validate=False)
    except Exception:
        try:
            padded = s + b"=" * ((4 - (len(s) % 4)) % 4)
            return base64.b64decode(padded, validate=False)
        except Exception:
            returnNone

def try_gzip_decompress(b: bytes) -> Optional[bytes]:
    try:
        if isinstance(b, (bytes, bytearray)) and b.startswith(b'\x1f\x8b'):
            return gzip.decompress(b)
    except Exception:
        pass
    returnNone

def xor_with_key_bytes(data: bytes, key_bytes: bytes, offset: int = 0) -> bytes:
    ifnot key_bytes:
        return data
    out = bytearray(len(data))
    k = len(key_bytes)
    for i in range(len(data)):
        out[i] = data[i] ^ key_bytes[(i + offset) % k]
    return bytes(out)

def to_preview(b: bytes, n: int = 200) -> str:
    try:
        return b.decode("utf-8", errors="replace")[:n]
    except Exception:
        return binascii.hexlify(b[:n]).decode()

# --------------- key variants ---------------
def generate_key_variants(orig_key: str, password: str = "pass"):
    variants = []
    variants.append(("raw", orig_key))
    try:
        mdk = hashlib.md5(orig_key.encode()).hexdigest()[:16]
        variants.append(("md5_key16", mdk))
    except Exception:
        pass
    try:
        md2 = hashlib.md5((password + orig_key).encode()).hexdigest()[:16]
        variants.append(("md5_pass_key16", md2))
    except Exception:
        pass
    return variants

# --------------- attempt functions ---------------
def attempt_eval_b64_variant(response: str, variant_val, offsets: List[int]):
    out = []
    middle = response
    if isinstance(response, str) and len(response) > 48:
        med = response[16:-16]
        if any(c in med for c in"=+/"):
            middle = med

    b = try_base64_decode(middle)
    if b isNone:
        return out

    if isinstance(variant_val, (bytes, bytearray)):
        kb = variant_val
    else:
        kb = str(variant_val).encode("latin1")

    klen = len(kb) or1
    max_off = min(klen, 32)
    for off in offsets:
        dec = xor_with_key_bytes(b, kb, offset=off)
        pr = printable_ratio(dec)
        if pr >= MIN_RATIO: 
            out.append((f"EVAL_B64 off={off}", dec, pr))
        gz = try_gzip_decompress(dec)
        if gz isnotNoneand printable_ratio(gz) >= MIN_RATIO: 
            out.append((f"EVAL_B64->GZIP off={off}", gz, printable_ratio(gz)))
    return out

def attempt_raw_variants(response: str, variant_val, offsets: List[int]):
    out = []
    s = response.strip()

    bfull = try_base64_decode(s)
    if bfull isnotNone:
        if isinstance(variant_val, (bytes, bytearray)):
            kb = variant_val
        else:
            kb = str(variant_val).encode("latin1")
        klen = len(kb) or1
        max_off = min(klen, 32)
        for off in offsets:
            dec = xor_with_key_bytes(bfull, kb, offset=off)
            pr = printable_ratio(dec)
            if pr >= MIN_RATIO:  
                out.append((f"RAW_B64FULL off={off}", dec, pr))
            gz = try_gzip_decompress(dec)
            if gz isnotNoneand printable_ratio(gz) >= MIN_RATIO:  
                out.append((f"RAW_B64FULL->GZIP off={off}", gz, printable_ratio(gz)))
    return out

# --------------- driver ---------------
def brute_force_response(response: str, keys_file: str, password: str = "pass"):
    try:
        with open(keys_file, "r", encoding="utf-8") as f:
            keys = [line.strip() for line in f if line.strip()]
    except FileNotFoundError:
        print(f"keys file '{keys_file}' not found.")
        return

    offsets_try = list(range(0, 8))
    for idx, orig in enumerate(keys, 1):
        print("\n" + "="*60)
        print(f"[{idx}/{len(keys)}] testing key: {orig!r}")
        variants = generate_key_variants(orig, password=password)
        for vname, kval in variants:
            eval_candidates = attempt_eval_b64_variant(response, kval, offsets_try)
            for label, dec, pr in eval_candidates:
                print(f"  {label:25} pr={pr:.3f} preview={to_preview(dec)}")
            raw_candidates = attempt_raw_variants(response, kval, offsets_try)
            for label, dec, pr in raw_candidates:
                print(f"  {label:25} pr={pr:.3f} preview={to_preview(dec)}")

# --------------- run ---------------
if __name__ == "__main__":
    print("Starting brute-force run (MIN_RATIO =", MIN_RATIO, ")")
    brute_force_response(RESPONSE, KEYS_FILE, password=PASSWORD)
flag{ccebdb78-4b5c-4252-b20a-0039913c5c94}
```
