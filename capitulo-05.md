# Consumidor


No exemplo do livro é uma aplicação java, bem simples, a unica difrerença é que tem que adicionar o spring-kafka no pom.xml. Além disso precisamos criar um listener para o tópico que queremos escutar.

``` java

package com.santana.service;

import org.springframework.kafka.annotation.KafkaListener;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.stereotype.Service;
import com.santana.dto.ShopDTO;
import com.santana.dto.ShopItemDTO;
import com.santana.entity.Product;
import com.santana.repository.ProductRepository;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;

@Slf4j
@Service
@RequiredArgsConstructor
public class ReceiveKafkaMessage {

    private static final String SHOP_TOPIC_NAME = "SHOP_TOPIC";
    private static final String SHOP_TOPIC_EVENT_NAME = "SHOP_TOPIC_EVENT";

    private final ProductRepository productRepository;
    private final KafkaTemplate<String, ShopDTO> kafkaTemplate;

    @KafkaListener(topics = SHOP_TOPIC_NAME, groupId = "group")
    public void listenShopTopic(ShopDTO shopDTO) {
        try {
            log.info("Compra recebida no tópico: {}.", shopDTO.getIdentifier());
            boolean success = true;

            for (ShopItemDTO item : shopDTO.getItems()) {
                Product product = productRepository.findByIdentifier(item.getProductIdentifier());
                if (!isValidShop(item, product)) {
                    shopError(shopDTO);
                    success = false;
                    break;
                }
            }

            if (success) {
                shopSuccess(shopDTO);
            }
        } catch(Exception e) {
            log.error("Erro no processamento da compra {}", shopDTO.getIdentifier());
        }
    }

    // valida se a compra possui algum erro
    private boolean isValidShop(ShopItemDTO item, Product product) {
        return product != null && product.getAmount() >= item.getAmount();
    }

    // Envia uma mensagem para o Kafka indicando erro na compra
    private void shopError(ShopDTO shopDTO) {
        log.info("Erro no processamento da compra {}.", shopDTO.getIdentifier());
        shopDTO.setStatus("ERROR");
        kafkaTemplate.send(SHOP_TOPIC_EVENT_NAME, shopDTO);
    }

    // Envia uma mensagem para o Kafka indicando sucesso na compra
    private void shopSuccess(ShopDTO shopDTO) {
        log.info("Compra {} efetuada com sucesso.", shopDTO.getIdentifier());
        shopDTO.setStatus("SUCCESS");
        kafkaTemplate.send(SHOP_TOPIC_EVENT_NAME, shopDTO);
    }
}
```

Esse listener, consumidor também é um produtor, ele envia mensagens para o tópico SHOP_TOPIC_EVENT_NAME, indicando se a compra foi realizada com sucesso ou não.




