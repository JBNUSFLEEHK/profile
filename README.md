# 작물의 최적환경을 lineNotify로 구현하기 
--------------------------------------------------------------------

```
print('import math
import requests
import time
import threading
import matplotlib.animation as animation
import matplotlib.pyplot as plt
import datetime

dateList = []
vpdList = []
startMain = True
vpd = 0.0



def loopGetData():
    global dateList
    global vpdList
    global startMain
    global vpd
    while True:
        url = 'https://api.thingspeak.com/channels/1680866/feeds.json?api_key=MWTMCIB9XJ6IW7IV&results=2'
        r = requests.get(url)
        date = None
        temp = 0.0
        humid = 0.0
        splitData = r.text.split("feeds")[1].split(",{")[1].split(",")
        for comma in splitData:
            print(comma)
            if comma.split(":")[0] == "\"created_at\"":
                date = comma.split("\":\"")[1].replace('Z"', "")
            elif comma.split(":")[0] == "\"field1\"":
                temp = float(comma.split(":")[1].replace('"', ""))
            elif comma.split(":")[0] == "\"field2\"":
                humid = float(comma.split(":")[1].replace('"', ""))
        vpd = (0.61078 * math.e ** (temp / (temp + 233.3) * 17.2694)) * ((1 - (humid) / 100))

        dateList.append(datetime.datetime.now())
        vpdList.append(vpd)


        time.sleep(2)
------------------------------------------------------------------------------------
저희 조는 split을 사용해 필요하지 않은 문자와 기호인 “”,:, created at 등을 제거했습니다. 
데이터에 공식을 사용하는 부분에서는 vpd를 구하는 과정에서 자연상수 e가 필요한데 math패키지를 사용해 math.e로 구현했습니다. 
그리고 구한 값을 리스트에서 append를 통해 계속 main에 넘겨줬습니다. 

def pltGraph():
    global dateList
    global vpdList

    while True:
        fig = plt.figure()
        ax = plt.axes()
        x = dateList
        y = vpdList
        global startMain
        line = ax.plot(x, y)
        print(x, y)

        def update(num, x, y, line):
            line.set_data(x[:num], y[:num])
            return line,


        plt.close()
        plt.clf()
        plt.plot(x, y)
        plt.draw()
        plt.pause(1)

--------------------------------------------------------------------------------
import matplotlib.pyplot as plt를 사용했고
def pltGraph라는 함수 내에 x변수에 datelist(날짜), y 변수 내에 vpdlist를 넣어 x가 시간을 다뤄 시간이 지남에 따라 y값에 vpd가 나오도록 함수를 설정했습니다. 

def lineNotify():
    while True:
        global vpd
        if vpd > 1.2:
            TARGET_URL = 'https://notify-api.line.me/api/notify'
            TOKEN = 'sgiakqYbahIUVYIcrhYJqxvms9BYselUlt66rYQkon4'
            response = requests.post(
                TARGET_URL,
                headers={
                    'Authorization': 'Bearer ' + TOKEN
                },
                data={
                    'message': '물을 시비해주세요. / 가습을 해주세요.'
                }
            )
            time.sleep(40)

        elif vpd < 0.8:
            TARGET_URL = 'https://notify-api.line.me/api/notify'
            TOKEN = 'sgiakqYbahIUVYIcrhYJqxvms9BYselUlt66rYQkon4'
            response = requests.post(
                TARGET_URL,
                headers={
                    'Authorization': 'Bearer ' + TOKEN
                },
                data={
                    'message': '난방을 해주세요. / 환기를 해주세요.'
                }
            )
            time.sleep(40)
        time.sleep(1)

--------------------------------------------------------------------------------
LineNotify를 통해 적정량 미만 또는 초과일 시, 알림이 오도록 설정하고자 한건데 이 과정은 직접 Line 사이트 내에 들어가 python 구현 방법을 복사하여 사용했습니다. 
하지만 적정량을 설정하는 부분은 저희가 직접 설정을 했는데 if, elif를 통해 적정값에 초과인 상황과 미만인 상황을 설정했고 적정값일 때는 알림이 오지 않도록 했습니다. 
그리고 알림이 매순간 짧은 시간마다 오게 된다면 불편함이 생길 것이기 때문에 time.sleep()부분에서 값을 600seconds, 10분 정도로 설정했습니다. 하지만 이 값은 코드 내에서 자신이 필요하다면 더 짧게, 더 길게 설정하여 바꿀 수 있습니다. 

def main():
    th = threading.Thread(target=loopGetData)
    th.start()
    th2 = threading.Thread(target=pltGraph)
    th2.start()
    th3 = threading.Thread(target=lineNotify)
    th3.start()


if __name__=="__main__":
    main()
')
```
