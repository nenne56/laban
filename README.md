# laban
My test lab

I have just started to test the moquette broker. (I'm new to GitHub as well)

I have written two java programs `MyPublisher` and `MySubscriber` that cause a NullPointerException in the broker when running on Win 7 x64.

Here is the sequence:
1. Start moquette
2. Start MySubscriber
3. Run MyPublisher sending a message (using QoS=0) that successfully arrives to MySubscriber
4. Stop MySubscriber
5. Run MyPublisher sending a second message
6. Start MySubscriber again. 
7. The broker get the `NullPointerException` directly after it issues "ProtocolProcessor  - republishing stored messages to client \<MySubscriber\>"

####MySubscriber.java
```java
package lny.mqttlab;
import org.eclipse.paho.client.mqttv3.MqttClient;
import org.eclipse.paho.client.mqttv3.MqttConnectOptions;
import org.eclipse.paho.client.mqttv3.MqttException;
import org.eclipse.paho.client.mqttv3.MqttMessage;
import org.eclipse.paho.client.mqttv3.persist.MemoryPersistence;
public class MyPublisher {
   public static void main( String[] args ) {
   if ( args.length > 0 ) { 
   String content = args[0];
   int qos = 0;
   String topic = "test";
   String broker = "tcp://localhost:1883";
   String clientId = "MyPublisher";
   try {
         MqttClient client = new MqttClient(broker, clientId, new MemoryPersistence());
         MqttConnectOptions connOpts = new MqttConnectOptions();
         connOpts.setCleanSession(false); 
         client.connect(connOpts);
         System.out.println("publishing message: \"" + content + "\"");
         System.out.println("qos: " + qos + "  topic: " + topic);
         MqttMessage message = new MqttMessage(content.getBytes());
         message.setQos(qos); 
         client.publish(topic, message); 
         client.disconnect(); 
    } catch (MqttException me) {   me.printStackTrace();  } 
    }
    else
    { 
    System.out.println("\nUsage: MyPublisher message \n");
    } 
    }
    }
```

hejs
