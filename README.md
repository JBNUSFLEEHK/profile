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
