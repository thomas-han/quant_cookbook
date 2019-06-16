# 퀀트 전략을 이용한 종목선정 (기본)

API와 크롤링을 이용하여 데이터를 모두 수집한 뒤, 이를 정리하였습니다. 데이터가 준비되었으므로 이제 각종 퀀트 전략을 활용하여 투자하고자 하는 종목을 선정해야 합니다. 

퀀트 투자는 크게 포트폴리오 운용 전략과 트레이딩 전략으로 나눌 수 있습니다. 포트폴트폴리오 운용 전략의 경우 과거 주식 시장을 분석하여 좋은 주식의 기준을 찾아낸 후 해당 기준에 만족하는 종목을 매수하거나, 이와 정반대에 있는 나쁜 주식을 공매도 하기도 합니다. 투자의 속도가 매우 느리며, 다수의 종목을 하나의 포트폴리오로 구성하여 운용하는 특징이 있습니다. 반면 트레이딩 전략의 경우, 단기간에 발생되는 주식의 움직임을 연구한 후 예측하여, 매수 혹은 매도하는 전략입니다. 투자의 속도가 매우 빠르며 소수의 종목을 대상으로 합니다.


Table: (\#tab:unnamed-chunk-1)퀀트 투자 종류의 비교

기준          포트폴리오 운용 전략   트레이딩 전략             
------------  ---------------------  --------------------------
투자철학      규칙에 기반한 투자     규칙에 기반한 투자        
투자목적      좋은 주식을 매수       좋은 시점을 매수          
학문적 기반   경제학, 통계학 등      통계학, 공학, 정보처리 등 
투자의 속도   느림                   빠름                      

이 중 본 책에서는 포트폴리오에 기반한 운용 전략에 대해 다루도록 합니다. 과거의 데이터를 바탕으로 주식의 수익률에 영향을 지표를 팩터^Factor^라 합니다. 즉 팩터의 강도가 양인 종목들로 구성한 포트폴리오의 경우 향후 수익률이 높을 것으로 예상되며, 팩터의 강도가 음인 종목들로 구성한 포트폴리오의 경우 반대로 향후 수익률이 낮을 것으로 예상됩니다.

팩터에 대한 연구는 학자들에 의해 수십년간 끊임없이 진행되어 왔지만, 일반 투자자들이 이러한 논문을 모두 찾아보고 연구하는 것은 사실상 불가능에 가깝습니다. 그러나 최근에는 **스마트베타**라는 이름으로 팩터 투자가 대중화되고 있습니다. 최근 유행하고 있는 스마트베타 ETF의 경우 팩터를 기준으로 포트폴리오를 구성한 상품으로써, 학계나 실무에서 검증된 팩터 전략을 기반으로 합니다.

해당 상품들의 홈페이지나 투자설명서에는 종목 선정 기준에 대해 자세히 나와있으므로 이는 매우 훌륭한 투자 전략이기도 합니다. 따라서 스마트베타 ETF에 나와있는 투자 전략을 자세히 분석하는 것만으로도 훌륭한 퀀트 투자 전략을 만들 수 있습니다. 

<div class="figure" style="text-align: center">
<img src="images/factor_smartbeta.png" alt="스마트베타 ETF 전략 예시" width="70%" />
<p class="caption">(\#fig:unnamed-chunk-2)스마트베타 ETF 전략 예시</p>
</div>

본 장에서는 투자에 많이 활용되는 기본적인 팩터에 대해 알아보고, 우리가 구한 데이터를 바탕으로 각 팩터 별 투자 종목을 선택하는 방법에 대해 알아보도록 하겠습니다.

아울러 본 책에서 각종 모델을 통해 나온 종목들은, 데이터를 받은 시점에서의 종목이며 매수 추천은 아님을 밝힙니다.

## 베타 이해하기

투자자들이라면 누구나 한번 쯤 들어봤을만한 용어가 베타^Beta^ 입니다. 기본적으로 개별 주식의 수익률에 가장 크게 영향을 주는 요소는 주식시장의 움직임일수 밖에 없습니다. 아무리 좋은 주식도 주식시장이 폭락한다면 같이 떨어지며, 아무리 나쁜 주식도 주식시장이 급등한다면 대부분 같이 오르기 마련입니다.

개별 주식이 전체 주식시장의 변동에 반응하는 정도를 나타낸 값이 베타입니다. 베타가 1이라는 뜻은 주식시장과 움직임이 정확히같다는 뜻으로써, 시장 그 자체를 나타냅니다. 베타가 1.5라는 뜻은 주식시장이 수익률이 +1% 일 때 개별 주식의 수익률은 +1.5% 이며, 반대로 주식시장의 수익률이 -1% 일 때 개별 주식의 수익률은 -1.5% 입니다. 반면 베타가 0.5라는 주식시장 수익률의 절반 정도만이 움직이게 됩니다.


Table: (\#tab:unnamed-chunk-3)베타에 따른 개별 주식의 수익률 움직임

베타   주식시장이 +1% 일 경우   주식시장이 -1% 일 경우 
-----  -----------------------  -----------------------
0.5    +0.5%                    -0.5%                  
1.0    +1.0%                    -1.0%                  
1.5    +1.5%                    -1.5%                  

이처럼 베타가 큰 주식은 주식시장보다 수익률의 움직임이 크며, 반대로 베타가 낮은 주식은 주식시장보다 수익률의 움직임이 작습니다. 따라서 일반적으로 상승장이 기대될 때는 베타가 큰 주식에, 하락장일이 기대될 때는 베타가 낮은 주식에 투자하는 것이 좋습니다.

주식시장에서의 베타는 통계학의 회귀분석모형에서 기울기를 나타내는 베타와 정확히 의미가 같습니다. 회귀분석모형은 $y = a + bx$ 형태로 나타나며, x의 변화에 따른 y의 변화의 기울기가 회귀계수인 b입니다. 이를 주식에 적용한 모형이 자산가격결정모형(CAPM: Capital Asset Pricing Model)이며, 그 식은 다음과 같습니다.

$$회귀분석모형: y = a + bx$$
$$자산가격결정모형: R_i = R_f + \beta_i×[R_m - R_f]$$

먼저 회귀분석모형의 상수항인 a에 해당하는 부분은 무위험 수익률을 나타내는 $R_f$입니다. 독립변수인 x에 해당하는 부분은 무위험 수익률 대비 주식 시장의 초과 수익률을 나태내는 $R_m - R_f$입니다. 종속변수인 y에 해당하는 부분은 개별주식의 수익률을 나타내는 $R_i$이며, 최종적으로 회귀계수인 b에 해당하는 부분은 개별 주식의 베타인 입니다.


Table: (\#tab:unnamed-chunk-4)회귀분석모형과 자산가격결정모형의 비교

구분       회귀분석모형   자산가격결정모형            
---------  -------------  ----------------------------
상수항     a              $R_f$ (무위험 수익률)       
독립변수   x              $R_m - R_f$ (초과 수익률)   
종속변수   y              $R_i$ (개별주식의 수익률    
회귀계수   b              $\beta_i$ (개별주식의 베타) 

통계학에서 회귀계수는 $\beta = \frac{cov(x,y)}{σ_x^2}$ 형태로 구할 수 있으며, x와 y에 각각 시장수익률과 개별주식의 수익률을 대입할 경우 개별주식의 베타는 $\beta_i= ρ(i,m) ×  \frac{σ_i}{σ_m}$  형태로 구할 수 있습니다. 그러나 이러한 수식을 모르더라도 R에서는 간단히 베타를 구할 수 있습니다.

### 베타 계산하기

베타를 구하는 방법을 알아보기 위해 주식시장에 대한 대용치로 KOSPI 200 ETF, 전통적 고베타주인 증권주를 이용하도록 하겠습니다.


```r
library(quantmod)
library(PerformanceAnalytics)
library(magrittr)

symbols = c('102110.KS', '039490.KS')
getSymbols(symbols)
```

```
## [1] "102110.KS" "039490.KS"
```

```r
prices = do.call(cbind, lapply(symbols, function(x) Cl(get(x))))

ret = Return.calculate(prices)
ret = ret['2016-01::2018-12']
```

1. KOSPI 200 ETF인 TIGER 200(102110.KS), 증권주인 키움증권(039490.KS)의 티커를 입력합니다.
2. `getSymbols()` 함수를 이용하여 해당 티커들의 데이터가 다운로드 받습니다.
3. `lapply()` 함수 내에 `Cl()`과 `get()`함수를 사용하여 종가에 해당하는 데이터만 추출하며, 리스트 형태의 데이터를 열의 형태로 묶어주기 위해 `do.call()` 함수와 `cbind()` 함수를 사용해 줍니다.
4. `Return.calculate()` 함수를 통해 수익률을 계산해 줍니다.
5. xts 형식의 데이터는 대괄호 속에 ['시작일자::종료일자']와 같은 형태로, 원하는 날짜를 편리하게 선택할 수 있으며, 위에서는 2016년 1월부터 2018년 12월 까지 데이터를 선택합니다.


```r
rm = ret[, 1]
ri = ret[, 2]

reg = lm(ri ~ rm)
summary(reg)
```

```
## 
## Call:
## lm(formula = ri ~ rm)
## 
## Residuals:
##      Min       1Q   Median       3Q      Max 
## -0.06886 -0.01294 -0.00169  0.01080  0.09545 
## 
## Coefficients:
##              Estimate Std. Error t value Pr(>|t|)    
## (Intercept) 0.0003446  0.0007213   0.478    0.633    
## rm          1.7622492  0.0906194  19.447   <2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.0195 on 729 degrees of freedom
## Multiple R-squared:  0.3416,	Adjusted R-squared:  0.3407 
## F-statistic: 378.2 on 1 and 729 DF,  p-value: < 2.2e-16
```

증권주를 대상으로 베타를 구하기 위한 회귀분석을 실시해 주도록 합니다. 자산가격결정모형의 수식인 $R_i = R_f + \beta_i×[R_m - R_f]$ 에서 편의를 위해 무위험 수익률인 $R_f$를 0으로 가정하면, $R_i = \beta_i×R_m$의 형태로 나타낼 수 있습니다. 이 중 $R_m$는 독립변수인 주식시장의 수익률을, $R_i$는 종속변수인 개별주식의 수익률을 의미합니다.

1. 독립변수는 첫번째 열인 KOSPI 200 ETF의 수익률을 선택해주며, 종속변수는 세번째 열인 증권주의 수익률을 선택해줍니다.
2. `lm()` 함수를 통해 손쉽게 선형회귀분석을 실시할 수 있으며, 회귀분석의 결과를 reg 변수에 저장해줍니다.
3. `summary()` 함수는 데이터의 요약 정보를 나타내며, 해당 예시에서는 회귀분석결과에 대한 정보를 보여줍니다.

회귀분석의 결과 중 가장 중요한 부분은 계수를 나타내는 Coefficients 부분입니다. Intercept 부분은 회귀분석의 상수항에 해당하는 부분으로써, 값이 거의 0에 가깝고 t밸류 또한 매우 작아 유의하지 않음이 보입니다. 우리가 원하는 베타에 해당하는 부분은 x의 Estimate 부분으로써, 베타값이 1.76로 증권주의 특성인 고베타주임이 확인되며, t밸류 또한 19.45로 매우 유의한 결과입니다. 조정된 결정계수(Adjusted R-square)는 0.34를 보입니다.

### 베타 시각화 ###

다음으로 구해진 베타를 그림으로 표현해보도록 하겠습니다.


```r
plot(as.numeric(rm), as.numeric(ri), pch = 4, cex = 0.3, 
     xlab = "KOSPI 200", ylab = "Individual Stock",
     xlim = c(-0.02, 0.02), ylim = c(-0.02, 0.02))
grid()
abline(a = 0, b = 1, lty = 2)
abline(reg, col = 'red')
```

<img src="09-factor_basic_files/figure-html/unnamed-chunk-7-1.png" width="40%" style="display: block; margin: auto;" />

1. `plot()` 함수를 통해 그림을 그려주며, x축과 y축에 주식시장 수익률과 개별주식 수익률을 입력합니다.pch는 점들의 모양을, cex는 점들의 크기를 나타내며, xlab과 ylab은 각각 x축과 y축에 들어갈 문구를 나타냅니다. xlim과 ylim은 x축과 y축의 최소 및 최대 범위를 지정해줍니다.
2. `grid()` 함수를 통해 격자무늬를 추가해줍니다.
3. 첫번째 `abline()`에서 a는 상수, b는 직선의 기울기, lty는 선의 유형을 나타냅니다. 이를 통해 기울기, 즉 베타가 1일 경우의 선을 점선으로 표현합니다.
4. 두번째 `abline()`에 회귀분석 결과를 입력해주면 자동적으로 회귀식을 그려줍니다. 

검은색의 점선이 기울기가 1인 경우이며, 붉은색의 직선이 증권주의 회귀분석결과를 나타냅니다. 기울기가 1보다 훨씬 가파름이 확인되며, 즉 베타가 1보다 크다는 사실을 알 수 있습니다. 

## 저변동성 전략

금융 시장에서 변동성은 수익률이 움직이는 정도로써, 일반적으로 표준편차가 사용됩니다. 표준편차는 자료가 평균을 중심으로 얼마나 퍼져 있는지를 나타내는 수치로써, 수식은 다음과 같습니다.

$$\sigma = \sqrt{\frac{\sum_{i=1}^{n}{(x_i - \bar{x})^2}}{n-1}}$$

관측값의 개수가 작을 경우에는 수식에 대입하여 계산하는 것이 가능하지만, 관측값이 수백 혹은 수천개로 늘어날 경우 컴퓨터를 이용하지 않고 계산하는 것은 사실상 불가능합니다. R에서는 복잡한 계산과정 없이 `sd()` 함수를 이용하여 간단하게 표준편차를 계산할 수 있습니다.


```r
example = c(85, 76, 73, 80, 72)
sd(example)
```

```
## [1] 5.357238
```

개별 주식의 표준편차를 측정할 때는 주식의 가격이 아닌 수익률로 계산해야 합니다. 수익률의 표준편차가 크다는 의미는 수익률의 위 아래로 많이 움직여 위험한 종목으로 여겨집니다. 반면, 표준편차가 작다는 의미는 수익률의 움직임이 적어 상대적으로 안전한 종목으로 여겨집니다.

전통적 금융 이론에서는 수익률의 변동성이 클수록 위험이 크고, 이런 위험에 대한 보상으로 기대수익률이 높아야 한다고 보았습니다. 따라서 고변동성 종목의 기대수익률이 크고, 저변동성 종목의 기대수익률이 낮은 고위험 고수익이 당연한 믿음이었습니다. 그러나 현실에서는 오히려 변동성이 낮은 종목들의 수익률이 변동성이 높은 종목들의 수익률 보다 높은, 저변동성 효과가 발견되고 있습니다. 이러한 저변동성 효과가 발생하는 원인으로는 여러 가설이 있습니다.

1. 투자자들은 대체로 자신의 능력을 과신하는 경향이 있으며, 복권과 같이 큰 수익을 가져다 주는 고변동성 주식을 선호하는 경향이 있습니다. 이러한 결과로 고변동성 주식은 과대 평가가 되어 수익률이 낮은 반면, 과소 평가된 저변동성 주식들은 높은 수익률을 보이게 됩니다.
2. 대부분 기관투자가들이 레버리지 투자가 되지 않는 상황에서, 벤치마크 대비 높은 성과를 얻기 위해 고변동성 주식에 투자하는 경향이 있으며, 이 또한 고변동성 주식이 과대 평가되는 결과로 이어집니다.
3. 시장의 상승과 하락이 반복됨에 따라 고변동성 주식이 변동성 손실(Volatility Drag)로 인해 수익률이 하락하게 되는 이유도 있습니다.

주식의 위험은 변동성뿐만 아니라 베타 등 여러 지표로도 측정할 수 있습니다. 저변동성 효과와 비슷하게 고유변동성이 낮은 주식의 수익률이 높은 저고유변동성 효과, 베타가 낮은 주식의 수익률이 오히려 높은 저베타 효과도 발견되고 있으며, 이러한 효과들을 합쳐 저위험 효과로 부르기도 합니다.

### 저변동성 포트폴리오 구하기: 일간 기준

먼저 최근 1년 일간 수익률 기준 변동성이 낮은 30종목을 선택하도록 하겠습니다.


```r
library(stringr)
library(xts)
library(PerformanceAnalytics)
library(magrittr)
library(ggplot2)
library(dplyr)

KOR_price = read.csv('data/KOR_price.csv', row.names = 1, stringsAsFactors = FALSE) %>% as.xts()
KOR_ticker = read.csv('data/KOR_ticker.csv', row.names = 1, stringsAsFactors = FALSE) 
KOR_ticker$'종목코드' = str_pad(KOR_ticker$'종목코드', 6, 'left', 0)

ret = Return.calculate(KOR_price)
std_12m_daily = xts::last(ret, 252) %>% apply(., 2, sd) %>% multiply_by(sqrt(252))
```

1. 먼저 미리 저장해둔 가격 정보와 티커 정보를 불러오도록 하며, 가격 정보의 경우 `as.xts()` 함수를 통해 xts 형태로 변경해주도록 합니다.
2. `Return.calculate()` 함수를 통해 수익률을 구하도록 합니다.
3. `last()` 함수는 마지막 n개를 선택해주는 함수이며, 1년 영업일 기준인 252개 데이터를 선택합니다. `dplyr` 패키지의 last() 함수와 변수명이 같으므로, `xts::last()` 형식을 통해 xts 패키지의 함수임을 정의해줍니다.
4. `apply()` 함수를 sd 즉 변동성을 계산해주며, 연율화를 해주기 위해 `multiply_by()` 함수를 통해 $\sqrt{252}$를 곱해주도록 합니다.


```r
std_12m_daily %>% 
  data.frame() %>%
  ggplot(aes(x = (`.`))) +
  geom_histogram(binwidth = 0.01) +
  xlab(NULL)
```

```
## Warning: Removed 78 rows containing non-finite values (stat_bin).
```

<img src="09-factor_basic_files/figure-html/unnamed-chunk-10-1.png" width="40%" style="display: block; margin: auto;" />

```r
std_12m_daily[std_12m_daily == 0] = NA
```

변동성을 히스토그램으로 나타내보면, 0에 위치하는 종목들이 다수 존재합니다. 해당 종목들은 최근 1년간 거래정지로 인해 가격이 변하지 않았고, 이로 인해 변동성이 없는 종목들이며, 해당 종믁들은 NA로 처리해주도록 합니다.


```r
std_12m_daily[rank(std_12m_daily) <= 30]
```

```
##    X030200    X023890    X001720    X019680    X015350    X017390 
## 0.15864637 0.17424383 0.14822527 0.13569188 0.12741449 0.16627173 
##    X034950    X015360    X092230    X018120    X092130    X006220 
## 0.13529149 0.11582731 0.13549969 0.10114390 0.14229132 0.15929146 
##    X117580    X003460    X003650    X034590    X007330    X023000 
## 0.11257087 0.09959043 0.16544129 0.05185911 0.14207463 0.16203629 
##    X040420    X000650    X003080    X004450    X001750    X006660 
## 0.11440447 0.16573652 0.13685077 0.14298328 0.13110224 0.16731155 
##    X014440    X084670    X115310    X066670    X025530    X066790 
## 0.17678836 0.17254448 0.15728105 0.15979474 0.17386322 0.10923847
```

```r
std_12m_daily[rank(std_12m_daily) <= 30] %>%
  data.frame() %>%
  ggplot(aes(x = rep(1:30), y = `.`)) +
  geom_col() +
  xlab(NULL)
```

<img src="09-factor_basic_files/figure-html/unnamed-chunk-11-1.png" width="40%" style="display: block; margin: auto;" />

`rank()` 함수를 통해 순위를 구할 수 있으며, R은 기본적으로 오름차순 즉 가장 낮은 값의 순위가 1이 됩니다. 따라서 변동성이 낮을수록 높은 순위가 되며, 30위 이하의 순위를 선택하면 변동성이 낮은 30종목이 선택됩니다. 또한 `ggplot()` 함수를 이용해 해당 종목들의 변동성을 확인해볼 수도 있습니다.

이번에는 해당 종목들의 티커 및 종목명을 확인하도록 하겠습니다.


```r
invest_lowvol = rank(std_12m_daily) <= 30
KOR_ticker[invest_lowvol, ] %>%
  select(`종목코드`, `종목명`) %>%
  mutate(`변동성` = std_12m_daily[invest_lowvol])
```

```
##    종목코드             종목명     변동성
## 1    030200                 KT 0.15864637
## 2    023890 한국아트라스비엑스 0.17424383
## 3    001720           신영증권 0.14822527
## 4    019680               대교 0.13569188
## 5    015350           부산가스 0.12741449
## 6    017390           서울가스 0.16627173
## 7    034950       한국기업평가 0.13529149
## 8    015360       예스코홀딩스 0.11582731
## 9    092230          KPX홀딩스 0.13549969
## 10   018120           진로발효 0.10114390
## 11   092130         이크레더블 0.14229132
## 12   006220           제주은행 0.15929146
## 13   117580         대성에너지 0.11257087
## 14   003460           유화증권 0.09959043
## 15   003650           미창석유 0.16544129
## 16   034590       인천도시가스 0.05185911
## 17   007330       푸른저축은행 0.14207463
## 18   023000           삼원강재 0.16203629
## 19   040420     정상제이엘에스 0.11440447
## 20   000650           천일고속 0.16573652
## 21   003080           성보화학 0.13685077
## 22   004450           삼화왕관 0.14298328
## 23   001750           한양증권 0.13110224
## 24   006660           삼성공조 0.16731155
## 25   014440           영보화학 0.17678836
## 26   084670           동양고속 0.17254448
## 27   115310           인포바인 0.15728105
## 28   066670       디스플레이텍 0.15979474
## 29   025530          SJM홀딩스 0.17386322
## 30   066790           씨씨에스 0.10923847
```

티커와 종목명, 그리고 연율화 변동성을 확인할 수 있습니다.

### 저변동성 포트폴리오 구하기: 주간 기준

이번에는 일간 변동성이 아닌 주간 변동성을 기준으로 저변동성 종목을 선택하도록 하겠습니다.


```r
std_12m_weekly = xts::last(ret, 252) %>%
  apply.weekly(Return.cumulative) %>%
  apply(., 2, sd) %>% multiply_by(sqrt(52))

std_12m_weekly[std_12m_weekly == 0] = NA
```

먼저 최근 252일 수익률울 선택한 후, `apply.weekly()` 함수 내 Return.cumulative를 입력하여 주간 수익률을 계산해주도록 합니다. 이 외에도 `apply.monthly()`, `apply.yearly()` 함수 등으로 일간 수익률을 월간, 연간 수익률 등으로 변환할 수 있습니다. 그 후 과정은 위와 동일합니다.



```r
std_12m_weekly[rank(std_12m_weekly) <= 30]
```

```
##    X316140    X030200    X001720    X019680    X002960    X015350 
## 0.15301163 0.15208526 0.11643598 0.13454286 0.14150612 0.13047960 
##    X017390    X034950    X015360    X092230    X018120    X092130 
## 0.15022308 0.12518089 0.09822208 0.11601173 0.07347689 0.15038813 
##    X117580    X003460    X024090    X003650    X034590    X007330 
## 0.12148441 0.08236426 0.16035917 0.13999037 0.03658793 0.14150051 
##    X040420    X003080    X004450    X001750    X009770    X006660 
## 0.10920275 0.15900078 0.11346761 0.12567221 0.16259817 0.16515502 
##    X115310    X066670    X049430    X033200    X002070    X066790 
## 0.12689801 0.14389858 0.14577188 0.16081369 0.15707650 0.15243463
```

```r
invest_lowvol_weekly = rank(std_12m_weekly) <= 30
KOR_ticker[invest_lowvol_weekly, ] %>%
  select(`종목코드`, `종목명`) %>%
  mutate(`변동성` = std_12m_weekly[invest_lowvol_weekly])
```

```
##    종목코드         종목명     변동성
## 1    316140   우리금융지주 0.15301163
## 2    030200             KT 0.15208526
## 3    001720       신영증권 0.11643598
## 4    019680           대교 0.13454286
## 5    002960     한국쉘석유 0.14150612
## 6    015350       부산가스 0.13047960
## 7    017390       서울가스 0.15022308
## 8    034950   한국기업평가 0.12518089
## 9    015360   예스코홀딩스 0.09822208
## 10   092230      KPX홀딩스 0.11601173
## 11   018120       진로발효 0.07347689
## 12   092130     이크레더블 0.15038813
## 13   117580     대성에너지 0.12148441
## 14   003460       유화증권 0.08236426
## 15   024090         디씨엠 0.16035917
## 16   003650       미창석유 0.13999037
## 17   034590   인천도시가스 0.03658793
## 18   007330   푸른저축은행 0.14150051
## 19   040420 정상제이엘에스 0.10920275
## 20   003080       성보화학 0.15900078
## 21   004450       삼화왕관 0.11346761
## 22   001750       한양증권 0.12567221
## 23   009770       삼정펄프 0.16259817
## 24   006660       삼성공조 0.16515502
## 25   115310       인포바인 0.12689801
## 26   066670   디스플레이텍 0.14389858
## 27   049430         코메론 0.14577188
## 28   033200         모아텍 0.16081369
## 29   002070     남영비비안 0.15707650
## 30   066790       씨씨에스 0.15243463
```

주간 수익률의 변동성이 낮은 30종목을 선택하여 종목코드, 종목명, 연율화 변동성을 확인하도록 합니다.



```r
intersect(KOR_ticker[invest_lowvol, '종목명'], KOR_ticker[invest_lowvol_weekly, '종목명'])
```

```
##  [1] "KT"             "신영증권"       "대교"           "부산가스"      
##  [5] "서울가스"       "한국기업평가"   "예스코홀딩스"   "KPX홀딩스"     
##  [9] "진로발효"       "이크레더블"     "대성에너지"     "유화증권"      
## [13] "미창석유"       "인천도시가스"   "푸른저축은행"   "정상제이엘에스"
## [17] "성보화학"       "삼화왕관"       "한양증권"       "삼성공조"      
## [21] "인포바인"       "디스플레이텍"   "씨씨에스"
```

`intersect()` 함수를 통해 일간 변동성 기준과 주간 변동성 기준 모두에 포함되는 종목을 찾을 수 있습니다.

## 모멘텀 전략

투자에서 모멘텀이란 주가 혹은 이익의 추세로써, 상승 추세의 주식은 지속적으로 상승하며 하락 추세의 주식은 지속적으로 하락하는 현상을 말합니다. 모멘텀 현상이 발생하는 원인 중 가장 큰 이유는 투자자들의 스스로에 대한 과잉 신뢰 때문입니다. 사람들은 자신의 판단을 지지하는 정보에 대해서는 과잉 반응하는, 자신의 판단을 부정하는 정보에 대해서는 과소 반응하는 경향이 있습니다. 이러한 투자자들의 비합리성으로 인해 모멘텀 현상이 생겨나게 됩니다.

모멘텀의 종류는 크게 기업의 이익에 대한 추세를 나타내는 이익 모멘텀과, 주가의 모멘텀에 대한 가격 모멘텀이 있습니다. 또한 가격 모멘텀도 1주일 혹은 1개월 이하를 의미하는 단기 모멘텀, 3개월에서 12개월을 의미하는 중기 모멘텀, 3년에서 5년을 의미하는 장기 모멘텀이 있으며, 이중에서도 3개월에서 12개월 가격 모멘텀을 흔히 모멘텀이라 합니다.

### 모멘텀 포트폴리오 구하기: 12개월 모멘텀

먼저 최근 1년 동안의 수익률이 높은 30 종목을 선택하도록 하겠습니다.


```r
library(stringr)
library(xts)
library(PerformanceAnalytics)
library(magrittr)
library(dplyr)

KOR_price = read.csv('data/KOR_price.csv', row.names = 1, stringsAsFactors = FALSE) %>% as.xts()
KOR_ticker = read.csv('data/KOR_ticker.csv', row.names = 1, stringsAsFactors = FALSE) 
KOR_ticker$'종목코드' = str_pad(KOR_ticker$'종목코드', 6, 'left', 0)

ret = Return.calculate(KOR_price) %>% xts::last(252) 
ret_12m = ret %>% sapply(., function(x) {
  prod(1+x) - 1
  })
```

1. 가격 정보와 티커 정보를 불러온 후, `Return.calculate()` 함수를 통해 수익률을 계산합니다. 그 후, 최근 252일 수익률을 선택합니다.
2. `sapply()` 함수 내부에 `prod()` 함수를 이용하여 각 종목의 누적수익률을 계산해줍니다.


```r
ret_12m[rank(-ret_12m) <= 30]
```

```
##  X081660  X032500  X091700  X078070  X214150  X230360  X033640  X011280 
## 1.670068 2.045085 1.346405 7.509434 1.529070 2.934579 1.930000 1.312155 
##  X048410  X239610  X008350  X138080  X047310  X088800  X078130  X049070 
## 2.202247 2.200000 2.228464 4.615023 1.697769 1.290837 1.978814 1.287762 
##  X179900  X143160  X263920  X009460  X176440  X033110  X263540  X190510 
## 1.900718 2.312263 2.427673 2.097778 2.806971 1.420690 2.030303 1.281407 
##  X214870  X100030  X051160  X119830  X139670  X033250 
## 1.293179 2.244240 3.036842 1.537688 1.944228 1.276000
```

`rank()` 함수를 통해 순위를 구하도록 하며, 모멘텀의 경우 높을수록 좋은 내림차순으로 순위를 계산해야 하므로 수익률 앞에 마이너스(-)를 붙여주도록 합니다. 12개월 누적수익률이 높은 종목들이 선택됨이 확인됩니다.


```r
invest_mom = rank(-ret_12m) <= 30
KOR_ticker[invest_mom, ] %>%
  select(`종목코드`, `종목명`) %>%
  mutate(`수익률` = ret_12m[invest_mom])
```

```
##    종목코드           종목명   수익률
## 1    081660       휠라코리아 1.670068
## 2    032500     케이엠더블유 2.045085
## 3    091700           파트론 1.346405
## 4    078070   유비쿼스홀딩스 7.509434
## 5    214150         클래시스 1.529070
## 6    230360       에코마케팅 2.934579
## 7    033640           네패스 1.930000
## 8    011280         태림포장 1.312155
## 9    048410       현대바이오 2.202247
## 10   239610 에이치엘사이언스 2.200000
## 11   008350       남선알미늄 2.228464
## 12   138080       오이솔루션 4.615023
## 13   047310       파워로직스 1.697769
## 14   088800       에이스테크 1.290837
## 15   078130         국일제지 1.978814
## 16   049070           인탑스 1.287762
## 17   179900         유티아이 1.900718
## 18   143160         아이디스 2.312263
## 19   263920     블러썸엠앤씨 2.427673
## 20   009460         한창제지 2.097778
## 21   176440       에이치엔티 2.806971
## 22   033110 코너스톤네트웍스 1.420690
## 23   263540             샘코 2.030303
## 24   190510           나무가 1.281407
## 25   214870           뉴지랩 1.293179
## 26   100030       모바일리더 2.244240
## 27   051160       지어소프트 3.036842
## 28   119830           아이텍 1.537688
## 29   139670       키네마스터 1.944228
## 30   033250           체시스 1.276000
```

티커와 종목명, 그리고 누적수익률을 확인할 수 있습니다.

### 모멘텀 포트폴리오 구하기: 위험조정 수익률

투자성과 평가시 단순 수익률 보다는 위험이 함께 고려된 위험조정 수익률을 사용하는 것이 일반적이며, 대표적으로 샤프지수가 사용됩니다. 모멘텀 측정 역시 단순 12개월 누적수익률을 사용하기 보다 변동성이 함께 고려된 지표를 사용할 수 있습니다. 이는 누적 수익률을 변동성으로 나누어 계산할 수 있습니다.


```r
ret = Return.calculate(KOR_price) %>% xts::last(252) 
ret_12m = ret %>% sapply(., function(x) {
  prod(1+x) - 1
  })
std_12m = ret %>% apply(., 2, sd) %>% multiply_by(sqrt(252))
sharpe_12m = ret_12m / std_12m
```

1. 최근 1년에 해당하는 수익률을 선택합니다.
2. `sapply()`와 `prod()` 함수를 이용해 분자에 해당하는 누적수익률을 계산합니다.
3. `apply()`와 `multiply_by()` 함수를 이용해 분모에 해당하는 연율화 변동성을 계산합니다.
4. 수익률을 변동성으로 나누어 위험조정 수익률을 계산해줍니다.

이를 통해 수익률이 높으면서 변동성이 낮은 종목을 선정할 수 있습니다.


```r
invest_mom_sharpe = rank(-sharpe_12m) <= 30
KOR_ticker[invest_mom_sharpe, ] %>%
  select(`종목코드`, `종목명`) %>%
  mutate(`수익률` = ret_12m[invest_mom_sharpe],
         `변동성` = std_12m[invest_mom_sharpe],
         `샤프지수` = sharpe_12m[invest_mom_sharpe])
```

```
##    종목코드             종목명    수익률    변동성  샤프지수
## 1    081660         휠라코리아 1.6700680 0.4908608  3.402325
## 2    032500       케이엠더블유 2.0450850 0.5830457  3.507589
## 3    091700             파트론 1.3464052 0.3953022  3.406015
## 4    078070     유비쿼스홀딩스 7.5094340 0.6193257 12.125177
## 5    214150           클래시스 1.5290698 0.6301642  2.426462
## 6    230360         에코마케팅 2.9345794 0.6313386  4.648186
## 7    033640             네패스 1.9300000 0.6813858  2.832463
## 8    001630       종근당홀딩스 0.8641618 0.2969553  2.910074
## 9    023890 한국아트라스비엑스 0.4294804 0.1742438  2.464824
## 10   011280           태림포장 1.3121547 0.5594643  2.345377
## 11   048410         현대바이오 2.2022472 0.9803136  2.246472
## 12   239610   에이치엘사이언스 2.2000000 0.6067024  3.626160
## 13   008350         남선알미늄 2.2284644 0.7675554  2.903327
## 14   138080         오이솔루션 4.6150235 0.6113676  7.548688
## 15   047310         파워로직스 1.6977688 0.5920970  2.867383
## 16   049070             인탑스 1.2877619 0.5077941  2.535992
## 17   123860           아나패스 1.0710059 0.4485606  2.387650
## 18   179900           유티아이 1.9007177 0.7737683  2.456443
## 19   143160           아이디스 2.3122630 0.8399575  2.752833
## 20   263920       블러썸엠앤씨 2.4276730 0.8194336  2.962623
## 21   068930         디지털대성 1.0968421 0.4484235  2.445996
## 22   009460           한창제지 2.0977778 0.7722633  2.716402
## 23   176440         에이치엔티 2.8069705 0.8530826  3.290385
## 24   263540               샘코 2.0303030 0.5992449  3.388102
## 25   190510             나무가 1.2814070 0.4532505  2.827150
## 26   214870             뉴지랩 1.2931785 0.5292056  2.443622
## 27   100030         모바일리더 2.2442396 0.6386265  3.514166
## 28   051160         지어소프트 3.0368421 0.8031847  3.781001
## 29   104460       동양피엔에프 1.1487179 0.4303734  2.669119
## 30   139670         키네마스터 1.9442283 0.7706976  2.522686
```

티커와 종목명, 누적수익률, 변동성, 샤프지수를 확인할 수 있습니다.


```r
intersect(KOR_ticker[invest_mom, '종목명'], KOR_ticker[invest_mom_sharpe, '종목명'])
```

```
##  [1] "휠라코리아"       "케이엠더블유"     "파트론"          
##  [4] "유비쿼스홀딩스"   "클래시스"         "에코마케팅"      
##  [7] "네패스"           "태림포장"         "현대바이오"      
## [10] "에이치엘사이언스" "남선알미늄"       "오이솔루션"      
## [13] "파워로직스"       "인탑스"           "유티아이"        
## [16] "아이디스"         "블러썸엠앤씨"     "한창제지"        
## [19] "에이치엔티"       "샘코"             "나무가"          
## [22] "뉴지랩"           "모바일리더"       "지어소프트"      
## [25] "키네마스터"
```

`intersect()` 함수를 통해 단순 수익률 및 위험조정 수익률 기준 모두에 포함되는 종목을 찾을 수 있습니다. 다음 그림은 위험조정 수익률 상위 30 종목의 가격 그래프입니다.


```r
library(xts)
library(tidyr)
library(ggplot2)

KOR_price[, invest_mom_sharpe] %>%
  fortify.zoo() %>%
  gather(ticker, price, -Index) %>%
  ggplot(aes(x = Index, y = price)) +
  geom_line() +
  facet_wrap(. ~ ticker, scales = 'free') +
  xlab(NULL) +
  ylab(NULL) +
  theme(axis.text.x=element_blank(),
        axis.text.y=element_blank())
```

<img src="09-factor_basic_files/figure-html/unnamed-chunk-22-1.png" width="672" style="display: block; margin: auto;" />

## 밸류 전략

가치주 효과란 내재 가치 대비 낮은 가격의 주식(저 PER, 저 PBR 등)이, 내재 가치 대비 비싼 주식보다 수익률이 높은 현상을 뜻합니다. 가치 효과가 발생하는 원인에 대한 이론은 다음가 같습니다.

1. 위험한 기업은 시장에서 상대적으로 낮은 가격에 거래되며, 이러한 위험을 감당하는 대가로 수익이 발생
2. 투자자들의 성장주에 대한 과잉 반응으로 인해 가치주는 시장에서 소외되며, 제자리를 찾아가는 과정에서 수익이 발생

가치를 나타내는 지표는 굉장히 많지만, 일반적으로 PER, PBR, PCR, PSR가 많이 사용됩니다.

### 밸류 포트폴리오 구하기: 저 PBR 

먼저 기업의 가치 여부를 판단할 때 가장 많이 사용되는 지표인 PBR을 이용한 포트폴리오를 구성하도록 하겠습니다.


```r
library(stringr)
library(ggplot2)

KOR_value = read.csv('data/KOR_value.csv', row.names = 1, stringsAsFactors = FALSE)
KOR_ticker = read.csv('data/KOR_ticker.csv', row.names = 1, stringsAsFactors = FALSE) 
KOR_ticker$'종목코드' = str_pad(KOR_ticker$'종목코드', 6, 'left', 0)

invest_pbr = rank(KOR_value$PBR) <= 30
KOR_ticker[invest_pbr, ] %>%
  select(`종목코드`, `종목명`) %>%
  mutate(`PBR` = KOR_value[invest_pbr, 'PBR'])
```

```
##    종목코드           종목명        PBR
## 1    015760         한국전력 0.22890911
## 2    000880             한화 0.11606599
## 3    034020       두산중공업 0.20544967
## 4    006120     SK디스커버리 0.20650481
## 5    032190       다우데이타 0.14267200
## 6    058650       세아홀딩스 0.12103881
## 7    005720             넥센 0.18200630
## 8    003300       한일홀딩스 0.18593347
## 9    001940      KISCO홀딩스 0.20079881
## 10   092230        KPX홀딩스 0.21762987
## 11   002030           아세아 0.17733393
## 12   003030     세아제강지주 0.16810363
## 13   036530        S&T홀딩스 0.15183556
## 14   000140 하이트진로홀딩스 0.19737822
## 15   033160       엠케이전자 0.22069751
## 16   035080   인터파크홀딩스 0.19322603
## 17   009200       무림페이퍼 0.19913383
## 18   042420   네오위즈홀딩스 0.23330114
## 19   007860             서연 0.10661426
## 20   002300         한국제지 0.18877865
## 21   005010           휴스틸 0.21588468
## 22   040610             SG&G 0.11514814
## 23   031980 피에스케이홀딩스 0.20475027
## 24   025530        SJM홀딩스 0.20175300
## 25   006200   한국전자홀딩스 0.14141955
## 26   000950             전방 0.23116800
## 27   037400         우리조명 0.17634443
## 28   194510       파티게임즈 0.20834533
## 29   192410           감마누 0.21648539
## 30   149940             모다 0.05385654
```

먼저 가치 지표들을 저장한 데이터와 티커 데이터를 불러오도록 하며, `rank()`를 통해 PBR이 낮은 30 종목을 선택해주도록 합니다. 그 후 종목코드와 종목명, PBR을 확인해보도록 합니다. 많은 홀딩스 등 지주사가 특성상 저PBR 포트폴리오에 많이 구성되어 있습니다.

### 각 지표를 결합하기

저PBR 하나의 지표만으로도 우수한 성과를 거둘 수 있음은 오랜 기간동안 증명되고 있습니다. 그러나 저평가 주식이 계속해서 저평가에 머무르는 가치 함정에 빠지지 않기 위해서는, 여러 지표를 동시에 볼 필요도 있습니다.


```r
rank_sum = KOR_value %>% 
  apply(., 2, rank) %>%
  rowSums()

invest_value = rank(rank_sum) <= 30

KOR_ticker[invest_value, ] %>%
  select(`종목코드`, `종목명`) %>%
  cbind(KOR_value[invest_value, ])
```

```
##      종목코드           종목명        PER       PBR       PCR        PSR
## 15     034730               SK  7.4635440 0.3304540 2.1404074 0.16567271
## 87     001040               CJ 10.8317359 0.2388349 1.9405894 0.10129735
## 114    000880             한화  4.2008258 0.1160660 0.7157234 0.04037051
## 150    042670   두산인프라코어  5.4405789 0.3508581 1.6000939 0.17342061
## 274    006840         AK홀딩스  5.8120731 0.4215419 1.9581173 0.16741502
## 408    017940               E1  4.9100384 0.2914158 2.0761992 0.08282018
## 440    058650       세아홀딩스 11.0975610 0.1210388 2.5870647 0.07031235
## 458    005720             넥센  5.5093481 0.1820063 2.5271494 0.25197166
## 478    015750       성우하이텍 14.2222222 0.2701055 1.2757475 0.09997397
## 497    003300       한일홀딩스  0.6267306 0.1859335 2.5311606 0.27024357
## 555    084690       대상홀딩스 11.6034218 0.2490687 1.9108648 0.08024733
## 598    002030           아세아  4.8980976 0.1773339 1.0349839 0.15077088
## 604    013580         계룡건설  2.6532403 0.5381288 2.2734778 0.10322721
## 638    003030     세아제강지주  0.7186377 0.1681036 1.7833488 0.12761799
## 645    036530        S&T홀딩스  8.3304527 0.1518356 1.7707867 0.16288761
## 733    005990       매일홀딩스  7.4802563 0.3681428 2.6170594 0.12744735
## 796    005960         동부건설  2.2599994 0.4614920 2.8355510 0.18594294
## 900    267290     경동도시가스  4.1844914 0.4452142 1.1821968 0.08882114
## 947    005710         대원산업  4.6706861 0.4753578 1.9885800 0.18830169
## 960    009200       무림페이퍼  3.8311742 0.1991338 1.5548742 0.11986453
## 981    016710       대성홀딩스  6.5430467 0.2395595 2.2622236 0.14023896
## 1083   036000           예림당  7.4597936 0.3057456 3.3767705 0.14996038
## 1180   003480 한진중공업홀딩스 11.2213286 0.2751824 1.7655937 0.10675683
## 1295   005010           휴스틸  5.0958827 0.2158847 0.7695275 0.14886070
## 1318   037350       성도이엔지  5.6355000 0.4457458 1.7427048 0.16399603
## 1350   002200         수출포장  4.9248555 0.3672414 2.0285714 0.30406852
## 1513   037330     인지디스플레  7.2892757 0.4026053 3.5694907 0.13461920
## 1584   004140             동방  4.0902385 0.4806502 2.1958123 0.11888421
## 1735   031980 피에스케이홀딩스  1.0394425 0.2047503 1.3400042 0.16918215
## 1867   012620         원일특강  5.9432836 0.3666667 4.2361702 0.15560766
```

먼저 `apply()` 함수를 이용해 각 열에 해당하는 가치 지표 별 랭킹을 구해준 후, `rowSums()` 함수를 이용해 종목 별 랭킹들의 합을 구해주도록 합니다. 그 후 4개 지표 랭킹의 합 기준 랭킹이 낮은 30 종목을 선택해 줍니다. 즉 하나의 지표 보다 4개 지표가 골고루 낮은 종목을 선택하여 줍니다. 해당 종목들의 티커, 종목명과 가치 지표들을 확인할 수 있습니다.



```r
intersect(KOR_ticker[invest_pbr, '종목명'], KOR_ticker[invest_value, '종목명'])
```

```
##  [1] "한화"             "세아홀딩스"       "넥센"            
##  [4] "한일홀딩스"       "아세아"           "세아제강지주"    
##  [7] "S&T홀딩스"        "무림페이퍼"       "휴스틸"          
## [10] "피에스케이홀딩스"
```

단순 PBR 기준 선택된 종목과 비교해봤을 때, 겹치는 종목이 상당히 줄어들었습니다.