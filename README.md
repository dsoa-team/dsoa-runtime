dsoa-runtime
============

1. Pré-Requisitos:
A versão atual da plataforma está preparada para rodar em um ambiente distribuído com base na implementação de referência do DOSGi, a qual utiliza
o Zookeeper como registro de serviços distribuído. Neste contexto, para que a plataforma funcione corretamente, o Zookeeper deve ser instalado e 
executado em uma máquina da rede que é indicada no arquivo load/org.apache.cxf.dosgi.discovery.zookeeper.cfg carregado quando da inicialização da 
plataforma. A implementação de DOSGi é responsável por obter a descrição dos serviços locais que devem ser expostos remotamente e publicá-las no registro
distribuído que pode ser consultado pelos clientes para obter um proxy para o referido serviço.

Na implementação de DOSGi, para que um serviço publicado localmente no registro do OSGi possa ser exposto como um serviço remoto, 
a seguintes propriedades devem ser setadas:
	1. "service.exported.interfaces": Indica quais interfaces do serviço devem ser expostas para acesso remoto. Um "*" indica
	que todas as interfaces implementadas pelo serviço devem estar remotamente acessíveis.
	2. "service.exported.configs": Indica como o serviço deve ser exposto. O valor "org.apache.cxf.ws" indica que o serviço deve ser exposto como um Web
	Service.			
	3. "org.apache.cxf.ws.address": Indica em que endereço o serviço será disponibilizado

A publicação da descrição de um serviço com as propriedades indicadas acima em um registro distribuído requer que o zookeeper (baixado separadamente) esteja em execução na rede. 
Para iniciar o zookeeper, utiliza-se o script zkServer.cmd disponível no diretório_de_instalação_do_zookeeper/bin. A publicação de um serviço no zookeeper corresponde à 
escrita de um descritor do servidor na pasta /osgi/service_registry. Nela, o dosgi/zookeeper organiza os serviços através dos nomes completos
das interfaces expostas. Assim, um serviço que expõe a interface br.com.bb.provider.InformationProvider estará publicado na pasta br/com/bb/provider/InformationProvider.
Em suma, para que um serviço OSGi possa ser exposto remotamente, ele deverá ser publicado localmente com as propriedades indicadas acima setadas. Neste contexto, a implementação
do DOSGi, que é notificada da publicação local do serviço, lê as suas propriedades e monta um descritor de serviço em formato XML. Este descritor é escrito no Zookeeper server e 
fica disponível para a consulta pelos clientes em execução nas instâncias da plataforma em execução na rede.
 
O zookeeper possui uma ferramenta cliente própria que pode ser utilizada por um administrador para consultar os serviços que estão publicados no registro. Este cliente,
zkCli, é uma ferramenta de linha de comandos que permite a listagem dos serviços publicados. Utilizando esta ferramenta podemos executar um comando tal como:

$> ls /osgi/service_registry/br/com/bb/provider/InformationProvider
[localhost#9797##ProviderBBC]

O resultado do comando, indicado entre colchetes, mostra que há um arquivo chamado localhost#9797##ProviderBBC que descreve o serviço exposto no endereço
correspondente. Para obtermos o descritor do serviço armazenado no Zookeeper, podemos utilizar o comando get, como indicado abaixo:

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

Veja que o comando get recupera o arquivo xml utilizado como descritor de serviço, o qual indica, dentre outras coisas o endereço do serviço e que o mesmo
está exposto através de Web Services. Além destas informações, o arquivo apresenta um conjunto de propriedades utilizadas para descrever a qualidade
do serviço exposto. Ex:
<property name="constraint.operation.qos.AvgResponseTime.getCotation.LE" value-type="Double" value="300.0" />

A propriedade descrita acima é interpretada pela plataforma DSOA como parte de uma Quality Offer que indica que o serviço exposto terá um valor médio de tempo de resposta
de 300 milissegundos (unidade default de tempo).
	
Para prover um serviço com atributos de qualidade tais como tempo médio de resposta e disponibilidade, a plataforma DSOA introduz um simulador de qualidade. Para o simulador,
um serviço a ser exposto deve ser descrito através de um arquivo xml indicando de que forma o serviço deve ser simulado. Um descritor exemplo é indicado abaixo:

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

No descritor acima, cada serviço individual é descrito pela tag <service> que indica além da interface exposta pelo mesmo, um identificador único
e um endereço no qual o serviço deve ser disponibilizado. De fato, o atributo address é utilizado como uma forma resumida e pré-definida de configuração 
de um serviço que expõe, por default, o mesmo como sendo um Web Service. No exemplo apresentado acima, um serviço simulado expõe a interface InformationProvider 
no endereço indicado. Este serviço simulado é programado para responder com valores padrão às requisições feitas às operações do serviço. O simulador é 
responsável por configurar as propriedades mencionadas de forma a expor as interfaces do serviço no endereço indicado utilizando Web Services como mecanismo de geração
de proxies.


 
2. Executando a Plataforma DSOA:
Select dsoa-runtime project and "Run as... Java Application" 

2.1. Installing DSOA Service Simulator: (JÁ FORAM EMBUTIDOS NA PLATAFORMA)
#start file:apps/simulation/commons-math-2.2.jar
#start file:apps/simulation/dsoa-qos-simulator.jar

2.2. Installing and Executing Homebroker Example:
start file:apps/HomebrokerModel.jar
start file:apps/simulated-service.jar
# ATÉ ESSE PONTO, O BUNDLE DA PLATAFORMA ESTARÁ NORMALMENTE NO AR, MAS NÃO HAVERÁ FÁBRICA DE COMPONENTE PUBLICADA
# UMA VEZ QUE NENHUM DsoaComponent FOI DECLARADO
start file:apps/simulation/simulatedBbc.jar
start file:apps/simulation/simulatedBlo.jar
start file:apps/simulation/simulatedCnn.jar
start file:apps/simulation/simulatedBov.jar
start file:apps/HomebrokerBB.jar
start file:apps/HomebrokerBBClient.jar

arch -instance Homebroker-BB-Instance

2.3. Basic Felix Commands:
A plataforma DSOA foi construída em cima de OSGi/DOSGi (em particular, Felix e CXF/Zookeeper) e iPojo. Neste contexto, conhecer os comandos disponibilizados pelo shell do 
Felix é bastante útil para a administração da plataforma. Segue abaixo uma descrição dos principais comandos de administração disponíveis:
2.3.1 ps: lists bundles
2.3.2 install/start file: comandos para instalar e inicializar bundles do OSGi na plataforma.
2.3.3 inspect service|package capability|requirement [id]: lists the services provided/required by bundle identified by id
2.3.4 arch 
	-factories: lists component factories
	-factory factory_name: details component factory
	-instances: lists component instances
	-instance instance_name: details component instance 

3. Executando Cenários de Teste
Para executar testes de carga utilizamos a Ferramenta JMeter (localmente em C:\Java\jakarta-jmeter-2.5.1\bin).

Observações:
a. A versão da plataforma aqui publicada NÃO UTILIZA os interceptadores do CXF. A versão anterior, que utilizava tais interceptadores
dependia de uma versão personalizada (instrumentada do DOSGi) disponível localmente em c:\Sources\dosgi2.