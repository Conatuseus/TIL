> Springboot로 토이 프로젝트를 진행중 Request DTO(requestBody로 오는)에 `@NoArgsConstructor`를 빠뜨려서 에러가 났다. (습관적으로 적어오던 어노테이션...)
> 그런데 `@RequestBody`로 넘어오는 객체에는 기본 생성자가 왜 필요한지, Setter는 왜 불필요한지 **이유**를 정확히 모르고 있었다.
> 이번 기회에 해당 내용들을 구체적으로 알아보자.

`@RequestBody`에 왜 기본생성자가 필요하고, Setter는 불필요한지 알아보기 위해선 Spring 내부에서 어떻게 처리하는지 확인해보면 됩니다.`@RequestBody` 까지 처리하는 flow를 보고싶다면 [여기](https://m.blog.naver.com/PostView.nhn?blogId=duco777&logNo=220605479481&proxyReferer=https%3A%2F%2Fwww.google.com%2F)를 확인하면 좋을 것 같다. 그리고 이동욱님의 [글](https://jojoldu.tistory.com/407)에는 Setter가 왜 필요없는지에 대해 적혀있어서 도움을 받아 적습니다.

 개발 환경과 코드는 아래와 같습니다.
 > OS: Mac OS
 >  IDE: Intellij
 >  Spring(boot) Version: SpringBoot 2.2.2
 >  etc: h2, JPA, Lombok 

**Posts Entity**
~~~ java
@Getter
@NoArgsConstructor
@Entity
public class Posts {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(length = 500, nullable = false)
    private String title;

    @Column(columnDefinition = "TEXT", nullable = false)
    private String content;

    private String author;

    @Builder
    public Posts(final String title, final String content, final String author) {
        this.title = title;
        this.content = content;
        this.author = author;
    }
}
~~~

**PostsSaveRequestDto**
~~~ java
@Getter
@NoArgsConstructor
public class PostsSaveRequestDto {

    private String title;
    private String content;
    private String author;

    @Builder
    public PostsSaveRequestDto(final String title, final String content, final String author) {
        this.title = title;
        this.content = content;
        this.author = author;
    }

    public Posts toEntity() {
        return Posts.builder()
            .title(title)
            .content(content)
            .author(author)
            .build();
    }
}
~~~
**PostsApiController**
~~~ java
@RequiredArgsConstructor
@RestController
public class PostsApiController {

    private final PostsService postsService;

    @PostMapping("/api/v1/posts")
    public Long savePosts(@RequestBody final PostsSaveRequestDto requestDto) {
        return postsService.save(requestDto);
    }
}
~~~
**PostsService**
~~~ java
@RequiredArgsConstructor
@Service
public class PostsService {

    private final PostsRepository postsRepository;

    @Transactional
    public Long save(final PostsSaveRequestDto requestDto) {
        return postsRepository.save(requestDto.toEntity())
            .getId();
    }
}
~~~
위와 같이 구현해 두었습니다.

# `@RequestBody` 바인딩을 어디서 하는지 알아보자


Request의 Body를 읽고 객체로 binding하는 과정을 알아보기 위해 아래와 같은 과정을 진행했습니다. `AbstractMessageConverterMethodArgumentResolver`의 `readWithMessageConverters` 메서드를 Breakpoint를 잡은 이유는 `@RequestBody`, `@ResponseBody`의 처리를 하는 `RequestResponseBodyMethodProcessor`에서 해당 메서드를 호출하기 때문입니다. 
1. request의 body를 binding하는 과정을 알아보기 위해 아래와 같이 해당 메서드에 Breakpoint를 잡았습니다.
![image.png](https://images.velog.io/post-images/conatuseus/b2fd6520-3db4-11ea-9149-31a630ce5639/image.png)

2. Postman을 통해 아래와 같이 POST 요청을 해봅니다.
![image.png](https://images.velog.io/post-images/conatuseus/1760bf20-3db6-11ea-a4b8-eb6deb732db9/image.png)

3. 그리고 계속 진행하다보면 **messageConverter들 중에서 적합한 HttpMessageConverter를 찾아 body를 읽는 부분**이 있습니다.
messageConverters에 10개의 컨버터가 저장되어 있는데, 이 중 REQUEST의 ContentType으로 읽을 수 있는 컨버터를 찾고, 해당 컨버터로 body를 읽는 메서드를 호출해 저장하는 과정이 포함되어 있습니다.
우리는 JSON으로 요청으므로 **MappingJackson2HttpMessageConverter**가 Body를 읽을 것입니다.
![image.png](https://images.velog.io/post-images/conatuseus/69a7b710-3db7-11ea-a4b8-eb6deb732db9/image.png)

4. 이제 **해당 컨버터(MappingJackson2HttpMessageConverter)가 어떻게 읽는지 `read()` 메서드를 이해하면 궁금증 2개를 해결할 수 있다는 것**을 알 수 있습니다.
해당 메서드를 타고 들어가보면, Jackson 라이브러리의 ObjectMapper를 사용해 body를 읽는 것을 확인할 수 있습니다. (이렇게 코드를 파다보면 공부할 것이 너무 많아지네요😅)
![image.png](https://images.velog.io/post-images/conatuseus/57127420-3dba-11ea-a4b8-eb6deb732db9/image.png)

---
여기까지는 제가 **어디서 RequestBody를 바인딩하는지 찾고 해석했느냐**에 대한 내용이었습니다. 

☛ 참고
Jackson 라이브러리는 `spring-boot-starter-web`에 포함되어 있습니다.

![image.png](https://images.velog.io/post-images/conatuseus/13319df0-3dc2-11ea-8cb9-370758fc36b2/image.png)


> `@RequestBody`에 대한 궁금증은 Jackson 라이브러리의 ObjectMapper가 어떻게 바인딩하는지 알아봐야한다는 결론이 나왔다. 따라서 이번 글에서는 여기까지만 적고, 다음 글에서는 ObjectMapper를 알아가보자