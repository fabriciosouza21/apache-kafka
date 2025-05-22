# Rententativas

O Kafka tem um mecanismo de retentativa, quando acontece um error não tratador, o kafka faz a retentativa até 10 vezes. No entando o Recurso do Consumidor fica travado até que o numero de tentativas seja atingido.

para evitar o travemento no caso do java podemos usar o **try cach** para não haver error, assim para o kafka não haverá error.

Uma estrategia de retentativa é criar um novo tópico para as mensagens que falharam(Fila Morta  - dead letter), assim podemos tratar essas mensagens depois.

> Importante nunca esquece de mapear bem o error o try cach para não perder mensagens importantes.
