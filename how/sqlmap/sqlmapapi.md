# sqlmapapi

### 为什么要使用SQLMAP API？

由于SQLMAP每检测一个站点都需要开启一个新的命令行窗口或者结束掉上一个检测任务。虽然 -m 参数可以批量扫描URL，但是模式也是一个结束扫描后才开始另一个扫描任务。通过api接口，下发扫描任务就简单了，无需开启一个新的命令行窗口。

sqlmap安装完成后，输入以下命令，返回内容如下图一样，意味着安装成功：

```
python sqlmap.py -h
```

![深入了解sqlmap\_api](https://image.3001.net/images/20190529/1559139187\_5cee93733eccc.png!small)

#### sqlmap api <a href="#h3-1" id="h3-1"></a>

说了那么多，到底api如何使用呢？在下载安装SQLMAP后，你会在sqlmap安装目录中找到一个 sqlmapapi.py 的文件，这个 sqlmapapi.py 文件就是sqlmmap api。sqlmap api分为服务端和客户端，sqlmap api有两种模式，一种是基于HTTP协议的接口模式，一种是基于命令行的接口模式。

### sqlmapapi.py的使用帮助 <a href="#h2-4" id="h2-4"></a>

通过以下命令获取sqlmapapi.py的使用帮助：

```
python sqlmapapi.py -h
```

返回的信息：

```
Usage: sqlmapapi.py [options]

Options:
  -h, --help            显示帮助信息并退出
  -s, --server          做为api服务端运行
  -c, --client          做为api客户端运行
  -H HOST, --host=HOST  指定服务端IP地址 (默认IP是 "127.0.0.1")
  -p PORT, --port=PORT  指定服务端端口 (默认端口8775)
  --adapter=ADAPTER     服务端标准接口 (默认是 "wsgiref")
  --username=USERNAME   可空，设置用户名
  --password=PASSWORD   可空，设置密码
```

### 开启api服务端 <a href="#h2-5" id="h2-5"></a>

无论是基于HTTP协议的接口模式还是基于命令行的接口模式，首先都是需要开启api服务端的。通过输入以下命令即可开启api服务端:

```
python sqlmapapi.py -s
```

命令成功后，在命令行中会返回一些信息。以下命令大概的意思是api服务端在本地8775端口上运行，admin token为1acac56427f272e316fceabe5ddff5a5，IPC数据库的位置在/tmp/sqlmapipc-zOIGm\_，api服务端已经和IPC数据库连接上了，正在使用bottle 框架wsgiref标准接口。

```
[19:53:57] [INFO] Running REST-JSON API server at '127.0.0.1:8775'..
[19:53:57] [INFO] Admin (secret) token: 1acac56427f272e316fceabe5ddff5a5
[19:53:57] [DEBUG] IPC database: '/tmp/sqlmapipc-zOIGm_'
[19:53:57] [DEBUG] REST-JSON API server connected to IPC database
[19:53:57] [DEBUG] Using adapter 'wsgiref' to run bottle
```

但是通过上面的这种方式开启api服务端有一个缺点，当服务端和客户端不是一台主机会连接不上，因此如果要解决这个问题，可以通过输入以下命令来开启api服务端:

```
python sqlmapapi.py -s -H "0.0.0.0" -p 8775
```

命令成功后，远程客户端就可以通过指定远程主机IP和端口来连接到API服务端。

#### 固定admin token <a href="#h3-2" id="h3-2"></a>

如果您有特殊的需求需要固定admin token的话，可以修改文件api.py，该文件在sqlmap目录下的/lib/utils/中，修改该文件的第661行代码，以下是源代码：

```
DataStore.admin_token = hexencode(os.urandom(16))
```

### sqlmap api的两种模式 <a href="#h2-6" id="h2-6"></a>

#### 命令行接口模式 <a href="#h3-3" id="h3-3"></a>

输入以下命令，可连接api服务端，进行后期的指令发送操作：

```
python sqlmapapi.py -c
```

如果是客户端和服务端不是同一台计算机的话，输入以下命令：

```
python sqlmapapi.py -c -H "192.168.1.101" -p 8775
```

输入以上命令后，会进入交互模式，如下图所示：

![深入了解sqlmap\_api](https://image.3001.net/images/20190529/1559139209\_5cee9389cec01.png!small)

通过在交互模式下输入help命令，获取所有命令，以下是该接口模式的所有命令：

```
api> help
help           显示帮助信息
new ARGS       开启一个新的扫描任务
use TASKID     切换taskid
data           获取当前任务返回的数据
log            获取当前任务的扫描日志
status         获取当前任务的扫描状态
option OPTION  获取当前任务的选项
options        获取当前任务的所有配置信息
stop           停止当前任务
kill           杀死当前任务
list           显示所有任务列表
flush          清空所有任务
exit           退出客户端        t
```

既然了解了命令行接口模式中所有命令，那么下面就通过一个sql注入来演示该模式接口下检测sql注入的流程。

#### 检测GET型注入 <a href="#h3-4" id="h3-4"></a>

通过输入以下命令可以检测GET注入

```
new -u "url"
```

虽然我们仅仅只指定了-u参数，但是从返回的信息中可以看出，输入new命令后，首先先请求了/task/new，来创建一个新的taskid，后又发起了一个请求去开始任务，因此可以发现该模式实质也是基于HTTP协议的。

![深入了解sqlmap\_api](https://image.3001.net/images/20190529/1559139219\_5cee939319313.png!small)

通过输入 status 命令，来获取该任务的扫描状态，若返回内容中的status字段为terminated，说明扫描完成，若返回内容中的status字段为run，说明扫描还在进行中。下图是扫描完成的截图：

![深入了解sqlmap\_api](https://image.3001.net/images/20190529/1559139225\_5cee939939a21.png!small)

通过输入 data 命令，来获取扫描完成后注入出来的信息，若返回的内容中data字段不为空就说明存在注入。下图是存在SQL注入返回的内容，可以看到返回的内容有数据库类型、payload、注入的参数等等。

![深入了解sqlmap\_api](https://image.3001.net/images/20190529/1559139235\_5cee93a35311b.png!small)

下图是不存在注入返回的内容，data字段为空：

![深入了解sqlmap\_api](https://image.3001.net/images/20190529/1559139243\_5cee93ab83462.png!small)

#### 检测POST型、cookie、UA等注入 <a href="#h3-5" id="h3-5"></a>

通过输入以下命令，在data.txt中加入星号，指定注入的位置，来达到检测POST、cookie、UA等注入的目的：

```
new -r data.txt
```

![深入了解sqlmap\_api](https://image.3001.net/images/20190529/1559139250\_5cee93b21150b.png!small)

#### 总结 <a href="#h3-6" id="h3-6"></a>

基于命令行的接口模式用起来还是比较方便的，我们只需要通过 new 命令就可以自动创建taskid并开始该任务，但是如果写程序调用的话可能就不是那么友善了，因此笔者会重点介绍下面的一种接口模式，基于HTTP协议的接口模式。

### 基于HTTP协议的接口模式 <a href="#h2-8" id="h2-8"></a>

下列都是基于HTTP协议API交互的所有方法：提示：“@get”就说明需要通过GET请求的，“@post”就说明需要通过POST请求的；POST请求需要修改HTTP头中的Content-Type字段为application/json。

```
#辅助
@get('/error/401')    
@get("/task/new")
@get("/task/<taskid>/delete")

#Admin 命令
@get("/admin/list")
@get("/admin/<token>/list")
@get("/admin/flush")
@get("/admin/<token>/flush")

#sqlmap 核心交互命令
@get("/option/<taskid>/list")
@post("/option/<taskid>/get")
@post("/option/<taskid>/set")
@post("/scan/<taskid>/start")
@get("/scan/<taskid>/stop")
@get("/scan/<taskid>/kill")
@get("/scan/<taskid>/status")
@get("/scan/<taskid>/data")
@get("/scan/<taskid>/log/<start>/<end>")
@get("/scan/<taskid>/log")
@get("/download/<taskid>/<target>/<filename:path>")
```

#### @get('/error/401') <a href="#h3-7" id="h3-7"></a>

该接口在我的理解表明首先需要登录（Admin token），不然会返回状态码401。具体代码如下：

```
    response.status = 401
    return response
```

![深入了解sqlmap\_api](https://image.3001.net/images/20190529/1559139258\_5cee93bac9433.png!small)

#### @get("/task/new") <a href="#h3-9" id="h3-9"></a>

该接口用于创建一个新的任务，使用后会返回一个随机的taskid。具体代码如下：

```
def task_new():
    """
    Create a new task
    """
    taskid = hexencode(os.urandom(8))
    remote_addr = request.remote_addr
    DataStore.tasks[taskid] = Task(taskid, remote_addr)
    logger.debug("Created new task: '%s'" % taskid)
    return jsonize({"success": True, "taskid": taskid})
```

下图是调用该接口的截图：

![深入了解sqlmap\_api](https://image.3001.net/images/20190529/1559139266\_5cee93c252984.png!small)

#### @get("/task//delete") <a href="#h3-10" id="h3-10"></a>

该接口用于删除taskid。在调用时指定taskid，不指定taskid会有问题。具体代码如下：

```
def task_delete(taskid):
    """
    Delete an existing task
    """
    if taskid in DataStore.tasks:
        DataStore.tasks.pop(taskid)
        logger.debug("(%s) Deleted task" % taskid)
        return jsonize({"success": True})
    else:
        response.status = 404
        logger.warning("[%s] Non-existing task ID provided to task_delete()" % taskid)
        return jsonize({"success": False, "message": "Non-existing task ID"})
```

下图是调用该接口的截图：

![深入了解sqlmap\_api](https://image.3001.net/images/20190529/1559139272\_5cee93c8056d3.png!small)

#### @get("/admin/list")/@get("/admin//list") <a href="#h3-11" id="h3-11"></a>

该接口用于返回所有taskid。在调用时指定taskid，不指定taskid会有问题。具体代码如下：

```
def task_list(token=None):    
    """
    Pull task list
    """
    tasks = {}
    for key in DataStore.tasks:
        if is_admin(token) or DataStore.tasks[key].remote_addr == request.remote_addr:
            tasks[key] = dejsonize(scan_status(key))["status"]
    logger.debug("(%s) Listed task pool (%s)" % (token, "admin" if is_admin(token) else request.remote_addr))
    return jsonize({"success": True, "tasks": tasks, "tasks_num": len(tasks)})
```

下图是调用该接口的截图：

![深入了解sqlmap\_api](https://image.3001.net/images/20190529/1559139281\_5cee93d155fac.png!small)

#### @get("/admin/flush")/@get("/admin//flush") <a href="#h3-12" id="h3-12"></a>

该接口用于删除所有任务。在调用时指定admin token，不指定admin token可能会有问题。具体代码如下：

```
def task_flush(token=None):
    """
    Flush task spool (delete all tasks)
    """
    for key in list(DataStore.tasks):
        if is_admin(token) or DataStore.tasks[key].remote_addr == request.remote_addr:
            DataStore.tasks[key].engine_kill()
            del DataStore.tasks[key]
    logger.debug("(%s) Flushed task pool (%s)" % (token, "admin" if is_admin(token) else request.remote_addr))
    return jsonize({"success": True})
```

下图是调用该接口的截图：

![深入了解sqlmap\_api](https://image.3001.net/images/20190529/1559139287\_5cee93d79ff4c.png!small)

#### @get("/option//list") <a href="#h3-13" id="h3-13"></a>

该接口可获取特定任务ID的列表选项，调用时请指定taskid，不然会出现问题。具体代码如下：

```
def option_list(taskid):
    """
    List options for a certain task ID
    """
    if taskid not in DataStore.tasks:
        logger.warning("[%s] Invalid task ID provided to option_list()" % taskid)
        return jsonize({"success": False, "message": "Invalid task ID"})
    logger.debug("(%s) Listed task options" % taskid)
    return jsonize({"success": True, "options": DataStore.tasks[taskid].get_options()})
```

下图是调用该接口的截图：

![深入了解sqlmap\_api](https://image.3001.net/images/20190529/1559139293\_5cee93dd8d8ea.png!small)

#### @post("/option//get") <a href="#h3-14" id="h3-14"></a>

该接口可获取特定任务ID的选项值，调用时请指定taskid，不然会出现问题。具体代码如下：

```
def option_get(taskid):
    """
    Get value of option(s) for a certain task ID
    """
    if taskid not in DataStore.tasks:
        logger.warning("[%s] Invalid task ID provided to option_get()" % taskid)
        return jsonize({"success": False, "message": "Invalid task ID"})
    options = request.json or []
    results = {}
    for option in options:
        if option in DataStore.tasks[taskid].options:
            results[option] = DataStore.tasks[taskid].options[option]
        else:
            logger.debug("(%s) Requested value for unknown option '%s'" % (taskid, option))
            return jsonize({"success": False, "message": "Unknown option '%s'" % option})
    logger.debug("(%s) Retrieved values for option(s) '%s'" % (taskid, ",".join(options)))
    return jsonize({"success": True, "options": results})
```

下图是调用该接口的截图：

![深入了解sqlmap\_api](https://image.3001.net/images/20190529/1559139300\_5cee93e42c4ed.png!small)

#### @post("/option//set") <a href="#h3-15" id="h3-15"></a>

该接口为特定任务ID设置选项值，调用时请指定taskid，不然会出现问题。具体代码如下：

```
def option_set(taskid):
    """
    Set value of option(s) for a certain task ID
    """
    if taskid not in DataStore.tasks:
        logger.warning("[%s] Invalid task ID provided to option_set()" % taskid)
        return jsonize({"success": False, "message": "Invalid task ID"})
    if request.json is None:
        logger.warning("[%s] Invalid JSON options provided to option_set()" % taskid)
        return jsonize({"success": False, "message": "Invalid JSON options"})
    for option, value in request.json.items():
        DataStore.tasks[taskid].set_option(option, value)
    logger.debug("(%s) Requested to set options" % taskid)
    return jsonize({"success": True})
```

下图是调用该接口的截图：

![深入了解sqlmap\_api](https://image.3001.net/images/20190529/1559139307\_5cee93eb39651.png!small)

#### @post("/scan//start") <a href="#h3-16" id="h3-16"></a>

该接口定义开始扫描特定任务，调用时请指定taskid，不然会出现问题。具体代码如下:

```
def scan_start(taskid):
    """
    Launch a scan
    """
    if taskid not in DataStore.tasks:
        logger.warning("[%s] Invalid task ID provided to scan_start()" % taskid)
        return jsonize({"success": False, "message": "Invalid task ID"})
    if request.json is None:
        logger.warning("[%s] Invalid JSON options provided to scan_start()" % taskid)
        return jsonize({"success": False, "message": "Invalid JSON options"})
    # Initialize sqlmap engine's options with user's provided options, if any
    for option, value in request.json.items():
        DataStore.tasks[taskid].set_option(option, value)
    # Launch sqlmap engine in a separate process
    DataStore.tasks[taskid].engine_start()
    logger.debug("(%s) Started scan" % taskid)
    return jsonize({"success": True, "engineid": DataStore.tasks[taskid].engine_get_id()})
```

下图是调用该接口的截图：

![深入了解sqlmap\_api](https://image.3001.net/images/20190529/1559139313\_5cee93f1d1c3e.png!small)

#### @get("/scan//stop") <a href="#h3-17" id="h3-17"></a>

该接口定义停止扫描特定任务，调用时请指定taskid，不然会出现问题。具体代码如下：

```
def scan_stop(taskid):
    """
    Stop a scan
    """
    if (taskid not in DataStore.tasks or DataStore.tasks[taskid].engine_process() is None or DataStore.tasks[taskid].engine_has_terminated()):
        logger.warning("[%s] Invalid task ID provided to scan_stop()" % taskid)
        return jsonize({"success": False, "message": "Invalid task ID"})
    DataStore.tasks[taskid].engine_stop()
    logger.debug("(%s) Stopped scan" % taskid)
    return jsonize({"success": True})
```

下图是调用该接口的截图：

![深入了解sqlmap\_api](https://image.3001.net/images/20190529/1559139320\_5cee93f862c62.png!small)

#### @get("/scan//kill") <a href="#h3-18" id="h3-18"></a>

该接口可杀死特定任务，需要指定taskid，不然会出现问题。具体代码如下：

```
def scan_kill(taskid):
    """
    Kill a scan
    """
    if (taskid not in DataStore.tasks or DataStore.tasks[taskid].engine_process() is None or DataStore.tasks[taskid].engine_has_terminated()):
        logger.warning("[%s] Invalid task ID provided to scan_kill()" % taskid)
        return jsonize({"success": False, "message": "Invalid task ID"})
    DataStore.tasks[taskid].engine_kill()
    logger.debug("(%s) Killed scan" % taskid)
    return jsonize({"success": True})
```

#### @get("/scan//status") <a href="#h3-19" id="h3-19"></a>

该接口可查询扫描状态，调用时请指定taskid，不然会出现问题。具体代码如下：

```
def scan_status(taskid):
    """
    Returns status of a scan
    """
    if taskid not in DataStore.tasks:
        logger.warning("[%s] Invalid task ID provided to scan_status()" % taskid)
        return jsonize({"success": False, "message": "Invalid task ID"})
    if DataStore.tasks[taskid].engine_process() is None:
        status = "not running"
    else:
        status = "terminated" if DataStore.tasks[taskid].engine_has_terminated() is True else "running"
    logger.debug("(%s) Retrieved scan status" % taskid)
    return jsonize({
        "success": True,
        "status": status,
        "returncode": DataStore.tasks[taskid].engine_get_returncode()
    })
```

下图是调用该接口的截图：

![深入了解sqlmap\_api](https://image.3001.net/images/20190529/1559139328\_5cee9400afdd5.png!small)

#### @get("/scan//data") <a href="#h3-20" id="h3-20"></a>

该接口可获得到扫描结果，调用时请指定taskid，不然会出现问题。具体代码如下：

```
def scan_data(taskid):
    """
    Retrieve the data of a scan
    """
    json_data_message = list()
    json_errors_message = list()
    if taskid not in DataStore.tasks:
        logger.warning("[%s] Invalid task ID provided to scan_data()" % taskid)
        return jsonize({"success": False, "message": "Invalid task ID"})
    # Read all data from the IPC database for the taskid
    for status, content_type, value in DataStore.current_db.execute("SELECT status, content_type, value FROM data WHERE taskid = ? ORDER BY id ASC", (taskid,)):
        json_data_message.append({"status": status, "type": content_type, "value": dejsonize(value)})
    # Read all error messages from the IPC database
    for error in DataStore.current_db.execute("SELECT error FROM errors WHERE taskid = ? ORDER BY id ASC", (taskid,)):
        json_errors_message.append(error)
    logger.debug("(%s) Retrieved scan data and error messages" % taskid)
    return jsonize({"success": True, "data": json_data_message, "error": json_errors_message})
```

下图是调用该接口的截图：存在SQL注入的返回结果，返回的内容包括payload、数据库类型等等。

![深入了解sqlmap\_api](https://image.3001.net/images/20190529/1559139336\_5cee94080813c.png!small)

#### @get("/scan//log") /@get("/scan//log//") <a href="#h3-21" id="h3-21"></a>

该接口可查询特定任务的扫描的日志，调用时请指定taskid，不然会出现问题。具体代码如下：

```
def scan_log(taskid):
    """
    Retrieve the log messages
    """
    json_log_messages = list()
    if taskid not in DataStore.tasks:
        logger.warning("[%s] Invalid task ID provided to scan_log()" % taskid)
        return jsonize({"success": False, "message": "Invalid task ID"})
    # Read all log messages from the IPC database
    for time_, level, message in DataStore.current_db.execute("SELECT time, level, message FROM logs WHERE taskid = ? ORDER BY id ASC", (taskid,)):
        json_log_messages.append({"time": time_, "level": level, "message": message})
    logger.debug("(%s) Retrieved scan log messages" % taskid)
    return jsonize({"success": True, "log": json_log_messages})


def scan_log_limited(taskid, start, end):
    """
    Retrieve a subset of log messages
    """
    json_log_messages = list()
    if taskid not in DataStore.tasks:
        logger.warning("[%s] Invalid task ID provided to scan_log_limited()" % taskid)
        return jsonize({"success": False, "message": "Invalid task ID"})
    if not start.isdigit() or not end.isdigit() or end < start:
        logger.warning("[%s] Invalid start or end value provided to scan_log_limited()" % taskid)
        return jsonize({"success": False, "message": "Invalid start or end value, must be digits"})
    start = max(1, int(start))
    end = max(1, int(end))
    # Read a subset of log messages from the IPC database
    for time_, level, message in DataStore.current_db.execute("SELECT time, level, message FROM logs WHERE taskid = ? AND id >= ? AND id <= ? ORDER BY id ASC", (taskid, start, end)):
        json_log_messages.append({"time": time_, "level": level, "message": message})
    logger.debug("(%s) Retrieved scan log messages subset" % taskid)
    return jsonize({"success": True, "log": json_log_messages})
```

下图是调用该接口的截图：

![深入了解sqlmap\_api](https://image.3001.net/images/20190529/1559139413\_5cee9455ab9a0.png!small)

#### @get("/download///") <a href="#h3-22" id="h3-22"></a>

下载服务端指定任务的文件。具体代码如下：

```
def download(taskid, target, filename):
    """
    Download a certain file from the file system
    """
    if taskid not in DataStore.tasks:
        logger.warning("[%s] Invalid task ID provided to download()" % taskid)
        return jsonize({"success": False, "message": "Invalid task ID"})
    path = os.path.abspath(os.path.join(paths.SQLMAP_OUTPUT_PATH, target, filename))
    # Prevent file path traversal
    if not path.startswith(paths.SQLMAP_OUTPUT_PATH):
        logger.warning("[%s] Forbidden path (%s)" % (taskid, target))
        return jsonize({"success": False, "message": "Forbidden path"})
    if os.path.isfile(path):
        logger.debug("(%s) Retrieved content of file %s" % (taskid, target))
        with open(path, 'rb') as inf:
            file_content = inf.read()
        return jsonize({"success": True, "file": base64encode(file_content)})
    else:
        logger.warning("[%s] File does not exist %s" % (taskid, target))
        return jsonize({"success": False, "message": "File does not exist"})
```

看完了以上的接口代码，我相信您一定会使用了吧，那么下面就通过一个sql注入来演示该模式接口下检测sql注入的流程。

#### 准备 <a href="#h3-23" id="h3-23"></a>

使用该模式接口需要用到python的两个库文件，一个是requests库，一个是json库。下图是导入库的操作：

![深入了解sqlmap\_api](https://image.3001.net/images/20190529/1559139371\_5cee942b4a22d.png!small)

#### 检测GET型注入 <a href="#h3-24" id="h3-24"></a>

下面是完整的一次API接口访问，"从创建任务ID，到发送扫描指令，再到查询扫描状态，最后查询结果”的过程。

![深入了解sqlmap\_api](https://image.3001.net/images/20190529/1559139378\_5cee94324eee2.png!small)

具体输入输出代码如下：

```
>>> r = requests.get("http://127.0.0.1:8775/task/new")  创建一个新的扫描任务
>>> r.json()
{'taskid': 'c87dbb00644ed7b7', 'success': True} 获取响应的返回内容
>>> r = requests.post('http://127.0.0.1:8775/scan/c87dbb00644ed7b7/start', data=json.dumps({'url':'http://192.168.1.104/sql-labs/Less-2/?id=1'}), headers={'Content-Type':'application/json'})  开启一个扫描任务
>>> r = requests.get("http://127.0.0.1:8775/scan/c87dbb00644ed7b7/status")  查询任务的扫描状态
>>> r.json()
{'status': 'terminated', 'returncode': 0, 'success': True}
>>> r = requests.get("http://127.0.0.1:8775/scan/c87dbb00644ed7b7/data")  
获取扫描的结果
>>> r.json()
{'data': [{'status': 1, 'type': 0, 'value': {'url': 'http://192.168.1.104:80/sql-labs/Less-2/', 'query': 'id=1', 'data': None}}, {'status': 1, 'type': 1, 'value': [{'dbms': 'MySQL', 'suffix': '', 'clause': [1, 8, 9], 'notes': [], 'ptype': 1, 'dbms_version': ['>= 5.0'], 'prefix': '', 'place': 'GET', 'data': {'1': {'comment': '', 'matchRatio': 0.957, 'title': 'AND boolean-based blind - WHERE or HAVING clause', 'trueCode': 200, 'templatePayload': None, 'vector': 'AND [INFERENCE]', 'falseCode': 200, 'where': 1, 'payload': 'id=1 AND 8693=8693'}..., 'success': True, 'error': []}
```

可能您会被最后返回的结果好奇，ptype、suffix、clause等等都是什么意思呢？下面我给出部分字段的含义：

| 字段            |  含义                                          | 值                                                                                                                                                                                                         |
| ------------- | -------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| dbms          | 数据库类型                                        | Microsoft AccessIBM DB2FirebirdSAP MaxDBMicrosoft SQL ServerMySQLOraclePostgreSQLSQLiteSybaseHSQLDBH2Informix                                                                                             |
| suffix        | 在有些时候，需要在注入的payload的后面加一些字符，来保证payload的正常执行。 | GENERIC\_SQL\_COMMENT] AND (\[RANDNUM]=\[RANDNUM] AND ((\[RANDNUM]=\[RANDNUM] AND (((\[RANDNUM]=...太多了，具体可查看xml/boundaries.xml文件。                                                                         |
| prefix        | 在有些时候，需要在注入的payload的前面加一些字符，来保证payload的正常执行。 |  )')'"...太多了，具体可查看xml/boundaries.xml文件。                                                                                                                                                                   |
| clause        | payload在哪个语句里生效                              | 0: "Always",1: "WHERE",2: "GROUP BY",3: "ORDER BY",4: "LIMIT",5: "OFFSET",6: "TOP",7: "Table name",8: "Column name",9: "Pre-WHERE (non-query)"                                                            |
| where         | 以什么样的方式将我们的payload添加进去                       | ORIGINAL = 1NEGATIVE = 2REPLACE = 3                                                                                                                                                                       |
| ptype         | parameter type；注入点的类型                        | <p>1: "Unescaped numeric",<br>2: "Single quoted string",<br>3: "LIKE single quoted string",<br>4: "Double quoted string",<br>5: "LIKE double quoted string", <br>6: "Identifier (e.g. column name)", </p> |
| dbms\_version | 数据库的粗略版本                                     |  >= 5.5>= 5.0.12...                                                                                                                                                                                       |
| place         | 请求方式                                         | GETPOST(custom) HEADER                                                                                                                                                                                    |
|  title        | 当前测试Payload的标题，通过标题就可以了解当前的注入手法与测试的数据库类型。    | AND boolean-based blind - WHERE or HAVING clause...                                                                                                                                                       |
| vector        | payload向量                                    | AND (SELECT \* FROM (SELECT(SLEEP(\[SLEEPTIME]-(IF(\[INFERENCE],0,\[SLEEPTIME])))))\[RANDSTR])...                                                                                                         |
| payload       | 有效负荷,进行测试的SQL语句                              | <p>1、news_id=1 AND 6788=6788<br>2、login_user=1&#x26;login_password=-3814' OR 7117=7117#&#x26;mysubmit=Login </p>                                                                                          |
| parameter     | 哪个参数可以注入                                     | usernamenews\_id....                                                                                                                                                                                      |

#### 检测POST注入、COOKIE、UA等注入 <a href="#h3-25" id="h3-25"></a>

检测POST注入和检测GET注入类似，但是还是有一定区别的，与GET注入检测区别如下，流程上是一样的，不同的是开启扫描任务的时候，多提交一个data字段。

```
requests.post('http://127.0.0.1:8775/scan/cb9c4b4e4f1996b5/start', data=json.dumps({'url':'http://192.168.1.104/sql/sql/post.php','data':'keyword=1'}), headers={'Content-Type':'application/json'})
```

下面是一次完整的POST注入检测过程

![深入了解sqlmap\_api](https://image.3001.net/images/20190529/1559139385\_5cee943922633.png!small)

具体输入输出代码如下：

```
>>> r = requests.get("http://127.0.0.1:8775/task/new")
>>> r.json()
{'taskid': 'cb9c4b4e4f1996b5', 'success': True}
>>> r = requests.post('http://127.0.0.1:8775/scan/cb9c4b4e4f1996b5/start', data=json.dumps({'url':'http://192.168.1.104/sql/sql/post.php','data':'keyword=1'}), headers={'Content-Type':'application/json'})
>>> r.json()
{'engineid': 9682, 'success': True}
>>> r = requests.get("http://127.0.0.1:8775/scan/cb9c4b4e4f1996b5/status")
>>> r.json()
{'status': 'terminated', 'returncode': 0, 'success': True}
>>> r = requests.get("http://127.0.0.1:8775/scan/cb9c4b4e4f1996b5/data")
>>> r.json()
{'data': [{'status': 1, 'type': 0, 'value': {'url': 'http://192.168.1.104:80/sql/sql/post.php', 'query': None, 'data': 'keyword=1'}}, {'status': 1, 'type': 1, 'value': [{'dbms': 'MySQL', 'suffix': '', 'clause': [1, 8, 9], 'notes': [], 'ptype': 1, 'dbms_version': ['>= 5.0.12'], 'prefix': '', 'place': 'POST', 'os': None, 'conf': {'code': None, 'string': 'Title=FiveAourThe??', 'notString': None, 'titles': None, 'regexp': None, 'textOnly': None, 'optimize': None}, 'parameter': 'keyword', 'data': {'1': {'comment': '', 'matchRatio': 0.863, 'trueCode': 200, 'title': 'AND boolean-based blind - WHERE or HAVING clause', 'templatePayload': None, 'vector': 'AND [INFERENCE]', 'falseCode': 200, 'where': 1, 'payload': 'keyword=1 AND 3424=3424'}...], 'success': True, 'error': []}
```

那么如何检测COOKIE注入、UA注入这些呢？下面笔者将列出api接口可接收的所有字段:

若要检测COOKIE注入的话，我们只要在@post("/scan//start")接口中，传入cookie字段；\
若要检测referer注入的话，我们只要在@post("/scan//start")接口中，传入referer字段。\
若要从注入点中获取数据库的版本、数据库的用户名这些，只要在@post("/scan//start")接口中，传入getBanner字段，并设置为True，传入getUsers字段，并设置为True。

```
crawlDepth: None
osShell: False
getUsers: False
getPasswordHashes: False
excludeSysDbs: True
ignoreTimeouts: False
regData: None
fileDest: None
prefix: None
code: None
googlePage: 1
skip: None
query: None
randomAgent: False
osPwn: False
authType: None
safeUrl: None
requestFile: None
predictOutput: False
wizard: False
stopFail: False
forms: False
uChar: None
secondReq: None
taskid: 630f50607ebf91dc
pivotColumn: None
preprocess: None
dropSetCookie: False
smart: False
paramExclude: None
risk: 1
sqlFile: None
rParam: None
getCurrentUser: False
notString: None
getRoles: False
getPrivileges: False
testParameter: None
tbl: None
charset: None
trafficFile: None
osSmb: False
level: 1
dnsDomain: None
outputDir: None
skipWaf: False
timeout: 30
firstChar: None
torPort: None
getComments: False
binaryFields: None
checkTor: False
commonTables: False
direct: None
tmpPath: None
titles: False
getSchema: False
identifyWaf: False
paramDel: None
safeReqFile: None
regKey: None
murphyRate: None
limitStart: None
crawlExclude: None
flushSession: False
loadCookies: None
csvDel: ,
offline: False
method: None
tmpDir: None
fileWrite: None
disablePrecon: False
osBof: False
testSkip: None
invalidLogical: False
getCurrentDb: False
hexConvert: False
proxyFile: None
answers: None
host: None
dependencies: False
cookie: None
proxy: None
updateAll: False
regType: None
repair: False
optimize: False
limitStop: None
search: False
shLib: None
uFrom: None
noCast: False
testFilter: None
ignoreCode: None
eta: False
csrfToken: None
threads: 1
logFile: None
os: None
col: None
skipStatic: False
proxyCred: None
verbose: 1
isDba: False
encoding: None
privEsc: False
forceDns: False
getAll: False
api: True
url: http://10.20.40.95/sql-labs/Less-4/?id=1
invalidBignum: False
regexp: None
getDbs: False
freshQueries: False
uCols: None
smokeTest: False
udfInject: False
invalidString: False
tor: False
forceSSL: False
beep: False
noEscape: False
configFile: None
scope: None
authFile: None
torType: SOCKS5
regVal: None
dummy: False
checkInternet: False
safePost: None
safeFreq: None
skipUrlEncode: False
referer: None
liveTest: False
retries: 3
extensiveFp: False
dumpTable: False
getColumns: False
batch: True
purge: False
headers: None
authCred: None
osCmd: None
suffix: None
dbmsCred: None
regDel: False
chunked: False
sitemapUrl: None
timeSec: 5
msfPath: None
dumpAll: False
fileRead: None
getHostname: False
sessionFile: None
disableColoring: True
getTables: False
listTampers: False
agent: None
webRoot: None
exclude: None
lastChar: None
string: None
dbms: None
dumpWhere: None
tamper: None
ignoreRedirects: False
hpp: False
runCase: None
delay: 0
evalCode: None
cleanup: False
csrfUrl: None
secondUrl: None
getBanner: False
profile: False
regRead: False
bulkFile: None
db: None
dumpFormat: CSV
alert: None
harFile: None
nullConnection: False
user: None
parseErrors: False
getCount: False
data: None
regAdd: False
ignoreProxy: False
database: /tmp/sqlmapipc-lI97N8
mobile: False
googleDork: None
saveConfig: None
sqlShell: False
tech: BEUSTQ
textOnly: False
cookieDel: None
commonColumns: False
keepAlive: False
```

#### 总结 <a href="#h3-26" id="h3-26"></a>

基于HTTP的接口模式用起来可能比较繁琐，但是对于程序调用接口还是很友善的。总之该模式的流程是：1、通过GET请求 http://ip:port/task/new 这个地址，创建一个新的扫描任务；\
2、通过POST请求 http://ip:port/scan//start 地址，并通过json格式提交参数，开启一个扫描；通过GET请求 http://ip:port/scan//status 地址，即可获取指定的taskid的扫描状态。这个返回值分为两种，一种是run状态（扫描未完成），一种是terminated状态（扫描完成）；\
3、扫描完成后获取扫描的结果。

## 使用Python3编写sqlmapapi调用程序

下面就来编写一个sqlmapapi调用程序，首先我们得再次明确一下流程：

1、通过 sqlmapapi.py -s -H "0.0.0.0" 开启sqlmap api的服务端。服务端启动后，在服务端命令行中会返回一个随机的admin token值，这个token值用于管理taskid（获取、清空操作），在这个流程中不需要amin token这个值，可以忽略。之后，服务端会处于一个等待客户端的状态。\
2、通过GET请求 http://ip:port/task/new 这个地址，即可创建一个新的扫描任务，在响应中会返回一个随机的taskid。这个taskid在这个流程中尤为重要，因此需要通过变量存储下来，方便后面程序的调用。\
3、通过POST请求 http://ip:port/scan//start 地址，并通过json格式提交参数(待扫描的HTTP数据包、若存在注入是否获取当前数据库用户名)，即可开启一个扫描任务，该请求会返回一个enginedid。\
4、通过GET请求 http://ip:port/scan//status 地址，即可获取指定的taskid的扫描状态。这个返回值分为两种，一种是run状态（扫描未完成），一种是terminated状态（扫描完成）。\
5、判断扫描状态，如果扫描未完成，再次请求 http://ip:port/scan//status 地址 ，直到扫描完成。\
6、扫描完成后获取扫描的结果，是否是SQL注入，若不存在SQL注入，data字段为空，若存在SQL注入，则会返回数据库类型、payload等等。

明确了流程后，为了可维护性好和main.py文件代码量少，笔者首先是写了一个类，代码如下：

```
#!/usr/bin/python
# -*- coding:utf-8 -*-
# wirter:En_dust
import requests
import json
import time

class Client():
    def __init__(self,server_ip,server_port,admin_token="",taskid="",filepath=None):
        self.server = "http://" + server_ip + ":" + server_port
        self.admin_token = admin_token
        self.taskid = taskid
        self.filepath = ""
        self.status = ""
        self.scan_start_time = ""
        self.scan_end_time = ""
        self.engineid=""
        self.headers = {'Content-Type': 'application/json'}



    def create_new_task(self):
        '''创建一个新的任务，创建成功返回taskid'''
        r = requests.get("%s/task/new"%(self.server))
        self.taskid = r.json()['taskid']
        if self.taskid != "":
            return self.taskid
        else:
            print("创建任务失败!")
            return None

    def set_task_options(self,url):
        '''设置任务扫描的url等'''
        self.filepath = url



    def start_target_scan(self,url):
        '''开始扫描的方法,成功开启扫描返回True，开始扫描失败返回False'''
        r = requests.post(self.server + '/scan/' + self.taskid + '/start',
                      data=json.dumps({'url':url,'getCurrentUser':True,'getBanner':True,'getCurrentDb':True}),
                      headers=self.headers)
        if r.json()['success']:
            self.scan_start_time = time.time()
            #print(r.json())
            #print(r.json()['engineid'])
            return r.json()['engineid']
        else:
            #print(r.json())
            return None

    def get_scan_status(self):
        '''获取扫描状态的方法,扫描完成返回True，正在扫描返回False'''
        self.status = json.loads(requests.get(self.server + '/scan/' + self.taskid + '/status').text)['status']
        if self.status == 'terminated':
            self.scan_end_time = time.time()
            #print("扫描完成!")
            return True
        elif self.status == 'running':
            #print("Running")
            return False
        else:
            #print("未知错误！")
            self.status = False



    def get_result(self):
        '''获取扫描结果的方法，存在SQL注入返回payload和注入类型等，不存在SQL注入返回空'''
        if(self.status):
            r = requests.get(self.server + '/scan/' + self.taskid + '/data')
            if (r.json()['data']):
                return r.json()['data']
            else:
                return None

    def get_all_task_list(self):
        '''获取所有任务列表'''
        r = requests.get(self.server + '/admin/' + self.admin_token + "/list")
        if r.json()['success']:
            #print(r.json()['tasks'])
            return r.json()['tasks']
        else:
            return None

    def del_a_task(self,taskid):
        '''删除一个任务'''
        r = requests.get(self.server + '/task/' + taskid + '/delete')
        if r.json()['success']:
            return True
        else:
            return False

    def stop_a_scan(self,taskid):
        '''停止一个扫描任务'''
        r = requests.get(self.server + '/scan/' + taskid + '/stop')
        if r.json()['success']:
            return True
        else:
            return False

    def flush_all_tasks(self):
        '''清空所有任务'''
        r =requests.get(self.server + '/admin/' + self.admin_token + "/flush")
        if r.json()['success']:
            return True
        else:
            return False

    def get_scan_log(self):
        '''获取log'''
        r = requests.get(self.server + '/scan/' + self.taskid + '/log')
        return r.json()
```

mian.py

```
#!/usr/bin/python
# -*- coding:utf-8 -*-
# wirter:En_dust
from Service import Client
import time
from threading import Thread

def main():
    '''实例化Client对象时需要传递sqlmap api 服务端的ip、port、admin_token和HTTP包的绝对路径'''
    print("————————————————Start Working！—————————————————")
    target = input("url:")
    task1 = Thread(target=set_start_get_result,args=(target,))
    task1.start()



def time_deal(mytime):
     first_deal_time = time.localtime(mytime)
     second_deal_time = time.strftime("%Y-%m-%d %H:%M:%S", first_deal_time)
     return  second_deal_time


def set_start_get_result(url):
    #/home/cheng/Desktop/sqldump/1.txt
    current_taskid =  my_scan.create_new_task()
    print("taskid: " + str(current_taskid))
    my_scan.set_task_options(url=url)
    print("扫描id:" + str(my_scan.start_target_scan(url=url)))
    print("扫描开始时间：" + str(time_deal(my_scan.scan_start_time)))
    while True:
        if my_scan.get_scan_status() == True:
            print(my_scan.get_result())
            print("当前数据库:" + str(my_scan.get_result()[-1]['value']))
            print("当前数据库用户名:" + str(my_scan.get_result()[-2]['value']))
            print("数据库版本:" + str(my_scan.get_result()[-3]['value']))
            print("扫描结束时间：" + str(time_deal(my_scan.scan_end_time)))
            print("扫描日志：\n" + str(my_scan.get_scan_log()))
            break




if __name__ == '__main__':
    my_scan = Client("127.0.0.1", "8775", "c88927c30abb1ef6ea78cb81ac7ac6b0")
    main()
```

Githun地址：[https://github.com/FiveAourThe/sqlmap\_api\_demo](https://github.com/FiveAourThe/sqlmap\_api\_demo)
