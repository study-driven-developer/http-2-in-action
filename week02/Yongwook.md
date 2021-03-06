# HOL Blocking 이란?

## 두 개의 head of line(HOL) blocking

웹에서 HOL Blocking을 말할 때에는 두 가지 종류가 있다.

- `HTTP`에서의 HOL Blocking
- `TCP`에서의 HOL Blocking

이 두 가지는 다른 것이므로 **어떤** 레이어에서의 이야기인지를 명시해야할 필요가 있다.

---

## HTTP에서의 HOL Blocking

`HTTP/1.1`의 요청-응답 쌍은 항상 순서를 유지하고 동기적으로 수행되어야 한다.

구체적으로 1개의 TCP 커넥션 상에서 3개의 이미지 (`a.png`, `b.png`, `c.png`)를 받는 경우, HTTP 리퀘스트는 다음과 같이 된다.

```text
|---a.png---|
            |---b.png---|
                        |---c.png---|
```

하나의 요청이 처리되고 응답을 받은 후에 다음 요청을 보낸다.

이전의 요청이 처리되지 않았다면 그 다음 요청은 보낼 수 없다는 것이다.

만약 `a.png`의 요청이 막혀버리게 되면 `b,c`가 아무리 빨리 처리될 수 있더라도 전체적으로 느려지게 된다.

```text
|------------a.png------------|
                              |-b.png-|
                                      |---c.png---|
```

이것이 바로 `HTTP/1.1`의 HOL Blocking이다.

`HTTP/1.1`의 `pipelining`이라는 사양은 (조건부로) 요청만 먼저 보내버리는 것으로, 이 문제를 회피하는 것처럼 보인다.

그러나 응답을 보낸 순서대로 무조건 받아야 하므로 `a.png`가 막혔을 경우에 생각보다 큰 효과를 보기 어렵다.

---

## HTTP/2의 경우

`HTTP/2`의 경우 요청은 하나의 연결에서 병렬적으로 보내질 수 있다.

즉, `a ~ c.png`가 모두 병렬적으로 요청되고, 응답된다는 것이다.

따라서 `a.png`가 시간이 걸리는 처리에서도, `b,c.png`는 먼저 받아서 보여줄 수 있다는 것이다.

```text
|------------a.png------------|
|-b.png-|
|---c.png---|
```

따라서 `HTTP/1.1`에서의 HOL Blocking은 `HTTP/2`에서는 발생하지 않는다고 말할 수 있다.

또한 `HTTP/2`는 접속의 `Flow Control` 과 중요한 자원의 우선순위를 부여하는 `Priority` 를 가지고 있기 때문에 세세한 제어가 가능하다.

---

## TCP에서의 HOL Blocking

`TCP`에서의 HOL Blocking은 `HTTP` 요청/응답을 `TCP` 패킷 레벨로 바꾼 거라고 생각하면 된다.

`TCP`는 패킷을 전송할 때에, 전달을 보장하기 때문에 패킷이 손실되면 재전송하게 된다.

그리고 재전송이 발생하면 패킷의 순서가 역전되지 않도록 후속 패킷이 대기하게 된다.

즉, `TCP` 상에서 3개의 패킷을 보낼 때, 먼저 보낸 패킷에서 손실이 발생하면 뒤도 막히게 된다.

이것이 `TCP`에서의 HOL Blocking 이다.

예를 들어, `HTTP/2`로 다중화된 요청은 `TCP`에서는 단순한 패킷이므로 패킷이 막히면 전체가 지연되는 문제는 피할 수 없다.

오히려 1개의 `TCP` 커넥션으로 전부를 처리하고 있기 때문에 여러 개의 `TCP`를 사용할 때보다 영향이 클 수도 있다.

> 실제로 테스트를 통해 패킷 손실률이 2%일 때 (이는 끔찍한 네트워크 품질이다), HTTP/1 사용자가 오히려 더 나은 것으로 입증되었다.
>
> 그 이유는 HTTP/1이 보통 패킷을 분배하는 데 6개의 TCP 연결을 갖고 있어서 손실된 패킷이 없는 연결은 계속 사용할 수 있기 때문이다.

```text
|----packet1----|xxx lost xxx|----packet1----|
                                             |-packet2-|
                                                       |--packet3--|
```

---

## 문제를 완화하기 위한 시도들

### 혼잡 제어 알고리즘 PRR과 BBR 사용

대부분의 `TCP` 구현에서는 `CUBIC` 알고리즘을 사용한다. 이 알고리즘은 패킷이 손실될 때 혼잡 제어 창을 절반으로 줄이는 것을 감소시키는 `PRR`로 강화됐다. 

그리고 훨씬 더 새로운 알고리즘인 `BBR (Bottleneck Bandwidth and Roundtrip propagation time)` 은 HTTP/2 연결에 대해 성능을 더 향상시키는 것으로 확인됐다. 

![](https://images.velog.io/images/dnr6054/post/7e23146b-19bf-4b12-acc1-79b57daea614/Screen%20Shot%202022-03-29%20at%2010.45.10%20PM.png)

> 구글은 BBR을 적용 후 TCP 트래픽을 강화해서 전 세계 평균적으로 YouTube 네트워크 처리량을 평균 4 %, 일부 국가에서는 14 % 이상 개선했다고 한다.

이 후에 BBRv2 알고리즘도 발표되었고, Youtube에 적용이 되었다. 다음과 같은 개선이 있다고 한다.

- Reduced queuing delays: RTTs lower than BBR v1 and CUBIC
- Reduced packet loss: loss rates closer to CUBIC than BBR v1

### QUIC 프로토콜의 제안

`QUIC(Quick UDP Internet Connections)`는 이름에서도 볼 수 있듯, `TCP`가 아닌 `UDP`위에서 자체적으로 재전송 제어 메커니즘을 구성한다.

그리고, 그 다중화는, 처음부터 패킷 레벨의 HOL Blocking이 발생하지 않도록 설계되고 있다.

```text
|----packet1----|xxx lost xxx|----packet1----|
|-packet2-|
|--packet3--|
```

따라서 `HTTP/3`의 경우 두 HOL Blocking 문제가 모두 해결된다.

> `HTTP/3`의 가장 큰 특징은 `TCP/IP` 기반의 애플리케이션 레이어 프로토콜인 `HTTP`를 `UCP` 기반의 `QUIC` 위에 얹었다는 것이다.
>
> 이를 `HTTP over QUIC`라고 표현하고 줄여서 `HQ`라고 한다.
>
> `HTTP/2`에 있는 프레임, 스트림, 메시지 구조와 기술들은 그대로 `HTTP/3`로 승계되었고, 명칭만 `HQframe`, `QPACK` 등으로 변경되었다.
