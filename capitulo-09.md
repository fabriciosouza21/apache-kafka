# Usando chaves nas mensagens

Em operações bancárias ou no fluxo de um workflow.

Garante apenas que todas as mensagens com a mesma chave serão enviadas para a mesma partição.

O kafka garante que todas as mensagens que compartilham a mesma chave sejam enviadas para srem alocadas sempre na mesm partição.

Dentro de uma partição, as mensagens são ordenadas por ordem de chegada.

## Iplementação do consumidor

### Cliente Kafka

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
        kafkaTemplate.send(SHOP_TOPIC_NAME, msg.getBuyerIdentifier(), msg);
    }
}
```


### Consumidor

``` java

@KafkaListener(topics = SHOP_TOPIC_NAME, groupId = "group")
public void listenShopTopic(ShopDTO shopDTO,
        @Header(KafkaHeaders.RECEIVED_MESSAGE_KEY) String key,
        @Header(KafkaHeaders.RECEIVED_PARTITION_ID) String partitionId,
        @Header(KafkaHeaders.RECEIVED_TIMESTAMP) String timestamp) {

    log.info("Compra recebida no tópico: {} com chave {} na partição {} hora {}.",
            shopDTO.getIdentifier(), key, partitionId, timestamp);

    // continuação do método
}
```
