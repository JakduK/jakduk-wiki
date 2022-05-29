<!-- TITLE: Optimistic And Pessimistic Locking -->
<!-- SUBTITLE: Lock, Synchronization -->

# Optimistic Locking
설명은 이걸로 대신한다.
* https://howtodoinjava.com/java/multi-threading/compare-and-swap-cas-algorithm/
자바 Atomic으로 시작하는 클래스들이 이 방식으로 구현되어 있음.
일종의 spin locking같은데 lock획득/반환이 없으므로 lock을 사용하는 방식과 성능 비교가 무의미하다.
다른 쓰레드들이 내가 접근할 대상을 변경하는지 안하는지 내가 계속 확인 해보자라는 방식이라 낙관적이라 하는 건가?

# Pessimistic Locking
locking하면 떠오르는 전통적인 방식의 locking.
synchronized 키워드 사용하는 locking을 떠올리면 된다. 이를 pessimistic technique이라고 부른다.
강력하고 성능 저하가 심한건 모두가 아는 상식.
쓰레드 상태 전환, context switching은 둘째치고 lock획득까지 아무것도 못하는게 제일 문제.
다른 쓰레드들이 내가 접근할 대상을 변경할 걸 염두에 두고 모조리 차단하는 방식이라 비관적이라 하는 건가?
