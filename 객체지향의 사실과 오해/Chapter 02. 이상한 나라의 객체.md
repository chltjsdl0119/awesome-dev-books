# Chapter 02. 이상한 나라의 객체

> 객체지향 패러다임은 지식을 추상화하고 추상화한 지식을 객체 안에 캡슐화함으로써 실세계 문제에 내재된 복잡성을 관리하려고 한다. 
> 객체를 발견하고 창조하는 것은 지식과 행동을 구조화하는 문제다.

## 객체지향과 인지 능력

- 객체지향은 세상을 자율적이고 독립적인 객체들로 분해할 수 있는 인간의 기본적인 인지 능력에 기반을 두고 있다.
- 인간의 인지 능력은 물리적인 한계를 넘어 개념적으로 경계 지을 수 있는 추상적인 사물까지도 객체로 인식할 수 있게 한다.
- 세상을 더 작은 객체로 분해하는 것은 본질적으로 세상이 포함하고 있는 복잡성을 극복하기 위한 인간의 작은 몸부림이다.

> 객체란 인간이 분명하게 인지하고 구별할 수 있는 물리적인 또는 개념적인 경계를 지닌 어떤 것이다.

> 객체지향 패러다임의 목적은 현실 세계를 모방하는 것이 아니라 현실 세계를 기반으로 새로운 세계를 창조하는 것이다.

## 객체 그리고 이상한 나라

### 이상한 나라의 앨리스

- 이상한 나라의 앨리스의 줄거리가 간략하게 나온다.

### 앨리스 객체

> 객체의 상태를 결정하는 것은 행동이지만 행동의 결과를 결정하는 것은 상태이다.

- 객체는 어떤 상태에 있더라도 유일하게 식별 가능하다.

## 객체, 그리고 소프트웨어 나라

- 하나의 개별적인 실체로 식별 가능한 물리적인 또는 개념적인 사물은 어떤 것이라도 객체가 될 수 있다.
- 객체의 다양한 특성을 효과적으로 설명하기 위해서는 객체를 **상태(state)**, **행동(behavior)**, **식별자(identity)** 를 지닌 실체로 보는 것이 가장 효과적이다.

> 객체란 식별 가능한 개체 또는 사물이다. 자동차처럼 만질 수 있는 구체적인 사물일 수도 있고, 시간처럼 추상적인 개념일 수도 있다.
> 객체는 구별 가능한 식별자, 특징적인 행동, 변경 가능한 상태를 가진다.

### 상태

#### 왜 상태가 필요한가

- 어떤 행동의 결과는 과거에 어떤 행동들이 일어났었느냐에 의존한다. 이는 과거에 행동을 기억해야만 하기 때문에 행동의 결과를 설명하는 것을 매우 어렵게 만든다.
- 상태를 이용하면 과거에 얽매이지 않고 현재를 기반으로 객체의 행동 방식을 이해할 수 있다.
- 상태는 근본적으로 세상의 복잡성을 완화하고 인지 과부하를 줄일 수 있는 중요한 개념이다.

#### 상태와 프로퍼티

- 단순한 값은 객체가 아니지만 객체의 상태를 표현하기 위한 중요한 수단이다.
- 모든 객체의 상태는 단순한 값과 객체의 조합으로 표현할 수 있다.

> **프로퍼티**
> - 객체의 상태를 구성하는 모든 특징을 통틀어 객체의 **프로퍼티(property)** 라고 한다.
> - 일반적으로 프로퍼티는 변경되지 않고 고정되기 때문에 '정적'이다.
> - **프로퍼티 값(property value)** 은 시간이 흐름에 따라 변경되기 때문에 '동적'이다.

- 객체와 객체 사이의 의미 있는 연결을 **링크(link)** 라고 하고, 링크가 존재해야만 메시지를 주고 받을 수 있다.
- 객체를 구성하는 단순한 값은 **속성(attribute)** 이라고 한다.

> 객체는 특정 시점에 객체가 가지고 있는 정보의 집합으로 객체의 구조적 특징을 표현한다. 객체의 상태는 객체에 존재하는 정적인 프로퍼티와 동적인 프로퍼티 값으로 구성된다. 객체의 프로퍼티는 단순한 값과 다른 객체를 참조하는 링크로 구분할 수 있다.

- 자율적인 객체는 스스로 자신의 상태를 책임져야 한다.

### 행동

#### 상태와 행동

- 객체의 행동은 상태에 영향을 받는다.
- 객체의 행동은 상태를 변경시킨다.

#### 협력과 행동

- 객체는 다른 객체와 메시지를 통해서만 의사소통 할 수 있다.

> 행동이란 외부의 요청 또는 수신된 메시지에 응답하기 위해 동작하고 반응하는 활동이다. 행동의 결과로 객체는 자신의 상태를 변경하거나 다른 객체에게 메시지를 전달할 수 있다. 객체는 행동을 통해 다른 객체와의 협력에 참여하므로 행동은 외부에 가시적이어야 한다.

#### 상태 캡슐화

- 객체는 상태를 캡슐 안에 감춰둔 채 외부로 노출하지 않는다. 객체가 외부에 노출하는 것은 행동 뿐이며, 외부에서 객체에 접근할 수 있는 유일한 방법 역시 행동 뿐이다.
- 상태를 잘 정의된 행동 집합 뒤로 캡슐화하는 것은 객체의 자율성을 높이고 협력을 단순하고 유연하게 만든다.

### 식별자

- 상태를 이용해 두 값이 같은지 판단할 수 있는 성질을 **동등성(equality)** 이라고 한다.
- 객체는 시간에 따라 변경되는 상태를 포함하며, 행동을 통해 상태를 변경한다. 따라서 객체는 **가변 상태(mutable state)** 를 가진다.
- 식별자를 기반으로 객체가 같은지를 판단할 수 있는 성질을 **동일성(identical)** 이라고 한다.
- 상태가 **가변적인** 객체의 동일성을 판단하기 위해서는 상태 변경에 독립적인 별도의 식별자를 이용할 수 밖에 없다.

> 식별자란 어떤 객체를 다른 객체와 구분하는 데 사용하는 객체의 프로퍼티다.

- **참조 객체(reference object)**, 또는 **엔티티(entity)** 는 식별자를 지닌 전통적인 의미의 객체를 가리키는 용어다.
- **값 객체(value object)** 는 식별자를 가지지 않는 값을 가리키는 용어다.

## 기계로서의 객체

- 객체의 상태를 조회하는 작업을 **쿼리(query)**, 객체의 상태를 변경하는 작업을 **명령(command)** 이라고 한다.
- 기계 은유의 가장 큰 장점은 객체 캡슐화를 직관적이고 시각적으로 묘사한다.
- 객체를 기계로서 바로보는 관점은 상태, 행동, 식별자에 대한 시각적인 이미지를 제공하고 캡슐화와 메시지를 통한 협력 관계를 매우 효과적으로 설명한다.

## 행동이 상태를 결정한다

- 상태를 먼저 결정하고 행동을 나중에 결정하는 방법은 설계에 나쁜 영향을 끼친다.

1. 상태를 먼저 결정할 경우 캡슐화가 저해된다.
2. 객체를 협력자가 아닌 고립된 섬으로 만든다.
3. 객체의 재사용성이 저해된다.

- 설계자로서 우리는 협력의 문맥에 맞는 적절한 행동을 수행하는 객체를 발견하거나 창조해야 한다.
- 협력 안에서 객체의 행동은 결국 객체가 협력에 참여하면서 완수해야 하는 책임을 의미한다.

## 은유와 객체

### 두 번째 도시전설

- 추상화란 실제의 사물에서 자신이 원하는 특성만 취하고 필요 없는 부분을 추려 핵심만 표현하는 행위를 말한다.

### 의인화

- 레베카 워프스브록은 현실의 객체보다 더 많은 일을 할 수 있는 소프트웨어 객체의 특징을 **의인화(anthropomorphism)** 라고 부른다.
- 의인화의 관점에서 소프트웨어를 생물로 생각하자. 모든 생물처럼 소프트웨어는 태어나고, 삶을 영위하고, 그리고 죽는다.
- 객체지향의 세계의 거리는 현실 속의 객체보다 더 많은 특징과 능력을 보유한 객체들로 넘쳐난다.

### 은유

- 현실 세계와 객체지향 세계 사이의 관계를 좀 더 정확하게 설명할 수 있는 단어는 **은유(metaphor)** 다.
- 은유는 하나의 의미를 다른 것을 이용해 전달한다는 의미를 가진다.

### 이상한 나라를 창조하라

- 우리의 목적은 현실을 모방하는 것이 아니다. 단지 이상한 나라를 창조하기만 하면 된다.