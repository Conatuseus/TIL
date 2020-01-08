# String, StringBuffer, StringBuilder Class



## 1. String 클래스

 몇몇 언어에서는 문자열을 char형의 배열로 다루지만, 자바에서는 문자열을 위한 클래스를 제공한다. 그 것이 String 클래스이다.

 ### 1.1 변경 불가능한(immutable) 클래스

![image](https://user-images.githubusercontent.com/22893111/71968566-88a88100-3248-11ea-98ec-8fae7852cbf6.png)

String 클래스의 내부를 보면, 문자열을 저장하기 위해서 문자형 배열 참조변수(char[]) value를 인스턴스 변수로 정의해놓고 있다. 인스턴스 생성 시 생성자의 매개변수로 입력받는 문자열은 이 **인스턴스 변수(value)에 문자형 배열(char[])로 저장**되는 것이다.

한번 생성된 String 인스턴스가 갖고 있는 문자열은 읽어 올 수만 있고, 변경할 수는 없다.
 `+` 연산자를 이용해서 문자열을 결합하는 경우, 인스턴스 내의 문자열이 바뀌는 것이 아니라 **새로운 문자열이 담긴 String 인스턴스가 생성되어 메모리 공간을 차지**하게 된다. **GC(Garbage Collector)가 호출되기 전까지 heap에 계속 쌓이므로 메모리 관리 측면에서 치명적이다.**



## 1.2 문자열의 비교

문자열을 만들 때는 문자열 리터럴을 지정하는 방법과 String 클래스의 생성자를 사용해서 만드는 방법이 있다.

~~~ java
String str1 = "abc";
String str2 = "abc";
String str3 = new String("abc");
String str4 = new String("abc");
~~~

String 클래스의 생성자를 이용한 경우에는 new 연산자에 의해서 메모리 할당이 이루어지기 때문에 항상 새로운 String 인스턴스가 생성된다. 그러나 문자열 리터럴은 이미 존재하는 것을 재사용하는 것이다.

![image](https://user-images.githubusercontent.com/22893111/71973339-15a40800-3252-11ea-8daa-b1f8028b7806.png)

## 2. StringBuffer 클래스와 StringBuilder 클래스

**String 클래스는 인스턴스를 생성할 때 지정된 문자열을 변경할 수 없지만 StringBuffer 클래스는 변경이 가능하다.** 내부적으로 문자열 편집을 위한 버퍼(buffer)를 가지고 있으며, StringBuffer 인스턴스를 생성할 때 그 크기를 지정할 수 있다.

StringBuffer 클래스는 String 클래스와 유사한 점이 많다. StringBuffer 클래스는 String 클래스와 같이 문자열을 저장하기 위한 char형 배열의 참조변수를 인스턴스 변수로 선언해 놓고 있다. StringBuffer 인스턴스가 생성될 때, char형 배열이 생성되며 이 때 생성된 char형 배열을 인스턴스 변수 value가 참조하게 된다.

### 2.1 StringBuffer의 생성자

StringBuffer 클래스의 인스턴스를 생성할 때, 적절한 길이의 char형 배열이 생성되고, 이 배열은 문자열을 저장하고 편집하기 위한 공간(buffer)으로 사용된다.

![image](https://user-images.githubusercontent.com/22893111/71974436-d4612780-3254-11ea-841c-f8476cf9627b.png)

위는 StringBuffer 생성자 코드다.

- public StringBuffer(): 버퍼의 크기를 지정하지 않은 것으로, 16개의 문자를 저장할 수 있는 크기의 버퍼를 생성한다.
- public StringBuffer(int capacity): capacity 크기의 버퍼를 생성한다.
- Public StringBuffer(String str): 지정한 문자열(`str`) 의 길이보다 16이 더 크게 버퍼를 생성한다.

아래의 코드는 StringBuffer 클래스의 일부인데, 버퍼의 크기를 변경하는 내용의 코드이다. StringBuffer 인스턴스로 문자열을 다루는 작업을 할 때, 버퍼의 크기가 작업하려는 문자열의 길이보다 작을 때는 내부적으로 버퍼의 크기를 증가시키는 작업이 수행된다. 배열의 길이는 변경될 수 없으므로 새로운 길이의 배열을 생성한 후에 이전 배열의 값을 복사해야 한다.

~~~ java
//새로운 길이(newCapacity)의 배열을 생성한다.
char newValue[] = new char[newCapacity];

//배열 value의 내용을 배열 newValue로 복사한다
System.arraycopy(value, 0, newValue, 0, count); // count는 문자열의 길이
value = newValue;	// 새로 생성된 배열의 주소를 참조변수 value에 저장
~~~

이렇게 함으로써 StringBuffer 클래스의 인스턴스 변수 value는 길이가 증가된 새로운 배열을 참조하게 된다.

### 2.2 StringBuilder의 변경

아래와 같이 StringBuilder를 생성해보자.

StringBuffer sb = new StringBuffer("abc");

![image](https://user-images.githubusercontent.com/22893111/71976070-c7dece00-3258-11ea-992b-1140c5423643.png)



그리고 sb에 문자열 "123"을 추가하면,

sb.append("123");

![image](https://user-images.githubusercontent.com/22893111/71976139-f2308b80-3258-11ea-92aa-bb14cb9c90be.png)



`append()` 는 반환타입이 StringBuffer인데 자신의 주소를 반환한다. 그래서 아래와 같은 문장이 수행되면, sb에 새로운 문자열이 추가되고 sb자신의 주소를 반환하여 sb2에는 sb의 주소인 0x100이 저장된다.

~~~ java
StringBuffer sb2 = sb.append("ZZ");	// sb의 내용 뒤에 "ZZ" 추가
System.out.println(sb);	// abc123ZZ
System.out.println(sb2); // abc123ZZ
~~~



### 2.3 StringBuilder란?

StringBuffer는 멀티쓰레드에 안전(thread safe)하도록 동기화되어있다. 따라서 **멀티쓰레드로 작성된 프로그램이 아닌 경우, StringBuffer의 동기화는 불필요하게 성능만 떨어뜨리게 된다.**

그래서 **StringBuffer에서 쓰레드의 동기화만 뺀 StringBuilder가 추가된 것이다.** StringBuilder는 StringBuffer와 완전히 똑같은 기능으로 작성되어 있어서, 소스코드에서 StringBuffer 대신 StringBuilder를 사용하도록 바꾸기만 하면 된다.





## 3. 결론

- String class

  문자열 연산이 적을 때 사용하면 좋다. 불변하기 때문에 멀티쓰레드환경에서 동기화를 신경쓸 필요가 없음.

- StringBuffer vs StringBuilder

  공통: 둘 다 문자열 연산이 많을 때 사용하면 좋다.

  차이점: StringBuffer는 thread safe하도록 동기화되어 있어서 멀티쓰레드 환경에서 사용, StringBuilder는 StringBuffer에서 쓰레드의 동기화만 뺀 것이므로 멀티쓰레드 환경이 아닐 때 사용