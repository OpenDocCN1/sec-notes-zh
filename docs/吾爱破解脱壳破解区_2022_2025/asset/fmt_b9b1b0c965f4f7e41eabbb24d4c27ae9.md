
# [原创] Typora 1.10.8公钥替换

- 原帖: <https://www.52pojie.cn/forum.php?mod=viewthread&tid=2040749>
- 抓取时间: 2026-11-01T22:07:51

## Typora 1.10.8公钥替换

在最新版本中的Typora中，解包`app.asar`会发现软件使用了`node vm`将`js`编译成了`jsc`，在之前的版本中，分析`atom.js`可以得知Typora的激活实际上就是一个简单RSA公钥解密，只要patch了公钥就可以编写注册机进行离线激活，然而`jsc`中并没有简单的将公钥作为字符串进行储存（猜测是使用数组进行了解密），而分析`jsc`机器码又十分困难（需要自行编译定制版v8），因此可以通过native层进行公钥替换，这样不管开发者如何在`js`层上进行防护，都无法封堵该方法（除非定制`node`）

我们知道electron应用实际上是对`node`进行了打包，因此庞大的主程序里面会有`node`提供的所有函数，进行RSA解密需要用到`Crypto.publicDecrypt`函数，而`Crypto`模块是`node`使用C++编写的。

我们来到 [nodejs/node](https://github.com/nodejs/node) 的代码仓库搜索`publicDecrypt`，可以找到接口定义（`node/src/crypto/crypto_cipher.cc`）：

```
  SetMethod(context,
            target,
            "publicDecrypt",
            PublicKeyCipher::Cipher);
```

向下翻，找到`PublicKeyCipher::Cipher`函数：

```
template 

void PublicKeyCipher::Cipher(const FunctionCallbackInfo& args) {

  MarkPopErrorOnReturn mark_pop_error_on_return;

  Environment* env = Environment::GetCurrent(args);

  unsigned int offset = 0;

  auto data = KeyObjectData::GetPublicOrPrivateKeyFromJs(args, &offset);

  if (!data) return;

  const auto& pkey = data.GetAsymmetricKey();

  if (!pkey) return;

  ArrayBufferOrViewContents buf(args[offset]);

  if (!buf.CheckSizeInt32()) [[unlikely]] {

    return THROW_ERR_OUT_OF_RANGE(env, "buffer is too long");

  }

  uint32_t padding;

  if (!args[offset + 1]->Uint32Value(env->context()).To(&padding)) return;

  if (cipher == ncrypto::Cipher::decrypt &&

      operation == PublicKeyCipher::kPrivate && padding == RSA_PKCS1_PADDING) {

    EVPKeyCtxPointer ctx = pkey.newCtx();

    CHECK(ctx);

    if (!ctx.initForDecrypt()) {

      return ThrowCryptoError(env, ERR_get_error());

    }

    // RSA implicit rejection here is not supported by BoringSSL.

    if (!ctx.setRsaImplicitRejection()) [[unlikely]] {

      return THROW_ERR_INVALID_ARG_VALUE(

          env,

          "RSA_PKCS1_PADDING is no longer supported for private decryption");

    }

  }

  Digest digest;

  if (args[offset + 2]->IsString()) {

    Utf8Value oaep_str(env->isolate(), args[offset + 2]);

    digest = Digest::FromName(*oaep_str);

    if (!digest) return THROW_ERR_OSSL_EVP_INVALID_DIGEST(env);

  }

  ArrayBufferOrViewContents oaep_label(

      !args[offset + 3]->IsUndefined() ? args[offset + 3] : Local());

  if (!oaep_label.CheckSizeInt32()) [[unlikely]] {

    return THROW_ERR_OUT_OF_RANGE(env, "oaepLabel is too big");

  }

  std::unique_ptr out;

  if (!Cipher(env, pkey, padding, digest, oaep_label, buf, &out)) {

    return ThrowCryptoError(env, ERR_get_error());

  }

  Local ab = ArrayBuffer::New(env->isolate(), std::move(out));

  args.GetReturnValue().Set(

      Buffer::New(env, ab, 0, ab->ByteLength()).FromMaybe(Local()));

}
```

寻找关键函数`KeyObjectData::GetPublicOrPrivateKeyFromJs`定义（`node/src/crypto/crypto_keys.cc`）：

```
KeyObjectData KeyObjectData::GetPublicOrPrivateKeyFromJs(
    const FunctionCallbackInfo& args, unsigned int* offset) {
  if (IsAnyBufferSource(args[*offset])) {
    Environment* env = Environment::GetCurrent(args);
    ArrayBufferOrViewContents data(args[(*offset)++]);
    if (!data.CheckSizeInt32()) [[unlikely]] {
      THROW_ERR_OUT_OF_RANGE(env, "keyData is too big");
      return {};
    }

    EVPKeyPointer::PrivateKeyEncodingConfig config;
    if (!KeyObjectData::GetPrivateKeyEncodingFromJs(
             args, offset, kKeyContextInput)
             .To(&config)) {
      return {};
    }

    ncrypto::Buffer buffer = {
        .data = reinterpret_cast(data.data()),
        .len = data.size(),
    };

    if (config.format == EVPKeyPointer::PKFormatType::PEM) {
      // For PEM, we can easily determine whether it is a public or private key
      // by looking for the respective PEM tags.
      auto res = EVPKeyPointer::TryParsePublicKeyPEM(buffer);
      if (res) {
        return CreateAsymmetric(kKeyTypePublic, std::move(res.value));
      }

      if (res.error.value() == EVPKeyPointer::PKParseError::NOT_RECOGNIZED) {
        return TryParsePrivateKey(env, config, buffer);
      }
      ThrowCryptoError(
          env, res.openssl_error.value_or(0), "Failed to read asymmetric key");
      return {};
    }

    // For DER, the type determines how to parse it. SPKI, PKCS#8 and SEC1 are
    // easy, but PKCS#1 can be a public key or a private key.
    static const auto is_public = [](const auto& config,
                                     const auto& buffer) -> bool {
      switch (config.type) {
        case EVPKeyPointer::PKEncodingType::PKCS1:
          return !EVPKeyPointer::IsRSAPrivateKey(buffer);
        case EVPKeyPointer::PKEncodingType::SPKI:
          return true;
        case EVPKeyPointer::PKEncodingType::PKCS8:
          return false;
        case EVPKeyPointer::PKEncodingType::SEC1:
          return false;
        default:
          UNREACHABLE("Invalid key encoding type");
      }
    };

    if (is_public(config, buffer)) {
      auto res = EVPKeyPointer::TryParsePublicKey(config, buffer);
      if (res) {
        return CreateAsymmetric(KeyType::kKeyTypePublic, std::move(res.value));
      }

      ThrowCryptoError(
          env, res.openssl_error.value_or(0), "Failed to read asymmetric key");
      return {};
    }

    return TryParsePrivateKey(env, config, buffer);
  }

  CHECK(args[*offset]->IsObject());
  KeyObjectHandle* key =
      BaseObject::Unwrap(args[*offset].As());
  CHECK_NOT_NULL(key);
  CHECK_NE(key->Data().GetKeyType(), kKeyTypeSecret);
  (*offset) += 4;
  return key->Data().addRef();
}
```

在之前的版本中，我们可以得知`js`层传入的是`PEM`格式的公钥，因此寻找`EVPKeyPointer::TryParsePublicKeyPEM`函数定义（`node/deps/ncrypto/ncrypto.cc`）：

```
EVPKeyPointer::ParseKeyResult EVPKeyPointer::TryParsePublicKeyPEM(
    const Buffer& buffer) {
  auto bp = BIOPointer::New(buffer.data, buffer.len);
  if (!bp) return ParseKeyResult(PKParseError::FAILED);

  // Try parsing as SubjectPublicKeyInfo (SPKI) first.
  if (auto ret = TryParsePublicKeyInner(
          bp,
          "PUBLIC KEY",
          [](const unsigned char** p, long l) {  // NOLINT(runtime/int)
            return d2i_PUBKEY(nullptr, p, l);
          })) {
    return ret;
  }

  // Maybe it is PKCS#1.
  if (auto ret = TryParsePublicKeyInner(
          bp,
          "RSA PUBLIC KEY",
          [](const unsigned char** p, long l) {  // NOLINT(runtime/int)
            return d2i_PublicKey(EVP_PKEY_RSA, nullptr, p, l);
          })) {
    return ret;
  }

  // X.509 fallback.
  if (auto ret = TryParsePublicKeyInner(
          bp,
          "CERTIFICATE",
          [](const unsigned char** p, long l) {  // NOLINT(runtime/int)
            X509Pointer x509(d2i_X509(nullptr, p, l));
            return x509 ? X509_get_pubkey(x509.get()) : nullptr;
          })) {
    return ret;
  };

  return ParseKeyResult(PKParseError::NOT_RECOGNIZED);
}
```

在这里可以看到字符串形式的`key`被传入到了这个函数中（`buffer`），因此使用ida打开`Typora.exe`，搜索字符串`RSA PUBLIC KEY`，可以定位到这个函数：
