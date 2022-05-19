---
layout : post
title : "데이터 수집 실습내용" 
date : "2022-05-18"
categories : Blog
---



# 데이터 수집(Webscraping)

## 데이터 분석을 위한 pandas 이용


```python
import pandas as pd

# 데이터프레임 저장하기 (인덱스는 제외하고 저장)
file_name = "___.csv"
df.to_csv(file_name, index=False)

# 데이터프레임 불러오기
pd.read_csv(file_name)

# Current Diroectory Location 확인
%pwd

```

## 웹에서 데이터 가져오기


```python
# 네이버 금융 사이트 - 특정종목 뉴스 URL
item_code = 300080 #플리토의 종목코드
page_no = 1 
url = f"https://finance.naver.com/item/news_news.nhn?code={item_code}&page={page_no}&sm=title_entity_id.basic&clusterId="
print(url)
```

    https://finance.naver.com/item/news_news.nhn?code=300080&page=1&sm=title_entity_id.basic&clusterId=
    

### 데이터 가져오기1 (pandas 사용)
* pandas의 read_html 사용 
    * 해당 URL의 테이블(html)을 데이터프레임들의 리스트로 반환


```python
import pandas as pd

table = pd.read_html(url, encoding="cp949" or "utf-8") #table은 리스트!
table[0][:5] #table의 각 리스트는 데이터프레임
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>제목</th>
      <th>정보제공</th>
      <th>날짜</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>플리토, 7.4억 규모 다국어 번역 용역 위탁계약 체결</td>
      <td>이데일리</td>
      <td>2022.02.22 16:33</td>
    </tr>
    <tr>
      <th>1</th>
      <td>플리토, 2.9억원 규모 자사주 처분 결정</td>
      <td>이데일리</td>
      <td>2022.01.18 14:59</td>
    </tr>
    <tr>
      <th>2</th>
      <td>연관기사 목록  플리토, 3억 규모 자사주 처분 결정…자사주 상여 지급  헤럴드경제...</td>
      <td>연관기사 목록  플리토, 3억 규모 자사주 처분 결정…자사주 상여 지급  헤럴드경제...</td>
      <td>연관기사 목록  플리토, 3억 규모 자사주 처분 결정…자사주 상여 지급  헤럴드경제...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>플리토, 3억 규모 자사주 처분 결정…자사주 상여 지급</td>
      <td>헤럴드경제</td>
      <td>2022.01.18 13:56</td>
    </tr>
    <tr>
      <th>4</th>
      <td>플리토, 3억 규모 자사주 처분 결정</td>
      <td>아시아경제</td>
      <td>2022.01.18 13:39</td>
    </tr>
  </tbody>
</table>
</div>



### 데이터 가져오기2 (requests 사용)
* requests 방식
    * get : 필요한 데이터를 Query String 에 담아 전송
    * post : 전송할 데이터를 HTTP 메시지의 Body의 Form Data에 담아 전송
    * get 과 post 여부는 브라우저의 네트워크 탭의 Headers > Request Method 를 통해 확인

* 참고자료
    * [Requests: HTTP for Humans™ — Requests documentation](https://requests.readthedocs.io/en/master/)
    * [Quickstart — Requests documentation # custom-headers](https://requests.readthedocs.io/en/latest/user/quickstart/#custom-headers)


```python
import requests

# 로봇이 아닌 브라우저 조회임을 알리기 위해 headers를 사용
headers = {"user-agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/101.0.4951.54 Safari/537.36"}

# get 방식으로 html 조회
response = requests.get(url, headers=headers)

# 정상 호출되는지 확인 / 보통 200은 정상
response.status_code

# BeautifulSoup으로 데이터 찾기
from bs4 import BeautifulSoup as bs
soup = bs(response.text, "lxml")
table = soup.select("table") # 같은것 soup.table == soup.find("table") / soup.find_all("table")
table[0]

```

#### BeautifulSoup 의 select 기능을 통한 링크 태그 찾기
* 개발자도구(F12) 중 Network => docs, Fetch/Xhr, JS 중에서 찾음
* 찾는 정보 우클릭 => inspection => 해당 태그 => copy selector를 찾음


```python
#copy selector가 아래와 같음
#content > div > div.view-content > div > table > tbody > tr:nth-child(1) > td.data-title.aLeft > a
a_list = html.select("#content > div > div.view-content > div > table > tbody > tr > td > a") # copy selector를 찾아라!
a_list[0]
```

## 가져온 데이터의 가공

### 데이터프레임 붙이기
* concat :  
    * axis=0 행을 기준으로 위아래로 같은 컬럼끼리 값을 이어 붙여 새로운 행을 만듦
    * axis=1 컬럼을 기준으로 인덱스가 같은 값을 옆으로 붙여 새로운 컬럼을 만듦
    * reset_index(drop=True)를 통해 데이터프레임을 붙일 때 기존 인덱스는 버리고 인덱스를 새로 만듬


```python
# pd.concat 
df = pd.concat([df1, df2, df3], axis = 0 or 1).reset_index(drop=True)
```

### 데이터프레임 중 결측치 제거
* dropna :
    * how : all은 모두가 결측치인 데이터만, any는 결측치가 하나라도 있으면 제거
    * axis : 0은 행을 제거, 1은 열을 제거


```python
# dropna, 
df = df.dropna(how="all", axis=0).dropna(how="all", axis=1)
```

### 특정 데이터 제거(contains)
* .str.contains
    * <font color="red">조건으로 .str.contains 를 사용하며 조건의 반대에는 앞에 ~ 표시로 표현할 수 있습니다.</red>
    * 자료형은 bool 형식으로 True 또는 False로 반환됨


```python
# "제목" Column 중 "연관기사" 문구가 포함된 데이터가 있는 행은 "~" 때문에 False => 인덱싱에서 빠짐
# "제목" Column 중 "연관기사" 문구가 포함되지 않은 데이터가 있는 행은 True => 인덱싱 됨
df = df_temp[~df_temp["제목"].str.contains("연관기사")]
```

## 기타

### FinanceDataReader 란?

* 한국 주식 가격, 미국주식 가격, 지수, 환율, 암호화폐 가격, 종목 리스팅 등 금융 데이터 수집 라이브러리

    * [FinanceData/FinanceDataReader: Financial data reader](https://github.com/FinanceData/FinanceDataReader)
    * [FinanceDataReader 사용자 안내서 | FinanceData](https://financedata.github.io/posts/finance-data-reader-users-guide.html)
    * https://pandas-datareader.readthedocs.io/en/latest/readers/index.html
* 설치하기
    * !pip install -U finance-datareader
* 버전확인 
    * fdr.__version__


```python
import FinanceDataReader as fdr

# 한국거래소 상장종목 전체 가져오기
df_krx = fdr.StockListing("krx")

# krx 데이터프레임 중 Name, Symbol만 가져오기
df_krx = df_krx[["Name", "Symbol"]]

# 플리토의 Symbol 찾기
symbol = df_krx.loc[df_krx["Name"] == "플리토", "Symbol"].iloc[0]
## df 인덱싱인 loc를 통해 name이 플리토인 행을 인덱싱, symbol 열을 갖는 시리즈 형태에서
## iloc[0] 인덱싱을 통해 data 형태로 가져옴
## iloc[0] 대신에 values[0]도 사용 가능

symbol
## 플리토의 코드(symbol)은 035720
```




    '300080'



### 파생변수 만들기
* '종목코드'와 '종목명' column을 추가하면서 각각 item_code와 item_name 값을 입력합니다.


```python
df["종목코드"] = item_code #df에 "종목코드" 열이 추가됨
df["종목명"] = item_name #df에 "종목명" 열이 추가됨
```

### 컬럼 순서 변경하기
* DataFrame에서 column 들의 이름을 순서를 조정하여 column순서를 변경할 수 있습니다.


```python
# 종목코드와 종목명 코드를 앞으로 옮김
cols = ['종목코드', '종목명', '날짜', '종가', '전일비', '시가', '고가', '저가', '거래량']
df = df[cols]
```

### 데이터 수집시 서버에 요청 간격두기(매너)


```python
import time
while True:
    time.sleep(0.1) #0.1초 쉬어간다
```

### 리스트컴프리헨션
* 한줄로 편리하게 사용하며 더 빠름?, 리스트에 append 사용 필요없음


```python
# 리스트컴프리헨션을 통해 리스트데이터 전처리시 for문을 한줄로 씀
a_link_no = [a["href"].split("/")[-1] for a in a_list]
```

### 일별 시세를 확인하는 함수


```python
import time
import requests
import pandas as pd

def get_item_list(item_code, item_name):
    page_no = 1
    df_temp = []
    prev = ""
    while True:
        time.sleep(0.01)
        url = f"https://finance.naver.com/item/sise_day.naver?code={item_code}&page={page_no}"
        table = pd.read_html(requests.get(url, headers={'User-agent':"Mozilla/5.0"}).text)
        temp = table[0].dropna(how="all", axis=0)
        curr = temp["날짜"].iloc[0]
        if curr == prev :
            break
        df_temp.append(temp)
        prev = curr
        print("*", end="")
        page_no += 1
    df = pd.concat(df_temp).reset_index(drop=True)
    df["종목코드"] = item_code
    df["종목명"] = item_name
    cols = ['종목코드', '종목명', '날짜', '종가', '전일비', '시가', '고가', '저가', '거래량']
    df = df[cols]
    date = df.iloc[0]["날짜"]
    file_name = f"{item_name}_{item_code}_{date}.csv"
    df.to_csv(file_name, index=False)
    return df
```


```python
item_code = "300080"
item_name = "플리토"

get_item_list(item_code, item_name)
```

    ***********************************************************************




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>종목코드</th>
      <th>종목명</th>
      <th>날짜</th>
      <th>종가</th>
      <th>전일비</th>
      <th>시가</th>
      <th>고가</th>
      <th>저가</th>
      <th>거래량</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>300080</td>
      <td>플리토</td>
      <td>2022.05.18</td>
      <td>39050.0</td>
      <td>500.0</td>
      <td>39600.0</td>
      <td>40050.0</td>
      <td>38200.0</td>
      <td>13039.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>300080</td>
      <td>플리토</td>
      <td>2022.05.17</td>
      <td>39550.0</td>
      <td>950.0</td>
      <td>38200.0</td>
      <td>39950.0</td>
      <td>37700.0</td>
      <td>21353.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>300080</td>
      <td>플리토</td>
      <td>2022.05.16</td>
      <td>38600.0</td>
      <td>150.0</td>
      <td>38750.0</td>
      <td>40700.0</td>
      <td>37850.0</td>
      <td>16525.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>300080</td>
      <td>플리토</td>
      <td>2022.05.13</td>
      <td>38750.0</td>
      <td>1000.0</td>
      <td>38000.0</td>
      <td>39500.0</td>
      <td>37150.0</td>
      <td>26688.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>300080</td>
      <td>플리토</td>
      <td>2022.05.12</td>
      <td>37750.0</td>
      <td>100.0</td>
      <td>37300.0</td>
      <td>38750.0</td>
      <td>37200.0</td>
      <td>18440.0</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>696</th>
      <td>300080</td>
      <td>플리토</td>
      <td>2019.07.23</td>
      <td>38550.0</td>
      <td>3200.0</td>
      <td>37400.0</td>
      <td>41100.0</td>
      <td>35700.0</td>
      <td>2480123.0</td>
    </tr>
    <tr>
      <th>697</th>
      <td>300080</td>
      <td>플리토</td>
      <td>2019.07.22</td>
      <td>35350.0</td>
      <td>2150.0</td>
      <td>35800.0</td>
      <td>38700.0</td>
      <td>32300.0</td>
      <td>3659530.0</td>
    </tr>
    <tr>
      <th>698</th>
      <td>300080</td>
      <td>플리토</td>
      <td>2019.07.19</td>
      <td>33200.0</td>
      <td>7650.0</td>
      <td>25900.0</td>
      <td>33200.0</td>
      <td>25750.0</td>
      <td>1918236.0</td>
    </tr>
    <tr>
      <th>699</th>
      <td>300080</td>
      <td>플리토</td>
      <td>2019.07.18</td>
      <td>25550.0</td>
      <td>2250.0</td>
      <td>27750.0</td>
      <td>28050.0</td>
      <td>25400.0</td>
      <td>391065.0</td>
    </tr>
    <tr>
      <th>700</th>
      <td>300080</td>
      <td>플리토</td>
      <td>2019.07.17</td>
      <td>27800.0</td>
      <td>3800.0</td>
      <td>31600.0</td>
      <td>33400.0</td>
      <td>26550.0</td>
      <td>3455088.0</td>
    </tr>
  </tbody>
</table>
<p>701 rows × 9 columns</p>
</div>


