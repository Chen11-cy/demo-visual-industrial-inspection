# M2 板卡正面装配检测

设备端视觉检查 Demo：真实 OCR 确认 M2 后，检测 **12 个接口和 4 根铜柱**，支持单模块复检、离线语音、单板表格和批次良品率统计。全部16项通过立即结算，未全通过最长20秒。

## 下载与运行

支持 Linux aarch64 / x86_64、Python 3.10–3.13；目标设备基线为 M2 Debian 13。安装时需要访问 PyPI，安装完成后可离线检测。

```bash
git clone https://github.com/Quectel-Pi/demo-visual-industrial-inspection.git
cd demo-visual-industrial-inspection
./install.sh
./run.sh
```

在**运行程序的机器**打开 http://127.0.0.1:8765/。默认自动选择唯一的 USB 相机，不会盲选 ISP 节点；多摄像头时指定：

```bash
./run.sh --camera /dev/v4l/by-id/usb-YOUR_CAMERA-video-index0
# 没有相机也可以体验内置样例：
./run.sh --image-only
```

如缺少基础环境，先由管理员安装：

```bash
sudo apt-get update
sudo apt-get install python3 python3-venv libglib2.0-0 libgl1
# 需要语音时另装：
sudo apt-get install pulseaudio-utils
```

建议用桌面登录用户运行，确保具有相机访问权限。禁止用 sudo pip 修改设备系统 Python。Python 3.13 下 RapidOCR 1.4.4 的元数据兼容例外和安装方式见英文 README。

## 操作

1. 放入 M2 板卡，型号丝印、全部接口和柱头应处于画面内。
2. 等待真实型号确认。全部16项通过立即保存并播报“合格”；超时未全通过则保存未通过结果并短滴提示。
3. 点击无法检测或未通过的模块行，系统取三张请求后的新帧，只复检该项。USB上组件与VRTC分别使用3秒内两次正常确认，不互相计票。
4. 结算后移出板卡约3秒，再放入下一块。同一在场板卡不重复计数。
5. 导出整板表、复检表或批次表。首次与复检后通过率分别保留，复检不增加板数。

仓库自带正常参考照片、示例工位标定、英文 OCR 后备模型和语音文件。**示例标定不是所有相机、取景位置通用的验收标准**；新工位应先用确认完好的完整板重新标定。

运行数据默认位于 `data/`，首次启动复制示例标定到其中，后续不会覆盖操作者修改。`data/output/` 保存图片和JSON，`data/production.sqlite3` 保存台账。可用 `--data-dir` 或 `INSPECTION_DATA_DIR` 指定其他位置；升级前备份整个数据目录。

## 验证与边界

```bash
.venv/bin/python -m unittest -q test_engine test_decisions test_live
```

普通正常外观门槛65%，同时保留图像质量、时间确认和缺陷优先判断。无法检测不等于缺失；只有对应真实缺失/损坏外观已标定时才输出该类别。尚不支持接线判断、电气功能检测或实物缺针/弯针的独立验收。软件测试通过不等于工业缺陷精度达标。

服务默认仅监听本机，属于无认证本地 Demo，不应直接暴露公网。图片、OCR和报表在本地处理。详细维护见 [docs/DEPLOYMENT.md](docs/DEPLOYMENT.md)，第三方许可见 [THIRD_PARTY.md](THIRD_PARTY.md)。