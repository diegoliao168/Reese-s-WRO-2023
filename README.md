# 移動性管理

我們的機器人是採用與汽車相同的轉向邏輯和驅動方式。在前輪部分我們使用伺服馬達來控制我們的轉向機構，後輪的部分我們使用的是直流馬達。我們採用的是Arduino的Uno版和OpenMV的H7，Arduino的主要功用與控制轉向的伺服馬達溝通和後輪的直流馬達進行溝通。OpenMV的功用就像我們人的眼睛，透過影相辨識的方式來達到跟Arduino溝通控制車哪時轉向和哪時加速或減速。

在選擇馬達的時候我們找到兩種適合我們使用的馬達分別有4000rpm負載500公克的馬達和320rpm負載1500公克的馬達，考慮到我們的小車可能會需要拖動較大的載重。我們選擇了320rpm轉速的馬達負載1500公克的馬達。這顆馬達的型號為GA25-370、工作電壓六到十二伏特、功率2.5瓦、額定電流0.65安培、齒輪箱速比1:32.2。馬達之後有加速齒比帶動傳動軸來帶動後輪的兩顆輪子。


在前輪的部分我們選擇使用精準控制轉向角度的伺服馬達是MG996R。MG996R
工作電壓4.8V-7.2V、空載工作電流120mA。

因為一開始購買車輛的時候廠家沒有設計放電池的地方。因此我們3D列印了一塊在車子後的背板，為的就是能讓我們到電池有地方可以固定

![](https://hackmd.io/_uploads/S12IA2hon.png)
# 電源和感測器管理
接下來是我們的車體結構是與市售的商家購買的，但是經過我們的改裝我們的車輛已經與市售的截然不同。我們車體裝上支架為了OpenMV和電池盒有辦法固定。應為電力不足得原因我們在車商裝了三套電池系統。第一套十二伏電池系統是供應主要的驅動馬達，接下來第二套八伏特的電池是供應給車上主要的溝通系統Arduino也是供應給前輪的轉向系統的伺服馬達，最後一套第三套四伏特的電池系統是給OpenMV單獨供電的為了讓OpenMV提供更穩定的訊息傳輸。

要讓我們與程式Arduino正確溝通我們需要Arduino IDE，Arduino IDE使與的程式語言是C語言透過C語言的溝通，我們的Arduino才能跟OpenMV和其他馬達傳送正確的指令。接下來OpenMV所使用的語言是Micro Python，Micro Python對我們來講是個全新的語言。當我們一開始使用的時候，我們一邊詢問我們的指導老師一遍看網路上的教學影片。開始撰寫OpenMV的程式從一開始的調顏色數值到顏色追蹤和如何用影像辨識距離。我們在寫OpenMV的程式時遇到最大的困難是在我們調顏色數值的時後，顏色數值可能會應為些微的燈光變化而有所不同。我們不得不反覆測試直到調到一個較好的數值。

原先我們使用的控制板是Arduino的Nano版經過我們的測試我們發現使用Nano版的時候Nano版的供電系統會不足以支撐兩顆馬達同時驅動，使轉向系統的伺服馬達無法轉到目標位子。經過測試後我們採用供電比較穩定的Arduino的Uno版，對支援更穩定的供電和更加優良的擴充。

經過我們的反覆測發現到原本8伏特的電池來去推動馬達的時後馬達的功率無法驅動我們的車。接下來我們就把原本的電壓乘以二到了十六伏特，但是我們沒有意識到我們所使用Arduino Uno的只能對應到十二伏特的電壓。應此我們燒壞了一塊主機板。經過這次經驗我們換了新的十二伏電池盒。也是因為改回了十二伏的電池，讓我們的機器人正常的執行任務，以及進行接下來的測試和實驗

# 障礙管理





程式想法
先將OpenMV的畫面分成六等份，先拿第三和第四個畫面來使用，因為在畫面正中間，
```
import sensor, image, time, math
from pyb import UART

threshold_index = 0




thresholds_black = [(38, 0, 100, -50, 74, -30)]
thresholds_red = [(8, 56, 127, 11, -128, 127)]
thresholds_green = [(25, 17, -24, -4, -28, 13)]
thresholds_blue = [(18, 45, 27, -7, -103, -7)]
thresholds_orange = [(47, 100, 71, 9, 127, -105)]

sensor.reset()
sensor.set_pixformat(sensor.RGB565)
sensor.set_framesize(sensor.QVGA)
sensor.skip_frames(time = 2000)
sensor.set_auto_gain(False)
sensor.set_auto_whitebal(False)
clock = time.clock()
uart = UART(3, 19200)
m = 0
angle = 0    #轉彎角度
mot_1 = 200  #轉彎PWM
mot_2 = 200   #直走PWM
middle = 95  #伺服馬達中間值





while(True):

    x3 = 0
    x4 = 0
    y3 = 0
    y4 = 0
    two_m = 0
    two = 0


    clock.tick()
    img = sensor.snapshot()
    #找橘色
    if m == 0 :
        for blob in img.find_blobs([thresholds_orange[0]], pixels_threshold=400, area_threshold=400, merge=True):

            if blob.elongation() > 0.5:
                #img.draw_edges(blob.min_corners(), color=(255,0,0))
                img.draw_line(blob.major_axis_line(), color=(0,255,0))
                print( blob.rotation_deg())
                m = blob.rotation_deg()
                #img.draw_line(blob.minor_axis_line(), color=(0,0,255))
                print (m)
                print (clock)
                if m > 90:
                    angle = 65
                else :
                    angle = 125


    threemax_x = 0
    threemax_y = 0
    #3
    for blob in img.find_blobs([thresholds_black[threshold_index]], pixels_threshold=200, area_threshold=200, merge=True,roi=(108,0,54,250)):

        '''
        if blob.elongation() > 0.5:
            img.draw_edges(blob.min_corners(), color=(255,0,0))
            img.draw_line(blob.major_axis_line(), color=(0,255,0))
            img.draw_line(blob.minor_axis_line(), color=(0,0,255))
        '''
        if blob.cy() > threemax_y:
            threemax_x = blob.cx()
            threemax_y = blob.cy()
    img.draw_rectangle(blob.rect())
    img.draw_cross(blob.cx(), blob.cy())
    x3 = threemax_x
    y3 = threemax_y

    img.draw_keypoints([(blob.cx(), blob.cy(), int(math.degrees(blob.rotation())))], size=20)
    if  threemax_y > 100:
        uart.write(str(mot_1)+","+ str(angle)+"E\n")
        time.sleep_ms(20)
    else:
        uart.write(str(mot_2)+","+ str(middle)+"E\n")
        time.sleep_ms(20)

    fourmax_x = 0
    fourmax_y = 0
    #4
    for blob in img.find_blobs([thresholds_black[threshold_index]], pixels_threshold=200, area_threshold=200, merge=True,roi=(162,0,54,250)):

        '''
        if blob.elongation() > 0.5:
            img.draw_edges(blob.min_corners(), color=(255,0,0))
            img.draw_line(blob.major_axis_line(), color=(0,255,0))
            img.draw_line(blob.minor_axis_line(), color=(0,0,255))
        '''
        if blob.cy() > fourmax_y:
            fourmax_x = blob.cx()
            fourmax_y = blob.cy()
    img.draw_rectangle(blob.rect())
    img.draw_cross(blob.cx(), blob.cy())
    x4 = fourmax_x
    y4 = fourmax_y
    if fourmax_y > 100:
        uart.write(str(mot_1)+","+ str(angle)+"E\n")
        time.sleep_ms(20)
    else:
        uart.write(str(mot_2)+","+ str(middle)+"E\n")
        time.sleep_ms(20)

    img.draw_keypoints([(blob.cx(), blob.cy(), int(math.degrees(blob.rotation())))], size=20)
    
```
