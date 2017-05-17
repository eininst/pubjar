#jar包自动化发布


## 说明
发布机依赖 maven 和 git


执行流程
   - git clone gitAddress 如果不存在
   - git pull
   - 执行maven 命令 
   - sftp 上传tar包
   - 解压tar包
   - 检查这个项目的进程是否存在 (存在会执行 kill -9)
   - java ${jvmOpts} -jar lib 1>stdout.log 2>stderr.log
    
##例子 :


- npm install pubjar
- create deploy.js
```
var pubjar= require('pubjar')
pubjar({
    gitAddress: 'git@git.chongkouwei.com:wangziqing/DA-parent.git',
    sourcePath: '/Users/wangziqing/github/pj-test',
    remotePath: '/root',
    test: [{
        name: 'vertx-web',
        host: '123.123.123.123',
        username: 'root',
        password: '123456',
        jvmOpts : '-Xms512m -Xmx512m -XX:NewSize=128m',
        mvn: 'mvn clean install -Pprod -Dmaven.test.skip=true'
    }],
    prod: [{
        name: 'vertx-web',
        host: '123.123.123.123',
        username: 'admin',
        privateKey: require('fs').readFileSync('/Users/wangziqing/.ssh/id_rsa')
        jvmOpts : '-Xms512m -Xmx512m -XX:NewSize=128m',
        mvn: 'mvn clean install -Pprod -Dmaven.test.skip=true'
    }]
})
```
## 开始发布
`node deploy.js --module=test:vertx-web --branch=master`


## maven 例子

```
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-assembly-plugin</artifactId>
    <version>2.6</version>
    <configuration>
        <finalName>${project.artifactId}</finalName>
        <descriptors>
            <descriptor>src/main/assembly/distribution.xml</descriptor>
        </descriptors>
    </configuration>
    <executions>
        <execution>
            <id>make-assembly</id>
                <phase>package</phase>
                <goals>
                    <goal>single</goal>
                </goals>
        </execution>
    </executions>
</plugin>
```

###distribution.xml
 - assembly ID 必须为deploy 
 - format 必须为tar.gz
```
<assembly>
    <id>deploy</id>
    <formats>
        <format>tar.gz</format>
    </formats>
    <includeBaseDirectory>false</includeBaseDirectory>
    <fileSets>
        <fileSet>
            <includes>
                <include>README*</include>
                <include>LICENSE*</include>
                <include>NOTICE*</include>
            </includes>
        </fileSet>
        <fileSet>
            <directory>deploy</directory>
            <outputDirectory></outputDirectory>
        </fileSet>
        <fileSet>
            <directory>src/main/java/assets</directory>
            <outputDirectory>assets</outputDirectory>
        </fileSet>
        <fileSet>
            <directory>src/main/java/views</directory>
            <outputDirectory>views</outputDirectory>
        </fileSet>
        <fileSet>
            <directory>target</directory>
            <outputDirectory>lib</outputDirectory>
            <includes>
                <include>${project.artifactId}-${project.version}.jar</include>
            </includes>
        </fileSet>
    </fileSets>
</assembly>
```