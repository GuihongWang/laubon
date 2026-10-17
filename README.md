jb.marisa.ml — Carbon 离线包
========================

由 carbon.sep.lol 原版构建, 域名改写为 jb.marisa.ml。
使用原版是因为它的 host 槽位 14 字节, 能容纳 12 字符的 jb.marisa.ml
(修改版槽位仅 11 字节, 放不下)。

目录内容
--------
浏览器阶段
  index.html            入口页(已移除证书安装链接)
  wkloader.js           改写为从 PNG 载体读取
  loader8.png loader9.png patch.png jber.png   载荷(PNG 载体)
  wkloader.orig.js      原始 wkloader.js, 供对照

载荷阶段
  everuntether-jsc.tar               454 KB
  everuntether-bin.deb                 19 KB
  com.lukezgd.everuntether-migrator.deb 19 KB
  Cydia / tar / launchctl / truststore 约 21 MB

  后四个只有在 bootstrap 缺失时才会用到。设备上如果这四个路径都在,
  它们不会被下载, 可以从仓库里删掉省空间:
    /Applications/Cydia.app/Cydia
    /private/var/lib/dpkg/status
    /usr/share/terminfo/x/xterm-xi
    /usr/libexec/bigram

重要: 端口 80, 明文 HTTP
-----------------------
jber 里写死了端口 80 且不支持 TLS(只链接了 libSystem/CoreFoundation/
CFNetwork, 没有 Security.framework)。所以站点必须在 80 端口提供明文 HTTP。

GitHub Pages 满足这一点 —— 实测 https 域名的 http:// 80 端口直接返回 200,
不会 301 跳 https。但若在仓库设置里打开了 Enforce HTTPS, 就会变成 301,
那样阶段二会失败(裸 socket 不跟随重定向)。

如果开启 HTTPS 强制后测到 301, 把它关掉即可。

页面与部署
----------
手机连到与电脑相同的 WiFi(或任意能访问本站的网络), Safari 打开:
  http://jb.marisa.ml/

设备必须是 iOS 8.0-9.3.6 的 32 位设备(iPhone 4S/5/5c 及更早)。
64 位设备按钮恒为禁用。这是 tethered 越狱, 重启后失效, 需重跑。