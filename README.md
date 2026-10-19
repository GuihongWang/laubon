jb.marisa.ml — Carbon 离线包(中文版)
================================

中文界面 + 进度显示, 不安装证书。域名改写为 jb.marisa.ml。

目录里有两个变体, 页面顶部有徽章标明当前是哪个, 互跳链接在按钮下方:

  /            原版 carbon.sep.lol 系 (8a7b36d)
  /cn/         修改版 jb.aptso.co 系 (93a641d)

中文界面只在 index.html 里, 与二进制无关, 所以两个二进制可以自由组合。

两个变体的差别
--------------
                       原版(根目录)        修改版(/cn/)
  提交                   8a7b36d             93a641d (落后 4 个)
  二进制                 106696 B            106660 B
  bootstrap 判定          4 个路径 stat()      dpkg/status 里找 Package: base
  bootstrap 检查路径      4 条                 1 条
  zebra 源 / uicache 修复 有                   无

原版检查的 4 条路径, 命中任意一条就跳过 21 MB 下载:
  /Applications/Cydia.app/Cydia
  /private/var/lib/dpkg/status
  /usr/share/terminfo/x/xterm-xi
  /usr/libexec/bigram

修改版只检查 /private/var/lib/dpkg/status, 所以每次都会重新下载完整的
Cydia bootstrap, 即使设备已经越狱过。

**根目录的 jber 打过补丁**

原版判断 bootstrap 是否可用只看 4 个文件在不在 (stat)。如果文件都在但
dpkg 数据库是空的或坏的, 原版会认为 bootstrap 完好 -> 整个跳过 strap ->
越狱失败。修改版则是 fopen(dpkg/status) 逐行找 "Package: base", 找不到就
判定数据库坏了 -> 执行 strap -> 成功。

根目录的 jber 把 0xc728 改成永远返回 1, 让原版和修改版行为一致:
  - 不再走 "skipping bootstrap strap" 早退
  - 4 个文件都在 -> 复用现有 bootstrap 文件, 不重新下 21 MB
  - 真正执行 "[*] strapping"

只改了 4 个字节, Mach-O 长度与 magic 不变。

为什么 bootstrap 放在根目录而不是 /cn/ 下
------------------------------------------
载荷按绝对路径取 bootstrap, 形如 http://<host>:80/Cydata, 前缀写死在
cstring 里。最长的那条 /com.lukezgd.everuntether-migrator.deb 占 38 字符、
槽位 39 字节, 加上 /cn 需要 42 字节 —— 塞不进去, 改写不了。

但两个变体的 7 个 bootstrap 文件逐字节相同(已用 SHA256 核对), 所以放在
根目录共用即可:

  页面用相对 URL 取载荷   wkloader.js 里写的是 "jber.png"
                          在 /cn/ 下自然解析成 /cn/jber.png
  载荷用绝对路径取 bootstrap  http://<host>:80/Cydia  ->  命中根目录共用文件

两条路径互不干扰, 实测请求记录:

  /  ->  /  /wkloader.js  /jber.png  /loader8.png  /loader9.png  /patch.png
  /cn/ -> /cn/  /cn/wkloader.js  /cn/jber.png  /cn/loader8.png
                 /cn/loader9.png  /cn/patch.png

host 槽位
---------
两版的 host 可用区实测都是 15 字节(最大域名 14 字符)。__cstring 里 host 和
端口串 "80" 之间有对齐填充, 所以 "80" 在 host 起点 +15 而不是紧挨着。

  原版    host @ 0x12f7c   端口串 @ 0x12f8b
  修改版  host @ 0x13003   端口串 @ 0x13012

早期文档写"修改版槽位只有 11 字节, 放不下 jb.marisa.ml", 那��是把域名长度
当成了槽位长度。patch-host.ps1 现在现测, 不再查表。

目录内容
--------
浏览器阶段(每个变体各一份)
  index.html            中文入口页, 四步进度条 + 详细日志面板
  wkloader.js           注入 prefetch 逻辑: 异步下载 PNG 载体 + 真实进度
  wkloader.orig.js      原始 wkloader.js, 供对照
  loader8.png loader9.png patch.png jber.png   载荷(PNG 载体)

载荷阶段(根目录, 两个变体共用)
  everuntether-jsc.tar               454 KB
  everuntether-bin.deb                 19 KB
  com.lukezgd.everuntether-migrator.deb 19 KB
  Cydia / tar / launchctl / truststore 约 21 MB

  后四个只有 bootstrap 缺失时才用到。上面 4 条路径都在就不会下载,
  可以删掉省空间。

进度显示的边界(重要)
--------------------
页面上能看到:
  下载载荷    真实字节进度, 每个文件的解码耗时
  利用漏洞    只有阶段提示; init() 前后会测出真实耗时并显示
  启动载荷    exec() 耗时

页面上**看不到**的:
  实际越狱进度。载荷已转为设备上的原生进程, 日志走 openlog/vsyslog
  进系统日志, 浏览器读不到。

  要看实时越狱进度, 另开终端连 SSH 后执行:
      log stream --style compact --predicate 'eventMessage CONTAINS "[*]"'

  利用阶段也加不了超时看门狗: 定时器在同步 JS 执行期间不触发, 而漏洞利用
  必须留在主线程(它操纵页面自身的堆对象), 无法搬到 Web Worker。

重要: 端口 80, 明文 HTTP
-----------------------
jber 里写死了端口 80 且不支持 TLS(只链接了 libSystem/CoreFoundation/
CFNetwork, 没有 Security.framework)。站点必须在 80 端口提供明文 HTTP。

GitHub Pages 实测满足 —— http:// 80 端口直接返回 200, 不 301 跳 https。
但若在仓库设置里打开 Enforce HTTPS, 就会变成 301 重定向, 而载荷用裸
socket 且不跟随重定向, 阶段二会静默失败。

自检(用载荷的方式发请求, 而不是浏览器的行为):
  http://jb.marisa.ml/Cydia                    应 200 + 19271680 字节, 不能是 301/404
  http://jb.marisa.ml/everuntether-jsc.tar     应 200 + 454144 字节

设备要求
--------
iOS 8.0-9.3.6 的 32 位设备(iPhone 4S/5/5c 及更早)。64 位按钮恒为禁用。
这是 tethered 越狱, 重启后失效, 需重跑。