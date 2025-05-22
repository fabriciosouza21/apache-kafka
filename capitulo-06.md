# Finalizando a shop-api

Criar um consumidor de mensagens que vai receber as mensagens do tópico.

``` java
package com.santana.events;

import org.springframework.kafka.annotation.KafkaListener;
import org.springframework.stereotype.Service;
import com.santana.dto.ShopDTO;
import com.santana.model.Shop;
import com.santana.repository.ShopRepository;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;

@Slf4j
@Service
@RequiredArgsConstructor
public class ReceiveKafkaMessage {

    private final ShopRepository shopRepository;
    private static final String SHOP_TOPIC_EVENT_NAME = "SHOP_TOPIC_EVENT";

    @KafkaListener(topics = SHOP_TOPIC_EVENT_NAME, groupId = "group")
    public void listenShopEvents(ShopDTO shopDTO) {
        try {
            log.info("Status da compra recebida no tópico: {}.", shopDTO.getIdentifier());
            Shop shop = shopRepository.findByIdentifier(shopDTO.getIdentifier());
            shop.setStatus(shopDTO.getStatus());
            shopRepository.save(shop);
        } catch(Exception e) {
            log.error("Erro no processamento da compra {}", shopDTO.getIdentifier());
        }
    }
}
```


Um resumo geral sobre a utilização básica: no caso do fluxo de compras, primeiro criamos uma aplicação que vai enviar as mensagens para o tópico. Temos uma outra aplicação que vai receber a mensagem e validar se a compra é válida, enviando então a mensagem para outro tópico com o status da compra. Para finalizar, a aplicação que gerou o pedido vai receber a mensagem com o status da compra.





