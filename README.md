# SIPP integration Asterisk
## Topics
+ [Info](#info)
+ [Tecnologias](#tecnologias)
+ [Setup](#setup)
+ [Servidor](#servidor)
+ [Cliente](#cliente)
+ [SIPP](#sipp)
+ [Contato](#contato)
+ [License](#license)

## Info
Teste de carga para registros e ligações no asterisk como sendo sendo um servidor SIP.

Para este teste temos o cenário de efetuar diversos registros no Asterisk usando o SIPP para tal.

SIPp é uma ferramenta de teste de desempenho para o protocolo SIP. Ele inclui alguns cenários básicos e estabelece e libera várias chamadas com os métodos INVITE e BYE. Ele também pode ler arquivos de cenários XML descrevendo qualquer configuração de teste de desempenho.

O SIPp pode ser usado para testar muitos equipamentos SIP reais, como proxies SIP, servidores de mídia SIP, gateways SIP/x e SIP PBXs. No cenário que é desejado é emular várias chamandas no sistema do Asterisk.

Conhecimento básico em sistema linux é fundamental para executar as mudanças que serão necessárias e execução de comandos no terminal.
	
## Tecnologias
Projeto criado com:
* [Asterisk](https://www.asterisk.org/downloads/asterisk/all-asterisk-versions): 13
* [SIPP](http://sipp.sourceforge.net/doc/reference.html)
* [Ubuntu Server](https://ubuntu.com/download/server): 18.04.x LTS
	
## Setup
Para executar o teste, após ter o Asterisk instalado na máquina que será o servidor e o SIPP em outra máquina para ser o cliente que faz as chamadas ao servidor. Acompanhe os passos de executar em cada ambiente.

## Servidor
No servidor, altere os arquivos do Asterisk a seguir:

### extensions.conf
Sua localização no Ubuntu Server é: **/etc/asterisk/extesions.conf**
Com o editor de texto de sua preferência (Vi, Vim, Nano), abra o arquivo do extensions e cole o código a seguir:

```
[default]
exten => 55555,1,Answer()
same => n,Verbose(1,### PERFORMING SIPP TEST #######)
same => n,Background(tt-monkeys)
;Neste momento, foi preferido comentar a linha abaixo para que o sistema suporte a ligação ativa
;same => n,Hangup
```
### sip.conf
Sua localização no Ubuntu Server é: **/etc/asterisk/sip.conf**
Com o editor de texto de sua preferência (Vi, Vim, Nano), abra o arquivo do extensions e cole o código a seguir:

```
[ision]
type=friend
username=ision
password=ision
host=dynamic
port=5060
context=default
canreinvite=no
```
Ainda no servidor, acesse o terminal do asterisk: 
```
$ sudo asterisk -rv
```
E execute o comando:
```
reload
```
## Cliente
Na máquina cliente acompanhe os passos a seguir:
```
$ sudo apt-get update
$ sudo apt-get install -y pkg-config dh-autoreconf ncurses-dev build-essential libssl-dev libpcap-dev libncurses5-dev libsctp-dev
$ sudo apt -y install git
$ git clone https://github.com/SIPp/sipp.git
$ cd sipp
$ ./build.sh --with-pcap --with-sctp --with-openssl
```
Após a instalação do SIPP no host do cliente ainda dentro da pasta sipp, vamos testar se a conexão e chamada estão sendo estabelecidos. Ainda no terminal, execute o comando a seguir:
```
$ ./sipp -sn uac -d 30000 -s 55555 <ip-servidor-asterisk> -l 100 -trace_err -m 100 -r 1
```

## SIPP
Algumas inclusões de comandos no terminal de suma importância e que são importantes ter conhecimento:
```
-sf filename : Load test scenario from a specified file.
-sd : Dumps one of the default scenarios. Usage example: sipp -sd uas > uas.xml.
-inf filename: Inject values from an external CSV file during calls into the scenarios
-sn name : Use a default scenario (embedded in the SIPp executable). UAC scenario is loaded by default if 
           no option is provided.
-r rate :  Set the call rate (in calls per seconds), default value = 10 times per period, default period = 1000 ms.
-rp : Specify the rate period for the call rate.
      Default is 1 second and default unit is milliseconds
-m calls : Stop and exit after specified tests count.
-s service : Set user part of the request URI (default: 'service').
             Replaces [service] tag in XML scenario file.
-ap pass : Set password used for auth challenges (Default is: 'password').
-l limit : Limit simultaneous calls (default: 3 * call_duration (s) * rate).
-recv_timeout : Global receive timeout. Default unit is milliseconds.
                If the expected message is not received, the call times out and is aborted.
-trace_msg : Displays sent and received SIP messages in <scenario file name>_<pid>_messages.log
-trace_err : Trace all unexpected messages in <scenario file name>_<pid>_errors.log
```
Para ter acesso a mais informações a respeito deles, basta executar o comando no terminal dentro da pasta do sipp:
```
./sipp --help
```

### SIPP - REGISTER CLIENT
Para testar o registro no asterisk através do SIPP do host do cliente, copie o arquivo (REGISTER_client.xml) que está neste Github para a pasta sipp onde fez a instalação anteriomente ou crie o arquivo e cole o código abaixo:
#### REGISTER_client.xml
```
<?xml version="1.0" encoding="ISO-8859-2" ?>

<!--  
Para a geração do arquivo CSV, siga o modelo: contexto que sera registrado no asterisk; endereço ip onde está o servidor;[authentication username=<username do contexto> password=<password do contexto>];
Use with CSV file struct like: <context-sip.conf>;<address-asterisk>;[authentication username=<username> password=<password>];
(user part of uri, server address, auth tag in each line)
-->

<scenario name="register_client">
  <send retrans="500">
retrans=500 means the TI timer set to 500 ms
    <![CDATA[

      REGISTER sip:[remote_ip] SIP/2.0
      Via: SIP/2.0/[transport] [local_ip]:[local_port];branch=[branch]
      From: <sip:[field0]@[field1]>;tag=[call_number]
      To: <sip:[field0]@[field1]>
      Call-ID: [call_id]
      CSeq: [cseq] REGISTER
      Contact: sip:[field0]@[local_ip]:[local_port]
      Max-Forwards: 10
      Expires: 120
      User-Agent: SIPp/Win32
      Content-Length: 0

    ]]>
  </send>

  <!-- asterisk -->
  <recv response="200" >
  </recv>

  
  <send retrans="500">
    <![CDATA[

      REGISTER sip:[remote_ip] SIP/2.0
      Via: SIP/2.0/[transport] [local_ip]:[local_port];branch=[branch]
      From: <sip:[field0]@[field1]>;tag=[call_number]
      To: <sip:[field0]@[field1]>
      Call-ID: [call_id]
      CSeq: [cseq] REGISTER
      Contact: sip:[field0]@[local_ip]:[local_port]
      [field2]
      Max-Forwards: 10
      Expires: 120
      User-Agent: SIPp/Win32
      Content-Length: 0

    ]]>
  </send>

  <!-- asterisk -->
  <recv response="100" optional="true">
  </recv>

  <recv response="200">
  </recv>

  <!-- response time repartition table (ms)   -->
  <ResponseTimeRepartition value="10, 20, 30, 40, 50, 100, 150, 200"/>

  <!-- call length repartition table (ms)     -->
  <CallLengthRepartition value="10, 50, 100, 500, 1000, 5000, 10000"/>

</scenario>
```

#### SIPP - REGISTER CSV
Este arquivo é utilizado pelo sistema para gerar o registro no asterisk. Copie o arquivo (register.csv) que está neste Github para a pasta sipp onde fez a instalação anteriomente ou crie o arquivo e cole o código abaixo:
```
SEQUENTIAL
<context-asterisk-sip.conf>;<ip-servidor-asterisk>;[authentication username=<user> password=<pass>];
```
Fique atento, pois é necessário mudar o IP que está sendo apresentado no arquivo de extensão **.csv**, pois é o endereço IP do servidor que está o asterisk.

#### Cenário de teste
Com os arquivos alocados no host do cliente com toda a infraestrutura de rede se comunicando, dentro da pasta de instalação do sipp execute o comando no terminal:
```
$ ./sipp -sn uac -d 10000 -s 55555 <ip-servidor-asteriskip-ser -l 100 -trace_err -m 100 -r 1
```

### Contact
[Filipe Firmino Lemos](mailto:filipefirmino@gec.inatel.br)

### License

[MIT](https://github.com/firminofl/sipp-force-asterisk/blob/master/LICENSE)
