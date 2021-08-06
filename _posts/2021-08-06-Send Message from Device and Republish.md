---
title: "[실습] RPI에서 AWS IoT로 값 보내기(MQTT 통신, 시스템 예약 주제로 재게시)"
excerpt: "RPI에서 MQTT 통신을 통해 메시지 게시 및 AWS IoT 시스템 예약 주제로 재게시하여 shadow에서 확인하기"
date: 2021-08-06
categories:
  - AWS_IoT
  - Tutorial
---
안녕하세요. 이전까지 AWS IoT 자습서에서 제공하는 예제를 따라하는 **튜토리얼** 이었다면 지금부터는 제가 AWS IoT를 이용해서 라즈베리파이로 테스트베드를 만드는데 필요한 부분을 **직접 실습**해볼 예정입니다.   
저도 하면서 배운 거라 헤맸던 부분 정리 및 나중에 제가 필요할 때마다 꺼내 볼 느낌으로 진행하려고 합니다.

# 목표

- 라즈베리파이에서 아두이노로부터 시리얼 통신으로 받아온 값을 AWS IoT에 MQTT 프로토콜을 이용하여 전송한다.
- 디바이스 연결 상태를 AWS IoT에서 보고 받는다.

    > 디바이스가 에러에 의해서 연결이 끊긴다면 정상적으로 `connected` 값을 false로 설정할 수 없습니다. 이럴 경우에는 MQTT Last Will and Testament(LWT)를 이용해서 `connected`의 값을 false로 설정해줘야 한다고 [AWS IoT Core Developer Guide](https://docs.aws.amazon.com/iot/latest/developerguide/device-shadow-comms-app.html#thing-connection)에 명시되어 있습니다. 그러나 AWS IoT Shadow는 LWT로 보낸 메시지는 무시한다고 합니다. 따라서 다른 `topic`으로 LWT 메시지를 받아서 다시 `/shadow/update` 주제로 재게시 하는 방법을 사용하라고 적혀 있어 이 부분을 실습해 볼 예정입니다.

    [좋은 클라우드 IoT 환경 조성](https://dongwon18.github.io/aws_iot/theory/Well-Architected-Iot-Framework/) 글에서도 다뤘듯이 디바이스가 현재 연결 상태인지 확인하는 것은 매우 중요한 일입니다. 따라서 저도 이 기능을 사용해보고자 했습니다. 
    결론부터 말씀드리자면 아직 구현 못했습니다. 디바이스가 꺼지거나 연결이 끊길 때 LWT를 보내야 하는데 그 부분을 어떻게 구현해야 하는지 찾지 못했거든요...

# 사용된 AWS

- AWS IoT Rule(재게시 규칙)
- AWS IAM(role 생성 및 사용)

# 전체적인 흐름

1. MQTT 통신에 사용할 정책을 결정합니다.
2. 사물을 등록하고 인증서를 생성하여 라즈베리파이를 AWS IoT와 연결합니다.
3. MQTT 통신 코드를 작성합니다.
4. MQTT 통신 코드를 라즈베리파이에서 실행하고 AWS IoT 콘솔 테스트 창에서 해당 주제를 구독하여 결과를 확인합니다.
5. 라즈베리파이에서는 `my/things/RPI/update` 라는 시스템 예약이 아닌 일반 주제로 메시지를 게시합니다.
6. AWS IoT에서 메시지를 재게시하는 Rule을 통해 `$aws/things/RPI/shadow/update` 에 메시지를 재게시합니다. 이 주제는 시스템 예약 주제라서 이 주제로 게시된 메시지는 `사물 -> 디바이스 섀도우 -> 클래식 섀도우` 에서도 확인할 수 있습니다.

# 실습

이번 글부터는 기본적인 단계는 생략해 정리하겠습니다. 어떻게 하는지 모르는 부분이 나왔다면 이전 AWS IoT 튜토리얼 시리즈 글들을 보고 와주세요.

1. 새로운 규칙을 생성한다.
    - 사물 이름: RPI
    - 이름: get_RPI_connect_status_rule

    ```sql
    SELECT * FROM 'my/things/RPI/update'
    ```

    - 작업: 메시지를 AWS IoT 주제에 재게시
    `$$aws/things/RPI/shadow/update` 
    **꼭 $를 2개 붙이세요. 저는 1개 붙이고 작동 안 돼서 한참을 헤맸습니다**.
    - role: republish_connected(새로 아무거나 생성)
    - 규칙을 활성화한다.

    `my/things/RPI/update` 라는 주제로 받은 MQTT 메시지를 시스템에 예약된 주제인 `$aws/things/RPI/shadow/update` 에 재게시합니다. 여기서 LWT는 `my/things/RPI/update` 로 게시되는 메시지입니다. (그러나 LWT 설정을 안 해서 아직 연결을 끊어도 동작 안 합니다.)

2. 정한 주제에 메시지를 게시하는 코드를 작성한다.
    - 코드를 작성하는데 참고할 만한 예제가 검색하니 잘 안 나와서 AWS IoT 자습서 및 SDK에서 제공하는 예제를 참고했습니다.
        - [pubsub.py](https://github.com/aws/aws-iot-device-sdk-python-v2/blob/6a6e6344fd552e045920e3565e3438d9bd5a79b9/samples/pubsub.py#L80) 
        SDK에서 제공되는 예제로 shadow를 사용하지 않고 바로 MQTT를 이용해 AWS IoT에 메시지를 전달하는 방법을 확인할 수 있습니다.
        이 예제에서는 command 명령어로 메시지를 지정해서 넣어주는데 저는 조금 변경하여 LED 색 바꾸던 shadow 예제처럼 프로그램 실행 중에 원하는 값을 전송하는 방식으로 해보려고 합니다.
        - [moistureSensor.py](https://docs.aws.amazon.com/iot/latest/developerguide/iot-moisture-raspi-setup.html)
        자습서에서 제공하는 라즈베리파이 이용 토양 수분 측정하는 예제에 사용되는 코드입니다. 여기서 어떻게 센서에서 측정한 값을 shadow을 통해 AWS IoT로 전송하는지 확인할 수 있습니다. 
        한 가지 의문인 건 센서에서 측정한 값까지 shadow로 전송해야 하는가 입니다. 물론 연결이 끊겨 측정값이 누락되는 것은 문제가 될 수 있겠지만 제 의견으로는 shadow로는 디바이스의 상태만 관리하고 값은 MQTT로 전송해서 바로 dynamoDB에 저장하는 게 맞지 않을까 생각합니다.
        - 기본적인 코드는 pubsub.py를 토대로 하고 필요한 부분만 변경했습니다.
        변경한 부분은 """ """로 주석처리 했습니다.

    ```python
    """
    file name: mqtt_connected.py
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

    - 코드를 ~/aws-iot-device-sdk-python-v2/samples 내에 저장한다.
    다른 곳에 저장하면 awsiot 라이브러리를 사용할 때 경로를 따로 지정해줘야 하므로 그냥 원래 예제 파일이 들어있던 곳에 넣었습니다.
    - 아직 메시지를 게시하는 부분 외 필요 없는 부분을 다듬지 못한 코드입니다.
3. `AWS IoT Console → 테스트`에서 두 주제(my/things/RPI/update, $aws/things/RPI/shadow/update)를 구독한다. (여기서는 $가 1개인 게 맞습니다.)
4. 작성한 코드를 라즈베리파이에서 실행한다.
    - 라즈베리파이 cmd 에서 아래와 같이 입력합니다.
        - your-end-point: `AWS IoT Console → 설정` 에서 확인
        - 인증서는 라즈베리파이 시작하기 글에서 설정했던 그대로 사용합니다.
        - your-thing-name: 사물 이름으로 저는 RPI 입니다.
        - your-client-id: 클라이언트 아이디로 저는 RPI로 적었습니다.
        - —topic 뒤에는 메시지를 게시할 주제를 작성합니다.

```
cd ~/aws-iot-device-sdk-python-v2/samples
python3 mqtt_connected.py --endpoint your-end-point --root-ca ~/certs/Amazon-root-CA-1.pem --cert ~/certs/device.pem.crt --key ~/certs/private.pem.key --thingName your-thing-name --client-id your-client-id --topic my/things/RPI/update
```

## 결과

- 위 명령어를 cmd에서 실행하면 아래와 같이 실행됩니다.

<p align = "center">
  <img src = "/assets/images/MQTT_CONNECTED.PNG" alt="cmd result"> <br/>
  mqtt_connected.py 실행 결과
</p>

   - Enter desired value: 에 `true11` 이라고 작성합니다.
   - AWS IoT 테스트에서 구독한 주제로 값이 전송되었는지 확인합니다.
       - `my/things/RPI/update` 에 게시 했지만 작성했던 rule에 따라서 `$aws/things/RPI/shadow/update` 에도 게시가 된 것을 확인할 수 있습니다.
        
<p align = "center">
  <img src = "/assets/images/MQTT_CONNECTED2.PNG" alt="published to my/things/RPI/update"> <br/>
  my/things/RPI/update에 라즈베리파이가 게시한 메시지 확인
</p>

<p align = "center">
  <img src = "/assets/images/MQTT_CONNECTED3.PNG" alt="republished to $aws/things/RPI/shadow/update"> <br/>
  시스템 예약 주제로 재게시된 메시지
</p>

   - 디바이스 섀도우에서도 `my/things/RPI/update` 로 게시한 메시지를 확인할 수 있습니다.(시스템에 예약된 주제인 $aws/things/RPI/shadow/update로 메시지를 재게시 했기 떄문)

<p align = "center">
  <img src = "/assets/images/MQTT_CONNECTED4.PNG" alt="device shadow updated"> <br/>
  디바이스 섀도우 업데이트 됨
</p>


# 결론

- 디바이스에서 MQTT 프로토콜을 통해 AWS IoT로 원하는 메시지를 보내는 실습을 완료했습니다.
- connected 상태 확인을 위한 LWT 메시지는 못했는데 LWT 메시지를 보내기 위한 설정이 더 필요한 것으로 보입니다. 이 내용은 다음에 다뤄보겠습니다.
- 헤매다가 디바이스 shadow를 관리하는 법도 알게 되어 제가 원하는 방식으로 수정 후 다음에 다뤄보겠습니다.

# 참고

- AWS IoT 자습서
- AWS IoT SDK 예제
