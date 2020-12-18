# Signlanguage_Translating_with_Arduino
아두이노와 구부림 센서를 활용한 청각장애인들을 위한 수화번역기(숭실대학교 아두이노 공모전)

## 개요
숭실대학교 아두이노 공모전 출품을 위해 제작한 청각장애인을 위한 수화번역기.    
아두이노와 구부림센서를 활용하여 수화를 번역하여 LCD패널로 출력해준다.
### 구부림 센서
<img widh="500" src="https://user-images.githubusercontent.com/33739448/102565767-781fac80-4121-11eb-9ff2-e6a0de88372f.jpg">

## 
<img width="500" src="https://user-images.githubusercontent.com/33739448/102565212-5245d800-4120-11eb-8d15-9bfffeb47fcb.png">

## 코드 
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
        lcd.print("everyone");       // 문구를 출력합니다.
      }else if(flexVal1 < 995 && flexVal2 < 990 && flexVal3 >= 1000 && flexVal4 < 1000 && AcZ > 0 && AcZ < 10000 && AcY > 0 && AcY <10000) {
        lcd.setCursor(0,0);           // 0번째 줄 0번째 셀부터 입력하게 합니다.
        lcd.print("I am");       // 문구를 출력합니다.
      }else if(flexVal1 < 995 && flexVal2 < 990 && flexVal3 < 1000 && flexVal4 >= 1000 && AcZ > 0 && AcZ < 10000 && AcY > 0 && AcY <10000) {
        lcd.setCursor(0,0);           // 0번째 줄 0번째 셀부터 입력하게 합니다.
        lcd.print("HwangonJang");       // 문구를 출력합니다.
      }else if(flexVal1 >= 995 && flexVal2 < 990 && flexVal3 < 1000 && flexVal4 < 1000 && AcZ < -3000) {
        lcd.setCursor(0,0);           // 0번째 줄 0번째 셀부터 입력하게 합니다.
        lcd.print("a");       // 문구를 출력합니다.
      }else if(flexVal1 >= 995 && flexVal2 < 990 && flexVal3 >= 1000 && flexVal4 < 1000 && AcZ > 13000 && AcY > 0 && AcY <10000) {
        lcd.setCursor(0,0);           // 0번째 줄 0번째 셀부터 입력하게 합니다.
        lcd.print("student");       // 문구를 출력합니다.
      }else if(flexVal1 >= 995 && flexVal2 < 990 && flexVal3 >= 1000 && flexVal4 < 1000 && AcZ < -3000 && AcY > 0 && AcY <10000) {
        lcd.setCursor(0,0);           // 0번째 줄 0번째 셀부터 입력하게 합니다.
        lcd.print("at");       // 문구를 출력합니다.
      }else if(flexVal1 < 995 && flexVal2 < 990 && flexVal3 >= 1000 && flexVal4 >= 1000 && AcZ > 13000 && AcY > 0 && AcY <10000) {
        lcd.setCursor(0,0);           // 0번째 줄 0번째 셀부터 입력하게 합니다.
        lcd.print("Soongsil");       // 문구를 출력합니다.
      }else if(flexVal1 < 995 && flexVal2 < 990 && flexVal3 >= 1000 && flexVal4 >= 1000 && AcZ < -3000 && AcY > 0 && AcY <10000) {
        lcd.setCursor(0,0);           // 0번째 줄 0번째 셀부터 입력하게 합니다.
        lcd.print("University");       // 문구를 출력합니다.
      }else if(flexVal1 >= 995 && flexVal2 < 990 && flexVal3 < 1000 && flexVal4 < 1000 && AcY < -13000 && AcZ > -5000 && AcZ < 10000) {
        lcd.setCursor(0,0);           // 0번째 줄 0번째 셀부터 입력하게 합니다.
        lcd.print("my");       // 문구를 출력합니다.
      }else if(flexVal1 >= 995 && flexVal2 < 990 && flexVal3 < 1000 && flexVal4 < 1000 && AcY > 13000 && AcZ > -5000 && AcZ < 10000) {
        lcd.setCursor(0,0);           // 0번째 줄 0번째 셀부터 입력하게 합니다.
        lcd.print("major");       // 문구를 출력합니다.
      }else if(flexVal1 >= 995 && flexVal2 < 990 && flexVal3 >= 1000 && flexVal4 < 1000 && AcY < -13000 && AcZ > -5000 && AcZ < 10000) {
        lcd.setCursor(0,0);           // 0번째 줄 0번째 셀부터 입력하게 합니다.
        lcd.print("is");       // 문구를 출력합니다.
      }else if(flexVal1 >= 995 && flexVal2 < 990 && flexVal3 >= 1000 && flexVal4 < 1000 && AcY > 13000 && AcZ > -5000 && AcZ < 10000) {
        lcd.setCursor(0,0);           // 0번째 줄 0번째 셀부터 입력하게 합니다.
        lcd.print("convergence");       // 문구를 출력합니다.
        lcd.setCursor(0,1);           // 0번째 줄 0번째 셀부터 입력하게 합니다.
        lcd.print("specialization");       // 문구를 출력합니다.
      }else if(flexVal1 >= 995 && flexVal2 >= 990 && flexVal3 >= 1000 && flexVal4 < 1000 && AcZ > 13000 && AcY > 0 && AcY <10000) {
        lcd.setCursor(0,0);           // 0번째 줄 0번째 셀부터 입력하게 합니다.
        lcd.print("glad");       // 문구를 출력합니다.
      }else if(flexVal1 >= 995 && flexVal2 < 990 && flexVal3 >= 1000 && flexVal4 >= 1000 && AcZ < -3000 && AcY > 0 && AcY <10000) {
        lcd.setCursor(0,0);           // 0번째 줄 0번째 셀부터 입력하게 합니다.
        lcd.print("participated");       // 문구를 출력합니다.
      }else if(flexVal1 >= 995 && flexVal2 < 990 && flexVal3 >= 1000 && flexVal4 >= 1000 && AcY < -13000 && AcZ > -5000 && AcZ < 10000) {
        lcd.setCursor(0,0);           // 0번째 줄 0번째 셀부터 입력하게 합니다.
        lcd.print("in");       // 문구를 출력합니다.
      }else if(flexVal1 >= 995 && flexVal2 < 990 && flexVal3 >= 1000 && flexVal4 >= 1000 && AcY > 13000 && AcZ > -5000 && AcZ < 10000) {
        lcd.setCursor(0,0);           // 0번째 줄 0번째 셀부터 입력하게 합니다.
        lcd.print("the");       // 문구를 출력합니다.
      }else if(flexVal1 >= 995 && flexVal2 < 990 && flexVal3 >= 1000 && flexVal4 >= 1000 && AcZ > 13000 && AcY > 0 && AcY <10000) {
        lcd.setCursor(0,0);           // 0번째 줄 0번째 셀부터 입력하게 합니다.
        lcd.print("Arduino contest");       // 문구를 출력합니다.
      }else if(flexVal1 >= 995 && flexVal2 >= 990 && flexVal3 >= 1000 && flexVal4 >= 1000 && AcZ > 0 && AcZ < 10000 && AcY > 0 && AcY <10000) {
        lcd.setCursor(0,0);           // 0번째 줄 0번째 셀부터 입력하게 합니다.
        lcd.print("Thank You");       // 문구를 출력합니다.
      }else if(flexVal1 >= 995 && flexVal2 >= 990 && flexVal3 >= 1000 && flexVal4 >= 1000 && AcZ < -3000 && AcY > 0 && AcY <10000) {
        lcd.setCursor(0,0);           // 0번째 줄 0번째 셀부터 입력하게 합니다.
        lcd.print("-Sign language-");       // 문구를 출력합니다.
      }
      // 1초간 대기합니다.
      delay(1000);
      // LCD의 모든 내용을 삭제합니다.
      lcd.clear();
      //loop를 반복합니다.
    }
    
## 완성
<img width="500" src=https://user-images.githubusercontent.com/33739448/102565518-f4fe5680-4120-11eb-8732-ce4bd5439dfb.jpg>

    
