# GCP with MariaDB

## Step 1. 註冊 Google Cloud Platform (GCP)
  * 準備一張信用卡，於註冊時需過卡，並非真正刷卡，到期不會自動扣款。

## Step 2. 建立 Compute engine VM
  * Name
    * 隨意取名。
  * Region
    * 選擇 asia-east-1(Taiwan) 與 asia-east1-c，資料中心於彰化，速度比較快。
  * Machine type
    * 無特殊需求設定最小即可。
  * Boot disk
    * 選擇 Debian GNU/Linux 9(stretch)或其他廠牌Linux皆可，並設定SSD，看需求設定SSD大小，建議15G。
  * Identity and API access
    * 選擇預設。
     
## Step 3. 設定SSH連線
  * 於終端機輸入ssh-keygen，連續按Enter至結束。
  * 於使用者資料夾下找到『.ssh』中的『id_rsa.pub』檔案，將其內容複製到 VM 的 SSH Key 中。
  * 於終端機便可以直接輸入```$ ssh {VM ip address}```進行連線。
  * 若出現『WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!』錯誤訊息，輸入```$ ssh -R {VM ip address}```，再重新連線。

## Step 4. 修改VM root密碼
 * ```$ sudo passwd root```。
 * 輸入新密碼。
 * 再次輸入。

## Step 5. 安裝MariaDB
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
   * ```bind-address = 0.0.0.0```
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
