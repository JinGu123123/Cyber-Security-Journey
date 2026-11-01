# DVWA - Command Injection 实战记录

## 1. 目标信息
- **靶场**: DVWA (Damn Vulnerable Web App) - Security Level: Low
- **环境**: Docker (vulnerables/web-dvwa)
- **目标 IP**: 192.168.184.129

## 2. 漏洞发现与利用过程
- **测试功能**: 页面提供 `Ping a device` 功能。
- **正常输入**: `127.0.0.1`，回显正常 ping 结果。
- **注入 Payload 1**: `127.0.0.1; ls -la`
  - **回显**: 成功列出服务器当前目录下的文件（`index.php` 等）。
  - ![Payload1](DVWA-Command-Injection/images/Payload1.png)
- **注入 Payload 2**: `127.0.0.1; whoami`
  - **回显**: `www-data`（确认当前 Web 服务运行权限）。
  - ![Payload2](DVWA-Command-Injection/images/Payload2.png)
- **注入 Payload 3**: `127.0.0.1; cat /etc/passwd`
  - **回显**: 成功读取系统所有用户信息（包括 root、mysql 等）。
  - ![Payload3](DVWA-Command-Injection/images/Payload3.png)

## 3. 漏洞原理
- **后端代码逻辑（推测）**: `system("ping -c 4 " . $_POST['ip'])`
- **成因**: 未对用户输入做过滤，直接将其拼接到系统命令后。
- **利用方式**: Linux 中的命令分隔符（如 `;`、`&`、`|`）可以截断原命令，执行我们注入的恶意命令，从而获得 RCE（远程命令执行）权限。

## 4. 修复建议
1. **输入校验**：使用正则表达式（如 `^\d{1,3}(\.\d{1,3}){3}$`）严格限制输入必须为合法的 IP。
2. **安全函数**：在 PHP 中使用 `escapeshellarg()` 对参数进行转义，或者使用 `proc_open` 等更底层的函数，避免 shell 解析。

## 5. 心得感悟
这次实战让我直观感受到了“拼接字符串”的危害。在前端看起来只是个普通的 Ping 输入框，但在后端却成了黑客突破系统的入口。以后自己写代码，必须对用户的每一个输入保持敬畏。
