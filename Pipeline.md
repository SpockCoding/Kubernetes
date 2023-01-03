O que é Pipeline?. Pipeline é um "passo a passo" no processo de desenvolvimento. A última etapa de uma pipeline se dá 
onde se inicia, é como se fosse um pipeline=encanamento. Um exemplo prático seria iniciar em uma API que coleta os dados
e os envia para o Apache Kafka. O Kafka por sua vez armazena os dados tal como um buffer e depois envia para análise
mais profunda dos dados por meio do Apache Spark, onde finalmente a pipeline chega ao "fim" com esses dados já tratados
sendo armazenados na base de dados. Então, a pipeline volta para a API e assim por diante.