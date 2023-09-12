from machine import Pin, PWM
from time import sleep_ms, sleep_us, ticks_us, ticks_diff, ticks_ms

import time

# Pinnar fyrir mótor A, ai1 og ai2 stjórna í hvora
# áttina mótorinn snýst en pwmA stjórnar hraðanum
ai1 = Pin(18, Pin.OUT)
ai2 = Pin(17, Pin.OUT)
pwmA = PWM(Pin(16, Pin.OUT), 10000)

# Pinnar fyrir mótor B
bi1 = Pin(11, Pin.OUT)
bi2 = Pin(10, Pin.OUT)
pwmB = PWM(Pin(9, Pin.OUT), 10000)

echo = Pin(47, Pin.IN)
trig = Pin(48, Pin.OUT)

def maela_fjarlaegd():
    # Sendum 10 míkrosekúndna púls
    trig.value(1)
    sleep_us(10)
    trig.value(0)
    
    # Bíðum eftir að svarpúlsinn byrji
    while not echo.value(): # eða echo.value() == 0
        pass # Ekkert að gera nema bíða
    
    # Svarpúlsinn er byrjaður að berast þannig að við setjum skeiðklukku í gang
    upphafstimi = ticks_us()
    
    # Bíðum eftir að svarpúlsinn endi
    while echo.value(): # eða echo.value() == 1
        pass # Ekkert að gera nema bíða
    
    # Svarpúlsinn er ekki lengur að berast þannig að við stoppum skeiðklukkuna
    endatimi = ticks_us()
    
    # Reiknum muninn á upphafs og endatímanum
    heildartimi = ticks_diff(endatimi, upphafstimi)
    
    # Notum svo helildartímann til að reikna út fjarlægðina skv. jöfnunni fjarlægð = hraði * tími
    # byrjum á að helminga heildartímann (merkið fer fram og til baka)
    heildartimi /= 2
    # Hljóðhraðinn er 340 m/s (34000 cm/s) og svo deilum við með 1000000 til að fá cm á míkrósekúndur.
    hljodhradi = 34000 / 1000000
    # Reiknum loks fjarlægðina í cm
    fjarlaegd = heildartimi * hljodhradi
    
    # Skilum svo fjarlægðinni sem heiltölu sem er næg nákvæmni
    return int(fjarlaegd)

def afram(hradi): # hraði er á bilinu 0 til og með 1023
    # Mótor A stilltur á áfram
    ai1.value(1)
    ai2.value(0)

    # Mótor B stilltur á áfram
    bi1.value(1)
    bi2.value(0)
 
    # Sami hraði settur á báða mótorana
    pwmA.duty(hradi)
    pwmB.duty(hradi)
def tilbaka(hradi):
    ai1.value(0)
    ai2.value(1)

    # Mótor B stilltur á áfram
    bi1.value(0)
    bi2.value(1)
 
    # Sami hraði settur á báða mótorana
    pwmA.duty(hradi)
    pwmB.duty(hradi)
    
def turnRight(right):
    ai1.value(0)
    ai2.value(0)
    
    bi1.value(1)
    bi2.value(0)
    pwmA.duty(500)
    pwmB.duty(500)

def turnLeft(left):
    ai1.value(1)
    ai2.value(0)
    bi1.value(0)
    bi2.value(0)
    pwmA.duty(500)
    pwmB.duty(500)

    
while True:
    fjarlaegd = maela_fjarlaegd()
    print(f"Mæld fjarlægð: {fjarlaegd}")
    afram(1000)
    
    if (fjarlaegd)<50:
        afram(0)
        sleep_ms(800)
        turnLeft(1)
        sleep_ms(800)
        fjarlaegd1 = maela_fjarlaegd()
        print("fjarlaegd1:",fjarlaegd1)
        sleep_ms(800)
        turnRight(1)
        sleep_ms(800)
        turnRight(1)
        sleep_ms(800)
        fjarlaegd2 = maela_fjarlaegd()
        print("fjarlaegd2:",fjarlaegd2)
        turnLeft(1)
        sleep_ms(800)
        if fjarlaegd1>50: 
            turnLeft(1)
            sleep_ms(800)
        elif fjarlaegd2>50:
            turnRight(1)
            sleep_ms(800)
                
        else:
            print("going back")
            tilbaka(1000)
            sleep_ms(800)
            turnRight(1)
            sleep_ms(800)
            turnRight(1)
            sleep_ms(800)
    
    
    
