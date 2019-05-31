# Baseado em Spring Cloud Arquitetura de microsserviço

Este projeto é baseado em Spring Boot、Spring Cloud、Spring Oauth2 e Spring Cloud Netflix, projetos de microsserviços construídos por frameworks.

# Tecnologias
* Spring boot - Microframe de nível de entrada de microsserviços para simplificar a configuração inicial do aplicativo e o processo de desenvolvimento.
* Eureka - Descoberta de serviço em nuvem, uma baseada em REST serviços de localização para habilitar o failover e descoberta de serviços de camada intermediária na nuvem.
* Spring Cloud Config - O kit de ferramentas de gerenciamento de configuração permite que você coloque a configuração em um servidor remoto, gerencie centralmente a configuração do cluster e, atualmente, ofereça suporte ao armazenamento local, Git e Subversion。
* Hystrix - ferramenta de gerenciamento tolerante a falhas, projetada para controlar os nós da biblioteca e-third-party de serviço,Isso fornece uma tolerância a falhas mais robusta para atrasar falhas e.
* Zuul - Zuul é uma estrutura para fornecer serviços como roteamento dinâmico, monitoramento, resiliência e segurança na plataforma de nuvem.
* Spring Cloud Bus - Evento, barramento de mensagens, usado para propagar mudanças de estado em um cluster (por exemplo, eventos de alteração de configuração), Spring Cloud Config 联合实现热部署。
* Spring Cloud Sleuth - Kit de coleta de log Dapper e log-based 追踪以及 Zipkin e HTrace, SpringCloud O aplicativo implementa uma solução de rastreamento distribuída.
* Ribbon - Fornecer balanceamento de carga em nuvem, uma variedade de estratégias de balanceamento de carga para escolher, pode ser usado em conjunto com disjuntores de descoberta de serviço e.
* Turbine - Turbine É uma ferramenta para o servidor de agregação enviar dados do fluxo de eventos, usados ​​para monitorar o cluster hystrix de metrics
* Spring Cloud Stream - Spring Pacote de desenvolvimento de operação de fluxo de dados, empacotado com Redis、Rabbit、Kafka
* Feign - Feign É um modelo declarativo HTTP cliente
* Spring Cloud OAuth2 - Baseado em Spring Security e OAuth2 Kit de ferramentas de segurança para adicionar controles de segurança ao seu aplicativo.

# Arquitetura de aplicativos

O projeto contém 8 serviços

* registry - Registro de serviço e descoberta
* config - Configuração externa
* monitor - Monitoramento
* zipkin - Rastreamento distribuído
* gateway - Gateway proxy para todos os microserviços
* auth-service - OAuth2 Serviço de autenticação
* svca-service - Serviço comercial A
* svcb-service - Serviço comercial B

## Arquitetura
![architecture](/screenshots/architecture.jpg)
## Componente de aplicação
![components](/screenshots/components.jpg)

# Iniciar projeto

* Use Docker 
    1. Configuração de ambiente Docker
    2. `mvn clean package` empacotar projeto e gerar imagem Docker
    3. Executar no diretório raiz do projeto `docker-compose up -d` inicia todos os projetos
* Executando localmente
    1. Configuração do RabbitMQ
    2. Alterar os host para apontar para localhost
       `127.0.0.1	registry config monitor rabbitmq auth-service`  
       Ou modifique o nome do host correspondente em cada arquivo de configuração de serviço para ser ip local
    3. Inicie registry、config、monitor、zipkin
    4. Inicie gateway、auth-service、svca-service、svcb-service

# Pré-visualização do projeto

## Eureka
Acesso: http://localhost:8761/ Conta padrão user, senha password

![registry](/screenshots/registry.jpg)
## Monitoramento
访问 http://localhost:8040/ Conta padrão admin, senha admin
### Painel de controle
![monitor](/screenshots/monitor1.jpg)
### Histórico de registro de aplicativos
![monitor](/screenshots/monitor2.jpg)
### Turbine Hystrix Painel
![monitor](/screenshots/monitor3.jpg)
### Informações sobre aplicativos, status de integridade, coleta de lixo, etc.
![monitor](/screenshots/monitor4.jpg)
### Contador
![monitor](/screenshots/monitor5.jpg)
### Ver e para modificar variáveis ​​de ambiente
![monitor](/screenshots/monitor6.jpg)
### Gestão de nível de log Logback
![monitor](/screenshots/monitor7.jpg)
### Visualizar e usar JMX
![monitor](/screenshots/monitor8.jpg)
### Ver tópico
![monitor](/screenshots/monitor9.jpg)
### Histórico de certificação
![monitor](/screenshots/monitor10.jpg)
### Visualizar solicitações Http 
![monitor](/screenshots/monitor11.jpg)
### Hystrix painel
![monitor](/screenshots/monitor12.jpg)
## Acompanhamento de links
Acesso http://localhost:9411/ Conta padrão admin, senha admin
### Painel de controle
![zipkin](/screenshots/zipkin1.jpg)
### Detalhes do acompanhamento de links
![zipkin](/screenshots/zipkin2.jpg)
### Dependência de serviço
![zipkin](/screenshots/zipkin3.jpg)
## RabbitMQ Monitoramento
Docker Iniciar acesso http://localhost:15673/ Conta padrão guest, senha guest（Porta padrão do sistema de gerenciamento rabbit 15672)

![rabbit](/screenshots/rabbit.jpg)
# Teste de interface
1. Obter Token
```
curl -X POST -vu client:secret http://localhost:8060/uaa/oauth/token -H "Accept: application/json" -d "password=password&username=anil&grant_type=password&scope=read%20write"
```
Retorna os seguintes dados:
```
{
  "access_token": "eac56504-c4f0-4706-b72e-3dc3acdf45e9",
  "token_type": "bearer",
  "refresh_token": "da1007dc-683c-4309-965d-370b15aa4aeb",
  "expires_in": 3599,
  "scope": "read write"
}
```
2. Use access token para acessar service A
```
curl -i -H "Authorization: Bearer eac56504-c4f0-4706-b72e-3dc3acdf45e9" http://localhost:8060/svca
```
Retorna os seguintes dados:
```
svca-service (172.18.0.8:8080)===>name:zhangxd
svcb-service (172.18.0.2:8070)===>Say Hello
```
3. Use access token para acessar service B
```
curl -i -H "Authorization: Bearer eac56504-c4f0-4706-b72e-3dc3acdf45e9" http://localhost:8060/svcb
```
Retorna os seguintes dados:
```
svcb-service (172.18.0.2:8070)===>Say Hello
```
4. Use refresh token para atualizar token
```
curl -X POST -vu client:secret http://localhost:8060/uaa/oauth/token -H "Accept: application/json" -d "grant_type=refresh_token&refresh_token=da1007dc-683c-4309-965d-370b15aa4aeb"
```
Retorno da atualização do Token：
```
{
  "access_token": "63ff57ce-f140-482e-ba7e-b6f29df35c88",
  "token_type": "bearer",
  "refresh_token": "da1007dc-683c-4309-965d-370b15aa4aeb",
  "expires_in": 3599,
  "scope": "read write"
}
```
5. Atualizar configuração
```
curl -X POST -vu user:password http://localhost:8888/bus/refresh
```