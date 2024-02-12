# Semana 4 - Cross Cutting - Service Registry Discovery Pattern

## Service Registry Pattern

Dentro da arquitetura de MS este padrão é considerado um Cross Cutting (preocupações transversais).

Seu papel é monitorar e armazenar as instâncias e localização dos micro serviços,  possibilitando a descoberta dinâmica desses serviços pelos clientes, este é o Service Registry. Cada instância  de um micro serviço se registra em um serviço de descoberta (Eureka) com o nome lógico e suas informações de endereço (ip e portas). Isso ocorre automaticamente se eu estiver usando Spring Cloud com a configuração correta definida nos arquivos de configuração dos microsserviços. 

| PRÓS | ATENÇÃO |
|------|---------|
| Os clientes podem descobrir a localização e instâncias dos serviços de forma dinâmica | O Service Registry é um componente crítico da arquitetura e deve estar altamente disponível |
| O número de instâncias e localização podem mudar sem afetar os clientes |   |

[mais sobre este pattern](https://microservices.io/patterns/service-registry.html)

##  Qual a relação deste patter com Spring Cloud?
Este padrão é implementado através do projeto componente Spring Cloud Netflix Eureka Server dentro do Spring Cloud

## como eu aprendi a configurar um service registry no projeto Decoder?
- criar um ms para ser o service registry
- declarar a dependência do Spring Cloud Starter Netflix Eureka Server no MS Service Registry.
- Definir a porta de funcionamento da aplicação
- Definir o nome da aplicação
- Habilitar a aplicação como um Servidor Eureka Server
- No arquivo de configuração da aplicação inserir as configurações para definir a aplicação como um Servidor Eureka Client.
	
##  Client side Discovery Pattern

Responsável por enviar ao Service Registry(eureka server) informações para que o Service Registry seja capaz de fornecer tais informações para os clientes(Eureka clients) do serviço que desejam enviar requisições. Quando um cliente(Eureka clients) do serviço fizer uma requisição, este cliente obtém a localização da instância de um serviço consultando o Service Registry que conhece todas as localizações. Todo Eureka Cliente já se registra no Service Registry logo que a aplicação é iniciada.

## Como configurar um Client side Discovery Pattern no projeto Decoder?
- declarar a dependência do Spring Cloud Starter Netflix Eureka Client nos serviços AuthUser e Course.
- Habilitar a aplicação como Eureka Cliente do Eureka Serve(Service Registry)
- Configurar o yml para os serviços AuthUser e Course encontrarem o Service Registry para poderem se registrar nele


## O papel do Load Balance 
No projeto Decoder os micro serviços AuthUser e Course  foram configurados para serem cliente Eureka em seus arquivos de configuração e ambos têm um Bean RestTemplate anotados com @Loadbalanced.
Um loadbalance  serve  para distribuir o tráfego de forma inteligente entre instâncias de um micro serviço, melhorando assim a disponibilidade e resiliência do sistema.
Em um contexto de várias instâncias de um serviço que está em um ambiente de micro serviços/nuvem que utilizam orquestradores de containers como Kubernetes, os endereços IP e portas podem mudar frequentemente. Usar a anotação @LoadBalanced garante que o aplicativo cliente ou serviço cliente não precisa saber exatamente onde cada instância do serviço está rodando.

##  A relação anotação @LoadBalanced com o RestTemplate/Web Client 
Um bean do tipo RestTemplate/WebClient sem a anotação @LoadBalanced fará chamadas HTTP diretas para o endereço especificado (definido previamente no arquivo de configuração yml).
Ao anotar um bean do tipo RestTemplate/Web Client com a anotação @LoadBalanced  o Spring Cloud irá interceptar as requisições para que seja usado o nome lógico do serviço na url (previamente configurado como um cliente Eureka por exemplo em um serviço de registro). Dessa forma, a descoberta da instância do serviço adequado para atender a requisição será de responsabilidade do Spring Cloud para que eu não precise lidar com as mudanças de IP que as instâncias desse serviços venham a sofrer. 
Permite que o cliente se comunique com o Service Registry para identificar as instâncias disponíveis também do serviço atendente. No caso de mais de uma Instância do serviço atendente disponível faz o balanceamento de requisições para dividir entre as instâncias ativas de um micro serviço.


##  Qual a relação entre Service Registry Discovery Pattern, Service Registry Pattern e Client Side Discovery?
- **Service Registry Discovery Pattern**: Fornece um meio para haver comunicação entre serviços sem a necessidade de declarar ips/urls. É responsável por descrever como os serviços se registram ou descobrem uns aos outros.
- **Service Registry Pattern**:  faz parte do Service Registry Discovery Pattern sua função é Fazer o registro de serviços.  Descreve a forma como os serviços se registram no Service Registry Discovery Pattern . Esse registro de informações contém portas metadados endereços IP dos serviços disponíveis no sistema
- **Client Side Discovery**: responsável por enviar dados ao Service Registry para que este seja capaz de fornecer informações necessárias para futuros clientes do serviço( que implementa  Client Side Discovery Pattern.) terem suas requisições atendidas. 

##  O que é Cross Cutting?
São questões/ aspectos que afetam múltiplos componentes ou serviços de um arquitetura de micro serviços.  são tratados de forma unificada para garantir consistência reutilização de código e eficiência. São preocupações transversais:
- Autenticação e autorização
- Logging e monitoramento
- tratamento de erro e resiliência
- comunicação entre microservices
- descoberta e registro de serviços
- balanceamento de carga e escalabilidade
- segurança
- gestão de configuração e infraestrutura.

##  API Gateway Pattern 
Implementação de um serviço que atua como único ponto de entrada para arquitetura, roteando as requisições para os respectivos micro serviços.

| Prós | Atenção |
|------|---------|
| Os clientes não conhecem a arquitetura de micro serviços e nem a localização das instâncias | API Gateway É um componente crítico da arquitetura e deve estar altamente disponível |
| O cliente acessa um único ponto para obter todos os recursos simplificando a interação | Pode acontecer aumento no tempo de resposta |

##  API Gateway Pattern no Spring Cloud

Spring Cloud Gateway é a Implementação concreta do padrão de projeto para micro serviços API Gateway. Permite que desenvolvedores criem e gerenciam( através de um único e Centralizado ponto) solicitações de cliente a diferente serviços de back end, especialmente em arquitetura de micro serviços.  sSimplifique a interação cliente/micro serviço através de uma interface além de um conjunto de funcionalidades: 
- **Predicate**: Condição verdadeira para combinar com a rota. É responsável por redirecionar solicitações para um micro serviço capaz de atender a esta solicitação. Para o redirecionamento ocorrer uma ou mais condições precisam ser atendidas(caminho da URI, método HTTP, cabeçalhos). Em resumo, o precicate  faz ou não roteamento de solicitações http quando uma ou mais condições pré-definidas são atendidas nos dados da requisição.
- **Handler Mapping**: Com base nas condições pré-definidas no predicate, este componente do API Gateway Encaminhará a solicitação recebida para algum micro serviço capaz de processar. O handler é responsável por associar solicitações http com os micro serviços da aplicação com o aval de algum predicate.
- **Filter Request**:  Analisam e/ ou modificam solicitações enviadas para os micro serviços da aplicação.
- **Filter Response**: Analisam e/ou  modificam as respostas recebidas dos micro serviços antes de retorná-la ao cliente solicitante do recurso

## Como API Gateway foi implementado no projeto decoder?

- Foi criado um Micro serviço (API Gateway) para ser o ponto de entrada da aplicação como um todo.
- Este serviço tem as dependências Spring Cloud Starter Netflix Eureka Client para registrá-lo no Service Registry e a dependência spring-cloud-starter-gateway que fornece os recursos necessários para transformar esta aplicação em um API Gateway. Também está configurado neste MS
- Tornar o micro serviço um cliente Eureka no arquivo de configuração
- Definir as rotas de configuração no arquivo de configuração.
- Definir um context-path nos micro-serviço AuthUser e Course para garantir que as rotas definidas no API Gateway sejam redirecionadas a para os serviços capazes de atendê-las.

##  Interação entre Service Discovery Pattern, Client Side Discovery Pattern, Service Registry e API Gateway?

Descoberta de serviço: Um serviço se registra no Service Registry. O API Gateway e/ou Os clientes consultam o Service Registry  para descobrir os serviços disponíveis.
- **Encaminhamento de solicitações**: Quando uma solicitação chega ao API Gateway ele consulta o Service Registry e decide a qual serviço encaminhar a solicitação se o Service Registry for usado, ele seleciona a Instância apropriada.
- **Client-Side Discovery**: Os serviços clientes (incluindo o  API Gateway, quando atuando em nome dos clientes para encaminhar requisições) utilizam o service registre para descobrir e selecionar diretamente as instâncias de serviços disponíveis.
- **Balanceamento de carga**: o Load Balancer ou a lógica de balanceamento de carga o cliente distribui a solicitação para evitar sobrecarga em qualquer Instância de serviço

