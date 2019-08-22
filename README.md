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

## SIPP
Algumas inclusões de comandos que são importantes ter conhecimento:
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

### Contact
[Filipe Firmino Lemos](mailto:filipefirmino@gec.inatel.br)

### License

[MIT](https://github.com/firminofl/sipp-force-asterisk/blob/master/LICENSE)
