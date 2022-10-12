# 2022-10

## 10/12

---

## Lazy Loading 과 Eager Loading (feat. Sequelize)

> ## 개념
>
> ### Eager Loading
>
> ```
> 접근하고자 하는 모델에 접근할때, 필요한 모델을 include(join)하여 한번에 불러오는 방식
> ```
>
> ### Lazy Loading
>
> ```
> 접근하고자 하는 모델에 먼저 접근하여 데이터를 불러오고, 거기와 관련된 모델은 필요할 때 불러오는 방식
> ```

## Eager Loading

전까지 사용했던 방식이다. 어떤 데이터를 불러올때, 필요할 것으로 생각 또는 예상되는 모든 모델을 include하여 가져온다.

```js
const order = await Main.Order.findOne({
    attributes: [~~],
    where: {
        ~~
    },
    include: [
        {
            model: Main.User,
            attributes: [~]
        },
        {
            model: Main.PaymentMethod,
            attributes: [~],
            where: {
                ~
            }
        }
    ]
})
```

> ### 💡 include의 옵션
>
> ```js
> required: true;
> ```
>
> include한 모델 안에서 required 옵션을 줄 수 있는데, 이는 inner join을 수행한다.

이러한 Eager Loading 방법은 한번의 쿼리문으로 원하는 데이터를 가져올 수 있다. 이것은 N+1 문제를 해결할 수 있다고 한다.

> ### 💡 N+1 문제
>
> 한번의 쿼리로 N건의 데이터를 가져온 후 원하는 데이터를 얻기 위해 N건의 데이터 수 만큼 2차쿼리를 날리는 것. ORM 성능저하를 일으키는 대표적인 이슈라고 한다.

하지만 이러한 Eager Loading으로 사용하다 보니 여러 불편한 점이 생겼다.

### **include를 많이 하게 되면 코드가 길어지고 그것을 리팩토링하기도 애매해진다.**

---

위의 예시처럼 2개 정도 밖에 없으면 문제가 없는데, 막 5개씩 모델을 include하게 되면 코드가 길어질 뿐만 아니라 가독성도 떨어지고 이를 리팩토링하기 위해서 엄청 머리를 싸맨적이 있다.

### **중첩으로 include를 하게 되면, 의도치 않은 결과를 불러온다.**

---

include안에 include안에 include를 하는 경우가 있었는데, 결과를 본 순간 이건 뭐지? 하는 순간이 있었다. 동료들과 찾아보니, sequelize가 쿼리를 뜻하는대로 생성을 못하는 경우였다. 그 경우 `subQuery: false` 옵션을 주어서 해결을 한 경우가 있었다.

### **변수 관리가 어렵다?**

---

개인 취향일수도 있지만, order.a.b.c 이런식으로 변수를 사용하는것이 보기가 좋지는 않았다.

## Lazy Loading

이번에 성재님이 models의 relation을 정리하시면 mixin을 이용해 Lazy Loading을 사용할 수 있게끔 기반을 닦아 놓으셨다.  
위의 방식을 Lazy Loading으로 바꾸면 아래와 같은 코드가 된다.

```js
const order = await Main.Order.findOne({
    attributes: [~~],
    where: {
        ~~
    },
})

const user = await order.getUser()
const paymentMethod = await order.getPaymentMethod()
```

Lazy Loading을 이용하게 되면 위에 말했던 불편함을 대부분 해소 할 수 있을거 같다.  
특히, 길었던 include를 더이상 안볼 수 있을거 같다.  
그 외에도 Eager Loading을 사용하게 되면, 데이터가 필요하지 않은 시점인데도 한번에 가져오기 때문에 메모리 관리에 조금더 유용하다고 한다.

물론, 무조건적으로 Lazy Loading을 사용할 수는 없을 것이다. N+1 문제 등이 일어날 수 있기 때문이다. 상황에 맞게 사용하면 좋을것 같다.
