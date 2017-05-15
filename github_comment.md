
## NettyMQTTHandler  - Bad error in processing the message


Hi,

I'm new to MQTT and have started to investigate moquette and eclipse paho. (I'm new to GitHub as well)

I have written two java programs `MyPublisher` and `MySubscriber` that cause a NullPointerException in the broker when running on Win 7 x64.

Here is the sequence:
1. Start moquette
2. Start MySubscriber
3. Run MyPublisher sending a message that successfully arrives to MySubscriber
4. Run MyPublisher sending a "stop" message that will cause MySubscriber to exit
5. Run MyPublisher sending a new message
6. Start MySubscriber again. 
7. The broker get the `NullPointerException` directly after it issues "ProtocolProcessor - republishing stored messages to client \<MySubscriber\>"

Am I 
#### MySubscriber.java
```java
package lny.mqttlab;

import org.eclipse.paho.client.mqttv3.IMqttDeliveryToken;
import org.eclipse.paho.client.mqttv3.MqttCallback;
import org.eclipse.paho.client.mqttv3.MqttClient;
import org.eclipse.paho.client.mqttv3.MqttConnectOptions;
import org.eclipse.paho.client.mqttv3.MqttException;
import org.eclipse.paho.client.mqttv3.MqttMessage;
import org.eclipse.paho.client.mqttv3.persist.MemoryPersistence;

public class MySubscriber implements MqttCallback
{
   private MqttClient client;
   boolean keepRunning = true;

   public MySubscriber()
   {
   }

   public static void main( String[] args )
   {
      new MySubscriber().doSubscribe();
   }

   public void doSubscribe()
   {
      String broker = "tcp://localhost:1883";
      String clientId = "MySubscriber";
      String topic = "test";
      try
      {
         client = new MqttClient( broker, clientId, new MemoryPersistence() );
         MqttConnectOptions connOpts = new MqttConnectOptions();
         connOpts.setCleanSession(false);
         client.connect( connOpts );
         client.setCallback( this );
         System.out.println( "subscribing to topic: " + topic );
         client.subscribe( topic );

         while ( keepRunning )
         {
            Thread.sleep( 1000 );
         }
         client.disconnect();
         System.out.println( "--- MySubscriber stopped ---" );

      }
      catch ( MqttException e )
      {
         e.printStackTrace();
      }
      catch ( InterruptedException dontCare )
      {
      }
   }

   @Override
   public void connectionLost( Throwable cause )
   {
      System.out.println( "connectionLost: " + cause.getMessage() );
   }

   @Override
   public void messageArrived( String topic, MqttMessage message ) throws Exception
   {
      System.out.println( "Message arrived: \"" + message + "\"" );
      if ( message.toString().equalsIgnoreCase( "stop" ) )
      {
         keepRunning = false;
      }
   }

   @Override
   public void deliveryComplete( IMqttDeliveryToken token )
   {
      System.out.println( "deliveryComplete: " + token.toString() );
   }

}
```

#### MyPublisher.java
```java
package lny.mqttlab;

import org.eclipse.paho.client.mqttv3.MqttClient;
import org.eclipse.paho.client.mqttv3.MqttConnectOptions;
import org.eclipse.paho.client.mqttv3.MqttException;
import org.eclipse.paho.client.mqttv3.MqttMessage;
import org.eclipse.paho.client.mqttv3.persist.MemoryPersistence;

public class MyPublisher
{

   public static void main( String[] args )
   {
      if ( args.length > 0 )
      {
         String content = args[ 0 ];
         int qos = 0;
         String topic = "test";
         String broker = "tcp://localhost:1883";
         String clientId = "MyPublisher";

         try
         {
            MqttClient client = new MqttClient( broker, clientId, new MemoryPersistence() );
            MqttConnectOptions connOpts = new MqttConnectOptions();
            client.connect( connOpts );
            System.out.println( "publishing message: \"" + content + "\"" );
            System.out.println( "qos: " + qos + "  topic: " + topic );
            MqttMessage message = new MqttMessage( content.getBytes() );
            message.setQos( qos );
            client.publish( topic, message );
            client.disconnect();
         }
         catch ( MqttException me )
         {
            me.printStackTrace();
         }
      }
      else
      {
         System.out.println( "\nUsage: MyPublisher message \n" );
      }
   }

}
```

#### Output from moquette
```
"                                                                         "
"  ___  ___                       _   _        ___  ________ _____ _____  "
"  |  \/  |                      | | | |       |  \/  |  _  |_   _|_   _| "
"  | .  . | ___   __ _ _   _  ___| |_| |_ ___  | .  . | | | | | |   | |   "
"  | |\/| |/ _ \ / _\ | | | |/ _ \ __| __/ _ \ | |\/| | | | | | |   | |   "
"  | |  | | (_) | (_| | |_| |  __/ |_| ||  __/ | |  | \ \/' / | |   | |   "
"  \_|  |_/\___/ \__, |\__,_|\___|\__|\__\___| \_|  |_/\_/\_\ \_/   \_/   "
"                   | |                                                   "
"                   |_|                                                   "
"                                                                         "
Using JAVA_HOME:       "C:\Program Files (x86)\Java\jdk1.7.0_60"
Using MOQUETTE_HOME:   "C:\spp\lny_mqttlab\moquette\distribution-0.9-bundle"
0    [main] INFO  Server  - Persistent store file: null
137  [main] INFO  ResourceAuthenticator  - Loading password file config/password_file.conf
815  [main] INFO  NettyAcceptor  - Server binded host: 0.0.0.0, port: 1883
817  [main] INFO  NettyAcceptor  - Server binded host: 0.0.0.0, port: 8080
Server started, version 0.9
10893 [nioEventLoopGroup-3-1] INFO  ProtocolProcessor  - CONNECT for client <MySubscriber>
10917 [nioEventLoopGroup-3-1] INFO  ProtocolProcessor  - No stored messages for client <MySubscriber>
10918 [nioEventLoopGroup-3-1] INFO  ProtocolProcessor  - Connection established
10919 [nioEventLoopGroup-3-1] INFO  ProtocolProcessor  - CONNECT processed
10919 [nioEventLoopGroup-3-1] INFO  ProtocolProcessor  - SUBSCRIBE client <MySubscriber>
10922 [nioEventLoopGroup-3-1] INFO  ClientSession  - <MySubscriber> subscribed to the topic filter <test> with QoS 1 - LEAST_ONE
35687 [nioEventLoopGroup-3-2] INFO  ProtocolProcessor  - CONNECT for client <MyPublisher>
35688 [nioEventLoopGroup-3-2] INFO  ClientSession  - cleaning old saved subscriptions for client <MyPublisher>
35693 [nioEventLoopGroup-3-2] INFO  ProtocolProcessor  - Connection established
35693 [nioEventLoopGroup-3-2] INFO  ProtocolProcessor  - CONNECT processed
35695 [nioEventLoopGroup-3-2] INFO  ProtocolProcessor  - PUB --PUBLISH--> SRV executePublish invoked with io.moquette.parser.proto.messages.PublishMessage@189471a
35697 [nioEventLoopGroup-3-2] INFO  PersistentQueueMessageSender  - send publish message to <MySubscriber> on topic <test>
35699 [nioEventLoopGroup-3-2] INFO  ProtocolProcessor  - cleaning old saved subscriptions for client <MyPublisher>
35700 [nioEventLoopGroup-3-2] INFO  ProtocolProcessor  - DISCONNECT client <MyPublisher> finished
46595 [nioEventLoopGroup-3-3] INFO  ProtocolProcessor  - CONNECT for client <MyPublisher>
46596 [nioEventLoopGroup-3-3] INFO  ClientSession  - cleaning old saved subscriptions for client <MyPublisher>
46597 [nioEventLoopGroup-3-3] INFO  ProtocolProcessor  - Connection established
46599 [nioEventLoopGroup-3-3] INFO  ProtocolProcessor  - CONNECT processed
46604 [nioEventLoopGroup-3-3] INFO  ProtocolProcessor  - PUB --PUBLISH--> SRV executePublish invoked with io.moquette.parser.proto.messages.PublishMessage@2bb129
46604 [nioEventLoopGroup-3-3] INFO  PersistentQueueMessageSender  - send publish message to <MySubscriber> on topic <test>
46606 [nioEventLoopGroup-3-3] INFO  ProtocolProcessor  - cleaning old saved subscriptions for client <MyPublisher>
46606 [nioEventLoopGroup-3-3] INFO  ProtocolProcessor  - DISCONNECT client <MyPublisher> finished
46932 [nioEventLoopGroup-3-1] INFO  ProtocolProcessor  - DISCONNECT client <MySubscriber> finished
52423 [nioEventLoopGroup-3-4] INFO  ProtocolProcessor  - CONNECT for client <MyPublisher>
52423 [nioEventLoopGroup-3-4] INFO  ClientSession  - cleaning old saved subscriptions for client <MyPublisher>
52425 [nioEventLoopGroup-3-4] INFO  ProtocolProcessor  - Connection established
52428 [nioEventLoopGroup-3-4] INFO  ProtocolProcessor  - CONNECT processed
52433 [nioEventLoopGroup-3-4] INFO  ProtocolProcessor  - PUB --PUBLISH--> SRV executePublish invoked with io.moquette.parser.proto.messages.PublishMessage@992f73
52436 [nioEventLoopGroup-3-4] INFO  ProtocolProcessor  - cleaning old saved subscriptions for client <MyPublisher>
52437 [nioEventLoopGroup-3-4] INFO  ProtocolProcessor  - DISCONNECT client <MyPublisher> finished
57293 [nioEventLoopGroup-3-5] INFO  ProtocolProcessor  - CONNECT for client <MySubscriber>
57294 [nioEventLoopGroup-3-5] INFO  ProtocolProcessor  - republishing stored messages to client <MySubscriber>
57295 [nioEventLoopGroup-3-5] ERROR NettyMQTTHandler  - Bad error in processing the message
java.lang.NullPointerException
        at io.moquette.spi.impl.InternalRepublisher.publishStored(InternalRepublisher.java:52)
        at io.moquette.spi.impl.ProtocolProcessor.republishStoredInSession(ProtocolProcessor.java:415)
        at io.moquette.spi.impl.ProtocolProcessor.republish(ProtocolProcessor.java:373)
        at io.moquette.spi.impl.ProtocolProcessor.processConnect(ProtocolProcessor.java:264)
        at io.moquette.server.netty.NettyMQTTHandler.channelRead(NettyMQTTHandler.java:54)
        at io.netty.channel.AbstractChannelHandlerContext.invokeChannelRead(AbstractChannelHandlerContext.java:373)
        at io.netty.channel.AbstractChannelHandlerContext.invokeChannelRead(AbstractChannelHandlerContext.java:359)
        at io.netty.channel.AbstractChannelHandlerContext.fireChannelRead(AbstractChannelHandlerContext.java:351)
        at io.moquette.server.netty.metrics.MQTTMessageLogger.channelRead(MQTTMessageLogger.java:41)
        at io.netty.channel.AbstractChannelHandlerContext.invokeChannelRead(AbstractChannelHandlerContext.java:373)
        at io.netty.channel.AbstractChannelHandlerContext.invokeChannelRead(AbstractChannelHandlerContext.java:359)
        at io.netty.channel.AbstractChannelHandlerContext.fireChannelRead(AbstractChannelHandlerContext.java:351)
        at io.moquette.server.netty.metrics.MessageMetricsHandler.channelRead(MessageMetricsHandler.java:46)
        at io.netty.channel.AbstractChannelHandlerContext.invokeChannelRead(AbstractChannelHandlerContext.java:373)
        at io.netty.channel.AbstractChannelHandlerContext.invokeChannelRead(AbstractChannelHandlerContext.java:359)
        at io.netty.channel.AbstractChannelHandlerContext.fireChannelRead(AbstractChannelHandlerContext.java:351)
        at io.netty.handler.codec.ByteToMessageDecoder.fireChannelRead(ByteToMessageDecoder.java:293)
        at io.netty.handler.codec.ByteToMessageDecoder.channelRead(ByteToMessageDecoder.java:267)
        at io.netty.channel.AbstractChannelHandlerContext.invokeChannelRead(AbstractChannelHandlerContext.java:373)
        at io.netty.channel.AbstractChannelHandlerContext.invokeChannelRead(AbstractChannelHandlerContext.java:359)
        at io.netty.channel.AbstractChannelHandlerContext.fireChannelRead(AbstractChannelHandlerContext.java:351)
        at io.netty.channel.ChannelInboundHandlerAdapter.channelRead(ChannelInboundHandlerAdapter.java:86)
        at io.netty.channel.AbstractChannelHandlerContext.invokeChannelRead(AbstractChannelHandlerContext.java:373)
        at io.netty.channel.AbstractChannelHandlerContext.invokeChannelRead(AbstractChannelHandlerContext.java:359)
        at io.netty.channel.AbstractChannelHandlerContext.fireChannelRead(AbstractChannelHandlerContext.java:351)
        at io.netty.handler.timeout.IdleStateHandler.channelRead(IdleStateHandler.java:266)
        at io.netty.channel.AbstractChannelHandlerContext.invokeChannelRead(AbstractChannelHandlerContext.java:373)
        at io.netty.channel.AbstractChannelHandlerContext.invokeChannelRead(AbstractChannelHandlerContext.java:359)
        at io.netty.channel.AbstractChannelHandlerContext.fireChannelRead(AbstractChannelHandlerContext.java:351)
        at io.moquette.server.netty.metrics.BytesMetricsHandler.channelRead(BytesMetricsHandler.java:47)
        at io.netty.channel.AbstractChannelHandlerContext.invokeChannelRead(AbstractChannelHandlerContext.java:373)
        at io.netty.channel.AbstractChannelHandlerContext.invokeChannelRead(AbstractChannelHandlerContext.java:359)
        at io.netty.channel.AbstractChannelHandlerContext.fireChannelRead(AbstractChannelHandlerContext.java:351)
        at io.netty.channel.DefaultChannelPipeline$HeadContext.channelRead(DefaultChannelPipeline.java:1334)
        at io.netty.channel.AbstractChannelHandlerContext.invokeChannelRead(AbstractChannelHandlerContext.java:373)
        at io.netty.channel.AbstractChannelHandlerContext.invokeChannelRead(AbstractChannelHandlerContext.java:359)
        at io.netty.channel.DefaultChannelPipeline.fireChannelRead(DefaultChannelPipeline.java:926)
        at io.netty.channel.nio.AbstractNioByteChannel$NioByteUnsafe.read(AbstractNioByteChannel.java:129)
        at io.netty.channel.nio.NioEventLoop.processSelectedKey(NioEventLoop.java:651)
        at io.netty.channel.nio.NioEventLoop.processSelectedKeysOptimized(NioEventLoop.java:574)
        at io.netty.channel.nio.NioEventLoop.processSelectedKeys(NioEventLoop.java:488)
        at io.netty.channel.nio.NioEventLoop.run(NioEventLoop.java:450)
        at io.netty.util.concurrent.SingleThreadEventExecutor$5.run(SingleThreadEventExecutor.java:873)
        at io.netty.util.concurrent.DefaultThreadFactory$DefaultRunnableDecorator.run(DefaultThreadFactory.java:144)
        at java.lang.Thread.run(Thread.java:745)
57315 [nioEventLoopGroup-3-5] INFO  ProtocolProcessor  - SUBSCRIBE client <MySubscriber>
57316 [nioEventLoopGroup-3-5] INFO  ClientSession  - <MySubscriber> subscribed to the topic filter <test> with QoS 1 - LEAST_ONE
```
