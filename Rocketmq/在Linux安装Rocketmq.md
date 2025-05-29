# 1. 先安装jdk



![image-20250528130900772](image-20250528130900772.png)

用这个命令解压Java

```
export JAVA_HOME=/app/java/jdk8u452-b09

export JRE_HOME=/app/java/jdk8u452-b09/jre

export ROCKETMQ_HOME=/app/Rocketmq/rocketmq-all-5.3.1-bin-release

export CLASSPATH=$CLASSPATH:$JAVA_HOME/lib:$JAVA_HOME/jre/lib

export PATH=$JAVA_HOME/bin:$JAVA_HOME/jre/bin:$ROCKETMQ_HOME/bin:$PATH:$HOME/bin

export NAMESRV_ADDR=192.168.70.132:9876
```

在/etc/profile中配置环境

java压缩包在这里E:\edgedownload

# 2.在安装rocketmq

正常解压然后配置环境变量

# 3. 然后启动NameServer

/app/Rocketmq/rocketmq-all-5.3.1-bin-release/bin 在这个目录中启动



```
nohup ./mqnamesrv -n 192.168.70.132:9876 &
```



# 4.然后启动broker

```
nohup ./mqbroker -n 192.168.70.132:9876 &
```



验证消息的发送

```
./tools.sh org.apache.rocketmq.example.quickstart.Producer
```

验证消息的接收

```
 ./tools.sh org.apache.rocketmq.example.quickstart.Consumer
```



# 5. 停止Name server和broker

```
./mqshutdown broker

./mqshutdown namesrv
```

















# nohup命令

`nohup` 是 Linux/Unix 系统中的一个命令，全称 **No Hang Up**（不挂断），用于在终端关闭后仍然保持进程的运行。它通过忽略 `HUP`（挂断）信号，使进程不会因为终端（或 SSH 连接）关闭而终止。

---

### **核心作用**
- **让进程在后台持续运行**：即使你关闭终端或断开 SSH 连接，进程也不会终止。
- **避免进程依赖终端**：普通进程在终端关闭时会收到 `HUP` 信号并终止，而 `nohup` 会阻止这一行为。

---

### **基本语法**
```bash
nohup 命令 [参数] &
```
- `命令`：需要执行的程序或脚本（如 `python app.py`、`java -jar app.jar`）。
- `&`：将进程放到后台运行。
- `nohup` 会自动将进程的输出重定向到当前目录下的 `nohup.out` 文件中（除非另有指定）。

---

### **示例**
#### 1. 运行一个 Python 脚本并让其在后台持续运行
```bash
nohup python myscript.py &
```
- 输出会写入 `nohup.out` 文件。
- 即使关闭终端，脚本仍会继续执行。

#### 2. 指定输出文件
```bash
nohup java -jar myapp.jar > app.log 2>&1 &
```
- `> app.log`：将标准输出重定向到 `app.log`。
- `2>&1`：将标准错误输出重定向到标准输出（即也写入 `app.log`）。

---

### **关键特性**
| 特性              | 说明                                                       |
| ----------------- | ---------------------------------------------------------- |
| **忽略 HUP 信号** | 进程不会因终端关闭而终止。                                 |
| **输出重定向**    | 默认输出到 `nohup.out`（当前目录需有写权限）。             |
| **后台运行**      | 通常与 `&` 结合使用，将进程放到后台执行。                  |
| **跨会话持久化**  | 进程独立于当前终端会话，适合长期任务（如服务、定时任务）。 |

---

### **常用操作**
#### 1. 查看正在运行的 `nohup` 进程
```bash
ps aux | grep "myscript.py"
```
或根据进程名过滤：
```bash
ps -ef | grep java
```

#### 2. 终止进程
找到进程 ID（PID）后：
```bash
kill <PID>
```
强制终止：
```bash
kill -9 <PID>
```

#### 3. 查看输出日志
```bash
tail -f nohup.out
```
或查看自定义日志文件：
```bash
tail -f app.log
```

---

### **注意事项**
1. **当前目录权限问题**：
   - `nohup.out` 默认生成在当前目录，需确保当前目录有写权限。
   - 可通过重定向修改输出路径：
     ```bash
     nohup python script.py > /var/log/myscript.log 2>&1 &
     ```

2. **进程依赖环境变量**：
   - 如果进程依赖某些环境变量（如 `PATH`），建议在脚本中显式设置。

3. **资源限制**：
   - 长时间运行的进程可能受系统资源限制（如内存、CPU），需监控系统负载。

4. **安全性**：
   - 避免在 `nohup` 命令中直接包含敏感信息（如密码），日志文件可能暴露这些信息。

---

### **与 `&` 的区别**
| 特性         | `&`                      | `nohup`                    |
| ------------ | ------------------------ | -------------------------- |
| **作用**     | 将进程放到后台运行       | 让进程在终端关闭后继续运行 |
| **依赖终端** | 是（终端关闭则进程终止） | 否（终端关闭后进程仍在）   |
| **输出文件** | 默认输出到终端           | 默认输出到 `nohup.out`     |
| **典型用途** | 短时任务                 | 长时任务（如服务、批处理） |

---

### **替代方案**
- **`screen` 或 `tmux`**：终端复用工具，允许在多个会话间切换，断开连接后仍可恢复。
- **`systemd` 服务**：将进程注册为系统服务，开机自启并后台运行。
- **`disown`**：结合 `&` 使用，手动移除进程与终端的关联：
  ```bash
  python script.py &
  disown
  ```

---

### **总结**
- **使用场景**：需要长时间运行的任务（如 Web 服务、爬虫、定时脚本）。
- **核心命令**：
  ```bash
  nohup 命令 > 日志文件 2>&1 &
  ```
- **验证方法**：
  ```bash
  ps aux | grep 命令名
  tail -f 日志文件
  ```

如果需要更详细的示例或特定场景的用法，请告诉我！ 🚀
