# 所有实验都在该文件夹下运行即可，已放上测试用的代码。免去配置 adb 的麻烦

- 复制 `wechat_jump_game` 根目录下的 `config` 文件夹以及 `wechat_jump_py3.py` 文件到本目录下 (如果需要自动运行则额外复制 `common`)
- 按住 `shift` + 右键  选择在该文件夹下打开命令窗口（Windows 用户请自行 cmd）
- 打开安卓手机的 usb 调试，并连接电脑，在终端输入 `adb devices` 进行测试，如果有连接设备号则表示成功
- 打开微信小游戏，然后运行代码 `python wechat_jump_py3.py`，点击出现的图形起点和终点，棋子自动跳转

**注意：这里使用的是不需要配置的 adb 方式，需要在该文件下操作，至于如何自动跳转，只需改变执行脚本即可，这里只做演示**


# 解决 `ModuleNotFoundError: No module named 'matplotlib'`

本文记录在 Windows 上运行 `wangshub/wechat_jump_game` 项目时，如何解决缺少 Matplotlib 依赖的问题。

## 1. 问题现象

在项目的 `Tools` 目录运行：

```bat
python wechat_jump_py3.py
```

出现错误：

```text
Traceback (most recent call last):
  File "E:\下载\Vivaldi\wechat_jump_game-master\Tools\wechat_jump_py3.py", line 5, in <module>
    import matplotlib.pyplot as plt
ModuleNotFoundError: No module named 'matplotlib'
```

原因是当前 Python 环境中没有安装 `matplotlib`。

## 2. 为什么直接安装 `requirements.txt` 会失败

项目根目录提供了 `requirements.txt`，其中固定了较旧的版本：

```text
matplotlib==2.1.1
```

执行：

```bat
pip install -r requirements.txt
```

安装过程中，`pip` 下载了 `matplotlib 2.1.1` 的源码包并尝试在本机编译，最终失败。日志中指出缺少 FreeType 和 libpng 的开发文件。

当时的环境为：

- Windows 64 位
- Python 3.10.0
- NumPy 1.26.4

因此，这不是简单的网络下载失败，而是旧版 Matplotlib 在当前环境中无法顺利完成源码构建。

## 3. 解决方法：安装新版 Matplotlib

不要先安装项目锁定的旧版 `matplotlib==2.1.1`，改为安装兼容当前 Python 环境的新版本。

在 CMD 中执行：

```bat
python -m pip install "matplotlib>=3.8,<4"
```

这里使用 `python -m pip`，是为了尽可能确保安装到当前 `python` 命令对应的环境中。

安装后验证：

```bat
python -c "import matplotlib.pyplot as plt; print(plt.matplotlib.__version__)"
```

如果命令输出 Matplotlib 版本号且没有报错，就说明安装成功。

## 4. 安装脚本可能需要的其他依赖

如果后续提示缺少 OpenCV、Pillow 或 NumPy，可以执行：

```bat
python -m pip install opencv-python pillow numpy
```

注意：Python 代码中导入 OpenCV 时使用的模块名是 `cv2`，但 pip 包名是 `opencv-python`。

如果项目的 `requirements.txt` 仍然固定了 `matplotlib==2.1.1`，不要在完成上述处理后再次直接执行：

```bat
pip install -r requirements.txt
```

否则 pip 可能再次尝试安装旧版 Matplotlib。更稳妥的长期做法是为当前 Python 环境维护一份经过验证的依赖清单。

## 5. 重新运行项目

进入脚本目录：

```bat
cd /d E:\下载\Vivaldi\wechat_jump_game-master\Tools
```

运行：

```bat
python wechat_jump_py3.py
```

本次处理后，脚本成功启动并弹出了 Matplotlib 图形窗口，能够显示手机截图。

## 6. 其他警告说明

启动时可能看到类似警告：

```text
UserWarning: frames=None which we can infer the length of ...
```

这是旧脚本的 Matplotlib 动画参数与新版库之间的兼容性警告。只要窗口正常显示，它通常不是启动失败的原因，可以先不处理。

也可能看到 ADB 关于多个显示屏的警告：

```text
[Warning] Multiple displays were found, but no display id was specified!
```

这与 Matplotlib 缺失无关。如果截图内容正确，可以先忽略；如果截取了错误的屏幕，再进一步检查 Android 显示屏 ID。

## 7. 总结

本次问题的解决步骤：

1. 确认报错是当前 Python 环境缺少 `matplotlib`。
2. 发现项目的 `requirements.txt` 固定使用旧版 `matplotlib==2.1.1`。
3. 旧版在 Windows + Python 3.10 环境下尝试源码构建，并因缺少 FreeType、libpng 开发文件而失败。
4. 使用以下命令安装新版 Matplotlib：

   ```bat
   python -m pip install "matplotlib>=3.8,<4"
   ```

5. 验证导入成功，再运行 `wechat_jump_py3.py`。

**关键点：** 对这个旧项目，不要在未评估兼容性的情况下原样安装全部旧依赖；先解决当前缺失模块，并在运行过程中根据实际错误逐项处理。

