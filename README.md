# trojan
![](https://img.shields.io/github/v/release/Jrohy/trojan.svg) 
![](https://img.shields.io/docker/pulls/jrohy/trojan.svg)
[![Go Report Card](https://goreportcard.com/badge/github.com/Jrohy/trojan)](https://goreportcard.com/report/github.com/Jrohy/trojan)
[![Downloads](https://img.shields.io/github/downloads/Jrohy/trojan/total.svg)](https://img.shields.io/github/downloads/Jrohy/trojan/total.svg)
[![License](https://img.shields.io/badge/license-GPL%20V3-blue.svg?longCache=true)](https://www.gnu.org/licenses/gpl-3.0.en.html)


trojan多用户管理部署程序

## 功能
- 在线web页面和命令行两种方式管理trojan多用户
- 启动 / 停止 / 重启 trojan 服务端
- 支持流量统计和流量限制
- 命令行模式管理, 支持命令补全
- 集成acme.sh证书申请
- 生成客户端配置文件
- 在线实时查看trojan日志
- 在线trojan和trojan-go随时切换
- 支持trojan://分享链接和二维码分享(仅限web页面)
- 支持转化为clash订阅地址并导入到[clash_for_windows](https://github.com/Fndroid/clash_for_windows_pkg/releases)(仅限web页面)
- 限制用户使用期限

## 安装方式
*trojan使用请提前准备好服务器可用的域名*  

###  a. 一键脚本安装
```
#安装/更新
source <(curl -sL https://raw.githubusercontent.com/john-captain/trojan/master/install.sh)

#卸载
source <(curl -sL https://raw.githubusercontent.com/john-captain/trojan/master/install.sh) --remove

```
安装完后输入'trojan'可进入管理程序   
浏览器访问 https://域名 可在线web页面管理trojan用户  
前端页面源码地址: [trojan-web](https://github.com/Jrohy/trojan-web)

### b. docker运行
1. 安装mysql  

因为mariadb内存使用比mysql至少减少一半, 所以推荐使用mariadb数据库
```
docker run --name trojan-mariadb --restart=always -p 3306:3306 -v /home/mariadb:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=trojan -e MYSQL_ROOT_HOST=% -e MYSQL_DATABASE=trojan -d mariadb:10.2
```
端口和root密码以及持久化目录都可以改成其他的

2. 安装trojan
```
docker run -it -d --name trojan --net=host --restart=always --privileged jrohy/trojan init
```
运行完后进入容器 `docker exec -it trojan bash`, 然后输入'trojan'即可进行初始化安装   

启动web服务: `systemctl start trojan-web`   

设置自启动: `systemctl enable trojan-web`

更新管理程序: `source <(curl -sL https://raw.githubusercontent.com/john-captain/trojan/master/install.sh)`

## 安全修复
本仓库修复了 **CVE-2024-55215** 漏洞（未授权修改管理员密码）

### 漏洞详情
原项目的 `/auth/register` 接口在管理员账户创建后仍然可以被未授权用户访问，导致攻击者可以直接修改管理员密码。

### 修复方案
修改 `web/auth.go` 文件，在 `/auth/register` 接口中添加检查逻辑：
- 只有当管理员密码**未设置**时（首次安装），才允许调用该接口
- 如果管理员密码已存在，返回 `403 Forbidden` 拒绝请求

### 修复后的行为
| 场景 | 修复前 | 修复后 |
|------|--------|--------|
| 首次安装，设置管理员密码 | ✅ 允许 | ✅ 允许 |
| 管理员已存在，未登录调用 `/auth/register` | ❌ 允许（漏洞） | ✅ 返回 403 拒绝 |
| 已登录用户调用 `/auth/reset_pass` | ✅ 允许 | ✅ 允许 |

### 参考
- 漏洞 POC：[ainrm/Jrohy-trojan-unauth-poc](https://github.com/ainrm/Jrohy-trojan-unauth-poc)
- 影响版本：v2.0.0 - v2.15.3

## 运行截图
![avatar](asset/1.png)
![avatar](asset/2.png)

## 命令行
```
Usage:
  trojan [flags]
  trojan [command]

Available Commands:
  add           添加用户
  clean         清空指定用户流量
  completion    自动命令补全(支持bash和zsh)
  del           删除用户
  help          Help about any command
  info          用户信息列表
  log           查看trojan日志
  port          修改trojan端口
  restart       重启trojan
  start         启动trojan
  status        查看trojan状态
  stop          停止trojan
  tls           证书安装
  update        更新trojan
  updateWeb     更新trojan管理程序
  version       显示版本号
  import [path] 导入sql文件
  export [path] 导出sql文件
  web           以web方式启动

Flags:
  -h, --help   help for trojan
```

## 注意
安装完trojan后强烈建议开启BBR等加速: [one_click_script](https://github.com/jinwyp/one_click_script)  

## Thanks
感谢JetBrains提供的免费GoLand  
[![avatar](asset/jetbrains.svg)](https://jb.gg/OpenSource)
