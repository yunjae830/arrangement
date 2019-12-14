## 동양미래대학교 H반 기말고사 소스코드 설명

### 작성자 : 박윤재, 김현준

### 작성일 : 2019.12.14



## 목차

- ### 간략한 주요 소스코드 설명 [이동](#이동)

- ### 전체 소스코드와 주석 [이동](#이동2)



## 간략한 주요 소스코드 설명

#이동

### 사용한 변수와 라이브러리들

```c
#include "U8glib.h"

U8GLIB_SSD1306_128X64 u8g(U8G_I2C_OPT_NO_ACK);
#include<Servo.h> //Servo 라이브러리를 추가

#include <TM1637Display.h> //segment
#define CLK 5
#define DIO 6
TM1637Display display(CLK, DIO); // clk, dio 셋업
int Sec1, Sec2, Min1, Min2, delays = 100; //세그먼트 일의자리 초, 일의자리 초, 일의 자리 분, 십의자리 분 
uint8_t data[] = { 0x0, 0x0, 0x0, 0x0 }; // 초기값으로 00:00 셋팅
uint8_t segto;

#define KEY_NONE 0 //초기값
#define KEY_PREV 1 
#define KEY_NEXT 2 //OLED 메뉴 이동 관련 변수
#define KEY_SELECT 3 // OLED 선택 관련 변수
#define KEY_BACK 4
#define KEY_POWER 5 //OLED 전원 관련 변수

#define MENU_ITEMS1 3  // Start클릭시 들어가는 서브메뉴 (easy, normal, hard)의 크기 3
#define MENU_ITEMS2 2 // 맨 처음 시작 메뉴 (Start, End)의 크기 2

const char *menu_strings1[MENU_ITEMS1] = { "Easy", "Normal", "Hard" }; // OLED 메뉴에 들어갈 값 선언
const char *menu_strings2[MENU_ITEMS2] = { "Start", "End" }; 
uint8_t menu_current = 0; // OLED 메뉴에서 NEXT 버튼을 클릭했을 때 이동을 했다는 것을 명시하기 위한 변수 (메뉴 이동을 위한 변수)
uint8_t menu_redraw_required = 0; // 메뉴 업데이트시 확인하기 위해, 화면을 빠져나오기 위한 대책
uint8_t last_key_code = KEY_NONE; // 키 초기화 (변수 이름대로 마지막 이벤트가 발생했던 버튼(키)를 저장하기 위한 변수

uint8_t uiKeyNext = 3; // 버튼기능 -> OLED 메뉴 이동
uint8_t uiKeySelect = 2; // 버튼기능 -> OLED 메뉴 선택
uint8_t uiKeyPower = 4; // 버튼기능 -> OLED (Start, End) 메뉴

//초기화
uint8_t uiKeyCodeFirst = KEY_NONE; 
uint8_t uiKeyCodeSecond = KEY_NONE;
uint8_t uiKeyCode = KEY_NONE;
int page = 4; // OLED를 변수로 페이지관리, 페이지 초기화
double largPos = 0.0;
double maxTime = 1.0; //세그먼트
int remenu = 0; //루프에서 다시 OLED로 갈지 결정하는 변수
unsigned long currentTime1 = 150; // 0.8 : < 난이도 별로 초단위로 시간을 결정했고 OLED와 세그먼트를 맞추기 위해 일련의 작업을 거침 >
unsigned long currentTime2 = 100; // 1.2 : < 세그먼트가 지금처럼 100초일 때 OLED를 그대로 똑같이 실행하면 좀더 늦게 끝난다. 그래서 + 1.2 씩 prograssbar를 늘려주었다. >
unsigned long currentTime3 = 60; // 2.0
int segment_check = 0; //세그먼트 시작전 체크 (setup이랑 겹치면 안돼서)
// 핀 3개와 뽑을 때 역할들 초기화
int led_r = 9; //LED 레드
int led_b = 10; //LED 블루
int redL = 13; // 빨간선의 핀을 13번으로 정합니다.
int blueL = 12; // 파란선의 핀을 12번으로 정합니다.
int blackL = 11; // 검정선의 핀을 11번으로 정합니다.
int success; // 성공시 사용할 변수
int fast; //핀을 뽑으면 속도가 붙게 하기 위한 변수
int res_fail = 0; // 각각 핀을 뽑을 때 결과 값을 가지고 활용, 특히 fail을 먼저 뽑을 시 fast와 success를 뽑아도 게임오버되도록
int res_success = 0; //성공한 결과가 담길 때
int res_fast = 0; //빨리의 결과가 담길 때
int pinIn = 0; 

Servo servo1;    //서보모터
Servo servo2;
int servo_value = 0;    // 각도를 조절할 변수 value
int servo_value2 = 180; //서보모터를 마주보게 해야 상자가 열렸다 닫았다 할 수 있었는데 그러기 위해서는 한 쪽을 180도에서 90도 갈 수 있도록 해줘야하기 때문에 초기화를 180으로!
```

#### 먼저 사용한 라이브러리를 설명하겠습니다. 

```c
#include "U8glib.h"
```

#### 위의 라이브러리를 사용한 이유는 

#### 사용한 OLED가 II'c 버전의 128x64 크기입니다. 

#### 이 OLED에 메뉴를 그려주기 위해서 라이브러리를 빌려서 사용했습니다.

#### [라이브러리 링크](https://github.com/olikraus/U8glib_Arduino/blob/master/examples/Menu/Menu.ino)

#### 위 링크에서 menu.ino파일의 소스코드를 분석하고 적절하게 버튼과 연결해서 메뉴를 구현했습니다.

#### 또 저희 기종과 맞는 `U8GLIB_SSD1306_128X64 u8g(U8G_I2C_OPT_NO_ACK);`로 초기화해서 사용했습니다.



#### 그 다음으로 서보모터 라이브러리입니다. `#include<Servo.h>` 설명은 생략!



#### 그 다음으로는 세그먼트 라이브러리입니다. `#include <TM1637Display.h>`

#### 제품에 사용한 세그먼트가 I2C 4핀짜리 세그먼트입니다. 위의 라이브러리를 주로 많이 사용한다고 해서 사용했고 CLK, DIO라는 핀이 연결이 되는데 아마.. cpu가 처리할 때 사용할 변수 일것 같습니다.

```c
#define CLK 5
#define DIO 6
TM1637Display display(CLK, DIO); 
```



#### 위와 같이 3개의 라이브러리로 모두 구현했으며 간략한 소스코드 설명을 하겠습니다.

#### 설명은 제품의 핵심코드인 핀을 뽑을 때 처리하는 소스코드만 하겠습니다. 나머지는 주석으로 적어놓았고 어려운 내용이 담기지 않았습니다.  [설명보기](#설명)

```c
void pinSetup(){
  Serial.begin(9600);
  randomSeed(analogRead(0));
  //아두이노의 랜덤값은 미리정해져있는 무작위 숫자의 연속을 순서대로 출력, 따라서 리셋을 누르거나 다시 시작하면 똑같은 순서의 랜덤값을 출력
  //해결: randomSeed를 이용, analogRead 0 은 연결하지 않은 플로팅상태이기 때문에 무작위 수 반환. 이렇게 randomSeed가 초기화되고 원하는 난수 발생
  // random함수의 시드값을 초기
  // 12 ~ 13 중에서 무작위 화 해줍니다.
  success = random(11, 14); //값을 반환하여 success에 저장합니다.
  if (success == 11 && success != 12 && success != 13){
    randomSeed(analogRead(0));
    fast = random(12, 14);
  }
  else if(success == 12 && success != 11 && success != 13){ // ramdom 값은 min ~ max까지 범위를 지정해야 하지만 11, 13번 값만 랜덤으로 돌려야 하는 상황이기 때문에 아래처럼 코드 짬
    randomSeed(analogRead(0)); // 초기화
    int temp = random(1, 3); // 1 ~ 2 중 랜덤으로 값 추출
    if(temp == 1){ // 1일 때
      fast = 11;
    }
    else{ // 2일 때
      fast = 13;
    }
  }
  else if(success == 13 && success != 11 && success != 12){
    randomSeed(analogRead(0));
    fast = random(11, 13);
  }

  Serial.print("성공핀 : ");
  Serial.println(success);
  Serial.print("빠른핀 : ");
  Serial.println(fast);

  Serial.println("빨간핀 : 13 파랑핀 : 12  검정핀 : 11");
}
```



```c
void pinOut(){ //핀을 뽑을 때 이벤트 처리를 위해쓰는 함수, 경우의 수를 세어보니 6개.. 다 구현
    //경우의 수란? 우선 우리는 3개의 핀을 다뤄야 했는데 2개는 간단하게 참 거짓을 비교하면 됩니다. 
    //3개의 핀이 실패할지 성공할지 정하는 것은 setUp에서 정했지만 컴퓨터는 안타깝게도 1과 0 밖에 모르는 바보입니다.
    //그래서 일일이 경우의 수를 구해서 코드를 만들어 주었더니 원하는 코드대로 나올 수 있었습니다.
    그래서 
  if(fast == 11 && success == 12){ // 여기는 설명하기가.. 복잡.. 하다보니된거라 만약 위에 pin_setup에서 난수로 정한 값중 fast가 11 success가 12일때, 이 두개만 비교한 이유는 2개만 비교해야 구별하니까
    if(success == 12 && digitalRead(13) == 1){ // success가 12이면서 13번 핀이 뽑혀 있을 때 그니까 success와 fail이 반대로 바껴서 비교했다고 생각하면됨
      res_success = 1;
      digitalWrite(led_b, HIGH);
      delay(300);
    }
    if(digitalRead(success) == 1){
      res_fail = 1;
      digitalWrite(led_r, HIGH);
      delay(1000);
      digitalWrite(led_r, LOW);
      delay(1000);
      digitalWrite(led_r, HIGH);
      delay(1000);
      digitalWrite(led_r, LOW);
      delay(1000);
    }
    if(digitalRead(11) == 1){
      res_fast = 1;
    }
  }
  else if(fast == 11 && success == 13){
    if(success == 13 && digitalRead(12) == 1){
      res_success = 1;
      digitalWrite(led_b, HIGH);
      delay(300);
    }
    if(digitalRead(success) == 1){
      res_fail = 1;
      digitalWrite(led_r, HIGH);
      delay(1000);
      digitalWrite(led_r, LOW);
      delay(1000);
      digitalWrite(led_r, HIGH);
      delay(1000);
      digitalWrite(led_r, LOW);
      delay(1000);
    }
    if(digitalRead(11) == 1){
      res_fast = 1;
    }
  }
  else if(fast == 12 && success == 13){
    if(success == 13 && digitalRead(11) == 1){
      res_success = 1;
      digitalWrite(led_b, HIGH);
      delay(300);
    }
    if(digitalRead(success) == 1){
      res_fail = 1;
      digitalWrite(led_r, HIGH);
      delay(1000);
      digitalWrite(led_r, LOW);
      delay(1000);
      digitalWrite(led_r, HIGH);
      delay(1000);
      digitalWrite(led_r, LOW);
      delay(1000);
    }
    if(digitalRead(12) == 1){
      res_fast = 1;
    }
  }
  else if(fast == 12 && success == 11){
    if(success == 11 && digitalRead(13) == 1){
      res_success = 1;
      digitalWrite(led_b, HIGH);
      delay(300);
    }
    if(digitalRead(success) == 1){
      res_fail = 1;
      digitalWrite(led_r, HIGH);
      delay(1000);
      digitalWrite(led_r, LOW);
      delay(1000);
      digitalWrite(led_r, HIGH);
      delay(1000);
      digitalWrite(led_r, LOW);
      delay(1000);
    }
    if(digitalRead(12) == 1){
      res_fast = 1;
    }
  }
  else if(fast == 13 && success == 11){
    if(success == 11 && digitalRead(12) == 1){
      digitalWrite(led_b, HIGH);
      delay(300);
      res_success = 1;
    }
    if(digitalRead(success) == 1){
      res_fail = 1;
      digitalWrite(led_r, HIGH);
      delay(1000);
      digitalWrite(led_r, LOW);
      delay(1000);
      digitalWrite(led_r, HIGH);
      delay(1000);
      digitalWrite(led_r, LOW);
      delay(1000);
    }
    if(digitalRead(13) == 1){
      res_fast = 1;
    }
  }
  else if(fast == 13 && success == 12){
    if(success == 12 && digitalRead(11) == 1){
      digitalWrite(led_b, HIGH);
      delay(300);
      res_success = 1;
    }
    if(digitalRead(success) == 1){
      res_fail = 1;
      digitalWrite(led_r, HIGH);
      delay(1000);
      digitalWrite(led_r, LOW);
      delay(1000);
      digitalWrite(led_r, HIGH);
      delay(1000);
      digitalWrite(led_r, LOW);
      delay(1000);
    }
    if(digitalRead(13) == 1){
      res_fast = 1;
    }
  }
}
```

#### [#설명]

#### 보셔도 이해하기 어려울 것입니다. 

#### 컴퓨터는 0과 1 을 가지고 연산하는데 저희는 핀3개를 셋업에서 랜덤값을 추출하고 변수에 담아서 출력 역시 3개를 동시에 이벤트를 주고 싶었습니다. 

#### 만약 2개였다면 true, false로 간단하게 비교할 수 있지만 3개는 간단하게 하지 못한다는 것을 깨달았고 결국 경우의 수를 만들어야겠다고 결심!

#### 그래서 위 코드를 보면 if문에 2개씩 비교하도록했고 3개의 핀이 뽑힐 때의 경우를 각각 만들어주었습니다. 그래서 총 if문을 6개 만들게 된 것입니다.

#### 예 > 11과 13번핀이 셋업에서 랜덤으로 추출한 각각의 fast = 13, success = 11일 때 ~~~등등

#### 위 코드는 이해하려고 하면 이해할 수 없고 저도 코드를 짜고 부딪히면서 결과를 만들어냈기 때문에 결과로 판단했습니다.



## 전체 소스코드와 주석

#이동2

```c
#include "U8glib.h"

U8GLIB_SSD1306_128X64 u8g(U8G_I2C_OPT_NO_ACK);
#include<Servo.h> //Servo 라이브러리를 추가

#include <TM1637Display.h> //segment
#define CLK 5
#define DIO 6
TM1637Display display(CLK, DIO); // clk, dio 셋업
int Sec1, Sec2, Min1, Min2, delays = 100; //세그먼트 일의자리 초, 일의자리 초, 일의 자리 분, 십의자리 분 
uint8_t data[] = { 0x0, 0x0, 0x0, 0x0 }; // 초기값으로 00:00 셋팅
uint8_t segto;

#define KEY_NONE 0 //초기값
#define KEY_PREV 1 
#define KEY_NEXT 2 //OLED 메뉴 이동 관련 변수
#define KEY_SELECT 3 // OLED 선택 관련 변수
#define KEY_BACK 4
#define KEY_POWER 5 //OLED 전원 관련 변수

#define MENU_ITEMS1 3  // Start클릭시 들어가는 서브메뉴 (easy, normal, hard)의 크기 3
#define MENU_ITEMS2 2 // 맨 처음 시작 메뉴 (Start, End)의 크기 2

const char *menu_strings1[MENU_ITEMS1] = { "Easy", "Normal", "Hard" }; // OLED 메뉴에 들어갈 값 선언
const char *menu_strings2[MENU_ITEMS2] = { "Start", "End" }; 
uint8_t menu_current = 0; // OLED 메뉴에서 NEXT 버튼을 클릭했을 때 이동을 했다는 것을 명시하기 위한 변수 (메뉴 이동을 위한 변수)
uint8_t menu_redraw_required = 0; // 메뉴 업데이트시 확인하기 위해, 화면을 빠져나오기 위한 대책
uint8_t last_key_code = KEY_NONE; // 키 초기화 (변수 이름대로 마지막 이벤트가 발생했던 버튼(키)를 저장하기 위한 변수

uint8_t uiKeyNext = 3; // 버튼기능 -> OLED 메뉴 이동
uint8_t uiKeySelect = 2; // 버튼기능 -> OLED 메뉴 선택
uint8_t uiKeyPower = 4; // 버튼기능 -> OLED (Start, End) 메뉴

//초기화
uint8_t uiKeyCodeFirst = KEY_NONE;
uint8_t uiKeyCodeSecond = KEY_NONE;
uint8_t uiKeyCode = KEY_NONE;
int page = 4;
double largPos = 0.0;
double maxTime = 1.0; // OLED 프로그래스 바 초기화 1~120까지 움직이는데 1부터 +1씩 가도록 하기 위해 1초먼저 초기화
int remenu = 0;
unsigned long currentTime1 = 150; // 0.8 : < 난이도 별로 초단위로 시간을 결정했고 OLED와 세그먼트를 맞추기 위해 일련의 작업을 거침 >
unsigned long currentTime2 = 100; // 1.2 : < 세그먼트가 지금처럼 100초일 때 OLED를 그대로 똑같이 실행하면 좀더 늦게 끝난다. 그래서 + 1.2 씩 prograssbar를 늘려주었다. >
unsigned long currentTime3 = 60; // 2.0
int segment_check = 0; //세그먼트 시작전 체크 (setup이랑 겹치면 안돼서)
// 핀 3개와 뽑을 때 역할들 초기화
int led_r = 9; 
int led_b = 10;
int redL = 13; // 빨간선의 핀을 13번으로 정합니다.
int blueL = 12; // 파란선의 핀을 12번으로 정합니다.
int blackL = 11; // 검정선의 핀을 11번으로 정합니다.
int success; // 임의의 선이 정해집니다.(빨간색이나 파란색 선)
int fast;
int res_fail = 0; // 각각 핀을 뽑을 때 결과 값을 가지고 활용, 특히 fail을 먼저 뽑을 시 fast와 success를 뽑아도 게임오버되도록
int res_success = 0;;
int res_fast = 0;
int pinIn = 0;

Servo servo1;    //서보모터
Servo servo2;
int servo_value = 0;    // 각도를 조절할 변수 value
int servo_value2 = 180; // 180도로 초기화 (서로 마주보도록 만들었기 때문에 180도에서 90도로 움직여야 상자를 여닫을 수 있음)
void uiSetup(void) {
  servo1.attach(7);     //맴버함수인 attach : 핀 설정 
  servo2.attach(8);
  servo_value = 0;      //처음 실행될 때 서보모터 0 초기화
  servo1.write(servo_value2);
  servo2.write(servo_value);
  pinMode(uiKeyNext, INPUT_PULLUP);   //버튼 풀업저항으로 세팅
  pinMode(uiKeySelect, INPUT_PULLUP);
  pinMode(uiKeyPower, INPUT_PULLUP);
  
  pinMode(redL, INPUT_PULLUP); // 핀 관련 (풀업으로 해준이유는 뽑을 때 1을 반환받기 위해서)
  pinMode(blueL, INPUT_PULLUP);
  pinMode(blackL, INPUT_PULLUP);
  pinMode(led_r, OUTPUT);
  pinMode(led_b, OUTPUT);
  randomSeed(analogRead(0)); // 시드 초기화해서 난수 발생
}

void uiStep(void) {
  display.setBrightness(0);  //세그먼트 밝기 (0~7 밝기 조절)    
  
  uiKeyCodeSecond = uiKeyCodeFirst;
  if ( digitalRead(uiKeyNext) == LOW )  //풀업이기 때문에 Low시 버튼 클릭됨, 각각 조건이 맞으면 변수에 임의의 설정 변수 값 넣어줌
    uiKeyCodeFirst = KEY_NEXT;
  else if ( digitalRead(uiKeySelect) == LOW )
    uiKeyCodeFirst = KEY_SELECT;
  else if(digitalRead(uiKeyPower) == LOW){
    uiKeyCodeFirst = KEY_POWER;
  }
  else 
    uiKeyCodeFirst = KEY_NONE;
  
  if ( uiKeyCodeSecond == uiKeyCodeFirst )
    uiKeyCode = uiKeyCodeFirst;
  else
    uiKeyCode = KEY_NONE;
}

void pinSetup(){
  Serial.begin(9600);
  randomSeed(analogRead(0));
  //아두이노의 랜덤값은 미리정해져있는 무작위 숫자의 연속을 순서대로 출력, 따라서 리셋을 누르거나 다시 시작하면 똑같은 순서의 랜덤값을 출력
  //해결: randomSeed를 이용, analogRead 0 은 연결하지 않은 플로팅상태이기 때문에 무작위 수 반환. 이렇게 randomSeed가 초기화되고 원하는 난수 발생
  // random함수의 시드값을 초기
  // 12 ~ 13 중에서 무작위 화 해줍니다.
  success = random(11, 14); //값을 반환하여 success에 저장합니다.
  if (success == 11 && success != 12 && success != 13){
    randomSeed(analogRead(0));
    fast = random(12, 14);
  }
  else if(success == 12 && success != 11 && success != 13){ // ramdom 값은 min ~ max까지 범위를 지정해야 하지만 11, 13번 값만 랜덤으로 돌려야 하는 상황이기 때문에 아래처럼 코드 짬
    randomSeed(analogRead(0)); // 초기화
    int temp = random(1, 3); // 1 ~ 2 중 랜덤으로 값 추출
    if(temp == 1){ // 1일 때
      fast = 11;
    }
    else{ // 2일 때
      fast = 13;
    }
  }
  else if(success == 13 && success != 11 && success != 12){
    randomSeed(analogRead(0));
    fast = random(11, 13);
  }

  Serial.print("성공핀 : ");
  Serial.println(success);
  Serial.print("빠른핀 : ");
  Serial.println(fast);

  Serial.println("빨간핀 : 13 파랑핀 : 12  검정핀 : 11");
}

void drawMenu() { // 메뉴를 그려주는 함수
  uint8_t i, h;
  u8g_uint_t w, d;
  u8g.setFont(u8g_font_10x20); // 글자의 폰트를 거의 최대크기로 설정함
  u8g.setFontRefHeightText(); // 폰트 관련해서 크기 설정같은데 써줘야함
  u8g.setFontPosTop();  //폰트의 위치를 이렇게 잡아줘야 OLED에 글자가 알맞게 배치됨
  
  h = u8g.getFontAscent()-u8g.getFontDescent(); // height
  w = u8g.getWidth(); //weight
  if(page == 0){
    for( i = 0; i < MENU_ITEMS1; i++ ) {
      d = (w-u8g.getStrWidth(menu_strings1[i]))/2;
      u8g.setDefaultForegroundColor();
      if ( i == menu_current ) {
        u8g.drawBox(0, i*h+10, w, h); // 글자의 위치를 가운데 쯤에 오도록 적절히 배치해줌
        u8g.setDefaultBackgroundColor();
      }
      u8g.drawStr(d, i*h+11, menu_strings1[i]); // 글자를 적절한 위치에서 보여주게 됨
    }
  } else if(page == 4){ // 위랑 같음
    for( i = 0; i < MENU_ITEMS2; i++ ) {
      d = (w-u8g.getStrWidth(menu_strings2[i]))/2;
      u8g.setDefaultForegroundColor();
      if ( i == menu_current ) {
        u8g.drawBox(0, i*h+18, w, h);
        u8g.setDefaultBackgroundColor();
      }
      u8g.drawStr(d, i*h+19, menu_strings2[i]);
    }
  }
}

void updateMenu(void) {
  if ( uiKeyCode != KEY_NONE && last_key_code == uiKeyCode && segment_check == 0 ) { // 이벤트를 수행하지 않았을 때 
    segment_setup(); //버튼을 누를 때마다 세그먼트 도트가 깜박거리도록 하기 위해서 일부러 여기다 넣음, 또 00:00으로 초기화하기 위함
    return;
  }
  last_key_code = uiKeyCode; // 마지막 키 저장
  switch ( uiKeyCode ) { // 버튼 클릭시 각각의 기능 수행 (NEXT, SELECT, POWER)
    case KEY_NEXT:
      if(page == 0){ // page 0 -> (SELECT, NEXT, POWER가 있는 OLED 메뉴 페이지) 0을 실행(선택)하는 버튼을 클릭했을 때 실행
        menu_current++; // 메뉴 이동을 위해 +1해줌
        if ( menu_current >= MENU_ITEMS1 ) // 만약 크기를 넘는다면 (OLED 메뉴의 값들의 크기만큼 이동할 수 있도록 설정)
          menu_current = 0; // 0으로 초기화
      } else if(page == 4){ // page 4 -> (Start, End가 있는 OLED 메뉴 페이지) 4를 실행(선택)한 버튼을 클릭했을 때 실행
        menu_current++; // 메뉴 이동을 위해 +1해줌
        if ( menu_current >= MENU_ITEMS2 ) // 위랑같음
          menu_current = 0;
      }
      menu_redraw_required = 1; // 메뉴가 업데이트 되었다는 것을 알려줌
      break;
    case KEY_PREV: // 지워도될듯 지금은 안쓰는 뒤로가기 이벤트
      if(page == 0){
        if ( menu_current == 0 )
          menu_current = MENU_ITEMS1;
        menu_current--;
      } else if(page == 4){
        if ( menu_current == 0 )
          menu_current = MENU_ITEMS2;
        menu_current--;
      }
      menu_redraw_required = 1;
      break;
    case KEY_SELECT: // 선택버튼을 클릭시 발생하는 이벤트
      if(page == 0){ // 페이지가 0일 때
        switch( menu_current ){ // 어떤 메뉴를 선택했는지 확인
          case 0:
             page = 1; // easy 페이지로 이동
             pinIn = 1;
             menu_redraw_required = 1; // 메뉴 업데이트 안해주면 전 페이지가 안끝남
             break;        
          case 1:
             page = 2; // normal 페이지 이동
             pinIn = 1;
             menu_redraw_required = 1;
             break;
          case 2:
             page = 3; // hard 페이지 이동
             pinIn = 1;
             menu_redraw_required = 1;
             break;         
          case 4 : // 나중에 4를 넣어주는 문장이 있다. 그때 앞전에 페이지를 수행하고 메뉴 페이지로 이동하기 위해서 일부러 넣은 것
            menu_current = 0; //모두들 초기화
            remenu = 1;
            maxTime = 1.0;
            largPos = 0.0;
            page = 0;
            menu_redraw_required = 1;
            break;
        }
      } else if(page == 4){ // 맨 처음 페이지인 (Start, End) OLED 메뉴 페이지
        switch( menu_current ){ // 서보모터는 동일한 값이 들어오면 동작하지 않기 때문에 코드절약함
          case 0:
            servo_value = 100; // Start일때
            servo_value2 = 90; // 위에 설명한 것처럼 180도에서 90도로 실행함
            servo1.write(servo_value2);
            servo2.write(servo_value);
            page = 0;
            menu_redraw_required = 1;
            break;
          case 1:
            servo_value = 0; // End 일때
            servo_value2 = 180;
            servo1.write(servo_value2);
            servo2.write(servo_value);
            break;     
        }
      }
      break;
    case KEY_POWER: // 만약 4번페이지 (Start, End 메뉴페이지)를 명시하고 있는 POWER버튼을 클릭할 때
      page = 4;
      menu_current = 0; // 초기화
      remenu = 1;
      maxTime = 1.0;
      largPos = 0.0;
      menu_redraw_required = 1;
      break;
  }
}

void setup() { 
  Serial.begin(9600);
  pinMode(A2, OUTPUT); // 부저 output, 디지털핀을 모두 써버려서 아날로그2번핀에 연결함
  uiSetup();
  pinSetup();
  menu_redraw_required = 1; // 시작페이지를 만들어주기 위해 우선 초기화해줌
  noTone(10); // 소리 안나게 
}

void loop() { 
  uiStep();
    u8g.firstPage();
    if(page == 4){
      display.setBrightness(0);  //OLED 최대 밝기 (1~7 밝기 조절)
      if (  menu_redraw_required != 0 ) { // 위에서 계속 나온 이유가 이 문장 ( 0이 아닐 때 밑에 업데이트하는 문장 실행)
        u8g.firstPage();
        do  {
            drawMenu();
        } while( u8g.nextPage() ); 
          menu_redraw_required = 0;
      }
    }
    else if(page == 0){
      display.setBrightness(0);  //OLED 최대 밝기 (1~7 밝기 조절)
      menu_page();
    }
    else if(page == 1) { //쉬운
    if((uiKeyCodeFirst > 1) && maxTime > 120){ // 실패시 메뉴로 돌아가기 위해서 몇가지 체크를 함 ( 버튼 아무거나 눌렀을때를 위해서 uiKeyCodeFirst를 1이상으로 잡음 maxtime이 120일 때 실패기 때문에 조건에 추가
      remenu = 1; 
      maxTime = 1.0;
      largPos = 0.0;
      page = 0;
      menu_current = 4; // 원래 메뉴에 (easy, normal, hard) 3가지 밖에 없는데 일부러 4를 적어서 페이지를 빠져나가기 위한 꼼수
      segment_check = 0;
      currentTime1 = 150;
      res_fail = 0;
      res_success = 0;
      res_fast = 0;
      noTone(1);
    }
    if(remenu != 1){ // 1이 아닐때만 prograssbar를 실행      
      timePrograssbar(1);
      if ((digitalRead(11) == 1 || digitalRead(12) == 1 || digitalRead(13) == 1) && pinIn == 0){
        remenu = 1; 
        uiKeyCode = KEY_NEXT; // 이걸 적어줘야 메뉴를 이동할 시 흰색 배경의 메뉴 선택 바가 배치됨
        maxTime = 1.0;
        largPos = 0.0;
        page = 0;
        menu_current = 4; // 원래 메뉴에 (easy, normal, hard) 3가지 밖에 없는데 일부러 4를 적어서 페이지를 빠져나가기 위한 꼼수
        segment_check = 0;
        delay(1000);
      }
    }
    remenu = 0;
  } else if(page == 2) { //보통
    if((uiKeyCodeFirst > 1) && maxTime > 120){ //초기화 작업 및 메뉴로 이동
      remenu = 1;
      maxTime = 1.0;
      largPos = 0.0;
      page = 0;
      menu_current = 4;
      segment_check = 0;
      res_fail = 0;
      res_success = 0;
      res_fast = 0;
      noTone(1);
      currentTime2 = 100;
    }
    if(remenu != 1){
      timePrograssbar(2);
      if ((digitalRead(11) == 1 || digitalRead(12) == 1 || digitalRead(13) == 1) && pinIn == 0){
        remenu = 1; 
        uiKeyCode = KEY_NEXT;
        maxTime = 1.0;
        largPos = 0.0;
        page = 0;
        menu_current = 4; // 원래 메뉴에 (easy, normal, hard) 3가지 밖에 없는데 일부러 4를 적어서 페이지를 빠져나가기 위한 꼼수
        segment_check = 0;
        delay(1000);
      }
    }
    remenu = 0;
  } else if(page == 3) { //어려운
    if((uiKeyCodeFirst > 1) && maxTime > 120){
      remenu = 1;
      maxTime = 1.0;
      largPos = 0.0;
      menu_current = 4;
      page = 0;
      noTone(1);
      res_fail = 0;
      res_success = 0;
      res_fast = 0;
      pinIn = 0;
      segment_check = 0;
      currentTime3 = 60;
    }
    if(remenu != 1){
      if ((digitalRead(11) == 1 || digitalRead(12) == 1 || digitalRead(13) == 1) && pinIn == 0){
        remenu = 1; 
        uiKeyCode = KEY_NEXT;
        maxTime = 1.0;
        largPos = 0.0;
        page = 0;
        menu_current = 4; // 원래 메뉴에 (easy, normal, hard) 3가지 밖에 없는데 일부러 4를 적어서 페이지를 빠져나가기 위한 꼼수
        segment_check = 0;
        delay(1000);
      }
      else if(res_success != 1 || res_fail != 1){
        timePrograssbar(3);
      } 
    }
    remenu = 0;
  }
  updateMenu();
}

void menu_page(){ // 공통되는 메뉴 페이지 분리
  if (  menu_redraw_required != 0 ) {
    u8g.firstPage();
    do  {
        drawMenu(); //메뉴를 그려주는 커스텀함수
    } while( u8g.nextPage() );
    menu_redraw_required = 0;
  }
}

void timePrograssbar(int page){ // 프로그래스 바 (시간이 얼마남았는지 OLED로 출력하기 위해서 만듬)
  if(digitalRead(11) == 0 && digitalRead(12) == 0 && digitalRead(13) == 0){
    u8g.setFont(u8g_font_tpssb); 
    u8g.setColorIndex(1); 
    u8g.firstPage();
      segment_check = 1; // segment_setup과 겹치는 것을 방지
      segment(page);
      tone(A2, 440,10); // 연결핀, 진동수, 발생시간
      pinOut();
      do  {
        if(maxTime < 120) { // 시간이 아직 남았을 때
          u8g.drawStr( 23, 50, "Time remaining"); // 텍스트 출력
          u8g.drawFrame(5,10,120,20);
          draw(); 
        }
        if(maxTime > 120){ //시간이 지났을 때
          u8g.setFont(u8g_font_10x20);
          u8g.drawStr(20, 40, "TIME OVER");
        }
        if(res_fast == 1){
          delays = 0;
        }
      } while( u8g.nextPage() );
      if(page == 1){
        times(0.8); // 커스텀 함수 호출
      }
      else if(page == 2){
        times(1.2); // 커스텀 함수 호출
      }
      else if(page == 3){
        times(2.0); // 커스텀 함수 호출
      }  
    }
    //아래코드는 핀이 뽑혀있을 때 실행 못하게 하기 위해 만들었지만 방대해져서 보류
//    if ((digitalRead(11) == 1 || digitalRead(12) == 1 || digitalRead(13) == 1) && pinIn == 0){
//      u8g.setFont(u8g_font_9x15);
//        u8g.firstPage();
//        do  {
//          u8g.drawStr(1, 30, "The pin should");
//          u8g.drawStr(5, 45, "be connected");
//       } while( u8g.nextPage() );
//    }
}

void times(double num){ // 이 함수에서는 프로그래스바와 시간을 증가하도록 함
  if(largPos < 120){
    largPos = (double)largPos + num; //세그먼트와 OLED를 맞추기 위해 +num 해준것
    maxTime = (double)maxTime + num;
    }
}

void draw(){ // 프로그래스바를 그려줌
    u8g.drawBox(5,15,largPos,10); // 작은 박스 (x,y,w,h) 인자
}

void segment(int n){
  unsigned long num = 0;
   if (n == 1){ // easy 일때 , 이렇게 나눈 이유는 세그먼트를 각각 출력할 때 들어갈 숫자들이 다르기 때문에.. 그리고 unsigned long currentTime1 = 150 이런식으로 선언해주어야 실행돼서 변수로 값을 제어할 수가 없음.. 방법이 없이 노가다행
    if (currentTime1 != -1){
     Sec2 = currentTime1%10;        // 초단위 '일의 자리' 만 분리하여 저장    
     Sec1 = (currentTime1/10)%10;   // 초단위 '십의 자리' 만 분리하여 저장
     Min2 = (currentTime1/100)%10;  // 분단위 '일의 자리' 만 분리하여 저장
     Min1 = (currentTime1/1000)%10; // 분단위 '십의 자리' 만 분리하여 저장
  
     data[0]=display.encodeDigit(Min1);    // 초시계
     data[1]=display.encodeDigit(Min2);
     data[2]=display.encodeDigit(Sec1); 
     data[3]=display.encodeDigit(Sec2);
     
     segto = 0x80 | display.encodeDigit(Min2);  // 초시계용 ':'도트
    display.setSegments(data);
    display.setSegments(&segto,1,1);
    display.setBrightness(0);
    delay(delays);
    if(currentTime1 < 60 && currentTime1 != 0){ //만약 60초 아래로 시간이 줄어들 때 세그먼트 밝기를 2만큼 더해준다.
      display.setBrightness(2);
    }
    else if(currentTime1 == 0){ //끝났을 때는 세그먼트 밝기를 0으로
      display.setBrightness(0);
    }
    display.setSegments(data);  //도트가 깜빡이게 하기 위해 반복 출력
    delay(delays);
    currentTime1 -= 1; // 시간을 -1 씩 줄임
    } else { //세그먼트가 무한 반복되는 이유는 세그먼트가 -1이 아닐 때는 계속 실행되는데 끝나면 0으로 초기화해주기 때문에 계속 돌고 돈다..
        currentTime1 = 0;
      }
   }
  else if (n == 2){
    if (currentTime2 != -1){
     Sec2 = currentTime2%10;        // 초단위 '일의 자리' 만 분리하여 저장    
     Sec1 = (currentTime2/10)%10;   // 초단위 '십의 자리' 만 분리하여 저장
     Min2 = (currentTime2/100)%10;  // 분단위 '일의 자리' 만 분리하여 저장
     Min1 = (currentTime2/1000)%10; // 분단위 '십의 자리' 만 분리하여 저장
  
     data[0]=display.encodeDigit(Min1);    // 초시계
     data[1]=display.encodeDigit(Min2);
     data[2]=display.encodeDigit(Sec1); 
     data[3]=display.encodeDigit(Sec2); 

    segto = 0x80 | display.encodeDigit(Min2);  // 초시계용 ':'도트
    display.setSegments(data);
    display.setSegments(&segto,1,1);
    display.setBrightness(0);
    delay(delays);
    if(currentTime2 < 60 && currentTime2 != 0){
      display.setBrightness(2);
    }
    else if(currentTime2 == 0){
      display.setBrightness(0);
    }
    display.setSegments(data);  //도트가 깜빡이게 하기 위해 반복 출력
    delay(delays);
    currentTime2 -= 1;
    } else {
        currentTime2 = 0;
      }
  }
  else if (n == 3){
    if (currentTime3 != -1){
     Sec2 = currentTime3%10;        // 초단위 '일의 자리' 만 분리하여 저장    
     Sec1 = (currentTime3/10)%10;   // 초단위 '십의 자리' 만 분리하여 저장
     Min2 = (currentTime3/100)%10;  // 분단위 '일의 자리' 만 분리하여 저장
     Min1 = (currentTime3/1000)%10; // 분단위 '십의 자리' 만 분리하여 저장
  
     data[0]=display.encodeDigit(Min1);    // 초시계
     data[1]=display.encodeDigit(Min2);
     data[2]=display.encodeDigit(Sec1); 
     data[3]=display.encodeDigit(Sec2); 

    segto = 0x80 | display.encodeDigit(Min2);  // 초시계용 ':'도트
    display.setSegments(data);
    display.setSegments(&segto,1,1);
    display.setBrightness(0);
    delay(delays);
    if(currentTime3 < 60 && currentTime3 != 0){
      display.setBrightness(2);
    }
    else if(currentTime3 == 0){
      display.setBrightness(0);
    }
    display.setSegments(data);  //도트가 깜빡이게 하기 위해 반복 출력
    delay(delays);
    currentTime3 -= 1;
    } else {
        currentTime3 = 0;
      }
  }
}

void segment_setup(){ // 이 함수를 만든이유는 처음 상자를 시작할 때 0000상태의 빨간 불이 들어오도록 하기 위해서.. 외적인 디자인상?
  if (0 != -1){
    Sec2 = 0%10;        // 초단위 '일의 자리' 만 분리하여 저장    
    Sec1 = (0/10)%10;   // 초단위 '십의 자리' 만 분리하여 저장
    Min2 = (0/100)%10;  // 분단위 '일의 자리' 만 분리하여 저장
    Min1 = (0/1000)%10; // 분단위 '십의 자리' 만 분리하여 저장
  
    data[0]=display.encodeDigit(Min1);    // 초시계
    data[1]=display.encodeDigit(Min2);
    data[2]=display.encodeDigit(Sec1); 
    data[3]=display.encodeDigit(Sec2); 

    segto = 0x80 | display.encodeDigit(Min2);  // 초시계용 ':'도트
    display.setSegments(data);
    display.setSegments(&segto,1,1);
    delay(delays);
    display.setSegments(data);  //도트가 깜빡이게 하기 위해 반복 출력
    delay(delays);
  }
}

void pinOut(){ //핀을 뽑을 때 이벤트 처리를 위해쓰는 함수, 경우의 수를 세어보니 6개.. 다 구현
    //경우의 수란? 우선 우리는 3개의 핀을 다뤄야 했는데 2개는 간단하게 참 거짓을 비교하면 됩니다. 
    //3개의 핀이 실패할지 성공할지 정하는 것은 setUp에서 정했지만 컴퓨터는 안타깝게도 1과 0 밖에 모르는 바보입니다.
    //그래서 일일이 경우의 수를 구해서 코드를 만들어 주었더니 원하는 코드대로 나올 수 있었습니다.
    그래서 
  if(fast == 11 && success == 12){ // 여기는 설명하기가.. 복잡.. 하다보니된거라 만약 위에 pin_setup에서 난수로 정한 값중 fast가 11 success가 12일때, 이 두개만 비교한 이유는 2개만 비교해야 구별하니까
    if(success == 12 && digitalRead(13) == 1){ // success가 12이면서 13번 핀이 뽑혀 있을 때 그니까 success와 fail이 반대로 바껴서 비교했다고 생각하면됨
      res_success = 1;
      digitalWrite(led_b, HIGH);
      delay(300);
    }
    if(digitalRead(success) == 1){
      res_fail = 1;
      digitalWrite(led_r, HIGH);
      delay(1000);
      digitalWrite(led_r, LOW);
      delay(1000);
      digitalWrite(led_r, HIGH);
      delay(1000);
      digitalWrite(led_r, LOW);
      delay(1000);
    }
    if(digitalRead(11) == 1){
      res_fast = 1;
    }
  }
  else if(fast == 11 && success == 13){
    if(success == 13 && digitalRead(12) == 1){
      res_success = 1;
      digitalWrite(led_b, HIGH);
      delay(300);
    }
    if(digitalRead(success) == 1){
      res_fail = 1;
      digitalWrite(led_r, HIGH);
      delay(1000);
      digitalWrite(led_r, LOW);
      delay(1000);
      digitalWrite(led_r, HIGH);
      delay(1000);
      digitalWrite(led_r, LOW);
      delay(1000);
    }
    if(digitalRead(11) == 1){
      res_fast = 1;
    }
  }
  else if(fast == 12 && success == 13){
    if(success == 13 && digitalRead(11) == 1){
      res_success = 1;
      digitalWrite(led_b, HIGH);
      delay(300);
    }
    if(digitalRead(success) == 1){
      res_fail = 1;
      digitalWrite(led_r, HIGH);
      delay(1000);
      digitalWrite(led_r, LOW);
      delay(1000);
      digitalWrite(led_r, HIGH);
      delay(1000);
      digitalWrite(led_r, LOW);
      delay(1000);
    }
    if(digitalRead(12) == 1){
      res_fast = 1;
    }
  }
  else if(fast == 12 && success == 11){
    if(success == 11 && digitalRead(13) == 1){
      res_success = 1;
      digitalWrite(led_b, HIGH);
      delay(300);
    }
    if(digitalRead(success) == 1){
      res_fail = 1;
      digitalWrite(led_r, HIGH);
      delay(1000);
      digitalWrite(led_r, LOW);
      delay(1000);
      digitalWrite(led_r, HIGH);
      delay(1000);
      digitalWrite(led_r, LOW);
      delay(1000);
    }
    if(digitalRead(12) == 1){
      res_fast = 1;
    }
  }
  else if(fast == 13 && success == 11){
    if(success == 11 && digitalRead(12) == 1){
      digitalWrite(led_b, HIGH);
      delay(300);
      res_success = 1;
    }
    if(digitalRead(success) == 1){
      res_fail = 1;
      digitalWrite(led_r, HIGH);
      delay(1000);
      digitalWrite(led_r, LOW);
      delay(1000);
      digitalWrite(led_r, HIGH);
      delay(1000);
      digitalWrite(led_r, LOW);
      delay(1000);
    }
    if(digitalRead(13) == 1){
      res_fast = 1;
    }
  }
  else if(fast == 13 && success == 12){
    if(success == 12 && digitalRead(11) == 1){
      digitalWrite(led_b, HIGH);
      delay(300);
      res_success = 1;
    }
    if(digitalRead(success) == 1){
      res_fail = 1;
      digitalWrite(led_r, HIGH);
      delay(1000);
      digitalWrite(led_r, LOW);
      delay(1000);
      digitalWrite(led_r, HIGH);
      delay(1000);
      digitalWrite(led_r, LOW);
      delay(1000);
    }
    if(digitalRead(13) == 1){
      res_fast = 1;
    }
  }
}
```

