# Javascript

这是 JavaScript 的第一页，下面会放置一些自己的油猴脚本编写的示例：



### 首字母大写

```javascript
    /**
     * 将字符串的第一个字母转换为大写，如果后续字符与第一个字符相同（忽略大小写），也将它们转换为大写。
     * 例如，'kkFile' 会被转换为 'KKFile'。
     *
     * @param {string} string - 待处理的字符串。
     * @returns {string} - 转换后的字符串。
     */
    function capitalizeFirstLetterAndMatched(string) {
        if (!string || string.length < 2) return string;

        // 将第一个字符转换为大写
        let result = string.charAt(0).toUpperCase();

        // 检查第二个字符是否与第一个字符相同
        if (string[1].toLowerCase() === string[0].toLowerCase()) {
            // 第二个字符与第一个字符相同，继续执行转换
            for (let i = 1; i < string.length; i++) {
                if (string[i].toLowerCase() === string[0].toLowerCase()) {
                    result += string[i].toUpperCase();
                } else {
                    result += string[i];
                }
            }
        } else {
            // 第二个字符与第一个字符不同，保持剩余部分不变
            result += string.substring(1);
        }

        return result;
    }
```



### 百度翻译

```javascript
    /**
     * 翻译内容的函数。
     * 这个函数使用百度翻译API来翻译给定的文本，并调用回调函数处理翻译后的文本。
     *
     * @param {string} query - 需要翻译的文本。
     * @param {Function} callback - 翻译成功后执行的回调函数，该函数接受翻译后的文本作为参数。
     */
    function translateContent(query, callback) {
            // 获取当前时间戳，用于生成签名
            const salt = (new Date()).getTime();
            // 创建用于生成签名的字符串
            const str1 = appid + query + salt + key;
            // 使用CryptoJS库生成MD5签名
            const sign = CryptoJS.MD5(str1).toString(CryptoJS.enc.Hex);
            // 百度翻译API的URL
            const url = 'http://api.fanyi.baidu.com/api/trans/vip/translate';
            // 创建POST请求的数据体
            const data = `q=${encodeURIComponent(query)}&appid=${appid}&salt=${salt}&from=${from}&to=${to}&sign=${sign}`;
            // 使用GM.xmlHttpRequest发送POST请求
            GM.xmlHttpRequest({
                method: 'POST',
                url: url,
                headers: {
                    'Content-Type': 'application/x-www-form-urlencoded'
                },
                data: data,
                onload: function(response) {
                    // 检查响应状态
                    if (response.status === 200) {
                        // 解析响应内容为JSON
                        const respjson = JSON.parse(response.responseText);
                        // 检查是否有翻译结果
                        if (respjson && respjson.trans_result && respjson.trans_result.length > 0) {
                            // 调用回调函数，并传递翻译结果
                            callback(respjson.trans_result[0].dst);
                        } else {
                            console.error('翻译失败:', data);
                        }
                    } else {
                        console.error('翻译请求失败:', response);
                    }
                }
            });
        }
```

### 页面加载完成后执行

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
    
    
        /**
     * initialCheck函数
     *
     * 该函数用于集中处理所有需要在页面加载或变化时运行的函数。
     * 只需在该函数中添加或删除函数调用，即可轻松管理哪些函数会被执行。
     */
    function initialCheck() {
        // 如果您有一个用于自动登录的函数
        autologin();
        // 检查是否出现了特定的规则添加窗体，并据此执行其他函数
        if (rule_addition_control()) {
            // 在这里执行您的其他函数
            console.log('初始检查已完成。');
        }
        // 在这里添加其他需要执行的函数
        // customFunction3();
        // customFunction4();
        // ...
    }

    // 初始化MutationObserver
    const observer = new MutationObserver(function(mutations) {
        initialCheck();
    });

    // 配置MutationObserver
    const config = { attributes: true, childList: true, subtree: true };

    // 开始观察
    observer.observe(document.body, config);

    // 页面加载完成后执行
    window.addEventListener('load', function() {
        showLoadingMessage();
    });
```

### 自动登录

```javascript
    /**
     * 自动登录函数
     */
    function autologin() {
        // 检查是否在登录页面，通过查找特定的<div>元素来判断
        const loginBackground = document.getElementById('LoginBackgroud-body');
        if (loginBackground) {
            try {
                // 获取用户名和密码输入框，并填充信息
                const usernameInput = document.querySelector('input[name="userName"]');
                const passwordInput = document.querySelector('input[name="password"]');

                if (usernameInput && passwordInput) {
                    usernameInput.value = GLOBAL_CONFIG.username;
                    passwordInput.value = GLOBAL_CONFIG.password;

                    // 找到登录按钮并模拟点击
                    const loginButton = document.querySelector('span.x-btn-icon-el.icon-login');
                    if (loginButton) {
                        loginButton.click();
                    } else {
                        console.warn('未找到登录按钮');
                    }
                } else {
                    console.warn('用户名或密码输入框未找到');
                }
            } catch (error) {
                console.error('自动登录失败:', error);
            }
        }
    }
```
