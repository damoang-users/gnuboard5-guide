---
description: 그누보드 hook 사용법
---
# Hook

그누보드5는 기능 확장 방법으로 Hook을 지원한다.

코드가 실행되는 중에 끼어들어 추가 동작을 실행하거나 값을 변경해주는 역할을 한다. **Event**와 **Replace** 두 가지 유형을 제공하고 둘의 방식은 같지만 Replace는 인자로 전달받은 값을 가공해서 반환하면 이후 동작에 반영된다.

Hook은 첫번째 인자에 사용되는 고유한 이름이 있으며 `Tag`로 부른다.  [Hook 목록](./hook-list)은 그누보드5 코어에 기본정의된 Tag 목록을 정리한 것이다.

::: info
Hook 목록은 따로 제공되고있지 않으며 그누보드 전체 파일에서 `run_event`와 `run_replace`를 검색해서 찾아볼 수 있다.
:::

::: details 그누보드 5.4 버전 이상에서만 지원
Hook 기능은 2019년에 배포된 그누보드 5.4 버전부터 지원하며 이전 버전에서는 Hook을 사용할 수 없다.

그누보드 5.4 이후 버전이라도 사용하려는 Hook Tag가 추가되지 않은 버전일 수 있다.
:::

## Hook 함수에 대해서

 그누보드5 코어에 정의된 [Hook 목록](./hook-list)의 Hook을 사용에대해 이해 하기 위해서는 자신만의 커스텀 Hook을 정의하고 사용해보면 이해하기 쉽다.

PHP Hook 시스템은 다른 여러 객체지향 프레임워크에서 지원하는 이벤트, 델리게이트를 사용한 Observer 패턴과 호환된다. 

* `add_`로 시작하는 Hook 함수호출은 이벤트 타입의 `변수`(그누보드에서는 Tag)에 콜백 핸들러(리스너)를 추가하는 즉,  구독 행위이며, 
* `run_`으로 시작하는 함수호출은 이벤트 변수 안에 저장된 핸들러들을 `invoke()` 하는 것과 같다. 

다만, PHP Hook의 Tag는 변수가 아닌 단순한 문자열로써 첫번째 인자로 선언하며 전달한다.

그누보드5의 PHP Hook 함수는 `lib/hook.lib.php`에 정의되어있으며, 접미사와 접두사를 통해 서로다른 두가지 분류로 나눌 수 있다.
* 접두사
  * `add_` : Tag(첫번째 인자)에 콜백 핸들러 등록
  * `run_` : Tag에 등록된 콜백 핸들러 실행
* 접미사
  * `_event` : 반환이 없는 Hook
  * `_replace` : 반환값이 있는 Hook

```php
add_event($tag, $func, $priority=G5_HOOK_DEFAULT_PRIORITY, $args=0)
run_event($tag, $arg = '')
add_replace($tag, $func, $priority=G5_HOOK_DEFAULT_PRIORITY, $args=1)
run_replace($tag, $arg = '')
delete_event($tag, $func, $priority=G5_HOOK_DEFAULT_PRIORITY)
delete_replace($tag, $func, $priority=G5_HOOK_DEFAULT_PRIORITY)
```

이것들을 이용해 새로운 Hook을 추가하거나 사용 할 수 있다. 

이것은 커스텀 Hook을 정의하는 사람과 그것을 사용하는 사람으로 나누어 생각해보자. 실제로 내가 정의한 Hook에 다른 플러그인 개발자가 자신의 핸들러를 등록할 수 있다.

### 커스텀 Hook 정의

자신의 커스텀 Hook은 `run_`접두어를 가진 Hook 함수로 Tag를 정의하고 실행(invoke)한다. 

~~~php
function my_saving_process($data, $user_id) {
    $context = [
        'data' => $data,
        'user_id' => $user_id,
    ];
    run_event('my_before_saving', $context);
    //do sometihg here to save data with $context
    run_event('my_after_saving', $context, 'additional_info');
}
~~~

첫번째로는 `my_before_saving` 라는 커스텀 Tag (Hook)를 지정하고 `run_event`를 호출했다. 이것은 `my_before_saving`에 등록된 핸들러(함수)들을 실행시키겠다는 의미이며 실행될 각각의 해들러에 `$context` 를 전달한다. 


커스텀 Hook의 정의와 실행은 이것으로 끝난다. 이렇게 `run_event`가 호출될 때 Tag에 등록된 핸들러가 없다면 아무일도 일어나지 않는다.

`run_event('my_before_saving', $context);`는 결국 아래 

```
my_before_saving($context);
```

처럼 여겨지고 `my_before_saving`는 함수를 담는 변수로, 등록된 모든 핸들러에 매개변수로 `$context`를 전달해 실행`()` 하는 것이다. 핸들러에 전할 인자가 여럿이라면 `my_after_saving` 의 경우처럼 매개변수를 추가하면된다.


### 커스텀 Hook에 핸들러 등록

위에 정의한 `run_event`가 `my_before_saving` Tag를 실행시킬 때 실제로 실행될 핸들러 함수는 `add_`로 등록한다. 

```php
`add_event(`my_before_saving`, `your_func`);

function your_func($your_context) {
  //do something
}
```
이것은 아래와 같은 의미로 생각하면된다.

```
$my_before_saving += your_func;
```

`$your_func`는 Hook을 사용하려는 자의 커스텀 핸들러 함수이다.  앞서 `my_before_saving` 태그를 정의한 예제에서 `run_even`가 실행될 때 `$context`가 `your_func`의 매개변수 `$your_context`에 전달되므로 보통은 매개변수 이름을 같게 짓는다. 그리고 위 예제상 전달받은 데이터는 다음배열이 되겠다:

~~~
[
    'data' => $data,
    'user_id' => $user_id,
];
~~~

위의 `add_event`와 `your_func` 핸들러 함수를 다시 작성 해보았다:

~~~php
add_event('my_before_saving', 'your_func');

function your_func($context) {
    $context['data'] = strtoupper($context['data']);
}
~~~

이렇게 `run_event`로 Tag를 실행할때 실행될 핸들러로 `your_func` 함수를를 등록했다. 

이렇게 실행될 핸들러는 반드시 함수 구현체가 있어야하는 것은 아니다. 간단한 동작이라면 `add_event`의 매개변수를 익명함수로 직접 작성해도된다.

```php
add_event('my_before_saving', 
           function($context) {
               $context['data'] = strtoupper($context['data']);
           }
         );
```



## Hook 사용하기 (리스너 등록)

Hook이 동작할 때 실행될 함수(콜백, Listener)를 등록해두면 해당 이벤트가 발생할 때 등록한 함수를 호출하여 실행한다.

Hook 유형에 따라 `add_event()`와 `add_replace()` 두 가지 함수를 사용하고, `add_replace()`는 반환하는 값이 결과에 반영된다는 것 외에는 같다.

```php
/**
 * Event Hook 리스너 등록
 *
 * @param string $tag Hook 이름
 * @param callable $callback 실행될 함수, 메소드, 클래스
 * @param int $priority 실행 우선 순위
 *            기본 값인 G5_HOOK_DEFAULT_PRIORITY는 int(8)이다
 * @param int $args 콜백에 전달될 인자의 갯수
 * @return void
 */
add_event($tag, $callback, $priority = G5_HOOK_DEFAULT_PRIORITY, $args = 0);

/**
 * Replace Hook 리스너 등록
 *
 * @param string $tag Hook 이름
 * @param callable $callback 실행될 함수, 메소드, 클래스
 * @param int $priority 실행 우선 순위
 *            기본 값인 G5_HOOK_DEFAULT_PRIORITY는 int(8)이다
 * @param int $args 콜백에 전달될 인자의 갯수
 * @return true|void
 */
add_replace($tag, $callback, $priority = G5_HOOK_DEFAULT_PRIORITY, $args = 0);
```

::: tip
네 번째 인자의 갯수 설정에 유의하자.

어떤 Hook은 인자로 아무것도 전달해주지 않을 수 있지만 동작에 필요한 데이터를 전달해주기도 한다.
사용하려는 Hook에서 전달하는 인자의 갯수를 확인하여 네번째 인자에 갯수를 지정해야 활용할 수 있다.
:::

### 함수를 리스너로 등록

```php
# 두 번째 인자에 실행할 함수명을 문자열로 지정
add_event('common_header', 'listenerCommonHeader');
function listenerCommonHeader()
{
    // ... 실행할 코드
    // Event Hook은 결과를 반환하지 않아도 된다
}
```

### 메소드를 리스너로 등록

메소드를 리스너로 등록할 때는 해당 메소드가 포함된 Class가 `getInstance()` 정적 메소드(static)를 포함해야한다. Hook은 `getInstance()`를 실행하여 객체를 반환받아 지정한 메소드를 실행한다.

::: details getInstance()
보통 Singleton 클래스를 작성하는 방법으로 `getInstance` 이름을 사용하여 클래스 자신의 인스턴스를 생성하여 반환한다.
:::

```php {1-7,22-27}
# 리스너 등록
add_replace(
    'board_content_head',
    ['MyHookListenerClass', 'listenerBoardContentHead'],
    G5_HOOK_DEFAULT_PRIORITY,
    1
);

class MyHookListenerClass
{
    private static self $instance;

    public static function getInstance(): self
    {
        if (!self::$instance) {
            self::$instance = new self();
        }

        return self::$instance;
    }

    # 실행될 메소드
    public function listenerBoardContentHead($arg1)
    {
        // ... 실행할 코드
        return $arg1;
    }
}
```

::: info
그누보드 5.5.8.3.2 버전<sup>2023.07.17 배포</sup>부터는 Closure 및 `ClassName::staticMethod` 형태의 문자열을 사용할 수 있다.
:::

::: warning
Hook이 실행될 때 마다 리스너로 등록한 메소드가 포함된 클래스의 `getInstance()`를 호출한다.

`getInstance()` 메소드가 없다면 Hook이 실행되면서 매번 새로운 인스턴스를 만든다. 2개 이상의 인스턴스가 생성될 수 있으며 다중 인스턴스로 인한 사이드 이펙트에 주의해야한다. Hook 리스너를 모아둔 개별 클래스를 따로 두는 것도 좋다.
:::

## 새로운 Hook 추가

자신이 만든 기능에 새로운 Hook을 정의할 수 있다.

그누보드 코드를 수정해야할 필요가 있을 때에도 새로운 Hook을 추가하여 코드 수정을 최소화하고 기능을 확장하는데 사용할 수도 있다.

::: details Hook 추가를 제안 해보자
새로 추가한 Hook이 다른 사람에게도 유용할 것 같다면 그누보드에 Hook을 추가할 것을 제안할 수도 있다.

제안하기: https://github.com/gnuboard/gnuboard5/issues
:::

Hook을 사용할 때와 마찬가지로 Event/Replace 두 유형을 사용할 수 있고 차이점도 같다.

```php
/**
 * Event Hook을 실행
 *
 * @param string $tag Hook 이름
 * @param ?mixed $arg 리스너에 전달될 데이터
 * @param ?mixed ...$args 리스너에 추가로 전달될 데이터
 *                        리스너에서 참고할 수 있는 부가 데이터를 전달할 수 있다
 * @return void
 */
run_event($tag, $arg = '', ...$args);

/**
 * Replace Hook을 실행
 *
 * @param string $tag Hook 이름
 * @param ?mixed $arg 리스너에 전달될 데이터
 *               리스너가 아무것도 반환하지 않는다면 이 인자의 값을 그대로 반환한다
 * @param ?mixed ...$args 리스너에 추가로 전달될 데이터
 *                        리스너에서 참고할 수 있는 부가 데이터를 전달할 수 있다
 * @return mixed
 */
run_replace($tag, $arg = '', ...$args);
```

아래와 같이 활용할 수 있다.

Replace Hook은 리스너에서 반환하는 값을 받아 이후 코드 실행에서 사용하도록 한다.

```php
# Event Hook 실행
$data = array('default_data');
$options = array('mb_id' => 'admin', 'bo_table' => 'free');
run_event('my_first_event_hook', $data, $options);

# Replace Hook 실행
// 리스너가 없거나 아무것도 반환하지 않는다면 $data에는 'default_data'가 그대로 들어있다
$data = 'default_data';
$data = run_event('my_first_replace_hook', $data);
```

::: warning
그누보드는 DB 트랜젝션을 이용하지 않으므로 Hook 리스너에서는 오류 등으로 의도하지 않게 동작이 멈추지 않도록 주의해야 한다.
:::