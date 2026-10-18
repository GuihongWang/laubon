jb.marisa.ml — Carbon 离线包(中文版)
================================

中文界面 + 进度显示, 不安装证书。域名改写为 jb.marisa.ml。

二进制用的是 carbon.sep.lol **原版**(提交 8a7b36d), 不是中文站那版。原因:
  1. host 槽位 14 字节, 放得下 12 字符的 jb.marisa.ml(中文站那版只有 11 字节)
  2. 原版比中文站那版新 4 个提交 —— zebra 源、uicache 修复、
     need_to_strap 检查 4 个路径
中文界面只在 index.html 里, 与二进制无关, 所以两者可以自由组合。

目录内容
--------
浏览器阶段
  index.html            中文入口页, 含四步进度条和详细日志面板
  wkloader.js           注入 prefetch 逻辑: 异步下载 PNG 载体 + 真实进度
  loader8.png loader9.png patch.png jber.png   载荷(PNG 载体)
  wkloader.orig.js      原始 wkloader.js, 供对照

载荷阶段
  everuntether-jsc.tar               454 KB
  everuntether-bin.deb                 19 KB
  com.lukezgd.everuntether-migrator.deb 19 KB
  Cydia / tar / launchctl / truststore 约 21 MB

  后四个只有 bootstrap 缺失时才用到。设备上这四个路径都在就不会下载,
  可以删掉省空间:
    /Applications/Cydia.app/Cydia
    /private/var/lib/dpkg/status
    /usr/share/terminfo/x/xterm-xi
    /usr/libexec/bigram

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

  会看到类似 [*] downloading bootstrap / [*] strapping / [*] running uicache
  这些是载荷自己的日志。

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