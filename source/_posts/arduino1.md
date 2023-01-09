---
title: arduino1
date: 2022-12-30 12:14:00
categories:
  - arduino
tags:
  - arduino
  - 编程
---

# Arduino 项目一览表

## 立项的项目

- 智能小车

- 1602a 小屏幕

- 蓝牙通信

- 暂定

## 已完成的项目

- 机械臂+控制器项目

```c
#include <LiquidCrystal_I2C.h>
#include <Servo.h>

LiquidCrystal_I2C lcd(0x27, 16, 2);
Servo Rotate;
Servo Left;
Servo Right;
Servo Grab;
int buzzer = 7;
int PIN_SPEAKER = 7;
//char notes[] = "eefehgeefeiheeljhgfddjhih";
//int beats[] = {1, 1, 2, 2, 2, 4, 1, 1, 2, 2, 2, 4, 1, 1, 2, 2, 2, 2, 2, 1, 1, 2, 2, 2, 4, 1};
//5L 5L 6L 5L 1 7L 5L 5L 6L 5L 2 1 5L 5L 5 3 1 7L 6L 4 4 3 1 2 1
//int length = 25;
//生日快乐歌
//char notes[] = "ceefecaabccbabceefecaabccbbadddfeecbceefecaabccbba";
//int beats[] = {2,2,2,1,2,2,2,1,1,2,2,2,2,8,2,2,2,1,2,2,2,1,1,2,2,2,2,8,4,4,2,6,4,2,2,8,2,2,2,1,2,2,2,1,1,2,2,2,2,8};
//int length = 51;
//蜜雪冰城
char notes[] = "cfjjikceiihjhihghjifchgf";
int beats[] = {2,2,2,2,1,6,2,2,2,2,1,6,1,1,1,2,2,2,1,6,2,2,4,6};
//int length = 24;
//最伟大的作品
//{'a', 'b', 'c', 'd', 'e', 'f', 'g',
// 'h', 'i', 'j', 'k', 'l', 'm', 'n',
// 'o', 'p', 'q', 'r', 's', 't', 'u'};
int tempo = 150;


int mode = 0;
//0:direct; 1:serial;
int GrabButton = 12;
int SoundButton = 13;
// order: Rotate9 Left11 Right8 Grab10
// limit: 0~180 80~180 60~180 80~100
int Initial[4] = {90, 120, 90, 45};//Initial servo status
String Servoname[4] = {"Rotate", "Left", "Right", "Grab"};
String comdata = "";
int numdata[6] = {0}, mark = 0;
boolean Grabbing;
int RotateRD;
int LeftRD;
int RightRD;
int GrabRD;
void output() {
  Serial.print("Rotate: ");
  Serial.print(RotateRD);
  Serial.print(" Left: ");
  Serial.print(LeftRD);
  Serial.print(" Right: ");
  Serial.print(RightRD);
  Serial.print(" Grab: ");
  Serial.println(GrabRD);
  lcd.setCursor(0, 1);
  lcd.print(Rotate.read());
  lcd.print("/");
  lcd.print(Left.read());
  lcd.print("/");
  lcd.print(Right.read());
  lcd.print("/");
  lcd.print(Grabbing);
  lcd.print("    ");
}

void playTone(int tone, int duration) {
  for (long i = 0; i < duration * 1000L; i += tone * 2) {
    digitalWrite(PIN_SPEAKER, HIGH);
    delayMicroseconds(tone);
    digitalWrite(PIN_SPEAKER, LOW);
    delayMicroseconds(tone);
  }
}

void playNote(char note, int duration) {
  char names[] = {'a', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 'i', 'j', 'k', 'l', 'm', 'n', 'o', 'p', 'q', 'r', 's', 't', 'u'};
  int tones[] = {1908, 1700, 1515, 1432, 1275, 1136, 1012, 956, 851, 758, 716, 637, 568, 506, 478, 425, 379, 357, 318, 284, 253};

  for (int i = 0; i < 12; i++) {
    if (names[i] == note) {
      playTone(tones[i] * 2, duration);
    }
  }
}

void buzzsound() {
  for (int i = 0; i < length; i++) {
    if (notes[i] == ' ') {
      delay(beats[i] * tempo);
    } else {
      playNote(notes[i], beats[i] * tempo);
    }
    delay(tempo / 2);
  }


}
void servowrite(int servonum, int goal, boolean slowbl, boolean ifdelay) {
  switch (servonum) {
    case 0:
      if (slowbl == true) {
        slowmove (Rotate, goal, ifdelay);
      } else {
        Rotate.write(goal);
      }
      //      Serial.print("Servo:Rotate Goal:");
      //      Serial.println(goal);
      break;
    case 1:
      if (slowbl == true) {
        slowmove (Left, goal, ifdelay);
      } else {
        Left.write(goal);
      }
      //      Serial.print("Servo:Left Goal:");
      //      Serial.println(goal);
      break;
    case 2:
      if (slowbl == true) {
        slowmove (Right, goal, ifdelay);
      } else {
        Right.write(goal);
      }
      //      Serial.print("Servo:Right Goal:");
      //      Serial.println(goal);
      break;
    default:
      break;
  }
}
boolean slowmove(Servo s, int goal, boolean ifdelay) {
  int pos = s.read();
  if (pos < goal) { //positive
    for (int i = 0; i < goal - pos; i++) {
      s.write(s.read() + 1);
      if (ifdelay == true) {
        delay(15);

      } else {

      }
    }
    if (ifdelay == true) {
      Serial.print("Slowmove Servo from ");
      Serial.print(pos);
      Serial.print(" to");
      Serial.println(goal);
      Serial.print("Require time: ");
      Serial.print((goal - pos) * 15);
      Serial.println("ms");
    }
  } else {
    for (int i = 0; i < pos - goal; i++) {
      s.write(s.read() - 1);
      if (ifdelay == true) {
        delay(15);

      } else {

      }
    }
    if (ifdelay == true) {
      Serial.print("Slowmove Servo from ");
      Serial.print(pos);
      Serial.print(" to");
      Serial.println(goal);
      Serial.print("Require time: ");
      Serial.print((pos - goal) * 15);
      Serial.println("ms");
    }
  }
  return true;
}
void setup() {

  Right.attach(8);
  Rotate.attach(9);
  Grab.attach(10);
  Left.attach(11);
  pinMode(GrabButton, INPUT_PULLUP);
  pinMode(SoundButton, INPUT_PULLUP);
  pinMode(buzzer, OUTPUT);
  Serial.begin(9600);
  Serial.println("=== WGNetwork Robotic Arm OS v0.1 Alpha ===");
  Serial.println("Default Mode: Direct Control");
  lcd.init(); // 初始化LCD
  lcd.backlight(); //设置LCD背景等亮
  lcd.setCursor(0, 0);               //设置显示指针
  lcd.print("Robotic Arm Proj");     //输出字符到LCD1602上
  lcd.setCursor(0, 1);
  lcd.print("*:Direct Control");
  buzzsound();
  slowmove(Rotate, map(analogRead(A2) / 4, 0, 256, 0, 180), true);
  slowmove(Left, map(analogRead(A0) / 4, 0, 256, 80, 180), true);
  slowmove(Right, map(analogRead(A1) / 4, 0, 256, 60, 180), true);
  output();

  //  slowmove(Rotate,Initial[0]);
  //  slowmove(Left,Initial[1]);
  //  slowmove(Right,Initial[2]);
  //  slowmove(Grab,Initial[3]);
}

void loop()
{
  RotateRD = Rotate.read();
  LeftRD = Left.read();
  RightRD = Right.read();
  GrabRD = Grab.read();
  boolean SoundState = digitalRead(SoundButton);
  if (SoundState == 0) {
    buzzsound();
  }
  //debug
  // order: 0Rotate9 1Left11 2Right8 3Grab10
  // limit: 0~180 80~180 60~180 80~100
  if (mode == 0) {
    boolean GrabState = digitalRead(GrabButton);
    if (GrabState == 0) {
      Grab.write(0);
      Grabbing = true;
      output();
    } else {
      if (Grab.read() != 90) {
        slowmove(Grab, 90, true);
        Grabbing = false;
        output();
      }
    }
    //Rotate,Left,Right
    int pot[3] = {analogRead(A2), analogRead(A0), analogRead(A1)};
    int Intend[3] = {
      map(pot[0] / 8, 0, 128, 0, 180),//Rotate
      map(pot[1] / 4, 0, 256, 80, 180),//Left
      map(pot[2] / 4, 0, 256, 60, 180)//Right
    };
    int Read[3] = {RotateRD, LeftRD, RightRD};
    for (int i = 0; i < 3; i++) {
      if (Intend[i] != Read[i]) {
        if ((Intend[i] < Read[i]) && (Read[i] - Intend[i] > 2)) {
          servowrite(i, Intend[i], true, false);
          output();
        } else if ((Intend[i] > Read[i]) && (Intend[i] - Read[i]   > 2)) {
          servowrite(i, Intend[i], true, false);
          output();
        }
      }
    }
  }

  int j = 0;
  while (Serial.available() > 0) {
    comdata += char(Serial.read());
    delay(2);
    mark = 1;
  }
  if (mark == 1) {
    Serial.println(comdata);
    for (int i = 0; i < comdata.length() ; i++) {
      if (comdata[i] == ',') {
        j++;
      }
      else {
        numdata[j] = numdata[j] * 10 + (comdata[i] - '0');
      }
    }
    comdata = String("");
    mode = 1;
    Serial.println("Serial Mode On, Manual(Direct) Control has been disabled.");
    lcd.setCursor(0, 1);
    lcd.print("Serial Mode ON");
    buzzsound();
    for (int i = 0; i < 4; i++) {
      Serial.print(i);
      Serial.print(Servoname[i]);
      Serial.print(" = ");
      Serial.print(numdata[i]);
      if ((i == 1) && (numdata[1] < 80)) {
        Serial.println(" Invalid");
      } else if ((i == 2) && (numdata[2] < 60))
      { Serial.println(" Invalid");
      } else if ((i == 3) && (numdata[2] > 60))
      { Serial.println(" Invalid");
      } else {
        Serial.println(" Valid");
        if (i == 0) {
          slowmove(Rotate, numdata[0], true);
        } else if (i == 1) {
          slowmove(Left, numdata[1], true);
        } else if (i == 2) {
          slowmove(Right, numdata[2], true);
        } else {
          slowmove(Grab, numdata[3], true);
        }
      }
      numdata[i] = 0;
    }
    mark = 0;
  }
}

```
