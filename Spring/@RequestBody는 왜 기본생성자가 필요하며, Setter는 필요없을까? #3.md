> [이전 글](https://velog.io/@conatuseus/RequestBody에-왜-기본-생정자는-필요하고-Setter는-필요-없을까-2-ejk5siejhh)에서는 RestController에서 `@RequestBody` 바인딩을 Jackson 라이브러리의 ObjectMapper가 하는 것을 확인했습니다.
> 그리고 RequestBody를 생성할 때, DTO가 Property기반이 아니거나 Delegate를 한 상태가 아니라면 **기본 생성자**로 생성한다는 것을 알아냈습니다.
> 이번 글에서는 **기본 생성자로 생성하는데, 어떻게 해당 DTO에 Setter가 필요없지?** 라는 **의문점**을 풀어보도록 하겠습니다.

![](https://images.velog.io/images/conatuseus/post/627d3022-d0b7-4839-b739-360d9642eb32/image.png)

저는 RequestBody로 들어오는 RequestDto에 위와 같이 `@NoArgsConstructor`, `Getter` 어노테이션만 작성했습니다.

위와 한다면 `Setter`가 없는데 스프링이 어떻게 DTO에 값을 넣어줄까 의문을 가졌습니다. 이 문제를 해결하기 위해 ObjectMapper에 대해 좀 더 알아볼 필요가 있었습니다.

## 어떻게 ObjectMapper가 JSON Field와 Java Field를 매칭시킬까?
이를 알아보기 위해 [여기](http://tutorials.jenkov.com/java-json/jackson-objectmapper.html#how-jackson-objectmapper-matches-json-fields-to-java-fields)를 참고했습니다.

![](https://images.velog.io/images/conatuseus/post/3b3585e4-11e9-459e-bae7-96191dbfdad8/image.png)

이를 해석해 보면, 다음과 같습니다.
> 기본적으로 Jackson은 JSON 필드의 이름을 Java 오브젝트의 **getter 및 setter 메소드**와 일치시켜 JSON 오브젝트의 필드를 Java 오브젝트의 필드에 매칭합니다. Jackson은 getter 및 setter 메소드 이름의 "get"및 "set"부분을 제거하고 나머지 이름의 첫 문자를 소문자로 변환합니다. 
> 예를 들어 brand라는 JSON 필드는 getBrand () 및 setBrand ()라는 Java getter 및 setter 메소드와 일치합니다. engineNumber라는 JSON 필드는 getEngineNumber () 및 setEngineNumber ()라는 getter 및 setter와 일치합니다.


쉽게말해 **동일한 이름의 변수를 직접 찾는 것이 아니라, Getter와 Setter를 통해 찾아서 매칭한다**는 것입니다.


## DTO에 필드 값을 주입하는 과정을 살펴보자
변수의 매칭은 getter와 setter를 하는 것을 알아냈습니다. 그럼 그렇게 매칭한 것을 어떻게 주입시키는지 알아볼 차례입니다.
왜냐하면 setter를 사용해서 주입시킬 것이라 생각했었기 때문입니다.

### 실제로 getter와 setter로 프로퍼티를 가져오는지 확인하기
![](https://images.velog.io/images/conatuseus/post/9b795348-64c8-4f96-85ce-0d2e23ae2ad2/image.png)

BeanDeserializer를 생성하기 위한 `BeanDeserializerFactory`에서 해당 작업을 수행하는 것을 확인할 수 있었습니다.
위의 addBeanProps 메서드를 들어가보면 아래와 같이 Setter와 Getter로 Bean의 프로퍼티를 찾는 것을 알 수 있습니다. (사진에 안보이지만 아래에는 Getter로 찾는 부분이 있습니다.)
![](https://images.velog.io/images/conatuseus/post/96fd36ce-92ef-4fc7-8283-e159c49ca89f/image.png)

### 그럼 값 주입은 어떻게?
`BeanDeserializer`에서 DTO에 주입하는 코드를 찾았습니다.
아래 코드에서 p는 JsonParser입니다. propertyName으로 이전에 저장했던 프로퍼티를 가져옵니다. 그리고 이제 JsonParser와 Bean을 `deserializeAndSet` 메서드로 넘겨 값을 저장해주는 것입니다.
![](https://images.velog.io/images/conatuseus/post/905254c3-c67c-4247-bb9f-145ff8c3e12b/image.png)

이제 `prop.deserializeAndSet` 메서드로 들어가봅시다!
![](https://images.velog.io/images/conatuseus/post/79fd292a-9c04-47a0-af3a-a7dff7aec838/image.png)

여기서 이유를 찾았습니다!! 🎉
`_field`는 `java.lang.reflect` 패키지의 Field 자료형입니다. 결국 **reflection을 사용해서 값을 주입해주므로 `Setter`는 필요하지 않는 것**입니다.


### 여러 상황 테스트

> 테스트에서는 reflection을 사용해 value값을 보여드리기 위해 dto의 변수들을 모두 public으로 변경했습니다.

1. Setter & Getter 모두 없는 경우 테스트
![](https://images.velog.io/images/conatuseus/post/b4e9d640-99a7-48ae-b3cf-2c031e27b441/image.png)
objectMapper가 바인딩하는데 DTO에 setter와 getter가 없으니 찾지 못해 오류가 납니다.

2. Setter만 있는 경우 테스트
![](https://images.velog.io/images/conatuseus/post/26b93ae3-8513-480c-ae4e-5e58b3b08be4/image.png)

3. Getter만 있는 경우 테스트
![](https://images.velog.io/images/conatuseus/post/1b8846f5-abfb-405c-9847-67ea385af1c9/image.png)


이 테스트들을 통해, Setter와 Getter 모두 없는 경우에는 `ObjectMapper`가 바인딩하는데 오류가 생기며
**Setter, Getter 둘 중 하나만 있으면 된다는 것**을 알았습니다.

## 3번째 글을 쓰면서 내리는 결론
- `@RestController`에서 `@RequestBody`의 바인딩은 Jackson라이브러리의 **`ObjectMapper`**가 해준다.
- `ObjectMapper`는 `@RequestBody`가 Property로 구현되어 있거나 생성을 위임한 경우가 아니라면 **기본 생성자**로 생성한다.
- 따라서 두 상황이 아니라면 기본 생성자는 꼭 필요하다. (`@NoArgsConstructor`가 필요한 이유)
- ObjectMapper는 **Setter와 Getter로 DTO의 필드**를 가져온다.
- 그리고 setter를 사용하는 것이 아니라 **reflection을 사용해서 필드 값들을 넣어준다.**
- 따라서 DTO에 사용할 필요가 거의 없는 `Setter`보다 프로덕션 코드에 사용되거나 테스트에서 필요한 `Getter`를 넣는게 좋겠다는 것이 저의 의견입니다. (쉽게 말해, Setter 또는 Getter 둘 중 하나만 있으면 되는데 Getter를 넣자!)


> 오타 또는 잘못된 내용이 있으면 댓글이나 메일 주시면 정말 감사하겠습니다.
>
> 참고한 자료
> - https://fasterxml.github.io/jackson-databind/javadoc/2.7/com/fasterxml/jackson/databind/ObjectMapper.html
> - http://tutorials.jenkov.com/java-json/jackson-objectmapper.html#how-jackson-objectmapper-matches-json-fields-to-java-fields 