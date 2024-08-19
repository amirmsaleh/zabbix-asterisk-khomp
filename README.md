# Zabbix-Asterisk-Khomp
Templates Zabbix para monitoramento SNMP do Asterisk e Khomp EBS
Mantenedor: Igor Silva
Estado: Finalizado
Data: 20240819

(c) Copyright 2024, Ultrix

# Introdução
Para fazer o monitoramento via SNMP na sua máquina é nescessário:
  
- Instalação do SNMP  
   - Configuração do SNMP  
- Configuração do Zabbix  
   - Hosts
   - Templates
    
## Instalação do SNMP

O processo de instalação do SNMP e suas ferramentas no seu sistema varia de acordo com o sistema operacional que você está usando.
SUSE/OpenSUSE: sudo zypper install net-snmp net-snmp-utils
Para iniciar o processo:sudo systemctl start snmpd ,  sudo systemctl enable snmpd
Esses comandos instalarão e configurarão o SNMP para que o serviço inicie automaticamente durante a inicialização do sistema.

**Configuração**

Para realizar o processo de configuração é preciso entrar no arquivo correspondente: cd /etc/snmp
Dentro deste diretório: vim snmpd.conf

snmpd.conf
"com2sec readonly  172.28.17.7 utx_snmp_1384611681

group MyROGroup v1         readonly
group MyROGroup v2c        readonly

view    all           included   .1

       "#"group     context   sec.model sec.level prefix read  write notif
access MyROGroup ""        any       noauth    exact  all   none  none

proxy -v 2c -c khomp localhost:14161 .1.3.6.1.4.1.32624

syslocation Ultrix
syscontact suporte@ultrix.srv.br

dontLogTCPWrappersConnects yes

master agentx
agentXPerms 0660 0550 nobody nobody"

Após isso dê o comando: systemctl restart snmpd.service para reiniciar o serviço.


Nesta configuração SNMP, criamos um grupo somente leitura que se conecta a um dispositivo exclusivo que possui um endereço IP 172.28.17.7. A chave utx_snmp_1384611681 é usada para proteger a conexão com este dispositivo. Os membros da comunidade só podem monitorar o que está acontecendo e não alterar as configurações.

A seguir, definimos um grupo de acesso chamado MyROGroup que usa as versões do protocolo SNMP v1 e v2c. Este grupo traz ferramentas para o grupo de leitura, permitindo-lhes acessar informações mas ler.

Para melhorar a acessibilidade, definimos todas as visualizações, que permitem ver mais informações dentro do dispositivo. A configuração também inclui regras de acesso, onde sabemos que o grupo MyROGroup pode ver tudo, mas não pode modificar ou receber notificações.

Outro fator nesta configuração é a configuração do agente SNMP. O comando proxy redireciona as solicitações SNMP recebidas da porta 14161 para a porta SNMP (ou seja, 161). No exemplo, o agente foi alterado para SNMP versão 2c, utilizando a comunidade khomp, e direcionou as solicitações para localhost na porta 14161 com identificador de informações de acesso .1.3.6.1.4.1.32624. Isso permite que o servidor SNMP local atue como intermediário, enviando consultas SNMP recebidas em portas que estão conectadas às portas SNMP padrão, coletando informações do dispositivo de destino e retornando as informações.

Além disso, esta configuração contém informações administrativas como o local onde o dispositivo está definido como “Ultrix” e o e-mail de suporte (supporte@ultrix.srv.br) se necessário.

No final, foram corrigidas opções avançadas, como optar por não registrar algumas conexões (dontLogTCPWrappersConnects sim) e configurar SNMP para o agente mestre (agente mestre x). Isso faz com que o sistema SNMP seja confiável e capaz de gerenciar vários dispositivos de maneira sistemática e segura.

### Configuração Zabbix

**Hosts**

Passo 1: Acessar a Interface do Zabbix
Abra um navegador web e acesse a interface web do Zabbix.
Faça login com suas credenciais de administrador.

Passo 2: Adicionar um Novo Host
Vá para a seção "Configuration":

No menu superior, clique em "Configuration" e depois em "Hosts".
Clique em "Create host":

Na parte superior direita da página "Hosts", clique no botão "Create host".
Passo 3: Configurar as Informações Básicas do Host
Preencha os Detalhes Básicos:

Host name: Insira o nome do host.
Visible name (opcional): Um nome mais amigável para exibição.
Selecione o Grupo:

Na seção "Groups", escolha um grupo de hosts existente ou crie um novo grupo clicando em "Select" e depois em "Create new group".
Adicione um Interface SNMP:

Na seção "Interfaces", clique em "Add".
Type: Selecione "SNMP".
IP address: Insira o endereço IP do dispositivo que você está monitorando.
Port: Insira "161" (porta padrão para SNMP).
DNS name (opcional): Insira um nome DNS se aplicável.
Connect to: Selecione "Agent" se não estiver visível na lista.
Defina a Comunidade SNMP

Na seção "SNMP Interfaces", você não configurará diretamente a comunidade aqui. Em vez disso, você configurará o Item mais tarde com a comunidade apropriada.

**Templates**

Atribuir um Template SNMP:

Na seção "Templates", clique em "Link new templates".
Na caixa de pesquisa, procure por "Template SNMP" ou "Template App SNMP". Dependendo da sua configuração, você pode ter diferentes templates SNMP pré-configurados.
Selecione um template apropriado (por exemplo, Template App SNMP Generic) e clique em "Add".
Adicionar o Template:

Clique em "Add" para adicionar o template ao seu host.

                       
                        




