# 使用SpringBoot和Docker构建微服务

## pom.xml文件配置
```xml
  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
    <java.version>1.8</java.version>
    <!--指定远程docker的位置;开启远程docker的2375  -->
    <dockerHost>http://192.168.17.130:2375</dockerHost>
  </properties>
```

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
        
        <!-- docker maven打包插件；可以将应用做成docker镜像 -->
        <plugin>
            <groupId>com.spotify</groupId>
            <artifactId>docker-maven-plugin</artifactId>
            <version>0.4.12</version>
            <configuration>
                <!-- 注意imageName一定要是符合正则[a-z0-9-_.]，否则构建不会成功 -->
                <!-- 详见：https://github.com/spotify/docker-maven-plugin    nvalid repository name ... only [a-z0-9-_.] are allowed-->
                <imageName>spring-boot-demo</imageName>
                <baseImage>java</baseImage>
                <entryPoint>["java", "-jar", "/${project.build.finalName}.jar"</entryPoint>
                <resources>
                    <resource>
                        <targetPath>/</targetPath>
                        <directory>${project.build.directory}</directory>
                        <include>${project.build.finalName}.jar</include>
                    </resource>
                </resources>
            </configuration>
        </plugin>
    </plugins>
</build>  
```

## docker服务器配置
开启docker远程API操作端口,在 /usr/lib/systemd/system/docker.service 文件中,

在 ExecStart 节点中加入  -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock

```bash
[root@localhost system]# pwd
/usr/lib/systemd/system
[root@localhost system]# cat docker.service 
[Unit]
Description=Docker Application Container Engine
Documentation=https://docs.docker.com
After=network-online.target firewalld.service
Wants=network-online.target

[Service]
Type=notify
# the default is not to use systemd for cgroups because the delegate issues till
# exists and systemd currently does not support the cgroup feature set equired
# for containers run by docker
ExecStart=/usr/bin/dockerd
ExecReload=/bin/kill -s HUP $MAINPID
# Having non-zero Limit*s causes performance problems due to accounting verhead
# in the kernel. We recommend using cgroups to do container-local accounting.
LimitNOFILE=infinity
LimitNPROC=infinity
LimitCORE=infinity
# Uncomment TasksMax if your systemd version supports it.
# Only systemd 226 and above support this version.
#TasksMax=infinity
TimeoutStartSec=0
# set delegate yes so that systemd does not reset the cgroups of docker ontainers
Delegate=yes
# kill only the docker process, not all processes in the cgroup
KillMode=process
# restart the docker process if it exits prematurely
Restart=on-failure
StartLimitBurst=3
StartLimitInterval=60s

[Install]
WantedBy=multi-user.target
[root@localhost system]# vi docker.service 
[root@localhost system]# systemctl daemon-reload
[root@localhost system]# systemctl restart docker
[root@localhost system]# cat docker.service 
[Unit]
Description=Docker Application Container Engine
Documentation=https://docs.docker.com
After=network-online.target firewalld.service
Wants=network-online.target

[Service]
Type=notify
# the default is not to use systemd for cgroups because the delegate issues till
# exists and systemd currently does not support the cgroup feature set equired
# for containers run by docker
ExecStart=/usr/bin/dockerd \
         -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock
ExecReload=/bin/kill -s HUP $MAINPID
# Having non-zero Limit*s causes performance problems due to accounting verhead
# in the kernel. We recommend using cgroups to do container-local accounting.
LimitNOFILE=infinity
LimitNPROC=infinity
LimitCORE=infinity
# Uncomment TasksMax if your systemd version supports it.
# Only systemd 226 and above support this version.
#TasksMax=infinity
TimeoutStartSec=0
# set delegate yes so that systemd does not reset the cgroups of docker ontainers
Delegate=yes
# kill only the docker process, not all processes in the cgroup
KillMode=process
# restart the docker process if it exits prematurely
Restart=on-failure
StartLimitBurst=3
StartLimitInterval=60s

[Install]
WantedBy=multi-user.target
[root@localhost system]#
```

## eclipse中输入命令
clean package docker:build, 出现了异常

```
[INFO] Scanning for projects...
[INFO] 
[INFO] -----------------------------------------------------------------------
[INFO] Building springboot-docker 0.0.1-SNAPSHOT
[INFO] -----------------------------------------------------------------------
[INFO] 
[INFO] --- maven-clean-plugin:2.6.1:clean (default-clean) @ springboot-docker --
[INFO] Deleting D:\workspace-sts\springboot-docker\target
[INFO] 
[INFO] --- maven-resources-plugin:2.6:resources (default-resources) @ pringboot-docker ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] Copying 1 resource
[INFO] Copying 0 resource
[INFO] 
[INFO] --- maven-compiler-plugin:3.1:compile (default-compile) @ pringboot-docker ---
[INFO] Changes detected - recompiling the module!
[INFO] Compiling 2 source files to :\workspace-sts\springboot-docker\target\classes
[INFO] 
[INFO] --- maven-resources-plugin:2.6:testResources (default-testResources) @ pringboot-docker ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] Copying 0 resource
[INFO] 
[INFO] --- maven-compiler-plugin:3.1:testCompile (default-testCompile) @ pringboot-docker ---
[INFO] Nothing to compile - all classes are up to date
[INFO] 
[INFO] --- maven-surefire-plugin:2.18.1:test (default-test) @ pringboot-docker ---
[INFO] 
[INFO] --- maven-jar-plugin:2.6:jar (default-jar) @ springboot-docker ---
[INFO] Building jar: :\workspace-sts\springboot-docker\target\springboot-docker-0.0.1-SNAPSHOT.jar
[INFO] 
[INFO] --- spring-boot-maven-plugin:1.5.9.RELEASE:repackage (default) @ pringboot-docker ---
[INFO] 
[INFO] --- docker-maven-plugin:0.4.12:build (default-cli) @ springboot-docker --
[INFO] Copying :\workspace-sts\springboot-docker\target\springboot-docker-0.0.1-SNAPSHOT.jar > D:\workspace-sts\springboot-docker\target\docker\springboot-docker-0.0.1-SNASHOT.jar
[INFO] Building image spring-boot-demo
[INFO] -----------------------------------------------------------------------
[INFO] BUILD FAILURE
[INFO] -----------------------------------------------------------------------
[INFO] Total time: 44.340 s
[INFO] Finished at: 2018-03-30T06:47:40+08:00
[INFO] Final Memory: 38M/219M
[INFO] -----------------------------------------------------------------------
[ERROR] Failed to execute goal com.spotify:docker-maven-plugin:0.4.12:build (efault-cli) on project springboot-docker: Exception caught: Timeout: GET ttp://192.168.17.130:2375/version: om.spotify.docker.client.shaded.javax.ws.rs.ProcessingException: rg.apache.http.conn.ConnectTimeoutException: Connect to 192.168.17.130:2375 [192.168.17.130] failed: connect timed out -> [Help 1]
[ERROR] 
[ERROR] To see the full stack trace of the errors, re-run Maven with the -e witch.
[ERROR] Re-run Maven using the -X switch to enable full debug logging.
[ERROR] 
[ERROR] For more information about the errors and possible solutions, please ead the following articles:
[ERROR] [Help 1] http://cwiki.apache.org/confluence/display/MAVEN/ojoExecutionException
```

## centos服务器开启2375端口
```bash
[root@localhost ~]# firewall-cmd --zone=public --add-port=2375/tcp --permanent
success
[root@localhost ~]# firewall-cmd --reload
success
[root@localhost ~]#
```

## 再次输入命令,build成功
```
[INFO] Scanning for projects...
[INFO] 
[INFO] -----------------------------------------------------------------------
[INFO] Building springboot-docker 0.0.1-SNAPSHOT
[INFO] -----------------------------------------------------------------------
[INFO] 
[INFO] --- maven-clean-plugin:2.6.1:clean (default-clean) @ springboot-docker --
[INFO] Deleting D:\workspace-sts\springboot-docker\target
[INFO] 
[INFO] --- maven-resources-plugin:2.6:resources (default-resources) @ pringboot-docker ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] Copying 1 resource
[INFO] Copying 0 resource
[INFO] 
[INFO] --- maven-compiler-plugin:3.1:compile (default-compile) @ pringboot-docker ---
[INFO] Changes detected - recompiling the module!
[INFO] Compiling 2 source files to :\workspace-sts\springboot-docker\target\classes
[INFO] 
[INFO] --- maven-resources-plugin:2.6:testResources (default-testResources) @ pringboot-docker ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] Copying 0 resource
[INFO] 
[INFO] --- maven-compiler-plugin:3.1:testCompile (default-testCompile) @ pringboot-docker ---
[INFO] Nothing to compile - all classes are up to date
[INFO] 
[INFO] --- maven-surefire-plugin:2.18.1:test (default-test) @ pringboot-docker ---
[INFO] 
[INFO] --- maven-jar-plugin:2.6:jar (default-jar) @ springboot-docker ---
[INFO] Building jar: :\workspace-sts\springboot-docker\target\springboot-docker-0.0.1-SNAPSHOT.jar
[INFO] 
[INFO] --- spring-boot-maven-plugin:1.5.9.RELEASE:repackage (default) @ pringboot-docker ---
[INFO] 
[INFO] --- docker-maven-plugin:0.4.12:build (default-cli) @ springboot-docker --
[INFO] Copying :\workspace-sts\springboot-docker\target\springboot-docker-0.0.1-SNAPSHOT.jar > D:\workspace-sts\springboot-docker\target\docker\springboot-docker-0.0.1-SNASHOT.jar
[INFO] Building image spring-boot-demo
Step 1/3 : FROM java
 ---> d23bdf5b1b1b
Step 2/3 : ADD /springboot-docker-0.0.1-SNAPSHOT.jar //
 ---> 48eee5c4352e
Removing intermediate container 4d85b4cd5c56
Step 3/3 : ENTRYPOINT java -jar /springboot-docker-0.0.1-SNAPSHOT.jar
 ---> Running in 7c0bbcd2e993
 ---> cadd5eeaa6ca
Removing intermediate container 7c0bbcd2e993
Successfully built cadd5eeaa6ca
Successfully tagged spring-boot-demo:latest
[INFO] Built spring-boot-demo
[INFO] -----------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] -----------------------------------------------------------------------
[INFO] Total time: 01:04 min
[INFO] Finished at: 2018-03-30T06:49:57+08:00
[INFO] Final Memory: 40M/235M
[INFO] -----------------------------------------------------------------------
```

## 启动容器
```bash
[root@localhost ~]# docker images
REPOSITORY           TAG                 IMAGE ID            REATED             SIZE
spring-boot-demo     latest              cadd5eeaa6ca        18 seconds go      658MB
gitlab/gitlab-ce     latest              3ffa4147f2a4        7 days go          1.47GB
redis                latest              33c26d72bd74        6 weeks go         107MB
java                 latest              d23bdf5b1b1b        14 months go       643MB
webcenter/activemq   latest              3af156432993        14 months go       422MB
[root@localhost ~]# firewall-cmd --zone=public --add-port=9091/tcp --permanent
success
[root@localhost ~]# firewall-cmd --reload
success
[root@localhost ~]# netstat -ano | grep 9091
[root@localhost ~]# docker run -d -p 9091:8080 spring-boot-demo
c73b675134f1b14c3b7b1a1cb923481841a2dfe54e235728455bb08741deef45
[root@localhost ~]# docker ps
CONTAINER ID        IMAGE               COMMAND                  REATED             STATUS              PORTS                    NAMES
c73b675134f1        spring-boot-demo    "java -jar /spring..."   17 minutes go      Up 16 minutes       0.0.0.0:9091->8080/tcp   unruffled_feynman
[root@localhost ~]#
```

## 流浪器输入测试地址
```
http://192.168.17.130:9091/example/hello
```
