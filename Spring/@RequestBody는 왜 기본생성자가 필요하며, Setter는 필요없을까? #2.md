> [이전 글](https://velog.io/@conatuseus/RequestBody%EC%97%90-%EA%B8%B0%EB%B3%B8-%EC%83%9D%EC%84%B1%EC%9E%90%EB%8A%94-%EC%99%9C-%ED%95%84%EC%9A%94%ED%95%9C%EA%B0%80)에서는 어떻게 `@RequestBody`를 처리하는지를 알아보기 위한 **과정**을 설명했습니다. 이번 글에서는 `@RequestBody`를 바인딩하는 ObjectMapper에 대해 알아보고, 결론을 짓겠습니다.
> 참고로 아래 사진들에서 현재 위치(class)는 다음과 같이 찾으시면 됩니다. 첫번째 사진을 예로 들어, `AbstractJackson2HttpMessageConverter > readJavaType()` 에서 AbstractJackson2HttpMessageConver가 현재 클래스 이름이며, readJavaType()은 메서드명입니다.

# ObjectMapper란?
**Jackson의 Document에서 ObjectMapper**를 보면 다음과 같이 설명되어 있습니다.


> _"ObjectMapper는 기본 POJO(Plain Old Java Objects) 또는 범용 JSON Tree Model(JsonNode)에서 JSON을 읽고 쓰는 기능과 변환 수행을 위한 기능을 제공합니다.
> 또한 다양한 스타일의 JSON 컨텐츠와 함께 작동하고 다형성 및 객체 동일성과 같은 고급 객체 개념을 지원하도록 Customizing할 수 있습니다."_

쉽게 말해, **Jackson의 ObjectMapper는 Java Object ←→ JSON 파싱을 쉽게 해주는 클래스** 입니다.
ObjectMapper 클래스를 살펴보면, `serialize`와 `deserialize`라는 단어가 포함된 메서드가 많이 있습니다. 
Json을 Java 오브젝트로 파싱하는 것을 **직렬화(serialize)**라고 하며, 반대는 **역직렬화(deserialize)**라고 합니다.

# ObjectMapper가 `@RequestBody`를 binding하는 과정 살펴보기


지난 글에서 아래 그림과 같이 AbstractJackson2HttpMessageConverter가 `objectMapper`의 `readValue()` 메서드를 호출한다는 것까지 확인했습니다.

![image.png](https://images.velog.io/post-images/conatuseus/d426c4b0-3ed8-11ea-8609-9b6fc89aa201/image.png)

---
이제 `objectMapper.readValue()` 메서드 내부를 한번 살펴보겠습니다.
![image.png](https://images.velog.io/post-images/conatuseus/98c73550-3ee0-11ea-a6b8-eb179f450c1c/image.png)

objectMapper의 `readValue()`내에서는 **Request의 Body가 null인지만 확인**하고, 위 사진과 같이 **`_readMapAndClose()` 메서드를 호출**합니다.
이 `_readMapAndClose()` 메서드는 여러가지 작업들을 하는데, **핵심은 JsonDeserializer를 찾고 해당 JsonDeserializer로 deserialize(역직렬화)를 해서 Object를 반환해준다는 것**입니다.
위 코드에서는 `deser`가 찾은 deserializer이고, 해당 deserializer로 역직렬화하는 메서드를 호출하는 것이 빨간색 부분입니다. `PostsSaveRequestDto`는 Bean이므로 찾아진 deserializer는 BeanDeserializer로 보여집니다.

---
이제 deser.deserialize() 내부로 들어가봅시다.

![image.png](https://images.velog.io/post-images/conatuseus/677e9d60-3ee7-11ea-8557-e73ccea7dc99/image.png)

먼저 `p.isExpectedStartObjectToken()` 메서드는 Json이 "{"로 시작하는지 확인하는 메서드입니다. 그리고 `_vanillaProcessing`가 맞는지 확인하는데, `_vanillaProcessing`의 주석을 보니 
"*특수 기능이 전혀 없음을 나타내는 플래그로 가장 간단한 처리가 가능합니다.*" 라고 적혀있습니다.
다음에는 `_objectIdReader`가 null이 아닌지 확인하는데, `_objectIdReader`는 deserializer가 처리하는 값에 Object Id를 사용하려는 경우에 사용됩니다. 현재는 Object Id를 사용하지 않으므로 `if(_objectIdReader !=null)` 내부로 들어가지 않습니다.

---
`deserializeFromObject()` 메서드 내부를 들어가봅시다.

![image.png](https://images.velog.io/post-images/conatuseus/58bec1e0-3ee9-11ea-a9b9-25498ef943d9/image.png)

**위 그림의 코드 위에는 조건문과 처리과정**이 있지만, 조건문 내부를 들어가지 않으므로 생략하겠습니다.
이제 대망의 **핵심부분**입니다. 바로 Object를 생성하고, 필드를 넣어주는 부분에 도착했습니다. 하지만 여기서도 모두 메서드를 호출하기 때문에 봐야할 것들이 많이 남았습니다...😂

---
먼저 Object를 생성하는 메서드인 `createUsingDefault()`를 살펴봅시다. 메서드명만 봐도 default 생성자로 생성하겠다는 느낌이 확 옵니다.

![image.png](https://images.velog.io/post-images/conatuseus/4e32c180-3eea-11ea-95e4-0f27f07d39d7/image.png)

먼저 `_defaultCreator` 가 null인지 확인합니다. 여기서 defaultCreator란 기본 생성자를 말합니다(default constructor, @NoArgsConstructor)

### 기본생성자가 Null이면, 즉 기본 생성자가 없으면 왜 안되는지 여기서 파악할 수 있을 것이라 생각했습니다.
만약 default생성자가 없었다면, `_defaultCreator == null`을 만족했을 것이고 `super.createUsingDefault()` 메서드가 호출될 것입니다.
현재는 `@NoArgsConstructor` 어노테이션을 DTO에 작성했기 때문에 위의 조건에 만족하지 않았을 것입니다. 


_defaultCreator.call() 내부를 알아보기 전에, `@NoArgsConstructor`를 지운 후에 super.createUsingDefault() 메서드가 호출되는지 확인해보았습니다.

---
### 하지만 createUsingDefault() 메서드가 호출되기 전에 문제가 발생했다.
당연히 `createUsingDefault()` 메서드 내부에서 예외가 발생할 것이라고 생각했다. 왜냐하면 해당 메서드 내에 defaultCreator가 null인지 확인하는 부분이 존재했기 때문이다.

`@NoArgsConstructor` 어노테이션을 지우고, if(_defaultCreator == null)에 breakpoint를 찍고 디버깅을 돌렸는데... **해당 breakPoint에서 디버깅이 걸리지 않았습니다**...😡
이전에 **조건문에 걸리지 않는다고 생략했던 곳**에서 걸린 것이었습니다. 아래는 deserializeFromObject() 메서드의 생략된 윗부분 코드입니다.

![image.png](https://images.velog.io/post-images/conatuseus/23ab34b0-3eef-11ea-a9b9-25498ef943d9/image.png)

`@NoArgsConstructor`을 삭제하고 디버깅 한 결과 위 사진의 빨간색 부분에서 문제가 발생했습니다.
default생성자가 없으므로 _nonStandardCreation 조건을 만족합니다. 따라서 `deserializeFromObjectUsingNonDefault` 메서드를 호출해서 Object를 가져옵니다. 메서드명만 봐도 default생성자를 사용하지 않고 Object로 deserialize한다는 것을 알 수 있습니다.

![](https://images.velog.io/images/conatuseus/post/171eff75-ea77-4101-bb3a-dfb777ec4399/image.png)

그 메서드의 내부를 보면 **delegateSerializer를 사용하도록** 되어 있습니다. 하지만 저의 DTO는 delegate를 하지 않아서 delegateSerializer를 못 가져오기 때문에 오류가 발생한 것입니다.(property도 설정해준 것이 아니라서)

### 결국 default 생성자가 없어서 오류가 생기는 것은 맞다.
하지만, default 생성자가 없더라도 property 기반 클래스(property 관련 어노테이션이 적용된)면 필요 없을 것입니다. 또는 delegate를 한다면.

---

다시 돌아와서, `_defaultCreator.call()`를 들어가보면 다음과 같이 인스턴스를 생성하는 것을 알 수 있습니다.

![](https://images.velog.io/images/conatuseus/post/a9d53705-bd65-407b-a5d2-a4f6cde5001a/image.png)

![image.png](https://images.velog.io/post-images/conatuseus/a2f05dc0-3ef6-11ea-b2aa-9ff9b912552a/image.png)



## 결론
이번 글에서는 `@RequestBody`에 왜 기본 생성자(`@NoArgsConstructor`)가 필요없는지 알아봤습니다. 
제가 이번 글을 작성하면서 내린 결론은 다음과 같습니다.
**1. 일반적인 상황에서는 기본 생성자가 꼭 필요하다.**
**2. 왜냐하면, RestController에서 @RequestBody를 바인딩을 하기 위해 ObjectMapper를 사용하는데 기본 생성자로 DTO를 생성하기 때문이다.**
**3. 하지만 `@JsonProperty`, `@JsonAutoDetect` 등을 사용한 Property 기반 클래스이거나, 생성자가 위임된 경우라면 필요하지 않다.**

**Object를 생성하는 것까지 확인했으니 이제 주입하는 과정을 보면
"`@Setter`는 왜 필요 없는지" 에 대해 알 수 있을 것 같습니다. 다음 글에서는 해당 내용을 알아보겠습니다.**


> 이번 글을 작성하면서 코드를 이해하는데 시간이 많이 걸렸고, 코드를 파면 팔수록 모르는 것들이 많이 나와서 힘들었지만 재밌고 유익했습니다.
> 잘못된 정보가 있다거나, 이해가 안되는 부분이 있다면 댓글 남겨주시면 감사하겠습니다.
> 읽어주셔서 감사합니다.

- 참고: https://fasterxml.github.io/jackson-databind/javadoc/2.7/com/fasterxml/jackson/databind/ObjectMapper.html