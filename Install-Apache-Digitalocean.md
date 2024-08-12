## [Passo 1 — Instale o Java](https://www.digitalocean.com/community/tutorials/how-to-install-apache-tomcat-9-on-debian-10-pt#passo-1-instale-o-java)

O Tomcat exige que o Java seja instalado no servidor para que qualquer código de aplicação web Java possa ser executado. Podemos satisfazer esse requisito instalando o OpenJDK com o apt.

Primeiro, atualize seu índice de pacotes apt:

```bash
sudo apt update
```

Copy

Em seguida, instale o pacote Java Development Kit com o apt:

```bash
sudo apt install default-jdk
```

Copy

Agora que o Java está instalado, podemos criar um usuário `tomcat`, que será usado para executar o serviço Tomcat.

## [Passo 2 — Crie um Usuário Tomcat](https://www.digitalocean.com/community/tutorials/how-to-install-apache-tomcat-9-on-debian-10-pt#passo-2-crie-um-usuario-tomcat)

Por motivos de segurança, o Tomcat deve ser executado como um usuário sem privilégios (ou seja, não **root**). Criaremos um novo usuário e grupo que executará o serviço Tomcat.

Primeiro, crie um novo grupo **tomcat**:

```bash
sudo groupadd tomcat
```

Copy

Em seguida, crie um novo usuário **tomcat**. Tornaremos esse usuário um membro do grupo **tomcat**, com um diretório home `/opt/tomcat` (onde instalaremos o Tomcat) e com um shell `/bin/false` (para que ninguém possa fazer login na conta):

```bash
sudo useradd -s /bin/false -g tomcat -d /opt/tomcat tomcat
```

Copy

Agora que nosso usuário **tomcat** está configurado, faça o download e instale o Tomcat.

## [Passo 3 — Instale o Tomcat](https://www.digitalocean.com/community/tutorials/how-to-install-apache-tomcat-9-on-debian-10-pt#passo-3-instale-o-tomcat)

A melhor maneira de instalar o Tomcat 9 é baixar a versão binária mais recente e configurá-la manualmente.

Encontre a versão mais recente do Tomcat 9 na [página de downloads do Tomcat 9](https://tomcat.apache.org/download-90.cgi). No momento da redação deste documento, a versão mais recente é a **9.0.27**, mas você deve usar a última versão estável, se estiver disponível. Na seção **Binary Distributions** e, em seguida, na lista **Core**, copie o link que aponta para o “tar.gz”.

Em seguida, retorne à sua sessão SSH e vá para o diretório `/tmp` no seu servidor. Este é um bom diretório para baixar itens efêmeros, como o tarball do Tomcat, que não precisaremos depois de extrair o conteúdo do Tomcat:

```bash
cd /tmp
```

Copy

Usaremos a ferramenta de linha de comando `curl` para baixar o tarball. Instale o `curl`:

```bash
sudo apt install curl
```

Copy

Agora, use o `curl` para baixar o link que você copiou do site do Tomcat:

```bash
curl -O http://www-eu.apache.org/dist/tomcat/tomcat-9/v9.0.11/bin/apache-tomcat-9.0.11.tar.gz
```

Copy

Vamos instalar o Tomcat no diretório `/opt/tomcat`. Crie o diretório e extraia o arquivo tar com estes comandos:

```bash
sudo mkdir /opt/tomcat
sudo tar xzvf apache-tomcat-9*tar.gz -C /opt/tomcat --strip-components=1
```

Copy

Em seguida, configuraremos as permissões de usuário adequadas para nossa instalação.

## [Passo 4 — Atualize as Permissões](https://www.digitalocean.com/community/tutorials/how-to-install-apache-tomcat-9-on-debian-10-pt#passo-4-atualize-as-permissoes)

O usuário **tomcat** que criamos precisa ter acesso à instalação do Tomcat. Vamos configurar isso agora.

Mude para o diretório em que desempacotamos a instalação do Tomcat:

```bash
cd /opt/tomcat
```

Copy

Conceda ao grupo **tomcat** a propriedade de todo o diretório de instalação:

```bash
sudo chgrp -R tomcat /opt/tomcat
```

Copy

A seguir, dê ao grupo **tomcat** acesso de leitura ao diretório `conf` e a todo o seu conteúdo e acesso de execução ou `execute` ao diretório em si:

```bash
sudo chmod -R g+r conf
sudo chmod g+x conf
```

Copy

Faça do usuário **tomcat** o proprietário dos diretórios `webapps`, `work`, `temp` e `logs`:

```bash
sudo chown -R tomcat webapps/ work/ temp/ logs/
```

Copy

Agora que as permissões apropriadas foram configuradas, criaremos um arquivo de serviço systemd para gerenciar o processo do Tomcat.

## [Passo 5 — Crie um Arquivo de Serviço systemd](https://www.digitalocean.com/community/tutorials/how-to-install-apache-tomcat-9-on-debian-10-pt#passo-5-crie-um-arquivo-de-servico-systemd)

Queremos poder executar o Tomcat como um serviço, portanto, configuraremos o arquivo de serviço systemd.

O Tomcat precisa saber onde o Java está instalado. Esse caminho é geralmente chamado de `JAVA_HOME`. A maneira mais fácil de procurar esse local é executando este comando:

```bash
sudo update-java-alternatives -l
```

Copy

```
Outputjava-1.11.0-openjdk-amd64      1111       /usr/lib/jvm/java-1.11.0-openjdk-amd64
```

Seu `JAVA_HOME` é a saída da última coluna (destacada acima). Dado o exemplo acima, o `JAVA_HOME` correto para este servidor seria:

```
JAVA_HOME/usr/lib/jvm/java-1.11.0-openjdk-amd64
```

Seu `JAVA_HOME` pode ser diferente.

Com essa informação, podemos criar o arquivo de serviço systemd. Abra um arquivo chamado `tomcat.service` no diretório `/etc/systemd/system` digitando:

```bash
sudo nano /etc/systemd/system/tomcat.service
```

Copy

Cole o seguinte conteúdo no seu arquivo de serviço. Modifique o valor de `JAVA_HOME` se necessário para corresponder ao valor encontrado em seu sistema. Você também pode querer modificar as configurações de alocação de memória especificadas em `CATALINA_OPTS`:

/etc/systemd/system/tomcat.service

```
[Unit]
Description=Apache Tomcat Web Application Container
After=network.target

[Service]
Type=forking

Environment=JAVA_HOME=/usr/lib/jvm/java-1.11.0-openjdk-amd64
Environment=CATALINA_PID=/opt/tomcat/temp/tomcat.pid
Environment=CATALINA_HOME=/opt/tomcat
Environment=CATALINA_BASE=/opt/tomcat
Environment='CATALINA_OPTS=-Xms512M -Xmx1024M -server -XX:+UseParallelGC'
Environment='JAVA_OPTS=-Djava.awt.headless=true -Djava.security.egd=file:/dev/./urandom'

ExecStart=/opt/tomcat/bin/startup.sh
ExecStop=/opt/tomcat/bin/shutdown.sh

User=tomcat
Group=tomcat
UMask=0007
RestartSec=10
Restart=always

[Install]
WantedBy=multi-user.target
```

Quando terminar, salve e feche o arquivo.

Em seguida, recarregue o daemon systemd para que ele conheça nosso arquivo de serviço:

```bash
sudo systemctl daemon-reload
```

Copy

Inicie o serviço Tomcat digitando:

```bash
sudo systemctl start tomcat
```

Copy

Verifique se ele foi iniciado sem erros digitando:

```bash
sudo systemctl status tomcat
```

Copy

Você deve ver uma saída semelhante à seguinte:

```
Output● tomcat.service - Apache Tomcat Web Application Container
   Loaded: loaded (/etc/systemd/system/tomcat.service; disabled; vendor preset: enabled)
   Active: active (running) since Thu 2019-10-24 17:18:11 UTC; 4s ago
  Process: 5962 ExecStart=/opt/tomcat/bin/startup.sh (code=exited, status=0/SUCCESS)
 Main PID: 5970 (java)
    Tasks: 44 (limit: 2377)
   Memory: 184.2M
   CGroup: /system.slice/tomcat.service
           └─5970 /usr/lib/jvm/java-1.11.0-openjdk-amd64/bin/java -Djava.util.logging.config.file=/opt/tomcat/conf/logging.properties -Djava.u

Oct 24 17:18:10 tomcat systemd[1]: Starting Apache Tomcat Web Application Container...
Oct 24 17:18:11 tomcat startup.sh[5962]: Tomcat started.
Oct 24 17:18:11 tomcat systemd[1]: Started Apache Tomcat Web Application Container.
```

Isso confirma que o Tomcat está funcionando no seu servidor.

## [Passo 6 — Ajuste o Firewall e Teste o Servidor Tomcat](https://www.digitalocean.com/community/tutorials/how-to-install-apache-tomcat-9-on-debian-10-pt#passo-6-ajuste-o-firewall-e-teste-o-servidor-tomcat)

Agora que o serviço Tomcat foi iniciado, podemos testar para garantir que a página padrão esteja disponível.

Antes de fazermos isso, precisamos ajustar o firewall para permitir que nossas solicitações cheguem ao serviço. Se você seguiu os pré-requisitos, terá um firewall `ufw` ativado no momento.

O Tomcat usa a porta `8080` para aceitar solicitações. Permita tráfego para essa porta digitando:

```bash
sudo ufw allow 8080
```

Copy

Com o firewall modificado, você pode acessar a página inicial padrão acessando seu domínio ou endereço IP, seguido de `:8080` em um navegador web:

```
Open in web browserhttp://domínio_do_servidor_ou_IP:8080
```

Você verá a página inicial padrão do Tomcat, além de outras informações. No entanto, se você clicar nos links do App Manager, por exemplo, seu acesso será negado. Podemos configurar esse acesso a seguir.

Se você conseguiu acessar o Tomcat com êxito, agora é uma boa hora para ativar o arquivo de serviço, para que o Tomcat seja iniciado automaticamente na inicialização:

```bash
sudo systemctl enable tomcat
```

Copy

## [Passo 7 — Configure a Interface Web de gerenciamento do Tomcat](https://www.digitalocean.com/community/tutorials/how-to-install-apache-tomcat-9-on-debian-10-pt#passo-7-configure-a-interface-web-de-gerenciamento-do-tomcat)

Para usar a aplicação web de gerenciamento que acompanha o Tomcat, precisamos adicionar um login ao nosso servidor Tomcat. Faremos isso editando o arquivo `tomcat-users.xml`:

```bash
sudo nano /opt/tomcat/conf/tomcat-users.xml
```

Copy

Você vai querer adicionar um usuário que possa acessar a `manager-gui` e a `admin-gui` (aplicações web que acompanham o Tomcat). Você pode fazer isso definindo um usuário, semelhante ao exemplo abaixo, entre as tags `tomcat-users`. Certifique-se de alterar o nome de usuário e a senha para algo seguro:

tomcat-users.xml

```
<tomcat-users>
. . .
    <user username="admin" password="senha" roles="manager-gui,admin-gui"/>
</tomcat-users>
```

Salve e feche o arquivo quando terminar.

Por padrão, as versões mais recentes do Tomcat restringem o acesso às aplicações Manager e Host Manager a conexões provenientes do próprio servidor. Como estamos instalando em uma máquina remota, você provavelmente desejará remover ou alterar essa restrição. Para alterar as restrições de endereço IP, abra os arquivos `context.xml` apropriados.

Para a aplicação Manager, digite:

```bash
sudo nano /opt/tomcat/webapps/manager/META-INF/context.xml
```

Copy

Para a aplicação Host Manager, digite:

```bash
sudo nano /opt/tomcat/webapps/host-manager/META-INF/context.xml
```

Copy

Dentro dele, comente a restrição de endereço IP para permitir conexões de qualquer lugar. Como alternativa, se você deseja permitir acesso apenas a conexões provenientes de seu próprio endereço IP, você pode adicionar seu endereço IP público à lista:

context.xml files for Tomcat webapps

```
<Context antiResourceLocking="false" privileged="true" >
  <!--<Valve className="org.apache.catalina.valves.RemoteAddrValve"
         allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1" />-->
</Context>
```

Salve e feche os arquivos quando terminar.

Para efetivar nossas alterações, reinicie o serviço Tomcat:

```bash
sudo systemctl restart tomcat
```

Copy

## [Passo 8 — Acesse a Interface Web](https://www.digitalocean.com/community/tutorials/how-to-install-apache-tomcat-9-on-debian-10-pt#passo-8-acesse-a-interface-web)

Agora que criamos um usuário, podemos acessar a interface de gerenciamento web novamente em um navegador. Mais uma vez, você pode acessar a interface correta digitando o nome de domínio ou endereço IP do servidor seguido da porta 8080 no seu navegador:

```
Open in web browserhttp://domínio_do_servidor_ou_IP:8080
```

A página que você vê deve ser a mesma que você recebeu quando testou anteriormente:

![Tomcat root](https://assets.digitalocean.com/articles/tomcat9/home-screen.png)

Vamos dar uma olhada no App Manager, acessível através do link ou `http://nome_de_dominio_do_servidor_ou_IP:8080/manager/html`. Você precisará inserir as credenciais da conta que adicionou ao arquivo `tomcat-users.xml`. Depois, você deverá ver uma página parecida com esta:

![Tomcat Web Application Manager](https://assets.digitalocean.com/articles/tomcat9/manager.png)

O Web Application Manager é usado para gerenciar suas aplicações Java. Você pode Iniciar, Parar, Recarregar, *Deployar* e Remover o deploy aqui. Você também pode executar alguns diagnósticos em suas aplicações (ou seja, encontrar vazamentos de memória). Por fim, as informações sobre o seu servidor estão disponíveis na parte inferior desta página.

Agora, vamos dar uma olhada no Host Manager, acessível através do link ou `http://nome_de_dominio_do_servidor_ou_IP:8080/host-manager/html/`:

![Tomcat Virtual Host Manager](https://assets.digitalocean.com/articles/tomcat9/host-manager.png)

N