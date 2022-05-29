<!-- TITLE: Fork/Join -->
<!-- SUBTITLE: Fork/Join Framework -->

![Fork Join Parallelism](/uploads/fork-join/fork-join-parallelism.png "Fork Join Parallelism")
*  http://gee.cs.oswego.edu/dl/cpjslides/fj.pdf Fork/Join 프레임워크를 만든분이 만든 문서

# ForkJoinTask
abstract타입으로 위 이미지처럼 task를 나누고 결과를 합치는 로직을 만들기 위해 사용하는 클래스.
task를 나누고 병렬 처리하는게 목적이기 때문에 리턴 타입은 없어도 된다.
쉽게 사용할 수 있도록 ForkJoinTask서브타입으로 RecursiveTask, RecursiveAction을 만들어두었다.
둘의 차이는 리턴타입 유무로, RecursiveTask는 결과를 리턴할때, RecursiveAction은 실행만하고 끝낼때 사용한다.
이것 외에도 그냥 쉽게 사용할수 있도록 간단한 ForkJoinTask인스턴스를 쉽게 만들어주는 ForkJoinTask.adapt(...) 메소드를 제공. Callable, Runnable 받음.

# ForkJoinWorkThread
ForkJoinPool 내부에서 쓰레드풀을 통해 괸리되는 worker쓰레드. 자신의 task큐를 가짐.
Fork/Join 개념상 task를 작게 나눠 처리하므로 task는 쓰레드보다 가볍기 때문에(lightweight) task수대로 쓰레드를 만들지 않고 하나의 쓰레드가 여러 task를 들고 있는다.
ForkJoinPool이 task큐에서 task를 뺀다음 다른 worker쓰레드에게 주거나 가져와서 넣음.(work stealing)

# ForkJoinPool
ForkJoin API 구현의 핵심<br>
worker 쓰레드 관리, task 큐 관리. 즉, task를 적절하게 분배하고 실행시킨다.
commonPool이라는 전역에서 사용할 수 있는 쓰레드풀을 갖고 있다.
commonPool은 다른 parallel api에서도 사용함. 그래서 commonPool이라 지었나보다.
ex) Arrays.parallelsort, Collection.parallelStream
별도 쓰레드풀을 사용하겠다면 인스턴스 생성시 ForkJoinWokerThreadFactory 인스턴스를 전달한다.

commonPool은 시스템 프로퍼티로, 인스턴스 생성시에는 파라미터로 parallelism값을 받는데 가용할 최대 쓰레드수를 의미한다.(쓰레드풀의 최대 쓰레드 생성수보다 적을수 있다.)
지정하지 않으면 Runtime.getRuntime().availableProcessors()값으로 설정되며 최대 32767을 넘길수 없다.
parallelism값을 설정하더라도 Runtime.getRuntime().availableProcessors()값이 1이면 1로 설정된다.
