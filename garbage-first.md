<!-- TITLE: G1 - Garbage First -->
<!-- SUBTITLE: Garbage First garbage collector -->

원글 : https://plumbr.io/handbook/garbage-collection-algorithms-implementations#g1

G1의 핵심 디자인은 stw pause를 분산하여 가비지 콜렉션을 예측 가능하고 설정가능하게 만드는 것이었다.
사실 G1은 soft real-time 가비지 콜렉터인데 구체적인 퍼포먼스 목표를 설정 할 수 있다는 뜻이다.
stw pause를 x밀리초보다 길지 않게하도록 요청 할 수 있다. G1은 이 목표를 높은 신뢰도로 만족시킬것이다.
하지만 확실하진 않지만 hard real-time은 어려울 것이다.

G1은 몇가지 통찰위에 만들어졌다. 첫째, 힙은 연속적인 Young/Old Gen으로 나뉘지 않아도 된다.
대신, 힙은 보통 2048개의 작은 구역으로 나뉘어 오브젝트를 저장한다.
각각의 구역은 Eden/Survivor/Old가 될 수 있다. 모든 Eden/Survivor 구역의 논리적인 결합이 Young Gen이고
모든 Old구역은 Old Gen이다.
![Ea 907 E 2 C Fdbd 4 E 8 C Bf 03 0 A 5 F 3764 D 2 Fb](/uploads/g-1/ea-907-e-2-c-fdbd-4-e-8-c-bf-03-0-a-5-f-3764-d-2-fb.png "Ea 907 E 2 C Fdbd 4 E 8 C Bf 03 0 A 5 F 3764 D 2 Fb")

이구조는 GC가 전체 힙을 한번에 콜렉팅하는 것을 회피하고 점진적으로 접근 할 수 있도록 해준다.
구역들을 묶은 힙의 하위세트(콜렉션 세트)가 콜렉팅 대상으로 고려될 것이다. 
모든 Young 구역은 pause 때마다 매번 콜렉팅되고 특정 Old구역도 함께 콜렉팅 된다.
![3 Fcd 3 C 52 B 6 E 8 476 B 87 E 0 06 Bf 8 F 837 Dfe](/uploads/g-1/3-fcd-3-c-52-b-6-e-8-476-b-87-e-0-06-bf-8-f-837-dfe.png "3 Fcd 3 C 52 B 6 E 8 476 B 87 E 0 06 Bf 8 F 837 Dfe")

또다른 G1의 참신함은 concurrent phase 동안 각 구역이 포함하고 있는 live오브젝트의 양을 계산한다. (이 계산은 콜렉션 세트 구축할때 사용된다.)
가비지가 대부분인 구역이 가장 먼저 콜렉팅된다. 그래서 이름이 garbage first collection이다.
활성화 옵션은 -XX:+UseG1GC.

# Evacuation Pause: Fully Young (Fully-Young Gen)
애플리케이션 라이프사이클이 시작될때 G1은 아직 실행되지 않은 concurrent phase에 대한 어떤 정보도 갖고 있지 않다.
그래서 처음에는 fully-Young mode로 동작 한다. Young Gen이 채워지고 있을때 모든 애플리케이션 쓰레드가 멈추고,
Young구역 내부에 live오브젝트는 Survivor구역으로 복사되거나 근처에 있는 Young/Old가 아닌 Free구역이 Survivor구역이 된다.

live오브젝트 복사 처리를 Evacuation으로 부르고 기존 Young콜렉터들과 아주 유사하게 작업한다.
evacuataion pause의 전체 로그양이 많기 때문에 단순화를 위해 첫 fully-young evacuation pause와 관련없는 로그를 생략한다.
concurrent phase를 자세하게 설명한뒤 생략한 부분들로 돌아올 것이다. 
완전한 로그 레코드를 위해 parallel phase 상세내용과 다른 phase의 상세내용이 분리된 섹션으로 추출됐다.
```text
0.134: [GC pause (G1 Evacuation Pause) (young), 0.0144119 secs]1
    [Parallel Time: 13.9 ms, GC Workers: 8]2
        …3
    [Code Root Fixup: 0.0 ms]4
    [Code Root Purge: 0.0 ms]5
    [Clear CT: 0.1 ms]
    [Other: 0.4 ms]6
        …7
    [Eden: 24.0M(24.0M)->0.0B(13.0M) 8Survivors: 0.0B->3072.0K 9Heap: 24.0M(256.0M)->21.9M(256.0M)]10
    [Times: user=0.04 sys=0.04, real=0.02 secs] 11
```
1. G1이 pause하고 Young구역 클리닝만 한다. JVM 스타트업 후 134밀리초 뒤에 pause되었고,  pause지속 시간이 0.0144초로 측정되었다.
2. 8개 쓰레드가 13.9밀리초 동안 병렬로 활동했다.
3. 생략한 내용은 다음 섹션에서 자세히 볼 것이다.
4. 병렬 활동을 관리하기 위해 사용중이던 데이터 구조들을 해제한다. 0에 근접해야 한다. 순차적으로 완료된다.
5. 더 많은 데이터 구조들을 비우고 매우 빨리 끝나야 하지만 반드시 0일 필요는 없다. 순차적으로 완료된다.
6. 기타 잡다한 활동들로 병렬화 되어있다.
7. 다음 섹션에서 자세히 볼 것이다.
8. pause 전/후 Eden사용량과 용량이다.
9. pause 전/후 Survivor구역으로 사용되는 공간이다.
10. pause 전/후 힙의 총 사용량과 용량이다.
11. GC이벤트의 지속시간을 다른 분류로 측정한 것이다.
    * user : 콜렉션 동안 가비지콜렉터가 소비한 총 CPU타입
    * sys : OS콜로 보낸 시간과 시스템 이벤트를 기다리는 동안 보낸 시간
    * real : 애플리케이션이 멈춰있는 동안의 clock time. GC동안 병렬화된 활동으로 이 숫자는 이론적으로 가비지콜렉터에 의해 사용된 쓰레드수로 (user + sys)를 나눈 것에 가깝다. 로그는 8개 쓰레드가 사용되었다. 어떤 활동은 병렬화되지 않았기 때문에 이 수치는 일정양에 대한 비율을 항상 초과한다.


무거운 작업의 대부분은 GC에 집중된 여러 워커 쓰레드들로 완료된다. 이들의 활동은 아래 로그를 통해 설명한다.
```text
[Parallel Time: 13.9 ms, GC Workers: 8]1
    [GC Worker Start (ms)2: Min: 134.0, Avg: 134.1, Max: 134.1, Diff: 0.1]
    [Ext Root Scanning (ms)3: Min: 0.1, Avg: 0.2, Max: 0.3, Diff: 0.2, Sum: 1.2]
    [Update RS (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.0]
        [Processed Buffers: Min: 0, Avg: 0.0, Max: 0, Diff: 0, Sum: 0]
    [Scan RS (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.0]
    [Code Root Scanning (ms)4: Min: 0.0, Avg: 0.0, Max: 0.2, Diff: 0.2, Sum: 0.2]
    [Object Copy (ms)5: Min: 10.8, Avg: 12.1, Max: 12.6, Diff: 1.9, Sum: 96.5]
    [Termination (ms)6: Min: 0.8, Avg: 1.5, Max: 2.8, Diff: 1.9, Sum: 12.2]
        [Termination Attempts7: Min: 173, Avg: 293.2, Max: 362, Diff: 189, Sum: 2346]
    [GC Worker Other (ms)8: Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.1]
    GC Worker Total (ms)9: Min: 13.7, Avg: 13.8, Max: 13.8, Diff: 0.1, Sum: 110.2]
    [GC Worker End (ms)10: Min: 147.8, Avg: 147.8, Max: 147.8, Diff: 0.0]
```
1. 8개 쓰레드가 병렬로 13.9밀리초 활동했다.
2. pause 시작에 맞춰 워커들의 활동이 시작될때까지 걸린 시간. 만약 Min/Max차이가 크다면 너무 많은 쓰레드들이 사용되고 있는 것이거나 다른 프로세스들이 JVM안에 가비지 콜렉션 프로세스로부터 CPU타임을 뺴앗고 있다.
3. external roots(클래스로더, JNI레퍼런스, JVM시스템 root 등등)를 스캔하는데 걸린 시간. Sum은 CPU타임이다.
4. 실제 코드로 부터의 roots(local변수 등등)를 스캔하는데 걸린 시간.
5. 콜렉팅된 구역에서 live오브젝트를 복사하는데 걸린 시간.
6. 워커 쓰레드들이 안전하게 멈출수 있게 보장하는데 걸린 시간. 완료될 작업이 없다면 정말 종료한다.
7. 워커 쓰레드가 종료을 시도한 횟수. 한번의 시도는 완료해야 할 작업이 더 있어 종결되기 너무 이른 상황을 발견했을때 실패한다.
8. 사소한 작은 활동들로 로그 섹션으로 불릴 건 아니다.
9. 워커 쓰레드들이 일한 총 시간.
10. 워커들이 작업을 마무리한 타임스탬프. 보통 이수치는 대충 같아야하고 너무 많은 쓰레드들이 매달려 있거나 간섭하면 차이가 있다.


추가적으로 evacuataion pause동안 수행되는 사소한 활동들이 있다. 이 섹션에서 일부분만 다루고 나중에 나머지를 다룰 것이다.
```text
[Other: 0.4 ms]1
    [Choose CSet: 0.0 ms]
    [Ref Proc: 0.2 ms]2
    [Ref Enq: 0.0 ms]3
    [Redirty Cards: 0.1 ms]
    [Humongous Register: 0.0 ms]
    [Humongous Reclaim: 0.0 ms]
    [Free CSet: 0.0 ms]4
```
1. 병렬화된 사소한 활동들.
2. non-strong레퍼런스를 제거하거나 제거하지 않아도 되는 것임을 결정하는데 걸린 시간.
3. 잔여 non-strong레퍼런스를 적절한 레퍼런스큐에 집어넣는데 걸린 시간.
4. 콜렉션 세트안에 해제된 구역들을 반환하는데 걸린 시간. 해제된 구역은 새로운 할당을 위해 활용할 수 있다.
# Concurrent Marking
G1콜렉터는 CMS의 여러가지 개념위에 구축됐다. 그래서 이 섹션을 진행하기 전에 CMS에 대해 충분히 이해해야 한다.
여러면에서 차이는 있더라도 Concurrent marking의 목표는 매우 비슷하다.
G1 Concurrent Marking은 Snapshot-At-The-Beginning접근을 사용한다.
marking사이클을 시작했을 당시의 모든 live오브젝트를 마킹한다. 마킹하는 동안 오브젝트가 가비지가 되었더라도 마킹한다.
live오브젝트의 정보는 각 구역을 위해 liveness stats(활성 통계)를 구축할 수 있게 해준다. 이 작업후 콜렉션 세트는 효율적으로 선택될 수 있게 된다.

이 정보는 Old구역안에서 가비지 콜렉션을 수행하기 위해 사용된다. 
마킹이 가비지만 가진 구역인지 가비지와 live오브젝트를 모두 가진 Old구역을 위한 STW evacuataion pause중인지 결정 한다면 완전 concurrently로 일어날수 있다.

Concurrent Marking은 전체 힙 점유가 충분히 클때 시작한다. 기본은 45%이다. InitiatingHeapOccupancyPercent JVM옵션으로 변경 할 수 있다.
CMS처럼 G1의 Concurrent Marking은 몇가지 phase로 구성된다. 어떤 phase는 완전 concurrent이고 이떤 phase는 애플리케이션 쓰레드들의 정지를 필요로한다.

## Phase 1: Initial Mark
이 phase는 GC roots로부터 직접 도달하는 모든 오브젝트들을 마킹한다. 
CMS는 이 phase를 위해 별도 STW pause phase를 필요로 한다. 그러나 G1은 Evacuation pause에 끼어들어가기 때문에 오버헤드가 최소화 된다.
Evacutation pause GC로그의 첫번째 줄에 있는 initial-mark로 알 수 있다

```text
1.631: [GC pause (G1 Evacuation Pause) (young) (initial-mark), 0.0062656 secs]
```

## Phase 2: Root Region Scan
이 phase는 root구역이라 불리는 구역에서 도달 할 수 있는 모든 live오브젝트들을 마킹한다. 비어있지 않는 구역은 마킹 사이클 중간에 콜렉팅 될 수 있다.
concurrent marking 중간에 오브젝트를 옮기는 건 트러블을 일으킬 것이므로 다음 차례 evacuation pause가 시작하기전에 반드시 완료되야 한다.
만약 끝나기전에 evacuation pause를 시작해야 한다면 root region scan의 중단을 미리 요청할 것이다. 그리고 evacuation pause가 끝날때까지 기다린다.
현재 구현은 root구역이 survivor구역이다. Young Gen의 일부분으로 다음 Evacuation pause때 확실히 콜렉팅 될 것이다.

```text
1.362: [GC concurrent-root-region-scan-start]
1.364: [GC concurrent-root-region-scan-end, 0.0028513 secs]
```

## Phase 3: Concurrent Mark
이 phase는 CMS와 아주 비슷하다. 단순히 오브젝트 그래프를 돌아다니면서 방문한 오브젝드를 스페셜 비트맵에 마킹한다.
snapshot-at-the-beginning의 의미를 만족시키기 위해 G1 GC는 오브젝트 그래프의 모든 concurrent update가 마킹 목적을 위해 알려진 이전 레퍼런스를 떠난 애플리케이션 쓰레드에 의해 만들어지는 것을 필요로 한다.
This phase is very much similar to that of CMS: it simply walks the object graph and marks the visited objects in a special bitmap. To ensure that the semantics of snapshot-at-the beginning are met, G1 GC requires that all the concurrent updates to the object graph made by the application threads leave the previous reference known for marking purposes.

Pre-Write배리어를 사용해 달성한다.(나중에 얘기할 Post-Write배리어와 혼동하면 안된다.)
배리어의 기능은 G1 Concurrent Marking 활동중 필드를 기록할때마다 이전에 참조된 오브젝트를 log buffer로 부르는 곳에 저장한다.
그리고 concurrent marking 쓰레드에 의해 처리된다.

```text
1.364: [GC concurrent-mark-start]
1.645: [GC concurrent-mark-end, 0.2803470 secs]
```

## Phase 4: Remark
이 단계는 CMS처럼 SWT이다. 마킹 처리가 완료된다. 
G1에서는 concurrent update log 유입을 멈추고 concurrent marking싸이클을 시작했을때 live오브젝트였지만 여전히 마킹안된 오브젝트들을 마킹하기 위해 애플리케이션 쓰레드들을 잠깐 멈춘다.
이 phase는 레퍼런스 처리(Evacuataion pause의 non-strong reference 참조)와 클래스 언로딩같은 부가적인 제거를 수행한다.

```text
1.645: [GC remark 1.645: [Finalize Marking, 0.0009461 secs] 1.646: [GC ref-proc, 0.0000417 secs] 1.646: [Unloading, 0.0011301 secs], 0.0074056 secs]
[Times: user=0.01 sys=0.00, real=0.01 secs]
```

## Phase 5: Cleanup
최종 phase이다. 다가올 evacuation phase를 준비한다. 힙구역들안에 모든 live오브젝트 갯수를 샌다. 그리고 예상하는 GC효율에 따라 각 구역을 정렬한다.
다음번 concurrent marking을 위한 내부 상태를 유지하기 위해 요구되는 모든 house-keeping활동들을 수행한다.

마지막으로 중요한 것은 no live오브젝트를 가진 구역들은 이 phase에서 전부 교정(reclaim)되었다.
이 phase에서 empty region reclamation이나 대부분 liveness calculation같은 특정 부분은 concurrent이지만 애플리케이션 쓰레드가 간섭하지 않는 동안 그림을 마무리 하기 위해 짧은 STW pause를 필요로한다.
STW pause로그는 아래 내용 같을 것이다.

```text
1.652: [GC cleanup 1213M->1213M(1885M), 0.0030492 secs]
[Times: user=0.01 sys=0.00, real=0.00 secs]
```

어떤 힙구역이 가비지만 갖고 있는 것을 발견했을때 pause 로그 포맷이 약간 다른 것을 볼 수 있다.

```text
1.872: [GC cleanup 1357M->173M(1996M), 0.0015664 secs]
[Times: user=0.01 sys=0.00, real=0.01 secs]
1.874: [GC concurrent-cleanup-start]
1.876: [GC concurrent-cleanup-end, 0.0014846 secs]
```

# Evacuation Pause: Mixed
concurrent cleanup에서 Old Gen의 전체 영역을 해제 가능하면 아주 좋은 경우다. 그러나 항상 있는 경우는 아니다.
Concurrent Marking이 성공적으로 끝난후에 G1은 mixed콜렉션을 스케쥴링한다.
Young구역에서 가비지를 버리는 작업뿐만 아니라 Old구역들의 뭉치를 콜렉션 세트로 던진다.

mixed Evacuation pause는 concurrent marking phase의 끝을 따라 항상 즉시 수행하진  않는다. 이에 영향을 주는 몇가지 룰과 휴리스틱이 있다.
예로 Old구역들의 상당한 부분을 concurrently 해제가 가능하다면 수행할 필요 없다.

따라서 concurrent marking의 끝과 mixed evacuation pause사이에 fully-young evacuation이 수차례 있을것이다.

Old구역의 정확한 갯수는 콜렉션 세트에 더해진다. 그리고 더해지는 순서가 있는데 몇가지 룰을 기준으로 선택된다.
애플리케이션을 위해 명시된 soft real-time performance 목표, liveness와 concurrent marking동안 콜렉팅된 gc 효율적인 데이터, 몇가지 JVM옵션이 포함된다.
mixed collection 처리는 앞서 보았던 fully-young gc와 많이 비슷하다. 그러나 지금은 remembered sets를 주제로 다룰것이다.

Remembered sets는 서로 다른 힙구역들을 독립적인 집단(콜렉션)이 될 수 있게 해준다.
예로 구역 A,B와 C를 콜렉팅할때 liveness를 결정하기 위해 구역 D,E로부터 레퍼런스가 있는지 반드시 알아야 한다.
그러나 전체 힙그래프 탐색은 상당히 오래 걸릴 것이고 점진적인 콜렉션의 전체 지점을 망가뜨릴 것이다. 그래서 최적화가 들어와야된다.
다른 GC알고리즘에서 Young영역의 독립적인 콜렉팅을 위한 카드 테이블보다 더 좋은 것을 갖고 있다.
G1은 Remembered Sets를 갖고 있다.

아래 이미지가 보여 주듯이 각 구역은 remembered set를 갖고 있다. remembered set은 바깥에서 이 구역을 가리키는 레퍼런스 목록이다.
이 레퍼런스들은 부가적인 GC roots들로 다루게 될 것이다. concurrent marking동안 가비지로 결정된 Old구역안에 오브젝트들은 바깥 레퍼런스가 이들을 향해도 무시하고 콜렉팅 될 것이다.
이 경우 가리키는 대상도 마찬가지로 가비지다.
![F 0 Ad 8 Ecd Bc 82 4 D 9 D Bac 8 60 C 6 F 0609 F 6 D](/uploads/g-1/f-0-ad-8-ecd-bc-82-4-d-9-d-bac-8-60-c-6-f-0609-f-6-d.png "F 0 Ad 8 Ecd Bc 82 4 D 9 D Bac 8 60 C 6 F 0609 F 6 D")

다음에 일어나는 일은 다른 콜렉터들이 하는 것과 똑같다.
multiple parallel GC가 어느 오브젝트가 live오브젝트고 가비지인지 파악한다.
![12 F 45 Bd 9 E 8 C 5 41 Df 9578 9 Dd 057837880](/uploads/g-1/12-f-45-bd-9-e-8-c-5-41-df-9578-9-dd-057837880.png "12 F 45 Bd 9 E 8 C 5 41 Df 9578 9 Dd 057837880")

마지막으로 live오브젝트들은 survivor구역으로 이동된다. 그리고 필요하다면 빈 구역을 새로 만든다.
이제 비워진 구역들이 해제되어 오브젝트들을 저장할 때 다시 사용할 수 있게 되었다.
![Untitled](/uploads/g-1/untitled.png "Untitled")

애플리케이션 런타임동안 remembered sets을 유지하기 위해 필드에 기록을 수행할때마다 Post-Write배리어를 발행한다.
만약 결과 레퍼런스가 구역간에 교차되는 경우, 예로 한 구역에서 다른 구역을 가리키는 한 대응 엔트리가 대상 구역의 remembered set안에 나타날 것이다.
이 오버헤드를 줄이기 위해 Write배리어가 도입되었다. remembered set안에 카드를 넣는 처리는 async이고 상당한 수의 최적화가 적용된 기능이다.
간단히 말하면 Write배리어는 dirty card informaion을 로컬 버퍼에 넣는다. 그리고 특수한 GC쓰레드가 이걸 집어내서 이 정보를 참조하는 구역의 remembered set으로 올려보낸다.

mixed mode로그는 fully young mode와 비교했을때 새로운 흥미로운 내용을 발행한다.

```text
[Update RS (ms)1: Min: 0.7, Avg: 0.8, Max: 0.9, Diff: 0.2, Sum: 6.1]
[Processed Buffers2: Min: 0, Avg: 2.2, Max: 5, Diff: 5, Sum: 18]
[Scan RS (ms)3: Min: 0.0, Avg: 0.1, Max: 0.2, Diff: 0.2, Sum: 0.8]
[Clear CT: 0.2 ms]4
[Redirty Cards: 0.1 ms]5
```
1. rememberes sets가 concurrently 처리된후 실제 콜렉션 시작전에 여전히 buffer에 저장된 카드가 처리되고 있는지 반드시 확인해야 한다. 이 수치가 높다면 concurrent GC쓰레드는 부하를 조절할 수 없다. 아마 들어오고 있는 필드 변경의 횟수가 압도적이거나 유휴 CPU가 불충분해서 그럴 것이다.
2. 각 쓰레드들이 로컬 버퍼를 처리하는데 걸린 시간
3. remembered sets으로부터 들어온 레퍼런스들을 탐색하는데 걸린 시간
4. 카드 테이블안에 카드들을 제거하는데 걸린 시간. 단순히 dirty상태만 지운다. dirty상태는 rememberes sets을 위해 사용되는 필드가 변경된 것을 알리기 위해 집어넣는다.
5. 카드 테이블안에 적당한 위치를 dirty로 마킹하는데 걸린 시간. 적당한 위치는 GC가 자체적으로 수행하는 힙 mutation으로 정해진다. The time it takes to mark the appropriate locations in the card table as dirty. Appropriate locations are defined by the mutations to the heap that GC does itself, e.g. while enqueuing references.

# Summary
이 내용은 G1이 어떻게 돌아가는지 충분한 기본 이해를 제공 할 것이다. 간결한 내용을 위해 humongous objects 등 자세한 구현은 많이 제외했다.
G1은 모든 것을 고려한 hotspot의 가장 진보한 production-ready 가비지 콜렉터다. 또한 hotspot엔지니어들에 의해 새로운 최적화들과 새로운 자바 버전을 통해 들어올 기능들로 끊임없이 향상되고 있다.

지금까지 본 내용처럼 G1은 pause예측으로 시작해서 힙 fragmentation으로 끝나는 CMS가 가진 넓은 범위의 문제들을 해결했다.
CPU사용에 제약없는 개별 연산들의 latency에 매우 민감한 애플리케이션이 주어진다면 G1은 hotspot사용자가 선택할 수 있는 최선이다.
특히 최신 자바버전에서 동작할때 좋을것이다. 그렇지만 latency향상은 공짜가 아니다.
G1의 throughput오버헤드는 write배리어와 더많은 active백그라운드 쓰레드 덕분에 더 크다.
그래서 만약 애플리케이션의 throughput이 제한되거나 CPU를 100% 점유하고 개별 pause지속시간이 얼마나 긴지 신경쓰지 않는다면 아마도 CMS나 Parallel이 더 나은 선택일것이다.

올바른 GC알고리즘을 선택하고 셋팅하는 방법은 오직 시도와 에러를 겪는 것이다.
그러나 다음 챕터에서 전반적인 가이드를 제공한다.

P.S G1은 Java9에서 십중팔구 기본 GC가 될것이다. http://openjdk.java.net/jeps/248
