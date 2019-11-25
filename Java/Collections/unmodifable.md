## 종류
- Collections.unmodifiableList()
- Collections.unmodifiableCollection()
- Collections.unmodifiableMap()
- Collections.unmodifiableSet()
- Collections.unmodifiableSortedSet()
- Collections.unmodifiableSortedMap()
- 등등 ...


## 용도

Map, List, Set 등의 자료구조를 읽기 전용으로 생성하고 싶을 때 사용.



## 어떻게?

Collections의 Unmodifiable~ 내부를 보면

add, put, remove 등의 메서드를 호출하면 UnsupportedOperationException 예외를 던지도록 했다.

읽기 전용이므로 value의 추가, 수정, 삭제 등의 작업은 할 수 없다.

 

## [Immutable vs Unmodifiable collection](https://stackoverflow.com/questions/8892350/immutable-vs-unmodifiable-collection)

