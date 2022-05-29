<!-- TITLE: Garbage Collection -->
<!-- SUBTITLE: -->

# 링크
* https://plumbr.io/handbook/what-is-garbage-collection 가장 잘 설명된 글
* https://howtodoinjava.com/java/garbage-collection/all-garbage-collection-algorithms/ 단순하게 설명한 글
* Garbage collector에 대해 총정리. 기초부터 설명하는데 기초없이 이해가 어렵다.
  * https://dzone.com/articles/garbage-collection-java-part-1 Heap Overview
  * https://dzone.com/articles/garbage-collection-java-part-2 Parallel GC
  * https://dzone.com/articles/garbage-collection-java-part-3 Mark Sweep
  * https://dzone.com/articles/garbage-collection-java-part-4 G1

# GC알고리즘 콤비네이션
G1이전까지 Young Gen, Old(Tenured) Gen 각각 다른 GC알고리즘을 사용했기 때문에 조합을 지정하는 옵션이 있다.
Java8기준으로 가능한 조합을 타나낸 테이블인데 굵은 글씨만 실제 사용되고 나머지는 사용하지 않는다.
* https://www.linkedin.com/pulse/jvm-why-cms-garbage-collector-deprecating-kunal-saxena CMS는 Java9부터 없다.
* https://blogs.oracle.com/jonthecollector/our-collectors Garbage collector 특징 요약 및 아래 테이블에 나열된 조합의 이유를 FAQ로 설명함.

![2018 10 01 12 56 36](/research/java/files/garbage-collection/-2018-10-01--12-56-36.png "2018 10 01 12 56 36")