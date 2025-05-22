# Capítulo 4

## Configuração

A biblioteca srping-kafka terá todo o código de que necessitaremos para implementa tanto o produto quanto o cosumidor.

``` xml

<dependency>
<groupId>org.springframework.kafka</groupId>
<artifactId>spring-kafka</artifactId>
</dependency>
```



## Implementação o produto

O produto é o código que gerar a mensagem que será enviada para o kafka.


`kafkaConfig` é apenas para configura o kafka em nossa aplicação


para configurar o produtor, utilizamos o bean `kafkaTemplate` que é o responsável por enviar as mensagens para o kafka.


`bootstrapsAddress` definira o endereço do kafka que vamos acessar.


`producerFactory`

os imports devem ser da biblioteca do spring-kafka.

``` java

package com.santana.events;

import java.util.HashMap;
import java.util.Map;
import org.apache.kafka.clients.consumer.ConsumerConfig;
import org.apache.kafka.clients.producer.ProducerConfig;
import org.apache.kafka.common.serialization.StringDeserializer;
import org.apache.kafka.common.serialization.StringSerializer;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.kafka.config.ConcurrentKafkaListenerContainerFactory;
import org.springframework.kafka.core.ConsumerFactory;
import org.springframework.kafka.core.DefaultKafkaConsumerFactory;
import org.springframework.kafka.core.DefaultKafkaProducerFactory;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.kafka.core.ProducerFactory;
import org.springframework.kafka.support.serializer.JsonDeserializer;
import org.springframework.kafka.support.serializer.JsonSerializer;
import com.santana.dto.ShopDTO;

@Configuration
public class KafkaConfig {

    @Value(value = "${kafka.bootstrapAddress:localhost:9092}")
    private String bootstrapAddress;

    public ProducerFactory<String, ShopDTO> producerFactory() {
        Map<String, Object> props = new HashMap<>();
        props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapAddress);
        props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, JsonSerializer.class);
        props.put(ProducerConfig.CLIENT_ID_CONFIG, "shop-api");
        return new DefaultKafkaProducerFactory<>(props);
    }

    @Bean
    public KafkaTemplate<String, ShopDTO> kafkaTemplate() {
        return new KafkaTemplate<>(producerFactory());
    }

    public ConsumerFactory<String, ShopDTO> consumerFactory() {
        JsonDeserializer<ShopDTO> deserializer = new JsonDeserializer<>(ShopDTO.class);
        Map<String, Object> props = new HashMap<>();
        props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapAddress);
        return new DefaultKafkaConsumerFactory<>(props, new StringDeserializer(), deserializer);
    }

    @Bean
    public ConcurrentKafkaListenerContainerFactory<String, ShopDTO> kafkaListenerContainerFactory() {
        ConcurrentKafkaListenerContainerFactory<String, ShopDTO> factory =
            new ConcurrentKafkaListenerContainerFactory<>();
        factory.setConsumerFactory(consumerFactory());
        return factory;
    }
}
```

## Kafka Client


com o Bean `kafkaTemplate` podemos enviar mensagens para o kafka.

``` java

package com.santana.events;

import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.stereotype.Service;
import com.santana.dto.ShopDTO;
import lombok.RequiredArgsConstructor;

@Service
@RequiredArgsConstructor
public class KafkaClient {

    private final KafkaTemplate<String, ShopDTO> kafkaTemplate;
    private static final String SHOP_TOPIC_NAME = "SHOP_TOPIC";

    public void sendMessage(ShopDTO msg) {
        kafkaTemplate.send(SHOP_TOPIC_NAME, msg);
    }
}
```


## Enviado os objetos para tópicos



``` java
package com.santana.controller;

import java.time.LocalTime;
import java.util.UUID;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import com.santana.dto.ShopDTO;
import com.santana.entity.Shop;
import com.santana.entity.ShopItem;
import com.santana.events.KafkaClient;
import com.santana.repository.ShopRepository;
import lombok.RequiredArgsConstructor;

@RestController
@RequestMapping("/shop")
@RequiredArgsConstructor
public class ShopController {

    private final ShopRepository shopRepository;
    private final KafkaClient kafkaClient;

    // other methods...

    @PostMapping
    public ShopDTO saveShop(@RequestBody ShopDTO shopDTO) {
        shopDTO.setIdentifier(UUID.randomUUID().toString());
        shopDTO.setDateShop(LocalTime.now());
        shopDTO.setStatus("PENDING");

        Shop shop = Shop.convert(shopDTO);
        for (ShopItem shopItem : shop.getItems()) {
            shopItem.setShop(shop);
        }

        shopDTO = ShopDTO.convert(shopRepository.save(shop));
        kafkaClient.sendMessage(shopDTO);

        return shopDTO;
    }
}
```


## Verificando a mensagem no kafka

``` bash

./bin/kafka-console-consumer.sh \
--topic SHOP_TOPIC \
--bootstrap-server localhost:9092 \
--from-beginning

```


``` json
{"identifier":"d351ea15-345d-4068-8f8d-e48812c8169b","dateShop":[
10,7,4,848385000],"items":[{"productIdentifier":"123456789","amou
nt":100,"price":1000.0},{"productIdentifier":"123456789","amount"
:100,"price":1000.0}]}
```
