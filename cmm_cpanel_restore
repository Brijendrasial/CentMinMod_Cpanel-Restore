#!/bin/bash

# Cpanel Backup Restore Script for CentMinMod Installer [CMM]

# Scripted by Brijendra Sial @ Bullten Web Hosting Solutions [https://www.bullten.com]

RED='\033[01;31m'
RESET='\033[0m'
GREEN='\033[01;32m'
YELLOW='\e[93m'
WHITE='\e[97m'
BLINK='\e[5m'

#set -e
set -x

echo " "
echo -e "$GREEN*******************************************************************************$RESET"
echo " "
echo -e $YELLOW"Cpanel Backup Restore Script for CentMinMod Installer [CMM]$RESET"
echo " "
echo -e $YELLOW"By Brijendra Sial @ Bullten Web Hosting Solutions [https://www.bullten.com]"$RESET
echo " "
echo -e $YELLOW"Web Hosting Company Specialized in Providing Managed VPS and Dedicated Server's"$RESET
echo " "
echo -e "$GREEN*******************************************************************************$RESET"

echo " "


ROOT_PASSWORD=$(cat /root/.my.cnf | grep password | cut -d' ' -f1 | cut -d'=' -f2)


function restore_cpanel_database
{
echo " "
echo -e $GREEN"Restoring All Databases"$RESET
echo " "

DATABASE_CREATE=$(ls -lht /home/${FILE_NAME}/mysql/ | awk '{print $9}' | sed -r '/^\s*$/d' | grep .create$)

                        for db in ${DATABASE_CREATE}; do
                        DBC=$(ls -lht /home/${FILE_NAME}/mysql/ | awk '{print $9}' | sed -r '/^\s*$/d' | grep .create$ | wc -l)
                                for ((x=1; x<=$DBC; x++)); do
                                        /usr/bin/mysql -u root --password=$ROOT_PASSWORD < /home/${FILE_NAME}/mysql/$db
                                        x=$((x + 1))
                                        echo -e $GREEN"Database Created $db"$RESET
                                        echo " "
                                done
                        done

DATABASE_RESTORE=$(ls -lht /home/${FILE_NAME}/mysql/ | awk '{print $9}' | sed -r '/^\s*$/d' | grep .sql$)

                        for db in ${DATABASE_RESTORE}; do
                        DBR=$(ls -lht /home/${FILE_NAME}/mysql/ | awk '{print $9}' | sed -r '/^\s*$/d' | grep .sql$ | wc -l)
                                for ((x=1; x<=$DBR; x++)); do
                                        /usr/bin/mysql -u root --password=$ROOT_PASSWORD ${db%.*}< /home/${FILE_NAME}/mysql/$db
                                        x=$((x + 1))
                                        echo -e $GREEN"Database Restored $db"$RESET
                                        echo " "
                                done
                        done

}

function restore_cpanel_files
{
echo " "
echo -e $GREEN"Restoring All Files"$RESET
echo " "

CREATE_DOMAINS=$(ls -lht /home/${FILE_NAME}/va/ | awk '{print $9}' | sed -r '/^\s*$/d')

                        for db in ${CREATE_DOMAINS}; do
                        CRD=$(ls -lht /home/${FILE_NAME}/va/ | awk '{print $9}' | sed -r '/^\s*$/d' | wc -l)
                               for ((x=1; x<=$CRD; x++)); do
                                        mkdir -p /home/nginx/domains/$db
                                        mkdir -p /home/nginx/domains/$db/backup
                                        mkdir -p /home/nginx/domains/$db/log
                                        mkdir -p /home/nginx/domains/$db/private
                                        mkdir -p /home/nginx/domains/$db/public
                                        chown -R nginx:nginx /home/nginx/domains/$db
                                       x=$((x + 1))
                                       echo -e $GREEN"Domain Created $db"$RESET
                                        echo " "
                                done
                        done


MAIN_DOMAIN=$(grep -ir "main_domain" /home/${FILE_NAME}/userdata/main | cut -d":" -f2 | tr -d " ")
cp /home/${FILE_NAME}/homedir/public_html/* /home/nginx/domains/$MAIN_DOMAIN/public/
chown -R nginx:nginx /home/nginx/domains/$MAIN_DOMAIN/*
chmod 2750 /home/nginx/domains/$MAIN_DOMAIN

cat > /usr/local/nginx/conf/conf.d/${MAIN_DOMAIN}.conf <<"EOF"
# Centmin Mod Getting Started Guide
# must read http://centminmod.com/getstarted.html

# redirect from non-www to www
# uncomment, save file and restart Nginx to enable
# if unsure use return 302 before using return 301
#server {
#            listen   80;
#            server_name demo.com;
#            return 301 $scheme://www.demo.com$request_uri;
#       }

server {
  server_name demo.com www.demo.com;

# ngx_pagespeed & ngx_pagespeed handler
#include /usr/local/nginx/conf/pagespeed.conf;
#include /usr/local/nginx/conf/pagespeedhandler.conf;
#include /usr/local/nginx/conf/pagespeedstatslog.conf;

  #add_header X-Frame-Options SAMEORIGIN;
  #add_header X-Xss-Protection "1; mode=block" always;
  #add_header X-Content-Type-Options "nosniff" always;

  # limit_conn limit_per_ip 16;
  # ssi  on;

  access_log /home/nginx/domains/demo.com/log/access.log combined buffer=256k flush=5m;
  error_log /home/nginx/domains/demo.com/log/error.log;

  root /home/nginx/domains/demo.com/public;

  location / {

# block common exploits, sql injections etc
#include /usr/local/nginx/conf/block.conf;

  # Enables directory listings when index file not found
  #autoindex  on;

  # Shows file listing times as local time
  #autoindex_localtime on;

  # Wordpress Permalinks example
  #try_files $uri $uri/ /index.php?q=$uri&$args;

  }

  include /usr/local/nginx/conf/staticfiles.conf;
  include /usr/local/nginx/conf/php.conf;
  #include /usr/local/nginx/conf/drop.conf;
  #include /usr/local/nginx/conf/errorpage.conf;
  include /usr/local/nginx/conf/vts_server.conf;
}
EOF
sed -i "s/demo.com/$MAIN_DOMAIN/g" /usr/local/nginx/conf/conf.d/${MAIN_DOMAIN}.conf
nprestart
}



case $1 in
        -c )
                        echo " "
                        echo -e $GREEN"Restoring Backup $2"$RESET
                        echo " "
                        read -n 1 -s -r -p "Press any key to continue"
                        echo " "
                        echo " "
                        echo -e $GREEN"Extracting Backup File $2"$RESET
                        echo " "
                        tar -zxvf $2 -C /home/
                        FILE_NAME=${2%.*.*}
                        sed '/localhost/!d' /home/${FILE_NAME}/mysql.sql >> /home/${FILE_NAME}/mysql_update.sql
                        mysql -u root --password=$ROOT_PASSWORD mysql < /home/${FILE_NAME}/mysql_update.sql
                        rm -rf /home/${FILE_NAME}/mysql_update.sql
                        restore_cpanel_database
                        restore_cpanel_files

        ;;
esac
