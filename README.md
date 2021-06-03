## Desenvolvimento de aplicativos MQ JMS com Spring Boot

# 1. Introdução

O IBM MQ tem um Spring Boot Starter que fornece aos desenvolvedores do Spring uma maneira fácil de configurar o pacote IBM MQ JMS.

O MQ permite que os aplicativos se comuniquem e compartilhem dados entre si de uma forma confiável e escalonável que separa um aplicativo do outro. Dessa forma, auxilia na integração de aplicativos executados em diferentes frameworks, linguagens, plataformas, nuvens e locais.

Use o MQ Spring JMS Starter para acessar um servidor IBM MQ a partir de um aplicativo Spring Boot

# 2. Objetivos de aprendizado
Este tutorial mostra como usar o MQ Spring JMS Starter para acessar um servidor IBM MQ a partir de um aplicativo Spring Boot. Este exemplo usa uma instância local do MQ em execução em um contêiner Docker. Você também pode usar um servidor MQ no IBM Cloud. O aplicativo inclui um exemplo de par de terminais REST por meio dos quais as mensagens podem ser enviadas e recuperadas do MQ.

# 3. Você executa as seguintes etapas:

- Criar uma aplicação Spring Boot usando o Spring Initializr;
- Inicie um servidor MQ local usando Docker;
- Adicione a configuração do servidor MQ (credenciais e URL) ao seu aplicativo;
- Adicione o MQ Spring Starter ao seu aplicativo;
- Adicionar um endpoint REST que envia uma mensagem;
- Adicionar um endpoint REST que recupera mensagens;
- Construa o aplicativo e chame o terminal REST e exiba os resultados do MQ.

# 4. Como usar

### 4.1 Etapa 01
Crie um aplicativo Spring Boot usando o Spring Initializr
Na página Spring Initializr gere um Projeto Maven com a linguagem Java e a dependência Web. Para este exemplo, usamos o grupo com.isaccanedo e o artefato mq. Baixe o projeto e descompacte-o.

### 4.2 Etapa 02
Inicie um servidor MQ local usando Docker
O contêiner IBM MQ for Developers fornece uma maneira rápida e simples de ativar um servidor MQ local via Docker. Você pode iniciar um usando o seguinte comando:

```
docker run ‑‑env LICENSE=accept ‑‑env MQ_QMGR_NAME=QM1
           ‑‑publish 1414:1414
           ‑‑publish 9443:9443
           ‑‑detach
           ibmcom/mq
```

Verifique se o servidor está sendo executado usando docker ps:

```
$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                                            NAMES
3a225c721428        ibmcom/mq           "runmqdevserver"    4 hours ago        Up 4 hours         0.0.0.0:1414‑>1414/tcp, 0.0.0.0:9443‑>9443/tcp   reverent_bartik
```

### 4.3 Etapa 04
Adicione a configuração do servidor MQ (credenciais e URL) ao seu aplicativo
A configuração padrão para o MQ Server local inclui um usuário admin com uma senha passw0rd. Isso pode ser passado para o aplicativo por meio do arquivo normal do Spring application.properties.

Edite o projeto Spring Initializr descompactado e adicione as informações do servidor ao arquivo src/main/resources/application.properties com os seguintes nomes e valores de propriedade:

```
ibm.mq.queueManager=QM1
ibm.mq.channel=DEV.ADMIN.SVRCONN
ibm.mq.connName=localhost(1414)
ibm.mq.user=admin
ibm.mq.password=passw0rd
```

### 4.4 Etapa 05
Adicione o MQ Spring Starter ao seu aplicativo
Para este exemplo, criaremos um aplicativo REST simples com um terminal que envia uma mensagem por meio do servidor MQ e um segundo terminal que recupera e retorna as mensagens enviadas.

Edite o projeto Spring Boot descompactado para fazer as seguintes alterações:

1 - Adicione as seguintes dependências à seção de dependências pom.xml:

```
 <dependency>
         <groupId>com.fasterxml.jackson.core</groupId>
         <artifactId>jackson-databind</artifactId>
     </dependency>
     <dependency>
         <groupId>com.ibm.mq</groupId>
         <artifactId>mq-jms-spring-boot-starter</artifactId>
         <version>2.0.0</version>
 </dependency>
```

2 - Adicione anotações à classe de aplicativo Spring Boot que foi criada por Spring Initializr - com/isaccanedo/mq/MqApplication.java. (Observe que o pacote Java e os nomes das classes são derivados dos valores de Grupo e Artefato inseridos no Initializr.)

```
@RestController to enable REST endpoints.
@EnableJms to allow discovery of methods annotated @JmsListener
```

3 - Crie a classe queueController.java e adicione uma anotação @Autowired para o objeto JmsTemplate. O IBM MQ Spring Boot Starter cria o JmsTemplate com as propriedades configuradas por meio de application.properties:

4 - Adicione um endpoint REST que envia uma mensagem via MQ
Adicione um ponto de extremidade REST ao com a anotação @GetMapping com um caminho de envio. Use o método JmsTemplate convertAndSend para enviar uma mensagem Hello World! para a fila DEV.QUEUE.1. Adicione tratamento de exceções conforme necessário.

5 - Adicionar um endpoint REST que recupera mensagens via MQ
Adicione um ponto de extremidade REST com a anotação @GetMapping com um caminho de recv. Use o método JmsTemplate receiveAndConvert para receber uma mensagem da fila DEV.QUEUE.1. Adicione tratamento de exceções conforme necessário.

```
package com.isaccanedo.controller;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jms.JmsException;
import org.springframework.jms.annotation.EnableJms;
import org.springframework.jms.core.JmsTemplate;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@EnableJms

public class queueController {
	@Autowired
	private JmsTemplate jmsTemplate;

	@GetMapping("/send")
	String send() {
		try {
			jmsTemplate.convertAndSend("DEV.QUEUE.1", "Hello World!");
			return "OK";
		} catch (JmsException ex) {
			ex.printStackTrace();
			return "FAIL";
		}
	}

	@GetMapping("/recv")
	String recv() {
		try {
			return jmsTemplate.receiveAndConvert("DEV.QUEUE.1").toString();
		} catch (JmsException ex) {
			ex.printStackTrace();
			return "FAIL";
		}
	}

}
```

```
Nota: A fila DEV.QUEUE.1 é pré-criada pelo contêiner IBM MQ for Developers. O método é breve para este tutorial, um aplicativo real provavelmente teria um melhor tratamento de exceções e pode usar objetos digitados como cargas úteis de mensagens. Consulte o guia Spring para Mensagens com JMS para obter mais informações.
```

### 4.5 - Etapa 06
Construir o aplicativo e invocar os endpoints REST e exibir os resultados
Crie e execute seu aplicativo com o seguinte comando:

```
mvn package spring-boot:run
```

Agora você pode invocar o ponto de extremidade REST para envio, ```http://localhost:8080/send.``` Você deve ver a resposta OK de seu endpoint, confirmando que a mensagem foi enviada.

Depois de enviar uma mensagem, você pode chamar o ponto de extremidade REST para recebimento, ```http://localhost:8080/recv.``` Você deverá ver a resposta do endpoint com o conteúdo da mensagem ```Hello World!```

# 5. Conclusão
O IBM MQ Spring Starter torna mais fácil enviar e receber mensagens de um serviço MQ usando a API JmsTemplate do Spring, com configuração automática do Spring.


