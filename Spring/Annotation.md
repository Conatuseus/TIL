

# Spring

- `@EnableWebSecurity` 

  Spring Security 설정들을 활성화시켜 줍니다.

## JPA





# Lombok



## `Data` 어노테이션

`Data` 어노테이션이 어떤 것들을 해 주는지 확인하기 위해 먼저 Document를 찾아봤다.

![image](https://user-images.githubusercontent.com/22893111/71266190-3ed82380-238b-11ea-905e-88ea3bb0e804.png)

 해석하자면, `@ToString`, `@EqualsAndHashCode`, `@Getter`, `@Setter,` `@RequiredArgsConstructor` 이 5개의 어노테이션  역할을 모두 해준다.

즉, toString, equals, hashCode, getter, setter 메서드를 자동생성 해주거나, (non-null이거나 final인 필드 값만 받는) 생성자를 만들어 준다.

처음에 이것을 보고 만능이라고 느꼈지만, 생각해보니 dto, vo 정도에서는 사용할 수 있겠지만 엔티티에서는 사용하면 안된다.

왜냐하면 엔티티에서는 Setter를 사용하지 않도록 배웠다. (의도 파악의 어려움, 객체의 일관성 등 때문)

그리고 엔티티에 `@ToString` 을 사용하게 되면 부모, 자식간 toString 메서드가 무한으로 호출될 가능성이 높다.

따라서 `@Data` 어노테이션 사용을 지양하도록 하자!
