# PYTHON 專題 
資管二 109403525 易靖程 
資管二 109403031 楊蓁 
## 專題名稱：防疫小幫手 
## 專題說明：
## 1. 背景:
 近年來世界各地飽受Covid-19病毒肆虐，於是我們跟進時事，想到我們可以做一個系統來查詢台灣各縣市的疫情，只要輸入想查詢的台灣縣市疫情，此系統便可一併列出此地目前累積確診人數、當日新增確診人數、最新第一劑與第二劑疫苗覆蓋率。
## 2. 目的:
 一鍵查詢台灣各縣市的疫情，如此便能更快速且清晰地了解台灣目前的疫情情況。
## 專題介紹
### 系統架構：GUI、爬蟲
### 流程圖:![](https://i.imgur.com/LKbsL90.jpg)


### 應用技術
#### * 爬蟲（requests、BeautifulSoup）
 到衛福部疾管署的各縣市區域每日新增確診網站爬取資料，以下為程式碼：
```python=
def catch():

#讀入輸入的縣市
country = input_country.get()
country = alt(country)

#建立可供查詢的縣市
countryList = ["台北市","新北市","桃園市","台中市","台南市","高雄市","基隆市",
               "新竹市","新竹縣","苗栗縣","彰化縣","南投縣","雲林縣","嘉義縣",
               "嘉義市","屏東縣","宜蘭縣","花蓮縣","台東縣","澎湖縣","金門縣","連江縣"]


#判斷是否有找到該縣市(Default = False)
isFind = False

#於countryList搜尋輸入縣市
#若找到則於疫情網站爬取相關資料，並設isFind = True
for i in range (len(countryList)):
    if country == countryList[i]:
        url = "https://covid-19.nchc.org.tw/city_confirmed.php?mycity=" + country
        request = requests.get(url)
        request.encoding="utf-8"
        data = bs4.BeautifulSoup(request.text, "html.parser")
        num = data.find("div", "col-lg-3 col-sm-6 col-6 text-center my-5").find('h1').string
        add = data.find("h1", "country_deaths mb-1 text-dark display-4").get_text()
        first = data.find("h1", "country_recovered mb-1 text-info display-4").get_text()
        second = data.find("h1", "country_recovered mb-1 text-danger display-4").get_text()
        date = data.find("div", "col-lg-3 col-sm-6 col-6 text-center my-5").get_text()
        isFind = True
        break
```
#### * GUI（tkinter、PIL）
設計一個使用者介面，讓使用者輸入要查詢的縣市，並用讀取條顯示爬蟲程式運行中，以下為程式碼：
```python=
#GUI   
window = tk.Tk()
window.title("防疫小幫手")
center_window(window, 600, 380)
window.resizable(False,False)
window.configure(bg = "#AEDDEF")
window.iconbitmap('ImageIcon/nurse.ico')

#Image
img = Image.open("ImageIcon/COVID-19.jpg")
img = ImageTk.PhotoImage(img)
imLabel=tk.Label(window,image = img)
imLabel.place(x = 0, y = 0)

#countryLabel
countryLabel = tk.Label(window,text = "請輸入要查詢的縣市", font = ("標楷體", 10))
countryLabel.configure(bg = "#AEDDEF")
countryLabel.place(x = 115, y = 270)

#Entry
temp = tk.StringVar()
input_country = tk.Entry(window, justify = "center", textvariable = temp)
input_country.place(x = 250, y = 270)

#progressbar
progressbar=ttk.Progressbar(window, length = 200, mode="determinate")  
progressbar.place(x = 220,y = 310)

#Label 
label = tk.Label(window, text = "等待中...", font = ("標楷體", 10))
label.place(x = 290, y = 340)
label.configure(bg = "#AEDDEF")

#Button
btn = tk.Button(text = "查詢", command = searching, font = ("標楷體", 10))
btn.place(x = 405, y = 268)
tk.messagebox.showinfo('訊息', '歡迎使用防疫小幫手~\n\n此系統可查詢台灣各縣市的\n‧目前累積確診人數\n‧當日確診人數\n‧疫苗覆蓋率\n\n請輸入要查詢的縣市') 
window.mainloop()
```
#### * 平行處理（threading）
透過Threading來達到一邊跑讀取條一邊爬蟲，使兩個不同程式能同時進行，並設條件判斷哪支程式已經完成，以下為程式碼
```python=
#平行處理使程式同時跑兩個Function
t = threading.Thread(target = catch)
t.start()

#判斷Thread t是否已經完成
if threading.active_count() == 7:
                progressbar["value"] += 1
                window.update()             
                time.sleep(0.05)          
            else:
                progressbar["value"] = progressbar["maximum"]
                label["text"] = "已完成!"
                window.update() 
                show()
                break;

```
#### * 時間延遲（time）
使用time.sleep()來等待爬蟲的時間，同時也可以控制讀取條的速度，以下為程式碼
```python=
#設置讀取條跑的速度，若catch已經跑完則設定讀取條完成
for i in range(progressbar["maximum"]):
    if progressbar["value"] < progressbar["maximum"]:
        if threading.active_count() == 7:
            progressbar["value"] += 1
            window.update()             
            time.sleep(0.05)          
        else:
            progressbar["value"] = progressbar["maximum"]
            label["text"] = "已完成!"
            window.update() 
            show()
            break;

```

## 成果畫面
### 初始畫面
![](https://i.imgur.com/rN8ZAq3.png)
### 查詢過程
![](https://i.imgur.com/nXx94D2.png)
### 查詢結果
![](https://i.imgur.com/tqlfu6C.png)
### 輸入錯誤
![](https://i.imgur.com/MYArfgo.png)

## 完整程式碼
```python=
import requests
import bs4
import tkinter as tk
from tkinter import ttk
from PIL import ImageTk, Image
from tkinter import messagebox
import threading
import time      

#若輸入縣市有臺則改為台以符合爬蟲網頁之網址
def show():
    country = input_country.get()
    country = alt(country)
    
    #若找到則顯示查詢結果，未找到則顯示錯誤
    if isFind:
        tk.messagebox.showinfo('查詢結果', '目前{}累積確診人數為{}'.format(country,num) + '人' 
                                       + "\n今日{}新增確診人數為{}".format(country,add) + '人'
                                       + "\n截至{}第一劑{}的疫苗覆蓋率為{}".format(date.split()[2],country,first)
                                       + "\n截至{}第二劑{}的疫苗覆蓋率為{}".format(date.split()[2],country,second))
    else:
        time.sleep(0.5) 
        tk.messagebox.showerror('錯誤', '輸入錯誤，請重新輸入!')
    
    #將輸入設為空的
    temp.set('')

def alt(country):
    
    
    seperate_country = list(country)
    
    for i in range(len(seperate_country)): 
        if seperate_country[i] == '臺':
            seperate_country[i] = '台'
            country = "".join(seperate_country)
            return country
        
    return country

#爬取資料
def catch():
    
    global isFind, num, add, first, second, date
    
    #讀入輸入的縣市
    country = input_country.get()
    country = alt(country)

    #建立可供查詢的縣市
    countryList = ["台北市","新北市","桃園市","台中市","台南市","高雄市","基隆市",
                   "新竹市","新竹縣","苗栗縣","彰化縣","南投縣","雲林縣","嘉義縣",
                   "嘉義市","屏東縣","宜蘭縣","花蓮縣","台東縣","澎湖縣","金門縣","連江縣"]
    
    
    #判斷是否有找到該縣市(Default = False)
    isFind = False
    
    #於countryList搜尋輸入縣市
    #若找到則於疫情網站爬取相關資料，並設isFind = True
    for i in range (len(countryList)):
        if country == countryList[i]:
            url = "https://covid-19.nchc.org.tw/city_confirmed.php?mycity=" + country
            request = requests.get(url)
            request.encoding="utf-8"
            data = bs4.BeautifulSoup(request.text, "html.parser")
            num = data.find("div", "col-lg-3 col-sm-6 col-6 text-center my-5").find('h1').string
            add = data.find("h1", "country_deaths mb-1 text-dark display-4").get_text()
            first = data.find("h1", "country_recovered mb-1 text-info display-4").get_text()
            second = data.find("h1", "country_recovered mb-1 text-danger display-4").get_text()
            date = data.find("div", "col-lg-3 col-sm-6 col-6 text-center my-5").get_text()
            isFind = True
            break



#顯示讀取條
def searching():
    progressbar["value"] = 0
    label["text"] = "查詢中..."
    
    #平行處理使程式同時跑兩個Function
    t = threading.Thread(target = catch)
    t.start()

    #設置讀取條跑的速度，若catch已經跑完則設定讀取條完成
    for i in range(progressbar["maximum"]):
        if progressbar["value"] < progressbar["maximum"]:
            if threading.active_count() == 7:
                progressbar["value"] += 1
                window.update()             
                time.sleep(0.05)          
            else:
                progressbar["value"] = progressbar["maximum"]
                label["text"] = "已完成!"
                window.update() 
                show()
                break;

    #重置讀取條
    if progressbar["value"] == progressbar["maximum"]:
        time.sleep(0.5)
        progressbar["value"] = 0
        label["text"] = "等待中..."

#使視窗位於螢幕中間
def center_window(window, width, height):
    screenwidth = window.winfo_screenwidth()
    screenheight = window.winfo_screenheight()
    size = '%dx%d+%d+%d' % (width, height, (screenwidth - width)/2, (screenheight - height)/2)
    window.geometry(size)

#GUI   
window = tk.Tk()
window.title("防疫小幫手")
center_window(window, 600, 380)
window.resizable(False,False)
window.configure(bg = "#AEDDEF")
window.iconbitmap('ImageIcon/nurse.ico')

#Image
img = Image.open("ImageIcon/COVID-19.jpg")
img = ImageTk.PhotoImage(img)
imLabel=tk.Label(window,image = img)
imLabel.place(x = 0, y = 0)

#countryLabel
countryLabel = tk.Label(window,text = "請輸入要查詢的縣市", font = ("標楷體", 10))
countryLabel.configure(bg = "#AEDDEF")
countryLabel.place(x = 115, y = 270)

#Entry
temp = tk.StringVar()
input_country = tk.Entry(window, justify = "center", textvariable = temp)
input_country.place(x = 250, y = 270)

#progressbar
progressbar=ttk.Progressbar(window, length = 200, mode="determinate")  
progressbar.place(x = 220,y = 310)

#Label 
label = tk.Label(window, text = "等待中...", font = ("標楷體", 10))
label.place(x = 290, y = 340)
label.configure(bg = "#AEDDEF")

#Button
btn = tk.Button(text = "查詢", command = searching, font = ("標楷體", 10))
btn.place(x = 405, y = 268)
tk.messagebox.showinfo('訊息', '歡迎使用防疫小幫手~\n\n此系統可查詢台灣各縣市的\n‧目前累積確診人數\n‧當日確診人數\n‧疫苗覆蓋率\n\n請輸入要查詢的縣市') 
window.mainloop()




```


