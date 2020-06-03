#!/bin/bash

# Ubuntu-16.04 or 18.04
# timedatectl set-timezone 'America/Sao_Paulo'
#PUBHOST=$(ec2metadata --public-hostname | cut -d : -f 2 | tr -d " ")

apt update && apt upgrade -y
# sudo dpkg --configure -a
# https://github.com/benadida/helios-server/blob/master/INSTALL.md

# install PostgreSQL 8.3+
# /usr/lib/postgresql/10/bin/pg_ctl
sudo apt install -y postgresql

sudo systemctl enable postgresql
sudo systemctl start postgresql

sudo -i -u postgres psql -c "CREATE ROLE helios LOGIN;"
sudo -i -u postgres psql -c "ALTER ROLE helios WITH SUPERUSER;" 
sudo -i -u postgres psql -c "CREATE DATABASE helios;"
#sudo -i -u postgres psql -c "GRANT ALL PRIVILEGES ON DATABASE helios TO helios;"
#sudo -i -u postgres psql -c "ALTER DATABASE helios OWNER TO helios;"

sudo systemctl restart postgresql

sudo apt-get install -y python-pip
sudo pip install virtualenv

sudo adduser --disabled-login --quiet --gecos Helios helios

sudo -i -u helios
git clone https://github.com/AdrianoRuseler/helios-server.git
cd /home/helios/helios-server 
virtualenv venv
source venv/bin/activate

pip install -r requirements.txt

sed -i "s/echo/#echo/g" reset.sh
echo '' >> reset.sh
echo $'echo "from helios_auth.models import User; User.objects.create(user_type=\'password\',user_id=\'admin@helios.vote\', info={\'name\':\'Helios Admin\',\'password\':\'heliosadm\'})" | python manage.py shell' >> reset.sh


./reset.sh

export URL_HOST="http://100.25.103.90:8000"

python manage.py runserver 0.0.0.0:8000

# apt-get install rabbitmq-server


