# Gradle笔记
[TOC]

## 修改maven库
参考[这里](http://www.tuicool.com/articles/363iy2n)

1. 在build.gradle中 替换jcenter() 或者 mavenCentral()
```gradle
allprojects {
    repositories {
        maven{ url 'http://maven.aliyun.com/nexus/content/groups/public/'}
    }
}
```

2. 如果对全局都生效的话， 新建一个 ~/.gradle/init.gradle
```gradle
allprojects{
    repositories {
        def REPOSITORY_URL = 'http://maven.aliyun.com/nexus/content/groups/public/'
        all { ArtifactRepository repo ->
            def url = repo.url.toString()
            if ((repo instanceof MavenArtifactRepository) && (url.startsWith('https://repo1.maven.org/maven2') || url.startsWith('https://jcenter.bintray.com'))) {
                project.logger.lifecycle 'Repository ${repo.url} replaced by $REPOSITORY_URL .'
                remove repo
            }
        }
        maven {
            url REPOSITORY_URL
        }
    }
}
```

## 设置代理
新建 gradle.properties, 对当前有效放在项目根目录下， 全局则放在 ~/.gradle/ 下
```init
systemProp.http.proxyHost=127.0.0.1
systemProp.http.proxyPort=10384
systemProp.https.proxyHost=127.0.0.1
systemProp.https.proxyPort=10384
```

## jar包默认路径
jar包默认放在: ~/.gradle/caches/modules-2/files-2.1/

有几种方法[修改](http://blog.csdn.net/yanzi1225627/article/details/52024632)

1. 修改bin下的gradle
```
# Add default JVM options here. You can also use JAVA_OPTS and GRADLE_OPTS to pass JVM options to this script
GRADLE_OPTS=-Dgradle.user.home=/yourpath/gradle/gradle_cache
```

2. 修改当前的 gradle.properties
```ini
gradle.user.home=D:/Cache/.gradle
```

3. 设置环境变量
```sh
export GRADLE_USER_HOME=D:/Cache/.gradle
```




