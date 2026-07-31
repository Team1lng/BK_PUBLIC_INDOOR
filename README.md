# two-wire-indoor

## 20260722
1. Tuya 6 门锁 DP 映射与状态上报修正
   问题：原移植代码仍沿用 Tuya 4 的 DP 定义，导致手机端门锁能力与实际设备不一致；门锁/门禁 DP 下发 `false` 时也会进入开锁处理。
   涉及文件：`src/tuya-6.2.6/app/app_main/include/ty_dp_define.h`、`src/tuya-6.2.6/app/app_main/src/ty_sdk_dp_callback.c`、`src/tuya-6.2.6/tuya_sdk.h`、`src/layout/layout_define.c`、`src/layout/layout_home.c`、`src/layout/layout_monitor.c`、`src/layout/layout_standby.c`。
   修改内容：统一 DP 定义：`232` 为门口机 `gate1`、`234` 为门口机主锁、`244` 为室内机 `gate2`；多锁支持 DP `233` 上报 `234,232,244`；初始化、UI 操作和延时复位均按新 DP 回报状态；只在收到 `true` 时执行开锁/开门，避免状态复位触发重复动作。

2. Tuya OTA 支持室内机与门口机分流升级
   问题：原 OTA 下载完成后统一按室内机 APP 包解压升级，不能将门口机固件交由两线协议下发。
   涉及文件：`src/tuya-6.2.6/app/app_user_upgrade/src/ty_sdk_upgrade.c`、`src/tuya-6.2.6/tuya_sdk.h`、`src/layout/layout_define.c`、`src/layout/layout_monitor.c`、`src/layout/layout_standby.c`、`src/layout/layout_setting_senior.c`。
   修改内容：室内机固件继续关闭看门狗、保留 Wi-Fi 配置、解压并执行 `/tmp/update.sh`；门口机固件下载后保存为 `/mnt/tf/cbin.update`，通过 `TUYA_EVENT_OTA_UPGRADE` 切换到高级设置页并启动门口机升级流程；升级进度改为按固件类型上报，监控/待机页面收到事件时先停止相应视频和录像任务。

3. Wi-Fi“忘记网络”改为禁用自动重连且保留密码缓存
   问题：仅把 `wifi_connect_flag` 设为 `false` 或仅执行 `disconnect`，设备重启后仍可能使用已有网络配置自动连接。
   涉及文件：`src/api/wlan/wlan.c`、`src/layout/layout_setting_wifi.c`。
   修改内容：忘记网络时关闭 WLAN 自动连接标志、停止 `udhcpc`、执行 `wpa_cli -i wlan0 disable_network all`、保存配置并断开连接，同时调用 `user_data_save()` 保存状态；不删除 `wpa_supplicant.conf` 中的密码缓存，后续可重新启用/连接网络。

4. 设置页增加 Tuya PID 显示
   问题：信息页无法直接确认当前固件编译使用的 Tuya PID，移植和产测时不便排查 PID 与云端产品不一致的问题。
   涉及文件：`src/layout/layout_setting_info.c`、`src/layout/layout_define.h`、`src/layout/layout_define.c`。
   修改内容：在 SD 卡容量和 Tuya UUID 之间增加 PID 信息项，显示编译宏 `IPC_APP_PID`；新增 `STR_TUYA_PID` 并按枚举序号从打包的 `language.xls` 读取文本，与 `palvision` 的语言映射机制保持一致。当前工程 PID 配置为 `be5lae0ogehtzyg9`。

5. 移除公共版本高级设置中的 App Unlock 选择 UI
   问题：公共版本显示的 App Unlock 设置会切换旧的 `lock_id`，与新的 Tuya 锁 DP 方案不一致，且容易造成配置歧义。
   涉及文件：`src/layout/layout_setting_senior.c`。
   修改内容：公共版本不再创建 App Unlock 设置项；非公共版本仍保留原有模式设置入口。

6. Tuya 音视频与运行稳定性修复
   问题：网络音频接收路径错误调用视频上报接口；H.264 NAL 类型判断受运算符优先级影响；Wi-Fi 健康检查和看门狗缺少有效的失败诊断。
   涉及文件：`src/api/network/network_audio_package.c`、`src/tuya-6.2.6/tuya_sdk.c`、`src/api/network/ping_host.c`、`src/layout/user_data.c`。
   修改内容：PCM 数据改为调用 `tuya_realtime_audio_put_frame()`；修正 NAL 类型表达式括号，正确区分 P/B 帧和 I 帧；减少 Wi-Fi 断开日志刷屏，并记录看门狗打开/喂狗失败以及 Wi-Fi 模组重载的开始、结束状态，便于定位异常重启和监控无图问题。


#### 介绍
博科两线室内机

#### 软件架构
软件架构说明


#### 安装教程

1.  xxxx
2.  xxxx
3.  xxxx

#### 使用说明

1.  xxxx
2.  xxxx
3.  xxxx

#### 参与贡献

1.  Fork 本仓库
2.  新建 Feat_xxx 分支
3.  提交代码
4.  新建 Pull Request


#### 特技

1.  使用 Readme\_XXX.md 来支持不同的语言，例如 Readme\_en.md, Readme\_zh.md
2.  Gitee 官方博客 [blog.gitee.com](https://blog.gitee.com)
3.  你可以 [https://gitee.com/explore](https://gitee.com/explore) 这个地址来了解 Gitee 上的优秀开源项目
4.  [GVP](https://gitee.com/gvp) 全称是 Gitee 最有价值开源项目，是综合评定出的优秀开源项目
5.  Gitee 官方提供的使用手册 [https://gitee.com/help](https://gitee.com/help)
6.  Gitee 封面人物是一档用来展示 Gitee 会员风采的栏目 [https://gitee.com/gitee-stars/](https://gitee.com/gitee-stars/)

## 20260507
1.支持更换自定义PID

## 20250701
1.同步APP开锁设置不保存问题

## 20250625
1.同步wifi列表消失问题
2.同步tuya二维码不刷新问题

## 20241223
1.优化wifi连接卡顿问题，具体原因三调用system时未后台执行，导致需要等待命令结束才会回到主进程中，导致卡顿，且直接system调用字符串常亮可能导致命令后面的&未执行，导致阻塞 
2.优化wifi配置文件wpa_supplicant丢失问题，etc/config目录下的wpa_supplicant文件只做拷贝覆盖，不直接删除，间接避免删除后拷贝新的配置文件失败，导致文件丢失从而wifi无法搜索 
3.修复涂鸦CCTV切换卡住，原因:切换cctv过程中线程未来得及关闭，导致新开的码流失败返回；修改方式:线程做阻塞返回，不适用分离接口,每次执行monitor_switch都重新打开解码器，避免脏视频数据导致解码失败死机 
4.修复涂鸦OTA升级下载后未执行升级操作; 
5.避免升级过程被看门狗重启导致升级失败，OTA升级关闭看门狗；
6.修复涂鸦ID切换过程中重启，涂鸦ID切换前关闭看门狗，切换完成重新打开 
7.优化待机息屏时，屏幕未完全刷黑屏,关闭息屏时右上提示小图标刷新，进入home界面后优先点亮背光 
8.将msg_task函数中，对wifi连接状态的判断更改为wifi_connection_status_sucess() ;重构wifi_connection_status_sucess函数，添加获取超时退出机制
9.将看门狗移植至用户数据处理线程，避免RTSP流导致主线程阻塞从而导致看门狗重启
10.修复视频播放频繁切换后播放异常，不可重新进入播放状态，播放按键无效果，根源：视频播放线程频繁销毁，导致异常，线程出错；解决：将线程销毁更改成阻塞释放
11.优化涂鸦id切换卡顿过长，将fopen涂鸦id缓存文件由a打开方式更换成w;xls表格库在识别xls文件时可能存在解析错误行数，导致创建过多无效的数据，添加后续切换解析ini文件负担；解决方案：判断每行数据是否有效，若存在无效数据则不更新至ini配置文件
12.修复视频播放界面快速切换后导致播放异常，将播放音视频线程改成阻塞释放
13.添加上电运行准备工作脚本Preparation.sh，若上电SD卡中存在该脚本，则运行该脚本
14.修复关闭移动侦测预览状态下，移动侦测录像时进入移动侦测录制文件列表界面，存在偶尔丢失当前录制文件问题，其根源是不同线程对文件操作函数接口的调用，同时访问操作一个缓存数组，导致冲突，在其数据修改后，因其他线程操作导致旧数据重新覆盖掉新数据，因此丢失当前的录制文件；目前修改方式：在进行扫描SD卡文件时，添加判断是否进行视频录制中，当进行录制则等待录制完毕，并优化相关UI显示逻辑

## 20241225
1.添加ADC按键检测，支持兼容两个按键功能

## 20250102
1.修复SD卡分区内存不足时，清除内存操作异常，导致循环进入清除操作，占用大量CPU资源，其原因是代码函数SD_card_space_clear在做清除数据类型判断时，未添加else，导致在某个判断成立后，指定删除的类型未退出判断代码，进行下一步的其他类型选择；解决方案：在每次判断选择清除数据的类型后添加else，避免if过后继续执行下一段代码

## 20250106
1.优化涂鸦id切换tuya_xls_head_valid_check接口，判断第一次检测到该行中uuid或key数据为空时，修改有效行数，避免解析出错误的行数导致切换id时返回文件错误等异常；及在tuya_uuid_and_key_swtch函数中添加对tuya_xls_head_valid_check函数的调用

## 20250117
1.修复软重启屏幕显示异常
2.视频解码修改为join阻塞关闭，避免detach分离关闭未及时释放导致下次开启解码异常

## 20250301
1.添加wifi扫描热点异常处理，添加至用户数据进程检测
