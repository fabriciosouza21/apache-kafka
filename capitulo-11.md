# Administrando o kakfka no java

depêndecia do kafka

``` xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
	https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>com.santana</groupId>
	<artifactId>kafka-manager</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<dependencies>
		<dependency>
			<groupId>org.apache.kafka</groupId>
			<artifactId>kafka-clients</artifactId>
			<version>3.8.0</version>
		</dependency>
	</dependencies>
</project>
```


Basicamente essa dependência é a biblioteca do kafka, que é o que vamos usar para fazer a comunicação com o kafka. Para gerenciar as funções do kafka.

Função principal para fazer a comunicação com o kafka.

``` java
package com.santana.kafka.admin;

import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.ExecutionException;
import org.apache.kafka.clients.admin.AdminClient;
import org.apache.kafka.clients.admin.CreateTopicsResult;
import org.apache.kafka.clients.admin.DeleteConsumerGroupsResult;
import org.apache.kafka.clients.admin.DeleteTopicsResult;
import org.apache.kafka.clients.admin.DescribeClusterResult;
import org.apache.kafka.clients.admin.DescribeTopicsResult;
import org.apache.kafka.clients.admin.ListConsumerGroupsResult;
import org.apache.kafka.clients.admin.ListTopicsResult;
import org.apache.kafka.clients.admin.NewTopic;

public class KafkaAdmin {

    // métodos que vão acessar o Kafka
    public static void list(AdminClient adminClient)
            throws InterruptedException, ExecutionException {
        ListTopicsResult topics = adminClient.listTopics();
        topics.names().get().forEach(System.out::println);
    }

    public static void create(String topicName, int partitions, short replications,
            AdminClient adminClient) {
        final NewTopic newTopic = new NewTopic(topicName, partitions, replications);
        List<NewTopic> topics = new ArrayList<NewTopic>();
        topics.add(newTopic);

        try {
            final CreateTopicsResult result = adminClient.createTopics(topics);
            result.all().get();
        } catch (final Exception e) {
            throw new RuntimeException("Failed to create topic:" + topicName, e);
        }
    }

    public static void describe(String topicName, AdminClient adminClient)
            throws InterruptedException, ExecutionException {
        List<String> topicNames = new ArrayList<>();
        topicNames.add(topicName);
        DescribeTopicsResult topics = adminClient.describeTopics(topicNames);
        topics.all().get().forEach((x, y) ->
            System.out.println(x + " " + y.topicId() + " " + y.partitions()));
    }

    public static void delete(String topicName, AdminClient adminClient) {
        List<String> topicNames = new ArrayList<>();
        topicNames.add(topicName);

        try {
            DeleteTopicsResult topics = adminClient.deleteTopics(topicNames);
            topics.all().get();
        } catch (final Exception e) {
            throw new RuntimeException("Failed to delete topic:" + topicName, e);
        }
    }

    public static void listCG(AdminClient adminClient)
            throws InterruptedException, ExecutionException {
        ListConsumerGroupsResult cgs = adminClient.listConsumerGroups();
        cgs.all().get().forEach(cg -> System.out.println(cg.groupId()));
    }

    public static void deleteCG(String groupId, AdminClient adminClient)
            throws InterruptedException, ExecutionException {
        List<String> groups = new ArrayList<>();
        groups.add(groupId);

        try {
            DeleteConsumerGroupsResult cgs = adminClient.deleteConsumerGroups(groups);
            cgs.all().get();
        } catch (final Exception e) {
            throw new RuntimeException("Failed to delete cg:" + groupId, e);
        }
    }

    public static void describeCluster(AdminClient adminClient)
            throws InterruptedException, ExecutionException {
        DescribeClusterResult cluster = adminClient.describeCluster();
        System.out.println(cluster.clusterId().get());
    }
}
```
