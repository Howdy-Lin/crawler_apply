# 爬蟲應用(抓取7-11與家樂福商品資訊)
## 目標抓取衛生紙資訊並存入csv檔案裡
呈現結果如下圖:

## (1) 使用selenium操作網頁
目的是要能做到自動進入7-11網站使用搜尋功能、換頁<br>
<br>
**搜尋:**<br>
如下圖，找到搜尋框的class <br>
<p align="center"><img src="https://user-images.githubusercontent.com/74965449/158956260-ed6403d0-6f99-45b8-92d1-81fa3f357e8d.png" width="50%" height="50%"></p><br>
使用 driver.find_element_by_class_name，並傳入衛生紙字串後啟用enter執行即可進入衛生紙頁面<br>
<br>

**換頁:**<br>
關鍵在這下圖這部分
<p align="center"><img src="https://user-images.githubusercontent.com/74965449/158957615-6026aadc-baae-4b56-aa18-ead0ba1be7e1.png" width="50%" height="50%"></p><br>
倘若有下一頁就抓取下一頁的超連結網址，並讓google driver切換過去，直到沒有下一頁為止就跳出迴圈

```bash
while  ('下一頁'in next_page)==True:
    
    record_data(elements)
    
   # print(title1,title2)
    catalog = url.find(class_='pagelist1')
    next_page = catalog.find_all('a',string='下一頁')   #抓含有下一頁超連結的element
    print(next_page)
    for href in next_page:            #要用for迴圈才可取得next_page裡的element
        url = href.get('href')
        driver.get(url)
        url = BeautifulSoup(driver.page_source, 'html.parser')   #抓下當前使用網址的html
        elements = url.find_all(class_='prodM')  
        goods = url.find_all(class_='headT')
    url = BeautifulSoup(driver.page_source, 'html.parser')   #抓下當前使用網址的html
    catalog = url.find(class_='pagelist1')
    next_page = catalog.find_all('a',string='下一頁')   #抓含有下一頁超連結的element
    next_page = str(next_page)   #建立新的next_page 讓while迴圈可以繼續跑   
record_data(elements)  #抓取最後一頁的資料
```

## (2) 爬蟲用法與資料篩選

**第一步:**<br>
先想好我們要抓那些資料，在此我選擇抓衛生紙的商品圖案、品名、價格、商品超連結<br>
解析網站的html後可發現我們所要的資訊都包含在class_='prodM'裡<br>
其中品名包在wordwidth、圖片包在img、連結網址則是從圖片中的src抓取出來<br>

```bash
element1 = element.find(class_='wordwidth').getText().strip()
pics = element.find_all('img')
image_links = [pic.get("src") for pic in pics]
image_links ='https://shop.7-11.com.tw'+image_links[0]   
dfimage.append(image_links)
url1=image_links
r=requests.get(url1)
```
**第二步:**<br>
取出品牌與衛生紙類型的方式<br>
這邊使用.partition與.split來做資料的切割整理<br>
最後最後，把我們爬下來的資料通通定義在list[]裡
```bash
dfname=[]
dffac=[]
dfcateg=[]
dfprice=[]
dfimage=[]
dfgoods=[]
```
## (3) 將資料轉存為csv檔即完成
這邊使用pandas的方式來將資料寫入csv
```bash
import pandas as pd            
print(len(dfname))
print(len(dffac))
print(len(dfcateg))
print(len(dfprice))
print(len(dfimage))
dict = {'品名':dfname,'品牌':dffac,'種類':dfcateg,'價格':dfprice,'圖片':dfimage,'網址':dfgoods}
print(dict)
df = pd.DataFrame(dict) 
df.to_csv('reptile711.csv')
```

## 參考資料
**爬蟲:**  https://www.learncodewithmike.com/2020/02/python-beautifulsoup-web-scraper.html<br>
**selenium:**  https://medium.com/marketingdatascience/selenium%E6%95%99%E5%AD%B8-%E4%B8%80-%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8webdriver-send-keys-988816ce9bed
