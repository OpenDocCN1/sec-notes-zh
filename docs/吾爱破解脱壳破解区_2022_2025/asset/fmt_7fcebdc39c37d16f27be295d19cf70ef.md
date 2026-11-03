
```
// 标准化屏幕分辨率
const fakeScreen = {
    width: 1920,
    height: 1080,
    availWidth: 1920,
    availHeight: 1040,
    colorDepth: 24,
    pixelDepth: 24,
    orientation: {
        angle: 0,
        type: 'landscape-primary'
    }
};

Object.keys(fakeScreen).forEach(key => {
    if (key === 'orientation') {
        Object.defineProperty(screen, key, {
            value: fakeScreen[key],
            writable: false
        });
    } else {
        Object.defineProperty(screen, key, {
            value: fakeScreen[key],
            writable: false,
            enumerable: true,
            configurable: false
        });
    }
});

// 设备像素比标准化
Object.defineProperty(window, 'devicePixelRatio', {
    value: 1,
    writable: false
});
```
### 3. 时区和语言统一
```
// 重写时区检测

const originalDateTimeFormat = Intl.DateTimeFormat;

Intl.DateTimeFormat = function(...args) {

    const instance = new originalDateTimeFormat(...args);

    const originalResolvedOptions = instance.resolvedOptions;

    instance.resolvedOptions = function() {

        const options = originalResolvedOptions.call(this);

        options.timeZone = 'America/New_York';

        options.locale = 'en-US';

        return options;

    };

    return instance;

};

// 重写时区偏移

const originalGetTimezoneOffset = Date.prototype.getTimezoneOffset;

Date.prototype.getTimezoneOffset = function() {

    return 300; // UTC-5 (Eastern Time)

};
```
## Canvas 指纹绕过
### 4. Canvas 2D 指纹干扰
```
// 方法1: 添加随机噪声
const originalToDataURL = HTMLCanvasElement.prototype.toDataURL;
HTMLCanvasElement.prototype.toDataURL = function(type, quality) {
    if (!this._fingerprinted) {
        const ctx = this.getContext('2d');
        const imageData = ctx.getImageData(0, 0, this.width, this.height);

        // 添加轻微的随机噪声
        for (let i = 0; i 
```
### 5. WebGL 指纹伪装
```
// Hook WebGL getParameter

const originalGetParameter = WebGLRenderingContext.prototype.getParameter;

WebGLRenderingContext.prototype.getParameter = function(parameter) {

    // 常见的WebGL参数伪装

    const fakeValues = {

        [this.VENDOR]: 'Intel Inc.',

        [this.RENDERER]: 'Intel(R) HD Graphics 4000',

        [this.VERSION]: 'WebGL 1.0 (OpenGL ES 2.0 Chromium)',

        [this.SHADING_LANGUAGE_VERSION]: 'WebGL GLSL ES 1.0 (OpenGL ES GLSL ES 1.0 Chromium)',

        [this.MAX_TEXTURE_SIZE]: 16384,

        [this.MAX_VIEWPORT_DIMS]: new Int32Array([16384, 16384]),

        [this.MAX_VERTEX_ATTRIBS]: 16,

        [this.MAX_VARYING_VECTORS]: 15,

        [this.ALIASED_LINE_WIDTH_RANGE]: new Float32Array([1, 1]),

        [this.ALIASED_POINT_SIZE_RANGE]: new Float32Array([1, 1024])

    };

    if (parameter in fakeValues) {

        return fakeValues[parameter];

    }

    return originalGetParameter.apply(this, arguments);

};

// Hook WebGL扩展

const originalGetExtension = WebGLRenderingContext.prototype.getExtension;

WebGLRenderingContext.prototype.getExtension = function(name) {

    // 返回标准化的扩展列表

    const standardExtensions = [

        'ANGLE_instanced_arrays',

        'EXT_blend_minmax',

        'EXT_color_buffer_half_float',

        'EXT_disjoint_timer_query',

        'EXT_frag_depth',

        'EXT_shader_texture_lod',

        'EXT_texture_filter_anisotropic',

        'WEBKIT_EXT_texture_filter_anisotropic',

        'EXT_sRGB',

        'OES_element_index_uint',

        'OES_standard_derivatives',

        'OES_texture_float',

        'OES_texture_float_linear',

        'OES_texture_half_float',

        'OES_texture_half_float_linear',

        'OES_vertex_array_object',

        'WEBGL_color_buffer_float',

        'WEBGL_compressed_texture_s3tc',

        'WEBKIT_WEBGL_compressed_texture_s3tc',

        'WEBGL_debug_renderer_info',

        'WEBGL_debug_shaders',

        'WEBGL_depth_texture',

        'WEBKIT_WEBGL_depth_texture',

        'WEBGL_draw_buffers',

        'WEBGL_lose_context',

        'WEBKIT_WEBGL_lose_context'

    ];

    if (standardExtensions.includes(name)) {

        return originalGetExtension.apply(this, arguments);

    }

    return null;

};

// Hook getSupportedExtensions

const originalGetSupportedExtensions = WebGLRenderingContext.prototype.getSupportedExtensions;

WebGLRenderingContext.prototype.getSupportedExtensions = function() {

    return [

        'ANGLE_instanced_arrays',

        'EXT_blend_minmax',

        'EXT_color_buffer_half_float',

        'EXT_disjoint_timer_query',

        'EXT_frag_depth',

        'EXT_shader_texture_lod',

        'EXT_texture_filter_anisotropic',

        'WEBKIT_EXT_texture_filter_anisotropic',

        'EXT_sRGB',

        'OES_element_index_uint',

        'OES_standard_derivatives',

        'OES_texture_float',

        'OES_texture_float_linear',

        'OES_texture_half_float',

        'OES_texture_half_float_linear',

        'OES_vertex_array_object',

        'WEBGL_color_buffer_float',

        'WEBGL_compressed_texture_s3tc',

        'WEBKIT_WEBGL_compressed_texture_s3tc',

        'WEBGL_debug_renderer_info',

        'WEBGL_debug_shaders',

        'WEBGL_depth_texture',

        'WEBKIT_WEBGL_depth_texture',

        'WEBGL_draw_buffers',

        'WEBGL_lose_context',

        'WEBKIT_WEBGL_lose_context'

    ];

};
```
## 音频指纹绕过
### 6. AudioContext 指纹干扰
```
// 方法1: 添加随机噪声到音频输出
const originalCreateOscillator = AudioContext.prototype.createOscillator;
AudioContext.prototype.createOscillator = function() {
    const oscillator = originalCreateOscillator.apply(this, arguments);

    const originalStart = oscillator.start;
    oscillator.start = function() {
        // 添加微小的频率偏移
        const offset = (Math.random() - 0.5) * 0.001;
        this.frequency.value += offset;
        return originalStart.apply(this, arguments);
    };

    return oscillator;
};

// 方法2: Hook AnalyserNode
const originalCreateAnalyser = AudioContext.prototype.createAnalyser;
AudioContext.prototype.createAnalyser = function() {
    const analyser = originalCreateAnalyser.apply(this, arguments);

    const originalGetFloatFrequencyData = analyser.getFloatFrequencyData;
    analyser.getFloatFrequencyData = function(array) {
        originalGetFloatFrequencyData.apply(this, arguments);

        // 添加轻微噪声
        for (let i = 0; i 
```
## 字体指纹绕过
### 7. 字体检测绕过
```
// 方法1: Hook measureText返回固定宽度

const originalMeasureText = CanvasRenderingContext2D.prototype.measureText;

CanvasRenderingContext2D.prototype.measureText = function(text) {

    const result = originalMeasureText.apply(this, arguments);

    // 为常见字体返回标准化的宽度

    const fontFamilies = ['Arial', 'Helvetica', 'Times', 'Courier', 'Verdana', 'Georgia', 'Palatino', 'Garamond', 'Bookman', 'Comic Sans MS', 'Trebuchet MS', 'Arial Black', 'Impact'];

    for (const font of fontFamilies) {

        if (this.font.includes(font)) {

            // 返回固定宽度，消除字体差异

            Object.defineProperty(result, 'width', {

                value: text.length * 12, // 简单的固定宽度计算

                writable: false

            });

            break;

        }

    }

    return result;

};

// 方法2: 标准化offsetWidth和offsetHeight

const originalOffsetWidth = Object.getOwnPropertyDescriptor(HTMLElement.prototype, 'offsetWidth');

const originalOffsetHeight = Object.getOwnPropertyDescriptor(HTMLElement.prototype, 'offsetHeight');

Object.defineProperty(HTMLElement.prototype, 'offsetWidth', {

    get: function() {

        if (this.style.fontFamily) {

            return 100; // 返回固定宽度

        }

        return originalOffsetWidth.get.call(this);

    },

    configurable: true

});

Object.defineProperty(HTMLElement.prototype, 'offsetHeight', {

    get: function() {

        if (this.style.fontFamily) {

            return 20; // 返回固定高度

        }

        return originalOffsetHeight.get.call(this);

    },

    configurable: true

});
```
## 时间指纹绕过
### 8. 性能时间戳干扰
