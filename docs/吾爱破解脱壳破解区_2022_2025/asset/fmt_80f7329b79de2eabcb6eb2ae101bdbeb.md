
```
console.warn(`检测到指纹识别API: ${api}`);

            }

        });

    });

    if (detectionCount > 3) {

        console.warn('⚠️ 检测到大量指纹识别代码，建议启用防护');

        return true;

    }

    return false;

}

// 自动启用防护

if (detectFingerprinting()) {

    console.log('🛡️ 自动启用指纹防护');

    // 执行防护代码

}
```
