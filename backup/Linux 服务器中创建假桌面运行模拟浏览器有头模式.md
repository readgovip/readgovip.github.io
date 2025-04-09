当我们要在 Linux 服务器上面使用 Selenium 或者 Puppeteer 运行爬虫的时候，为了能够使用模拟浏览器的有头模式，我们需要搞一个假的图形界面出来，从而欺骗浏览器，让它的有头模式能够正常使用

使用 Xvfb，我们就可以欺骗 Selenium 或者 Puppeteer，让它以为自己运行在一个有图形界面的系统里面，这样一来就能够正常使用有头模式了。
要安装 Xvfb 非常简单，在 Ubuntu 中，只需要执行下面两行命令就可以了：
```
sudo apt-get update
sudo apt-get install xvfb
```
当然，我们也可以调整一下窗口大小，增加参数：就能假装在一个分辨率为1920x1280的显示器上运行程序了。
```
xvfb-run python3 test.py -s -screen 0 1920x1080x16
```
