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
'''
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
'''
