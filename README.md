# 자세교정알리미

본 프로젝트는 PC이용 중 거북목을 예방하기 위해 만든 자세교정 알리미입니다.

설계 및 순서도

![image](https://github.com/smw1875/-_-/assets/67328010/a3fff247-0f44-4f28-add9-e8701663720b)

예상 이미지

![image](https://github.com/smw1875/-_-/assets/67328010/bf1d22b2-73d2-4548-9582-2f6eb000e64f)

거리 측정을 위한 초음파센서 코드 및 이미지

![image](https://github.com/smw1875/-_-/assets/67328010/3a0d6a20-f7cd-4a8f-9bdc-e4376cd3450d)

![image](https://github.com/smw1875/-_-/assets/67328010/e9cbcf52-64e2-42f2-b966-b1c72c2f4e44)

결괏값

![image](https://github.com/smw1875/-_-/assets/67328010/b52360de-6ee9-4bd2-879b-c4a75984ee64)

알림을 위한 부저 코드 및 이미지

![image](https://github.com/smw1875/-_-/assets/67328010/76e36297-61b6-48cc-aaa5-8dfc01f93efb)

![image](https://github.com/smw1875/-_-/assets/67328010/a00417ce-bce7-4651-b217-61ce39d23fe5)

알리미 보조를 위한 LED 코드 및 이미지

![image](https://github.com/smw1875/-_-/assets/67328010/d6fbf1eb-d064-4a63-8efe-492af07b57d6)

허리 교정을 위한 가속도 센서 코드 및 이미지

![image](https://github.com/smw1875/-_-/assets/67328010/2da9aff4-290a-4b15-915b-72c6a1160b43)

완성 이미지

![image](https://github.com/smw1875/-_-/assets/67328010/62aa369c-3087-47ce-9f9a-5e9d9f91aac1)

1. 동작기능
A. 사용자와 모니터 사이 거리측정 기능
B. 허리교정 알리미 기능
C. 사용자 자세에 따른 시청각 알리미 기능
D. 일정시간(60분)마다 스트레칭 알람 기능

전체 소스코드 

import RPi.GPIO as GPIO
import time
from multiprocessing import Process
import schedule
import datetime
import sys
import board
import busio
import adafruit_adxl34x

GPIO.setmode(GPIO.BCM)
GPIO.setwarnings(False)

print ("start")
trig = 13
echo = 19
GPIO_pin =26
GPIO.setup(27,GPIO.OUT)
GPIO.setup(trig,GPIO.OUT)
GPIO.setup(echo,GPIO.IN)
GPIO.setup(GPIO_pin,GPIO.OUT)

scale = [ 261, 294, 329, 349, 392, 440, 493, 523 ]
list = [4, 4, 5, 5, 4, 4, 2, 4, 4, 2, 2, 1]
list2 = [0, 1, 2, 3, 4, 5, 6, 7, 7, 7, 7, 0]
list3 = [7, 6, 5, 4, 3, 2, 1, 0, 0, 0, 0, 0]

i2c = busio.I2C(board.SCL, board.SDA)
accelerometer = adafruit_adxl34x.ADXL345(i2c)
array = []

def acc() :
    while True :
        time.sleep(0.5)
        num = accelerometer.acceleration
        array.append(num)
        found = 10
        if num[0] > found :
          GPIO.output(27,True)
          p = GPIO.PWM(GPIO_pin,100)
          p.start(100)
          p.ChangeDutyCycle(90)          
          for i in range(12):  
             p.ChangeFrequency(scale[list3[i]])
             if i == 6:
               time.sleep(1)
             else :
               time.sleep(0.5)
          p.stop()
          GPIO.output(27,False)

        else:
          break
def timer():

           GPIO.output(27,True)
           p = GPIO.PWM(GPIO_pin,100)
           p.start(100)
           p.ChangeDutyCycle(90)
           for i in range(12):  
              p.ChangeFrequency(scale[list2[i]])
              if i == 6:
               time.sleep(1)
              else :
               time.sleep(0.5)
           p.stop()
           GPIO.output(27,False)

schedule.every(1).hours.do(timer)

try :
    while(True):     
      GPIO.output(GPIO_pin,False)
      GPIO.output(trig, False)
      time.sleep(0.5)
      GPIO.output(trig, True)
      time.sleep(0.00001)
      GPIO.output(trig, False)
      while GPIO.input(echo) == 0 :
        pulse_start = time.time()

      while GPIO.input(echo) == 1 :
        pulse_end = time.time() 

      pulse_duration = pulse_end - pulse_start
      distance = pulse_duration * 17000
      distance = round(distance, 2)
      print("Distance : ", distance, "cm")

      acc()
      schedule.run_pending()
      time.sleep(1)
      if distance <=30:
          GPIO.output(27,True)
      else:
          GPIO.output(27,False)
      if distance <=20:
          GPIO.output(27,True)
          p = GPIO.PWM(GPIO_pin,100)
          p.start(100)
          p.ChangeDutyCycle(90)
          
          for i in range(12):  
             p.ChangeFrequency(scale[list[i]])
             if i == 6:
               time.sleep(1)
             else :
               time.sleep(0.5)
          p.stop()
          GPIO.output(27,False)

except KeyboardInterrupt:
    GPIO.cleanup()
