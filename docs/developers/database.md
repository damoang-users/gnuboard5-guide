# Database

DB 질의를 위한 몇 가지 헬퍼 함수로 MySQL/MySQLi API를 사용하여 MySQL 및 MariaDB를 지원한다.

::: warning
그누보드는 트랜잭션을 사용하지 않으며 Prepared Statement를 사용하지 않는다. 항상 SQL Injection에 주의해야 한다.
:::

## 데이터 가져오기

### sql_query()

질의문의 실행 후 `mysqli_result` 객체를 반환(실패 시 `null`, `boolean`) 한다.

```php
/**
 * @param string $sql 질의문
 * @param bool $error 오류 표시 여부
 * @param ?mysqli $link MySQLi 객체
 * @return null|bool|mysqli_result
 */
function sql_query($sql, $error = G5_DISPLAY_SQL_ERROR, $link = null);
```

```php
$result = sql_query("SELECT * FROM {$g5['member_table']} WHERE mb_id = 'admin'");
$num_rows = sql_num_rows($result);
$data = sql_fetch_array($result);
```

### sql_fetch()

```php
/**
 * @param string $sql 질의문
 * @param bool $error 오류 표시 여부
 * @param ?mysqli $link MySQLi 객체
 * @return mixed
 */
function sql_fetch($sql, $error = G5_DISPLAY_SQL_ERROR, $link = null);
```

### sql_fetch_array()

```php
/**
 * @param mysqli_result $result
 * @return mixed
 */
function sql_fetch_array($result);
```

### sql_data_seek()

```php
/**
 * @param mysqli_result $result
 * @param int $offset
 */
function sql_data_seek($result, $offset = 0);
```

### sql_free_result()

```php
/**
 * @param mysqli_result $result
 */
function sql_free_result($result);
```

### sql_insert_id()

```php
/**
 * @param ?mysqli $link
 */
function sql_insert_id($link = null);
```

### sql_num_rows()

```php
/**
 * @param mysqli_result $result
 */
function sql_num_rows($result);
```

## DB 연결

### sql_connect()

```php
/**
 * @param string $host
 * @param string $user
 * @param string $pass
 * @param string $db
 */
function sql_connect($host, $user, $pass, $db = G5_MYSQL_DB);
```

### sql_select_db()

```php
/**
 * @param string $db
 * @param mysqli $connect
 */
function sql_select_db($db, $connect);
```

### sql_set_charset()

```php
/**
 * @param string $charset
 * @param ?mysqli $link
 */
function sql_set_charset($charset, $link=null);
```

## 기타

### sql_password()

::: warning 폐기된 함수

이 함수는 더 이상 사용되지 않아야 한다. 취약한 알고리즘을 사용하고 있으며, MySQL 8.0.11 버전부터는 오류를 발생시킨다.

MySQL PASSORD 함수로 생성된 기존 비밀번호를 비교해야 할 때만 사용되어야하며, 새로운 비밀번호 등을 이 함수로 생성하여 사용하지 않아야 한다.

::: danger
이 함수는 MySQL 8.0.11 버전부터 완전히 제거된 기능에 의존하므로 오류를 발생시킨다. 이 문제는 보고되었지만 현재 고쳐지지 않고 있다. [그누보드 이슈 247](https://github.com/gnuboard/gnuboard5/issues/247)
:::

MySQL PASSWORD() 함수로 생성된 비밀번호의 hash 값을 반환 한다. MySQL 버전에 따라 다른 결과 값을 반환할 수 있다.

```php
/**
 * @param string $value
 * @return string
 */
function sql_password($value);
```

### sql_field_names()

```php
/**
 * @param string $table
 * @param ?mysqli $link
 */
function sql_field_names($table, $link = null);
```

### sql_error_info()

```php
/**
 * @param ?mysqli $link
 */
function sql_error_info($link = null);
```

### sql_real_escape_string()

```php
/**
 * @param string $str
 * @param ?mysqli $link
 */
function sql_real_escape_string($str, $link = null);
```

::: danger
TBD. SQL Injection
:::
