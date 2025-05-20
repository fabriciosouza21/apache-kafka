# Introdução

## Apache Kafka

O Apache Kafka é uma ferramenta para o porcessamento de fluxos de evento que são representados na forma de mensagens. O Kafka recebe uma mensagem indicando um evento.

Essas três características são comuns a praticamente todos os sistemas de fila.

### Produtor

sistema que gera um evento, enviará uma mensagem para o kafka indicando que esse evento ocorreu.


### Tópico

O  Kafka pode possuir vários tópicos, que são formas de separar as mensagens e, principalmente, agrupar quem vai receber cada uma das mensagens. Sempre que um produtor envia uma mensagem, ele deve indicar para qual tópico da kafka a mensagem está sendo enviada.


## Consumidor
sistema que rece a mensagem do kafka. Um consumidor sempore se inscreve em um tópico e recebe, por meio kafka, as mensagens enviadas para esse tópico.


## Vantagem dos sistemas de filas

Todas mensagem no Kafka é formada por três valores:
- chave
- valor(mensagem)
- timestamp

``` yml
Key: "compra-1"
Value: "Comprador: Eduardo, CPF: 123, Valor: 500"
Timestamp: "Jun. 25, 2020 at 2:06 p.m."
```

### Durabilidade

O kafka armazena as mensagens toda a sua configuração e os dados em disco, seja em finalização normal ou por causa de uma falha.

### Paralelismo

permite que diferente processos se conectem a um mesm **tópico**. O kafka permite que processos recebam as mensagens de forma paralelamente de forma automática.

### Balacemento de carga

O Kafka possuir um mecanismo proprio de para distribuir as mensagens em diferente processos que estão inscritos no mesmo tópico.


### Streams

O Kafk possui suporte ao processament de fluxos contínuos de dados nativo.



## Spring Boot


### Dependências

Consumidor em java com spring boot

- spring-starter-web
- spring-kafka-client (utilizado para receber e enviar mensagens para um tópico do kafka)
- spring-data-jpa
- spring-boot-starter-test

## Python

Vai ser implementado um consumidor em python

- kafka-python
- flask

## Versões do kafka, Spring boot e python

Usar o KRaf, apartir da versão 4.0.0 não vai mais haver suporte para ZooKeeper.

spring boot 3.3.5 e java 21

python 3.13

