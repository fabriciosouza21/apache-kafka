# Capítulo 2

## Instalação do Kafka

(https://kafka.apache.org/)[https://kafka.apache.org/]


para iniciar o kafka basta baixa o arquivo, definir o Id

``` bash

KAFKA_CLUSTER_ID="$(bin/kafka-storage.sh random-uuid)"
./bin/kafka-storage.sh format -t $KAFKA_CLUSTER_ID -c config/kraf
t/server.properties
```


comando para iniciar o kafka

``` bash
./bin/kafka-server-start.sh config/kraft/server.properties
```

## Criar um tópico

é aonde adicionamos e lemos as mensagens que as aplicações vaõ compartilhar.

``` bash
./bin/kafka-topics.sh \
--create # criar um novo tópico \
--topic topico-teste # nome do tópico \
--bootstrap-server localhost:9092 # endereço do kafka
```


## Listar os tópicos

``` bash
./bin/kafka-topics.sh \
--list \
--bootstrap-server localhost:9092
```

## Detalhar um tópico

``` bash
./bin/kafka-topics.sh \
--describe \
--topic topico-teste \
--bootstrap-server localhost:9092
```


``` yml
Topic: topico-teste
TopicId: om2YlC2yQrSKraJZ1zk_gA
PartitionCount: 1
ReplicationFactor: 1
Configs: segment.bytes=1073741824
Topic: topico-teste
Partition: 0
Leader: 0
Replicas:0
Isr: 0
```


## Enviando e recebendo mensagens

### Conectando ao cluster

Esse comando iniciará um Consumidor de mensagens, cada mensagem digitada será enviada para o tópico especificado.

``` bash
./bin/kafka-console-producer.sh  \
--topic topico-teste #topico \
--bootstrap-server localhost:9092 # endereço do kafka
```


podemos criar mensagem para os tópicos

``` bash
bin/kafka-console-consume
r.sh --topic topico-teste --bootstrap-server localhost:9092
mensagem1
mensagem2
mensagem3
```

Por padrão o kafka o producer só começa a receber mensagens que foram enviadas depois que le é criado.

podemos alterar esse comportamento com o parâmetro --from-beginning, que faz com que o producer receba todas as mensagens do tópico, desde a sua criação.

``` bash
./bin/kafka-console-consumer.sh \
--topic topico-teste \
--from-beginning \
--bootstrap-server localhost:9092
```

## Consumer Groups

No kafka é possível definir grupos de consumidores, que definiram como será a distribuição das mensagens entre os consumidores de um tópico.

definindo o grupo de consumidores

``` bash

./bin/kafka-console-consumer.sh \
--topic topico-teste \
--from-beginning \
--bootstrap-server localhost:9092 \
--consumer-property group.id=grupo-1
```

podemos ter mais de um grupo de consumidores,



``` bash
./bin/kafka-console-consumer.sh \
--topic topico-teste \
--from-beginning \
--bootstrap-server localhost:9092 \
--consumer-property group.id=grupo-2
```

Os dois consumidores estão lendo as mesmas mensagens, mas cada um está em um grupo diferente, então cada um vai receber todas as mensagens do tópico. eles podem ser usados para fazer processamento paralelo, por exemplo, um grupo pode fazer o processamento e o outro pode fazer trabalhar com envio de e-mail.

> Os grupos são um dos mecanismo mais pedeross do kafka, eles permitem que façamos o balanceamento de carga e a distribuição de mensagens sem praticamente temos que implementar nenhuma linha de código.
>
>

## Paranda o Kafka

basta para a execução do kafka, os dados ainda estarão disponíveis, pois o kafka armazena os dados em disco.

para remover os dados é preciso apagar a a pasta

``` bash
rm -rf /tmp/kraft-combined-logs
```
