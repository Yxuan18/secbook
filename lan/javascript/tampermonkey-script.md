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

### 使用 百度翻译

```javascript
    /**
     * @function translateContent
     * @description 使用百度翻译API将中文文本翻译成英文
     * @param {string} query - 源文本
     * @returns {Promise<string>} 翻译后的文本
     */
    function translateContent(query) {
        return new Promise((resolve, reject) => {
            const salt = Date.now();
            const signStr = BAIDU_APPID + query + salt + BAIDU_KEY;
            const sign = CryptoJS.MD5(signStr).toString();
            const url = 'http://api.fanyi.baidu.com/api/trans/vip/translate';
            const data = `q=${encodeURIComponent(query)}&appid=${BAIDU_APPID}&salt=${salt}&from=${BAIDU_FROM}&to=${BAIDU_TO}&sign=${sign}`;

            GM.xmlHttpRequest({
                method: 'POST',
                url,
                headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
                data,
                onload: (response) => {
                    if (response.status === 200) {
                        const respJSON = JSON.parse(response.responseText);
                        if (respJSON && respJSON.trans_result?.length) {
                            return resolve(respJSON.trans_result[0].dst);
                        }
                        console.error('翻译失败:', respJSON);
                        return reject('翻译失败');
                    }
                    console.error('翻译请求失败:', response);
                    reject('翻译请求失败');
                },
                onerror: (err) => {
                    console.error('请求发送错误:', err);
                    reject(err);
                }
            });
        });
    }
```

### 文本框之间的自动翻译

```javascript
    /**
     * @function setupInputEventListener
     * @description 当用户在源输入框中输入时，自动翻译并写入目标输入框（防抖处理）
     * @param {string} containerId - 容器的 ID
     * @param {string} sourceSelector - 源输入框的选择器
     * @param {string} targetSelector - 目标输入框的选择器
     */
    function setupInputEventListener(containerId, sourceSelector, targetSelector) {
        const container = document.getElementById(containerId);
        if (!container) {
            console.error(`未找到容器 ID: ${containerId}`);
            return;
        }
        const sourceEl = container.querySelector(sourceSelector);
        if (!sourceEl) {
            console.error(`未在容器 '${containerId}' 中找到源元素: '${sourceSelector}'`);
            return;
        }
        sourceEl.addEventListener('input', debounce(() => {
            const sourceText = sourceEl.value.trim();
            if (!sourceText) return; // 若为空，不发起翻译请求

            translateContent(sourceText)
                .then((translated) => {
                    const targetEl = container.querySelector(targetSelector);
                    if (targetEl) {
                        targetEl.value = translated;
                    } else {
                        console.error(`未在容器 '${containerId}' 中找到目标元素: '${targetSelector}'`);
                    }
                })
                .catch((error) => console.error('翻译失败:', error));
        }, 500));
    }
```

