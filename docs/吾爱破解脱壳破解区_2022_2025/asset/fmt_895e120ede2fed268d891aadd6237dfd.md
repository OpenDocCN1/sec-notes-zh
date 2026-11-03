
# [原创] JSC字节码反编译初探——以Typora 1.10.8为例

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=2054765
- 抓取时间: 2026-11-01T22:05:19

在之前的 [https://www.52pojie.cn/thread-2040749-1-1.html](https://www.52pojie.cn/thread-2040749-1-1.html) 中，我们已经尝试了通过 hook node api 的方式替换公钥，这次我们来尝试一下从那个已经被编译为字节码的 `jsc` 入手。

在网上搜索 `jsc` 反编译相关的内容，找到一篇相关 [https://guage.cool/wiz-license.html](https://guage.cool/wiz-license.html)，作者通过修改 `d8`，添加 `Disassemble`、`LoadJSC` 函数，从而实现解析 `jsc`，我们也依照此思路进行分析。

## 制作反编译器

在安装目录下的 `version` 文件中可以看到 electron 版本为 32.1.2，在网上搜索可知对应的 v8 版本为 12.8.374.33，我们先在本地搭建 v8 编译环境，并 `git checkout 12.8.374.33`。

由于自 12 版本开始，v8 引擎做了比较大的 api 变动，作者原先教程中的修改代码不再适用，下面是我在此基础上做的修正：

在 `src/d8/d8.cpp` 中添加下面两个方法：

```
static void Disassemble(v8::internal::Isolate* isolate, 
                        v8::internal::Tagged bytecode, 
                        std::unordered_set& visited,
                        int depth) {
  if (depth > 100) { 
    v8::internal::PrintF("Recursion depth limit reached, aborting disassembly for this path.\n");
    fflush(stdout);
    return;
  }

  uintptr_t key = reinterpret_cast(bytecode.ptr());
  if (visited.count(key)) {
    return;
  }
  visited.insert(key);
  for (int i = 0; i (bytecode.ptr()));
  fflush(stdout);
  v8::internal::OFStream os(stdout);
  bytecode->Disassemble(os);

  auto consts = bytecode->constant_pool();

  for (int i = 0; i length());
  fflush(stdout);

  for (int i = 0; i length(); i++) {
    auto obj = consts->get(i);
    if (v8::internal::IsSharedFunctionInfo(obj)) {
      auto shared = v8::internal::Cast(obj);

      for (int i = 0; i  Found SFI in constant pool at index %d: ", i);

      auto function_name = shared->Name();
      if (function_name->length() > 0) {
          v8::internal::PrintF("%s\n", function_name->ToCString().get());
      } else {
          v8::internal::PrintF("(anonymous)\n");
      }
      fflush(stdout);

      if (shared->HasBytecodeArray()) {
          Disassemble(isolate, shared->GetBytecodeArray(isolate), visited, depth + 1);
      } else {
          for (int i = 0; i & args) {
  auto isolate = reinterpret_cast(args.GetIsolate());
  for (int i = 0; i ThrowException(v8::Exception::Error(
          v8::String::NewFromUtf8(args.GetIsolate(), "Error loading file").ToLocalChecked()));
      return;
    }
    int length = 0;
    auto filedata = reinterpret_cast(ReadChars(*filename, &length));
    if (filedata == NULL) {
      args.GetIsolate()->ThrowException(v8::Exception::Error(
          v8::String::NewFromUtf8(args.GetIsolate(), "Error reading file").ToLocalChecked()));
      return;
    }
    v8::internal::AlignedCachedData cached_data(filedata, length);
    auto source = isolate->factory()
                      ->NewStringFromUtf8(base::CStrVector("source"))
                      .ToHandleChecked();
    v8::internal::ScriptDetails script_details;
    v8::internal::MaybeHandle maybe_fun =
        v8::internal::CodeSerializer::Deserialize(isolate, &cached_data, source, script_details);

    v8::internal::Handle fun;
    if (!maybe_fun.ToHandle(&fun)) {
      args.GetIsolate()->ThrowException(v8::Exception::Error(
          v8::String::NewFromUtf8(args.GetIsolate(), "Deserialize failed, possibly version mismatch or invalid .jsc file").ToLocalChecked()));
      delete[] filedata;
      return;
    }

    v8::internal::PrintF("---- Starting disassembly of %s ----\n", *filename);
    fflush(stdout);

    std::unordered_set visited;
    Disassemble(isolate, fun->GetBytecodeArray(isolate), visited, 0); 

    v8::internal::PrintF("---- Finished disassembly of %s ----\n", *filename);
    fflush(stdout);

    delete[] filedata;
  }
}
```

并在 `Shell::CreateGlobalTemplate` 中添加代码：

```
global_template->Set(
    v8::String::NewFromUtf8(isolate, "loadjsc", v8::NewStringType::kNormal)
        .ToLocalChecked(),
    v8::FunctionTemplate::New(isolate, v8::Shell::LoadJSC));
```

在 `src/d8/d8.h` 的 `class Shell` 中添加 `LoadJSC` 声明：

```
static void LoadJSC(const v8::FunctionCallbackInfo& args);
```

`src/diagnostics/objects-printer.cc`：

注释掉：`PrintSourceCode(os);`

在

```
  os 
```

后添加

```
  os GetActiveBytecodeArray(isolate)->Disassemble(os);
  os 
```

在

```
void HeapObject::HeapObjectShortPrint(std::ostream& os) {

  PtrComprCageBase cage_base = GetPtrComprCageBase();
```

后添加

```
  Isolate* isolate = nullptr;
  if (!GetIsolateFromHeapObject(*this, &isolate) || isolate == nullptr) {
    os (this->ptr())  map_of_this_object = this->map(cage_base);
  if (map_of_this_object.ptr() == kNullAddress) {
    os (this->ptr()) map(cage_base) != roots.meta_map()) {
    os (this->ptr()) 
```

在

```
    os 
```

后添加

```
  if (map(cage_base)->instance_type() == ASM_WASM_DATA_TYPE) {
    os  ";
    Cast(*this)
        ->constant_elements()
        ->HeapObjectShortPrint(os);
    return;
  }
```

在

```
    case FIXED_ARRAY_TYPE:

      os (*this)->length() ";
```

后添加

```
      os (*this)->FixedArrayPrint(os);
      os 
```

在

```
    case OBJECT_BOILERPLATE_DESCRIPTION_TYPE:

      os (*this)->capacity() ";
```

后添加

```
      os (*this)
          ->ObjectBoilerplateDescriptionPrint(os);
      os 
```

在

```
    case FIXED_DOUBLE_ARRAY_TYPE:

      os (*this)->length()

         ";
```

后添加

```
      os (*this)->FixedDoubleArrayPrint(os);
      os 
```

在

```
      } else {

        os ";

      }
```

后添加

```
      os SharedFunctionInfoPrint(os);
      os 
```

`src/snapshot/code-serializer.cc`：

替换 `SanityCheck`、`SanityCheckWithoutSource` 函数：

```
SerializedCodeSanityCheckResult SerializedCodeData::SanityCheck(
    uint32_t expected_ro_snapshot_checksum,
    uint32_t expected_source_hash) const {
  return SerializedCodeSanityCheckResult::kSuccess;
}

SerializedCodeSanityCheckResult SerializedCodeData::SanityCheckWithoutSource(
    uint32_t expected_ro_snapshot_checksum) const {
  // Always return kSuccess to bypass all checks.
  return SerializedCodeSanityCheckResult::kSuccess;
}
```

`src/snapshot/deserializer.cc`：

替换 `ReadReadOnlyHeapRef` 函数：

```
int Deserializer::ReadReadOnlyHeapRef(uint8_t data,
                                                SlotAccessor slot_accessor) {
  uint32_t chunk_index = source_.GetUint30();
  uint32_t chunk_offset = source_.GetUint30();

  ReadOnlySpace* read_only_space = isolate()->heap()->read_only_space();

  if (chunk_index >= read_only_space->pages().size()) {
    Tagged the_hole = *isolate()->factory()->the_hole_value();

    return WriteHeapPointer(slot_accessor, the_hole,
                            GetAndResetNextReferenceDescriptor());
  }

  ReadOnlyPageMetadata* page = read_only_space->pages()[chunk_index];
  Address address = page->OffsetToAddress(chunk_offset);
  Tagged heap_object = HeapObject::FromAddress(address);

  return WriteHeapPointer(slot_accessor, heap_object,
                          GetAndResetNextReferenceDescriptor());
}
```

以上为全部修改，之后使用 `ninja -C out.gn/x64.release d8` 编译。

编译好后运行 `./out.gn/x64.release/d8 -e "loadjsc('atom.compiled.dist.jsc')" > atom.txt` 即可得到反编译后的结果：

[https://wwri.lanzouo.com/iB9Ea34181jc](https://wwri.lanzouo.com/iB9Ea34181jc)

## 分析 `atom.txt`

面对海量的字节码，我们直奔主题，寻找 rsa 公钥，在之前版本的 `atom.js` 中，我们可以得知公钥是 base64 解析出来的：

```
T = JSON.parse(Buffer.from("WyItLS0tLUJFR0lOIFBVQkxJQyBLRVktLS0tLSIsIk1JSUJJakFOQmdrcWhraUc5dzBCQVFFRkFBT0NBUThBTUlJQkNnS0NBUUVBN25Wb0dDSHFJTUp5cWdBTEVVcmMiLCI1SkpoYXAwK0h0SnF6UEUwNHB6NHkrbnJPbVk3LzEyZjNIdlp5eW9Sc3hLZFhUWmJPMHdFSEZJaDBjUnFzdWFKIiwiUHlhT09QYkEwQnNhbG9mSUFZM21SaFFRM3ZTZitybjNnK3cwUyt1ZFdtS1Y5RG5tSmxwV3FpekZhalU0VC9FNCIsIjVaZ01OY1h0M0UxaXBzMzJyZGJUUjBObmVuOVBWSVR2cmJKM2w2Q0kyQkZCSW1aUVoyUDhOK0xzcWZKc3F5VlYiLCJ3RGt0M21IQVZ4VjdGWmJmWVdHKzhGRFN1S1FIYUNtdmdBdENoeDlod2wzSjZSZWtrcURWYTZHSVYxM0QyM0xTIiwicWRrMEpiNTIxd0ZKaS9WNlFBSzZTTEJpYnk1Z1lONnpRUTVSUXBqWHRSNTNNd3pUZGlBekdFdUtkT3RyWTJNZSIsIkR3SURBUUFCIiwiLS0tLS1FTkQgUFVCTElDIEtFWS0tLS0tIiwiIiwiIl0=","base64").toString("utf8")).join("\n"),

I = 864e5;
```
