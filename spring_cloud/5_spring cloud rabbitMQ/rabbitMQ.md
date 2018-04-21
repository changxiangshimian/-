### 在接受端和发送端的pom文件里面分别加入依赖

```xml
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-bus-amqp</artifactId>
</dependency>
```

### 方法一


* rabbitMQ发送端
  ```java
  @Controller
  public class TestClient {
        //注入rabbitMQ模板
        @Autowired
        private AmqpTemplate amqpTemplate;

        /**
         * rabbitMQ发送端
         * @return
         */
        @RequestMapping("/send")
        @ResponseBody
        public String sendMSG(){
            //定义消息队列的名称与消息的内容
            amqpTemplate.convertAndSend("TestName","时间 : "+new Date());
            return "ok";
        }
  }
  ```

* rabbitMQ接收端

  ```java
  /**
   * rabbitMQ接收端
   */
  @Component
  public class MqReceiver {
      //接受并创建(如果该消息队列不存在)消息
      @RabbitListener(queuesToDeclare = @Queue("TestName"))
      public void process(String message){
          System.out.println(message);
      }
  }
  ```

### 方法二`接收端与发送端进行绑定(只向指定的接受方发送消息)`


* rabbitMQ发送端

  ```java
  @Controller
  public class TestClient {
        //注入rabbitMQ模板
        @Autowired
        private AmqpTemplate amqpTemplate;

        /**
         * rabbitMQ发送端
         * @return
         */
        @RequestMapping("/sendMonkey")
        @ResponseBody
        public String sendMonkey(){
            /**
             * animal : exchange
             * monkey : key
             * "时间 : "+new Date() : message
             */
            amqpTemplate.convertAndSend("animal","monkey","时间 : "+new Date());
            return "ok";
        }
  }
  ```

* rabbitMQ接收端

  ```java
  /**
   * rabbitMQ接收端
   */
  @Component
  public class MqReceiver {
        //接受并创建
        @RabbitListener(bindings = @QueueBinding(
              exchange = @Exchange("animal"),
              key = "monkey",
              value = @Queue("MaleMonkey")
        ))
        public void getMonkey(String message){
          System.out.println("Monkey -- "+message);
        }
  }
  ```

### 通过spring cloud stream操作消息队列

* 加入依赖
  ```xml
  <dependency>
  	<groupId>org.springframework.cloud</groupId>
  	<artifactId>spring-cloud-starter-stream-rabbit</artifactId>
  </dependency>
  ```

* 定义一个接口

  ```java
  @Component
  public interface StreamClient {
      //接受消息 BY 消息名
      @Input("resultMessage")
      MessageChannel resultMessage();

      //发送消息 BY 消息名
      @Output("myMessage")
      MessageChannel myMessage();

  }
```

* 发送消息

  ```java
  @RestController
  public class SendMessageController {

      //注入接口
      @Autowired
      private StreamClient streamClient;

      @GetMapping("/sendMsg")
      public void process(){
          streamClient.myMessage().send(MessageBuilder.withPayload("时间 : "+new Date()).build());
      }
  }
  ```

* 接受消息

  ```java
  @Component
  //写入刚刚定义的接口
  @EnableBinding(StreamClient.class)
  public class StreamReceiver {

      @StreamListener("myMessage")
      //接受消息成功后发送到该名称的队列消息
      @SendTo("resultMessage")
      public String process(String message){
          System.out.println(message.toString() );
          //返回的参数发送到resultMessage消息中
          return "result getMessage ok";
      }

      //接受上面返回的消息
      @StreamListener("resultMessage")
      public void process2(String message){
          //打印结果
          System.out.println(message.toString() );
      }
  }
  ```

* 在项目配置文件中配置消息规则

```yml
spring:
  cloud:
    stream:
      bindings:
        #消息分组,防止模块部署多个,消息往每个模块都发送消息,只需往一个模块下发送消息就可以
        #消息名为 myMessage
        myMessage:
          #分组的名称没有特定取名要求
          group: msg
          #发生消息堆积在rabbitMQ控制页面以json的参数展示
          content-type: application/json
        #消息名为 resultMessage
        resultMessage:
          #分组的名称没有特定取名要求
          group: msg
          #发生消息堆积在rabbitMQ控制页面以json的参数展示
          content-type: application/json
```
