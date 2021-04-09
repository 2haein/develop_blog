---
title: "React의 장점 (논외: Vue.js와의 비교)"
date: 2021-04-01T15:00:00.000Z
description: "두 프론트엔드 라이브러리 중 React"
categories: react vue
tags:
  - javascript
  - react
toc: true
toc_sticky: true
excerpt: "Strength of React"
---

## 도입

회사에서 주로 SPA 기반의 웹 프론트엔드를 개발하였다. 개발은 [Vue.js](https://vuejs.org/)를 메인으로 [Angular.js](https://angularjs.org/)를 서브 기술로 사용하고 그 결과 내린 결론이 있는데, **선택권이 있는 상황에선 무조건 React를 사용하겠다**는 것이다.

이런 입장을 갖게 된 이유를 한 번 글로 정리해두면 편하기도 하고, 도움이 될 수 있겠다는 생각이 들어서 이 글을 적게 되었다.

React를 Vue.js보다 선호하는 이유는 크게 세 가지 정도로 정리할 수 있다.

- TS 지원
- 컴포넌트 용의함
- 빠른 개선

하나씩 살펴보자.

## TS 지원

이 항목은 React의 비교우위가 명백하다. React와 TS의 결합은 아주 매끄럽다. 큰 BP 없이도 SFC와 클래스 기반 컴포넌트 둘 모두의 타입을 정확하게 기술할 수 있고, [redux](https://github.com/reactjs/redux)를 비롯한 툴도 대부분 훌륭한 타입 지원을 제공한다.

반면 Vue.js는 2.5 버전 업데이트 당시 TS 지원의 큰 개선을 있지만 아직까지 많이 미흡하다. `computed`나 `methods` 내의 `this` 타입은 제대로 추론되지 않는다. [vue-class-component](https://github.com/vuejs/vue-class-component)를 사용하면 상황이 좀 나아지지만 이는 아직 실험 기능이고 문법과 이질적이다. [vuex](https://github.com/vuejs/vuex)의 타입 지원 또한 redux와 비슷한 수준이 되기는 아직 멀었다.

TS의 필요성에 공감하지 못하거나 도입이 너무 어렵고 사용하지 않는 사람들을 꽤 보았는데, 그런 이에겐 이 항목은 별로 해당 사항이 없다.

## 컴포넌트 정의 용이함

Vue.js는 컴포넌트에 관한 템플릿, 스타일과 스크립트를 `.vue` 확장자를 갖는 한 파일 내에 모두 작성할 수 있는 [단일 파일 컴포넌트(Single File Component)](https://vuejs.org/v2/guide/single-file-components.html)를 제공한다. 이를 사용해 허구의 `UserList` 컴포넌트를 작성하는 경우를 생각해보자.

```html
<template>
  <ul :id="$style.userList">
    <li
      v-for="user in users"
      :key="user.id"
      :class="{
        [$style.userItem]: true,
        [$style.selected]: user.id === selectedUserId
      }"
    >
      {{ user.name }}
    </li>
  </ul>
</template>

<script>
  export default {
    data() {
      return { selectedUserId: undefined };
    },
    props: ["users"],
  };
</script>

<style module>
  /* style definition */
</style>
```

React를 사용한다면 이 컴포넌트를 대략 아래와 같이 작성할 것이다.

```jsx
import React, { Component } from 'react'
import classNames from 'classnames'
import * as styles from './UserList.css'

const UserItem = ({ user, selected }) => (
  <li className={classNames(style.userItem, { [style.selected]: selected })}>
    { user.name }
  </li>
)

export default class UserList extends Component {
  constructor(props) {
    super(props)
    this.state = { selectedUserId: undefined }
  }

  render() {
    const { users } = this.props
    const { selectedUserId } = this.state

    return (
      <ul className={styles.userList}>
        {users.map(user => (
          <li className={classNames(styles.userItem, { [styles.selected]: user.id === selectedUserId })}>
            { user.name }
          </li>
        )}
      </ul>
    )
  }
}
```

이 시점에서는 비교해보면, Vue.js에 비해 React의 문법이 갖는 몇 가지 단점이 눈에 띈다.

- [styled-components](https://github.com/styled-components/styled-components)등을 사용하지 않는 이상 컴포넌트의 스타일시트를 별도의 파일에 작성해야 한다.
- 상태를 갖는 컴포넌트를 정의할 때 필요한 BP가 크다.

하지만 이 파일 내에서만 사용 될 `UserItem` 컴포넌트를 별도의 컴포넌트로 빼는 경우를 생각해보자.

Vue.js에서는 두 가지 선택지가 있다. 하나는 별도의 `UserItem.vue`를 만드는 것이다.

```html
<template>
  <ul :id="$style.userList">
    <user-item
      v-for="user in users"
      :key="user.id"
      :user="user"
      :selected="user.id === selectedUserId"
    />
  </ul>
</template>

<script>
  import UserItem from "./UserItem.vue";

  export default {
    components() {
      UserItem;
    },
    data() {
      return { selectedUserId: undefined };
    },
    props: ["users"],
  };
</script>

<style module>
  /* style definition */
</style>
```

또는 파일 내에서 `ComponentOption` 객체를 정의 할 수 있다.

```html
<template>
  <ul :id="$style.userList">
    <user-item
      v-for="user in users"
      :key="user.id"
      :user="user"
      :selected="user.id === selectedUserId"
    />
  </ul>
</template>

<script>
  const UserItem = {
    template:
      '<li :class="{ [styles.userItem]: true, [styles.selected]: selected }">{{ user.name }}</li>',
    props: ["user", "selected"],
  };

  export default {
    components() {
      UserItem;
    },
    data() {
      return { selectedUserId: undefined };
    },
    props: {
      users: {
        type: Array,
        default: [],
      },
    },
  };
</script>

<style module>
  /* style definition */
</style>
```

이러한 두 가지 방법은 각각의 단점을 갖고 있다.

먼저 단일 파일 컴포넌트를 사용한 접근의 경우, 한 군데에서만 사용될 작은 컴포넌트를 정의할 때에도 무조건 새 파일을 만들어야 한다. 이는 보일러플레이트의 증가로 이어진다.

한 편, `ComponentOptions`를 사용한 접근의 경우, 템플릿을 플레인 문자열로 표현하는 탓에 많은 정보를 잃게 된다. `template` 대신 `render` 함수를 사용하고 그 안에서 JSX를 사용할 수 있지만, Vue.js가 권장하는 방식은 아니다.

React의 SFC(Stateless Functional Component)를 사용하면 같은 작업을 아래와 같이 해낼 수 있다.

```jsx
import React, { Component } from 'react'
import classNames from 'classnames'
import * as styles from './UserList.css'

const UserItem = ({ user, selected }) => (
  <li className={classNames(style.userItem, { [style.selected]: selected })}>
    { user.name }
  </li>
)

export default class UserList extends Component {
  render() {
    const { users } = this.props
    const { selectedUserId } = this.state

    return (
      <ul className={styles.userList}>
        {users.map(user => (
          <UserItem user={user} selected={this.selectedUserId === user.id} />
        )}
      </ul>
    )
  }
}
```

`UserItem` 컴포넌트가 부모가 던져준 데이터에 의존하는 함수라는 점이 코드 자체에서 드러난다.

컴포넌트를 잘게 쪼갤 때 React는 강점을 갖는다.

- 작은 컴포넌트를 정의하는 문법이 직관적이고 간결하다.
- 컴포넌트 정의하는 두 문법(SFC, 클래스 기반 컴포넌트)이 더 일관적이다.

## 빠른 개선

React에는 새로운 Context API, 라이플사이클 메소드 등 [React Blog](https://reactjs.org/blog/)에 많은 변경사항이 있다.

반면 Vue.js의 Release는 주로 마이너한 변경사항 위주인 경우가 많다. [현존하는 두 라이브러리 사이의 간극](https://stateofjs.com/2017/front-end/results)

### 마지막으로

- Vue.js는 사용자에게 쉽게 느껴지는 API를 제공하기 위해 라이브러리가 직접 헤비 리프팅을 하는 경우가 많다.
- React는 Vue.js에 비해 더 적은 가정을 하고, 컴포넌트 기반의 선언적 UI 렌더링이라는 가장 핵심적인 기능과 관련된 부분만 코어에 포함한다.

끝으로, 글의 목적 상 React.js의 장점을 더 많이 언급했지만 Vue.js 역시 분명 좋은 도구다.

1. 자바스크립트에 익숙하지 않은 팀원이 많이 참여하는 2. 소규모의 프로젝트라면 Vue.js도 목적에 맞는 좋은 프레임워크이다.
