---
layout: post
title:  "Improving Realtime Bidding Using A Constrained Markov - Matrix Factorization"
author: "Neo.B"
tags: [PaperReview]
comments: true
---

### Introduction
이번 포스팅은 Matrix Factorization, 행렬 분해에 대한 포스팅입니다. 행렬 분해는 추천 문제를 풀기 위해 사용되며, 보통 이를 이해하기 위해 Low-rank Matrices와 Singular Value Decomposition 등에 대한 설명을 먼저 시작하게 됩니다. 하지만 이 포스팅에서는 위의 개념들은 제외하고 영화 평점 데이터를 사용해서 행렬 분해의 과정과 업데이트 방식에 대해서 설명하는 것을 목적으로 합니다. 

### Method

여기서 행은 각 유저를 나타내고 열은 영화(제목)이며 행렬의 각 요소들은 유저가 해당 영화에 부여한 평점을 의미합니다. Matrix Factorization을 사용하면 각 유저가 평점을 부여하지 않은 영화 즉 요소가 0인 곳에 대해서, 추정할 수 있게 됩니다. 추정 과정은 다음과 같습니다.

![image](https://user-images.githubusercontent.com/49015329/57129001-31583400-6dd0-11e9-9255-8df88017b057.png)

1.	MF는 원래의 R 행렬을 그보다 작은 2개의 User 행렬과 Movie 행렬로 분해합니다.
2.	R 행렬의 사이즈(5*4)는 분해한 User 행렬(5*k)과 Movie 행렬(k*4)의 내적과 동일합니다.
3.	k는 잠재 요소(latent factor)로 원래의 R 행렬의 행 혹은 열보다 작은 숫자 입니다.
4.	분해한 행렬에는 랜덤한 값을 채워 넣습니다. 
5.	두 행렬의 내적을 통해 구한 R’ 행렬을 R 행렬 요소와의 차이를 반영해서 정답에 가까워지도록 P와 Q를 업데이트 해나갑니다.

위의 순서에 따라 원래 R 사이즈(5 * 4)의 행렬을 그보다 작은 2개의 유저 행렬과 영화 행렬로 분해 합니다. 여기선 잠재 요소(latent factor)의 크기를 2라고 해봅니다. 잠재 요소란 유저가 영화에 평점을 부여하는데 작용하는 어떤 잠재적인 영향을 나타낸다고 할 수 있습니다. 예를 들면 어떤 유저는 영화에 평점을 부여할 때 장르를 중요시 할 수 있고, 어떤 유저는 특정 감독을 중요시 하는 등의 요소들이 있다면, 정확하게 무엇인지는 알 수 없지만 해당 유저가 평점을 부여하는데 영향을 미칠 것이라 생각해볼 수 있습니다.
이제 User 행렬과 Movie 행렬을 각각 P와 Q라고 칭하도록 하겠습니다. P와 Q의 내적을 구하게 되면 R매트릭스의 크기와 동일한 매트릭스가 됩니다. P와 Q의 요소 값은 랜덤한 값으로 지정합니다. 
 
![image](https://user-images.githubusercontent.com/49015329/57129028-3e752300-6dd0-11e9-921d-2b5546a542c3.png)

P의 각 행은 유저 한 명 한 명을 나타내는 것이고, 열은 그 유저가 영화 평점을 부여할 때 작용하는 어떤 잠재적인 변수입니다. Q는 반대로 열이 영화(제목)을 나타냅니다. 이 둘은 R 행렬 크기에서 행 크기와 열 크기를 동일하게 따온 것이므로 내적을 구하게 되면 R 행렬의 크기와 동일할 겁니다. 

![image](https://user-images.githubusercontent.com/49015329/57129053-4cc33f00-6dd0-11e9-83d3-9517979184d7.png)

이제 행렬의 크기와는 동일하지만 요소 값은 랜덤하게 만든 P행렬과 Q행렬의 내적으로 R행렬의 첫번째 요소인 5 위치의 값을 구해봅니다. 0.54722372 * 0.77650817 + 0.69886878 * 0.91561631인데, 그럼 이제 이 값과 원래 매트릭스의 값인 5의 차이를 구해서 그 요소 위치의 에러라고 지정을 하고, P매트릭스와 Q매트릭스에 보정해줍니다. 이렇게 계속 각 요소별로 수차례 반복하다 보면 각 요소에 가까운 값이 나오게 될 겁니다. 

![image](https://user-images.githubusercontent.com/49015329/57129067-5482e380-6dd0-11e9-81ad-5ba8c64f7cdc.png)

![image](https://user-images.githubusercontent.com/49015329/57129093-66648680-6dd0-11e9-82c1-3d077293c42e.png)
![image](https://user-images.githubusercontent.com/49015329/57129096-68c6e080-6dd0-11e9-8c8e-1038ac664b43.png)

이번엔 R 매트릭스에서 요소 값이 0인 경우를 살펴보도록 하겠습니다. R(1,3)의 위치에는 0인 요소가 있습니다. 이 위치의 값을 구하기 위해서는 P와 Q에서 각 빨간 네모 두 개의 내적을 구하면 됩니다. 위의 코드에서 보이는 것처럼, 0인 부분이기 때문에 업데이트가 이루어지지 않는다고 생각할 수 있습니다. 그러나 P와 Q의 빨간 네모 부분은 R(1,3)을 구하는데만 사용되지 않고, 다른 0이 아닌 요소의 부분의 값을 추정하는데도 사용되기 때문에 지정한 반복 횟수(여기서는 5천 번)만큼의 업데이트가 이루어지게 됩니다. 

![image](https://user-images.githubusercontent.com/49015329/57129121-72504880-6dd0-11e9-86bd-fad5302dee1e.png)
![image](https://user-images.githubusercontent.com/49015329/57129125-74b2a280-6dd0-11e9-9147-6468f1394f58.png)

위는 랜덤한 값으로 분해했던 P와 Q의 내적과 R 매트릭스의 각 요소 위치 별로 0이 아닌 요소
들과의 오차를 반복 횟수만큼 업데이트 해서 원래의 R 매트릭스 요소들에 근사하는 행렬을 구한 것입니다. 각 위치 별로 확인해 보면, 0이 아니었던 위치는 그 값에 근사하는 값들이 형성됐습니다. 처음에 0이였던 위치도 0이 아닌 값들로 채워진 것을 확인할 수 있습니다. 이는 Matrix Completion 문제라고도 칭해지며, 마지막으로 정리해보자면 다음과 같습니다.

유저가 영화에 평점을 부여한 매트릭스가 있고, 그 매트릭스에서 유저가 평점을 부여하지 않은 영화에 대한 평점을 예측하는 문제를 풀기 위해 Matrix Factorization을 사용해보았습니다. 이미 평점이 부여되어 있는 부분을 정답으로 사용하고, 원래의 R 행렬을 그보다 작은 2개의 행렬로 분해해서 각 요소별로 정답과의 오차를 업데이트 해나가는 과정을 통해서 평점이 부여되지 않은 0인 요소를 갖는 부분의 추정치도 구할 수 있게 해주는 방법이 행렬 분해라고 할 수 있습니다. 

참고 : [자료1](http://sanghyukchun.github.io/73/), [자료2](http://www.quuxlabs.com/blog/2010/09/matrix-factorization-a-simple-tutorial-and-implementation-in-python/#source-code), [자료3](https://blog.insightdatascience.com/explicit-matrix-factorization-als-sgd-and-all-that-jazz-b00e4d9b21ea), [자료4](http://www.albertauyeung.com/post/python-matrix-factorization/)

