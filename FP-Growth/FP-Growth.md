# FP-Growth

FP-Growth은 Apriori 알고리즘을 혁신적으로 개선한 알고리즘입니다. Apriori 알고리즘에서는 패턴을 찾기 위해 candidate라는 길이에 따른 패턴 후보를 만들고 그 패턴의 support를 구하기 위해 DB를 스캔해야 했습니다.

즉 DB를 스캔하는 횟수는 최소 1번에서 최대 가장 긴 트랜잭션의 아이템 수 까지 일어날 수 있습니다. 하지만 FP-Growth 알고리즘에서 DB를 읽는 횟수는 단 2번입니다. 그리고 또 하나의 큰 특징은 패턴 후보인 candidate를 만들지 않는다는 것입니다. 대신 트리와 노드 링크라는 특별한 자료구조를 사용합니다.

이 알고리즘의 풀 네임은 Frepuent Patterns Growth. 자주 일어나는 반복적인 패턴을 성장시킨다라고 생각 할 수 있습니다.

트랜잭션에서 minsup 이상의 아이템들로 트리를 구축하고, 그 트리에서 분할 정복 기법을 사용해서 또 하나의 아이템을 선택해서 새로운 서브 트리를 만듭니다. 이렇게 아이템의 길이를 하나씩 늘려가면서 서브 트리를 여러개 만드는데, 중지 조건은 이전 트리에서 더 이상 서브 트리를 구축할 수 없을 때, 선택한 아이템을 포함한 패턴의 support가 minsup보다 작을 때 입니다.



FP-Growth 알고리즘을 적용시키기 위해서 가장 먼저 해야 할 일은 FP-Tree를 만드는 것 입니다.

이 FP-Tree를 만듦으로써 우리는 기존에 데이터셋을 모두 찾아보면서 찾아야했던 frequent itemset들을 FP-Tree 하나에만 찾을 수 있게 됩니다. 우리에게 다음과 같은 데이터 셋이 있다고 가정해봅시다.

![img](https://blog.kakaocdn.net/dn/sx68t/btqz5zHLWHz/hTIQMAiEADPvvkC6BLmMCK/img.png)

> 초장만 산 사람이 9명, 과메기만 산 사람이 40명, 과메기와 김을 함께 산 사람이 50명, 과메기, 김, 초장을 모두 함께 산 사람이 1명이라는 뜻입니다.

1. 모든 각각의 item 들에 대해 전체 데이터셋에서 나온 빈도를 계산합니다.

   데이터셋의 모든 item 들에 대해 빈도를 계산합니다. 우리의 예시에서는 초장이 9+1 = 10으로 10번, 과메기가 40+50+1=91번, 김이 50+1=51번 나타났습니다. 이를 빈도가 높은 순서대로 표현하면 과메기:91, 김:51, 초장:10입니다.

2. itemset 하나씩 tree에 더함으로써 tree를 만들어줍니다.

   우선, 맨 처음에 0 note를 하나 만들어줍니다.

   ![img](https://blog.kakaocdn.net/dn/bDBObD/btqz5zuejDi/0TRvzTKIBinxnTXSxydin1/img.png)

   그리고 각 instance를 하나씩 더해주면서 tree를 만들어갑니다. 이 때, 빈도가 높은 item이 더 0에 가까운 node가 됩니다. 예를 들어, 우선 {과메기, 김} itemset으로 tree를 만든다고 합시다. 그러면 다음과 같은 형태가 됩니다.

   ![img](https://blog.kakaocdn.net/dn/EFyTW/btqz4Vq4Y22/p5bceFO0M8fkMY8X502Wu0/img.png)

   빈도가 더 높은 과메기가 더 가까운 곳에 위치하는 tree가 만들어 졌습니다. 이 단계에서 우리가 {과메기, 김, 초장} itemset을 추가한다고 해봅시다. 그러면 다음과 같은 형태가 될 것 입니다.

   ![img](https://blog.kakaocdn.net/dn/b9w2Ws/btqz3BUkP2I/b9wh9fqjkigvpa9GwVf5rk/img.png)

   이런식으로 dataset 안의 모든 itemset에 대해 tree를 만들 수 있습니다. 그러면 최종적으로 다음과 같은 FP-Tree를 얻을 수 있습니다.

   ![img](https://blog.kakaocdn.net/dn/qHVMC/btqz4iUxrfA/acTTH36wbZr7pbEJiXUSlK/img.png)

   이렇게 dataset 안의 모든 itemset의 정보를 반영하는 FP-Tree를 만들어낼 수 있습니다. 이 FP-Tree를 이용하면 아주 쉽게 support 값을 구할 수 있습니다. 예를 들어, tree에서 {김, 과메기}의 support를 구하고 싶다고 합시다. 김은 무조건 과메기와 함께 나타나기 때문에 김의 frequency만 고려하면 되고, 이는 51이므로 {김, 과메기}의 support는 51이 됩니다. 이러한 방법으로 우리의 dataset의 support를 모두 구하면 다음과 같습니다.

   |     item sets      | support |
   | :----------------: | :-----: |
   |      {과메기}      |   91    |
   | {김}, {김, 과메기} |   50    |
   |       {초장}       |   10    |
   | {과메기, 김, 초장} |    1    |

   예시 데이터는 굉장히 간단한 데이터였기 때문에 한 눈에 각 itemset들의 support를 구할 수 있었습니다. 하지만 데이터의 크기가 크거나 복잡한 데이터는 이러한 방법으로 support를 구할 수는 없습니다. 그러므로 복잡한 데이터에서 support를 쉽게 계산하는 방법을 알아보겠습니다.

   

다음과 같은 데이터셋이 있다고 가저앟ㅂ니다.

![img](https://blog.kakaocdn.net/dn/cC8tDq/btqz3AODDPQ/Pxaf42xtfqE1VSuDHJMo2K/img.png)

1. frequency를 이용해 infrequent item을 필터링합니다.

   우선, 각 item들의 frequency를 모두 계산해줍니다. 이는 다음과 같습니다.

   | item                      | frequency |
   | ------------------------- | --------- |
   | f, c                      | 4         |
   | a, b, m ,p                | 3         |
   | l, o                      | 2         |
   | d, e, g, i, j, h, k, n, s | 1         |

   우리는 frequency가 3인 이상인 아이템들만 고려한다고 가정합니다. 즉, 우리가 원하는 minsup이 3인 것입니다. 그러면 우리는 다음과 같은 아이템들만을 고려할 것입니다.

   | item       | frequency |
   | ---------- | --------- |
   | f, c       | 4         |
   | a, b, m ,p | 3         |

   제거된 item들을 제외하고 itemset을 다시 만들면 다음과 같습니다.

   ![img](https://blog.kakaocdn.net/dn/bOD5wi/btqz6IcVeae/ZlWG2xFWK3GzANwevGC16K/img.png)

2.  Filtered Dataset을 이용해 FP-Tree를 만듭니다.

   위에서 했던 과정과 같은 방법으로 FP-Tree를 만들어줍니다. 그러면 아래와 같은 FP-Tree가 만들어집니다.

   ![img](https://blog.kakaocdn.net/dn/cnHmKC/btqz6tAj7gB/Agu2fmlDD0GUUg0uYRpdKk/img.png)

3. frequent item들 각각을 postfix로 놓고 각 item 별로 recursice하게 support를 구합니다.

   우선, frequent item 인 f, c, a, b, m, p중 p를 먼저 살펴봅시다. 우선, 우리의 itemset에 p의 support가 3인 것을 넣을 수 있습니다. 다음으로, 우리의 FP-Tree 중 p가 포함되는 tree 가지만을 남기고 다른 것은 고려하지 않습니다. 그러면 tree는 다음과 같은 형태가 됩니다.

   ![img](https://blog.kakaocdn.net/dn/1obZV/btqz6hGTyu1/P8Yh6g1oMozNC0K6QWZSKk/img.png)

   이에 따라서 다시 각 item의 빈도를 구하면 위 그림의 왼쪽 박스와 같습니다. 우리의 minsup은 3이기 때문에, 이를 만족하는 item은 그림과 같이 p와 c밖에 없습니다.

   ![img](https://blog.kakaocdn.net/dn/zxvEK/btqz5z15ViE/oHdRikbw397Z8yVPNeFWa1/img.png)

   그러면 우리는 최종적으로 다음과 같은 FP-Tree를 얻게 되고, {p, c} itemset은 support가 3이되므로 조건을 만족하는 itemset입니다.

   ![img](https://blog.kakaocdn.net/dn/bNXdUU/btqz3B7SEwa/usDhnOKXG7X5Gpgznbzqn1/img.png)

   다음으로, m을 살펴봅시다. 우선, 위에서와 같은 우리의 itemset에 m의 support가 3인 것을 넣을 수 있습니다. 다음으로 우리의 FP-Tree중 m이 포함되는 tree 가지만을 남기고 다른 것은 고려하지 않습니다. 그러면 tree는 다음과 같은 형태가 됩니다.

   ![img](https://blog.kakaocdn.net/dn/baFP9u/btqz3BGL853/przVKxryEcygYDDZkxD1u1/img.png)

   이에 따라서 다시 각 item의 빈도를 구하면 위 그림의 왼쪽 박스와 같습니다. 우리의 minsup은 3이기 때문에 이를 만족하는 item은 아래 그림과 같이 m, f와 c와 a가 됩니다.

   ![img](https://blog.kakaocdn.net/dn/dtan42/btqz4Wcn5ei/UaM8wtF5FS7Drd3gBWgGRK/img.png)

   그러므로 우리는 최종적으로 위 그림에서 파란색의 FP-Tree를 얻게 됩니다. 그러므로 우리는 recursion을 통해 {m, f}, {m, c}, {m, a}, {m, c, a}, {m, f, c}, {m, f, a}, {m, a, c, f} itemset의 support가 3이 됨을 알 수 있습니다.

   모든 frequent item 들에 대해 이와 같은 과정을 적용시킴으로써 support가 3 이상인 모든 itemset을 얻을 수 있습니다. 그 결과는 다음과 같습니다.

   |                             item                             | frequency |
   | :----------------------------------------------------------: | :-------: |
   |                           {f}, {c}                           |     4     |
   | {a}, {b}, {m}, {p}, {p, c}, {m, a, c, f}, {m,c, f}, {m, a, f}, {m, a, c}, {m, a}, {m, c}, {m, f}, {a, c, f}, {a, c}, {a, f}, {c, f} |     3     |

   이러한 과정을 통해 모든 데이터셋에 대해 minsup를 만족하는 itemset을 구할 수 있습니다.

   

FP-Growth 알고리즘은 dataset 전체에 대한 순회를 FP-Tree를 만들 때에만 하면 되기 때문에 매우 빠르고 메모리에 무리도 적다는 장점이 있습니다. 이를 Association Rule, Sequence Mining에서 적절히 활용한다면 더욱 효율적인 처리가 가능할 것입니다.

