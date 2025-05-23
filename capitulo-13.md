# Configurações do KAFKA

## Configurações gerais

`admin.serverPort`

``` yml
clientPort=2181
admin.enableServer=false
# admin.serverPort=8080
```

`server.properties`

``` properties

broker.id=0
num.partitions=1
log.dirs=/tmp/kafka-logs # diretório onde o kafka vai armazenar os logs
# tempo de retenção dos logs e tamanho máximo dos logs
log.dirs=/tmp/kafka-logs
log.retention.hours=168
log.retention.bytes=1073741824
# Configuração do zookeeper
zookeeper.connect=localhost:2181
zookeeper.connection.timeout.ms=18000

```

## Configuração do consumidor

para alterar a configuração basta adicionar a configuração no Map, como no exemplo abaixo

``` java
public ConsumerFactory<String, ShopDTO> consumerFactory() {
    JsonDeserializer<ShopDTO> deserializer =
        new JsonDeserializer<>(ShopDTO.class);
    Map<String, Object> props = new HashMap<>();
    props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapAddress); // endereço do kafka
    return new DefaultKafkaConsumerFactory<>(
        props,
        new StringDeserializer(),
        deserializer);
}
```

## Configuração consumidor


### ALLOW_AUTO_CREATE_TOPICS_CONFIG
por padrão é true
### MAX_POLL_RECORDS_CONFIG
(indicar quantas mensagens o consumidor vai consumir por vez, padrão 500)
### AUTO_OFFSET_RESET_CONFIG
por padrão é 'latest' pode ser 'earliest' caso queira consumir todas as mensagens do tópico, o valor latest é o padrão, ou seja, o consumidor vai consumir apenas as mensagens que foram publicadas depois que o consumidor foi iniciado.
### GROUP_ID_CONFIG
Podemos configurar para que todos os consumidores para usar o mesmo consumer, todos os consumidores por padrão estam no mesmo grupo, Com essa configuração não precisamos definir o grupo na anotaçã @KafkaListener.
### HEARTBEAT_INTERVAL_MS_CONFIG
mecanismo chamado heartbeat para que o consumidor notifique o kafk que ele ainda está funcionando. Por padrão, esse intervalo é de 3.000 milissegundos.
### SESSION_TIMEOUT_MS_CONFIG
Podemos configura um timeout para o kafka considera que o consumidor não está mais conecado.
### ENABLE_AUTO_COMMIT_CONFIG
Outra configuração importante é habilitar ou não os commits automáticos. Por padrão, quando um consumidor iniciar o processamento de uma mensagem, ele já avisa que a mensgm já foi processada


## Configuração do produtor


basta adicionar a configuração no Map, como no exemplo abaixo

``` java

public ProducerFactory<String, ShopDTO> producerFactory() {
    Map<String, Object> props = new HashMap<>();
    props.put(
        ProducerConfig.BOOTSTRAP_SERVERS_CONFIG,
        bootstrapAddress);
    props.put(
        ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG,
        StringSerializer.class);
    props.put(
        ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG,
        JsonSerializer.class);
    return new DefaultKafkaProducerFactory<>(props);
}
```

### BATCH_SIZE_CONFIG

configuração para envia mensagens em lote. por padrão o valro padrão é 16384 bytes, caos o valor passado seja 0, o batch é desabilitado totalmente. Para cada mensagem enviada pelo produto uma nova conexão será feita.

`props.put(ProducerConfig.BATCH_SIZE_CONFIG, "16384");`


### CLIENT_ID_CONFIG

apenas para identificar o cliente, não é obrigatório, mas é interessante para identificar o cliente que está enviando a mensagem deburar mais fácil.

`props.put(ProducerConfig.CLIENT_ID_CONFIG, "client-1");`

### CONNECTIONS_MAX_IDLE_MS_CONFIG

configuração para definir o tempo máximo de inatividade de uma conexão, ou seja, o tempo máximo que uma conexão pode ficar ociosa. O valor padrão é 54.000 milissegundos (9 minutos)
