## 搭建 selenium 网页文件转图片环境

### 一、 环境准备

```
需要有 chrome 浏览器 + chrome driver + selenium 客户端

离线 chrome 下载地址。

# 64位 linux 系统
https://dl.google.com/linux/direct/google-chrome-stable_current_x86_64.rpm

# 64位 window 系统
http://www.google.cn/chrome/browser/desktop/index.html?standalone=1&platform=win64

# 32位 window 系统
http://www.google.cn/chrome/browser/desktop/index.html?standalone=1&platform=win

# 官网 chromedriver
http://chromedriver.storage.googleapis.com/index.html

# 淘宝 chromedriver 镜像
https://npm.taobao.org/mirrors/chromedriver/

#selenium 官网
https://www.selenium.dev/zh-cn/documentation/
```

**说明：chrome 和 chromedriver 版本需要一致**

### 二、 POM导入

```xml
	<dependency>
            <groupId>org.seleniumhq.selenium</groupId>
            <artifactId>selenium-java</artifactId>
            <version>3.141.59</version>
    </dependency>
```

### 三、Windows环境准备

#### 3.1 查看chrome版本 

```
chrome://version/
```

![image-20220707123957945](https://raw.githubusercontent.com/lilu188011/img-repo/master/image-20220707123957945.png)

#### 3.2 下载对应的chromeDriver

![image-20220707124056808](https://raw.githubusercontent.com/lilu188011/img-repo/master/image-20220707124056808.png)

#### 3.3 将 chromedriver 放到 chrome 安装目录下

![img](https://raw.githubusercontent.com/lilu188011/img-repo/master/852501-20200807145103094-1616762352.png)



### 四、Linux 环境准备

#### 4.1 更新yum 源

```ini
cat > /etc/yum.repos.d/google-chrome.repo << EOF
[google-chrome]
name=google-chrome
baseurl=http://dl.google.com/linux/chrome/rpm/stable/$basearch
enabled=1
gpgcheck=1
gpgkey=https://dl-ssl.google.com/linux/linux_signing_key.pub
EOF
```

#### 4.2 查看版本信息

```
yum info google-chrome-stable
google-chrome -version   # 安装完后查看
```

#### 4.3 yum安装

```
yum -y install google-chrome-stable --nogpgcheck
yum install xorg-x11-server-Xvfb
yum -y install python3
pip3 install pyvirtualdisplay
pip3 install selenium
```

#### 4.4 下载对应的chromeDriver

```
# 官网 chromedriver
http://chromedriver.storage.googleapis.com/index.html

# 淘宝 chromedriver 镜像
https://npm.taobao.org/mirrors/chromedriver/

# 解压后配置环境变量
cp chromedriver /usr/local/bin/
chmod +x /usr/local/bin/chromedriver
```

#### 4.5  python 测试脚本

```python
from pyvirtualdisplay import Display
from selenium import webdriver

display = Display(visible=0, size=(800, 800))

display.start()

chrome_options = webdriver.ChromeOptions()
chrome_options.add_argument("no-sandbox")
chrome_options.add_argument("--disable-extensions")
chrome_options.add_argument("--headless")
driver = webdriver.Chrome(chrome_options=chrome_options)
driver.get('https://www.baidu.com')

print(driver.page_source)

driver.quit()

display.stop()

```

### 五、java+ selenium截图方式

#### 5.1 方式1：TakeScreenshout是[selenium](https://so.csdn.net/so/search?q=selenium&spm=1001.2101.3001.7020)工具自带的截图方法（截图类）,这个类主要是获取浏览器窗体内的内容，不包括浏览器的菜单和桌面的任务栏区域

```java
package org.seleniumhq.selenium;

import java.io.File;
import java.io.IOException;
import java.util.concurrent.TimeUnit;
import java.text.SimpleDateFormat;
import java.util.Calendar;

import org.apache.commons.io.FileUtils;
import org.openqa.selenium.OutputType;
import org.openqa.selenium.TakesScreenshot;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;

/**
 * （1）访问度娘首页
 * （2）调用截图类截图
 * （3）保存截图
 */
public class TakeScreenshot {

	public static void main(String[] args) throws Exception {
		System.setProperty("webdriver.chrome.driver", "F:\\selenium\\chromedriver.exe");
		WebDriver driver = new ChromeDriver();
		driver.manage().window().maximize();
		driver.manage().timeouts().implicitlyWait(5, TimeUnit.SECONDS);
		driver.get("https://www.baidu.com");
		Thread.sleep(1000);
		// 调用截图方法
		File src = ((TakesScreenshot) driver).getScreenshotAs(OutputType.FILE);
		/**
		 * 截屏操作
		 * 图片已当前时间命名
		 */
		SimpleDateFormat dateFormat = new SimpleDateFormat("yyyyMMddHHmmss"); //转换时间格式
		String time = dateFormat.format(Calendar.getInstance().getTime()); //获取当前时间
		try {
			// 拷贝截图文件到我们项目./Screenshots
			FileUtils.copyFile(src, new File("Screenshots", time + ".png"));
			Thread.sleep(3000);
			System.out.println("browser will be close");
			driver.quit();
		} catch (IOException e) {
			System.out.println(e.getMessage());
		}
	}
}

```

```java
public static void convertHtml2Image(String url) throws InterruptedException {　　　　　// 设置 chromedriver 地址
        System.setProperty("webdriver.chrome.driver", "C:\\Program Files (x86)\\Google\\Chrome\\Application\\chromedriver.exe");　　　　// 关闭日志　　　　　System.setProperty("webdriver.chrome.silentOutput", "true");
        //实例化一个Chrome浏览器的实例
        ChromeOptions options = new ChromeOptions();
        // 字符编码 utf-8 支持中文字符
        options.addArguments("lang=zh_CN.UTF-8");
        // 开启最大化
        options.addArguments("–start-maximized");
        options.addArguments("–no-sandbox");
        // 开启无头模式
        options.addArguments("--headless");
        options.addArguments("--disable-gpu");      
        options.addArguments("--window-size=1920,1080");       
        options.addArguments("--silent");        
        // 关闭日志       
        options.addArguments("--disable-logging");
        WebDriver driver = new ChromeDriver(options);
        driver.get(url);
        Thread.sleep(3000);
        int height = Integer.parseInt((String) ((JavascriptExecutor) driver).executeScript("return document.body.scrollHeight.toString()"));
        int width = Integer.parseInt((String) ((JavascriptExecutor) driver).executeScript("return document.body.scrollWidth.toString()"));

        driver.manage().window().setSize(new Dimension(width, height));
        driver.manage().window().maximize();
        //截屏操作
        //截图到output
        File scrFile = ((TakesScreenshot) driver).getScreenshotAs(OutputType.FILE);
        try {
            String desImage = UUID.randomUUID().toString() + ".png";
            //复制内容到指定文件中
            FileUtil.copyFile(scrFile, new File(desImage));
            System.out.println(desImage);
        } catch (IOException e) {
            e.printStackTrace();
        }
        driver.close();
    }
```



#### 5.2 方式2：Robot这个类java.awt.Robot。电脑整个屏幕的截图

```java
package org.seleniumhq.selenium;

import java.awt.Rectangle;
import java.awt.Robot;
import java.awt.Toolkit;
import java.awt.image.BufferedImage;
import java.io.File;

import javax.imageio.ImageIO;

import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;

/**
 （1）访问搜狐首页
 （2）调用截图类截图
 （3）保存截图
 */
public class RobotScreenShot {

	public static void main(String[] args) throws Exception {

		System.setProperty("webdriver.chrome.driver", "F:\\selenium\\chromedriver.exe");
		WebDriver driver = new ChromeDriver();
		driver.manage().window().maximize();
		driver.get("https://www.sohu.com/");
		robotSnapshot();
		Thread.sleep(2000);
		System.out.println("browser will be close");
		driver.quit();

	}

	/**
	 * 截屏方法二、Robot实现截屏
	 * @throws Exception
	 */
	public static void robotSnapshot() throws Exception {
		//调用截图方法
		BufferedImage img = new Robot().createScreenCapture(new Rectangle(Toolkit.getDefaultToolkit().getScreenSize()));
		ImageIO.write(img, "png", new File("RobotScreenshots","robot_screen01.png"));
	}

}
```

#### 5.3 方式3: 截取某个元素（或者目标区域）的图片。对图片进行指定坐标裁剪

```java
package org.seleniumhq.selenium;

import java.awt.image.BufferedImage;
import java.io.File;
import java.io.IOException;

import javax.imageio.ImageIO;

import org.apache.commons.io.FileUtils;
import org.openqa.selenium.By;
import org.openqa.selenium.OutputType;
import org.openqa.selenium.Point;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.interactions.Actions;

/*
具体步骤就是：
（1）访问百度首页
（2）点击“设置”中的“搜索设置”
（3）调用截图类截图搜索设置页面
（4）保存截图（搜索设置页面）
 */
public class ElementScreenShot {

	private static WebDriver driver;
	public static void main(String[] args) throws Exception {
		System.setProperty("webdriver.chrome.driver", "F:\\selenium\\chromedriver.exe");
		driver = new ChromeDriver();
		driver.get("http://www.baidu.com");
		driver.manage().window().maximize();

		Thread.sleep(2000);
		WebElement setting = driver.findElement(By.id("s-usersetting-top"));
		Actions actions = new Actions(driver);
		actions.clickAndHold(setting).perform();   //点击左键不松开
		driver.findElement(By.linkText("搜索设置")).click();
		Thread.sleep(1000);
		WebElement xuanxiang = driver.findElement(By.xpath("/html/body/div[1]/div[6]/div"));
		File src = ((ChromeDriver) driver).getScreenshotAs(OutputType.FILE);
		try {
			FileUtils.copyFile(src, new File("D:\\screenshoot\\result.png"));
			FileUtils.copyFile(ElementScreenShot.captureElement(src, xuanxiang), new File("D:\\screenshoot\\test.png"));
			Thread.sleep(2000);
			System.out.println("browser will be close");
			driver.quit();
		} catch (IOException e) {
			e.printStackTrace();
		}

	}

	public static File captureElement(File screenshot, WebElement element){
		try {
			BufferedImage img = ImageIO.read(screenshot);
			int width = element.getSize().getWidth();
			int height = element.getSize().getHeight();
			//获取指定元素的坐标
			Point point = element.getLocation();
			//从元素左上角坐标开始，按照元素的高宽对img进行裁剪为符合需要的图片
			BufferedImage dest = img.getSubimage(point.getX(), point.getY(), width, height);
			ImageIO.write(dest, "png", screenshot);
		} catch (IOException e) {
			e.printStackTrace();
		}
		return screenshot;
	}

}
```

#### 

```python
from pyvirtualdisplay import Display
from selenium import webdriver

display = Display(visible=0, size=(800, 800))

display.start()

chrome_options = webdriver.ChromeOptions()
chrome_options.add_argument("no-sandbox")
chrome_options.add_argument("--disable-extensions")
chrome_options.add_argument("--headless")
driver = webdriver.Chrome(chrome_options=chrome_options)
driver.get('https://www.baidu.com')

print(driver.page_source)

driver.quit()

display.stop()

```

### 六、制作docker镜像 chrome + chrome driver

```dockerfile
FROM centos:centos7   #我自己使用的是我本地的centos的基础镜像，大家可以改成docker官方仓库，centos:7

#maintainer 作者

MAINTAINER sqy

#add 把包添加到容器的指定目录，如果是tar包会自动解压

ADD 64.0.3282.119_x86_64.rpm /usr/local/lib

ADD chromedriver /usr/local/lib

#workdir 相当于cd到这个目录

WORKDIR /usr/local/lib

RUN yum localinstall -y 64.0.3282.119_x86_64.rpm
```

