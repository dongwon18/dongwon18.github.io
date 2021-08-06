---
title: "[실습] AWS IoT 디바이스와 연결 모니터링"
excerpt: "AWS IoT reserved topic, LWT 이용해 예상치 못한 연결 종료 메시지 받기"
date: 2021-08-06
categories:
  - AWS_IoT
  - Tutorial
---
안녕하세요. 저번 실습 글에서 디바이스와 통신 상태를 확인하도록 연결이 끊기면 MQTT 메시지를 받도록 구현해보려다 실패했었습니다. 구 박사님(=Google)과 함께 헤매다 보니 구현 성공해서 정리해보겠습니다.

# 목표

- 디바이스와의 통신 상태를 확인한다.
- 만약 디바이스와 연결이 예상치 못하게 끊긴다면 MQTT 메시지로 이를 보고 받는다.

# 사용한 AWS

- AWS IoT Rule

# 전체적인 흐름

1. 이전 글에서 했던 것처럼 정책 생성 하기, 사물 등록하기, 사물 연결하기, 규칙 만들기 등등을 해줍니다.
2. LWT를 설정하여 만약 디바이스의 연결이 예상치 못하게 끊기면 연결이 끊겼다는 메시지를 게시하도록 합니다.
3. LWT 메시지는 AWS IoT Shadow service에서 무시하므로 일반 주제로 게시합니다.
4. 일반 주제로 게시한 LWT 메시지를 rule을 이용하여 시스템 예약된 주제로 재게시 합니다.
5. 게시한 메시지를 확인합니다.
6. 추가로 AWS IoT에서 디바이스 연결을 모니터링하기 위해 제공하는 예약된(reserved topic) 주제를 사용해서 결과를 확인합니다.

# 실습

찾아보니 디바이스와 통신 상태를 확인할 수 있는 시스템 예약된 주제가 AWS IoT에 존재했습니다.(그러면 도대체 왜 나보고 LWT 쓰라고 한 거니...??) 비록 제가 만든 코드보다 훨씬 자세하게 디바이스의 상태를 알려주지만 LWT에 대해 공부할 겸 직접 코드를 작성해보기로 했습니다. 

## 시스템 예약 주제 구독하기

간단하게 확인할 수 있습니다. 제가 동작시켜 봤을 때 connected는 동작하지 않았지만 disconnected는 매우 잘 동작하더라고요.

`$aws/events/presence/connected/+` 와 `$aws/events/presence/disconnected/+` 주제를 구독하면 연결 상태를 보고 받을 수 있습니다. + 부분에는 `client ID` 가 들어갑니다.

1. `AWS IoT 테스트` 에서 위의 두 주제를 구독합니다.
2. 라즈베리파이에서 아래 코드를 실행합니다.
사실 굳이 mqtt_connected.py일 필요는 없습니다. 그냥 AWS IoT에 연결하는 코드이기만 하면 됩니다.

    ```
    cd ~/aws-iot-device-sdk-python-v2/samples
    python3 mqtt_connected.py --endpoint your-end-point --root-ca ~/certs/Amazon-root-CA-1.pem --cert ~/certs/device.pem.crt --key ~/certs/private.pem.key --thingName your-thing-name --client-id your-client-id --topic my/things/RPI/update
    ```

3. MQTT 통신을 하다가 `ctrl + c` 를 눌러 프로그램을 종료시킵니다.
4. 아래처럼 disconnected 주제에서 client ID, timestamp, disconnectReason 등을 확인할 수 있습니다.

<p align = "center">
  <img src = "/assets/images/CONNECTION.PNG" alt = "AWS IoT result"> <br/>
  시스템 예약된 주제로 연결 상태 보고 받기
</p>


cf.) 이 상태에서 다시 프로그램을 실행해봤는데 connected에서 메시지를 확인할 수는 없더라고요. 저 기능을 사용하려면 **프로그램을 실행하고 있는 채로** 통신을 끊는 등의 방식으로 테스트해봐야 할 것 같은데  저는 라즈베리파이를 원격 연결해놔서 통신을 끊으면 연결 자체가 끊기기 때문에 테스트가 힘드네요. 한 번 끊은 통신을 연결하려면 다시 SD 카드를 꺼내서 wifi 다시 설정해야 돼서...ㅎ

## LWT 이용하기

저번 글에서 했던 거와 거의 다른 게 없습니다. 다만 코드에 will 부분을 추가했습니다. will 을 이용해서 예상치 못하게 연결이 끊기면 게시할 메시지를 설정할 수 있습니다. AWS IoT와 메시지를 구독하는 클라이언트에서 게시된 메시지의 내용에 따라 행동을 진행하게 됩니다. 

1. 디바이스와 AWS IoT를 연결한다.(자세한 방법은 이전 글 참고)
2. 아래와 같이 코드를 적습니다. 수정된 부분은 아래 부분 뿐입니다. 
connection을 build 할 때부터 will을 설정해야 되더라고요.
작성한 코드는 라즈베리파이 `~/aws-iot-device-sdk-python-v2/samples` 에 저장합니다.

```python
else:        
        will_payload = json.dumps({"state": {"reported": {"connected":"false"}}})
        will_payload = will_payload.encode()
        print(type(will_payload))
        mqtt_connection = mqtt_connection_builder.mtls_from_path(
            endpoint=args.endpoint,
            port=args.port,
            cert_filepath=args.cert,
            pri_key_filepath=args.key,
            client_bootstrap=client_bootstrap,
            ca_filepath=args.root_ca,
            on_connection_interrupted=on_connection_interrupted,
            on_connection_resumed=on_connection_resumed,
            client_id=args.client_id,
            clean_session=False,
            keep_alive_secs=30,
            will = mqtt.Will(
                topic='my/things/RPI/update',
                qos=mqtt.QoS.AT_LEAST_ONCE,
                payload=will_payload,
                retain = False),
            http_proxy_options=proxy_options)
```

- will_payload: 예상치 못하게 연결이 끊기면 게시할 메시지를 지정합니다. json 메시지를 str 타입으로 변환합니다. will의 payload에 들어갈 내용은 byte type이어야 하기 때문에 encode()로 str를 byte로 변환해줍니다.
- mqtt_connection_builder의 변수 중 will을 위와 같이 설정합니다.
    - topic: 메시지를 게시할 주제(이전 글에서 설정한 주제, 예약된 주제로는 바로 보내지 못하므로 저 주제로 보낸 후 $aws/things/RPI/shadow/update 주제로 재게시합니다.)
    - qos: 규칙을 정할 때 정했던 서비스 품질 부분입니다.
    - payload: 보낼 메시지 내용, json을 byte 타입으로 할당해줘야 합니다.
    - retain: 메시지가 게시됐을 때 LWT 메시지가 저장되고 유지될 것인지 정하는 Boolean 타입 값입니다.
- mtls_from_path는 args를 이용해서 변수를 지정할 때 사용하는 builder인 것 같습니다. 해당 builder에서는 mqtt_connection_builder의 모든 변수를 사용할 수 있습니다.

저 부분을 수정한 전체 코드는 다음과 같습니다.

```python
"""
 file name: my_lwt.py
 """
 import argparse
 from awscrt import io, mqtt, auth, http
 from awsiot import mqtt_connection_builder
 import sys
 import threading
 import time
 from uuid import uuid4
 import json

 """
 # modified part
 	# default change
 		signing-region: ap-northeast-2
 		port: 8883
 	# add argument
 		thingName
 """
 parser = argparse.ArgumentParser(description="Send and receive messages through and MQTT connection.")
 parser.add_argument('--endpoint', required=True, help="Your AWS IoT custom endpoint, not including a port. " +
                                                       "Ex: \"abcd123456wxyz-ats.iot.us-east-1.amazonaws.com\"")
 parser.add_argument('--port', default = 8883, type = int, help="Specify port. AWS IoT supports 443 and 8883.")
 parser.add_argument('--cert', help="File path to your client certificate, in PEM format.")
 parser.add_argument('--key', help="File path to your private key, in PEM format.")
 parser.add_argument('--root-ca', help="File path to root certificate authority, in PEM format. " +
                                       "Necessary if MQTT server uses a certificate that's not already in " +
                                       "your trust store.")
 parser.add_argument('--client-id', default="test-" + str(uuid4()), help="Client ID for MQTT connection.")
 parser.add_argument('--topic', default="test/topic", help="Topic to subscribe to, and publish messages to.")
 parser.add_argument('--count', default=3, type=int, help="Number of messages to publish/receive before exiting. " +
                                                           "Specify 0 to run forever.")
 parser.add_argument('--use-websocket', default=False, action='store_true',
     help="To use a websocket instead of raw mqtt. If you " +
     "specify this option you must specify a region for signing.")
 parser.add_argument('--signing-region', default='ap-northeast-2', help="If you specify --use-web-socket, this " +
     "is the region that will be used for computing the Sigv4 signature")
 parser.add_argument('--proxy-host', help="Hostname of proxy to connect to.")
 parser.add_argument('--proxy-port', type=int, default=8080, help="Port of proxy to connect to.")
 parser.add_argument('--verbosity', choices=[x.name for x in io.LogLevel], default=io.LogLevel.NoLogs.name,
     help='Logging level')
 parser.add_argument('--thingName', help="Thing's name to connect")

 # Using globals to simplify sample code
 args = parser.parse_args()

 io.init_logging(getattr(io.LogLevel, args.verbosity), 'stderr')

 received_count = 0
 received_all_event = threading.Event()

 # Callback when connection is accidentally lost.
 def on_connection_interrupted(connection, error, **kwargs):
     print("Connection interrupted. error: {}".format(error))

 # Callback when an interrupted connection is re-established.
 def on_connection_resumed(connection, return_code, session_present, **kwargs):
     print("Connection resumed. return_code: {} session_present: {}".format(return_code, session_present))

     if return_code == mqtt.ConnectReturnCode.ACCEPTED and not session_present:
         print("Session did not persist. Resubscribing to existing topics...")
         resubscribe_future, _ = connection.resubscribe_existing_topics()

         # Cannot synchronously wait for resubscribe result because we're on the connection's event-loop thread,
         # evaluate result with a callback instead.
         resubscribe_future.add_done_callback(on_resubscribe_complete)

 def on_resubscribe_complete(resubscribe_future):
         resubscribe_results = resubscribe_future.result()
         print("Resubscribe results: {}".format(resubscribe_results))

         for topic, qos in resubscribe_results['topics']:
             if qos is None:
                 sys.exit("Server rejected resubscribe to topic: {}".format(topic))

 # Callback when the subscribed topic receives a message
 def on_message_received(topic, payload, dup, qos, retain, **kwargs):
     print("Received message from topic '{}': {}".format(topic, payload))
     global received_count
     received_count += 1
     if received_count == args.count:
         received_all_event.set()
 """
 get message while running program form cmd
 """
 def get_message():
     message = input("Enter desired value: \n")
     return message

 if __name__ == '__main__':
     # Spin up resources
     event_loop_group = io.EventLoopGroup(1)
     host_resolver = io.DefaultHostResolver(event_loop_group)
     client_bootstrap = io.ClientBootstrap(event_loop_group, host_resolver)

     proxy_options = None
     if (args.proxy_host):
         proxy_options = http.HttpProxyOptions(host_name=args.proxy_host, port=args.proxy_port)

     if args.use_websocket == True:
         credentials_provider = auth.AwsCredentialsProvider.new_default_chain(client_bootstrap)
         mqtt_connection = mqtt_connection_builder.websockets_with_default_aws_signing(
             endpoint=args.endpoint,
             client_bootstrap=client_bootstrap,
             region=args.signing_region,
             credentials_provider=credentials_provider,
             http_proxy_options=proxy_options,
             ca_filepath=args.root_ca,
             on_connection_interrupted=on_connection_interrupted,
             on_connection_resumed=on_connection_resumed,
             client_id=args.client_id,
             clean_session=False,
             keep_alive_secs=30)

     else:
        will_payload = json.dumps({"state": {"reported": {"connected":"false"}}})
        will_payload = will_payload.encode()
        mqtt_connection = mqtt_connection_builder.mtls_from_path(
            endpoint=args.endpoint,
            port=args.port,
            cert_filepath=args.cert,
            pri_key_filepath=args.key,
            client_bootstrap=client_bootstrap,
            ca_filepath=args.root_ca,
            on_connection_interrupted=on_connection_interrupted,
            on_connection_resumed=on_connection_resumed,
            client_id=args.client_id,
            clean_session=False,
            keep_alive_secs=30,
            will = mqtt.Will(
                topic='my/things/RPI/update',
                qos=mqtt.QoS.AT_LEAST_ONCE,
                payload=will_payload,
                retain = False),
            http_proxy_options=proxy_options)

     print("Connecting to {} with client ID '{}'...".format(
         args.endpoint, args.client_id))

     connect_future = mqtt_connection.connect()

     # Future.result() waits until a result is available
     connect_future.result()
     print("Connected!")

     # Subscribe
     print("Subscribing to topic '{}'...".format(args.topic))
     subscribe_future, packet_id = mqtt_connection.subscribe(
         topic=args.topic,
         qos=mqtt.QoS.AT_LEAST_ONCE,
         callback=on_message_received)

     subscribe_result = subscribe_future.result()
     print("Subscribed with {}".format(str(subscribe_result['qos'])))

 		"""
 		# get message from cmd
 		# change message in json form
 		# publish message
 		"""  
     msg = get_message()
     if(len(msg) != 0):
         payload = {"state":{"reported":{"connected": msg}}}
         print("Publishing message to topic '{}': {}".format(args.topic, msg))
         mqtt_connection.publish(
                 topic=args.topic,
                 payload=json.dumps(payload),
                 qos=mqtt.QoS.AT_LEAST_ONCE)
         time.sleep(1)
            

     # Wait for all messages to be received.
     # This waits forever if count was set to 0.
     if args.count != 0 and not received_all_event.is_set():
         print("Waiting for all messages to be received...")

     received_all_event.wait()
     print("{} message(s) received.".format(received_count))

     # Disconnect
     print("Disconnecting...")
     disconnect_future = mqtt_connection.disconnect()
     disconnect_future.result()
     print("Disconnected!")
```

1. AWS IoT에서 이전에 만들었던 `get_RPI_connect_status_rule` 이 활성화 되어 있는지 확인합니다.
2. 라즈베리파이 cmd에서 아래와 같이 실행해 연결합니다.

    ```python
    cd ~/aws-iot-device-sdk-python-v2/samples
    python3 my_lwt.py --endpoint your-end-point --root-ca ~/certs/Amazon-root-CA-1.pem --cert ~/certs/device.pem.crt --key ~/certs/private.pem.key --thingName your-thing-name --client-id your-client-id --topic my/things/RPI/updat
    ```

### 결과

<p align = "center">
  <img src = "/assets/images/CONNECTION5.PNG" alt = "my_lwt.py"> <br/>
  my_lwt.py 실행하고 'trye' MQTT로 보낸 후 프로그램 종료
</p>

1. 결과는 `AWS IoT 테스트` 에서 해당 주제를 구독하여 확인합니다.
2. 위 파일을 실행하면 MQTT 통신을 하면서 메시지를 게시할 수 있습니다. 통신을 하다 `ctrl + c` 로 프로그램을 종료시키면 통신이 끊겼다는 메시지를 받을 수 있습니다.

<p align = "center">
  <img src = "/assets/images/CONNECTION.PNG" alt = "AWS IoT result"> <br/>
  시스템 예약된 주제로 연결 상태 보고 받기
</p>

<p align = "center">
  <img src = "/assets/images/CONNECTION3.PNG" alt = "reserved topic gets messages"> <br/>
  시스템 예약된 주제로 메시지 재게시(rule을 통해)
</p>

<p align = "center">
  <img src = "/assets/images/CONNECTION.PNG" alt = "AWS IoT result"> <br/>
  시스템 예약된 주제로 연결 상태 보고 받기
</p>

3. 예약된 주제로 재게시 했으므로 `사물 -> 디바이스 섀도우 -> 클래식 섀도우` 에서도 확인할 수 있습니다.

<p align = "center">
  <img src = "/assets/images/CONNECTION6.PNG" alt = "AWS IoT Thing shadow"> <br/>
  사물 → 디바이스 섀도우 에서 확인
</p>

# 결론

- 드디어 클라우드에서 디바이스와 연결 상태를 LWT를 이용하여 모니터링할 수 있게 되었습니다.
- LWT를 사용하는 방법을 제외하고 AWS IoT 예약 주제로도 모니터링이 가능합니다.
- 프로젝트를 진행할 때는 두 가지 방법 중 편의에 따라 사용할 예정이며 연결이 끊기면 해당 내용을 관리자에게 메일로 보내 적절한 조치를 취하도록 Lambda 함수와 Rule을 사용해 설정해놓으면 좋을 듯 합니다.

# 참고

- [AWS IoT Developer Guide MQTT Reserved topics](https://docs.aws.amazon.com/iot/latest/developerguide/reserved-topics.html#reserved-topics-event)
AWS IoT에 시스템 예약된 주제를 모아 놓은 페이지입니다. 이번에 확인 안 하고 헤맨 시간이 길어서 이제 기본적인 기능을 일단 이 페이지를 찾아보고 없으면 구 박사님을 뒤져봐야 할 거 같습니다.
- [aws-crt-python will API Document](https://awslabs.github.io/aws-crt-python/api/mqtt.html#awscrt.mqtt.Will)
awscrt.mqtt.Will 클래스 및 QoS 클래스 사용법에 대해 적혀 있어 코드 작성할 때 참고했습니다.
- [aws-iot-device-sdk-python-v2 API Document](https://aws.github.io/aws-iot-device-sdk-python-v2/awsiot/mqtt_connection_builder.html)
awsiot.mqtt_connection_builder의 변수를 확인할 수 있습니다. 역히 코드 작성할 때 참고했습니다.
- 이전 글: [RPI에서 AWS IoT로 값 보내기](https://dongwon18.github.io/aws_iot/tutorial/Send-Message-from-Device-and-Republish)
