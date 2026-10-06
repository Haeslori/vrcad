# vrcad
VM 替换:学生交接版 Fadenpendel + Freier Fall(2026-10-06)

目标:把 remotelabs-m.ostfalia.de 上原有的两个实验换成 Software_Uebergabe_2026-09-28 里的学生版本。

原则(来自学生交接 README)
01_Webanwendung 是自包含 webroot(deploy/ + start/ + Jie/ 共享平台), 不能拆开塞进 /experiments/<id>/。
稳定入口:deploy/fadenpendel.html 和 deploy/freierfall.html (不要链接会变的 index.html)。
旧的 /experiments/freifall/ 原样保留作为回退,不删。
已完成(本机,无需再做)
交接包完整性校验:1100 文件,0 偏差(dateien-pruefen.mjs)。
newclaude/lab-hub/js/config/experimentRegistry.js:
freifall → 生产环境指向 /uebergabe/deploy/freierfall.html
新增 fadenpendel 条目 → /uebergabe/deploy/fadenpendel.html
本地开发不受影响(freifall 本地仍走 freifall-clean/deploy/)。
labChannels.js:新增 channel-10(Fadenpendel,station-fadenpendel, generic-messwert,不进自动轮播)。
要在 PowerShell 里执行(你的电脑 → VM)
powershell
cd C:\Users\id942606\Documents

# 1. 学生 webroot 上传(43 MB;先传成临时名再原子切换)
scp -r "release10\Software_Uebergabe_2026-09-28\01_Webanwendung" user@remotelabs-m.ostfalia.de:/var/www/remotelab/
ssh user@remotelabs-m.ostfalia.de "rm -rf /var/www/remotelab/uebergabe && mv /var/www/remotelab/01_Webanwendung /var/www/remotelab/uebergabe"

# 2. 注册表 + 频道表更新(只有这两个文件,场景无需重新 Package)
scp "newclaude\lab-hub\js\config\experimentRegistry.js" "newclaude\lab-hub\js\config\labChannels.js" user@remotelabs-m.ostfalia.de:/var/www/remotelab/lab-hub/js/config/

user 换成你的 VM 用户名(与 DEPLOY_GUIDE_CN.md 第 3 节一致)。 nginx 不需要改配置(静态子目录,HTTPS 已就位)。

上线验证
浏览器开 https://remotelabs-m.ostfalia.de/uebergabe/deploy/fadenpendel.html 和 .../freierfall.html → 两个场景直接能进。
开 Lobby:实验列表应出现 Fadenpendel(新)和 Freier Fall (指向新版本);各开一次确认加载的是学生版 (特征:运动方式默认"Gleiten"、Teleport 需左摇杆选择、 摆图表首页保留整段衰减曲线)。
回退:出问题时只需把 experimentRegistry.js 里 freifall 的 sceneUrl 改回 sceneUrlFor("freifall-clean", "freifall") 再传一次; 旧场景一直都在 /experiments/freifall/。
待确认(非阻塞)
学生场景与 Lab-Hub/Monitor 的房间参数(?room=...)和实时读数 集成未在本地验证过——上线后在 Monitor 里看一眼这两个站是否 正常出现;不影响学生直接做实验。
多人通信(PeerJS/TURN)按交接 README 属生产配置,学生的本地 检查不覆盖,需在 VM 域名下实测一次。
VR 头显复测(03_Pruefung/VR_KURZPRUEFUNG.md)学生标记为"未完成", 上线后建议在 Quest 上过一遍他列的 4 步检查。
