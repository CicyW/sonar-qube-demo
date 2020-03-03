目的：利用 docker 本地搭建的sonarqube和jenkins 环境，使用sonarqube + jenkins 对maven项目的单元测试，集成测试，代码扫描等进行质量监控管理

## 环境准备
1. 安装docker 
2. 拉取sonarqube和jenkins镜像

    ```
    docker pull sonarqube:7.8-community 
    docker pull cicddraft/jenkins:v0.4

    或者使用 aliyun 镜像仓库
    docker pull registry.cn-hangzhou.aliyuncs.com/cicddraft/jenkins:v0.4 
    docker pull registry.cn-hangzhou.aliyuncs.com/cicddraft/sonarqube:7.8-community 
    ```

## Maven项目使用sonarqube

### 安装sonarqube
1. 启动sonarqube:

    ```
    docker run -d -m=4g --restart=always --name tw-sonarqube -p 9000:9000 sonarqube:7.8-community

    或者aliyun 镜像
    docker run -d -m=4g --restart=always --name tw-sonarqube -p 9000:9000 registry.cn-hangzhou.aliyuncs.com/cicddraft/sonarqube:7.8-community
    ```
2. 浏览器中访问： http://localhost:9000 进入 sonarqube 页面


### 在maven 项目中使用sonar
1. 在项目根目录的pom.xml文件加入如下配置，引入jacoco插件
    ```
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
2. 从命令行进入项目的根目录下，依次执行如下命令
   ```
    mvn clean
    mvn install
    mvn sonar:sonar -D sonar.host.url=http://localhost:9000 -Dsonar.login=admin \
    -Dsonar.password=admin -Dsonar.core.codeCoveragePlugin=jacoco -Dsonar.coverage.jacoco.xmlReportPaths=target/coverage-reports/jacoco.xml
   ```
3. 执行完后，再次回到浏览器中访问： http://localhost:9000

## 在jenkins pipeline中使用sonarqube

### jenkins环境搭建
1. 启动jenkins
    ``` 
    docker run --name --restart=always tw-jenkins -d -p 8081:8080 -v /var/run/docker.sock:/var/run/docker.sock cicddraft/jenkins:v0.4
    
    或者aliyun 镜像
    docker run -d --restart=always --name tw-jenkins -d -p 8081:8080 -v /var/run/docker.sock:/var/run/docker.sock registry.cn-hangzhou.aliyuncs.com/cicddraft/jenkins:v0.4
    ```
2. 浏览器中访问： http://localhost:8081 进入 sonarqube 页面

