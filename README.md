# Mini_Project_Web_Crawling_Naver_Stock
# 요약
- 사용한 언어 : Python
- 사용한 기술 : urllib, requests, bs4, pandas
- 문제 : 금 시세, 원달러 환율, 주가, 뉴스 간의 상관 관계를 분석하기 위해 데이터 크롤링하기
- 해결 : requests, bs4 를 이용해 데이터를 불러오고 pandas로 정리하는 작업 수행
- 성과  
     1. 종목 코드를 입력하면 일별 종가를 불러올 수 있는 알고리즘 생성
     2. 원자재 코드를 입력하면 일별 종가를 불러올 수 있는 알고리즘 생성
     3. 원하는 환율을 입력하면 일별 환율을 불러올 수 있는 알고리즘 생성
     4. 원하는 종목의 뉴스별 최다 사용 단어 5개를 불러오는 알고리즘 생성

# 코드

## 원자재 

```python
def crawl_gold_index(index_cd, end_page):
    date_list = []
    price_list = []
    ratio_list = []
    for page_n in range(1, end_page+1):
        naver_index = f"https://finance.naver.com/marketindex/worldDailyQuote.nhn?marketindexCd={index_cd}&fdtc=2&page={page_n}"
        src = urlopen(naver_index).read()
        source = bs4.BeautifulSoup(src, 'lxml')
        td = source.find_all('td')
        dates = source.find_all('td', class_='date')
        prices = source.find_all('td', class_='num')
        for i in range(len(dates)):
            this_date = dates[i].text.replace('\m', '').replace('\t', '').strip()
            this_date =date_format(this_date)
            this_close = prices[i*3].text.replace('\n','').replace('\t','').replace(',','')
            this_close = float(this_close)
            this_ratio = prices[i*3+2].text.replace('\n','').replace('\t','').replace('%','')
            this_ratio = float(this_ratio)
            date_list.append(this_date)
            price_list.append(this_close)
            ratio_list.append(this_ratio)
        
    df = pd.DataFrame({'날짜' : date_list, "체결가" : price_list, "등락률" : ratio_list})
    return df
    
crawl_gold_index("CMDT_GC", 20)
```       
 ![image](https://user-images.githubusercontent.com/69458840/141086847-1e1a8423-5b1e-4fb2-a547-9e32f3bf5a3a.png)
 
 
 ## 환율

```python
def crawl_dollar_index(index_cd, end_page):
    date_list = []
    price_list = []
    ratio_list = []
    for page_n in range(1, end_page+1):
        naver_index = f"https://finance.naver.com/marketindex/worldDailyQuote.nhn?fdtc=4&marketindexCd={index_cd}&page={page_n}"
        src = urlopen(naver_index).read()
        source = bs4.BeautifulSoup(src, 'lxml')
        td = source.find_all('td')
        dates = source.find_all('td', class_='date')
        prices = source.find_all('td', class_='num')
        for i in range(len(dates)):
            this_date = dates[i].text.replace('\n', '').replace('\t', '').strip()
            this_date =date_format(this_date)
            this_close = prices[i*3].text.replace('\n','').replace('\t','').replace(',','')
            this_close = float(this_close)
            this_ratio = prices[i*3+2].text.replace('\n','').replace('\t','').replace('%','')
            this_ratio = float(this_ratio)
            date_list.append(this_date)
            price_list.append(this_close)
            ratio_list.append(this_ratio)
    df = pd.DataFrame({'날짜' : date_list, "체결가" : price_list, "등락률" : ratio_list})
    return df
crawl_dollar_index('FX_EURUSD', 100)
```
![image](https://user-images.githubusercontent.com/69458840/141087470-abb98c47-de66-42cf-91c8-d4ed0ef772ff.png)


## 주식
```python
def crawl_stock_price(stock_cd, end_page):
    time_list = []
    price_list = []
    for page_n in range(1, end_page+1):
        
        url_stock = f"https://finance.naver.com/item/sise_day.nhn?code={stock_cd}&page={page_n}"
        headers = {
        'authority': 'finance.naver.com',
        'user-agent' : 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/92.0.4515.107 Safari/537.36'
        }
        res = requests.get(url_stock, headers=headers)
        source = res.text
        src = bs4.BeautifulSoup(source, 'lxml')
        dates = src.find_all('td', align='center')
        prices = src.find_all('td', class_='num')
        for i in range(len(dates)):
            this_time = dates[i].text
            this_time = date_format(this_time)
            this_price = prices[i*6].text.replace(',','')
            this_price = float(this_price)
            time_list.append(this_time)
            price_list.append(this_price)
    df = pd.DataFrame({"날짜" : time_list, "종가" : price_list})
    return df

crawl_stock_price("263750",95)
```

## 뉴스 크롤링

```python
def crawl_news_data(stock_cd, end_page):
    date_list = []
    title_list = []
    info_list = []
    word_list1 = []
    word_list2 = []
    word_list3 = []
    word_list4 = []
    word_list5 = []
    for i in range(1, end_page+1):
        page_n = i
        news_url = f"https://finance.naver.com/item/news_news.nhn?code={stock_cd}&page={page_n}&sm=title_entity_id.basic&clusterId="
        source = urlopen(news_url).read()
        src = bs4.BeautifulSoup(source, 'lxml')
        dates = src.find_all('td', class_='date')
        titles = src.find_all('td', class_='title')
        infos = src.find_all('td', class_='info')
        for i in range(len(dates)):
            news_date = dates[i].text.strip().split()[0]
            news_date = date_format(news_date)
            news_title = titles[i].text.replace('\n','')
            news_info = infos[i].text
            news_link = "https://finance.naver.com" + src.find_all('td', class_='title')[0].find_all('a', class_='tit')[0]["href"]
            news_source = urlopen(news_link).read()
            news_src = bs4.BeautifulSoup(news_source)
            news_art = news_src.find('div', id='news_read').text.split()
            word_dict = dict()
            for word in news_art:
                word_dict[word] = word_dict.get(word, 0) + 1
            a = sorted(word_dict, key= lambda x : word_dict[x], reverse=True)
            news_word1 = a[0]
            news_word2 = a[1]
            news_word3 = a[2]
            news_word4 = a[3]
            news_word5 = a[4]
#             news_word1 = most_frequent(news_art)
#             news_art.remove(news_word1)
#             news_word2 = most_frequent(news_art)
#             news_art.remove(news_word2)
#             news_word3 = most_frequent(news_art)
            date_list.append(news_date)
            title_list.append(news_title)
            info_list.append(news_info)
            word_list1.append(news_word1)
            word_list2.append(news_word2)
            word_list3.append(news_word3)
            word_list4.append(news_word4)
            word_list5.append(news_word5)
    df = pd.DataFrame({"날짜" : date_list, "뉴스 제목" : title_list, "제공" : info_list, '키워드1' : word_list1, '키워드2' : word_list2, '키워드3' : word_list3,
                      "키워드4" : word_list4, "키워드5" : word_list5})
    return df
crawl_news_data(263750, 10)
```
![image](https://user-images.githubusercontent.com/69458840/141091031-5866d345-12dd-42ba-b9c4-b9b4aa25e80d.png)
