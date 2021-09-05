# 아두이노 수화번역기
아두이노와 구부림 센서를 활용한 청각장애인들을 위한 수화번역기(숭실대학교 아두이노 공모전 (대상)

## 개요
숭실대학교 아두이노 공모전 출품을 위해 어떤 작품을 만들지 고민하였습니다. 그때 고등학교 선생님이 가르쳐주신 소셜 임팩트 라는 단어를 기억했습니다. 단순한 개발이 아닌 사회에 도움이 되는 소셜 임팩트를 실현할 수 있는 작품을 만들면 좋을 것 같다고 생각하였고 수화를 읽지 못하는 청각 장애인이 많다는 사실을 바탕으로 청각 장애인을 위한 수화 번역기를 만들게 되었습니다. 

## 개발과정
청각장애인의 수화 문맹률은 매우 높은 수준으로 나타나있습니다. 따라서 저는 아두이노의 구부림센서, 자이로센서 lcd모듈을 이용하여 손의 모양, 움직임, 위치 정보를 받아 수화를 번역해주는 작품을 만들기로 하였습니다.

#### 기술
* Arduino
  * 개발을 위한 보드입니다. 아두이노 코딩은 기본적으로 C++을 기반으로 합니다.

### 1. 재료 준비
1. 구부림센서
<img src='https://github.com/HwanGonJang/HwanGonJang.github.io/blob/master/Pictures/as_1.jpg?raw=true'>

2. 자이로 센서
<img src='https://github.com/HwanGonJang/HwanGonJang.github.io/blob/master/Pictures/as_2.jpg?raw=true'>

3. lcd 모듈
<img src='https://github.com/HwanGonJang/HwanGonJang.github.io/blob/master/Pictures/as_3.jpg?raw=true'>


### 2. 회로 작성
<img src='https://github.com/HwanGonJang/HwanGonJang.github.io/blob/master/Pictures/as_4.png?raw=true'>
<img src='https://github.com/HwanGonJang/HwanGonJang.github.io/blob/master/Pictures/as_5.jpg?raw=true'>


### 코딩
~~~c
// 자이로센서, LCD를 쉽게 제어하기 위한 라이브러리를 추가합니다.
#include <LiquidCrystal_I2C.h>
#include <Wire.h>
int flexpin1 = A3;  //구부림센서 값을 센싱할 아날로그 핀을 지정합니다.
int flexpin2 = A2;
int flexpin3 = A1; 
int flexpin4 = A0;
const int MPU=0x68;  //MPU 6050 의 I2C 기본 주소입니다.
int16_t AcX, AcY,AcZ;
// 0x3F I2C 주소를 가지고 있는 16x2 LCD객체를 생성합니다.
LiquidCrystal_I2C lcd(0x27, 16, 2);

// 실행시 가장 먼저 호출되는 함수이며, 최초 1회만 실행됩니다.
// 변수를 선언하거나 초기화를 위한 코드를 포함합니다.
void setup() {
	Wire.begin();                //Wire 라이브러리 초기화
	Wire.beginTransmission(MPU); //MPU로 데이터 전송 시작
	Wire.write(0x6B);            // PWR_MGMT_1 register
	Wire.write(0);               //MPU-6050 시작 모드로
	Wire.endTransmission(true); 
	Serial.begin(9600); 

	// I2C LCD를 초기화 합니다..
	lcd.init();
	// I2C LCD의 백라이트를 켜줍니다.
	lcd.backlight();
}

// setup() 함수가 호출된 이후, loop() 함수가 호출되며,
// 블록 안의 코드를 무한히 반복 실행됩니다.
void loop() { 
	//센서값을 저장할 변수 설정합니다.
	int flexVal1; 
	int flexVal2; 
	int flexVal3; 
	int flexVal4; 
    // 센서로 부터 보내오는 값을 입력 받습니다.(0-1023)
    flexVal1 = analogRead(flexpin1); 
    flexVal2 = analogRead(flexpin2); 
    flexVal3 = analogRead(flexpin3); 
    flexVal4 = analogRead(flexpin4); 

    Wire.beginTransmission(MPU);    //데이터 전송시작
    Wire.write(0x3B);               // register 0x3B (ACCEL_XOUT_H), 큐에 데이터 기록
    Wire.endTransmission(false);    //연결유지
    Wire.requestFrom(MPU,14,true);  //MPU에 데이터 요청
    //데이터 한 바이트 씩 읽어서 반환
    AcX=Wire.read()<<8|Wire.read(); 
    AcY=Wire.read()<<8|Wire.read();  
    AcZ=Wire.read()<<8|Wire.read();  


    //시리얼 모니터에 출력
    Serial.print(" | AcX = "); Serial.print(AcX);
    Serial.print(" | AcY = "); Serial.print(AcY);
    Serial.print(" | AcZ = "); Serial.print(AcZ);
    Serial.print("     sensor: ");
    // sensor:XXX 로 출력한다(XXX값은 센서로 부터 읽어 온 값)
    Serial.print(flexVal1); 
    Serial.print("     sensor: ");
    Serial.print(flexVal2);
    Serial.print("     sensor: ");
    Serial.print(flexVal3);
    Serial.print("     sensor: ");
    Serial.println(flexVal4);


    //구부림센서와 자이로센서 값의 경우의 수에 따른 수화번역 조건문입니다.
    //여러가지 수화에 대한 번역을 if문으로 코딩합니다.
    if (flexVal1 < 995 && flexVal2 < 990 && flexVal3 < 1000 && flexVal4 < 1000 ) {
    // 0번째 줄 0번째 셀부터 입력하게 합니다.
    lcd.setCursor(0,1);           // 0번째 줄 1번째 셀부터 입력하게 합니다.
    lcd.print("transrating...");       // 문구를 출력합니다.
    }
    else if(flexVal1 >= 995 && flexVal2 < 990 && flexVal3 < 1000 && flexVal4 < 1000 && AcZ > 0 && AcZ < 10000 && AcY > 0 && AcY <10000){
    lcd.setCursor(0,0);           // 0번째 줄 0번째 셀부터 입력하게 합니다.
    lcd.print("hi");       // 문구를 출력합니다.
    }else if(flexVal1 < 995 && flexVal2 >= 990 && flexVal3 >= 1000 && flexVal4 < 1000 && AcZ > 0 && AcZ < 10000 && AcY > 0 && AcY <10000) {
    lcd.setCursor(0,0);           // 0번째 줄 0번째 셀부터 입력하게 합니다.
    ...(이하 생략)
~~~

## 결과
이렇게 청각 장애인을 위한 수화 번역기를 제작하였습니다. 아두이노는 정말 소셜임팩트를 실현하기에 가장 좋은 방법 중 하나라고 생각합니다. 이 작품이 사회에 도움이 될 수 있어 뿌듯합니다.

<img src='https://github.com/HwanGonJang/HwanGonJang.github.io/blob/master/Pictures/as_6.jpg?raw=true'>

다음은 시연영상입니다.
https://youtu.be/tnRkoJWIAME
