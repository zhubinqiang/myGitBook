# Groovy 笔记

[TOC]

[语法规则](http://www.groovy-lang.org/syntax.html)

## 注释
```groovy
#!/usr/bin/env groovy

// 单行注释
def a = 'abc'
println a

/**
    javadoc 注释
*/
def fun() {
    /*
    多行注释1
    多行注释2
     */
    println 'haha'
}
```

## 字符串
```groovy
def s1 = 'a single quoted string'
def s2 = '''a triple single quoted string'''
def s3 = "a double quoted string"
def aMultilineString = '''line one
line two
line three'''

def name = 'Guillaume' // a plain string
def greeting = "Hello ${name}"

def sum = "The sum of 2 and 3 equals ${2 + 3}"
assert sum.toString() == 'The sum of 2 and 3 equals 5'

// 字符串连接
assert 'ab' = 'a' + 'b'
```

## if/else
```groovy
def x = 0

if ( x > 0 ) {
    println "+"
} else if ( x < 0 ) {
    println "-"
} else {
    println "0"
}
```

## switch/case
```groovy
def x = 1.23
def result = ""

switch ( x ) {
    case "foo":
        result = "found foo"
        // lets fall through

    case "bar":
        result += "bar"

    case [4, 5, 6, 'inList']:
        result = "list"
        break

    case 12..30:
        result = "range"
        break

    case Integer:
        result = "integer"
        break

    case Number:
        result = "number"
        break

    case ~/fo*/: // toString() representation of x matches the pattern?
        result = "foo regex"
        break

    case { it < 0 }: // or { x < 0 }
        result = "negative"
        break

    default:
        result = "default"
}

assert result == "number"
```

## for
### Classic for loop
```groovy
String message = ''
for (int i = 0; i < 5; i++) {
    message += 'Hi '
}
assert message == 'Hi Hi Hi Hi Hi '
```

### for in loop
```groovy
// iterate over a range
def x = 0
for ( i in 0..9 ) {
    x += i
}
assert x == 45

// iterate over a list
x = 0
for ( i in [0, 1, 2, 3, 4] ) {
    x += i
}
assert x == 10

// iterate over an array
def array = (0..4).toArray()
x = 0
for ( i in array ) {
    x += i
}
assert x == 10

// iterate over a map
def map = ['abc':1, 'def':2, 'xyz':3]
x = 0
for ( e in map ) {
    x += e.value
}
assert x == 6

// iterate over values in a map
x = 0
for ( v in map.values() ) {
    x += v
}
assert x == 6

// iterate over the characters in a string
def text = "abc"
def list = []
for (c in text) {
    list.add(c)
}
assert list == ["a", "b", "c"]
```

## while
```groovy
def x = 0
def y = 5

while ( y-- > 0 ) {
    x++
}

assert x == 5
```

## try/catch
```groovy
try {
    'moo'.toLong()   // this will generate an exception
    assert false     // asserting that this point should never be reached
} catch ( e ) {
    assert e in NumberFormatException
}


try {
    /* ... */
} catch ( IOException | NullPointerException e ) {
    /* one block to handle 2 exceptions */
}
```


```groovy

```

## 正则表达式
```groovy
def p = ~/foo/
assert p instanceof Pattern

p = ~'foo'                                                        
p = ~"foo"                                                        
p = ~$/dollar/slashy $ string/$                                   
p = ~"${pattern}"
```

### find 操作符
```groovy
def text = "some text to match"
def m = text =~ /match/                                           
assert m instanceof Matcher                                       
if (!m) {                                                         
    throw new RuntimeException("Oops, text not found!")
}
```

## 其他操作符
```groovy
class Car {
    String make
    String model
}
def cars = [
       new Car(make: 'Peugeot', model: '508'),
       new Car(make: 'Renault', model: 'Clio')]       
def makes = cars*.make                                
assert makes == ['Peugeot', 'Renault']
```

## 范围操作符
```groovy
def range = 0..5                                    
assert (0..5).collect() == [0, 1, 2, 3, 4, 5]       
assert (0..<5).collect() == [0, 1, 2, 3, 4]         
assert (0..5) instanceof List                       
assert (0..5).size() == 6 
```

## in 操作符
```groovy
def list = ['Grace','Rob','Emmy']
assert ('Emmy' in list)
```

## as 操作符
```groovy
Integer x = 123
String s = (String) x

Integer x = 123
String s = x as String
```

## 列表 list
```groovy
def letters = ['a', 'b', 'c', 'd']
assert letters instanceof List  
assert letters.size() == 4

assert letters[0] == 'a'

letters << 'e'
assert letters[ 4] == 'e'
assert letters[-1] == 'e'

assert letters[1, 3] == ['b', 'd']
assert letters[2..4] == ['C', 'd', 'e']
```

## 数组 array
```groovy
String[] arrStr = ['Ananas', 'Banana', 'Kiwi']

assert arrStr instanceof String[]
assert !(arrStr instanceof List)
```

## 映射 map
```groovy
def colors = [red: '#FF0000', green: '#00FF00', blue: '#0000FF']

assert colors['red'] == '#FF0000'    
assert colors.green  == '#00FF00'

colors['pink'] = '#FF00FF'
colors.yellow  = '#FFFF00'
```


```groovy
def key = 'name'
def person = [key: 'Guillaume']      

assert !person.containsKey('name')   
assert person.containsKey('key')
```

## 文件
### 读文件
一行一行读取
```groovy
def file = new File("a.txt")
file.eachLine("UTF-8") {line ->
    println line
}
```

整个内容作为字符串获取
```groovy
def file = new File("a.txt")
println file.text
```

获取机器上的驱动器 "C:\", "D:\"
```groovy
def rootFiles = new File("test").listRoots() 
rootFiles.each {file -> 
    println file.absolutePath 
}
```

递归显示目录及其子目录中的所有文件
```groovy
new File("com").eachFileRecurse() { file ->
    println file.getAbsolutePath()
}
```

### 写文件
```groovy
def file = new File("abc.txt")
def out = file.newPrintWriter("UTF-8")
out.write("Hello World!")
out.write("\n")

out.flush()
out.close()
```

复制文件
```groovy
def src = new File("E:/Example.txt")
def dst = new File("E:/Example1.txt")
dst << src.text
```


## 单元测试
参考[这里](https://www.w3cschool.cn/groovy/groovy_unit_testing.html)

```groovy
import groovy.sql.Sql;

class MySql {
    String version() {
        def sql = Sql.newInstance('jdbc:mysql://localhost:3306/test',
                'root', 'mysqladmin', 'com.mysql.cj.jdbc.Driver')

        def ver
        sql.eachRow('SELECT VERSION()'){ row ->
            ver = row[0]
        }

        sql.close()
        return ver
    }
```

```groovy
import groovy.util.GroovyTestCase

class TestmySql extends GroovyTestCase  {
    void testVersion() {
        def mysql = new MySql()
        def expected = "5.6.38"
        println mysql.version()
        assertToString(mysql.version(), expected)
    }
}
```


## 附件
### maven
```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>org.zbq</groupId>
    <artifactId>groovy01</artifactId>
    <version>0.0.1-SNAPSHOT</version>

    <properties>
        <groovy.version>2.4.8</groovy.version>
    </properties>


    <dependencies>
        <!-- https://mvnrepository.com/artifact/mysql/mysql-connector-java -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>6.0.6</version>
            <scope>runtime</scope>
        </dependency>

        <dependency>
            <groupId>org.apache.maven</groupId>
            <artifactId>maven-model</artifactId>
            <version>2.2.0</version>
        </dependency>
        <dependency>
            <groupId>org.codehaus.groovy</groupId>
            <artifactId>groovy</artifactId>
            <version>${groovy.version}</version>
        </dependency>
        <dependency>
            <groupId>org.codehaus.groovy</groupId>
            <artifactId>groovy-json</artifactId>
            <version>${groovy.version}</version>
        </dependency>

        <!-- https://mvnrepository.com/artifact/org.codehaus.groovy/groovy-sql -->
        <dependency>
            <groupId>org.codehaus.groovy</groupId>
            <artifactId>groovy-sql</artifactId>
            <version>${groovy.version}</version>
        </dependency>

        <dependency>
            <groupId>org.codehaus.groovy</groupId>
            <artifactId>groovy-test</artifactId>
            <version>${groovy.version}</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.codehaus.gmavenplus</groupId>
                <artifactId>gmavenplus-plugin</artifactId>
                <version>1.5</version>
                <dependencies>
                    <dependency>
                        <groupId>org.codehaus.groovy</groupId>
                        <artifactId>groovy-ant</artifactId>
                        <version>${groovy.version}</version>
                    </dependency>
                    <dependency>
                        <groupId>org.codehaus.groovy</groupId>
                        <artifactId>groovy-all</artifactId>
                        <version>${groovy.version}</version>
                    </dependency>

                </dependencies>
                <executions>
                    <execution>
                        <goals>
                            <goal>addSources</goal>
                            <goal>addStubSources</goal>
                            <goal>compile</goal>
                            <goal>execute</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>

</project>
```







