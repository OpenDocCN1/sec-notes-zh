
## 四、源码简要分析

### 4.1. Mp42Hls加密流程

这里主要分析和加解密相关的，解复用之类的就不过多介绍。

- 首先是这里，设置了`key`以及随机的`iv`，然后设置了加密方式，目录支持不加密，整体`aes`加密以及`SAMPLE_AES`

```
Options.encryption_key_hex="000102030405060708090a0b0c0d0e0f";
AP4_ParseHex(Options.encryption_key_hex, Options.encryption_key, 16);
Options.encryption_iv_mode = ENCRYPTION_IV_MODE_RANDOM;
AP4_System_GenerateRandomBytes(Options.encryption_iv, 16);
Options.encryption_mode=ENCRYPTION_MODE_SAMPLE_AES;
```

不仅加密视频帧，也加密了音频帧，所以之前生成`ts`文件，你直接播放没有声音也没有画面，后续我们也主要修改这来实现自己的帧加密。

- 然后就是解复用那一套，获取对应的码流，看不懂不重要，直接看怎么写入文件

![](https://attach.52pojie.cn/forum/202401/19/222856c510zj5yz6yrh60j.png)

- 根据不同加密方式设置加密函数，本质上就是一个`aes-cbc`

![](https://attach.52pojie.cn/forum/202401/19/222901vx2mibmh77mimvhd.png)

![](https://attach.52pojie.cn/forum/202401/19/222907hg3qqb3vb3ckjlak.png)

- 选择不同轨道来分别加密，主要看看视频加密部分

![](https://attach.52pojie.cn/forum/202401/19/222917jmqbmy14ky5rbjyw.png)

- `nal`加密部分，只加密了`nal=1`或者`5`的部分数据，至于为啥不知道苹果这样规定的

![](https://attach.52pojie.cn/forum/202401/19/222924aybxz5b5w957o87t.png)

- 在后面就是写入文件了，先判断是`h264`还是`h265`，然后写入了`sps`/`pps`

![](https://attach.52pojie.cn/forum/202401/19/222931ai1t5u7b7id1z5kt.png)

- 写入了加密后的`nalu`

- 组装`pes`

```
WritePES(pes_data.GetData(), pes_data.GetDataSize(), dts, true, pts, with_pcr, output)
```

- 生成`pes`头

```
unsigned int pes_header_size = 14+(with_dts?5:0);
AP4_BitWriter pes_header(pes_header_size);
.....................................................
pes_header.Write(1, 1);                    // market_bit
```

- 切分`nal`按`188`字节组装`ts`，然后就写入文件

![](https://attach.52pojie.cn/forum/202401/19/222935gncc4caqin34q4b6.png)

- 一直循环上面操作，完成后就是生成`m3u8`文件，后面不重要了不分析了。

### 4.2. hls解密流程

- 请求加载文件，一次加载`m3u8`文件，`key`文件，`ts`文件

![](https://attach.52pojie.cn/forum/202401/19/222937j0n1mmnpzr0n0yr0.png)

其中`key`加载部分，`loadKeyHTTP`函数，请求成功后回调函数中把，`key`赋值给了`keyInfo.decryptdata.key = frag.decryptdata.key`，

如果`key`有加密，大部分都在这里解密了，好处是只需要解密一次`key`，不需要额外修改函数逻辑，坏处就是一眼就看出来了

```
loadKeyHTTP = function loadKeyHTTP(keyInfo, frag) {
..................
return keyInfo.keyLoadPromise = new Promise(function (resolve, reject) {
  var loaderCallbacks = {
    onSuccess: function onSuccess(response, stats, context, networkDetails) {
......
keyInfo.decryptdata.key = frag.decryptdata.key = new Uint8Array(response.data);
frag.keyLoader = null;
keyInfo.loader = null;
resolve({
  frag: frag,
  keyInfo: keyInfo
});
}
.....
```

- 然后是`ts`回调函数，完全看不懂，唯一看起来眼熟的`frag`，`key`就存在这里面，然后就进入了`transmuxer`

```
function _handleFragmentLoadProgress(data) {
var _frag$initSegment;
var frag = data.frag,
.........
transmuxer.push(payload, initSegmentData, audioCodec, videoCodec, frag, part, details.totalduration, accurateTimeOffset, chunkMeta, initPTS);
};
```

- 后面又是一大堆看不懂的，把`key`赋值给了`decryptdata`，然后判断是否开启了`worker`，就开始解复用了

```
unction push(data, initSegmentData, audioCodec, videoCodec, frag, part, duration, accurateTimeOffset, chunkMeta, defaultInitPTS) {
................
  var decryptdata = frag.decryptdata;
  this.frag = frag;
  this.part = part;
  if (this.workerContext) {
    // post fragment payload as transferable objects for ArrayBuffer (no copy)
    this.workerContext.worker.postMessage({
      cmd: 'demux',
      data: data,
      decryptdata: decryptdata,
      chunkMeta: chunkMeta,
      state: state
    }, data instanceof ArrayBuffer ? [data] : []);
  } else if (transmuxer) {
    var _transmuxResult = transmuxer.push(data, decryptdata, chunkMeta, state);
    if (isPromise(_transmuxResult)) {
      transmuxer.async = true;
      _transmuxResult.then(function (data) {
        _this2.handleTransmuxComplete(data);
      }).catch(function (error) {
        _this2.transmuxerError(error, chunkMeta, 'transmuxer-interface push error');
      });
    } else {
      transmuxer.async = false;
      this.handleTransmuxComplete(_transmuxResult);
    }
  }
```

- 解复用之前，首先判断是否是`ts`整体加密

```
var keyData = getEncryptionType(uintData, decryptdata);
if (keyData && keyData.method === 'AES-128') {
  var decrypter = this.getDecrypter();
  // Software decryption is synchronous; webCrypto is not
  if (decrypter.isSync()) {
    var decryptedData = decrypter.softwareDecrypt(uintData, keyData.key.buffer, keyData.iv.buffer);
    uintData = new Uint8Array(decryptedData);
```

- 然后判断是否是`SAMPLE`，进入不同解析流程，解复完成后再进行解密内容
