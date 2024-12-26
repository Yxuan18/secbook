# Tampermonkey Script

这里放一些我写油候脚本时候的示例

### 获取元素的异步函数

```javascript
    /**
 * 获取元素的异步函数
 * @param {string} selector - 元素的选择器
 * @param {number} delay - 尝试获取元素的最大等待时间（毫秒）
 * @returns {Promise<HTMLElement>} - 返回找到的元素或抛出错误
 */
    function getElement(selector, delay) {
        return new Promise((resolve, reject) => {
            const element = document.querySelector(selector);
            if (element) {
                resolve(element);
            } else {
                setTimeout(() => {
                    const elementAfterDelay = document.querySelector(selector);
                    if (elementAfterDelay) {
                        resolve(elementAfterDelay);
                    } else {
                        reject(new Error(`元素未找到: ${selector}`));
                    }
                }, delay);
            }
        });
    }
```

### 显示 正在加载插件 消息

```javascript
     /**
     * 显示"正在加载插件"消息
     */
    function showLoadingMessage() {
        // 创建一个新的div元素来显示加载消息
        var loadingMessage = document.createElement('div');
        // 设置消息内容
        loadingMessage.innerHTML = '正在加载插件';
        // 设置消息的样式
        loadingMessage.style.position = 'fixed';
        loadingMessage.style.top = '50px';
        loadingMessage.style.left = '50%';
        loadingMessage.style.fontSize = '20px';
        loadingMessage.style.opacity = '1';
        // 将消息元素添加到页面上
        document.body.appendChild(loadingMessage);
        // 设置一个定时器，在2秒后开始淡出动画
        setTimeout(function() {
            // 使用CSS过渡来改变透明度
            loadingMessage.style.transition = 'opacity 1s ease-out';
            // 设置透明度为0以启动淡出动画
            loadingMessage.style.opacity = '0';
        }, 2000);
    }
```
