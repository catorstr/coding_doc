# Ubuntu Java环境安装

```
# 更新软件包列表
sudo apt-get update
# 安装openjdk-8-jdk
sudo apt-get install openjdk-8-jdk
# 查看java与javac版本
java -version
javac -version

# 利用语句选择数字切换java与javac版本
sudo update-alternatives --config java
sudo update-alternatives --config javac
sudo update-alternatives --config javadoc
# 新建文件夹和java文件
mkdir demo
cd demo
vim hello.java
# 输入
`
public class hello {
	public static void main(String[] args) {
		System.out.println("hello world!");//输出hello world;
	}
}
`
# 生成class文件
javac hello.java
# 运行java文件
java hello
```
