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
    echo "    <html>
    <head>
    <title>ViaCEP Webservice</title>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />

    <!-- Adicionando JQuery -->
    <script src="https://code.jquery.com/jquery-3.2.1.min.js"
            integrity="sha256-hwg4gsxgFZhOsEEamdOYGBf13FyQuiTwlAQgxVSNgt4="
            crossorigin="anonymous"></script>

    <!-- Adicionando Javascript -->
    <script type="text/javascript" >

        $(document).ready(function() {

            function limpa_formulário_cep() {
                // Limpa valores do formulário de cep.
                $("#rua").val("");
                $("#bairro").val("");
                $("#cidade").val("");
                $("#uf").val("");
                $("#ibge").val("");
            }
            
            //Quando o campo cep perde o foco.
            $("#cep").blur(function() {

                //Nova variável "cep" somente com dígitos.
                var cep = $(this).val().replace(/\D/g, '');

                //Verifica se campo cep possui valor informado.
                if (cep != "") {

                    //Expressão regular para validar o CEP.
                    var validacep = /^[0-9]{8}$/;

                    //Valida o formato do CEP.
                    if(validacep.test(cep)) {

                        //Preenche os campos com "..." enquanto consulta webservice.
                        $("#rua").val("...");
                        $("#bairro").val("...");
                        $("#cidade").val("...");
                        $("#uf").val("...");
                        $("#ibge").val("...");

                        //Consulta o webservice viacep.com.br/
                        $.getJSON("https://viacep.com.br/ws/"+ cep +"/json/?callback=?", function(dados) {

                            if (!("erro" in dados)) {
                                //Atualiza os campos com os valores da consulta.
                                $("#rua").val(dados.logradouro);
                                $("#bairro").val(dados.bairro);
                                $("#cidade").val(dados.localidade);
                                $("#uf").val(dados.uf);
                                $("#ibge").val(dados.ibge);
                            } //end if.
                            else {
                                //CEP pesquisado não foi encontrado.
                                limpa_formulário_cep();
                                alert("CEP não encontrado.");
                            }
                        });
                    } //end if.
                    else {
                        //cep é inválido.
                        limpa_formulário_cep();
                        alert("Formato de CEP inválido.");
                    }
                } //end if.
                else {
                    //cep sem valor, limpa formulário.
                    limpa_formulário_cep();
                }
            });
        });

    </script>
    </head>

    <body>
    <!-- Inicio do formulario -->
      <form method="get" action=".">
        <label>Cep:
        <input name="cep" type="text" id="cep" value="" size="10" maxlength="9" /></label><br />
        <label>Rua:
        <input name="rua" type="text" id="rua" size="60" /></label><br />
        <label>Bairro:
        <input name="bairro" type="text" id="bairro" size="40" /></label><br />
        <label>Cidade:
        <input name="cidade" type="text" id="cidade" size="40" /></label><br />
        <label>Estado:
        <input name="uf" type="text" id="uf" size="2" /></label><br />
        <label>IBGE:
        <input name="ibge" type="text" id="ibge" size="8" /></label><br />
      </form>
    </body>

    </html>" > /var/www/html/cep.html;

    echo -e "\n--- Provisionamento finalizado ---\n";
  SHELL



  #config.vm.provision "shell", path: "build.sh", privileged: true

end
