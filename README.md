# GCP with DB cheatsheet (free forever)

## Step 1. 註冊 Google Cloud Platform (GCP)
  * 網址：https://cloud.google.com/free/docs/gcp-free-tier
  * 準備一張信用卡，於註冊時需過卡，並非真正刷卡，到期不會自動扣款。

## Step 2. 從 Compute engine 建立 VM 執行個體
  * Name
    * 隨意取名。
  * Region
    * 選擇 us-west1(奧勒岡州) 與 us-west1-b (永久免費方案)。
    * 選擇 asia-east-1(Taiwan) 與 asia-east1-c，資料中心於彰化，速度比較快，但非永久免費方案。
  * Machine type
    * f1-micro (1 個 vCPU，614MB 記憶體)(永久免費方案)。
  * Boot disk
    * 選擇 Ubuntu 18.04 LTS 30GB HDD (永久免費方案)。
    * 選擇 Ubuntu 18.04 LTS 15GB SSD，速度比較快，但非永久免費方案。
  * Identity and API access
    * 選擇預設。
  * Firewall
    * 允許 HTTP 流量、允許 HTTPS 流量。
    * 選擇預設。


## Step 3. 修改VM root密碼
 * ```$ sudo passwd root```。
 * 輸入新密碼。
 * 再次輸入。

     
## Step 4. 設定SSH連線
  * 於自己電腦終端機(命令提示字元)輸入ssh-keygen，連續按Enter至結束。
  * 於自己電腦使用者資料夾下找到『.ssh』中的『id_rsa.pub』檔案，將其內容複製到 VM 的 SSH Key 中。
  * 於終端機便可以直接輸入```$ ssh {VM ip address}```進行連線。
  * 若出現『WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!』錯誤訊息，輸入```$ ssh-keygen -R {VM ip address}```，再重新連線。
  * 進到 /etc/ssh/sshd_config 進行設定不需再使用 SSH Key
    * 找到 ```#Port 22``` 將註解移除
    * 找到 ```PasswordAuthenticate no``` 將 no 改成 yes
    * 找到 ```PermitRootLogin without-password``` 將 without-password 改成 yes
    * 在檔案後面加入 ```AllowUsers {其餘使用者名稱}```
  * 重啟 SSH
    * ```$ sudo service ssh restart```

## Step 5. 安裝 MySQL
 * 下載與安裝 MySQL
   * ```$ sudo apt install mysql-server```

## Step 6. 設定 MySQL
 * 把 root 改為使用密碼驗證登入
   * ```$ sudo mysql```
   * ```> SELECT user, authentication_string, plugin, host FROM mysql.user WHERE user = 'root';```
   * ```> ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '{root 要修改的密碼}';```
 * 讓設定生效
   * ```> FLUSH PRIVILEGES;``` 
 * 設定MariaDB
 * 以 root 登入 MySQL
   * ```$ mysql -u root -p```
   * 輸入重新設定後的 root 密碼
 * MySQL 常用指令
   * 查看所有資料庫 
     ```> show databases;```
   * 查看資料庫資訊  
     ```> status``` 
   * 創建資料庫  
     ```> CREATE DATABASE `{database name}`;``` 
 * 設定文字編碼 (可支援 iPhone 表情符號)，在 /etc/mysql/my.cnf 貼上以下程式
   ```
   [client]
   default-character-set=utf8mb4


   [mysql]
   default-character-set=utf8mb4


   [mysqld]
   character-set-server = utf8mb4
   collation-server = utf8mb4_unicode_ci
   init_connect = 'SET NAMES utf8mb4'
   character-set-client-handshake = false
   ```
     
 * 設定資料庫可以遠端連線
   * ```$ sudo vi /etc/mysql/mysql.conf.d```
   * 將 ```bind-address = 127.0.0.1``` 改成 ```bind-address = 0.0.0.0```
   * 進入資料庫 ```> GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '遠端連線密碼' WITH GRANT OPTION;```
   * 重啟資料庫 ```$ sudo /etc/init.d/mysql restart ```
   * 方可使用 MySQL workbench 或其他軟體進行連線
   
 * 資料庫備份與還原
   * 備份 
     
     ```$ mysqldump -u root -p --all-databases > backup.sql;``` 
     
   * 還原 
   
     ```$ mysql -u root -p < mysql.sql``` 
     
 
## Step 7. GCP Firewall Rules設定
  * 至 VPC network 新增防火牆規則
  * 名稱(隨意)、目標代碼(隨意)、來源 IP 範圍(0.0.0.0/0)、port(tcp:3306)、其他不變
  * 回到VM 將『目標代碼』加入『網路標記』


## Step 8. 設定時區
  * 設定linux主機時區
    * ```$ sudo dpkg-reconfigure tzdata```
    * 選擇 Asia -> Taipei
  * 設定 MySQL 時區
    * ```> SET GLOBAL time_zone = '+8:00';```
  * 最後重啟資料庫 ```$ sudo /etc/init.d/mysql restart ```


## Step 9. 安裝 Apach (可選)
  * ```$ sudo apt-get install apache2```
  * ```$ sudo service apache2 restart```
  * 即可在 /var/www/html 中放置已寫好 html 專案
  * 並使用外部IP連線
  * 若要關閉Apache index頁面，至 /etc/apache2/apache2.conf，將 ```Options Indexes FollowSymLinks``` 改成 ```Options FollowSymLinks```
  * Apache 指令
    * 開啟 ```$ sudo service apache2 start```
    * 重啟 ```$ sudo service apache2 restart```
    * 停止 ```$ sudo service apache2 stop```


## Step 10. Code Server 建置 (可選)
  * 下載所有檔案並將整個資料夾傳入 Server
    * https://drive.google.com/file/d/1D7gtcqem4mDxmRcM94_M7pjevLDjkQQa/view?usp=sharing
  * 在 Server 上創一個 shell 檔案，內容為：
    * ```export PASSWORD="密碼" && nohup ./code-server --host 0.0.0.0 --port 80 &```
  * 在 /etc/crontab 定時啟動 shell 檔案
 
 
 ## Step 11. 安裝 Jenkins (可選)
  * ```$ sudo apt update```
  * ```$ sudo apt install openjdk-8-jdk```
  * ```$ wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -```
  * ```$ sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'```
  * ```$ sudo apt install jenkins```
  * ```$ sudo systemctl start jenkins```
  * ```$ sudo systemctl enable jenkins```
  * 即可前往網址，預設port為8080
  * 查找預設密碼：
    * ```$ sudo cat /var/lib/jenkins/secrets/initialAdminPassword```
