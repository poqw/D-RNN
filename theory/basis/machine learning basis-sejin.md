문제를 해결하는 데 있어서 가장 기본이 되는 일은 목표가 무엇인지 정확히 아는 것이다. 모든 기계학습의 목표는 결국 **loss function의 값이 최소가 되는 hypothesis function을 찾는 것**이다. 그럼 이 둘이 무엇인지 알아봐야겠다.



## 가설 함수 (Hypothesis Function)

세상의 모든 일은 함수로 표현할 수 있다. 여기서 [함수](https://en.wikipedia.org/wiki/Function_(mathematics)#Definition)란, 주어진 입력에 대해서 출력값이 1:1로 대응되는 관계를 일컫는다. 예를 들어, 내가 오늘 먹을 저녁메뉴는 내가 오늘 먹은 점심메뉴($x_1$), 내 위치($x_2$), 같이 먹는 사람들($x_3$), 내가 갖고 있는 돈($x_4$) 등 여러 입력을 고려하여 결정하게 된다. 

기계학습은 입력들과 그에 대응되는 출력이 무엇인지가 주어지고, 궁극적으로 그 사이의 함수를 찾아내는 것을 목표로 한다. 이러한 함수는 여러 형태가 될 수 있지만, 만약 그 형태가 선형함수라고 한다면, $h(x_1, x_2, x_3, x_4) = \theta _1x_1 + \theta _2x_2 + \theta _3x_3 + \theta _4x_4 + b$와 같은 형태로 나타낼 수 있다. 학습이 진행되기 전에는 계수에 대해 아는 바가 없기 때문에 계수로 랜덤 값을 쓴다. 이 때의 함수 $h$를 **가설 함수(hypothesis function)**라 한다. 

GPU의 이점을 최대한 활용하기 위해서는 식들을 최대한 행렬로 표현하는 게 좋다. 가설함수는 계수와 변수를 별도의 행렬로 분리해서 곱하는 형태로 표현할 수 있는데 이 때 계수의 벡터 $W = \{\theta _1, \theta _2, \theta _3, \theta _4, b\}$를 **Weight**라 하고, 가설 함수가 선형이라면 $h = WX$ 형태로 표현 가능하다.



## 손실 함수 (Loss Function)

가설 함수는 가설일 뿐이므로, 가설 함수에 주어진 입력을 넣었을 때 나오는 값은 실제 나와야 하는 값과 차이가 있기 마련이다. 그래서 오차의 총량을 구해서 이 값이 최소가 되는 가설 함수를 구한다면, 그 가설 함수는 우리가 구하고자 하는 함수와 가장 가까울 것이다. 이 때 오차의 총량을 나타내는 함수가 바로 **손실 함수(loss function)**, 또는 **비용 함수(cost function)**이다.

Loss function은 다음과 같이 정의된다.
$$
J(x) = {1 \over 2m} \sum^m_1(y - h(x))^2
$$
약간 TMI지만 [저 $J$는 Jacobian Matrix의 J를 따온 것](https://stackoverflow.com/questions/42420235/what-does-the-capital-letter-j-mean-in-cost-function-j%CE%B8)이라고 한다. 자세한 건 나도 모르겠다. 

제곱이 있는 이유는 양수와 음수의 상쇄를 막기 위함이다. 쉽게 생각해서 통계에서 배우는 분산과 똑같은 공식이라고 보면 된다. 

분모에 2가 곱해져 있는 이유가 의아할 수 있는데, 나중에 [gradient descent](https://enhanced.kr/postviewer/1697) 다룰 때 나오겠지만 사실 손실 함수는 편미분을 당하기 위해 태어난 녀석이다. 따라서 그 때의 계산을 깔끔하게 만들어주기 위해 분모에 2를 그냥 붙여놓은 것이다. '아니, 저렇게 계산 쉽게 만들겠다고 분모에 2를 막 붙여도 되나' 싶을텐데, 결론적으로 된다. 손실 함수는 사실 꼭 저 꼴이 아니어도 된다. 제곱 대신 절댓값을 써도 되고 네제곱을 써도 상관없다. 가설함수와 실제 데이터 간의 오차를 제대로 표현만 하면 되기 때문에 꼭 저 꼴이 아니더라도 코드 짜는 사람이 정의하기 나름이다. 실제로 [가설함수에 지수함수가 들어가는 경우](http://gnujoow.github.io/ml/2016/01/29/ML3-Logistic-Regression/), loss function에 밑도 끝도 없이 $\log$를 집어넣어 버리기도 한다.



## Hypothesis Function Optimization

기계학습의 목표가 loss function이 최소가 되는 hypothesis function을 찾는 것이라고 했다. 그럼 이번에는 그 오차가 최소가 되는 hypothesis function을 **어떻게 찾을 것인지**에 대해 고민해보자.

### Differentiation

[대한민국 중등교육](https://ko.wikipedia.org/wiki/%EC%A4%91%EB%93%B1_%EA%B5%90%EC%9C%A1#%ED%95%9C%EA%B5%AD)을 잘 이수한 사람이라면(안타깝게도 일부 교육과정에서의 문과 출신자는 예외일 수 있다), 연속함수에서 최솟/최댓값을 구하기 위한 방법으로 미분을 쓸 수 있다고 배워왔을 것이다. 물론 이 케이스도 예외는 아니다. 가설 함수가 2차 이하의 간단한 함수고, 변수의 종류도 적다면 미분 후 방정식을 푸는 게 더 쉬울 수 있다. 하지만, 그런 경우는 잘 없다. 심지어 [5차 이상의 방정식을 풀어야 하거나 초월함수가 들어오게 되면 해를 구할 방법이 없을 수도 있다](https://ko.wikipedia.org/wiki/%EC%98%A4%EC%B0%A8_%EB%B0%A9%EC%A0%95%EC%8B%9D#5%EC%B0%A8%EB%B0%A9%EC%A0%95%EC%8B%9D%EC%9D%98_%EA%B7%BC). 따라서 우리는 어떤 경우에도 최솟값을 구할 수 있는 다른 방법을 찾아야 한다.

### Gradient Descent

다행스럽게도, 우리가 필요로 하는 건 완벽한 최솟값이 아니다. 주어진 데이터를 학습해서 새로 들어오는 데이터의 결과를 높은 정확도로 예측하면 되는데, 주어진 데이터를 오차 없이 완벽히 학습한다고 해서 새로 들어오는 데이터가 꼭 그 규칙을 따른다는 보장이 없기 때문에, 그냥 오차가 작기만 하면 된다.

따라서, 우리는 미분계수가 0이 되는 지점을 정확히 찾지 않고, 해당 지점을 향해 점점 가까워지는 방법만 취해도 충분하다. 이 방법이 바로 **경사 하강법(gradient descent)**이다.
$$
\theta_j := \theta_j - \alpha {{\partial} \over {\partial \theta_j}} J(\theta_0, \: \theta_1, \: \cdots , \: \theta_n)\quad\mathrm{for} \:\: j=0, \: j=1, \: \cdots , \: j=n
$$
식만 보면 복잡해보이는데, 별거 아니고 그냥 점 하나 구해서 접선 방향으로 긋겠다는 것이다. 이를 변수 종류만큼의 차원이 존재하므로 그 수만큼 편미분을 반복해서 weight 값을 업데이트하면 된다. 이 작업을 loss가 충분히 작아질 때까지 계속해서 반복한다.

여기서 헷갈리면 안된다. loss function을 편미분할 때는 $x$가 아닌 $\theta$로 편미분해야 한다. $x$는 입력값이고, 우리가 구해야 하는 건 hypothesis function이다. hypothesis function을 구한다는 건 각 단항의 계수를 구하는 것이므로 결국 $\theta$를 구해야 한다. 따라서 $\theta$로 편미분해야 한다. 헷갈리지 말자.

$:=$는 대입연산자다. 수많은 프로그래밍 언어에서 쓰는 `=`와 정확히 같은 뜻으로, 새로운 값으로 업데이트하라는 뜻이다.

$\alpha$는 **learning rate**이다. 한 번에 하강하는 보폭을 결정짓는데, 그냥 코드 짜는 사람이 적당한 값 넣어보면서 실험적으로 구해야 하는 값이다. 이 값이 너무 작으면 최솟값을 구하는데 몇백년이 걸리고, 너무 크면 아예 최솟값을 못 구하고 발산해버린다. 이유에 대해서는 이차함수 그래프 하나 그려놓고 곰곰이 생각해보길 바란다.