# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|
  config.vm.box = "bento/ubuntu-16.04"
  config.vm.network "forwarded_port", guest: 80, host: 8080
  #config.vm.network "public_network"

  #config.vm.provider "virtualbox" do |vb|
  #  config.vm.hostname = "zabbix.lab.local"
  #end
  
  config.vm.provision "shell", inline: <<-SHELL
    echo -e "\n--- Provisionamento iniciado ---\n";
    echo -e "\n--- Adicionando repositorio Zabbix ---\n";
    sudo apt-get install policycoreutils -y -f;
    setsebool -P httpd_can_network_connect on;
    setsebool -P httpd_can_connect_zabbix 1;
    setsebool -P zabbix_can_network 1;
    wget http://repo.zabbix.com/zabbix/3.4/ubuntu/pool/main/z/zabbix-release/zabbix-release_3.4-1+xenial_all.deb
    dpkg -i zabbix-release_3.4-1+xenial_all.deb;
    echo -e "\n--- Apt-get Update ---\n";
    apt-get update;
    apt-get install -y apache2;
    echo -e "\n--- Configurando o Zabbix ---\n";
    echo -e "--- Usuario Zabbix front-End : Admin---\n";
    echo -e "--- Senha Zabbix front-End : zabbix---\n";
    apt-get install zabbix-server-mysql zabbix-frontend-php zabbix-agent php-bcmath php-mbstring php-xml -y -f;
    sed 's@;date.timezone =@date.timezone=America/Sao_Paulo@' -i /etc/php/7.0/apache2/php.ini;
    echo -e "[mysqld]\ndefault-storage-engine = innodb" | sudo tee /etc/mysql/conf.d/mysqld.conf;
    service mysql restart;   
    echo -e "\n--- Configurando o Mysql e criando nosso banco de dados ---\n";
    echo -e "--- Mysql Database : zabbixdb---\n";
    echo -e "--- Usuario Zabbix mysql : zabbix---\n";
    echo -e "--- Senha Zabbix mysql : zabbix---\n";
    mysql -uroot -proot -e "create database zabbix character set utf8 collate utf8_bin;"
    mysql -uroot -proot -e "CREATE USER 'zabbix'@'%' IDENTIFIED BY 'zabbix';";    
    mysql -uroot -proot -e "grant all privileges on *.* to 'zabbix'@'%' identified by 'zabbix';";
    zcat /usr/share/doc/zabbix-server-mysql/create.sql.gz | mysql -uzabbix -p"zabbix" zabbix;
    sed 's/# DBPassword=/DBPassword=zabbix/g' /etc/zabbix/zabbix_server.conf -i;
    service zabbix-server restart;
    service zabbix-agent restart; 
    service apache2 restart;
    echo -e "\n--- Provisionamento finalizado ---\n";
  SHELL



  #config.vm.provision "shell", path: "build.sh", privileged: true

end
