###　PYQT5使用中遇到的重点，和坑　
书写时间位于项目开发进度的60%左右，可能看起来顺序可能有点混乱
1.我的项目中，界面需要显示Scrapy的日志，所以使用 subprocess 方式创建子进程，通过获取他的输出流得到日志，
2.在日志的显示效果看，
    我尝试1：使用了 GBK 在调试时就会报错;使用utf-8 本地调试貌似可以，但打包后 就会显示乱码  
      尝试2：p.stdout.readline().decode('utf-8','ignore') 采用这个编码 本地调试时 ，界面正常显示中文，但通过pyinstaller打包后，中文显示乱码
      尝试3：通过查看博客后，另一位也是使用两个编码方式都不行，它使用了另一个编码：gb18030==>p.stdout.readline().decode('gb18030')
            经过测试，打包后都可以正常显示中文，但是 本地调试 会报错:
                  UnicodeDecodeError: 'gb18030' codec can't decode byte 0x80 in position 441: illegal multibyte sequence
3.在使用ui文件转化的py文件时，不要直接使用该
4.lambda表达式。我目前的使用场景是：我从主界面跳转到设置界面时，，我的设置界面是一个Tab widgits ,在主界面通过点击不同的菜单选项，显示设置界面的不同的tab，就需要给菜单选项的关联函数场地参数：
    一般的关联语句：self.local.triggered.connect(self.clickOnLocal)
    使用lambda表达式传递参数：self.database.triggered.connect(lambda:self.showSettingsByIndex(0))

