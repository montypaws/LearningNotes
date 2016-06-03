# 在sublime上搭建Java开发环境

相比于集成环境启动缓慢，插件复杂，本人更倾向于使用Sublime Text 3编写代码，代码免不了要进行调试，在文本编辑器与集成开发环境中来回进行切换是一件很繁琐的事情，因此就有了在ST(Sublime Text 3)上搭建集成开发环境的想法。经过各种百度，先将我的配置过程总结如下：

## Java开发环境搭建
***
1. 配置Java环境变量。
2. 在jdk根目录下的`bin`文件夹下建立批处理文件`runJava.bat`文件。
3. 打开文本编辑器，`runJava.bat`的内容如下：

	    @ECHO OFF
	    cd %~dp1
	    ECHO ------------------我是华丽丽的分割线，编译：------------------
	    ECHO Compiling %~nx1(源文件名)
	    IF EXIST %~n1.class (
	    ECHO 字节文件已经存在啦！
	    DEL %~n1.class
	    ECHO 字节文件已删除！
	    )
	    javac -encoding UTF-8 %~nx1
	    IF EXIST %~n1.class (
	    ECHO -------------我是华丽丽的分割线，以下为输出结果：-------------
	    java %~n1
	    )
4. 进入ST安装目录下，打开Package文件夹找到Java.sublime-package文件，用解压缩软件打开，找到包内的J`avaC.sublime-build`文件，然后使用ST打开编辑，其内容如下：

	    {
	    "shell_cmd": "runJava.bat \"$file\"",
	    "file_regex": "^(...*?):([0-9]*):?([0-9]*)",
	    "selector": "source.java",
	    "encoding": "GBK"
	    }
5. 保存文件，编写java代码进行测试Demo.java，如下：

    	public class Demo{
    	public static void main(String[] args){
    		System.out.println("中文测试：This is my test program.");
    		int a = 10;
    		int b = 20;
    		int c = a + b;
    		System.out.println("Result : " + c);
    		}
    	}
    