dsoa-runtime
============

1. Pr�-Requisitos:
A vers�o atual da plataforma est� preparada para rodar em um ambiente distribu�do com base na implementa��o de refer�ncia do DOSGi, a qual utiliza
o Zookeeper como registro de servi�os distribu�do. Neste contexto, para que a plataforma funcione corretamente, o Zookeeper deve ser instalado e 
executado em uma m�quina da rede que � indicada no arquivo load/org.apache.cxf.dosgi.discovery.zookeeper.cfg carregado quando da inicializa��o da 
plataforma. A implementa��o de DOSGi � respons�vel por obter a descri��o dos servi�os locais que devem ser expostos remotamente e public�-las no registro
distribu�do que pode ser consultado pelos clientes para obter um proxy para o referido servi�o.

Na implementa��o de DOSGi, para que um servi�o publicado localmente no registro do OSGi possa ser exposto como um servi�o remoto, 
a seguintes propriedades devem ser setadas:
	1. "service.exported.interfaces": Indica quais interfaces do servi�o devem ser expostas para acesso remoto. Um "*" indica
	que todas as interfaces implementadas pelo servi�o devem estar remotamente acess�veis.
	2. "service.exported.configs": Indica como o servi�o deve ser exposto. O valor "org.apache.cxf.ws" indica que o servi�o deve ser exposto como um Web
	Service.			
	3. "org.apache.cxf.ws.address": Indica em que endere�o o servi�o ser� disponibilizado

A publica��o da descri��o de um servi�o com as propriedades indicadas acima em um registro distribu�do requer que o zookeeper (baixado separadamente) esteja em execu��o na rede. 
Para iniciar o zookeeper, utiliza-se o script zkServer.cmd dispon�vel no diret�rio_de_instala��o_do_zookeeper/bin. A publica��o de um servi�o no zookeeper corresponde � 
escrita de um descritor do servidor na pasta /osgi/service_registry. Nela, o dosgi/zookeeper organiza os servi�os atrav�s dos nomes completos
das interfaces expostas. Assim, um servi�o que exp�e a interface br.com.bb.provider.InformationProvider estar� publicado na pasta br/com/bb/provider/InformationProvider.
Em suma, para que um servi�o OSGi possa ser exposto remotamente, ele dever� ser publicado localmente com as propriedades indicadas acima setadas. Neste contexto, a implementa��o
do DOSGi, que � notificada da publica��o local do servi�o, l� as suas propriedades e monta um descritor de servi�o em formato XML. Este descritor � escrito no Zookeeper server e 
fica dispon�vel para a consulta pelos clientes em execu��o nas inst�ncias da plataforma em execu��o na rede.
 
O zookeeper possui uma ferramenta cliente pr�pria que pode ser utilizada por um administrador para consultar os servi�os que est�o publicados no registro. Este cliente,
zkCli, � uma ferramenta de linha de comandos que permite a listagem dos servi�os publicados. Utilizando esta ferramenta podemos executar um comando tal como:

$> ls /osgi/service_registry/br/com/bb/provider/InformationProvider
[localhost#9797##ProviderBBC]

O resultado do comando, indicado entre colchetes, mostra que h� um arquivo chamado localhost#9797##ProviderBBC que descreve o servi�o exposto no endere�o
correspondente. Para obtermos o descritor do servi�o armazenado no Zookeeper, podemos utilizar o comando get, como indicado abaixo:

$> get /osgi/service_registry/br/com/bb/provider/InformationProvider/localhost#9797##ProviderBBC
<?xml version="1.0" encoding="UTF-8"?>
<endpoint-descriptions xmlns="http://www.osgi.org/xmlns/rsa/v1.0.0">
  <endpoint-description>
    <property name="constraint.operation.qos.AvgResponseTime.getCotation.LE" value-type="Double" value="300.0" />
    <property name="endpoint.framework.uuid" value="59db6b34-9327-44d1-b46f-a28dcb8c2be9" />
    <property name="endpoint.id" value="http://localhost:9797/ProviderBBC" />
    <property name="endpoint.package.version.br.com.bb.provider" value="0.0.0" />
    <property name="endpoint.service.id" value-type="Long" value="113" />
    <property name="objectClass">
      <array>
        <value>br.com.bb.provider.InformationProvider</value>
      </array>
    </property>
    <property name="org.apache.cxf.ws.address" value="http://localhost:9797/ProviderBBC" />
    <property name="provider.pid" value="Provider-BBC" />
    <property name="service.imported" value="true" />
    <property name="service.imported.configs">
      <array>
        <value>org.apache.cxf.ws</value>
      </array>
    </property>
    <property name="service.intents">
      <array>
        <value>HTTP</value>
        <value>SOAP.1_1</value>
        <value>SOAP</value>
      </array>
    </property>
    <property name="service.managed" value-type="Boolean" value="true" />
  </endpoint-description>
</endpoint-descriptions>


cZxid = 903
ctime = Sat Sep 10 11:34:16 GMT-03:00 2016
mZxid = 903
mtime = Sat Sep 10 11:34:16 GMT-03:00 2016
pZxid = 903
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 96568440109006849
dataLength = 1381
numChildren = 0

Veja que o comando get recupera o arquivo xml utilizado como descritor de servi�o, o qual indica, dentre outras coisas o endere�o do servi�o e que o mesmo
est� exposto atrav�s de Web Services. Al�m destas informa��es, o arquivo apresenta um conjunto de propriedades utilizadas para descrever a qualidade
do servi�o exposto. Ex:
<property name="constraint.operation.qos.AvgResponseTime.getCotation.LE" value-type="Double" value="300.0" />

A propriedade descrita acima � interpretada pela plataforma DSOA como parte de uma Quality Offer que indica que o servi�o exposto ter� um valor m�dio de tempo de resposta
de 300 milissegundos (unidade default de tempo).
	
Para prover um servi�o com atributos de qualidade tais como tempo m�dio de resposta e disponibilidade, a plataforma DSOA introduz um simulador de qualidade. Para o simulador,
um servi�o a ser exposto deve ser descrito atrav�s de um arquivo xml indicando de que forma o servi�o deve ser simulado. Um descritor exemplo � indicado abaixo:

<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<services>
	<service interfaceName="br.com.bb.provider.InformationProvider" pid="Provider-BBC" address="http://localhost:9797/ProviderBBC">
		<qos>
			<!-- Disponibilidade -->
			<attribute operation="priceAlert" value="95.0" name="Availability">
				<simulation>
					<interval unit="ms" time="60000">
						<distribution name="bernoulli">
							<parameter value="100.0" name="mean" />
						</distribution>
					</interval>
					<interval unit="ms" time="15000">
						<distribution name="bernoulli">
							<parameter value="0.0" name="mean" />
						</distribution>
					</interval>
					<interval unit="ms" time="45000">
						<distribution name="bernoulli">
							<parameter value="0.0" name="mean" />
						</distribution>
					</interval>
				</simulation>
				</attribute>
				<!-- Monitoracao funcional -->
				<attribute operation="priceAlert" name="BusinessException">
					<simulation>
						<interval unit="ms" time="75000"></interval>
						<interval unit="ms" time="3000" exceptionClass="br.com.bb.exception.OutOfScheduleException">
						<interval unit="ms" time="42000"></interval>
						</interval>
					</simulation>
				</attribute>
				<!-- Tempo de resposta -->
				<attribute operation="priceAlert" value="200" name="AvgResponseTime">
					<simulation>
						<interval unit="ms" time="120000">
							<distribution name="exponential">
								<parameter value="500.0" name="mean" />
							</distribution>
						</interval>
					</simulation>
				</attribute>
			</qos>
		</service>
</services>

No descritor acima, cada servi�o individual � descrito pela tag <service> que indica al�m da interface exposta pelo mesmo, um identificador �nico
e um endere�o no qual o servi�o deve ser disponibilizado. De fato, o atributo address � utilizado como uma forma resumida e pr�-definida de configura��o 
de um servi�o que exp�e, por default, o mesmo como sendo um Web Service. No exemplo apresentado acima, um servi�o simulado exp�e a interface InformationProvider 
no endere�o indicado. Este servi�o simulado � programado para responder com valores padr�o �s requisi��es feitas �s opera��es do servi�o. O simulador � 
respons�vel por configurar as propriedades mencionadas de forma a expor as interfaces do servi�o no endere�o indicado utilizando Web Services como mecanismo de gera��o
de proxies.


 
2. Executando a Plataforma DSOA:
Select dsoa-runtime project and "Run as... Java Application" 

2.1. Installing DSOA Service Simulator: (J� FORAM EMBUTIDOS NA PLATAFORMA)
#start file:apps/simulation/commons-math-2.2.jar
#start file:apps/simulation/dsoa-qos-simulator.jar

2.2. Installing and Executing Homebroker Example:
start file:apps/HomebrokerModel.jar
start file:apps/simulated-service.jar
# AT� ESSE PONTO, O BUNDLE DA PLATAFORMA ESTAR� NORMALMENTE NO AR, MAS N�O HAVER� F�BRICA DE COMPONENTE PUBLICADA
# UMA VEZ QUE NENHUM DsoaComponent FOI DECLARADO
start file:apps/simulation/simulatedBbc.jar
start file:apps/simulation/simulatedBlo.jar
start file:apps/simulation/simulatedCnn.jar
start file:apps/simulation/simulatedBov.jar
start file:apps/HomebrokerBB.jar
start file:apps/HomebrokerBBClient.jar

arch -instance Homebroker-BB-Instance

2.3. Basic Felix Commands:
A plataforma DSOA foi constru�da em cima de OSGi/DOSGi (em particular, Felix e CXF/Zookeeper) e iPojo. Neste contexto, conhecer os comandos disponibilizados pelo shell do 
Felix � bastante �til para a administra��o da plataforma. Segue abaixo uma descri��o dos principais comandos de administra��o dispon�veis:
2.3.1 ps: lists bundles
2.3.2 install/start file: comandos para instalar e inicializar bundles do OSGi na plataforma.
2.3.3 inspect service|package capability|requirement [id]: lists the services provided/required by bundle identified by id
2.3.4 arch 
	-factories: lists component factories
	-factory factory_name: details component factory
	-instances: lists component instances
	-instance instance_name: details component instance 

3. Executando Cen�rios de Teste
Para executar testes de carga utilizamos a Ferramenta JMeter (localmente em C:\Java\jakarta-jmeter-2.5.1\bin).

Observa��es:
a. A vers�o da plataforma aqui publicada N�O UTILIZA os interceptadores do CXF. A vers�o anterior, que utilizava tais interceptadores
dependia de uma vers�o personalizada (instrumentada do DOSGi) dispon�vel localmente em c:\Sources\dosgi2.