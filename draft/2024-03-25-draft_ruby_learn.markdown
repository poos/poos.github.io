---
layout:     post
title:      
subtitle:   app exit
hide-in-nav: true
date:       2024-03-25
author:     poos
header-img: img/post-bg-2015.jpg
catalog: true
tags:
- Summary
- Code
---

--------------------------正则


ruby正则表达
1、Ruby中正则表达式的写法
主要有三种
* 		在//之间，要进行转义
* 		在%r{}内，不用进行转义
* 		Regexp.new()内，不用进行转义
/mm\/dd/，Regexp.new(“mm/dd”)，%r{mm/dd}三者效果相同，实质都是新建了一个Regexp的类。
2、匹配的两种方法
* 		=~肯定匹配, !~否定匹配。=~表达式返回匹配到的位置索引，失败返回nil，符号左右内容可交换
* 		regexp#match(str)，返回MatchData，一个数组，从0开始，还有match.pre_match返回匹配前内容，match.post_match返回匹配后内容
1
2
3	/cat/ =~ "dog and cat"  #返回8
mt = /cat/.match("bigcatcomes")
"#{mt.pre_match}->#{mt[0]}<-#{mt.post_match}" #返回big->cat<-comes
3、替换
很多时候匹配是为了替换，Ruby中进行正则替换非常简单，两个方法即可搞定，sub()+gsub()。
sub只替换第一次匹配，gsub（g:global）会替换所有的匹配，没有匹配到返回原字符串的copy
1
2
3	str = "ABDADA"
new_str = str.sub(/A/, "*")     #返回"*BDADA"
new_str2 = str.gsub(/A/, "*")   #返回"*BD*D*"
如果想修改原始字符串用sub!()和gsub!()，没有匹配到返回nil。
方法后面还可以跟block，对匹配的字符串进行操作
1	a.gsub(/[aeiou]/) {|vowel| vowel.upcase } # => "qUIck brOwn fOx"
4、分组匹配
Ruby的分组匹配与其它语言差别不大，分组匹配表达式是对要进行分组的内容加()。
对于匹配到的结果，可以用系统变量#$1，#$2…索引，也可用matchData数组来索引
1
2
3	md = /(\d\d):(\d\d)(..)/.match("12:50am") # md为一个MatchData对象
puts "Hour is #$1, minute #$2"
puts "Hour is #{md[1]}, minute #{md[2]}"
5、匹配所有
regexp#match()只能匹配一次，如果想匹配所有要用regexp#scan()
用法示例：
1	"abcabcabz".scan(%r{abc}).each {|item| puts item} # 输出2行abc
6、贪婪匹配vs懒惰匹配
这两种匹配属于标准正则表达式内容，与Ruby没关，但新手如果不明白匹配时会发生莫名其妙的错误，所以特别总结一下。
* 		贪婪匹配：尽可能多匹配，正则默认是贪婪匹配。例子：a.*b它将会匹配最长的以a开始，以b结束的字符串。对于aabab的匹配结果是aabab。
* 		懒惰匹配：尽可能少匹配。例子：a.*?b对于aabab的匹配结果是aab和ab。
一般是在原来表达式结尾加?就由贪婪匹配变成了懒惰匹配。常用的懒惰限定符有（去年最后的问题就是贪婪匹配）：
* 		?重复任意次，但尽可能少重复
* 		+?重复1次或更多次，但尽可能少重复
* 		??重复0次或1次，但尽可能少重复
* 		{n,m}?重复n到m次，但尽可能少重复
* 		{n,}?重复n次以上，但尽可能少重复


--------------------------文件



ruby 对文件的操作
读取一个文件,将其打印出来:
lines = File.open('dom.js').readlines
puts "======================="
lines.each { |line| puts(line)}
或者:
File.open("dom.js") do |file|
  while line = file.gets
    puts line
  end
end
后一种能确保文件用完后被关闭。
向目标文件追加内容：
file = File.open("dom.js","a")
file.puts "//this is new content. "
file.close
但这有时可能出现不能添加中文内容的情况，报“invalid multibyte char (US-ASCII) ”错误，我们就要在当前脚本的最上面添加这么一下注释，就没事了，即
# coding: utf-8 
file = File.open("dom.js","a")
file.puts "//这是新追加的内容. "
file.close
创建一个新文件，并往其里面添加内容。
# coding: utf-8 
file = File.new("new_file.js","w");
file << 'var a = "test";'
file.close;
文件重命名：
# coding: utf-8 
File.rename( "new_file.js", "new.js" )
文件重命名：
# coding: utf-8 
File.rename( "new_file.js", "new.js" )#原来的文件名，新的文件名
删除文件
# coding: utf-8 
File.delete( "new.js" )#原来的文件名
目录操作：
# coding: utf-8 
Dir.mkdir("new")#创建一个新文件夹
Dir.rmdir("new")#删除指定的文件夹
将一个文件拷贝到目标目标：
require 'fileutils'
FileUtils.cp 'new.js', 'new'
将一个文件移动到目标目标：
require 'fileutils'
FileUtils.mv 'new.js', 'new'


字符串包含





--------------------------以下是文档


Ruby 环境变量
Ruby 语法
Ruby 类和对象
Ruby 类案例
Ruby 变量
Ruby 运算符
Ruby 注释
Ruby 判断
Ruby 循环
Ruby 方法
Ruby 块
Ruby 模块（Module）
Ruby 字符串（String）
Ruby 数组（Array）
Ruby 哈希（Hash）
Ruby 日期 & 时间
Ruby 范围（Range）
Ruby 迭代器
Ruby 文件的输入与输出
Ruby File 类和方法
Ruby Dir 类和方法
Ruby 异常

Ruby 高级教程
Ruby 教程
Ruby 教程
Ruby 简介
Ruby 环境
Ruby 安装 - Unix
Ruby 安装 - Windows
Ruby 命令行选项
Ruby 环境变量
Ruby 语法
Ruby 类和对象
Ruby 类案例
Ruby 变量
Ruby 运算符
Ruby 注释
Ruby 判断
Ruby 循环
Ruby 方法
Ruby 块
Ruby 模块（Module）
Ruby 字符串（String）
Ruby 数组（Array）
Ruby 哈希（Hash）
Ruby 日期 & 时间
Ruby 范围（Range）
Ruby 迭代器
Ruby 文件的输入与输出
Ruby File 类和方法
Ruby Dir 类和方法
Ruby 异常

Ruby 高级教程
Ruby 面向对象
Ruby 正则表达式
Ruby 数据库访问 - DBI 教程
Ruby CGI 编程
Ruby CGI方法
Ruby CGI Cookies
Ruby CGI Sessions
Ruby 发送邮件 - SMTP
Ruby Socket 编程
Ruby XML, XSLT 和 XPath 教程
Ruby Web Services
Ruby 多线程


- evn 查看环境

TMPDIR=/var/folders/ql/ffwybqz52qzbhym41vmjm71r0000gn/T/
__CFBundleIdentifier=com.apple.Terminal
XPC_FLAGS=0x0
TERM=xterm-256color
SSH_AUTH_SOCK=/private/tmp/com.apple.launchd.M0V7F2kHuB/Listeners
XPC_SERVICE_NAME=0
TERM_PROGRAM=Apple_Terminal
TERM_PROGRAM_VERSION=440
TERM_SESSION_ID=E514348F-7C19-4BD2-A596-E5E04B4CED41
SHELL=/bin/zsh
HOME=/Users/bieshixuan
LOGNAME=bieshixuan
USER=bieshixuan
PATH=/usr/local/Cellar/oclint/20.11/bin:/Library/Frameworks/Python.framework/Versions/3.9/bin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:/Library/Apple/usr/bin
SHLVL=1
PWD=/Users/bieshixuan
OLDPWD=/Users/bieshixuan
ZSH=/Users/bieshixuan/.oh-my-zsh
PAGER=less
LESS=-R
LSCOLORS=Gxfxcxdxbxegedabagacad
LANG=zh_CN.UTF-8
_=/usr/bin/env


空格
a + b 被解释为 a+b （这是一个局部变量）
a  +b 被解释为 a(+b) （这是一个方法调用）

Ruby 程序中的行尾
Ruby 把分号和换行符解释为语句的结尾。但是，如果 Ruby 在行尾遇到运算符，比如 +、- 或反斜杠，它们表示一个语句的延续。
Ruby 标识符
标识符是变量、常量和方法的名称。Ruby 标识符是大小写敏感的。这意味着 Ram 和 RAM 在 Ruby 中是两个不同的标识符。
Ruby 标识符的名称可以包含字母、数字和下划线字符（ _ ）。
保留字
下表列出了 Ruby 中的保留字。这些保留字不能作为常量或变量的名称。但是，它们可以作为方法名。
BEGIN	do	next	then
END	else	nil	true
alias	elsif	not	undef
and	end	or	unless
begin	ensure	redo	until
break	false	rescue	when
case	for	retry	while
class	if	return	while
def	in	self	__FILE__
defined?	module	super	__LINE__
Ruby 中的 Here Document
"Here Document" 是指建立多行字符串。在 << 之后，您可以指定一个字符串或标识符来终止字符串，且当前行之后直到终止符为止的所有行是字符串的值。
如果终止符用引号括起，引号的类型决定了面向行的字符串类型。请注意<< 和终止符之间必须没有空格。
下面是不同的实例：
#!/usr/bin/ruby -w

print <<EOF
    This is the first way of creating
    here document ie. multiple line string.
EOF

print <<"EOF";                # 与上面相同
    This is the second way of creating
    here document ie. multiple line string.
EOF

print <<`EOC`                 # 执行命令
	echo hi there
	echo lo there
EOC

print <<"foo", <<"bar"	      # 您可以把它们进行堆叠
	I said foo.
foo
	I said bar.
bar
这将产生以下结果：
    This is the first way of creating
    her document ie. multiple line string.
    This is the second way of creating
    her document ie. multiple line string.
hi there
lo there
        I said foo.
        I said bar.
Ruby BEGIN 语句
语法
BEGIN {
   code
}
声明 code 会在程序运行之前被调用。
实例
#!/usr/bin/ruby

puts "This is main Ruby Program"

BEGIN {
   puts "Initializing Ruby Program"
}
这将产生以下结果：
Initializing Ruby Program
This is main Ruby Program
Ruby END 语句
语法
END {
   code
}
声明 code 会在程序的结尾被调用。
实例
#!/usr/bin/ruby

puts "This is main Ruby Program"

END {
   puts "Terminating Ruby Program"
}
BEGIN {
   puts "Initializing Ruby Program"
}
这将产生以下结果：
Initializing Ruby Program
This is main Ruby Program
Terminating Ruby Program
Ruby 注释
注释会对 Ruby 解释器隐藏一行，或者一行的一部分，或者若干行。您可以在行首使用字符（ # ）：
# 我是注释，请忽略我。
或者，注释可以跟着语句或表达式的同一行的后面：
name = "Madisetti" # 这也是注释
您可以注释多行，如下所示：
# 这是注释。
# 这也是注释。
# 这也是注释。
# 这还是注释。
下面是另一种形式。这种块注释会对解释器隐藏 =begin/=end 之间的行：
=begin
这是注释。
这也是注释。
这也是注释。
这还是注释。
=end


		

Ruby 类和对象
Ruby 是一种完美的面向对象编程语言。面向对象编程语言的特性包括：
* 数据封装
* 数据抽象
* 多态性
* 继承
这些特性将在 面向对象的 Ruby 中进行讨论。
一个面向对象的程序，涉及到的类和对象。类是个别对象创建的蓝图。在面向对象的术语中，您的自行车是自行车类的一个实例。
以车辆为例，它包括车轮（wheels）、马力（horsepower）、燃油或燃气罐容量（fuel or gas tank capacity）。这些属性形成了车辆（Vehicle）类的数据成员。借助这些属性您能把一个车辆从其他车辆中区分出来。
车辆也能包含特定的函数，比如暂停（halting）、驾驶（driving）、超速（speeding）。这些函数形成了车辆（Vehicle）类的数据成员。因此，您可以定义类为属性和函数的组合。
类 Vehicle 的定义如下：
Class Vehicle
{
   Number no_of_wheels
   Number horsepower
   Characters type_of_tank
   Number Capacity
   Function speeding
   {
   }
   Function driving
   {
   }
   Function halting
   {
   }
}
通过给这些数据成员分配不同的值，您可以创建类 Vehicle 的不同实例。例如，一架飞机有三个轮子，马力 1,000，燃油罐容量为 100 升。以同样的方式，一辆汽车有四个轮子，马力 200，煤气罐容量为 25 升。
在 Ruby 中定义类
为了使用 Ruby 实现面向对象编程，您需要先学习如何在 Ruby 中创建对象和类。
在 Ruby 中，类总是以关键字 class 开始，后跟类的名称。类名的首字母应该大写。类 Customer 如下所示：
class Customer
end
您可以使用关键字 end 终止一个类。类 中的所有数据成员都是介于类定义和 end 关键字之间。
Ruby 类中的变量
Ruby 提供了四种类型的变量：
* 局部变量：局部变量是在方法中定义的变量。局部变量在方法外是不可用的。在后续的章节中，您将看到有关方法的更多细节。局部变量以小写字母或 _ 开始。
* 实例变量：实例变量可以跨任何特定的实例或对象中的方法使用。这意味着，实例变量可以从对象到对象的改变。实例变量在变量名之前放置符号（@）。
* 类变量：类变量可以跨不同的对象使用。类变量属于类，且是类的一个属性。类变量在变量名之前放置符号（@@）。
* 全局变量：类变量不能跨类使用。如果您想要有一个可以跨类使用的变量，您需要定义全局变量。全局变量总是以美元符号（$）开始。
实例
使用类变量 @@no_of_customers，您可以判断被创建的对象数量，这样可以确定客户数量。
class Customer
   @@no_of_customers=0
end
在 Ruby 中使用 new 方法创建对象
对象是类的实例。现在您将学习如何在 Ruby 中创建类的对象。在 Ruby 中，您可以使用类的方法 new 创建对象。
方法 new 是一种独特的方法，在 Ruby 库中预定义。new 方法属于类方法。
下面的实例创建了类 Customer 的两个对象 cust1 和 cust2：
cust1 = Customer. new
cust2 = Customer. new
在这里，cust1 和 cust2 是两个对象的名称。对象名称后跟着等号（=），等号后跟着类名，然后是点运算符和关键字 new。
自定义方法来创建 Ruby 对象
您可以给方法 new 传递参数，这些参数可用于初始化类变量。
当您想要声明带参数的 new 方法时，您需要在创建类的同时声明方法 initialize。
initialize 方法是一种特殊类型的方法，将在调用带参数的类的 new 方法时执行。
下面的实例创建了 initialize 方法：
class Customer
   @@no_of_customers=0
   def initialize(id, name, addr)
      @cust_id=id
      @cust_name=name
      @cust_addr=addr
   end
end
在本实例中，您可以声明带有 id、name、addr 作为局部变量的 initialize方法。在这里，def 和 end 用于定义 Ruby 方法 initialize。在后续的章节中，您将学习有关方法的更多细节。
在 initialize 方法中，把这些局部变量的值传给实例变量 @cust_id、@cust_name 和 @cust_addr。在这里，局部变量的值是随着 new 方法进行传递的。
现在，您可以创建对象，如下所示：
cust1=Customer.new("1", "John", "Wisdom Apartments, Ludhiya")
cust2=Customer.new("2", "Poul", "New Empire road, Khandala")
Ruby 类中的成员函数
在 Ruby 中，函数被称为方法。类中的每个方法是以关键字 def 开始，后跟方法名。
方法名总是以小写字母开头。在 Ruby 中，您可以使用关键字 end 来结束一个方法。
下面的实例定义了一个 Ruby 方法：
class Sample
   def function
      statement 1
      statement 2
   end
end
在这里，statement 1 和 statement 2 是类 Sample 内的方法 function 的主体的组成部分。这些语句可以是任何有效的 Ruby 语句。例如，我们可以使用方法 puts 来输出 Hello Ruby，如下所示：
class Sample
   def hello
      puts "Hello Ruby!"
   end
end
下面的实例将创建类 Sample 的一个对象，并调用 hello 方法：
#!/usr/bin/ruby

class Sample
   def hello
      puts "Hello Ruby!"
   end
end

# 使用上面的类来创建对象
object = Sample. new
object.hello
这将会产生下面的结果：
Hello Ruby!

		
Ruby 类案例
下面将创建一个名为 Customer 的 Ruby 类，您将声明两个方法：
* display_details：该方法用于显示客户的详细信息。
* total_no_of_customers：该方法用于显示在系统中创建的客户总数量。
#!/usr/bin/ruby

class Customer
   @@no_of_customers=0
   def initialize(id, name, addr)
      @cust_id=id
      @cust_name=name
      @cust_addr=addr
   end
   def display_details()
      puts "Customer id #@cust_id"
      puts "Customer name #@cust_name"
      puts "Customer address #@cust_addr"
    end
    def total_no_of_customers()
       @@no_of_customers += 1
       puts "Total number of customers: #@@no_of_customers"
    end
end
display_details 方法包含了三个 puts 语句，显示了客户 ID、客户名字和客户地址。其中，puts 语句：
puts "Customer id #@cust_id"
将在一个单行上显示文本 Customer id，后跟变量 @cust_id 的值。
当您想要在一个单行上显示实例变量的文本和值时，您需要在 puts 语句的变量名前面放置符号（#）。文本和带有符号（#）的实例变量应使用双引号标记。
第二个方法，total_no_of_customers，包含了类变量 @@no_of_customers。表达式 @@no_of_ customers+=1 在每次调用方法 total_no_of_customers 时，把变量 no_of_customers 加 1。通过这种方式，您将得到类变量中的客户总数量。
现在创建两个客户，如下所示：
cust1=Customer.new("1", "John", "Wisdom Apartments, Ludhiya")
cust2=Customer.new("2", "Poul", "New Empire road, Khandala")
在这里，我们创建了 Customer 类的两个对象，cust1 和 cust2，并向 new 方法传递必要的参数。当 initialize 方法被调用时，对象的必要属性被初始化。
一旦对象被创建，您需要使用两个对象来调用类的方法。如果您想要调用方法或任何数据成员，您可以编写代码，如下所示：
cust1.display_details()
cust1.total_no_of_customers()
对象名称后总是跟着一个点号，接着是方法名称或数据成员。我们已经看到如何使用 cust1 对象调用两个方法。使用 cust2 对象，您也可以调用两个方法，如下所示：
cust2.display_details()
cust2.total_no_of_customers()
保存并执行代码
现在，把所有的源代码放在 main.rb 文件中，如下所示：
#!/usr/bin/ruby

class Customer
   @@no_of_customers=0
   def initialize(id, name, addr)
      @cust_id=id
      @cust_name=name
      @cust_addr=addr
   end
   def display_details()
      puts "Customer id #@cust_id"
      puts "Customer name #@cust_name"
      puts "Customer address #@cust_addr"
   end
   def total_no_of_customers()
      @@no_of_customers += 1
      puts "Total number of customers: #@@no_of_customers"
   end
end

# 创建对象
cust1=Customer.new("1", "John", "Wisdom Apartments, Ludhiya")
cust2=Customer.new("2", "Poul", "New Empire road, Khandala")

# 调用方法
cust1.display_details()
cust1.total_no_of_customers()
cust2.display_details()
cust2.total_no_of_customers()
接着，运行程序，如下所示：
$ ruby main.rb
这将产生以下结果：
Customer id 1
Customer name John
Customer address Wisdom Apartments, Ludhiya
Total number of customers: 1
Customer id 2
Customer name Poul
Customer address New Empire road, Khandala
Total number of customers: 2

		
Ruby 变量
变量是持有可被任何程序使用的任何数据的存储位置。
Ruby 支持五种类型的变量。您已经在前面的章节中大概了解了这些变量，本章节将为您详细讲解这五种类型的变量。
Ruby 全局变量
全局变量以 $ 开头。未初始化的全局变量的值为 nil，在使用 -w 选项后，会产生警告。
给全局变量赋值会改变全局状态，所以不建议使用全局变量。
下面的实例显示了全局变量的用法。
#!/usr/bin/ruby

$global_variable = 10
class Class1
  def print_global
     puts "Global variable in Class1 is #$global_variable"
  end
end
class Class2
  def print_global
     puts "Global variable in Class2 is #$global_variable"
  end
end

class1obj = Class1.new
class1obj.print_global
class2obj = Class2.new
class2obj.print_global
在这里，$global_variable 是全局变量。这将产生以下结果：
注意：在 Ruby 中，您可以通过在变量或常量前面放置 # 字符，来访问任何变量或常量的值。
Global variable in Class1 is 10
Global variable in Class2 is 10
Ruby 实例变量
实例变量以 @ 开头。未初始化的实例变量的值为 nil，在使用 -w 选项后，会产生警告。
下面的实例显示了实例变量的用法。
#!/usr/bin/ruby

class Customer
   def initialize(id, name, addr)
      @cust_id=id
      @cust_name=name
      @cust_addr=addr
   end
   def display_details()
      puts "Customer id #@cust_id"
      puts "Customer name #@cust_name"
      puts "Customer address #@cust_addr"
    end
end

# 创建对象
cust1=Customer.new("1", "John", "Wisdom Apartments, Ludhiya")
cust2=Customer.new("2", "Poul", "New Empire road, Khandala")

# 调用方法
cust1.display_details()
cust2.display_details()
在这里，@cust_id、@cust_name 和 @cust_addr 是实例变量。这将产生以下结果：
Customer id 1
Customer name John
Customer address Wisdom Apartments, Ludhiya
Customer id 2
Customer name Poul
Customer address New Empire road, Khandala
Ruby 类变量
类变量以 @@ 开头，且必须初始化后才能在方法定义中使用。
引用一个未初始化的类变量会产生错误。类变量在定义它的类或模块的子类或子模块中可共享使用。
在使用 -w 选项后，重载类变量会产生警告。
下面的实例显示了类变量的用法。
#!/usr/bin/ruby

class Customer
   @@no_of_customers=0
   def initialize(id, name, addr)
      @cust_id=id
      @cust_name=name
      @cust_addr=addr
   end
   def display_details()
      puts "Customer id #@cust_id"
      puts "Customer name #@cust_name"
      puts "Customer address #@cust_addr"
    end
    def total_no_of_customers()
       @@no_of_customers += 1
       puts "Total number of customers: #@@no_of_customers"
    end
end

# 创建对象
cust1=Customer.new("1", "John", "Wisdom Apartments, Ludhiya")
cust2=Customer.new("2", "Poul", "New Empire road, Khandala")

# 调用方法
cust1.total_no_of_customers()
cust2.total_no_of_customers()
在这里，@@no_of_customers 是类变量。这将产生以下结果：
Total number of customers: 1
Total number of customers: 2
Ruby 局部变量
局部变量以小写字母或下划线 _ 开头。局部变量的作用域从 class、module、def 或 do 到相对应的结尾或者从左大括号到右大括号 {}。
当调用一个未初始化的局部变量时，它被解释为调用一个不带参数的方法。
对未初始化的局部变量赋值也可以当作是变量声明。变量会一直存在，直到当前域结束为止。局部变量的生命周期在 Ruby 解析程序时确定。
在上面的实例中，局部变量是 id、name 和 addr。
Ruby 常量
常量以大写字母开头。定义在类或模块内的常量可以从类或模块的内部访问，定义在类或模块外的常量可以被全局访问。
常量不能定义在方法内。引用一个未初始化的常量会产生错误。对已经初始化的常量赋值会产生警告。
#!/usr/bin/ruby

class Example
   VAR1 = 100
   VAR2 = 200
   def show
       puts "Value of first Constant is #{VAR1}"
       puts "Value of second Constant is #{VAR2}"
   end
end

# 创建对象
object=Example.new()
object.show
在这里，VAR1 和 VAR2 是常量。这将产生以下结果：
Value of first Constant is 100
Value of second Constant is 200

		
Ruby 伪变量
它们是特殊的变量，有着局部变量的外观，但行为却像常量。您不能给这些变量赋任何值。
* self: 当前方法的接收器对象。
* true: 代表 true 的值。
* false: 代表 false 的值。
* nil: 代表 undefined 的值。
* __FILE__: 当前源文件的名称。
* __LINE__: 当前行在源文件中的编号。
Ruby 基本文字
Ruby 用于文字的规则简单而又直观。这部分解释了 Ruby 中所有的基本文字。
整数
Ruby 支持整数。整数范围从 -230 到 230-1 或 -262 到 262-1。在这个范围内的整数是类 Fixnum 的对象，在这个范围外的整数存储在类 Bignum 的对象中。
您可以在整数前使用一个可选的前导符号，一个可选的基础指标（0 对应 octal，0x 对应 hex，0b 对应 binary），后跟一串数字。下划线字符在数字字符串中被忽略。
您可以获取一个 ASCII 字符或一个用问号标记的转义序列的整数值。
实例：
123                  # Fixnum 十进制
1_234                # Fixnum 带有下划线的十进制
-500                 # 负的 Fixnum
0377                 # 八进制
0xff                 # 十六进制
0b1011               # 二进制
?a                   # 'a' 的字符编码
?\n                  # 换行符（0x0a）的编码
12345678901234567890 # Bignum
注意：类和对象会在本教程中一个单独的章节进行讲解。
浮点数
Ruby 支持浮点数。它们是带有小数的数字。浮点数是类 Float 的对象，且可以是下列中任意一个。
实例：
123.4                # 浮点值
1.0e6                # 科学记数法
4E20                 # 不是必需的
4e+20                # 指数前的符号
字符串文字
Ruby 字符串简单地说是一个 8 位字节序列，它们是类 String 的对象。双引号标记的字符串允许替换和使用反斜线符号，单引号标记的字符串不允许替换，且只允许使用 \\ 和 \' 两个反斜线符号。
实例：
#!/usr/bin/ruby -w

puts 'escape using "\\"';
puts 'That\'s right';
这将产生以下结果：
escape using "\"
That's right
您可以使用序列 #{ expr } 替换任意 Ruby 表达式的值为一个字符串。在这里，expr 可以是任意的 Ruby 表达式。
#!/usr/bin/ruby -w

puts "Multiplication Value : #{24*60*60}";
这将产生以下结果：
Multiplication Value : 86400
反斜线符号
下表列出了 Ruby 支持的反斜线符号：
符号	表示的字符
\n	换行符 (0x0a)
\r	回车符 (0x0d)
\f	换页符 (0x0c)
\b	退格键 (0x08)
\a	报警符 Bell (0x07)
\e	转义符 (0x1b)
\s	空格符 (0x20)
\nnn	八进制表示法 (n 是 0-7)
\xnn	十六进制表示法 (n 是 0-9、a-f 或 A-F)
\cx, \C-x	Control-x
\M-x	Meta-x (c | 0x80)
\M-\C-x	Meta-Control-x
\x	字符 x
如需了解更多有关 Ruby 字符串的细节，请查看 Ruby 字符串（String）。
		
Ruby 数组
Ruby 数组是在方括号内放置一系列逗号分隔的对象引用。尾部的逗号会被忽略。
实例：
#!/usr/bin/ruby

ary = [  "fred", 10, 3.14, "This is a string", "last element", ]
ary.each do |i|
   puts i
end
这将产生以下结果：
fred
10
3.14
This is a string
last element
如需了解更多有关 Ruby 数组的细节，请查看 Ruby 数组（Array）。
Ruby 哈希
Ruby 哈希是在大括号内放置一系列键/值对，键和值之间使用逗号和序列 => 分隔。尾部的逗号会被忽略。
实例：
#!/usr/bin/ruby

hsh = colors = { "red" => 0xf00, "green" => 0x0f0, "blue" => 0x00f }
hsh.each do |key, value|
   print key, " is ", value, "\n"
end
这将产生以下结果：
green is 240
red is 3840
blue is 15
如需了解更多有关 Ruby 哈希的细节，请查看 Ruby 哈希（Hash）。
Ruby 范围
一个范围表示一个时间间隔。范围是通过设置一个开始值和一个结束值来表示。范围可使用 s..e 和 s...e 来构造，或者通过 Range.new 来构造。
使用 .. 构造的范围从开始值运行到结束值（包含结束值）。使用 ... 构造的范围从开始值运行到结束值（不包含结束值）。当作为一个迭代器使用时，范围会返回序列中的每个值。
范围 (1..5) 意味着它包含值 1, 2, 3, 4, 5，范围 (1...5) 意味着它包含值 1, 2, 3, 4 。
实例：
#!/usr/bin/ruby

(10..15).each do |n| 
   print n, ' ' 
end
这将产生以下结果：
10 11 12 13 14 15
		
Ruby 运算符
Ruby 支持一套丰富的运算符。大多数运算符实际上是方法调用。例如，a + b 被解释为 a.+(b)，其中指向变量 a 的 + 方法被调用，b 作为方法调用的参数。
对于每个运算符（+ - * / % ** & | ^ << >> && ||），都有一个相对应的缩写赋值运算符（+= -= 等等）。
Ruby 算术运算符
假设变量 a 的值为 10，变量 b 的值为 20，那么：
运算符	描述	实例
+	加法 - 把运算符两边的操作数相加	a + b 将得到 30
-	减法 - 把左操作数减去右操作数	a - b 将得到 -10
*	乘法 - 把运算符两边的操作数相乘	a * b 将得到 200
/	除法 - 把左操作数除以右操作数	b / a 将得到 2
%	求模 - 把左操作数除以右操作数，返回余数	b % a 将得到 0
**	指数 - 执行指数计算	a**b 将得到 10 的 20 次方
Ruby 比较运算符
假设变量 a 的值为 10，变量 b 的值为 20，那么：
运算符	描述	实例
==	检查两个操作数的值是否相等，如果相等则条件为真。	(a == b) 不为真。
!=	检查两个操作数的值是否相等，如果不相等则条件为真。	(a != b) 为真。
>	检查左操作数的值是否大于右操作数的值，如果是则条件为真。	(a > b) 不为真。
<	检查左操作数的值是否小于右操作数的值，如果是则条件为真。	(a < b) 为真。
>=	检查左操作数的值是否大于或等于右操作数的值，如果是则条件为真。	(a >= b) 不为真。
<=	检查左操作数的值是否小于或等于右操作数的值，如果是则条件为真。	(a <= b) 为真。
<=>	联合比较运算符。如果第一个操作数等于第二个操作数则返回 0，如果第一个操作数大于第二个操作数则返回 1，如果第一个操作数小于第二个操作数则返回 -1。	(a <=> b) 返回 -1。
===	用于测试 case 语句的 when 子句内的相等。	(1...10) === 5 返回 true。
.eql?	如果接收器和参数具有相同的类型和相等的值，则返回 true。	1 == 1.0 返回 true，但是 1.eql?(1.0) 返回 false。
equal?	如果接收器和参数具有相同的对象 id，则返回 true。	如果 aObj 是 bObj 的副本，那么 aObj == bObj 返回 true，a.equal?bObj 返回 false，但是 a.equal?aObj 返回 true。
Ruby 赋值运算符
假设变量 a 的值为 10，变量 b 的值为 20，那么：
运算符	描述	实例
=	简单的赋值运算符，把右操作数的值赋给左操作数	c = a + b 将把 a + b 的值赋给 c
+=	加且赋值运算符，把右操作数加上左操作数的结果赋值给左操作数	c += a 相当于 c = c + a
-=	减且赋值运算符，把左操作数减去右操作数的结果赋值给左操作数	c -= a 相当于 c = c - a
*=	乘且赋值运算符，把右操作数乘以左操作数的结果赋值给左操作数	c *= a 相当于 c = c * a
/=	除且赋值运算符，把左操作数除以右操作数的结果赋值给左操作数	c /= a 相当于 c = c / a
%=	求模且赋值运算符，求两个操作数的模赋值给左操作数	c %= a 相当于 c = c % a
**=	指数且赋值运算符，执行指数计算，并赋值给左操作数	c **= a 相当于 c = c ** a
Ruby 并行赋值
Ruby 也支持变量的并行赋值。这使得多个变量可以通过一行的 Ruby 代码进行初始化。例如：
a = 10
b = 20
c = 30
使用并行赋值可以更快地声明：
a, b, c = 10, 20, 30
并行赋值在交换两个变量的值时也很有用：
a, b = b, c
Ruby 位运算符
位运算符作用于位，并逐位执行操作。
假设如果 a = 60，且 b = 13，现在以二进制格式，它们如下所示：
a = 0011 1100
b = 0000 1101
-----------------
a&b = 0000 1100
a|b = 0011 1101
a^b = 0011 0001
~a  = 1100 0011
下表列出了 Ruby 支持的位运算符。
运算符	描述	实例
&	如果同时存在于两个操作数中，二进制 AND 运算符复制一位到结果中。	(a & b) 将得到 12，即为 0000 1100
|	如果存在于任一操作数中，二进制 OR 运算符复制一位到结果中。	(a | b) 将得到 61，即为 0011 1101
^	如果存在于其中一个操作数中但不同时存在于两个操作数中，二进制异或运算符复制一位到结果中。	(a ^ b) 将得到 49，即为 0011 0001
~	二进制补码运算符是一元运算符，具有"翻转"位效果。	(~a ) 将得到 -61，即为 1100 0011，2 的补码形式，带符号的二进制数。
<<	二进制左移运算符。左操作数的值向左移动右操作数指定的位数。	a << 2 将得到 240，即为 1111 0000
>>	二进制右移运算符。左操作数的值向右移动右操作数指定的位数。	a >> 2 将得到 15，即为 0000 1111
Ruby 逻辑运算符
下表列出了 Ruby 支持的逻辑运算符。
假设变量 a 的值为 10，变量 b 的值为 20，那么：
运算符	描述	实例
and	称为逻辑与运算符。如果两个操作数都为真，则条件为真。	(a and b) 为真。
or	称为逻辑或运算符。如果两个操作数中有任意一个非零，则条件为真。	(a or b) 为真。
&&	称为逻辑与运算符。如果两个操作数都非零，则条件为真。	(a && b) 为真。
||	称为逻辑或运算符。如果两个操作数中有任意一个非零，则条件为真。	(a || b) 为真。
!	称为逻辑非运算符。用来逆转操作数的逻辑状态。如果条件为真则逻辑非运算符将使其为假。	!(a && b) 为假。
not	称为逻辑非运算符。用来逆转操作数的逻辑状态。如果条件为真则逻辑非运算符将使其为假。	not(a && b) 为假。
Ruby 三元运算符
有一个以上的操作称为三元运算符。第一个计算表达式的真假值，然后根据这个结果决定执行后边两个语句中的一个。条件运算符的语法如下：
运算符	描述	实例
? :	条件表达式	如果条件为真 ? 则值为 X : 否则值为 Y
Ruby 范围运算符
在 Ruby 中，序列范围用于创建一系列连续的值 - 包含起始值、结束值（视情况而定）和它们之间的值。
在 Ruby 中，这些序列是使用 ".." 和 "..." 范围运算符来创建的。两点形式创建的范围包含起始值和结束值，三点形式创建的范围只包含起始值不包含结束值。
运算符	描述	实例
..	创建一个从开始点到结束点的范围（包含结束点）	1..10 创建从 1 到 10 的范围
...	创建一个从开始点到结束点的范围（不包含结束点）	1...10 创建从 1 到 9 的范围
		
Ruby 点运算符 "." 和双冒号运算符 "::"
您可以通过在方法名称前加上模块名称和一条下划线来调用模块方法。您可以使用模块名称和两个冒号来引用一个常量。
:: 是一元运算符，允许在类或模块内定义常量、实例方法和类方法，可以从类或模块外的任何地方进行访问。
请记住：在 Ruby 中，类和方法也可以被当作常量。
您只需要在表达式的常量名前加上 :: 前缀，即可返回适当的类或模块对象。
如果未使用前缀表达式，则默认使用主 Object 类。
下面是两个实例：
MR_COUNT = 0        # 定义在主 Object 类上的常量
module Foo
  MR_COUNT = 0
  ::MR_COUNT = 1    # 设置全局计数为 1
  MR_COUNT = 2      # 设置局部计数为 2
end
puts MR_COUNT       # 这是全局常量
puts Foo::MR_COUNT  # 这是 "Foo" 的局部常量 
第二个实例：
CONST = ' out there'
class Inside_one
   CONST = proc {' in there'}
   def where_is_my_CONST
      ::CONST + ' inside one'
   end
end
class Inside_two
   CONST = ' inside two'
   def where_is_my_CONST
      CONST
   end
end
puts Inside_one.new.where_is_my_CONST
puts Inside_two.new.where_is_my_CONST
puts Object::CONST + Inside_two::CONST
puts Inside_two::CONST + CONST
puts Inside_one::CONST
puts Inside_one::CONST.call + Inside_two::CONST
Ruby 运算符的优先级
下表按照运算符的优先级从高到低列出了所有的运算符。
方法	运算符	描述
是	::	常量解析运算符
是	[ ] [ ]=	元素引用、元素集合
是	**	指数
是	! ~ + -	非、补、一元加、一元减（最后两个的方法名为 +@ 和 -@）
是	* / %	乘法、除法、求模
是	+ -	加法和减法
是	>> <<	位右移、位左移
是	&	位与
是	^ |	位异或、位或
是	<= < > >=	比较运算符
是	<=> == === != =~ !~	相等和模式匹配运算符（!= 和 !~ 不能被定义为方法）
	&&	逻辑与
	||	逻辑或
	.. ...	范围（包含、不包含）
	? :	三元 if-then-else
	= %= { /= -= += |= &= >>= <<= *= &&= ||= **=	赋值
	defined?	检查指定符号是否已定义
	not	逻辑否定
	or and	逻辑组成
注意：在方法列标识为 是 的运算符实际上是方法，因此可以被重载。
		
Ruby 注释
注释是在运行时会被忽略的 Ruby 代码内的注释行。单行注释以 # 字符开始，直到该行结束，如下所示：
#!/usr/bin/ruby -w

# 这是一个单行注释。

puts "Hello, Ruby!"
当执行时，上面的程序会产生以下结果：
Hello, Ruby!
Ruby 多行注释
您可以使用 =begin 和 =end 语法注释多行，如下所示：
#!/usr/bin/ruby -w

puts "Hello, Ruby!"

=begin
这是一个多行注释。
可扩展至任意数量的行。
但 =begin 和 =end 只能出现在第一行和最后一行。 
=end
当执行时，上面的程序会产生以下结果：
Hello, Ruby!
请确保尾部的注释离代码有足够的距离，以便容易区分注释和代码。如果块中超过一条尾部注释，请对齐它们。例如：
@counter      # 跟踪页面被击中的次数
@siteCounter  # 跟踪所有页面被击中的次数

		

Ruby 循环 »
Ruby 判断
Ruby 提供了其他现代语言中很常见的条件结构。在这里，我们将解释所有的条件语句和 Ruby 中可用的修饰符。
Ruby if...else 语句
语法
if conditional [then]
	  code...
[elsif conditional [then]
	  code...]...
[else
	  code...]
end
if 表达式用于条件执行。值 false 和 nil 为假，其他值都为真。请注意，Ruby 使用 elsif，不是使用 else if 和 elif。
如果 conditional 为真，则执行 code。如果 conditional 不为真，则执行 else 子句中指定的 code。
if 表达式的 conditional 通过保留字 then、一个换行符或一个分号，来与代码分离开。
实例
#!/usr/bin/ruby

x=1
if x > 2
   puts "x is greater than 2"
elsif x <= 2 and x!=0
   puts "x is 1"
else
   puts "I can't guess the number"
end
x is 1
Ruby if 修饰符
语法
code if condition
如果 conditional 为真，则执行 code。
实例
#!/usr/bin/ruby

$debug=1
print "debug\n" if $debug
这将产生以下结果：
debug
Ruby unless 语句
语法
unless conditional [then]
   code
[else
   code ]
end
如果 conditional 为假，则执行 code。如果 conditional 为真，则执行 else 子句中指定的 code。
实例
#!/usr/bin/ruby

x=1
unless x>2
   puts "x is less than 2"
 else
  puts "x is greater than 2"
end
这将产生以下结果：
x is less than 2
Ruby unless 修饰符
语法
code unless conditional
如果 conditional 为假，则执行 code。
实例
#!/usr/bin/ruby

$var =  1
print "1 -- Value is set\n" if $var
print "2 -- Value is set\n" unless $var

$var = false
print "3 -- Value is set\n" unless $var
这将产生以下结果：
1 -- Value is set
3 -- Value is set
Ruby case 语句
语法
case expression
[when expression [, expression ...] [then]
   code ]...
[else
   code ]
end
比较 case 所指定的 expression，当使用 === 运算符指定时，执行匹配的 when 子句的 code。
when 子句所指定的 expression 背当作左操作数。如果没有匹配的 when 子句，case 执行 else 子句的代码。
when 语句的表达式通过保留字 then、一个换行符或一个分号，来与代码分离开。
因此：
case expr0
when expr1, expr2
   stmt1
when expr3, expr4
   stmt2
else
   stmt3
end
基本上类似于：
_tmp = expr0
if expr1 === _tmp || expr2 === _tmp
   stmt1
elsif expr3 === _tmp || expr4 === _tmp
   stmt2
else
   stmt3
end
实例
#!/usr/bin/ruby

$age =  5
case $age
when 0 .. 2
    puts "baby"
when 3 .. 6
    puts "little child"
when 7 .. 12
    puts "child"
when 13 .. 18
    puts "youth"
else
    puts "adult"
end
这将产生以下结果：

		
Ruby while 语句
语法
while conditional [do]
   code
end
当 conditional 为真时，执行 code。while 循环的 conditional 通过保留字 do、一个换行符、反斜线 \ 或一个分号 ; ，来与 code 分离开。
实例
#!/usr/bin/ruby

$i = 0
$num = 5

while $i < $num  do
   puts("Inside the loop i = #$i" )
   $i +=1
end
这将产生以下结果：
Inside the loop i = 0
Inside the loop i = 1
Inside the loop i = 2
Inside the loop i = 3
Inside the loop i = 4
Ruby while 修饰符
语法
code while condition

OR

begin 
  code 
end while conditional
当 conditional 为真时，执行 code。
如果 while 修饰符跟在一个没有 rescue 或 ensure 子句的 begin 语句后面，code 会在 conditional 判断之前执行一次。
实例
#!/usr/bin/ruby

$i = 0
$num = 5
begin
   puts("Inside the loop i = #$i" )
   $i +=1
end while $i < $num
这将产生以下结果：
Inside the loop i = 0
Inside the loop i = 1
Inside the loop i = 2
Inside the loop i = 3
Inside the loop i = 4
Ruby until 语句
until conditional [do]
   code
end
当 conditional 为假时，执行 code。until 语句的 conditional 通过保留字 do、一个换行符或一个分号，来与 code 分离开。
实例
#!/usr/bin/ruby

$i = 0
$num = 5

until $i > $num  do
   puts("Inside the loop i = #$i" )
   $i +=1;
end
这将产生以下结果：
Inside the loop i = 0
Inside the loop i = 1
Inside the loop i = 2
Inside the loop i = 3
Inside the loop i = 4
Inside the loop i = 5
Ruby until 修饰符
语法
code until conditional

OR

begin
   code
end until conditional
当 conditional 为假时，执行 code。
如果 until 修饰符跟在一个没有 rescue 或 ensure 子句的 begin 语句后面，code 会在 conditional 判断之前执行一次。
实例
#!/usr/bin/ruby

$i = 0
$num = 5
begin
   puts("Inside the loop i = #$i" )
   $i +=1;
end until $i > $num
这将产生以下结果：
Inside the loop i = 0
Inside the loop i = 1
Inside the loop i = 2
Inside the loop i = 3
Inside the loop i = 4
Inside the loop i = 5
Ruby for 语句
语法
for variable [, variable ...] in expression [do]
   code
end
针对 expression 中的每个元素分别执行一次 code。
实例
#!/usr/bin/ruby

for i in 0..5
   puts "Value of local variable is #{i}"
end
在这里，我们已经定义了范围 0..5。语句 for i in 0..5 允许 i 的值从 0 到 5（包含 5）。这将产生以下结果：
Value of local variable is 0
Value of local variable is 1
Value of local variable is 2
Value of local variable is 3
Value of local variable is 4
Value of local variable is 5
for...in 循环几乎是完全等价于：
(expression).each do |variable[, variable...]| code end
但是，for 循环不会为局部变量创建一个新的作用域。for 循环的 expression 通过保留字 do、一个换行符或一个分号，来与 code 分离开。.
实例
#!/usr/bin/ruby

(0..5).each do |i|
   puts "Value of local variable is #{i}"
end
这将产生以下结果：
Value of local variable is 0
Value of local variable is 1
Value of local variable is 2
Value of local variable is 3
Value of local variable is 4
Value of local variable is 5
Ruby break 语句
语法
break
终止最内部的循环。如果在块内调用，则终止相关块的方法（方法返回 nil）。
实例
#!/usr/bin/ruby

for i in 0..5
   if i > 2 then
      break
   end
   puts "Value of local variable is #{i}"
end
这将产生以下结果：
Value of local variable is 0
Value of local variable is 1
Value of local variable is 2
Ruby next 语句
语法
next
跳到最内部循环的下一个迭代。如果在块内调用，则终止块的执行（yield 或调用返回 nil）。
实例
#!/usr/bin/ruby

for i in 0..5
   if i < 2 then
      next
   end
   puts "Value of local variable is #{i}"
end
这将产生以下结果：
Value of local variable is 2
Value of local variable is 3
Value of local variable is 4
Value of local variable is 5
Ruby redo 语句
语法
redo
重新开始最内部循环的该次迭代，不检查循环条件。如果在块内调用，则重新开始 yield 或 call。
实例
#!/usr/bin/ruby

for i in 0..5
   if i < 2 then
      puts "Value of local variable is #{i}"
      redo
   end
end
这将产生以下结果，并会进入一个无限循环：
Value of local variable is 0
Value of local variable is 0
............................
Ruby retry 语句
语法
retry
如果 retry 出现在 begin 表达式的 rescue 子句中，则从 begin 主体的开头重新开始。
begin
   do_something # 抛出的异常
rescue
   # 处理错误
   retry  # 重新从 begin 开始
end
如果 retry 出现在迭代内、块内或者 for 表达式的主体内，则重新开始迭代调用。迭代的参数会重新评估。
for i in 1..5
   retry if some_condition # 重新从 i == 1 开始
end
实例
#!/usr/bin/ruby

for i in 1..5
   retry if  i > 2
   puts "Value of local variable is #{i}"
end
这将产生以下结果，并会进入一个无限循环：
		
Ruby 方法
Ruby 方法与其他编程语言中的函数类似。Ruby 方法用于捆绑一个或多个重复的语句到一个单元中。
方法名应以小写字母开头。如果您以大写字母作为方法名的开头，Ruby 可能会把它当作常量，从而导致不正确地解析调用。
方法应在调用之前定义，否则 Ruby 会产生未定义的方法调用异常。
语法
def method_name [( [arg [= default]]...[, * arg [, &expr ]])]
   expr..
end
所以，您可以定义一个简单的方法，如下所示：
def method_name 
   expr..
end
您可以定义一个接受参数的方法，如下所示：
def method_name (var1, var2)
   expr..
end
您可以为参数设置默认值，如果方法调用时未传递必需的参数则使用默认值：
def method_name (var1=value1, var2=value2)
   expr..
end
当您要调用方法时，只需要使用方法名即可，如下所示：
method_name
但是，当您调用带参数的方法时，您在写方法名时还要带上参数，例如：
method_name 25, 30
使用带参数方法最大的缺点是调用方法时需要记住参数个数。例如，如果您向一个接受三个参数的方法只传递了两个参数，Ruby 会显示错误。
实例
#!/usr/bin/ruby

def test(a1="Ruby", a2="Perl")
   puts "The programming language is #{a1}"
   puts "The programming language is #{a2}"
end
test "C", "C++"
test
这将产生以下结果：
The programming language is C
The programming language is C++
The programming language is Ruby
The programming language is Perl
从方法返回值
Ruby 中的每个方法默认都会返回一个值。这个返回的值是最后一个语句的值。例如：
def test
   i = 100
   j = 10
   k = 0
end
在调用这个方法时，将返回最后一个声明的变量 k。
Ruby return 语句
Ruby 中的 return 语句用于从 Ruby 方法中返回一个或多个值。
语法
return [expr[`,' expr...]]
如果给出超过两个的表达式，包含这些值的数组将是返回值。如果未给出表达式，nil 将是返回值。
实例
return

OR

return 12

OR

return 1,2,3
看看下面的实例：
#!/usr/bin/ruby

def test
   i = 100
   j = 200
   k = 300
return i, j, k
end
var = test
puts var
这将产生以下结果：
100
200
300
可变数量的参数
假设您声明了一个带有两个参数的方法，当您调用该方法时，您同时还需要传递两个参数。
但是，Ruby 允许您声明参数数量可变的方法。让我们看看下面的实例：
#!/usr/bin/ruby

def sample (*test)
   puts "The number of parameters is #{test.length}"
   for i in 0...test.length
      puts "The parameters are #{test[i]}"
   end
end
sample "Zara", "6", "F"
sample "Mac", "36", "M", "MCA"
在这段代码中，您已经声明了一个方法 sample，接受一个参数 test。但是，这个参数是一个变量参数。这意味着参数可以带有不同数量的变量。所以上面的代码将产生下面的结果：
The number of parameters is 3
The parameters are Zara
The parameters are 6
The parameters are F
The number of parameters is 4
The parameters are Mac
The parameters are 36
The parameters are M
The parameters are MCA
类方法
当方法定义在类定义外部时，方法默认标记为 private。另一方面，定义在类定义中的方法默认标记为 public。方法默认的可见性和 private 标记可通过模块（Module）的 public 或 private 改变。
当你想要访问类的方法时，您首先需要实例化类。然后，使用对象，您可以访问类的任何成员。
Ruby 提供了一种不用实例化类即可访问方法的方式。让我们看看如何声明并访问类方法：
class Accounts
   def reading_charge
   end
   def Accounts.return_date
   end
end
我们已经知道方法 return_date 是如何声明的。它是通过在类名后跟着一个点号，点号后跟着方法名来声明的。您可以直接访问类方法，如下所示：
Accounts.return_date
如需访问该方法，您不需要创建类 Accounts 的对象。
Ruby alias 语句
这个语句用于为方法或全局变量起别名。别名不能在方法主体内定义。即使方法被重写，方法的别名也保持方法的当前定义。
为编号的全局变量（$1, $2,...）起别名是被禁止的。重写内置的全局变量可能会导致严重的问题。
语法
alias method-name method-name
alias global-variable-name global-variable-name
实例
alias foo bar
alias $MATCH $&
在这里，我们已经为 bar 定义了别名为 foo，为 $& 定义了别名为 $MATCH。
Ruby undef 语句
这个语句用于取消方法定义。undef 不能出现在方法主体内。
通过使用 undef 和 alias，类的接口可以从父类独立修改，但请注意，在自身内部方法调用时，它可能会破坏程序。
语法
undef method-name
实例
下面的实例取消名为 bar的方法定义：
undef bar
		
Ruby 块
您已经知道 Ruby 如何定义方法以及您如何调用方法。类似地，Ruby 有一个块的概念。
* 块由大量的代码组成。
* 您需要给块取个名称。
* 块中的代码总是包含在大括号 {} 内。
* 块总是从与其具有相同名称的函数调用。这意味着如果您的块名称为 test，那么您要使用函数 test 来调用这个块。
* 您可以使用 yield 语句来调用块。
语法
block_name{
   statement1
   statement2
   ..........
}
在这里，您将学到如何使用一个简单的 yield 语句来调用块。您也将学到如何使用带有参数的 yield 语句来调用块。在实例中，您将看到这两种类型的 yield 语句。
yield 语句
让我们看一个 yield 语句的实例：
#!/usr/bin/ruby

def test
   puts "You are in the method"
   yield
   puts "You are again back to the method"
   yield
end
test {puts "You are in the block"}
这将产生以下结果：
You are in the method
You are in the block
You are again back to the method
You are in the block
您也可以传递带有参数的 yield 语句。下面是一个实例：
#!/usr/bin/ruby

def test
   yield 5
   puts "You are in the method test"
   yield 100
end
test {|i| puts "You are in the block #{i}"}
这将产生以下结果：
You are in the block 5
You are in the method test
You are in the block 100
在这里，yield 语句后跟着参数。您甚至可以传递多个参数。在块中，您可以在两个竖线之间放置一个变量来接受参数。因此，在上面的代码中，yield 5 语句向 test 块传递值 5 作为参数。
现在，看下面的语句：
test {|i| puts "You are in the block #{i}"}
在这里，值 5 会在变量 i 中收到。现在，观察下面的 puts 语句：
puts "You are in the block #{i}"
这个 puts 语句的输出是：
You are in the block 5
如果您想要传递多个参数，那么 yield 语句如下所示：
yield a, b
此时，块如下所示：
test {|a, b| statement}
参数使用逗号分隔。
块和方法
您已经看到块和方法之间是如何相互关联的。您通常使用 yield 语句从与其具有相同名称的方法调用块。因此，代码如下所示：
#!/usr/bin/ruby

def test
  yield
end
test{ puts "Hello world"}
本实例是实现块的最简单的方式。您使用 yield 语句调用 test 块。
但是如果方法的最后一个参数前带有 &，那么您可以向该方法传递一个块，且这个块可被赋给最后一个参数。如果 * 和 & 同时出现在参数列表中，& 应放在后面。
#!/usr/bin/ruby

def test(&block)
   block.call
end
test { puts "Hello World!"}
这将产生以下结果：
Hello World!
BEGIN 和 END 块
每个 Ruby 源文件可以声明当文件被加载时要运行的代码块（BEGIN 块），以及程序完成执行后要运行的代码块（END 块）。
#!/usr/bin/ruby

BEGIN { 
  # BEGIN block code 
  puts "BEGIN code block"
} 

END { 
  # END block code 
  puts "END code block"
}
  # MAIN block code 
puts "MAIN code block"
一个程序可以包含多个 BEGIN 和 END 块。BEGIN 块按照它们出现的顺序执行。END 块按照它们出现的相反顺序执行。当执行时，上面的程序产生产生以下结果：
BEGIN code block
MAIN code block
END code block
		
Ruby 模块（Module）
模块（Module）是一种把方法、类和常量组合在一起的方式。模块（Module）为您提供了两大好处。
* 模块提供了一个命名空间和避免名字冲突。
* 模块实现了 mixin 装置。
模块（Module）定义了一个命名空间，相当于一个沙箱，在里边您的方法和常量不会与其他地方的方法常量冲突。
语法
module Identifier
   statement1
   statement2
   ...........
end
模块常量命名与类常量命名类似，以大写字母开头。方法定义看起来也相似：模块方法定义与类方法定义类似。
通过类方法，您可以在类方法名称前面放置模块名称和一个点号来调用模块方法，您可以使用模块名称和两个冒号来引用一个常量。
实例
#!/usr/bin/ruby

# 定义在 trig.rb 文件中的模块

module Trig
   PI = 3.141592654
   def Trig.sin(x)
   # ..
   end
   def Trig.cos(x)
   # ..
   end
end
我们可以定义多个函数名称相同但是功能不同的模块：
#!/usr/bin/ruby

# 定义在 moral.rb 文件中的模块

module Moral
   VERY_BAD = 0
   BAD = 1
   def Moral.sin(badness)
   # ...
   end
end
就像类方法，当您在模块中定义一个方法时，您可以指定在模块名称后跟着一个点号，点号后跟着方法名。
Ruby require 语句
require 语句类似于 C 和 C++ 中的 include 语句以及 Java 中的 import 语句。如果一个第三方的程序想要使用任何已定义的模块，则可以简单地使用 Ruby require 语句来加载模块文件：
语法
require filename
在这里，文件扩展名 .rb 不是必需的。
实例
$LOAD_PATH << '.'

require 'trig.rb'
require 'moral'

y = Trig.sin(Trig::PI/4)
wrongdoing = Moral.sin(Moral::VERY_BAD)
在这里，我们使用 $LOAD_PATH << '.' 让 Ruby 知道必须在当前目录中搜索被引用的文件。如果您不想使用 $LOAD_PATH，那么您可以使用 require_relative 来从一个相对目录引用文件。
注意：在这里，文件包含相同的函数名称。所以，这会在引用调用程序时导致代码模糊，但是模块避免了这种代码模糊，而且我们可以使用模块的名称调用适当的函数。
Ruby include 语句
您可以在类中嵌入模块。为了在类中嵌入模块，您可以在类中使用 include 语句：
语法
include modulename
如果模块是定义在一个单独的文件中，那么在嵌入模块之前使用 require 语句引用该文件时必需的。
实例
假设下面的模块写在 support.rb 文件中。
module Week
   FIRST_DAY = "Sunday"
   def Week.weeks_in_month
      puts "You have four weeks in a month"
   end
   def Week.weeks_in_year
      puts "You have 52 weeks in a year"
   end
end
现在，您可以在类中引用该模块，如下所示：
#!/usr/bin/ruby
$LOAD_PATH << '.'
require "support"

class Decade
include Week
   no_of_yrs=10
   def no_of_months
      puts Week::FIRST_DAY
      number=10*12
      puts number
   end
end
d1=Decade.new
puts Week::FIRST_DAY
Week.weeks_in_month
Week.weeks_in_year
d1.no_of_months
这将产生以下结果：
Sunday
You have four weeks in a month
You have 52 weeks in a year
Sunday
120
Ruby 中的 Mixins
在阅读本节之前，您需要初步了解面向对象的概念。
当一个类可以从多个父类继承类的特性时，该类显示为多重继承。
Ruby 不直接支持多重继承，但是 Ruby 的模块（Module）有另一个神奇的功能。它几乎消除了多重继承的需要，提供了一种名为 mixin 的装置。
Mixins 向您提供了一种完美的为类添加功能的控制方式。但是，它们真正的强大在于当 mixin 中的代码开始与使用它的类中的代码交互时。
让我们看看下面的示例代码，深入了解 mixin：
module A
   def a1
   end
   def a2
   end
end
module B
   def b1
   end
   def b2
   end
end

class Sample
include A
include B
   def s1
   end
end

samp=Sample.new
samp.a1
samp.a2
samp.b1
samp.b2
samp.s1
模块 A 由方法 a1 和 a2 组成。模块 B 由方法 b1 和 b2 组成。类 Sample 包含了模块 A 和 B。类 Sample 可以访问所有四个方法，即 a1、a2、b1 和 b2。因此，您可以看到类 Sample 继承了两个模块。因此，您可以说类 Sample 显示了多重继承或 mixin 。
		
Ruby 字符串（String）
Ruby 中的 String 对象存储并操作一个或多个字节的任意序列，通常表示那些代表人类语言的字符。
最简单的字符串是括在单引号（单引号字符）内。在引号标记内的文本是字符串的值：
'This is a simple Ruby string literal'
如果您需要在单引号字符串内使用单引号字符，那么需要在单引号字符串使用反斜杠，这样 Ruby 解释器就不会认为这个单引号字符会终止字符串：
'Won\'t you read O\'Reilly\'s book?'
反斜杠也能转义另一个反斜杠，这样第二个反斜杠本身不会解释为转义字符。
以下是 Ruby 中字符串相关的特性。
表达式置换
表达式置换是一种使用 #{ 和 } 把任意 Ruby 表达式的值嵌入到字符串中的方式：
#!/usr/bin/ruby

x, y, z = 12, 36, 72
puts "The value of x is #{ x }."
puts "The sum of x and y is #{ x + y }."
puts "The average was #{ (x + y + z)/3 }."
这将产生以下结果：
The value of x is 12.
The sum of x and y is 48.
The average was 40.
一般的分隔字符串
通过一般的分隔字符串，您可以在以百分号字符（%）为前导的一对匹配的任意分隔字符（例如，!、 (、 {、 <，等等）内创建字符串。Q、 q 和 x 有特殊的意义。一般的分隔字符串可以是：
%{Ruby is fun.}  相当于 "Ruby is fun."
%Q{ Ruby is fun. } 相当于 " Ruby is fun. "
%q[Ruby is fun.]  相当于以单引号字符串
%x!ls! 相当于反勾号命令输出 `ls`
转义字符
下标列出了可使用反斜杠符号转义的转义字符或非打印字符。
注意：在一个双引号括起的字符串内，转义字符会被解释；在一个单引号括起的字符串内，转义字符会被保留。
反斜杠符号	十六进制字符	描述
\a	0x07	报警符
\b	0x08	退格键
\cx		Control-x
\C-x		Control-x
\e	0x1b	转义符
\f	0x0c	换页符
\M-\C-x		Meta-Control-x
\n	0x0a	换行符
\nnn		八进制表示法，其中 n 的范围为 0.7
\r	0x0d	回车符
\s	0x20	空格符
\t	0x09	制表符
\v	0x0b	垂直制表符
\x		字符 x
\xnn		十六进制表示法，其中 n 的范围为 0.9、 a.f 或 A.F
字符编码
Ruby 的默认字符集是 ASCII，字符可用单个字节表示。如果您使用 UTF-8 或其他现代的字符集，字符可能是用一个到四个字节表示。
您可以在程序开头使用 $KCODE 改变字符集，如下所示：
$KCODE = 'u'
下面是 $KCODE 可能的值。
编码	描述
a	ASCII （与 none 相同）。这是默认的。
e	EUC。
n	None （与 ASCII 相同）。
u	UTF-8。
字符串内建方法
我们需要有一个 String 对象的实例来调用 String 方法。下面是创建 String 对象实例的方式：
new [String.new(str="")]
这将返回一个包含 str 副本的新的字符串对象。现在，使用 str 对象，我们可以调用任意可用的实例方法。例如：
#!/usr/bin/ruby

myStr = String.new("THIS IS TEST")
foo = myStr.downcase

puts "#{foo}"
这将产生以下结果：
this is test
下面是公共的字符串方法（假设 str 是一个 String 对象）：
序号	方法 & 描述
1	str % arg
使用格式规范格式化字符串。如果 arg 包含一个以上的替代，那么 arg 必须是一个数组。如需了解更多格式规范的信息，请查看"内核模块"下的 sprintf。
2	str * integer
返回一个包含 integer 个 str 的新的字符串。换句话说，str 被重复了 integer 次。
3	str + other_str
连接 other_str 到 str。
4	str << obj
连接一个对象到字符串。如果对象是范围为 0.255 之间的固定数字 Fixnum，则它会被转换为一个字符。把它与 concat 进行比较。
5	str <=> other_str
把 str 与 other_str 进行比较，返回 -1（小于）、0（等于）或 1（大于）。比较是区分大小写的。
6	str == obj
检查 str 和 obj 的相等性。如果 obj 不是字符串，则返回 false，如果 str <=> obj，则返回 true，返回 0。
7	str =~ obj
根据正则表达式模式 obj 匹配 str。返回匹配开始的位置，否则返回 false。
8	str =~ obj
根据正则表达式模式 obj 匹配 str。返回匹配开始的位置，否则返回 false。
9	str.capitalize
把字符串转换为大写字母显示。
10	str.capitalize!
与 capitalize 相同，但是 str 会发生变化并返回。
11	str.casecmp
不区分大小写的字符串比较。
12	str.center
居中字符串。
13	str.chomp
从字符串末尾移除记录分隔符（$/），通常是 \n。如果没有记录分隔符，则不进行任何操作。
14	str.chomp!
与 chomp 相同，但是 str 会发生变化并返回。
15	str.chop
移除 str 中的最后一个字符。
16	str.chop!
与 chop 相同，但是 str 会发生变化并返回。
17	str.concat(other_str)
连接 other_str 到 str。
18	str.count(str, ...)
给一个或多个字符集计数。如果有多个字符集，则给这些集合的交集计数。
19	str.crypt(other_str)
对 str 应用单向加密哈希。参数是两个字符长的字符串，每个字符的范围为 a.z、 A.Z、 0.9、 . 或 /。
20	str.delete(other_str, ...)
返回 str 的副本，参数交集中的所有字符会被删除。
21	str.delete!(other_str, ...)
与 delete 相同，但是 str 会发生变化并返回。
22	str.downcase
返回 str 的副本，所有的大写字母会被替换为小写字母。
23	str.downcase!
与 downcase 相同，但是 str 会发生变化并返回。
24	str.dump
返回 str 的版本，所有的非打印字符被替换为 \nnn 符号，所有的特殊字符被转义。
25	str.each(separator=$/) { |substr| block }
使用参数作为记录分隔符（默认是 $/）分隔 str，传递每个子字符串给被提供的块。
26	str.each_byte { |fixnum| block }
传递 str 的每个字节给 block，以字节的十进制表示法返回每个字节。
27	str.each_line(separator=$/) { |substr| block }
使用参数作为记录分隔符（默认是 $/）分隔 str，传递每个子字符串给被提供的 block。
28	str.empty?
如果 str 为空（即长度为 0），则返回 true。
29	str.eql?(other)
如果两个字符串有先攻的长度和内容，则这两个字符串相等。
30	str.gsub(pattern, replacement) [or]
str.gsub(pattern) { |match| block }
返回 str 的副本，pattern 的所有出现都替换为 replacement 或 block 的值。pattern 通常是一个正则表达式 Regexp；如果是一个字符串 String，则没有正则表达式元字符被解释（即，/\d/ 将匹配一个数字，但 '\d' 将匹配一个反斜杠后跟一个 'd'）。
31	str[fixnum] [or] str[fixnum,fixnum] [or] str[range] [or] str[regexp] [or] str[regexp, fixnum] [or] str[other_str]
使用下列的参数引用 str：参数为一个 Fixnum，则返回 fixnum 的字符编码；参数为两个 Fixnum，则返回一个从偏移（第一个 fixnum）开始截至到长度（第二个 fixnum）为止的子字符串；参数为 range，则返回该范围内的一个子字符串；参数为 regexp，则返回匹配字符串的部分；参数为带有 fixnum 的 regexp，则返回 fixnum 位置的匹配数据；参数为 other_str，则返回匹配 other_str 的子字符串。一个负数的 Fixnum 从字符串的末尾 -1 开始。
32	str[fixnum] = fixnum [or] str[fixnum] = new_str [or] str[fixnum, fixnum] = new_str [or] str[range] = aString [or] str[regexp] =new_str [or] str[regexp, fixnum] =new_str [or] str[other_str] = new_str ]
替换整个字符串或部分字符串。与 slice! 同义。
33	str.gsub!(pattern, replacement) [or] str.gsub!(pattern) { |match| block }
执行 String#gsub 的替换，返回 str，如果没有替换被执行则返回 nil。
34	str.hash
返回一个基于字符串长度和内容的哈希。
35	str.hex
把 str 的前导字符当作十六进制数字的字符串（一个可选的符号和一个可选的 0x），并返回相对应的数字。如果错误则返回零。
36	str.include? other_str [or] str.include? fixnum
如果 str 包含给定的字符串或字符，则返回 true。
37	str.index(substring [, offset]) [or]
str.index(fixnum [, offset]) [or]
str.index(regexp [, offset])
返回给定子字符串、字符（fixnum）或模式（regexp）在 str 中第一次出现的索引。如果未找到则返回 nil。如果提供了第二个参数，则指定在字符串中开始搜索的位置。
38	str.insert(index, other_str)
在给定索引的字符前插入 other_str，修改 str。负值索引从字符串的末尾开始计数，并在给定字符后插入。其意图是在给定的索引处开始插入一个字符串。
39	str.inspect
返回 str 的可打印版本，带有转义的特殊字符。
40	str.intern [or] str.to_sym
返回与 str 相对应的符号，如果之前不存在，则创建符号。
41	str.length
返回 str 的长度。把它与 size 进行比较。
42	str.ljust(integer, padstr=' ')
如果 integer 大于 str 的长度，则返回长度为 integer 的新字符串，新字符串以 str 左对齐，并以 padstr 作为填充。否则，返回 str。
43	str.lstrip
返回 str 的副本，移除了前导的空格。
44	str.lstrip!
从 str 中移除前导的空格，如果没有变化则返回 nil。
45	str.match(pattern)
如果 pattern 不是正则表达是，则把 pattern 转换为正则表达式 Regexp，然后在 str 上调用它的匹配方法。
46	str.oct
把 str 的前导字符当作十进制数字的字符串（一个可选的符号），并返回相对应的数字。如果转换失败，则返回 0。
47	str.replace(other_str)
把 str 中的内容替换为 other_str 中的相对应的值。
48	str.reverse
返回一个新字符串，新字符串是 str 的倒序。
49	str.reverse!
逆转 str，str 会发生变化并返回。
50	str.rindex(substring [, fixnum]) [or]
str.rindex(fixnum [, fixnum]) [or]
str.rindex(regexp [, fixnum])
返回给定子字符串、字符（fixnum）或模式（regexp）在 str 中最后一次出现的索引。如果未找到则返回 nil。如果提供了第二个参数，则指定在字符串中结束搜索的位置。超出该点的字符将不被考虑。
51	str.rjust(integer, padstr=' ')
如果 integer 大于 str 的长度，则返回长度为 integer 的新字符串，新字符串以 str 右对齐，并以 padstr 作为填充。否则，返回 str。
52	str.rstrip
返回 str 的副本，移除了尾随的空格。
53	str.rstrip!
从 str 中移除尾随的空格，如果没有变化则返回 nil。
54	str.scan(pattern) [or]
str.scan(pattern) { |match, ...| block }
两种形式匹配 pattern（可以是一个正则表达式 Regexp 或一个字符串 String）遍历 str。针对每个匹配，会生成一个结果，结果会添加到结果数组中或传递给 block。如果 pattern 不包含分组，则每个独立的结果由匹配的字符串、$& 组成。如果 pattern 包含分组，每个独立的结果是一个包含每个分组入口的数组。
55	str.slice(fixnum) [or] str.slice(fixnum, fixnum) [or]
str.slice(range) [or] str.slice(regexp) [or]
str.slice(regexp, fixnum) [or] str.slice(other_str)
See str[fixnum], etc.
str.slice!(fixnum) [or] str.slice!(fixnum, fixnum) [or] str.slice!(range) [or] str.slice!(regexp) [or] str.slice!(other_str)
从 str 中删除指定的部分，并返回删除的部分。如果值超出范围，参数带有 Fixnum 的形式，将生成一个 IndexError。参数为 range 的形式，将生成一个 RangeError，参数为 Regexp 和 String 的形式，将忽略执行动作。
56	str.split(pattern=$;, [limit])
基于分隔符，把 str 分成子字符串，并返回这些子字符串的数组。
如果 pattern 是一个字符串 String，那么在分割 str 时，它将作为分隔符使用。如果 pattern 是一个单一的空格，那么 str 是基于空格进行分割，会忽略前导空格和连续空格字符。
如果 pattern 是一个正则表达式 Regexp，则 str 在 pattern 匹配的地方被分割。当 pattern 匹配一个玲长度的字符串时，str 被分割成单个字符。
如果省略了 pattern 参数，则使用 $; 的值。如果 $; 为 nil（默认的），str 基于空格进行分割，就像是指定了 ` ` 作为分隔符一样。
如果省略了 limit 参数，会抑制尾随的 null 字段。如果 limit 是一个正数，则最多返回该数量的字段（如果 limit 为 1，则返回整个字符串作为数组中的唯一入口）。如果 limit 是一个负数，则返回的字段数量不限制，且不抑制尾随的 null 字段。
57	str.squeeze([other_str]*)
使用为 String#count 描述的程序从 other_str 参数建立一系列字符。返回一个新的字符串，其中集合中出现的相同的字符会被替换为单个字符。如果没有给出参数，则所有相同的字符都被替换为单个字符。
58	str.squeeze!([other_str]*)
与 squeeze 相同，但是 str 会发生变化并返回，如果没有变化则返回 nil。
59	str.strip
返回 str 的副本，移除了前导的空格和尾随的空格。
60	str.strip!
从 str 中移除前导的空格和尾随的空格，如果没有变化则返回 nil。
61	str.sub(pattern, replacement) [or]
str.sub(pattern) { |match| block }
返回 str 的副本，pattern 的第一次出现会被替换为 replacement 或 block 的值。pattern 通常是一个正则表达式 Regexp；如果是一个字符串 String，则没有正则表达式元字符被解释。
62	str.sub!(pattern, replacement) [or]
str.sub!(pattern) { |match| block }
执行 String#sub 替换，并返回 str，如果没有替换执行，则返回 nil。
63	str.succ [or] str.next
返回 str 的继承。
64	str.succ! [or] str.next!
相当于 String#succ，但是 str 会发生变化并返回。
65	str.sum(n=16)
返回 str 中字符的 n-bit 校验和，其中 n 是可选的 Fixnum 参数，默认为 16。结果是简单地把 str 中每个字符的二进制值的总和，以 2n - 1 为模。这不是一个特别好的校验和。
66	str.swapcase
返回 str 的副本，所有的大写字母转换为小写字母，所有的小写字母转换为大写字母。
67	str.swapcase!
相当于 String#swapcase，但是 str 会发生变化并返回，如果没有变化则返回 nil。
68	str.to_f
返回把 str 中的前导字符解释为浮点数的结果。超出有效数字的末尾的多余字符会被忽略。如果在 str 的开头没有有效数字，则返回 0.0。该方法不会生成异常。
69	str.to_i(base=10)
返回把 str 中的前导字符解释为整数基数（基数为 2、 8、 10 或 16）的结果。超出有效数字的末尾的多余字符会被忽略。如果在 str 的开头没有有效数字，则返回 0。该方法不会生成异常。
70	str.to_s [or] str.to_str
返回接收的值。
71	str.tr(from_str, to_str)
返回 str 的副本，把 from_str 中的字符替换为 to_str 中相对应的字符。如果 to_str 比 from_str 短，那么它会以最后一个字符进行填充。两个字符串都可以使用 c1.c2 符号表示字符的范围。如果 from_str 以 ^ 开头，则表示除了所列出的字符以外的所有字符。
72	str.tr!(from_str, to_str)
相当于 String#tr，但是 str 会发生变化并返回，如果没有变化则返回 nil。
73	str.tr_s(from_str, to_str)
把 str 按照 String#tr 描述的规则进行处理，然后移除会影响翻译的重复字符。
74	str.tr_s!(from_str, to_str)
相当于 String#tr_s，但是 str 会发生变化并返回，如果没有变化则返回 nil。
75	str.unpack(format)
根据 format 字符串解码 str（可能包含二进制数据），返回被提取的每个值的数组。format 字符由一系列单字符指令组成。每个指令后可以跟着一个数字，表示重复该指令的次数。星号（*）将使用所有剩余的元素。指令 sSiIlL 每个后可能都跟着一个下划线（_），为指定类型使用底层平台的本地尺寸大小，否则使用独立于平台的一致的尺寸大小。format 字符串中的空格会被忽略。
76	str.upcase
返回 str 的副本，所有的小写字母会被替换为大写字母。操作是环境不敏感的，只有字符 a 到 z 会受影响。
77	str.upcase!
改变 str 的内容为大写，如果没有变化则返回 nil。
78	str.upto(other_str) { |s| block }
遍历连续值，以 str 开始，以 other_str 结束（包含），轮流传递每个值给 block。String#succ 方法用于生成每个值。
字符串 unpack 指令
下表列出了方法 String#unpack 的解压指令。
指令	返回	描述
A	String	移除尾随的 null 和空格。
a	String	字符串。
B	String	从每个字符中提取位（首先是最高有效位）。
b	String	从每个字符中提取位（首先是最低有效位）。
C	Fixnum	提取一个字符作为无符号整数。
c	Fixnum	提取一个字符作为整数。
D, d	Float	把 sizeof(double) 长度的字符当作原生的 double。
E	Float	把 sizeof(double) 长度的字符当作 littleendian 字节顺序的 double。
e	Float	把 sizeof(float) 长度的字符当作 littleendian 字节顺序的 float。
F, f	Float	把 sizeof(float) 长度的字符当作原生的 float。
G	Float	把 sizeof(double) 长度的字符当作 network 字节顺序的 double。
g	Float	把 sizeof(float) 长度的字符当作 network 字节顺序的 float。
H	String	从每个字符中提取十六进制（首先是最高有效位）。
h	String	从每个字符中提取十六进制（首先是最低有效位）。
I	Integer	把 sizeof(int) 长度（通过 _ 修改）的连续字符当作原生的 integer。
i	Integer	把 sizeof(int) 长度（通过 _ 修改）的连续字符当作有符号的原生的 integer。
L	Integer	把四个（通过 _ 修改）连续字符当作无符号的原生的 long integer。
l	Integer	把四个（通过 _ 修改）连续字符当作有符号的原生的 long integer。
M	String	引用可打印的。
m	String	Base64 编码。
N	Integer	把四个字符当作 network 字节顺序的无符号的 long。
n	Fixnum	把两个字符当作 network 字节顺序的无符号的 short。
P	String	把 sizeof(char *) 长度的字符当作指针，并从引用的位置返回 \emph{len} 字符。
p	String	把 sizeof(char *) 长度的字符当作一个空结束字符的指针。
Q	Integer	把八个字符当作无符号的 quad word（64 位）。
q	Integer	把八个字符当作有符号的 quad word（64 位）。
S	Fixnum	把两个（如果使用 _ 则不同）连续字符当作 native 字节顺序的无符号的 short。
s	Fixnum	把两个（如果使用 _ 则不同）连续字符当作 native 字节顺序的有符号的 short。
U	Integer	UTF-8 字符，作为无符号整数。
u	String	UU 编码。
V	Fixnum	把四个字符当作 little-endian 字节顺序的无符号的 long。
v	Fixnum	把两个字符当作 little-endian 字节顺序的无符号的 short。
w	Integer	BER 压缩的整数。
X		向后跳过一个字符。
x		向前跳过一个字符。
Z	String	和 * 一起使用，移除尾随的 null 直到第一个 null。
@		跳过 length 参数给定的偏移量。
实例
尝试下面的实例，解压各种数据。
"abc \0\0abc \0\0".unpack('A6Z6')   #=> ["abc", "abc "]
"abc \0\0".unpack('a3a3')           #=> ["abc", " \000\000"]
"abc \0abc \0".unpack('Z*Z*')       #=> ["abc ", "abc "]
"aa".unpack('b8B8')                 #=> ["10000110", "01100001"]
"aaa".unpack('h2H2c')               #=> ["16", "61", 97]
"\xfe\xff\xfe\xff".unpack('sS')     #=> [-2, 65534]
"now=20is".unpack('M*')             #=> ["now is"]
"whole".unpack('xax2aX2aX1aX2a')    #=> ["h", "e", "l", "l", "o"]
		
Ruby 数组（Array）
Ruby 数组是任何对象的有序的、整数索引的集合。数组中的每个元素都与一个索引相关，并可通过索引进行获取。
数组的索引从 0 开始，这与 C 或 Java 中一样。一个负数的索引时相对于数组的末尾计数的，也就是说，索引为 -1 表示数组的最后一个元素，-2 表示数组中的倒数第二个元素，依此类推。
Ruby 数组可存储诸如 String、 Integer、 Fixnum、 Hash、 Symbol 等对象，甚至可以是其他 Array 对象。Ruby 数组不像其他语言中的数组那么刚性。当向数组添加元素时，Ruby 数组会自动增长。
创建数组
有多种方式创建或初始化数组。一种方式是通过 new 类方法：
names = Array.new
您可以在创建数组的同时设置数组的大小：
names = Array.new(20)
数组 names 的大小或长度为 20 个元素。您可以使用 size 或 length 方法返回数组的大小：
#!/usr/bin/ruby

names = Array.new(20)
puts names.size  # 返回 20
puts names.length # 返回 20
这将产生以下结果：
20
20
您可以给数组中的每个元素赋值，如下所示：
#!/usr/bin/ruby

names = Array.new(4, "mac")

puts "#{names}"
这将产生以下结果：
macmacmacmac
您也可以使用带有 new 的块，每个元素使用块中的计算结果来填充：
#!/usr/bin/ruby

nums = Array.new(10) { |e| e = e * 2 }

puts "#{nums}"
这将产生以下结果：
024681012141618
数组还有另一种方法，[]，如下所示：
nums = Array.[](1, 2, 3, 4,5)
数组创建的另一种形式如下所示：
nums = Array[1, 2, 3, 4,5]
在核心 Ruby 中可用的 Kernel 模块有一个 Array 方法，只接受单个参数。在这里，该方法带有一个范围作为参数来创建一个数字数组：
#!/usr/bin/ruby

digits = Array(0..9)

puts "#{digits}"
这将产生以下结果：
0123456789
数组内建方法
我们需要有一个 Array 对象的实例来调用 Array 方法。下面是创建 Array 对象实例的方式：
Array.[](...) [or] Array[...] [or] [...]
这将返回一个使用给定对象进行填充的新的数组。现在，使用创建的对象，我们可以调用任意可用的实例方法。例如：
#!/usr/bin/ruby

digits = Array(0..9)

num = digits.at(6)

puts "#{num}"
这将产生以下结果：
6
下面是公共的数组方法（假设 array 是一个 Array 对象）：
序号	方法 & 描述
1	array & other_array
返回一个新的数组，包含两个数组中共同的元素，没有重复。
2	array * int [or] array * str
返回一个新的数组，新数组通过连接 self 的 int 副本创建的。带有 String 参数时，相当于 self.join(str)。
3	array + other_array
返回一个新的数组，新数组通过连接两个数组产生第三个数组创建的。
4	array - other_array
返回一个新的数组，新数组是从初始数组中移除了在 other_array 中出现的项的副本。
5	str <=> other_str
把 str 与 other_str 进行比较，返回 -1（小于）、0（等于）或 1（大于）。比较是区分大小写的。
6	array | other_array
通过把 other_array 加入 array 中，移除重复项，返回一个新的数组。
7	array << obj
把给定的对象附加到数组的末尾。该表达式返回数组本身，所以几个附加可以连在一起。
8	array <=> other_array
如果数组小于、等于或大于 other_array，则返回一个整数（-1、 0 或 +1）。
9	array == other_array
如果两个数组包含相同的元素个数，且每个元素与另一个数组中相对应的元素相等（根据 Object.==），那么这两个数组相等。
10	array[index] [or] array[start, length] [or]
array[range] [or] array.slice(index) [or]
array.slice(start, length) [or] array.slice(range)
返回索引为 index 的元素，或者返回从 start 开始直至 length 个元素的子数组，或者返回 range 指定的子数组。负值索引从数组末尾开始计数（-1 是最后一个元素）。如果 index（或开始索引）超出范围，则返回 nil。
11	array[index] = obj [or]
array[start, length] = obj or an_array or nil [or]
array[range] = obj or an_array or nil
设置索引为 index 的元素，或者替换从 start 开始直至 length 个元素的子数组，或者替换 range 指定的子数组。如果索引大于数组的当前容量，那么数组会自动增长。负值索引从数组末尾开始计数。如果 length 为零则插入元素。如果在第二种或第三种形式中使用了 nil，则从 self 删除元素。
12	array.abbrev(pattern = nil)
为 self 中的字符串计算明确的缩写集合。如果传递一个模式或一个字符串，只考虑当字符串匹配模式或者以该字符串开始时的情况。
13	array.assoc(obj)
搜索一个数组，其元素也是数组，使用 obj.== 把 obj 与每个包含的数组的第一个元素进行比较。如果匹配则返回第一个包含的数组，如果未找到匹配则返回 nil。
14	array.at(index)
返回索引为 index 的元素。一个负值索引从 self 的末尾开始计数。如果索引超出范围则返回 nil。
15	array.clear
从数组中移除所有的元素。
16	array.collect { |item| block } [or]
array.map { |item| block }
为 self 中的每个元素调用一次 block。创建一个新的数组，包含 block 返回的值。
17	array.collect! { |item| block } [or]
array.map! { |item| block }
为 self 中的每个元素调用一次 block，把元素替换为 block 返回的值。
18	array.compact
返回 self 的副本，移除了所有的 nil 元素。
19	array.compact!
从数组中移除所有的 nil 元素。如果没有变化则返回 nil。
20	array.concat(other_array)
追加 other_array 中的元素到 self 中。
21	array.delete(obj) [or] 
array.delete(obj) { block }
从 self 中删除等于 obj 的项。如果未找到相等项，则返回 nil。如果未找到相等项且给出了可选的代码 block，则返回 block 的结果。
22	array.delete_at(index)
删除指定的 index 处的元素，并返回该元素。如果 index 超出范围，则返回 nil。
23	array.delete_if { |item| block }
当 block 为 true 时，删除 self 的每个元素。
24	array.each { |item| block }
为 self 中的每个元素调用一次 block，传递该元素作为参数。
25	array.each_index { |index| block }
与 Array#each 相同，但是传递元素的 index，而不是传递元素本身。
26	array.empty?
如果数组本身没有包含元素，则返回 true。
27	array.eql?(other)
如果 array 和 other 是相同的对象，或者两个数组带有相同的内容，则返回 true。
28	array.fetch(index) [or] 
array.fetch(index, default) [or] 
array.fetch(index) { |index| block }
尝试返回位置 index 处的元素。如果 index 位于数组外部，则第一种形式会抛出 IndexError 异常，第二种形式会返回 default，第三种形式会返回调用 block 传入 index 的值。负值的 index 从数组末尾开始计数。
29	array.fill(obj) [or]
array.fill(obj, start [, length]) [or]
array.fill(obj, range) [or]
array.fill { |index| block } [or]
array.fill(start [, length] ) { |index| block } [or]
array.fill(range) { |index| block }
前面三种形式设置 self 的被选元素为 obj。以 nil 开头相当于零。nil 的长度相当于 self.length。最后三种形式用 block 的值填充数组。block 通过带有被填充的每个元素的绝对索引来传递。
30	array.first [or] 
array.first(n)
返回数组的第一个元素或前 n 个元素。如果数组为空，则第一种形式返回 nil，第二种形式返回一个空的数组。
31	array.flatten
返回一个新的数组，新数组是一个一维的扁平化的数组（递归）。
32	array.flatten!
把 array 进行扁平化。如果没有变化则返回 nil。（数组不包含子数组。）
33	array.frozen?
如果 array 被冻结（或排序时暂时冻结），则返回 true。
34	array.hash
计算数组的哈希代码。两个具有相同内容的数组将具有相同的哈希代码。
35	array.include?(obj)
如果 self 中包含 obj，则返回 true，否则返回 false。
36	array.index(obj)
返回 self 中第一个等于 obj 的对象的 index。如果未找到匹配则返回 nil。
37	array.indexes(i1, i2, ... iN) [or]
array.indices(i1, i2, ... iN)
该方法在 Ruby 的最新版本中被废弃，所以请使用 Array#values_at。
38	array.indices(i1, i2, ... iN) [or]
array.indexes(i1, i2, ... iN)
该方法在 Ruby 的最新版本中被废弃，所以请使用 Array#values_at。
39	array.insert(index, obj...)
在给定的 index 的元素前插入给定的值，index 可以是负值。
40	array.inspect
创建一个数组的可打印版本。
41	array.join(sep=$,)
返回一个字符串，通过把数组的每个元素转换为字符串，并使用 sep 分隔进行创建的。
42	array.last [or] array.last(n)
返回 self 的最后一个元素。如果数组为空，则第一种形式返回 nil。
43	array.length
返回 self 中元素的个数。可能为零。
44	array.map { |item| block } [or]
array.collect { |item| block }
为 self 的每个元素调用一次 block。创建一个新的数组，包含 block 返回的值。
45	array.map! { |item| block } [or]
array.collect! { |item| block }
为 array 的每个元素调用一次 block，把元素替换为 block 返回的值。
46	array.nitems
返回 self 中 non-nil 元素的个数。可能为零。
47	array.pack(aTemplateString)
根据 aTemplateString 中的指令，把数组的内容压缩为二进制序列。指令 A、 a 和 Z 后可以跟一个表示结果字段宽度的数字。剩余的指令也可以带有一个表示要转换的数组元素个数的数字。如果数字是一个星号（*），则所有剩余的数组元素都将被转换。任何指令后都可以跟一个下划线（_），表示指定类型使用底层平台的本地尺寸大小，否则使用独立于平台的一致的尺寸大小。在模板字符串中空格会被忽略。
48	array.pop
从 array 中移除最后一个元素，并返回该元素。如果 array 为空则返回 nil。
49	array.push(obj, ...)
把给定的 obj 附加到数组的末尾。该表达式返回数组本身，所以几个附加可以连在一起。
50	array.rassoc(key)
搜索一个数组，其元素也是数组，使用 == 把 key 与每个包含的数组的第二个元素进行比较。如果匹配则返回第一个包含的数组。
51	array.reject { |item| block }
返回一个新的数组，包含当 block 不为 true 时的数组项。
52	array.reject! { |item| block }
当 block 为真时，从 array 删除元素，如果没有变化则返回 nil。相当于 Array#delete_if。
53	array.replace(other_array)
把 array 的内容替换为 other_array 的内容，必要的时候进行截断或扩充。
54	array.reverse
返回一个新的数组，包含倒序排列的数组元素。
55	array.reverse!
把 array 进行逆转。
56	array.reverse_each {|item| block }
与 Array#each 相同，但是把 array 进行逆转。
57	array.rindex(obj)
返回 array 中最后一个等于 obj 的对象的索引。如果未找到匹配，则返回 nil。
58	array.select {|item| block }
调用从数组传入连续元素的 block，返回一个数组，包含 block 返回 true 值时的元素。
59	array.shift
返回 self 的第一个元素，并移除该元素（把所有的其他元素下移一位）。如果数组为空，则返回 nil。
60	array.size
返回 array 的长度（元素的个数）。length 的别名。
61	array.slice(index) [or] array.slice(start, length) [or]
array.slice(range) [or] array[index] [or]
array[start, length] [or] array[range]
返回索引为 index 的元素，或者返回从 start 开始直至 length 个元素的子数组，或者返回 range 指定的子数组。负值索引从数组末尾开始计数（-1 是最后一个元素）。如果 index（或开始索引）超出范围，则返回 nil。
62	array.slice!(index) [or] array.slice!(start, length) [or]
array.slice!(range)
删除 index（长度是可选的）或 range 指定的元素。返回被删除的对象、子数组，如果 index 超出范围，则返回 nil。
63	array.sort [or] array.sort { | a,b | block }
返回一个排序的数组。
64	array.sort! [or] array.sort! { | a,b | block }
把数组进行排序。
65	array.to_a
返回 self。如果在 Array 的子类上调用，则把接收参数转换为一个 Array 对象。
66	array.to_ary
返回 self。
67	array.to_s
返回 self.join。
68	array.transpose
假设 self 是数组的数组，且置换行和列。
69	array.uniq
返回一个新的数组，移除了 array 中的重复值。
70	array.uniq!
从 self 中移除重复元素。如果没有变化（也就是说，未找到重复），则返回 nil。
71	array.unshift(obj, ...)
把对象前置在数组的前面，其他元素上移一位。
72	array.values_at(selector,...)
返回一个数组，包含 self 中与给定的 selector（一个或多个）相对应的元素。选择器可以是整数索引或者范围。
73	array.zip(arg, ...) [or] 
array.zip(arg, ...){ | arr | block }
把任何参数转换为数组，然后把 array 的元素与每个参数中相对应的元素合并。
数组 pack 指令
下表列出了方法 Array#pack 的压缩指令。
指令	描述
@	移动到绝对位置。
A	ASCII 字符串（填充 space，count 是宽度）。
a	ASCII 字符串（填充 null，count 是宽度）。
B	位字符串（降序）
b	位字符串（升序）。
C	无符号字符。
c	字符。
D, d	双精度浮点数，原生格式。
E	双精度浮点数，little-endian 字节顺序。
e	单精度浮点数，little-endian 字节顺序。
F, f	单精度浮点数，原生格式。
G	双精度浮点数，network（big-endian）字节顺序。
g	单精度浮点数，network（big-endian）字节顺序。
H	十六进制字符串（高位优先）。
h	十六进制字符串（低位优先）。
I	无符号整数。
i	整数。
L	无符号 long。
l	Long。
M	引用可打印的，MIME 编码。
m	Base64 编码字符串。
N	Long，network（big-endian）字节顺序。
n	Short，network（big-endian）字节顺序。
P	指向一个结构（固定长度的字符串）。
p	指向一个空结束字符串。
Q, q	64 位数字。
S	无符号 short。
s	Short。
U	UTF-8。
u	UU 编码字符串。
V	Long，little-endian 字节顺序。
v	Short，little-endian 字节顺序。
w	BER 压缩的整数 \fnm。
X	向后跳过一个字节。
x	Null 字节。
Z	与 a 相同，除了 null 会被加上 *。
实例
尝试下面的实例，压缩各种数据。
a = [ "a", "b", "c" ]
n = [ 65, 66, 67 ]
puts a.pack("A3A3A3")   #=> "a  b  c  "
puts a.pack("a3a3a3")   #=> "a\000\000b\000\000c\000\000"
puts n.pack("ccc")      #=> "ABC"
这将产生以下结果：
a  b  c
abc
ABC
		
Ruby 哈希（Hash）
哈希（Hash）是类似 "employee" => "salary" 这样的键值对的集合。哈希的索引是通过任何对象类型的任意键来完成的，而不是一个整数索引，其他与数组相似。
通过键或值遍历哈希的顺序看起来是随意的，且通常不是按照插入顺序。如果您尝试通过一个不存在的键访问哈希，则方法会返回 nil。
创建哈希
与数组一样，有各种不同的方式来创建哈希。您可以通过 new 类方法创建一个空的哈希：
months = Hash.new
您也可以使用 new 创建带有默认值的哈希，不带默认值的哈希是 nil：
months = Hash.new( "month" )

or

months = Hash.new "month"
当您访问带有默认值的哈希中的任意键时，如果键或值不存在，访问哈希将返回默认值：
#!/usr/bin/ruby

months = Hash.new( "month" )

puts "#{months[0]}"
puts "#{months[72]}"
这将产生以下结果：
month
month
#!/usr/bin/ruby

H = Hash["a" => 100, "b" => 200]

puts "#{H['a']}"
puts "#{H['b']}"
这将产生以下结果：
100
200
您可以使用任何的 Ruby 对象作为键或值，甚至可以使用数组，所以下面的实例是一个有效的实例：
[1,"jan"] => "January"
哈希内建方法
我们需要有一个 Hash 对象的实例来调用 Hash 方法。下面是创建 Hash 对象实例的方式：
Hash[[key =>|, value]* ] or

Hash.new [or] Hash.new(obj) [or]

Hash.new { |hash, key| block }
这将返回一个使用给定对象进行填充的新的哈希。现在，使用创建的对象，我们可以调用任意可用的实例方法。例如：
#!/usr/bin/ruby

$, = ", "
months = Hash.new( "month" )

months = {"1" => "January", "2" => "February"}

keys = months.keys

puts "#{keys}"
这将产生以下结果：
2, 1
下面是公共的哈希方法（假设 hash 是一个 Hash 对象）：
序号	方法 & 描述
1	hash == other_hash
检查两个哈希是否具有相同的键值对个数，键值对是否相互匹配，来判断两个哈希是否相等。
2	hash.[key]
使用键，从哈希引用值。如果未找到键，则返回默认值。
3	hash.[key]=value
把 value 给定的值与 key 给定的键进行关联。
4	hash.clear
从哈希中移除所有的键值对。
5	hash.default(key = nil)
返回 hash 的默认值，如果未通过 default= 进行设置，则返回 nil。（如果键在 hash 中不存在，则 [] 返回一个默认值。）
6	hash.default = obj
为 hash 设置默认值。
7	hash.default_proc
如果 hash 通过块来创建，则返回块。
8	hash.delete(key) [or]
array.delete(key) { |key| block }
通过 key 从 hash 中删除键值对。如果使用了块 且未找到匹配的键值对，则返回块的结果。把它与 delete_if 进行比较。
9	hash.delete_if { |key,value| block }
为 block 为 true 的每个块，从 hash 中删除键值对。
10	hash.each { |key,value| block }
遍历 hash，为每个 key 调用一次 block，传递 key-value 作为一个二元素数组。
11	hash.each_key { |key| block }
遍历 hash，为每个 key 调用一次 block，传递 key 作为参数。
12	hash.each_key { |key_value_array| block }
遍历 hash，为每个 key 调用一次 block，传递 key 和 value 作为参数。
13	hash.each_key { |value| block }
遍历 hash，为每个 key 调用一次 block，传递 value 作为参数。
14	hash.empty?
检查 hash 是否为空（不包含键值对），返回 true 或 false。
15	hash.fetch(key [, default] ) [or]
hash.fetch(key) { | key | block }
通过给定的 key 从 hash 返回值。如果未找到 key，且未提供其他参数，则抛出 IndexError 异常；如果给出了 default，则返回 default；如果指定了可选的 block，则返回 block 的结果。
16	hash.has_key?(key) [or] hash.include?(key) [or]
hash.key?(key) [or] hash.member?(key)
检查给定的 key 是否存在于哈希中，返回 true 或 false。
17	hash.has_value?(value)
检查哈希是否包含给定的 value。
18	hash.index(value)
为给定的 value 返回哈希中的 key，如果未找到匹配值则返回 nil。
19	hash.indexes(keys)
返回一个新的数组，由给定的键的值组成。找不到的键将插入默认值。该方法已被废弃，请使用 select。
20	hash.indices(keys)
返回一个新的数组，由给定的键的值组成。找不到的键将插入默认值。该方法已被废弃，请使用 select。
21	hash.inspect
返回哈希的打印字符串版本。
22	hash.invert
创建一个新的 hash，倒置 hash 中的 keys 和 values。也就是说，在新的哈希中，hash 中的键将变成值，值将变成键。
23	hash.keys
创建一个新的数组，带有 hash 中的键。/td>
24	hash.length
以整数形式返回 hash 的大小或长度。
25	hash.merge(other_hash) [or]
hash.merge(other_hash) { |key, oldval, newval| block }
返回一个新的哈希，包含 hash 和 other_hash 的内容，重写 hash 中与 other_hash 带有重复键的键值对。
26	hash.merge!(other_hash) [or]
hash.merge!(other_hash) { |key, oldval, newval| block }
与 merge 相同，但实际上 hash 发生了变化。
27	hash.rehash
基于每个 key 的当前值重新建立 hash。如果插入后值发生了改变，该方法会重新索引 hash。
28	hash.reject { |key, value| block }
为 block 为 true 的每个键值对创建一个新的 hash。
29	hash.reject! { |key, value| block }
与 reject 相同，但实际上 hash 发生了变化。
30	hash.replace(other_hash)
把 hash 的内容替换为 other_hash 的内容。
31	hash.select { |key, value| block }
返回一个新的数组，由 block 返回 true 的 hash 中的键值对组成。
32	hash.shift
从 hash 中移除一个键值对，并把该键值对作为二元素数组返回。
33	hash.size
以整数形式返回 hash 的 size 或 length。
34	hash.sort
把 hash 转换为一个包含键值对数组的二维数组，然后进行排序。
35	hash.store(key, value)
存储 hash 中的一个键值对。
36	hash.to_a
从 hash 中创建一个二维数组。每个键值对转换为一个数组，所有这些数组都存储在一个数组中。
37	hash.to_hash
返回 hash（self）。
38	hash.to_s
把 hash 转换为一个数组，然后把该数组转换为一个字符串。
39	hash.update(other_hash) [or]
hash.update(other_hash) {|key, oldval, newval| block}
返回一个新的哈希，包含 hash 和 other_hash 的内容，重写 hash 中与 other_hash 带有重复键的键值对。
40	hash.value?(value)
检查 hash 是否包含给定的 value。
41	hash.values
返回一个新的数组，包含 hash 的所有值。
42	hash.values_at(obj, ...)
返回一个新的数组，包含 hash 中与给定的键相关的值。

		
Ruby 日期 & 时间（Date & Time）
Time 类在 Ruby 中用于表示日期和时间。它是基于操作系统提供的系统日期和时间之上。该类可能无法表示 1970 年之前或者 2038 年之后的日期。
本教程将让您熟悉日期和时间的所有重要的概念。
创建当前的日期和时间
下面是获取当前的日期和时间的简单实例：
#!/usr/bin/ruby -w

time1 = Time.new

puts "Current Time : " + time1.inspect

# Time.now 是一个同义词
time2 = Time.now
puts "Current Time : " + time2.inspect
这将产生以下结果：
Current Time : Mon Jun 02 12:02:39 -0700 2008
Current Time : Mon Jun 02 12:02:39 -0700 2008
获取 Date & Time 组件
我们可以使用 Time 对象来获取各种日期和时间的组件。请看下面的实例：
#!/usr/bin/ruby -w

time = Time.new

# Time 的组件
puts "Current Time : " + time.inspect
puts time.year    # => 日期的年份
puts time.month   # => 日期的月份（1 到 12）
puts time.day     # => 一个月中的第几天（1 到 31）
puts time.wday    # => 一周中的星期几（0 是星期日）
puts time.yday    # => 365：一年中的第几天
puts time.hour    # => 23：24 小时制
puts time.min     # => 59
puts time.sec     # => 59
puts time.usec    # => 999999：微秒
puts time.zone    # => "UTC"：时区名称
这将产生以下结果：
Current Time : Mon Jun 02 12:03:08 -0700 2008
2008
6
2
1
154
12
3
8
247476
UTC
Time.utc、Time.gm 和 Time.local 函数
这些函数可用于格式化标准格式的日期，如下所示：
# July 8, 2008
Time.local(2008, 7, 8)  
# July 8, 2008, 09:10am，本地时间
Time.local(2008, 7, 8, 9, 10)   
# July 8, 2008, 09:10 UTC
Time.utc(2008, 7, 8, 9, 10)  
# July 8, 2008, 09:10:11 GMT （与 UTC 相同）
Time.gm(2008, 7, 8, 9, 10, 11)  
下面的实例在数组中获取所有的组件：
[sec,min,hour,day,month,year,wday,yday,isdst,zone]
尝试下面的实例：
#!/usr/bin/ruby -w

time = Time.new

values = time.to_a
p values
这将产生以下结果：
[26, 10, 12, 2, 6, 2008, 1, 154, false, "MST"]
该数组可被传到 Time.utc 或 Time.local 函数来获取日期的不同格式，如下所示：
#!/usr/bin/ruby -w

time = Time.new

values = time.to_a
puts Time.utc(*values)
这将产生以下结果：
Mon Jun 02 12:15:36 UTC 2008
下面是获取时间的方式，从纪元以来的秒数（平台相关）：
# 返回从纪元以来的秒数
time = Time.now.to_i  

# 把秒数转换为 Time 对象
Time.at(time)

# 返回从纪元以来的秒数，包含微妙
time = Time.now.to_f
时区和夏令时
您可以使用 Time 对象来获取与时区和夏令时有关的所有信息，如下所示：
time = Time.new

# 这里是解释
time.zone       # => "UTC"：返回时区
time.utc_offset # => 0：UTC 是相对于 UTC 的 0 秒偏移
time.zone       # => "PST"（或其他时区）
time.isdst      # => false：如果 UTC 没有 DST（夏令时）
time.utc?       # => true：如果在 UTC 时区
time.localtime  # 转换为本地时区
time.gmtime     # 转换回 UTC
time.getlocal   # 返回本地区中的一个新的 Time 对象
time.getutc     # 返回 UTC 中的一个新的 Time 对象
格式化时间和日期
有多种方式格式化日期和时间。下面的实例演示了其中一部分：
#!/usr/bin/ruby -w
time = Time.new

puts time.to_s
puts time.ctime
puts time.localtime
puts time.strftime("%Y-%m-%d %H:%M:%S")
这将产生以下结果：
Mon Jun 02 12:35:19 -0700 2008
Mon Jun  2 12:35:19 2008
Mon Jun 02 12:35:19 -0700 2008
2008-06-02 12:35:19
时间格式化指令
下表所列出的指令与方法 Time.strftime 一起使用。
指令	描述
%a	星期几名称的缩写（比如 Sun）。
%A	星期几名称的全称（比如 Sunday）。
%b	月份名称的缩写（比如 Jan）。
%B	月份名称的全称（比如 January）。
%c	优选的本地日期和时间表示法。
%d	一个月中的第几天（01 到 31）。
%H	一天中的第几小时，24 小时制（00 到 23）。
%I	一天中的第几小时，12 小时制（01 到 12）。
%j	一年中的第几天（001 到 366）。
%m	一年中的第几月（01 到 12）。
%M	小时中的第几分钟（00 到 59）。
%p	子午线指示（AM 或 PM）。
%S	分钟中的第几秒（00 或 60）。
%U	当前年中的周数，从第一个星期日（作为第一周的第一天）开始（00 到 53）。
%W	当前年中的周数，从第一个星期一（作为第一周的第一天）开始（00 到 53）。
%w	一星期中的第几天（Sunday 是 0，0 到 6）。
%x	只有日期没有时间的优先表示法。
%X	只有时间没有日期的优先表示法。
%y	不带世纪的年份表示（00 到 99）。
%Y	带有世纪的年份。
%Z	时区名称。
%%	% 字符。
时间算法
您可以用时间做一些简单的算术，如下所示：
now = Time.now           # 当前时间
puts now

past = now - 10          # 10 秒之前。Time - number => Time
puts past

future = now + 10        # 从现在开始 10 秒之后。Time + number => Time
puts future

diff = future - now      # => 10  Time - Time => 秒数
puts diff
这将产生以下结果：
Thu Aug 01 20:57:05 -0700 2013
Thu Aug 01 20:56:55 -0700 2013
Thu Aug 01 20:57:15 -0700 2013
10.0
		
Ruby 范围（Range）
范围（Range）无处不在：January 到 December、 0 到 9、等等。Ruby 支持范围，并允许我们以不同的方式使用范围：
* 作为序列的范围
* 作为条件的范围
* 作为间隔的范围
作为序列的范围
范围的第一个也是最常见的用途是表达序列。序列有一个起点、一个终点和一个在序列产生连续值的方式。
Ruby 使用 ''..'' 和 ''...'' 范围运算符创建这些序列。两点形式创建一个包含指定的最高值的范围，三点形式创建一个不包含指定的最高值的范围。
(1..5)        #==> 1, 2, 3, 4, 5
(1...5)       #==> 1, 2, 3, 4
('a'..'d')    #==> 'a', 'b', 'c', 'd'
序列 1..100 是一个 Range 对象，包含了两个 Fixnum 对象的引用。如果需要，您可以使用 to_a 方法把范围转换为列表。尝试下面的实例：
#!/usr/bin/ruby

$, =", "   # Array 值分隔符
range1 = (1..10).to_a
range2 = ('bar'..'bat').to_a

puts "#{range1}"
puts "#{range2}"
这将产生以下结果：
1, 2, 3, 4, 5, 6, 7, 8, 9, 10
bar, bas, bat
范围实现了让您可以遍历它们的方法，您可以通过多种方式检查它们的内容：
#!/usr/bin/ruby

# Assume a range
digits = 0..9

puts digits.include?(5)
ret = digits.min
puts "Min value is #{ret}"

ret = digits.max
puts "Max value is #{ret}"

ret = digits.reject {|i| i < 5 }
puts "Rejected values are #{ret}"

digits.each do |digit|
   puts "In Loop #{digit}"
end
这将产生以下结果：
true
Min value is 0
Max value is 9
Rejected values are 5, 6, 7, 8, 9
In Loop 0
In Loop 1
In Loop 2
In Loop 3
In Loop 4
In Loop 5
In Loop 6
In Loop 7
In Loop 8
In Loop 9
作为条件的范围
范围也可以用作条件表达式。例如，下面的代码片段从标准输入打印行，其中每个集合的第一行包含单词 start，最后一行包含单词 end.：
while gets
   print if /start/../end/
end
范围可以用在 case 语句中：
#!/usr/bin/ruby

score = 70

result = case score
   when 0..40: "Fail"
   when 41..60: "Pass"
   when 61..70: "Pass with Merit"
   when 71..100: "Pass with Distinction"
   else "Invalid Score"
end

puts result
这将产生以下结果：
Pass with Merit
作为间隔的范围
范围的最后一个用途是间隔测试：检查某些值是否落在范围表示的间隔里。这是使用 === 相等运算符来完成的。
#!/usr/bin/ruby

if ((1..10) === 5)
  puts "5 lies in (1..10)"
end

if (('a'..'j') === 'c')
  puts "c lies in ('a'..'j')"
end

if (('a'..'j') === 'z')
  puts "z lies in ('a'..'j')"
end
这将产生以下结果：
5 lies in (1..10)
c lies in ('a'..'j')
		
Ruby 迭代器
迭代器是集合支持的方法。存储一组数据成员的对象称为集合。在 Ruby 中，数组和散列可以称之为集合。
迭代器返回集合的所有元素，一个接着一个。在这里我们将讨论两种迭代器，each 和 collect。
Ruby each 迭代器
each 迭代器返回数组或哈希的所有元素。
语法
collection.each do |variable|
   code
end
为集合中的每个元素执行 code。在这里，集合可以是数组或哈希。
实例
#!/usr/bin/ruby

ary = [1,2,3,4,5]
ary.each do |i|
   puts i
end
这将产生以下结果：
1
2
3
4
5
each 迭代器总是与一个块关联。它向块返回数组的每个值，一个接着一个。值被存储在变量 i 中，然后显示在屏幕上。
Ruby collect 迭代器
collect 迭代器返回集合的所有元素。
语法
collection = collection.collect
collect 方法不需要总是与一个块关联。collect 方法返回整个集合，不管它是数组或者是哈希。
实例
#!/usr/bin/ruby

a = [1,2,3,4,5]
b = Array.new
b = a.collect
puts b
这将产生以下结果：
1
2
3
4
5
注意：collect 方法不是数组间进行复制的正确方式。这里有另一个称为 clone 的方法，用于复制一个数组到另一个数组。
当您想要对每个值进行一些操作以便获得新的数组时，您通常使用 collect 方法。例如，下面的代码会生成一个数组，其值是 a 中每个值的 10 倍。
#!/usr/bin/ruby

a = [1,2,3,4,5]
b = a.collect{|x| 10*x}
puts b
这将产生以下结果：
10
20
30
40
50
		
Ruby 文件的输入与输出
Ruby 提供了一整套 I/O 相关的方法，在内核（Kernel）模块中实现。所有的 I/O 方法派生自 IO 类。
类 IO 提供了所有基础的方法，比如 read、 write、 gets、 puts、 readline、 getc 和 printf。
本章节将讲解所有 Ruby 中可用的基础的 I/O 函数。如需了解更多的函数，请查看 Ruby 的 IO 类。
puts 语句
在前面的章节中，您赋值给变量，然后使用 puts 语句打印输出。
puts 语句指示程序显示存储在变量中的值。这将在每行末尾添加一个新行。
实例
#!/usr/bin/ruby

val1 = "This is variable one"
val2 = "This is variable two"
puts val1
puts val2
这将产生以下结果：
This is variable one
This is variable two
gets 语句
gets 语句可用于获取来自名为 STDIN 的标准屏幕的用户输入。
实例
下面的代码演示了如何使用 gets 语句。该代码将提示用户输入一个值，该值将被存储在变量 val 中，最后会被打印在 STDOUT 上。
#!/usr/bin/ruby

puts "Enter a value :"
val = gets
puts val
这将产生以下结果：
Enter a value :
This is entered value
This is entered value
putc 语句
与 puts 语句不同，puts 语句输出整个字符串到屏幕上，而 putc 语句可用于依次输出一个字符。
实例
下面代码的输出只是字符 H：
#!/usr/bin/ruby

str="Hello Ruby!"
putc str
这将产生以下结果：
H
print 语句
print 语句与 puts 语句类似。唯一的不同在于 puts 语句在输出内容后会跳到下一行，而使用 print 语句时，光标定位在同一行。
实例
#!/usr/bin/ruby

print "Hello World"
print "Good Morning"
这将产生以下结果：
Hello WorldGood Morning
打开和关闭文件
截至现在，您已经读取并写入标准输入和输出。现在，我们将看看如何操作实际的数据文件。
File.new 方法
您可以使用 File.new 方法创建一个 File 对象用于读取、写入或者读写，读写权限取决于 mode 字符串。最后，您可以使用 File.close 方法来关闭该文件。
语法
aFile = File.new("filename", "mode")
   # ... 处理文件
aFile.close
File.open 方法
您可以使用 File.open 方法创建一个新的 file 对象，并把该 file 对象赋值给文件。但是，File.open 和 File.new 方法之间有一点不同。不同点是 File.open 方法可与块关联，而 File.new 方法不能。
File.open("filename", "mode") do |aFile|
   # ... process the file
end
下表列出了打开文件的不同模式：
模式	描述
r	只读模式。文件指针被放置在文件的开头。这是默认模式。
r+	读写模式。文件指针被放置在文件的开头。
w	只写模式。如果文件存在，则重写文件。如果文件不存在，则创建一个新文件用于写入。
w+	读写模式。如果文件存在，则重写已存在的文件。如果文件不存在，则创建一个新文件用于读写。
a	只写模式。如果文件存在，则文件指针被放置在文件的末尾。也就是说，文件是追加模式。如果文件不存在，则创建一个新文件用于写入。
a+	读写模式。如果文件存在，则文件指针被放置在文件的末尾。也就是说，文件是追加模式。如果文件不存在，则创建一个新文件用于读写。
读取和写入文件
用于简单 I/O 的方法也可用于所有 file 对象。所以，gets 从标准输入读取一行，aFile.gets 从文件对象 aFile 读取一行。
但是，I/O 对象提供了访问方法的附加设置，为我们提供了便利。
sysread 方法
您可以使用方法 sysread 来读取文件的内容。当使用方法 sysread 时，您可以使用任意一种模式打开文件。例如：
下面是输入文本文件：
This is a simple text file for testing purpose.
现在让我们尝试读取这个文件：
#!/usr/bin/ruby

aFile = File.new("input.txt", "r")
if aFile
   content = aFile.sysread(20)
   puts content
else
   puts "Unable to open file!"
end
该语句将输入文件的头 20 个字符。文件指针将被放置在文件中第 21 个字符的位置。
syswrite 方法
您可以使用方法 syswrite 来向文件写入内容。当使用方法 syswrite 时，您需要以写入模式打开文件。例如：
#!/usr/bin/ruby

aFile = File.new("input.txt", "r+")
if aFile
   aFile.syswrite("ABCDEF")
else
   puts "Unable to open file!"
end
该语句将写入 "ABCDEF" 到文件中。
each_byte 方法
该方法属于类 File。方法 each_byte 总是与块相关联。请看下面的代码实例：
#!/usr/bin/ruby

aFile = File.new("input.txt", "r+")
if aFile
   aFile.syswrite("ABCDEF")
   aFile.each_byte {|ch| putc ch; putc ?. }
else
   puts "Unable to open file!"
end
字符一个接着一个被传到变量 ch，然后显示在屏幕上，如下所示：
s. .a. .s.i.m.p.l.e. .t.e.x.t. .f.i.l.e. .f.o.r. .t.e.s.t.i.n.g. .p.u.r.p.o.s.e...
.
.
IO.readlines 方法
类 File 是类 IO 的一个子类。类 IO 也有一些用于操作文件的方法。
IO.readlines 是 IO 类中的一个方法。该方法逐行返回文件的内容。下面的代码显示了方法 IO.readlines 的使用：
#!/usr/bin/ruby

arr = IO.readlines("input.txt")
puts arr[0]
puts arr[1]
在这段代码中，变量 arr 是一个数组。文件 input.txt 的每一行将是数组 arr 中的一个元素。因此，arr[0] 将包含第一行，而 arr[1] 将包含文件的第二行。
IO.foreach 方法
该方法也逐行返回输出。方法 foreach 与方法 readlines 之间不同的是，方法 foreach 与块相关联。但是，不像方法 readlines，方法 foreach 不是返回一个数组。例如：
#!/usr/bin/ruby

IO.foreach("input.txt"){|block| puts block}
这段代码将把文件 test 的内容逐行传给变量 block，然后输出将显示在屏幕上。
重命名和删除文件
您可以通过 rename 和 delete 方法重命名和删除文件。
下面的实例重命名一个已存在文件 test1.txt：
#!/usr/bin/ruby

# 重命名文件 test1.txt 为 test2.txt
File.rename( "test1.txt", "test2.txt" )

下面的实例删除一个已存在文件 test2.txt：
#!/usr/bin/ruby

# 删除文件 test2.txt
File.delete("text2.txt")
文件模式与所有权
使用带有掩码的 chmod 方法来改变文件的模式或权限/访问列表：
下面的实例改变一个已存在文件 test.txt 的模式为一个掩码值：
#!/usr/bin/ruby

file = File.new( "test.txt", "w" )
file.chmod( 0755 )
下表列出了 chmod 方法中可使用的不同的掩码：
掩码	描述
0700	rwx 掩码，针对所有者
0400	r ，针对所有者
0200	w ，针对所有者
0100	x ，针对所有者
0070	rwx 掩码，针对所属组
0040	r ，针对所属组
0020	w ，针对所属组
0010	x ，针对所属组
0007	rwx 掩码，针对其他人
0004	r ，针对其他人
0002	w ，针对其他人
0001	x ，针对其他人
4000	执行时设置用户 ID
2000	执行时设置所属组 ID
1000	保存交换文本，甚至在使用后也会保存
文件查询
下面的命令在打开文件前检查文件是否已存在：
#!/usr/bin/ruby

File.open("file.rb") if File::exists?( "file.rb" )
下面的命令查询文件是否确实是一个文件：
#!/usr/bin/ruby

# 返回 <i>true</i> 或 <i>false</i>
File.file?( "text.txt" ) 
下面的命令检查给定的文件名是否是一个目录：
#!/usr/bin/ruby

# 一个目录
File::directory?( "/usr/local/bin" ) # => true

# 一个文件
File::directory?( "file.rb" ) # => false
下面的命令检查文件是否可读、可写、可执行：
#!/usr/bin/ruby

File.readable?( "test.txt" )   # => true
File.writable?( "test.txt" )   # => true
File.executable?( "test.txt" ) # => false
下面的命令检查文件是否大小为零：
#!/usr/bin/ruby

File.zero?( "test.txt" )      # => true
下面的命令返回文件的大小：
#!/usr/bin/ruby

File.size?( "text.txt" )     # => 1002
下面的命令用于检查文件的类型：
#!/usr/bin/ruby

File::ftype( "test.txt" )     # => file
ftype 方法通过返回下列中的某个值来标识了文件的类型：file、 directory、 characterSpecial、 blockSpecial、 fifo、 link、 socket 或 unknown。
下面的命令用于检查文件被创建、修改或最后访问的时间：
#!/usr/bin/ruby

File::ctime( "test.txt" ) # => Fri May 09 10:06:37 -0700 2008
File::mtime( "text.txt" ) # => Fri May 09 10:44:44 -0700 2008
File::atime( "text.txt" ) # => Fri May 09 10:45:01 -0700 2008
Ruby 中的目录
所有的文件都是包含在目录中，Ruby 提供了处理文件和目录的方式。File 类用于处理文件，Dir 类用于处理目录。
浏览目录
为了在 Ruby 程序中改变目录，请使用 Dir.chdir。下面的实例改变当前目录为 /usr/bin。
Dir.chdir("/usr/bin")
您可以通过 Dir.pwd 查看当前目录：
puts Dir.pwd # 返回当前目录，类似 /usr/bin
您可以使用 Dir.entries 获取指定目录内的文件和目录列表：
puts Dir.entries("/usr/bin").join(' ')
Dir.entries 返回一个数组，包含指定目录内的所有项。Dir.foreach 提供了相同的功能：
Dir.foreach("/usr/bin") do |entry|
   puts entry
end
获取目录列表的一个更简洁的方式是通过使用 Dir 的类数组的方法：
Dir["/usr/bin/*"]
创建目录
Dir.mkdir 可用于创建目录：
Dir.mkdir("mynewdir")
您也可以通过 mkdir 在新目录（不是已存在的目录）上设置权限：
注意：掩码 755 设置所有者（owner）、所属组（group）、每个人（world [anyone]）的权限为 rwxr-xr-x，其中 r = read 读取，w = write 写入，x = execute 执行。
Dir.mkdir( "mynewdir", 755 )
删除目录
Dir.delete 可用于删除目录。Dir.unlink 和 Dir.rmdir 执行同样的功能，为我们提供了便利。
Dir.delete("testdir")
创建文件 & 临时目录
临时文件是那些在程序执行过程中被简单地创建，但不会永久性存储的信息。
Dir.tmpdir 提供了当前系统上临时目录的路径，但是该方法默认情况下是不可用的。为了让 Dir.tmpdir 可用，使用必需的 'tmpdir' 是必要的。
您可以把 Dir.tmpdir 和 File.join 一起使用，来创建一个独立于平台的临时文件：
require 'tmpdir'
   tempfilename = File.join(Dir.tmpdir, "tingtong")
   tempfile = File.new(tempfilename, "w")
   tempfile.puts "This is a temporary file"
   tempfile.close
   File.delete(tempfilename)
这段代码创建了一个临时文件，并向其中写入数据，然后删除文件。Ruby 的标准库也包含了一个名为 Tempfile 的库，该库可用于创建临时文件：
require 'tempfile'
   f = Tempfile.new('tingtong')
   f.puts "Hello"
   puts f.path
   f.close
内建函数
下面提供了 Ruby 中处理文件和目录的内建函数的完整列表：
* File 类和方法。
* Dir 类和方法。
		
Ruby File 类和方法
File 表示一个连接到普通文件的 stdio 对象。open 为普通文件返回该类的一个实例。
类方法
序号	方法 & 描述
1	File::atime( path)
返回 path 的最后访问时间。
2	File::basename( path[, suffix])
返回 path 末尾的文件名。如果指定了 suffix，则它会从文件名末尾被删除。
例如：File.basename("/home/users/bin/ruby.exe") #=> "ruby.exe"
3	File::blockdev?( path)
如果 path 是一个块设备，则返回 true。
4	File::chardev?( path)
如果 path 是一个字符设备，则返回 true。
5	File::chmod( mode, path...)
改变指定文件的权限模式。
6	File::chown( owner, group, path...)
改变指定文件的所有者和所属组。
7	File::ctime( path)
返回 path 的最后一个 inode 更改时间。
8	File::delete( path...)
File::unlink( path...)
删除指定的文件。
9	File::directory?( path)
如果 path 是一个目录，则返回 true。
10	File::dirname( path)
返回 path 的目录部分，不包括最后的文件名。
11	File::executable?( path)
如果 path 是可执行的，则返回 true。
12	File::executable_real?( path)
如果 path 通过真正的用户权限是可执行的，则返回 true。
13	File::exist?( path)
如果 path 存在，则返回 true。
1	File::expand_path( path[, dir])
返回 path 的绝对路径，扩展 ~ 为进程所有者的主目录，~user 为用户的主目录。相对路径是相对于 dir 指定的目录，如果 dir 被省略则相对于当前工作目录。
14	File::file?( path)
如果 path 是一个普通文件，则返回 true。
15	File::ftype( path)
返回下列其中一个字符串，表示文件类型：
	•	file - 普通文件
	•	directory - 目录
	•	characterSpecial - 字符特殊文件
	•	blockSpecial - 块特殊文件
	•	fifo - 命名管道（FIFO）
	•	link - 符号链接
	•	socket - Socket
	•	unknown - 未知的文件类型
16	File::grpowned?( path)
如果 path 由用户的所属组所有，则返回 true。
17	File::join( item...)
返回一个字符串，由指定的项连接在一起，并使用 File::Separator 进行分隔。
例如：File::join("", "home", "usrs", "bin") # => "/home/usrs/bin"
18	File::link( old, new)
创建一个到文件 old 的硬链接。
19	File::lstat( path)
与 stat 相同，但是它返回自身符号链接上的信息，而不是所指向的文件。
20	File::mtime( path)
返回 path 的最后一次修改时间。
21	File::new( path[, mode="r"])
File::open( path[, mode="r"])
File::open( path[, mode="r"]) {|f| ...}
打开文件。如果指定了块，则通过传递新文件作为参数来执行块。当块退出时，文件会自动关闭。这些方法有别于 Kernel.open，即使 path 是以 | 开头，后续的字符串也不会作为命令运行。
22	File::owned?( path)
如果 path 由有效的用户所有，则返回 true。
23	File::pipe?( path)
如果 path 是一个管道，则返回 true。
24	File::readable?( path)
如果 path 是可读的，则返回 true。
25	File::readable_real?( path)
如果 path 通过真正的用户权限是可读的，则返回 true。
25	File::readlink( path)
返回 path 所指向的文件。
26	File::rename( old, new)
改变文件名 old 为 new。
27	File::setgid?( path)
如果设置了 path 的 set-group-id 权限位，则返回 true。
28	File::setuid?( path)
如果设置了 path 的 set-user-id 权限位，则返回 true。
29	File::size( path)
返回 path 的文件大小。
30	File::size?( path)
返回 path 的文件大小，如果为 0 则返回 nil。
31	File::socket?( path)
如果 path 是一个 socket，则返回 true。
32	File::split( path)
返回一个数组，包含 path 的内容，path 被分成 File::dirname(path) 和 File::basename(path)。
33	File::stat( path)
返回 path 上带有信息的 File::Stat 对象。
34	File::sticky?( path)
如果设置了 path 的 sticky 位，则返回 true。
35	File::symlink( old, new)
创建一个指向文件 old 的符号链接。
36	File::symlink?( path)
如果 path 是一个符号链接，则返回 true。
37	File::truncate( path, len)
截断指定的文件为 len 字节。
38	File::unlink( path...)
删除 path 给定的文件。
39	File::umask([ mask])
如果未指定参数，则为该进程返回当前的 umask。如果指定了一个参数，则设置了 umask，并返回旧的 umask。
40	File::utime( atime, mtime, path...)
改变指定文件的访问和修改时间。
41	File::writable?( path)
如果 path 是可写的，则返回 true。
42	File::writable_real?( path)
如果 path 通过真正的用户权限是可写的，则返回 true。
43	File::zero?( path)
如果 path 的文件大小是 0，则返回 true。
实例方法
假设 f 是 File 类的一个实例：
序号	方法 & 描述
1	f.atime
返回 f 的最后访问时间。
2	f.chmode( mode)
改变 f 的权限模式。
3	f.chown( owner, group)
改变 f 的所有者和所属组。
4	f.ctime
返回 f 的最后一个 inode 更改时间。
5	f.flock( op)
调用 flock(2)。op 可以是 0 或一个逻辑值或 File 类常量 LOCK_EX、LOCK_NB、LOCK_SH 和 LOCK_UN。
6	f.lstat
与 stat 相同，但是它返回自身符号链接上的信息，而不是所指向的文件。
7	f.mtime
返回 f 的最后修改时间。
8	f.path
返回用于创建 f 的路径名。
9	f.reopen( path[, mode="r"])
重新打开文件。
10	f.truncate( len)
截断 f 为 len 字节。
		
Ruby Dir 类和方法
Dir 是一个表示用于给出操作系统中目录中的文件名的目录流。Dir 类也拥有与目录相关的操作，比如通配符文件名匹配、改变工作目录等。
类方法
序号	方法 & 描述
1	Dir[pat]
Dir::glob( pat)
返回一个数组，包含与指定的通配符模式 pat 匹配的文件名：
	•	* - 匹配包含 null 字符串的任意字符串
	•	** - 递归地匹配任意字符串
	•	? - 匹配任意单个字符
	•	[...] - 匹配封闭字符中的任意一个
	•	{a,b...} - 匹配字符串中的任意一个
Dir["foo.*"] # 匹配 "foo.c"、 "foo.rb" 等等
Dir["foo.?"] # 匹配 "foo.c"、 "foo.h" 等等
2	Dir::chdir( path)
改变当前目录。
3	Dir::chroot( path)
改变根目录（只允许超级用户）。并不是在所有的平台上都可用。
4	Dir::delete( path)
删除 path 指定的目录。目录必须是空的。
5	Dir::entries( path)
返回一个数组，包含目录 path 中的文件名。
6	Dir::foreach( path) {| f| ...}
为 path 指定的目录中的每个文件执行一次块。
7	Dir::getwd
Dir::pwd
返回当前目录。
8	Dir::mkdir( path[, mode=0777])
创建 path 指定的目录。权限模式可被 File::umask 的值修改，在 Win32 的平台上会被忽略。
9	Dir::new( path)
Dir::open( path)
Dir::open( path) {| dir| ...}
返回 path 的新目录对象。如果 open 给出一个块，则新目录对象会传到该块，块会在终止前关闭目录对象。
10	Dir::pwd
参见 Dir::getwd。
11	Dir::rmdir( path)
Dir::unlink( path)
Dir::delete( path)
删除 path 指定的目录。目录必须是空的。
实例方法
假设 d 是 Dir 类的一个实例：
序号	方法 & 描述
1	d.close
关闭目录流。
2	d.each {| f| ...}
为 d 中的每一个条目执行一次块。
3	d.pos
d.tell
返回 d 中的当前位置。
4	d.pos= offset
设置目录流中的位置。
5	d.pos= pos
d.seek(pos)
移动到 d 中的某个位置。pos 必须是一个由 d.pos 返回的值或 0。
6	d.read
返回 d 的下一个条目。
7	d.rewind
移动 d 中的位置到第一个条目。
8	d.seek(po s)
参见 d.pos=pos。
9	d.tell
参见 d.pos。
		
Ruby 异常
异常和执行总是被联系在一起。如果您打开一个不存在的文件，且没有恰当地处理这种情况，那么您的程序则被认为是低质量的。
如果异常发生，则程序停止。异常用于处理各种类型的错误，这些错误可能在程序执行期间发生，所以要采取适当的行动，而不至于让程序完全停止。
Ruby 提供了一个完美的处理异常的机制。我们可以在 begin/end 块中附上可能抛出异常的代码，并使用 rescue 子句告诉 Ruby 完美要处理的异常类型。
语法
begin  
# -  
rescue OneTypeOfException  
# -  
rescue AnotherTypeOfException  
# -  
else  
# 其他异常
ensure
# 总是被执行
end
从 begin 到 rescue 中的一切是受保护的。如果代码块执行期间发生了异常，控制会传到 rescue 和 end 之间的块。
对于 begin 块中的每个 rescue 子句，Ruby 把抛出的异常与每个参数进行轮流比较。如果 rescue 子句中命名的异常与当前抛出的异常类型相同，或者是该异常的父类，则匹配成功。
如果异常不匹配所有指定的错误类型，我们可以在所有的 rescue 子句后使用一个 else 子句。
实例
#!/usr/bin/ruby

begin
   file = open("/unexistant_file")
   if file
      puts "File opened successfully"
   end
rescue
      file = STDIN
end
print file, "==", STDIN, "\n"
这将产生以下结果。您可以看到，STDIN 取代了 file ，因为打开失败。
#<IO:0xb7d16f84>==#<IO:0xb7d16f84>
使用 retry 语句
您可以使用 rescue 块捕获异常，然后使用 retry 语句从开头开始执行 begin 块。
语法
begin
    # 这段代码抛出的异常将被下面的 rescue 子句捕获
rescue
    # 这个块将捕获所有类型的异常
    retry  # 这将把控制移到 begin 的开头
end
实例
#!/usr/bin/ruby

begin
   file = open("/unexistant_file")
   if file
      puts "File opened successfully"
   end
rescue
   fname = "existant_file"
   retry
end
以下是处理流程：
* 打开时发生异常。
* 跳到 rescue。fname 被重新赋值。
* 通过 retry 跳到 begin 的开头。
* 这次文件成功打开。
* 继续基本的过程。
注意：如果被重新命名的文件不存在，本势力代码会无限尝试。所以异常处理时，谨慎使用 retry。
使用 raise 语句
您可以使用 raise 语句抛出异常。下面的方法在调用时抛出异常。它的第二个消息将被输出。
语法
raise 

OR

raise "Error Message" 

OR

raise ExceptionType, "Error Message"

OR

raise ExceptionType, "Error Message" condition

第一种形式简单地重新抛出当前异常（如果没有当前异常则抛出一个 RuntimeError）。这用在传入异常之前需要解释异常的异常处理程序中。
第二种形式创建一个新的 RuntimeError 异常，设置它的消息为给定的字符串。该异常之后抛出到调用堆栈。
第三种形式使用第一个参数创建一个异常，然后设置相关的消息为第二个参数。
第四种形式与第三种形式类似，您可以添加任何额外的条件语句（比如 unless）来抛出异常。
实例
#!/usr/bin/ruby

begin  
    puts 'I am before the raise.'  
    raise 'An error has occurred.'  
    puts 'I am after the raise.'  
rescue  
    puts 'I am rescued.'  
end  
puts 'I am after the begin block.'  
这将产生以下结果：
I am before the raise.  
I am rescued.  
I am after the begin block.  
另一个演示 raise 用法的实例：
#!/usr/bin/ruby

begin  
  raise 'A test exception.'  
rescue Exception => e  
  puts e.message  
  puts e.backtrace.inspect  
end  
这将产生以下结果：
A test exception.
["main.rb:4"]
使用 ensure 语句
有时候，无论是否抛出异常，您需要保证一些处理在代码块结束时完成。例如，您可能在进入时打开了一个文件，当您退出块时，您需要确保关闭文件。
ensure 子句做的就是这个。ensure 放在最后一个 rescue 子句后，并包含一个块终止时总是执行的代码块。它与块是否正常退出、是否抛出并处理异常、是否因一个未捕获的异常而终止，这些都没关系，ensure 块始终都会运行。
语法
begin 
   #.. 过程
   #.. 抛出异常
rescue 
   #.. 处理错误 
ensure 
   #.. 最后确保执行
   #.. 这总是会执行
end
实例
begin
  raise 'A test exception.'
rescue Exception => e
  puts e.message
  puts e.backtrace.inspect
ensure
  puts "Ensuring execution"
end
这将产生以下结果：
A test exception.
["main.rb:4"]
Ensuring execution
使用 else 语句
如果提供了 else 子句，它一般是放置在 rescue 子句之后，任意 ensure 之前。
else 子句的主体只有在代码主体没有抛出异常时执行。
语法
begin 
   #.. 过程 
   #.. 抛出异常
rescue 
   #.. 处理错误
else
   #.. 如果没有异常则执行
ensure 
   #.. 最后确保执行
   #.. 这总是会执行
end
实例
begin
 # 抛出 'A test exception.'
 puts "I'm not raising exception"
rescue Exception => e
  puts e.message
  puts e.backtrace.inspect
else
   puts "Congratulations-- no errors!"
ensure
  puts "Ensuring execution"
end
这将产生以下结果：
I'm not raising exception
Congratulations-- no errors!
Ensuring execution
使用 $! 变量可以捕获抛出的错误消息。
Catch 和 Throw
raise 和 rescue 的异常机制能在发生错误时放弃执行，有时候需要在正常处理时跳出一些深层嵌套的结构。此时 catch 和 throw 就派上用场了。
catch 定义了一个使用给定的名称（可以是 Symbol 或 String）作为标签的块。块会正常执行知道遇到一个 throw。
语法
throw :lablename
#.. 这不会被执行
catch :lablename do
#.. 在遇到一个 throw 后匹配将被执行的 catch
end

OR

throw :lablename condition
#.. 这不会被执行
catch :lablename do
#.. 在遇到一个 throw 后匹配将被执行的 catch
end
实例
下面的实例中，如果用户键入 '!' 回应任何提示，使用一个 throw 终止与用户的交互。
def promptAndGet(prompt)
   print prompt
   res = readline.chomp
   throw :quitRequested if res == "!"
   return res
end

catch :quitRequested do
   name = promptAndGet("Name: ")
   age = promptAndGet("Age: ")
   sex = promptAndGet("Sex: ")
   # ..
   # 处理信息
end
promptAndGet("Name:")
上面的程序需要人工交互，您可以在您的计算机上进行尝试。这将产生以下结果：
Name: Ruby on Rails
Age: 3
Sex: !
Name:Just Ruby
类 Exception
Ruby 的标准类和模块抛出异常。所有的异常类组成一个层次，包括顶部的 Exception 类在内。下一层是七种不同的类型：
* Interrupt
* NoMemoryError
* SignalException
* ScriptError
* StandardError
* SystemExit
Fatal 是该层中另一种异常，但是 Ruby 解释器只在内部使用它。
ScriptError 和 StandardError 都有一些子类，但是在这里我们不需要了解这些细节。最重要的事情是创建我们自己的异常类，它们必须是类 Exception 或其子代的子类。
让我们看一个实例：
class FileSaveError < StandardError
   attr_reader :reason
   def initialize(reason)
      @reason = reason
   end
end
现在，看下面的实例，将用到上面的异常：
File.open(path, "w") do |file|
begin
    # 写出数据 ...
rescue
    # 发生错误
    raise FileSaveError.new($!)
end
end
在这里，最重要的一行是 raise FileSaveError.new($!)。我们调用 raise 来示意异常已经发生，把它传给 FileSaveError 的一个新的实例，由于特定的异常引起数据写入失败。

		

		

		

		

		

		

		

		

		

