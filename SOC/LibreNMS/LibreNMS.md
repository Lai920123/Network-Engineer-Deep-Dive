# LibreNMS #

## 虛擬機規格 ##

- 2 CPU 
- 2 GB RAM

## 安裝方式 ##

```bash
#安裝所需套件
sudo apt install acl curl fping git graphviz imagemagick mariadb-client mariadb-server mtr-tiny nginx-full nmap php-cli php-curl php-fpm php-gd php-gmp php-json php-mbstring php-mysql php-snmp php-xml php-zip python3-command-runner python3-dotenv python3-pip python3-psutil python3-pymysql python3-redis python3-setuptools python3-systemd rrdtool snmp snmpd traceroute unzip whois -y
#新增使用者
sudo useradd librenms -d /opt/librenms -M -r -s "$(which bash)"
#進入opt目錄
cd /opt
#git下載使用者家目錄
git clone https://github.com/librenms/librenms.git
#變更目錄擁有者
sudo chown -R librenms:librenms /opt/librenms/
#變更目錄權限
sudo chmod 771 /opt/librenms 
sudo setfacl -d -m g::rwx /opt/librenms/rrd /opt/librenms/logs /opt/librenms/bootstrap/cache/ /opt/librenms/storage/
sudo setfacl -R -m g::rwx /opt/librenms/rrd /opt/librenms/logs /opt/librenms/bootstrap/cache/ /opt/librenms/storage/
#安裝php相依性套件
sudo su - librenms 
./scripts/composer_wrapper.php install --no-dev
#退出librenms
exit
#設定時區，於990行加入date.timezone = Asia/Taipei 
sudo vim /etc/php/8.3/fpm/php.ini
#設定時區，於990行加入.date.timezone = Asia/Taipei
sudo vim /etc/php/8.3/cli/php.ini 
#更改系統時區
sudo timedatectl set-timezone Asia/Taipei
#配置MariaDB
sudo vim /etc/mysql/mariadb.conf.d/50-server.cnf 
#在[mysql]底下新增以下內容
innodb_file_oer_table=1
lower_case_table_names=0
#重啟MariaDB
sudo systemctl enable mariadb 
sudo systemctl restart mariadb 
#連入MariaDB
sudo mysql -u root
#創建LibreNMS並配置密碼
CREATE DATABASE librenms CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'librenms'@'localhost' IDENTIFIED BY 'Znk37654128';
GRANT ALL PRIVILEGES ON librenms.* TO 'librenms'@'localhost';
exit
#配置PHP-FPM
#複製配置文件
sudo cp /etc/php/8.3/fpm/pool.d/www.conf /etc/php/8.3/fpm/pool.d/librenms.conf
#編輯配置文件
sudo vim /etc/php/8.3/fpm/pool.d/librenms.conf
#將[www]變更為[librenms]
#將user = www-data變更為user = librenms 
#將group = www-date變更為group = librenms 
#將listen = /run/php/php8.3-fpm.sock變更為listen = /run/php-fpm-librenms.sock
#配置Web Server
sudo vim /etc/nginx/conf.d/librenms.conf 
#加入以下內容
server {
 listen      80;
 server_name 192.168.21.145;
 root        /opt/librenms/html;
 index       index.php;

 charset utf-8;
 gzip on;
 gzip_types text/css application/javascript text/javascript application/x-javascript image/svg+xml text/plain text/xsd text/xsl text/xml image/x-icon;
 location / {
  try_files $uri $uri/ /index.php?$query_string;
 }
 location ~ [^/]\.php(/|$) {
  fastcgi_pass unix:/run/php-fpm-librenms.sock;
  fastcgi_split_path_info ^(.+\.php)(/.+)$;
  include fastcgi.conf;
 }
 location ~ /\.(?!well-known).* {
  deny all;
 }
}
#刪除預設網頁
sudo rm /etc/nginx/sites-enabled/default /etc/nginx/sites-available/default
#重新啟動nginx
sudo systemctl restart nginx
#重新啟動php-fpm
sudo systemctl restart php8.3-fpm
#啟用lnms命令補全
sudo ln -s /opt/librenms/lnms /usr/bin/lnms 
sudo cp /opt/librenms/misc/lnms-completion.bash /etc/bash_completion.d/
#配置snmpd 
未配置
#配置Crontab
sudo cp /opt/librenms/dist/librenms.cron /etc/cron.d/librenms
#啟用Crontab程序
sudo cp /opt/librenms/dist/librenms-scheduler.service /opt/librenms/dist/librenms-scheduler.timer /etc/systemd/system/
sudo systemctl enable librenms-scheduler.timer
sudo systemctl start librenms-scheduler.timer
#啟用日誌輪換
sudo cp /opt/librenms/misc/librenms.logrotate /etc/logrotate.d/librenms
#進入https://192.168.x.x並跟隨頁面步驟進行安裝
```