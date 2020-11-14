# 1 安装
- pip install kivy
- [KV4Jetbrains](https://github.com/noembryo/KV4Jetbrains): Syntax highlighting and auto-completion for Kivy/KivyMD .kv files in PyCharm/Intellij IDEA
- vim 中用到的两个插件，[vim-glsl](https://github.com/tikhomirov/vim-glsl)和[vim-json](https://github.com/elzr/vim-json)，提供 JSON、OpenGL 语法高亮
- 打包apk工具：[Buildozer](https://buildozer.readthedocs.io/en/latest/)
- adb (Android Debug Bridge) 命令行调试手机  brew cask install android-platform-tools
- sdl2 ()DirectMedia Layer) 是一个跨平台开发库，旨在通过OpenGL和Direct3D提供对音频，键盘，鼠标，操纵杆和图形硬件的低层访问。 它被视频播放软件，模拟器和流行游戏使用，包括Valve屡获殊荣的目录和许多Humble Bundle游戏 brew info sdl2, brew install sdl2(brew install SDL2_ttf , brew install SDL2_image)
- 1 cd /Users/sunny/Android/tools/bin
- 2 ./sdkmanager --sdk_root=~/Android "cmdline-tools;latest"
- 3 cmdline-tools解压后得到的tools目录，要改名为latest,并且上级目录名是cmdline-tools，即：ANDROID_HOME = ~/Android/cmdline-tools/latest
- android.ndk_path = ~/Android/ndk-bundle/android-ndk-r19c
```
buildozer init  # 会产生buildozer.spec文件，可以进行编辑
buildozer -v android debug
```
## 关键词示例
控件（widget）、按钮（button)、复选框（checkbox）、标签（label）、输入框（textinput)
浮动容器（scrollable container）、盒式布局（boxlayout）
## App 类
App类是 Kivy 应用程序的基类，main entry point into the kivy run loop
大多数情况下都需要继承App类并实现自己的子类
覆盖 build() 初始化并返根控件（root widget）、kivy 将自动缩放根控件让它填满整个窗口
覆盖 build_config(self, config) 应用配置文件进行配置

## 页面布局
对于控件大小控制，一般用 size_hint 属性，取值 0-1之间，代表宽（或高）与当前窗口的宽（或高）的比例值
也可使用 size 属性，表示是一个固定值，不会因为设备而改变，此时 size_hint 必须设置为 [None, None]
位置指定：可以用 pos 指定；也可以用 pos_hint: {'x': 0.2, 'y': .6}    pos_hint:{'right': 0.8, 'top': .4}
pos_hint 是一个计算公式，窗口中任一点位置为: [y/x, x/L]
对于一个控件，x 轴上可以确定三条线，左边界x线、正中间的center_x线、右边界right线
对于一个控件，y 轴上可以确定三条线，上边界top线、正中间的center_y线、下边界y线
BoxLayout 布局中，间距分为两种：布局和子级之间填充需使用padding，默认【0，0，0，0】；子级与子级之间填充需spacing，默认0；padding又有四个参数，【padding_left, padding_top, padding_right, padding_bottom】,[padding_horizontal, padding_vertical]

# 2 Kivy Blueprints (book)
write one, run anywhere, the problem isn't exactly new, Java created by Sun in 1995 wanted too.
Kivy 是一个图形化用户界面库，便于轻松创建多平台 Python 应用程序；主要特性如下：
- 1 兼容新, 无论工作于什么系统，全部来自一个代码库
- 2 用户友好的接口，给不同的输入方法建立桥梁，允许使用相似的代码处理多种外设、鼠标事件、多点触控
- 3 快速的图像的硬件加速，基于OpenGL的渲染使得 Kivy 适合穿件图形化应用程序（如视频游戏），并通过平滑过渡改善用户体验
- 4 Kivy 本身采用 Python 编写，具有 Python 固有的可移植、可读性、表达性，Python 标准库和第三方软件（PyPI)
Kivy 本身也基于一些良好的第三方库，如Pygame，SDL，GStreamer，当然 Kivy 是更高层和统一的表达。（MIT）
## 1 运行第一个 Hello Kivy
Hello Kivy， 包含固定的两个文件，python 模块 .py 和 .ky 布局文件两部分
``` JavaScript
# main.py
# -*- coding:utf-8 -*-
from kivy.app import App
class HellApp(App):
    pass
if __name__ == '__main__':
    HellApp().run()
# hello.kv
Label:
    text: 'Hello Kivy'
```
- MyApp -> my.kv （或 Builder.load_file('path/to/file.kv')）
- HelloApp 的 Hello 做了标题(当然也可以通过self.title设置)，如果 .kv 文件名字不是 Hello 找不到，也就不显示
- Label 标签组件,内部还有一个文本
- 布局文件可以让组件的复杂组织变得简洁

Layouts Widgets 页面布局部件
AnchorLayout 锚点布局
BoxLayout 盒式布局
FloatLayout 浮动布局
GridLayout 网状布局
PageLayout 页状布局
RelativeLayout 相对布局
ScatterLayout 散点布局
StackLayout 堆叠布局

## 2 Building a Clock App
- 内容：学会基本的kivy内置的页面布局语言，domain-speciic language（DSL领域专用语音)；
- 内容：设计内置的 Kivy 组件（最终实现子类化）
- 内容：加载自定义的字体和文本格式
- 内容：事件的调度和监听
### tip
- BoxLayout 盒式布局的容器，允许2个或多个部件并排布局，垂直或水平堆叠
- 可以给部件指定字体文件，如：font_name: 'Lobster.ttf' (ttf, True Type Font)
- 多个.ttf文件还可以使用 LabelBase.register(...) 进行组织
- Kivy 只支持 .tff 文件格式的字体，其他格式需要转换；每个字体最多四种类型（普通、斜体、粗体、粗体斜体)
- Kivy 使用 BBCode（Bulletin Board Code)的轻量标记语言，类似 HTML，但区分是使用两个中括号 [...]
- BBCode 对于设置一部分很有效，如果本身全部都改变了，那直接设置部件的属性
- 建议get_color_from_hex中使用CSS样式的#RRGGBB颜色格式，代替（R，G，B)元祖，阅读友好、搜索简单
- 使用事件驱动的方式思考，永远不要让程序无响应( 拒绝 while True)
- keep in mind，a main loop running inside Kivy, take advantage of it by utilizing events and timers.
- App.on_start. A method will be called as soon as the app is fully initialized. 
- on_press, which fires when the user clicks, taps, or otherwise interacts with a button.
- 时间和计数器，time and timers，内置的 Clock 类，可以简便的实现代码调度

- Clock.schedule_once: Runs a function once after a timeout
- Clock.schedule_interval: Runs a function periodically
- 所有 Clock 组织的时间事件，都 run as a part of Kivy's main event loop.
• self: This always refers to the current widget;
• root: This is the outermost widget of a given scope;
• app: This is the application class instance.
## 3 Building a Paint App

pos: [x , y] # x, y 代表固定的坐标值,单位是pixe
pos_hint: {'center_x': 0.5, 'center_y': 0.5} # x/center_x,right,y/center_y/top 表示的是边线、不是点，并且是一个比例值
size:[width,height]
size_hint: (None, None)

# 1 代码理解 class LoginScreen(Screen, utils.UtilMix):
```
class UtilMix(Resource, FocusIndex, SystemKeyboard, WidgetUtils):
    """基础混合"""
    def __str__(self):
        return "{}:{}".format(self.__class__.__name__, self.__doc__)
    # class FocusIndex(object) -> self.ids=[] 表示控件Widget的id列表，self.focus_able_ids=[]列表中存放的是可切换控件的id、self.focus_index 索引表示的是当前的控件id
    # class WidgetUtils(object) -> 通过父控件获得子控件id，通过父控件获得是哪个子控件在获得焦点，通过id查找控件
    # class SystemKeyboard(object) -> 系统键盘绑定
```

# 打包
## windows10 打包 exe [packaging-windows](https://kivy.org/doc/stable/guide/packaging-windows.html)
- 执行的命令：color_kivy.py -> mkdir color-app -> cd color-app
- 执行的命令：python -m PyInstaller --name color-app /c/Users/Administrator/code/color_kivy.py
- 修改：color-app.spec 文件 a = Analysis 第一个参数，就是py文件的路径，改成 windows 可以认得格式
- 修改：color-app.spec 文件 -> from kivy_deps import sdl2, glew -> Tree('C:\\Users\\Administrator\\code\\'), -> *[Tree(p) for p in (sdl2.dep_bins + glew.dep_bins)],
- 执行的命令：python -m PyInstaller color-app.spec -> 没有问题的话，exe 在 dist 文件夹下
- 备注：汉语的注释是原生不支持的，如果打包需要去掉，或者安装依赖(以后试一下)

![kivy](https://github.com/Django-27/workspace/blob/master/pic/kvTopyAndpyTokv.png)

## centos 打包 apk , and use adb check log
- download mumu and adb, use :adb connect 127.0.0.1:7555 -> adb list (adb需要与mumu模拟器建立连接)
- adb shell
- run-as org.test.myapp -> goto: /data/data/org.test.myapp/files/app/.kivy/logs
- buildozer init -> buildozer -v android debug (打包产生apk文件)
## 遇到的问题：打包后的程序，打开后程序闪退
- 这个闪退的问题，在使用pyInstaller打包exe的时候遇到过，通过在.spec文件中添加代码可以解决
- 使用buildozer打包apk的时候，通过adb调试找到日志文件，提示信息是：AttributeError: 'MyColor' object has no attribute 'slider_colors'
- 将列表变量进行替换，换成绑定canvas的属性变量
```
from kivy.properties import ListProperty
class MyColor(BoxLayout):
    slider_colors = ListProperty([0.5, 0.5, 0.5])
    def __init__(self, **kwargs):
        super(MyColor, self).__init__(**kwargs)
        self.label_update()  # print(self.ids)
```
