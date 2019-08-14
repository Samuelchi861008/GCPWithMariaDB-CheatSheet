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
  * 於終端機便可以直接輸入```$ ssh {VM ip address}```。

## Step 4. 修改root密碼
 * ```$ sudo passwd root```。
 * 輸入新密碼。
 * 再次輸入。
