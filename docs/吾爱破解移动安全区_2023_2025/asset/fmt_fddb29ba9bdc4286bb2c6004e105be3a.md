
```
        }

    }

private:

    JNIEnv *env_;

    jclass classLoader_;

};

void zClassLoader::checkGlobalRef(JNIEnv *env, jclass clazz) {

    LOGD("checkGlobalRef called");

    auto VisitRoots = (void (*)(void *, void *)) zLinker::getInstance()->find_lib("libart.so").find_symbol("_ZN3art9JavaVMExt10VisitRootsEPNS_11RootVisitorE");

    if (VisitRoots == nullptr) {

        LOGE("Failed to find method 'VisitRoots' in JavaVMExt");

        return;

    }

    JavaVM *jvm;

    env->GetJavaVM(&jvm);

    ClassLoaderVisitor visitor(env, clazz);

    VisitRoots(jvm, &visitor);

    classLoaderStringList.insert(classLoaderStringList.end(), visitor.classLoaderStringList.begin(), visitor.classLoaderStringList.end());

    classNameList.insert(classNameList.end(), visitor.classNameList.begin(), visitor.classNameList.end());

}
```

## 技术难点解析

- `ART`内部API: 直接调用ART运行时内部API，绕过Java层限制
- 内存布局: 理解ART对象内存布局，正确解析类信息

## 4. Linker遍历技术实现

### 技术背景

Android的动态链接器(`linker`)管理所有加载的共享库，通过遍历`linker`内部数据结构可以检测异常库加载和Hook行为。

### 实现流程

#### 4.1 Linker内部结构解析

```
zLinker::zLinker() {
    LOGD("Constructor called");
    this->elf_file_ptr = parse_elf_file("/system/bin/linker64");
    LOGI("linker64 elf_file_ptr %p", this->elf_file_ptr);

    parse_elf_head();
    parse_program_header_table();
    parse_section_table();

    this->elf_mem_ptr = get_maps_base("linker64");
    LOGI("linker64 elf_mem_ptr %p", this->elf_mem_ptr);

    soinfo*(*solist_get_head)() = (soinfo*(*)())(this->find_symbol("__dl__Z15solist_get_headv"));

    LOGI("linker64 solist_get_head %p", solist_get_head);

    soinfo_head = solist_get_head();
    LOGI("soinfo_head %p", soinfo_head);

    soinfo_get_realpath =  (char*(*)(void*))(this->find_symbol("__dl__ZNK6soinfo12get_realpathEv"));
    LOGI("soinfo_get_realpath %p", soinfo_get_realpath);
}
```

#### 4.2 库遍历机制

```
vector zLinker::get_libpath_list(){

    LOGD("get_libpath_list called");

    vector libpath_list = vector();

    soinfo* soinfo = soinfo_head;

    while(soinfo->next != nullptr){

        char* real_path = soinfo_get_realpath(soinfo);

        libpath_list.push_back(real_path);

        soinfo = soinfo->next;

    }

    LOGI("get_libpath_list: found %zu libraries", libpath_list.size());

    return libpath_list;

}
```

#### 4.3 库完整性校验

```
bool zLinker::check_lib_crc(const char* so_name){
    LOGD("check_lib_crc called with so_name: %s", so_name);
    LOGI("check_lib_hash so_name: %s", so_name);
    zElf elf_lib_file = zLinker::getInstance()->find_lib(so_name);
    uint64_t elf_lib_file_crc = elf_lib_file.get_elf_header_crc() + elf_lib_file.get_program_header_crc() + elf_lib_file.get_text_segment_crc();

    zElf elf_lib_mem = zElf((void*)zLinker::get_maps_base(so_name));
    uint64_t elf_lib_mem_crc = elf_lib_mem.get_elf_header_crc() + elf_lib_mem.get_program_header_crc() + elf_lib_mem.get_text_segment_crc();

    LOGI("check_lib_hash elf_lib_file: %p crc: %lu", elf_lib_file.elf_file_ptr, elf_lib_file_crc);
    LOGI("check_lib_hash elf_lib_mem: %p crc: %lu", elf_lib_mem.elf_mem_ptr, elf_lib_mem_crc);

    bool crc_mismatch = elf_lib_file_crc != elf_lib_mem_crc;
    if (crc_mismatch) {
        LOGW("check_lib_crc: CRC mismatch detected for %s", so_name);
    } else {
        LOGI("check_lib_crc: CRC match for %s", so_name);
    }
    return crc_mismatch;
}
```

## 技术难点解析

- 内存布局: 理解`linker`内部数据结构布局
- 符号解析: 动态解析`linker`内部符号
- 完整性校验: 通过`CRC`检测内存篡改

## 5. SSL证书检测技术实现

### 技术背景

SSL证书检测是验证网络通信安全性的重要手段，本项目通过 `mbedtls` 库实现完整的TLS握手和证书验证流程。

### 实现流程

#### 5.1 自定义HTTPS客户端

```
HttpsResponse zHttps::performRequest(const HttpsRequest& request) {

    HttpsResponse response;

    // 使用请求的超时时间，如果没有设置则使用默认超时
    int timeout_seconds = request.timeout_seconds > 0 ? request.timeout_seconds : default_timeout_seconds;

    // 为本次请求创建独立的计时器和资源
    RequestTimer timer(timeout_seconds);

    RequestResources resources;

    LOGI("Starting HTTPS request with timeout: %d seconds", timer.timeout_seconds);

    // 安全检查：确保只使用HTTPS协议，但允许任意端口
    if (request.url.substr(0, 8) != "https://") {
        response.error_message = "Only HTTPS URLs are supported";
        LOGE("Security Error: Only HTTPS protocol is allowed");
        return response;
    }

    // 初始化mbedtls（如果需要）
    if (!initialized) {
        if (!initialize()) {
            response.error_message = "Failed to initialize mbedtls";
            LOGE("Failed to initialize mbedtls");
            return response;
        }
    }

    // 确保每次请求都使用干净的状态
    // 这可以防止前一次请求的失败状态影响当前请求
    LOGI("Starting new HTTPS request to %s", request.host.c_str());

    // 重置全局SSL状态，确保每次请求都是独立的
    if (initialized) {
        // 重新初始化全局资源以确保干净状态
        cleanup();
        if (!initialize()) {
            response.error_message = "Failed to reinitialize mbedtls";
            LOGE("Failed to reinitialize mbedtls");
            return response;
        }
    }

    // 记录本地设置的证书固定信息
    auto it = pinned_certificates.find(request.host);
    if (it != pinned_certificates.end()) {
        response.pinned_certificate = it->second;
    }

    char error_buf[0x1000];
    int ret;

    // 使用自定义socket连接
    LOGI("Connecting to %s:%d with custom socket...", request.host.c_str(), request.port);

    // 检查超时
    if (timer.isTimeout()) {
        response.error_message = "Connection timeout before establishing connection";
        LOGE("Connection timeout before establishing connection");
        return response;
    }

    // 记录连接开始时间
    time_t connect_start = time(nullptr);
    LOGI("Connection attempt started at: %ld", connect_start);

    // 使用自定义socket连接，带超时控制
    resources.sockfd = connectWithTimeout(request.host, request.port, timeout_seconds);

    // 记录连接结束时间
    time_t connect_end = time(nullptr);
    int connect_duration = connect_end - connect_start;
    LOGI("Connection attempt completed in %d seconds", connect_duration);

    if (resources.sockfd 
```

#### 5.2 证书指纹验证

```
bool zHttps::verifyCertificatePinning(const mbedtls_x509_crt* cert, const string& hostname) {
    LOGD("verifyCertificatePinning called for hostname: %s", hostname.c_str());
    auto it = pinned_certificates.find(hostname);
    if (it == pinned_certificates.end()) {
        LOGI("No pinned certificate for hostname: %s, skipping pinning check.", hostname.c_str());
        return true; // 没有固定证书时跳过验证
    }

    const CertificateInfo& pinned = it->second;
    bool serial_match = (pinned.serial_number == extractCertificateInfo(cert).serial_number);
    bool fingerprint_match = (pinned.fingerprint_sha256 == extractCertificateInfo(cert).fingerprint_sha256);

    // 只有当期望的subject不为空时才验证subject
    bool subject_match = true;
    if (!pinned.subject.empty()) {
        subject_match = (pinned.subject == extractCertificateInfo(cert).subject);
    }

    if (!serial_match || !fingerprint_match || !subject_match) {
        LOGW("Certificate pinning verification failed for %s", hostname.c_str());
        LOGD("serial_match %d fingerprint_match %d subject_match %d ", serial_match, fingerprint_match, subject_match);

        // 输出详细信息用于调试
        if (!serial_match) {
            LOGD("Expected serial: %s, Got: %s", 
                 pinned.serial_number.c_str(), 
                 extractCertificateInfo(cert).serial_number.c_str());
        }
        if (!fingerprint_match) {
            LOGD("Expected fingerprint: %s, Got: %s", 
                 pinned.fingerprint_sha256.c_str(), 
                 extractCertificateInfo(cert).fingerprint_sha256.c_str());
        }
        if (!subject_match && !pinned.subject.empty()) {
            LOGD("Expected subject: %s, Got: %s", 
                 pinned.subject.c_str(), 
                 extractCertificateInfo(cert).subject.c_str());
        }

        return false;
    }
    LOGI("Certificate pinning verification passed for %s", hostname.c_str());
    return true;
}
```

#### 5.3 mbedtls静态库编译

```
wget https://dl.google.com/android/repository/android-ndk-r25c-linux.zip

unzip android-ndk-r25c-linux.zip

export ANDROID_NDK=/home/lxz/android-ndk-r25c
```
