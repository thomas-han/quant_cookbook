
# 금융 데이터 수집하기 (심화)

지난 장에서는 수집한 주식티커를 바탕으로 이번 장에서는 퀀트 투자의 핵심 자료인 수정주가, 재무제표 및 가치지표를 크롤링 하는법에 대해 알아보도록 하겠습니다.

## 수정주가 크롤링

주가 데이터는 투자를 함에 있어 반드시 필요한 데이터이며, 인터넷에서 주가를 수집할 수 있는 방법은 매우 많습니다. 먼저, API를 이용한 데이터 수집에서 살펴본 것과 같이, `getSymbols()` 함수를 이용하여 데이터를 받을 수 있습니다. 그러나 야후 파이낸스에서 제공하는 데이터의 경우 미국 주가는 이상없이 다운로드 되지만, 국내 중소형주의 경우 주가가 없는 경우가 있습니다.

또한 단순 주가를 구할수 있는 방법은 많지만, 투자에 필요한 수정주가를 구할 수 있는 방법은 찾기 힘듭니다. 다행히 네이버 금융에서 제공하는 정보를 통해 모든 종목의 수정주가를 매우 손쉽게 구할 수 있습니다. 

### 개별 종목 주가 크롤링

먼저 네이버 금융에서 특정종목(예: 삼성전자)의 차트 탭^[https://finance.naver.com/item/fchart.nhn?code=005930]을 선택합니다.^[플래쉬가 차단되어 화면이 나오지 않는 경우, 주소창의 왼쪽 상단에 위치한 자물쇠 버튼을 클릭한 다음, Flash를 허용으로 바꾼 후 새로고침을 누르면 차트가 나오게 됩니다.] 해당 차트는 주가 데이터를 받아 그래프를 그려주는 형태입니다. 따라서 해당 데이터가 어디에서 오는지 알기 위해 개발자도구 화면을 이용하도록 합니다. 

\begin{figure}[h]

{\centering \includegraphics[width=0.7\linewidth]{images/crawl_practice_price2} 

}

\caption{네이버금융 차트의 통신기록}(\#fig:unnamed-chunk-2)
\end{figure}

화면을 연 상태에서 일봉 탭을 선택하면 **sise.nhn**, **schedule.nhn**, **notice.nhn** 총 3가지 항목이 생성됩니다. 이 중 sise.nhn 항목의 Request URL이 주가 데이터를 요청하는 주소입니다. 해당 url에 접속해 보도록 하겠습니다.

\begin{figure}[h]

{\centering \includegraphics[width=0.7\linewidth]{images/crawl_practice_price3} 

}

\caption{주가 데이터 페이지}(\#fig:unnamed-chunk-3)
\end{figure}

각 날짜별로 시가, 고가, 저가, 종가, 거래량이 있으며, 주가의 경우 모두 수정주가 기준입니다. 또한 해당 데이터가 item 태그 내 data 속성에 위치하고 있습니다.

url에서 symbol= 뒤에 위치하는 6자리 티커만 변경하면 해당 종목의 주가 데이터가 위치한 페이지로 이동할 수 있으며, 우리가 원하는 모든 종목들의 주가 데이터를 크롤링 할 수 있습니다. 

```r
library(stringr)

KOR_ticker = read.csv('data/KOR_ticker.csv', row.names = 1)
print(KOR_ticker$'종목코드'[1])
```

```
## [1] 5930
```

```r
KOR_ticker$'종목코드' =
  str_pad(KOR_ticker$'종목코드', 6, side = c('left'), pad = '0')
```

먼저 저장해두었던 csv 파일을 불러오도록 합니다. 종목코드를 살펴보면 005930 이어야 할 삼성전자의 티커가 5930으로 입력되어 있습니다. 이는 파일을 불러오는 과정에서 0으로 시작하는 숫자들이 지워졌기 때문입니다. `stringr` 패키지의 `str_pad()` 함수를 사용해 6자리가 되지 않는 문자는 왼쪽에 0을 추가하여 강제로 6자리로 만들어 주도록 합니다.

다음은 첫번째 종목인 삼성전자의 주가를 크롤링한 후 가공하는 방법입니다.


```r
library(xts)

ifelse(dir.exists('data/KOR_price'), FALSE,
       dir.create('data/KOR_price'))
```

```
## [1] FALSE
```

```r
i = 1
name = KOR_ticker$'종목코드'[i]

price = xts(NA, order.by = Sys.Date())
print(price)
```

```
##            [,1]
## 2019-07-07   NA
```

1. 먼저 data 폴더 내에 KOR_price 폴더를 생성해줍니다.
2. i = 1 을 입력해 줍니다. 향후 for loop 구문을 통해 i 값만 변경하면 모든 종목의 주가를 다운로드 받을 수 있습니다.
3. name에 해당 티커를 입력해줍니다.
4. `xts()` 함수를 이용해 빈 시계열 데이터를 생성해주며, 인덱스는 `Sys.Date()`를 통해 현재 날짜를 입력합니다.


```r
library(httr)
library(rvest)

url = paste0(
  'https://fchart.stock.naver.com/sise.nhn?symbol=',
  name,'&timeframe=day&count=500&requestType=0')
data = GET(url)
data_html = read_html(data, encoding = 'EUC-KR') %>%
  html_nodes('item') %>%
  html_attr('data') 

print(head(data_html))
```

```
## [1] "20170620|47240|48140|47220|48140|300900"
## [2] "20170621|47740|48120|47480|47480|199473"
## [3] "20170622|47960|48080|47720|47960|229116"
## [4] "20170623|47600|47780|47420|47620|190302"
## [5] "20170626|47520|48360|47520|48280|171056"
## [6] "20170627|48220|48400|47900|48300|192335"
```

1. `paste0()` 함수를 이용해 원하는 종목의 url을 생성해 줍니다. url 중 티커에 해당하는 6자리 부분만 위에서 입력한 name으로 설정해주면 됩니다.
2. `GET()` 함수를 통해 페이지의 데이터를 불러옵니다.
3. `read_html()` 함수를 통해 html 정보를 읽어옵니다.
4. `html_nodes()`와 `html_attr()` 함수를 통해 item 태그 및 data 속성의 데이터를 추출합니다.

결과적으로 날짜 및 주가, 거래량 데이터가 추출됩니다. 해당 데이터는 **|**으로 구분되어 있으며, 이를 테이블 형태로 바꿀 필요가 있습니다.


```r
library(readr)

price = read_delim(data_html, delim = '|')
print(head(price))
```

```
## # A tibble: 6 x 6
##   `20170620` `47240` `48140` `47220` `48140_1` `300900`
##        <dbl>   <dbl>   <dbl>   <dbl>     <dbl>    <dbl>
## 1   20170621   47740   48120   47480     47480   199473
## 2   20170622   47960   48080   47720     47960   229116
## 3   20170623   47600   47780   47420     47620   190302
## 4   20170626   47520   48360   47520     48280   171056
## 5   20170627   48220   48400   47900     48300   192335
## 6   20170628   47600   48000   47560     47700   191450
```

`readr` 패키지의 `read_delim()` 함수를 쓸 경우 구분자로 이루어진 데이터를 테이블로 쉽게 변경할 수 있습니다. 데이터를 확인해보면 테이블 형태로 변경되었으며 각 열은 날짜, 시가, 고가, 저가, 종가, 거래량을 의미합니다. 이 중 우리가 필요한 날짜와 종가를 선택한 후, 데이터 클랜징을 해주도록 합니다.


```r
library(lubridate)
library(timetk)

price = price[c(1, 5)] 
price = data.frame(price)
colnames(price) = c('Date', 'Price')
price[, 1] = ymd(price[, 1])
price = tk_xts(price, date_var = Date)
 
print(tail(price))
```

```
##            Price
## 2019-06-28 47000
## 2019-07-01 46600
## 2019-07-02 46250
## 2019-07-03 45400
## 2019-07-04 46000
## 2019-07-05 45650
```

1. 날짜에 해당하는 첫번째 열과, 종가에 해당하는 다섯번째 열만을 선택해 저장해 줍니다.
2. 티블 형태의 데이터를 데이터프레임 형태로 변경해 줍니다.
3. 열이름을 Date와 Price로 변경해 줍니다.
4. `lubridate` 패키지의 `ymd()` 함수를 이용하면 yyyymmdd 형태를 yyyy-mm-dd로 변경해주며, 데이터 형태 또한 Date 타입으로 변경됩니다.
5. `timetk` 패키지의 `tk_xts()` 함수를 이용해 시계열 형태로 변경해주며, 인덱스는 Date 열을 설정해줍니다. 형태 변경 후 해당 열은 자동으로 삭제됩니다.

데이터를 확인해보면 우리에게 필요한 형태로 정리가 되었습니다. 


```r
write.csv(price, paste0('data/KOR_price/', name,
                        '_price.csv'))
```

마지막으로 해당 데이터를 data 폴더의 KOR_price 폴더 내에 **티커_price.csv** 이름으로 저장해주도록 합니다. 

### 전 종목 주가 크롤링

위의 코드에서 for loop 구문을 이용하여 i 값만 변경해주면 모든 종목의 주가를 다운로드 받을 수 있습니다. 전 종목 주가를 다운로드 받는 전체 코드는 다음과 같습니다.


```r
library(httr)
library(rvest)
library(stringr)
library(xts)
library(lubridate)
library(readr)

KOR_ticker = read.csv('data/KOR_ticker.csv', row.names = 1)
print(KOR_ticker$'종목코드'[1])
KOR_ticker$'종목코드' =
  str_pad(KOR_ticker$'종목코드', 6, side = c('left'), pad = '0')

ifelse(dir.exists('data/KOR_price'), FALSE,
       dir.create('data/KOR_price'))

for(i in 1 : nrow(KOR_ticker) ) {
  
  price = xts(NA, order.by = Sys.Date()) # 빈 시계열 데이터 생성
  name = KOR_ticker$'종목코드'[i] # 티커 부분 선택
  
  # 오류 발생 시 이를 무시하고 다음 루프로 진행
  tryCatch({
    # url 생성
    url = paste0(
      'https://fchart.stock.naver.com/sise.nhn?symbol='
      ,name,'&timeframe=day&count=500&requestType=0')
    
    # 이 후 과정은 위와 동일함
    # 데이터 다운로드
    data = GET(url)
    data_html = read_html(data, encoding = 'EUC-KR') %>%
      html_nodes("item") %>%
      html_attr("data") 
    
    # 데이터 나누기
    price = read_delim(data_html, delim = '|')
    
    # 필요한 열만 선택 후 클렌징
    price = price[c(1, 5)] 
    price = data.frame(price)
    colnames(price) = c('Date', 'Price')
    price[, 1] = ymd(price[, 1])
    
    rownames(price) = price[, 1]
    price[, 1] = NULL
    
  }, error = function(e) {
    
    # 오류 발생시 해당 종목명을 출력하고 다음 루프로 이동
    warning(paste0("Error in Ticker: ", name))
  })
  
  # 다운로드 받은 파일을 생성한 폴더 내 csv 파일로 저장
  write.csv(price, paste0('data/KOR_price/', name,
                          '_price.csv'))
  
  # 타임슬립 적용
  Sys.sleep(2)
}
```

위의 코드에서 추가된 점은 다음과 같습니다. 페이지 오류, 통신 오류 등 오류가 발생할 경우 for loop 구문은 멈추어 버리며, 전체 데이터를 처음부터 다시 받는 일은 매우 귀찮은 작업입니다. 따라서 `tryCatch()` 함수를 이용해 [오류가 발생 시 해당 티커를 출력한 후 다음 loop로 넘어가게 합니다.][오류에 대한 예외처리]

또한 오류가 발생했을 시에는 `xts()` 함수를 통해 만들어 둔 빈 데이터를 저장하게 됩니다. 마지막으로 무한크롤링을 방지하기 위해, 한번의 루프가 끝날때 마다 2초의 타임슬립을 적용하였습니다.

위의 코드가 모두 돌아가는데는 수시간이 걸리며, 작업이 끝난 후 data/KOR_price 폴더를 확인해보면 전 종목 주가가 csv 형태로 저장되어 있습니다.

## 재무제표 및 가치지표 크롤링

주가와 더불어 재무제표와 가치지표 역시 투자에 있어 핵심이 되는 데이터입니다. 해당 데이터 역시 구할 수 잇는 여러 사이트가 있지만, 국내의 데이터 제공업체인 FnGuide에서 운영하는 Company Guide 홈페이지^[http://comp.fnguide.com/]에서 손쉽게 구할 수 있습니다. 

### 재무제표 다운로드

먼저 개별종목의 재무제표를 탭을 선택하면 **포괄손익계산서, 재무상태표, 현금흐름표** 항목이 보이게 되며, 티커에 해당하는 A005930 뒤의 주소는 불필요한 내용이므로, 이를 제거한 주소로 접속하도록 합니다. A 뒤의 6자리 티커만 변경할 경우, 해당 종목의 재무제표 페이지로 이동하게 됩니다.

**http://comp.fnguide.com/SVO2/ASP/SVD_Finance.asp?pGB=1&gicode=A005930**


우리가 원하는 재무제표 항목들은 모두 테이블 형태로 제공되고 있으므로, `html_table()` 함수를 이용하여 손쉽게 추출할 수 있습니다.


```r
library(httr)
library(rvest)

ifelse(dir.exists('data/KOR_fs'), FALSE,
       dir.create('data/KOR_fs'))

Sys.setlocale("LC_ALL", "English")

url = paste0('http://comp.fnguide.com/SVO2/ASP/',
             'SVD_Finance.asp?pGB=1&gicode=A005930')

data = GET(url)
data = data %>%
  read_html() %>%
  html_table()

Sys.setlocale("LC_ALL", "Korean")
```

```r
lapply(data, function(x) {
  head(x, 3)})
```

```
## [[1]]
##   IFRS(연결)   2016/12   2017/12   2018/12 2019/03
## 1     매출액 2,018,667 2,395,754 2,437,714 523,855
## 2   매출원가 1,202,777 1,292,907 1,323,944 327,465
## 3 매출총이익   815,890 1,102,847 1,113,770 196,391
##   전년동기 전년동기(%)
## 1  605,637       -13.5
## 2  319,095         2.6
## 3  286,542       -31.5
## 
## [[2]]
##   IFRS(연결) 2018/06 2018/09 2018/12 2019/03 전년동기
## 1     매출액 584,827 654,600 592,651 523,855  605,637
## 2   매출원가 312,746 351,944 340,160 327,465  319,095
## 3 매출총이익 272,081 302,656 252,491 196,391  286,542
##   전년동기(%)
## 1       -13.5
## 2         2.6
## 3       -31.5
## 
## [[3]]
##                          IFRS(연결)   2016/12   2017/12
## 1                              자산 2,621,743 3,017,521
## 2 유동자산계산에 참여한 계정 펼치기 1,414,297 1,469,825
## 3                          재고자산   183,535   249,834
##     2018/12   2019/03
## 1 3,393,572 3,450,679
## 2 1,746,974 1,773,885
## 3   289,847   314,560
## 
## [[4]]
##                          IFRS(연결)   2018/06   2018/09
## 1                              자산 3,186,884 3,371,958
## 2 유동자산계산에 참여한 계정 펼치기 1,569,768 1,762,820
## 3                          재고자산   273,588   282,428
##     2018/12   2019/03
## 1 3,393,572 3,450,679
## 2 1,746,974 1,773,885
## 3   289,847   314,560
## 
## [[5]]
##                     IFRS(연결) 2016/12 2017/12 2018/12
## 1     영업활동으로인한현금흐름 473,856 621,620 670,319
## 2                   당기순손익 227,261 421,867 443,449
## 3 법인세비용차감전계속사업이익                        
##   2019/03
## 1  52,443
## 2  50,436
## 3        
## 
## [[6]]
##                     IFRS(연결) 2018/06 2018/09 2018/12
## 1     영업활동으로인한현금흐름 134,378 155,497 224,281
## 2                   당기순손익 110,434 131,507  84,622
## 3 법인세비용차감전계속사업이익                        
##   2019/03
## 1  52,443
## 2  50,436
## 3
```

1. 먼저 data 폴더 내에 KOR_fs 폴더를 생성해줍니다.
2. `Sys.setlocale()` 함수를 통해 로케일 언어를 영어로 설정 해줍니다.
3. url을 입력한 후, `GET()` 함수를 통해 페이지 내용을 받아옵니다.
4. `read_html()` 함수를 통해 html 내용을 읽어오며, `html_table()` 함수를 통해 테이블 내용만을 추출합니다.
5. 로케일 언어를 다시 한글로 설정 해줍니다.

위의 과정을 거치면 data 변수에는 총 리스트 형태로 총 6개의 테이블이 들어오게 되며, 그 내용은 표 \@ref(tab:fstable)와 같습니다.

\begin{table}[!h]

\caption{(\#tab:fstable)재무제표 테이블 내역}
\centering
\begin{tabular}{c>{\centering\arraybackslash}p{5cm}}
\toprule
순서 & 내용\\
\midrule
\rowcolor{gray!6}  \rowcolor{black}  \textcolor{white}{\textbf{1}} & \textcolor{white}{\textbf{포괄손익계산서 (연간)}}\\
2 & 포괄손익계산서 (분기)\\
\rowcolor{gray!6}  \rowcolor{black}  \textcolor{white}{\textbf{3}} & \textcolor{white}{\textbf{재무상태표 (연간)}}\\
4 & 재무상태표 (분기)\\
\rowcolor{gray!6}  \rowcolor{black}  \textcolor{white}{\textbf{5}} & \textcolor{white}{\textbf{현금흐름표 (연간)}}\\
6 & 현금흐름표 (분기)\\
\bottomrule
\end{tabular}
\end{table}

이 중 연간 기준 재무제표에 해당하는 첫번째, 세번째, 다섯번째 테이블을 선택합니다.


```r
data_IS = data[[1]]
data_BS = data[[3]]
data_CF = data[[5]]

print(names(data_IS))
```

```
## [1] "IFRS(연결)"  "2016/12"     "2017/12"    
## [4] "2018/12"     "2019/03"     "전년동기"   
## [7] "전년동기(%)"
```

```r
data_IS = data_IS[, 1:(ncol(data_IS)-2)]
```

포괄손익계산서 테이블(data_IS)에는 전년동기, 전년동기(%) 열이 존재하며, 통일성을 위해 이를 삭제해줍니다. 이제 테이블을 묶은 후 클랜징을 해주도록 하겠습니다.


```r
data_fs = rbind(data_IS, data_BS, data_CF)
data_fs[, 1] = gsub('계산에 참여한 계정 펼치기',
                    '', data_fs[, 1])
data_fs = data_fs[!duplicated(data_fs[, 1]), ]

rownames(data_fs) = NULL
rownames(data_fs) = data_fs[, 1]
data_fs[, 1] = NULL

data_fs = data_fs[, substr(colnames(data_fs), 6,7) == '12']
```

1. 먼저 `rbind()` 함수를 이용하여 세 테이블을 행으로 묶은 뒤 data_fs에 저장합니다.
2. 첫번째 열인 계정명에는 **계산에 참여한 계정 펼치기** 라는 글자가 들어간 항목이 존재합니다. 이는 페이지 내에서 펼치기 역할을 하는 (+) 항목에 해당하며, `gsub()` 함수를 이용해 해당 글자를 삭제해 줍니다.
3. 중복되는 계정명이 다수 존재하며, 이는 대부분 불필요한 항목입니다. `!duplicated()` 함수를 사용해 중복되지 않는 계정명만을 선택해 줍니다.
4. 행이름을 초기화 한 후, 첫번째 열의 계정명을 행이름으로 변경합니다. 그 후 첫번째 열은 삭제해주도록 합니다.
5. 간혹 12월 결산법인이 아닌 종목, 혹은 연간 재무제표임에도 불구하고 분기 재무제표가 들어와 있는 경우가 있습니다. 비교의 통일성을 위해 `substr()` 함수를 이용하여 끝 글자가 12 인 열, 즉 12월 결산 데이터만을 선택해 줍니다. 


```r
print(head(data_fs))
```

```
##                    2016/12   2017/12   2018/12
## 매출액           2,018,667 2,395,754 2,437,714
## 매출원가         1,202,777 1,292,907 1,323,944
## 매출총이익         815,890 1,102,847 1,113,770
## 판매비와관리비     523,484   566,397   524,903
## 인건비              59,763    67,972    64,514
## 유무형자산상각비    10,018    13,366    14,477
```

```r
sapply(data_fs, typeof)
```

```
##     2016/12     2017/12     2018/12 
## "character" "character" "character"
```

데이터를 확인해보면 연간 기준 재무제표가 정리되었으며, 문자형 데이터이므로 숫자형으로 변경해주도록 합니다.


```r
library(stringr)

data_fs = sapply(data_fs, function(x) {
  str_replace_all(x, ',', '') %>%
    as.numeric()
}) %>%
  data.frame(., row.names = rownames(data_fs))

print(head(data_fs))
```

```
##                  X2016.12 X2017.12 X2018.12
## 매출액            2018667  2395754  2437714
## 매출원가          1202777  1292907  1323944
## 매출총이익         815890  1102847  1113770
## 판매비와관리비     523484   566397   524903
## 인건비              59763    67972    64514
## 유무형자산상각비    10018    13366    14477
```

```r
sapply(data_fs, typeof)
```

```
## X2016.12 X2017.12 X2018.12 
## "double" "double" "double"
```

1. `sapply()` 함수를 이용해 각 열에 `stringr` 패키지의 `str_replace_allr()` 함수를 적용하여 콤마(,)를 제거한 후, `as.numeric()` 함수를 통해 숫자형 데이터로 변경합니다.
2. `data.frame()` 함수를 이용해 데이터프레임 형태로 만들어주며, 행이름은 기존 내용을 그대로 유지해줍니다.

정리된 데이터를 출력해보면 문자형이던 데이터가 숫자형으로 변경되었습니다.


```r
write.csv(data_fs, 'data/KOR_fs/005930_fs.csv')
```

data 폴더의 KOR_fs 폴더 내에 **티커_fs.csv** 이름으로 저장해주도록 합니다.

### 가치지표 계산하기

위에서 구한 재무제표 데이터를 이용해 가치지표를 계산할 수 있습니다. 흔히 사용되는 가치지표는 **PER, PBR, PCR, PSR** 이며 분자는 주가, 분모는 재무제표 데이터가 사용됩니다.

\begin{table}[!h]

\caption{(\#tab:unnamed-chunk-18)가치지표의 종류}
\centering
\begin{tabular}{cc}
\toprule
순서 & 분모\\
\midrule
\rowcolor{gray!6}  PER & Earnings (순이익)\\
PBR & Book Value (순자산)\\
\rowcolor{gray!6}  PCR & Cashflow (영업활동현금흐름)\\
PSR & Sales (매출액)\\
\bottomrule
\end{tabular}
\end{table}

위에서 구한 재무제표 항목에서 분모 부분에 해당하는 데이터만 선택하도록 하겠습니다.


```r
ifelse(dir.exists('data/KOR_value'), FALSE,
       dir.create('data/KOR_value'))
```

```
## [1] FALSE
```

```r
value_type = c('지배주주순이익',
               '자본',
               '영업활동으로인한현금흐름',
               '매출액')

value_index = data_fs[match(value_type, rownames(data_fs)),
                      ncol(data_fs)]
print(value_index)
```

```
## [1]  438909 2477532  670319 2437714
```

1. 먼저 data 폴더 내에 KOR_value 폴더를 생성해줍니다.
2. 분모에 해당하는 항목을 저장한 후, `match()` 함수를 이용하여 해당 항목이 위치하는 지점을 찾으며, `ncol()` 함수를 이용하여 가장 우측, 즉 최근년도 재무제표 데이터를 선택해줍니다.

다음으로 분자 부분에 해당하는 **현재 주가**를 수집해야 합니다. 이 역시 Company Guide 접속화면에서 구할 수 있으며, 불필요한 부분을 제거한 url은 다음과 같습니다.

**http://comp.fnguide.com/SVO2/ASP/SVD_main.asp?pGB=1&gicode=A005930**

위의 주소 역시 A 뒤의 6자리 티커만 변경할 경우, 해당 종목의 스냅샷 페이지로 이동하게 됩니다.

\begin{figure}[h]

{\centering \includegraphics[width=0.7\linewidth]{images/crawl_practice_comp_price} 

}

\caption{Company Guide 스냅샷 화면}(\#fig:unnamed-chunk-20)
\end{figure}

주가추이 부분에 우리가 원하는 현재 주가가 있으며, 해당 데이터의 Xpath는 다음과 같습니다.


```css
//*[@id="svdMainChartTxt11"]
```

위에서 구한 주가의 Xpath를 이용하여 해당 데이터를 크롤링하도록 하겠습니다.


```r
library(readr)

url =
  paste0('http://comp.fnguide.com/SVO2/ASP/SVD_main.asp',
         '?pGB=1&gicode=A005930')
data = GET(url)

price = read_html(data) %>%
  html_node(xpath = '//*[@id="svdMainChartTxt11"]') %>%
  html_text() %>%
  parse_number()

print(price)
```

```
## [1] 45650
```

1. 먼저 url을 입력한 후, `GET()` 함수를 이용하여 데이터를 불러옵니다.
2. `read_html()` 함수를 이용해 html 데이터를 불러온 후, `html_node()` 함수 내에 위에서 구한 Xpath를 입력하여, 해당 지점의 데이터를 추출합니다. 
3. `html_text()` 함수를 통해 텍스트 데이터만을 추출하며, `readr` 패키지의  `parse_number()` 함수를 적용합니다. 해당 함수는 문자형 데이터에서 콤마와 같은 불필요한 문자를 제거한 후, 숫자형 데이터로 변경해줍니다.

가치지표를 계산하기 위해서는 발행주식수 역시 필요합니다. 예를 들어 PER를 계산하는 방법은 다음과 같습니다.

$$ PER = Price / EPS  = 주가 / 주당순이익$$
  
주당순이익의 경우 순이익을 전체 주식수로 나눈값이므로, 해당 값의 계산을 위해 전체 주식수를 구해야합니다. 해당 데이터 역시 웹페이지에 존재하므로 위에서 주가를 크롤링한 방법과 동일하게 구할 수 있으며, Xpath는 다음과 같습니다.


```css
//*[@id="svdMainGrid1"]/table/tbody/tr[7]/td[1]
```

이를 이용해 발행주식수 중 보통주를 선택하는 방법은 다음과 같습니다.


```r
share = read_html(data) %>%
  html_node(
    xpath =
      '//*[@id="svdMainGrid1"]/table/tbody/tr[7]/td[1]') %>%
  html_text()

print(share)
```

```
## [1] "5,969,782,550/ 822,886,700"
```

`read_html()` 함수와 `html_node()` 함수를 이용해, html 내에서 Xpath에 해당하는 데이터를 추출합니다. 그 후 `html_text()` 함수를 통해 텍스트 부분만 추출하도록 합니다. 해당 과정을 거치면 보통주/우선주의 형태로 발행주식주가 저장되어 있습니다. 이 중 우리가 원하는 데이터는 **/** 앞에 위치한 보통주 발행주식수 입니다.


```r
share = share %>%
  strsplit('/') %>%
  unlist() %>%
  .[1] %>%
  parse_number()

print(share)
```

```
## [1] 5969782550
```

1. `strsplit()` 함수를 통해 **/**를 기준으로 데이터를 나누어주며, 해당 결과는 리스트 형태로 저장이 됩니다.
2. `unlist()` 함수를 통해 리스트를 벡터 형태로 변환합니다.
3. `.[1]`을 통해 보통주 발행주식수인 첫번째 데이터를 선택합니다.
4. `parse_number()` 함수를 통해 문자형 데이터를 숫자형으로 변환해 줍니다.

재무데이터, 현재 주가, 발행주식수를 이용하여 가치지표를 계산해보도록 하겠습니다.


```r
data_value = price / (value_index * 100000000 / share)
names(data_value) = c('PER', 'PBR', 'PCR', 'PSR')
data_value[data_value < 0] = NA

print(data_value)
```

```
##   PER   PBR   PCR   PSR 
## 6.209 1.100 4.066 1.118
```

분자에는 현재 주가를 입력하며, 분모에는 재무 데이터를 보통주 발행주식수로 나눈 값을 입력합니다. 단, 주가는 원 단위, 재무 데이터는 억 단위이므로, 둘 간의 단위를 동일하게 맞춰주기 위해 분모에 억을 곱해 줍니다. 또한 가치지표가 음수인 경우는 `NA`로 변경해주도록 합니다.

결과를 확인해보면 4가지 가치지표가 잘 계산되었습니다.^[분모에 사용되는 재무데이터의 구체적인 항목과 발행주식수를 계산하는 방법의 차이로 인해 여러 업체에서 제공하는 가치지표와 다소 차이가 발생할 수 있습니다.]


```r
write.csv(data_value, 'data/KOR_value/005930_value.csv')
```

data 폴더의 KOR_value 폴더 내에 **티커_value.csv** 이름으로 저장해주도록 합니다.

### 전 종목 재무제표 및 가치지표 다운로드

위의 코드에서 for loop 구문을 이용하여 url 중 6자리 티커에 해당하는 값만 변경해주면 모든 종목의 재무제표를 다운로드 받고, 이를 바탕으로 가치지표를 계산할 수 있습니다. 해당 코드는 다음과 같습니다.


```r
library(stringr)
library(httr)
library(rvest)
library(stringr)
library(readr)

KOR_ticker = read.csv('data/KOR_ticker.csv', row.names = 1)
KOR_ticker$'종목코드' =
  str_pad(KOR_ticker$'종목코드', 6,side = c('left'), pad = '0')

ifelse(dir.exists('data/KOR_fs'), FALSE,
       dir.create('data/KOR_fs'))
ifelse(dir.exists('data/KOR_value'), FALSE,
       dir.create('data/KOR_value'))

for(i in 1 : nrow(KOR_ticker) ) {
  
  data_fs = c()
  data_value = c()
  name = KOR_ticker$'종목코드'[i]
  
  # 오류 발생 시 이를 무시하고 다음 루프로 진행
  tryCatch({
    
    Sys.setlocale('LC_ALL', 'English')
    
    # url 생성
    url = paste0(
      'http://comp.fnguide.com/SVO2/ASP/'
      ,'SVD_Finance.asp?pGB=1&gicode=A',
      name)
    
    # 이 후 과정은 위와 동일함
    
    # 데이터 다운로드 후 테이블 추출
    data = GET(url) %>%
      read_html() %>%
      html_table()
    
    Sys.setlocale('LC_ALL', 'Korean')
    
    # 3개 재무제표를 하나로 합치기
    data_IS = data[[1]]
    data_BS = data[[3]]
    data_CF = data[[5]]
    
    data_IS = data_IS[, 1:(ncol(data_IS)-2)]
    data_fs = rbind(data_IS, data_BS, data_CF)
    
    # 데이터 클랜징
    data_fs[, 1] = gsub('계산에 참여한 계정 펼치기',
                        '', data_fs[, 1])
    data_fs = data_fs[!duplicated(data_fs[, 1]), ]
    
    rownames(data_fs) = NULL
    rownames(data_fs) = data_fs[, 1]
    data_fs[, 1] = NULL
    
    # 12월 재무제표만 선택
    data_fs =
      data_fs[, substr(colnames(data_fs), 6,7) == "12"]
    
    data_fs = sapply(data_fs, function(x) {
      str_replace_all(x, ',', '') %>%
        as.numeric()
    }) %>%
      data.frame(., row.names = rownames(data_fs))
    
    
    # 가치지표 분모부분
    value_type = c('지배주주순이익', 
                   '자본', 
                   '영업활동으로인한현금흐름', 
                   '매출액') 
    
    # 해당 재무데이터만 선택
    value_index = data_fs[match(value_type, rownames(data_fs)),
                          ncol(data_fs)]
    
    # Snapshot 페이지 불러오기
    url =
      paste0(
        'http://comp.fnguide.com/SVO2/ASP/SVD_Main.asp',
        '?pGB=1&gicode=A',name)
    data = GET(url)
    
    # 현재 주가 크롤링
    price = read_html(data) %>%
      html_node(xpath = '//*[@id="svdMainChartTxt11"]') %>%
      html_text() %>%
      parse_number()
    
    # 보통주 발행장주식수 크롤링
    share = read_html(data) %>%
      html_node(
        xpath =
        '//*[@id="svdMainGrid1"]/table/tbody/tr[7]/td[1]') %>%
      html_text() %>%
      strsplit('/') %>%
      unlist() %>%
      .[1] %>%
      parse_number()
    
    # 가치지표 계산
    data_value = price / (value_index * 100000000/ share)
    names(data_value) = c('PER', 'PBR', 'PCR', 'PSR')
    data_value[data_value < 0] = NA
    
  }, error = function(e) {
    
    # 오류 발생시 해당 종목명을 출력하고 다음 루프로 이동
    data_fs <<- NA
    data_value <<- NA
    warning(paste0("Error in Ticker: ", name))
  })
  
  # 다운로드 받은 파일을 생성한 각각의 폴더 내 csv 파일로 저장
  
  # 재무제표 저장
  write.csv(data_fs, paste0('data/KOR_fs/', name, '_fs.csv'))
  
  # 가치지표 저장
  write.csv(data_value, paste0('data/KOR_value/', name,
                               '_value.csv'))
  
  # 2초간 타임슬립 적용
  Sys.sleep(2)
}
```

전종목 주가 데이터를 받는 과정과 동일하게, **KOR_ticker.csv** 파일을 불러온 후 for loop을 통해 i값이 변함에 따라 티커를 변경해가며 모든 종목의 재무제표 및 가치지표를 다운로드 받습니다. `tryCatch()` 함수를 이용해 오류가 발생 시 `NA`로 이루어진 빈 데이터를 저장한 후, 다음 루프로 넘어가게 됩니다.

data/KOR_fs 폴더에는 전 종목의 재무제표 데이터가, data/KOR_value 폴더에는 전 종목의 가치지표 데이터가 csv 형태로 저장되어 있게 됩니다.

## 야후 파이낸스 데이터 구하기

크롤링을 이용하여 데이터를 수집할 경우 주의해야 할 점은 웹페이지 구조가 변경되는 경우이며, 이런 경우에는 변경된 구조에 맞게 코드를 다시 짜야합니다. 그러나 최악의 경우는 웹사이트가 폐쇄되는 경우입니다. 실제로 투자자들이 많이 사용하던 구글 파이낸스가 2018년 서비스를 중단함에 따라 이를 이용하던 많은 사용자들이 혼란을 겪기도 했습니다.

이러한 상황에 대비하여 데이터를 구할수 있는 예비 사이트를 알아두어 테스트 코드를 작성해 둘 필요가 있으며, 이를 위해 야후 파이낸스에서 데이터 구하는 법을 살펴보도록 하겠습니다. 주가 데이터의 경우 `getSymbols()` 함수를 통해 주가를 다운로드 받는 방법을 이미 살펴보았으며, 웹페이지에서 재무제표를 크롤링하는법 및 가치지표를 계산하는 법에 대해 알아보도록 하겠습니다.

### 재무제표 다운로드

\begin{figure}[h]

{\centering \includegraphics[width=0.7\linewidth]{images/crawl_practice_yahoo} 

}

\caption{야후 파이낸스 재무제표}(\#fig:unnamed-chunk-29)
\end{figure}

먼저 야후 파이낸스에 접속하여 삼성전자 티커에 해당하는 `005930.KS`를 입력합니다. 그 후, 재무제표 데이터에 해당하는 Financials 항목을 선택합니다. 손익계산서(Income Statement), 재무상태표(Balance Sheet), 현금흐름표(Cash Flow) 총 3개 지표가 있으며, 각각의 url은 표 \@ref(tab:yahoofs)와 같습니다.

\begin{table}[!h]

\caption{(\#tab:yahoofs)야후 파이낸스 재무제표 url}
\centering
\fontsize{7}{9}\selectfont
\begin{tabular}{cc}
\toprule
항목 & url\\
\midrule
\rowcolor{gray!6}  Income Statement & https://finance.yahoo.com/quote/005930.KS/financials?p=005930.KS\\
Balance Sheet & https://finance.yahoo.com/quote/005930.KS/balance-sheet?p=005930.KS\\
\rowcolor{gray!6}  Cash Flow & https://finance.yahoo.com/quote/005930.KS/cash-flow?p=005930.KS\\
\bottomrule
\end{tabular}
\end{table}

각 페이지에서 Xpath를 이용하여 재무제표에 해당하는 테이블 부분만을 선택하여 추출할 수 있으며, 3개 페이지의 해당 Xpath는 모두 아래와 같이 동일합니다.


```css
//*[@id="Col1-1-Financials-Proxy"]/section/div[3]/table
```

위의 정보를 이용하여 재무제표를 다운로드 받는 과정은 다음과 같습니다.


```r
library(httr)
library(rvest)

url_IS = paste0(
  'https://finance.yahoo.com/quote/005930.KS/financials?p=',
  '005930.KS')

url_BS = paste0(
  'https://finance.yahoo.com/quote/005930.KS/balance-sheet?p=',
  '005930.KS')

url_CF = paste0(
  'https://finance.yahoo.com/quote/005930.KS/cash-flow?p=',
  '005930.KS')

yahoo_finance_xpath =
  '//*[@id="Col1-1-Financials-Proxy"]/section/div[3]/table'

data_IS = GET(url_IS) %>%
  read_html() %>%
  html_node(xpath = yahoo_finance_xpath) %>%
  html_table()

data_BS = GET(url_BS) %>%
  read_html() %>%
  html_node(xpath = yahoo_finance_xpath) %>%
  html_table()

data_CF = GET(url_CF) %>%
  read_html() %>%
  html_node(xpath = yahoo_finance_xpath) %>%
  html_table()

data_fs = rbind(data_IS, data_BS, data_CF)

print(head(data_fs))
```

```
##                     X1                 X2
## 1              Revenue         12/31/2018
## 2        Total Revenue    243,771,415,000
## 3      Cost of Revenue    132,394,411,000
## 4         Gross Profit    111,377,004,000
## 5   Operating Expenses Operating Expenses
## 6 Research Development     18,354,080,000
##                   X3                 X4
## 1         12/31/2017         12/31/2016
## 2    239,575,376,000    201,866,745,000
## 3    129,290,661,000    120,277,715,000
## 4    110,284,715,000     81,589,030,000
## 5 Operating Expenses Operating Expenses
## 6     16,355,612,000     14,111,381,000
##                   X5
## 1         12/31/2015
## 2    200,653,482,000
## 3    123,482,118,000
## 4     77,171,364,000
## 5 Operating Expenses
## 6     13,705,695,000
```

1. 위에서 구한 url을 저장해줍니다.
2. `GET()` 함수를 통해 페이지 정보를 받아온 후, `read_html()` 함수를 통해 html 정보를 받아옵니다.
3. `html_node()` 함수 내에서 위에서 구한 Xpath를 이용해 테이블 부분의 html을 선택한 후, `html_table()`을 통해 테이블 형태만 추출합니다.
4. 3개 페이지에 위의 내용을 동일하게 적용한 후, `rbind()`를 이용해 행으로 묶어줍니다.

다운로드 받은 데이터를 클랜징 작업을 해주도록 하며, 그 과정은 앞선 예시와 거의 동일합니다.


```r
library(stringr)

data_fs = data_fs[!duplicated(data_fs[, 1]), ]
rownames(data_fs) = NULL
rownames(data_fs) = data_fs[, 1]

colnames(data_fs) = data_fs[1, ]
data_fs = data_fs[-1, ]

data_fs = data_fs[, substr(colnames(data_fs), 1,2) == "12"]

data_fs = sapply(data_fs, function(x) {
  str_replace_all(x, ',', '') %>%
    as.numeric()
  }) %>%
  data.frame(., row.names = rownames(data_fs))

print(head(data_fs))
```

```
##                                     X12.31.2018
## Total Revenue                      243771415000
## Cost of Revenue                    132394411000
## Gross Profit                       111377004000
## Operating Expenses                           NA
## Research Development                18354080000
## Selling General and Administrative  32688565000
##                                     X12.31.2017
## Total Revenue                      239575376000
## Cost of Revenue                    129290661000
## Gross Profit                       110284715000
## Operating Expenses                           NA
## Research Development                16355612000
## Selling General and Administrative  38947445000
##                                     X12.31.2016
## Total Revenue                      201866745000
## Cost of Revenue                    120277715000
## Gross Profit                        81589030000
## Operating Expenses                           NA
## Research Development                14111381000
## Selling General and Administrative  37235161000
##                                     X12.31.2015
## Total Revenue                      200653482000
## Cost of Revenue                    123482118000
## Gross Profit                        77171364000
## Operating Expenses                           NA
## Research Development                13705695000
## Selling General and Administrative  36081636000
```

1. `!duplicated()` 함수를 사용해 중복되지 않는 계정명만을 선택해 줍니다.
2. 행이름을 초기화 한 후, 첫번째 열의 계정명을 행이름으로 변경합니다. 그 후 첫번째 열은 삭제해주도록 합니다.
3. 열이름으로 첫번째 행으로 변경한 후, 해당 행은 삭제해주도록 합니다.
4. `substr()` 함수를 이용하여 처음 두 글자가 **12**인 열, 즉 12월 결산 데이터만을 선택해 줍니다. 
5. `sapply()` 함수를 이용해 각 열에 `stringr` 패키지의 `str_replace_all()` 함수를 적용하여 콤마(,)를 제거한 후, `as.numeric()` 함수를 통해 숫자형 데이터로 변경합니다.
6. `data.frame()` 함수를 이용해 데이터프레임 형태로 만들어주며, 행이름은 기존 내용을 그대로 유지해줍니다.


### 가치지표 계산하기

가치지표를 계산하는 과정도 앞선 예시와 거의 동일합니다.


```r
value_type =
  c('Net Income Applicable To Common Shares', # Earnings
    'Total Stockholder Equity', # Book Value
    'Total Cash Flow From Operating Activities', # Cash Flow
    'Total Revenue') # Sales

value_index = data_fs[match(value_type, rownames(data_fs)), 1]

print(value_index)
```

```
## [1]  43890877000 240068993000  67031863000 243771415000
```

먼저 분모에 해당하는 항목을 저장한 후, `match()` 함수를 이용하여 해당 항목이 위치하는 지점을 찾아 데이터를 선택해줍니다.기존 예제와 다른점은, 야후 파이낸스의 경우 최근년도 데이터가 가장 좌측에 위치하므로, 첫번째 열을 선택해 줍니다.

다음으로 현재 주가 및 상장주식수는 Statistics 항목에서 구할 수 있습니다. 먼저 현재 주가를 크롤링 하는 방법입니다.


```r
url = paste0(
  'https://finance.yahoo.com/quote/005930.KS/',
  'key-statistics?p=005930.KS')

data = GET(url)

price = read_html(data) %>%
  html_node(
    xpath =
    '//*[@id="quote-header-info"]/div[3]/div/div/span[1]') %>%
  html_text() %>%
  parse_number()

print(price)
```

```
## [1] 45650
```

1. 해당 페이지 url을 저장 후, `GET()` 함수를 통해 페이지 정보를 받습니다.
2. `read_html()` 함수를 이용해 html 데이터를 받고, `html_node()` 함수와 Xpath를 이용해 현재 주가에 해당하는 부분을 추출합니다. 주가의 경우 페이지 상단에서 확인할 수 있습니다.
3. `html_text()` 함수를 이용해 텍스트 데이터만을 추출한 후, `parse_number()` 함수를 통해 콤마 삭제 및 숫자형태로 변경합니다.

이처럼 주가의 경우 상대적으로 쉽게 데이터를 구할 수 있습니다. 다음은 상장주식수 데이터를 크롤링하는 방법입니다.


```r
share_xpath = 
  paste0('//*[@id="Col1-0-KeyStatistics-Proxy"]/section',
  '/div[2]/div[2]/div/div[2]/table/tbody/tr[3]/td[2]')

share_yahoo = read_html(data) %>% 
  html_node(xpath = share_xpath) %>%
  html_text() 

print(share_yahoo)
```

```
## [1] "5.97B"
```

상장주식수의 경우 **Shares Outstanding** 부분에서 찾을 수 있습니다. 해당 지점의 Xpath를 이용해 데이터를 찾으면 5.97B가 추출됩니다. 이 중 숫자 뒤 알파벳 부분은 단위에 해당하며, 각 문자 별 단위는 다음과 같습니다.

\begin{table}[!h]

\caption{(\#tab:unnamed-chunk-36)발행주식수 단위}
\centering
\begin{tabular}{ccc}
\toprule
알파벳 & 단위 & 숫자\\
\midrule
\rowcolor{gray!6}  M & 백만 (Million) & 1,000,000\\
B & 십억 (Billion) & 1,000,000,000\\
\rowcolor{gray!6}  T & 일조 (Triliion) & 1,000,000,000,000\\
\bottomrule
\end{tabular}
\end{table}

따라서 알파벳을 해당하는 숫자로 변경한 뒤, 이를 앞의 숫자에 곱해주어야 제대로 된 상장주식수가 계산됩니다.




```r
library(stringr)

share_unit = str_match(share_yahoo, '[a-zA-Z]')
print(share_unit)
```

```
##      [,1]
## [1,] "B"
```

```r
share_multiplier = switch(share_unit, 
       'M' = { 1000000 },
       'B' = { 1000000000 },
       'T' = { 1000000000000 }
)
print(share_multiplier)
```

```
## [1] 1000000000
```

먼저 `str_match()` 함수 내에서 정규표현식을 사용하여 알파벳을 추출한 후, 이를 share_unit에 저장합니다. 그 후, `switch` 구문을 이용하여 알파벳에 해당하는 숫자를 share_multiplier에 저장해줍니다. 




```r
share_yahoo = share_yahoo %>% 
  str_match('[0-9.0-9]*') %>% as.numeric()
share_yahoo = share_yahoo * share_multiplier

print(share_yahoo)
```

```
## [1] 5970000000
```

숫자 부분과 위에서 구한 단위 부분을 곱하여 최종 발행주식수를 구하도록 하겠습니다. 먼저 `str_match()` 함수 내에 정규표현식을 이용하여 숫자에 해당하는 부분만 추출한 후, `as.numeric()`을 통해 숫자 형태로 변경합니다. 그 후 단위에 해당하는 숫자를 곱해 최종값을 구하도록 합니다.

위에서 구한 재무데이터, 현재주가, 발행주식수를 이용하여 가치지표를 계산하도록 하겠습니다.


```r
data_value_yahoo = price / (value_index * 1000 / share_yahoo)
names(data_value_yahoo) = c('PER', 'PBR', 'PCR', 'PSR')

data_value_yahoo[data_value_yahoo < 0] = NA
print(data_value_yahoo)
```

```
##   PER   PBR   PCR   PSR 
## 6.209 1.135 4.066 1.118
```

분자에는 주가를, 분모에는 재무 데이터를 보통주 발행주식수로 나눈 값을 입력합니다. 야후 파이낸스의 재무 데이터는 천원 단위이므로, 둘 간의 단위를 동일하게 맞춰주기 위해 분모에 천을 곱해 줍니다. 또한 가치지표가 음수인 경우는 `NA`로 변경해주도록 합니다.

결과를 확인해보면 4가지 가치지표가 잘 계산되었습니다. Company Guide에서 구한 값 6.21, 1.1, 4.07, 1.12 와는 재무데이터의 차이로 인해 미세한 차이가 있지만, 이는 무시해도 될 정도입니다.

해당 방법 또한 url의 티커 부분만 변경하면 전 종목의 재무제표와 가치지표 데이터를 다운로드 받을 수 있습니다. 그러나 주의해야 할점은 코스피 종목은 끝이 .KS, 코스닥 종목은 끝이 .KQ가 되어야 합니다. 자세한 코드는 생략하도록 합니다.