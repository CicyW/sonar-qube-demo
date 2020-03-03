目的：利用 docker 本地搭建的sonarqube和jenkins 环境，使用sonarqube + jenkins 对maven项目的单元测试，集成测试，代码扫描等进行质量监控管理

> 由于jenkins、sonarqube安装过程从简,不建议直接使用本文档用于生产等重要环境。

## 一、环境准备
1. 安装 docker 
2. 拉取 sonarqube(7.8-community) 和 jenkins(2.176.3)镜像

```bash
docker pull sonarqube:7.8-community 
docker pull cicddraft/jenkins:v0.4

#或者使用 aliyun 镜像仓库
docker pull registry.cn-hangzhou.aliyuncs.com/cicddraft/jenkins:v0.4 
docker pull registry.cn-hangzhou.aliyuncs.com/cicddraft/sonarqube:7.8-community 
```

## 二、Maven项目使用sonarqube

### 2.1安装sonarqube
- 启动sonarqube:

```bash
docker run -d -m=4g --restart=always --name tw-sonarqube -p 9000:9000 sonarqube:7.8-community

#或者aliyun 镜像
docker run -d -m=4g --restart=always --name tw-sonarqube -p 9000:9000 registry.cn-hangzhou.aliyuncs.com/cicddraft/sonarqube:7.8-community
```

- 浏览器中访问： http://localhost:9000 进入 sonarqube 页面

### 2.2在maven 项目中使用sonar
- 在项目根目录的pom.xml文件加入如下配置，引入jacoco插件
```xml
    <build>
        <plugins>
            <plugin>
                <groupId>org.jacoco</groupId>
                <artifactId>jacoco-maven-plugin</artifactId>
                <version>0.8.1</version>
                <configuration>
                    <skip>false</skip>
                </configuration>
                <executions>
                    <execution>
                        <goals>
                            <goal>prepare-agent</goal>
                        </goals>
                    </execution>
                    <execution>
                        <configuration>
                            <outputDirectory>${basedir}/target/coverage-reports</outputDirectory>
                        </configuration>
                        <id>report</id>
                        <phase>test</phase>
                        <goals>
                            <goal>report</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
```
- 从命令行进入项目的根目录下，依次执行如下命令
   ```
    mvn clean
    mvn install
    mvn sonar:sonar -D sonar.host.url=http://localhost:9000 -Dsonar.login=admin \
    -Dsonar.password=admin -Dsonar.core.codeCoveragePlugin=jacoco -Dsonar.coverage.jacoco.xmlReportPaths=target/coverage-reports/jacoco.xml
   ```
- 执行完后，再次回到浏览器中访问： http://localhost:9000

## 三、在jenkins pipeline中使用sonarqube

### 3.1 jenkins环境搭建
-  启动jenkins
    ``` 
    docker run --name --restart=always tw-jenkins -d -p 8081:8080 -v /var/run/docker.sock:/var/run/docker.sock cicddraft/jenkins:v0.4
    
    或者aliyun 镜像
    docker run -d --restart=always --name tw-jenkins -d -p 8081:8080 -v /var/run/docker.sock:/var/run/docker.sock registry.cn-hangzhou.aliyuncs.com/cicddraft/jenkins:v0.4
    ```
-  浏览器中访问： http://localhost:8081 进入 sonarqube 页面


### 3.2 利用pieline将质量报告推送到sonarqube
> 补充pipeline 示例，或 Jenkinsfile_v2

### 3.3 当SonarQube quality 不通过时，jenkins job 标记为失败
- 在Jenkins上安装插件 **sonar scaner**
“Manage Jenkins” -> "Manage Plugins" -> "Available" 

搜索插件 **SonarQube Scanner for Jenkins**，点击 “install without restart”

- 在sonarqube页面创建token ，以便共Jenkins使用
“My Account” -> "Security" -> "Generate Tokens"

- 在jenkins 上配置sonarqube信息
“Manage Jenkins” -> "Configure System" -> ""

找到**SonarQube servers**，补充以下信息

|    key    |    value   | Note  |
|:-----------|-----------|---------
|    Name   | sonarqube-123 |   定义一个名称，后续pipeline `withSonarQubeEnv('sonarqube-123')`与此处保持一致  |
|	Server URL | <YOUR_SONAR_ADDRESS>|   sonarqube的地址          |
|  Server authentication token |  <下拉选择>     |   先在jenkins凭证中添加上一步生成的 token，然后选择 |

更新jenkinsfile,添加如下示例内容：

```groovy
        stage('Sonar-Scan'){
            agent any
            steps{
                withSonarQubeEnv('sonarqube-123') {
                    sh '''
                    /usr/local/bin/mvn sonar:sonar -Dsonar.projectKey=sonarqube
                    '''
                }
            }
        }

        stage("Quality Gate") {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
```

> 补充pipeline示例，或 Jenkinsfile_v3
