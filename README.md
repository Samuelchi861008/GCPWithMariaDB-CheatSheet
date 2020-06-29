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
    * 選擇 Debian GNU/Linux 9(stretch) 30GB HDD (永久免費方案)。
    * 選擇 Debian GNU/Linux 9(stretch) 15GB SSD，速度比較快，但非永久免費方案。
  * Identity and API access
    * 選擇預設。
  * Firewall
    * 允許 HTTP 流量、允許 HTTPS 流量。
    * 選擇預設。
     
## Step 3. 設定SSH連線
  * 於自己電腦終端機(命令提示字元)輸入ssh-keygen，連續按Enter至結束。
  * 於自己電腦使用者資料夾下找到『.ssh』中的『id_rsa.pub』檔案，將其內容複製到 VM 的 SSH Key 中。
  * 於終端機便可以直接輸入```$ ssh {VM ip address}```進行連線。
  * 若出現『WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!』錯誤訊息，輸入```$ ssh-keygen -R {VM ip address}```，再重新連線。

## Step 4. 修改VM root密碼
 * ```$ sudo passwd root```。
 * 輸入新密碼。
 * 再次輸入。

## Step 5. 安裝MariaDB (即 MySQL)
 * 更新apt和安裝相依套件
   * ```$ sudo apt update && apt upgrade```
   * ```$ sudo apt-get install software-properties-common dirmngr```
 * 添加 MariaDB signing key
   * ```$ sudo apt-key adv --recv-keys --keyserver keyserver.ubuntu.com 0xF1656F24C74CD1D8```
 * 下載 MariaDB
   * ```$ sudo add-apt-repository 'deb [arch=amd64,i386,ppc64el] http://ftp.ubuntu-tw.org/mirror/mariadb/repo/10.3/debian stretch main'```
 * 安裝 mariadb-serverMariaDB
   * ```$ sudo apt-get install mariadb-server```
 * 檢查版本
   * ```$ mysql -V```

## Step 6. 設定 MariaDB
 * 切換VM權限至root
   * ```$ su```
   * 輸入VM root密碼
 * 設定MariaDB
   * ```# mysql_secure_installation```
   * 變更MariaDB root密碼，輸入『Y』，即可變更密碼
   * 移除匿名使用者，輸入『Y』
   * 移除遠端操作root的權限，輸入『n』
   * 移除測試資料庫，輸入『Y』
   * 重新載入權限的table資訊，輸入『Y』
 * 登入MariaDB root
   * ```$ mysql -u root -p```
   * 輸入MariaDB root密碼
 * MariaDB指令
   * 確認MariaDB版本 
   
     ```> SELECT VERSION();```
   * 查看資料庫 
   
     ```> show databases;```
   * 查看資料庫資訊 
   
     ```> status``` 
   * 創建資料庫
    
     ```> CREATE DATABASE `{database name}`;``` 
     
 * 設定資料庫可以遠端連線
   * ```$ sudo vi /etc/mysql/mariadb.conf.d/50-server.cnf```
   * 將 ```bind-address = 127.0.0.1``` 改成 ```bind-address = 0.0.0.0```
   * 進入資料庫 ```> GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '遠端連線密碼' WITH GRANT OPTION;```
   * 重啟資料庫 ```$ service mysqld restart ```
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
  

## Step 8. 安裝 Apach
  * ```$ sudo apt-get install apache2```
  * ```$ sudo service apache2 restart```
  * 即可在 /var/www/html 中放置已寫好 html 專案
  * 並使用外部IP連線
  * 若要關閉Apache index頁面，至 /etc/apache2/apache2.conf，將 ```Options Indexes FollowSymLinks``` 改成 ```Options FollowSymLinks```
  * Apache 指令
    * 開啟 ```$ sudo service apache2 start```
    * 重啟 ```$ sudo service apache2 restart```
    * 停止 ```$ sudo service apache2 stop```


## Step 9. 設定時區
  * 設定linux主機時區
    * ```$ sudo dpkg-reconfigure tzdata```
    * 選擇 Asia -> Taipei
  * 設定 MySQL 時區
    * ```> SET GLOBAL time_zone = '+8:00';```
  * 最後重啟資料庫 ```$ service mysqld restart ```


## Step 10. Code Server 建置 (可選)
  * 下載所有檔案並將 code-server 整個資料夾傳入 Server
    * https://drive.google.com/file/d/1D7gtcqem4mDxmRcM94_M7pjevLDjkQQa/view?usp=sharing
  * 在 Server 上創一個 shell 檔案，內容為：
    * ```export PASSWORD="密碼" && nohup ./code-server --host 0.0.0.0 --port 80 &```
  * 在 /etc/crontab 定時啟動 shell 檔案
