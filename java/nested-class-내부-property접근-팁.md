<!-- TITLE: Nested Class 내부 Property접근 팁 -->
<!-- SUBTITLE: 꿀팁 -->

# 문제점
A클래스 하위에 B클래스 하위에 C클래스 하위에 D클래스 를 가지고 있는 객체가 존재함

이 객체에서 D클래스의 getValueD()라는 메소드를 호출하고 싶음

그렇게 하기 위해서는 

getValueA().getValueB().getValueC().getValueD() 를 수행해야 하지만, null pointException이 생길가능성이 존재

그렇기 때문에 getValueA(), getValueB(), getValueC(), getValueD()를 모두 null check를 해야함

코드가 길어지고 가독성 사라짐


#팁
	public static <T> Optional<T> optionalOf(Supplier<T> resolver) {
        try {
            T result = resolver.get();
            return Optional.ofNullable(result);
        } catch (NullPointerException | IndexOutOfBoundsException e) {
            return Optional.empty();
        }
    }