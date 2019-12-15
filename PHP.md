# PHP 기말고사 시험 정리

### 작성자 : 박윤재

### 작성일 : 2019-12-15



## 목차

- 230p - 7-1 예제 기본예제
- 235p - 회원가입페이지 
- 240p - 
- 247p
- 249p 
- 260p 
- 276p
- 282p 세션1.php
- 295p  -안에 인클루드
- 304p 
- 314p 
- 312p
- 331p
- 333p



## 230p API 함수를 이용한 레코드 삽입

```php
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<?
    $connect = mysql_connect("localhost", "kdhong", "1234");
	mysql_select_db("kdhong_db", $connect);

	$sql = "insert into biz_card (num, name, company, tel, hp, address)";
	$sql.= " values (1, '원선우', '미래전자', '031-276-1829',";
	$sql.= " '010-8723-2837', '경기도 용인시 신갈동 388-23 번지')";

	$result = mysql_query($sql);

	if($result)
        echo "레코드 삽입 완료!";
	else
        echo "레코드 삽입 실패! 에러 확인 요망!"
        
    mysql_close($connect);
?>
```

### 중요한 함수와 내용

1. `mysql_connect() 함수` : DB를 연결하는 함수(명령어)입니다. 

   첫번째 인자 `localhost` : 주소를 의미 (IP주소로 127.0.0.1) , <feat 외울필요는 없지만 알아두면 좋음)

   두번째 인자 `kdhong` : 데이터베이스 접속 아이디입니다.

   세번째 인자 `1234` : 데이터베이스 접속 비밀번호입니다.

   그리고 `$connect`라는 변수에 접속 정보를 담습니다.

2. `mysql_select_db` : 데이터베이스(DB)에 접속을 하면 DB를 사용하기 위해 DB를 선택해야합니다. DB선택을 하기 위해서는 mysql_select_db()함수를 사용합니다. mysql은 (데이터베이스 안에 테이블) 구조로 이루어져 있습니다.

   첫번째 인자 `kdhong_db` : 데이터베이스명이 들어갑니다.

   두번째 인자 `$connect` : 위에서 본 DB접속 정조가 담긴 $connect 변수를 적어줍니다. 이 $connect 변수에 DB접속 주소와 아이디, 비밀번호 정보가 있어서 DB접속을 하고 해당 DB를 찾아 사용하기 위해 선택하는 과정입니다.

3. 레코드를 삽입할 때 사용하는 SQL문은 Insert구문 사용법

   ```c
   $sql = "insert into biz_card (num, name, company, tel, hp, address)";
   $sql.= " values (1, '원선우', '미래전자', '031-276-1829',";
   $sql.= " '010-8723-2837', '경기도 용인시 신갈동 388-23 번지')";
   ```

   위 코드는 biz_card라는 테이블안에 각각의 `(num, name, company, tel, hp, address)` 테이블 컬럼들에 맞게 `values (1, '원선우', '미래전자', '031-276-1829','010-8723-2837', '경기도 용인시 신갈동 388-23 번지')` 이렇게 값을 넣어준 것입니다. 

4. `mysql_query() 함수` : 다른 형식의 SQL 구문, INSERT, UPDATE, DELETE, DROP 등에서 성공하면 TRUE를, 실패하면 FALSE를 반환합니다.

   ```c
   $result = mysql_query($sql);
   if($result)
           echo "레코드 삽입 완료!";
   	else
           echo "레코드 삽입 실패! 에러 확인 요망!"
           
   mysql_close($connect);
   ```

   위 코드를 해석하면 mysql 쿼리문을 `mysql_query($sql)` 통해서 실행하고 실행된 결과 true, false를 $result에 담아줍니다. if문에서 true이면 삽인 완료 텍스트가 뜨고 실패시 에러 확인 요망 텍스트 출력!



## 235p 회원가입 페이지 생성

```html
<html>
    <head>
        <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
    </head>
    <body>
        <h2>▶ 회원가입</h2>
        <form name="mem_form" method="post" action="mem_print.php">
            <input type="hidden" name="title" value="회원가입 양식">
            <table border="1" width="640" cellspacing="1" cellpadding="4">
                <tr>
                	<td align="right">* 아이디 :</td>
                    <td><input type="text" size="15" maxlength="12" name="id" value="guest"</td>
                </tr>
                ....
            </table>
        </form>
    </body>
</html>
```

### 다쓰기엔 시간이 많이 걸리고 코드가 길어져서 생략!

### 중요한 태그와 기능들

1. `<form method="post">` : method의 타입은 2개가 있는데 Get과 Post입니다. Get방식은 주소창을 이용해 값을 전달합니다.

   예시

   ![](https://www.codingfactory.net/wp-content/uploads/php-get-post-02.png)

   만약 `Post` 타입으로 보내게 되면

   ![](https://i.imgur.com/D14VcjC.png)

   이런 형식으로 form태그만으로 주소창에 데이터나 변수 없이 데이터를 보내게 됩니다.

2. `<form action="url주소">` : action은 form태그안에 있는 input태그들의 값들을 전송할 페이지의 주소값을 가지고 있습니다. 한마디로 다음페이지의 주소값을 가지고 있다고 생각하면됨

3. `input` 태그의 속성들 : 

   ​	`input type="text" >` : 텍스트 값을 넣을 수 있음, 

   ​	`<input type="hidden>"` input태그 자체가 웹페이지에 보이지 않도록하지만 값을 form태그로 전송할 때는 보낸다. 	한마디	로 우리눈에 안보이는 스파이 존재(보안에 활용을 자주함)

   ​	`<input name="">` : name에 들어가는 값들은 추후에 form태그로 보낸값들을 받을 때 사용함

   ​	`<input type="password>"` password를 쓸 때 사용. `****` 같이 출력되지만 값들이 감춰지는 것임

   ​	`<input type="radio>"`  

   ​	![](https://i.stack.imgur.com/Ngv2E.png)

   ​	radio는 한가지만 선택가능함

   ​	`<select>` 태그

   ```html
   <select>
       <option>옵션 1</option>
   </select>
   ```

   ![](https://i.stack.imgur.com/ssyGR.jpg)

   ​	`<input type="checkbox>"`  체크박스는 라디오 버튼과 다르게 중복 선택 가능

   ​	![](https://dzone.com/storage/temp/10361090-3.png)

   `<textarea>` : 여러행의 텍스트를 입력할 때 사용, (댓글이나 게시글 적을 때 많이 사용)

   ![](https://i.stack.imgur.com/9LsrC.png)

   `<input type="submit" value="Submit">` : form태그에서 버튼역할을 하게 되며 클릭시 action속성에 설정된 url을 통해 페이지로 값을 전달함

   ![](https://media.geeksforgeeks.org/wp-content/uploads/20190529140659/html-input-type-submit.png)

   `<input type="reset">` : form태그에서 사용자가 입력한 input태그의 값을 모두 초기화 시켜주는 버튼