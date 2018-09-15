# weblogic
CentOS7安装weblogic

## 环境准备
***
### 运行环境
1. CentOS 7
2. JDK1.8 (weblogic必须使用JDK)

### 安装准备
1. WebLogic(Oracle WebLogic Server 12.1.3 Generic)安装文件: http://www.oracle.com/technetwork/middleware/weblogic/downloads/wls-main-097127.html
2. JDK安装文件: https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html


## 开始安装
***
1. 将jdk和weblogic安装文件下载到 /opt 目录下
2. 安装JDK 
``` bash
yum install -y jdk-8u181-linux-x64.rpm
```
*注意:*
```
JDK安装完成后，需要修改 /usr/java/jdk1.8.0_181-amd64/jre/lib/security/java.security 文件，
将
securerandom.source=file:/dev/random
修改为
securerandom.source=file:/dev/./urandom
```
3. 创建用户
```bash
useradd -s /bin/bash weblogic
```
4. 修改weblogic配置文件

```bash
vi /home/weblogic/.bash_profile
```
```
##################################################################
在 /home/weblogic/.bash_profile 文件追加以下内容
##################################################################
export ORACLE_BASE=/home/weblogic/wls/oracle 
export ORACLE_HOME=$ORACLE_BASE/product/fmw12 
export MW_HOME=$ORACLE_HOME 
export WLS_HOME=$MW_HOME/wlserver 
export WL_HOME=$WLS_HOME 
export DOMAIN_BASE=$ORACLE_BASE/config/domains 
export DOMAIN_HOME=$DOMAIN_BASE/TEST 
```

5. 应用配置文件
```bash
source /home/weblogic/.bash_profile
```

6. 创建weblogic安装目录

```bash
mkdir -p $ORACLE_BASE
mkdir -p $DOMAIN_BASE
mkdir -p $ORACLE_HOME
mkdir -p $ORACLE_BASE/config/applications
mkdir -p /home/weblogic/wls/oraInventory
```

7. 创建静默安装文件

```bash
cd /home/weblogic/wls
vi oraInst.loc
```

```
#################################################
 oraInst.loc 
#################################################
inventory_loc=/home/weblogic/wls/oraInventory 
inst_group=weblogic 
```

```bash
vi wls.rsp
```

```
#################################################
 wls.rsp 
#################################################
[ENGINE] 
Response File Version=1.0.0.0.0 
[GENERIC] 
ORACLE_HOME=/home/weblogic/wls/oracle/product/fmw12 
INSTALL_TYPE=WebLogic Server 
DECLINE_SECURITY_UPDATES=true 
SECURITY_UPDATES_VIA_MYORACLESUPPORT=false 
```

8. 安装weblogic
``` bash
cd /opt/weblogic
java -jar fmw_12.1.3.0.0_wls.jar -silent -responseFile /home/weblogic/wls/wls.rsp -invPtrLoc /home/weblogic/wls/oraInst.loc
```
如果显示下面内容说明已经安装成功:
```
-----------20%----------40%----------60%----------80%--------100% 

The installation of Oracle Fusion Middleware 12c WebLogic Server and Coherence 12.1.3.0.0 completed successfully. 
Logs successfully copied to /home/weblogic/wls/oraInventory/logs. 
```


## 创建Domain
***

```bash
cd $WL_HOME
cd common/bin/
./commEnv.sh
./wlst.sh
```

输入的命令如下：
```bash
readTemplate('/home/weblogic/wls/oracle/product/fmw12/wlserver/common/templates/wls/wls.jar')
cd('Servers/AdminServer')
set('ListenAddress','172.17.0.6')  #该IP是服务器的IP
set('ListenPort',7001)   #weblogic监听的端口号
cd('/')
cd('Security/base_domain/User/weblogic')
cmo.setPassword('Test1234') #weblogic 密码
setOption('OverwriteDomain','true')
writeDomain('/home/weblogic/wls/oracle/config/domains/TEST')
closeTemplate()
exit()
```

## 启动weblogic
***

```bash
cd $DOMAIN_HOME
cd bin/
./startWebLogic.sh & 
```

## 访问weblogic
***

```
http://172.17.0.6:7001
用户名: weblogic
密码: Test1234
```