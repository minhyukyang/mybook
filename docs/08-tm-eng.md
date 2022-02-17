# Text Mining (Eng)

## 제인 오스틴(Jane Austen) 작품 분석

제인 오스틴(Jane Austen)이 탈고해 출판한 소설 여섯 개를 [janeaustenr](https://cran.r-project.org/package=janeaustenr) 패키지 에서 가져온 다음 tidy 형식으로 변형해 보자.

-   janeaustenr 패키지는 텍스트를 1줄당 1행(one-row-per-line) 형식으로 제공
-   `mutate()`를 사용해 `linenumber` 수에 해당하는 만큼을 주석으로 처리함으로써 원래 줄 형식을 추적하는데 사용
-   `chapter`를 사용해 모든 장이 어디부터 나오는지 찾아낸다


```r
library(janeaustenr)
library(dplyr)
library(stringr)

original_books <- austen_books() %>%
  group_by(book) %>%
  mutate(linenumber = row_number(),
         chapter = cumsum(str_detect(text, 
                                     regex("^chapter [\\divxlc]",
                                           ignore_case = TRUE)))) %>%
  ungroup()

original_books
## # A tibble: 73,422 x 4
##    text                    book                linenumber chapter
##    <chr>                   <fct>                    <int>   <int>
##  1 "SENSE AND SENSIBILITY" Sense & Sensibility          1       0
##  2 ""                      Sense & Sensibility          2       0
##  3 "by Jane Austen"        Sense & Sensibility          3       0
##  4 ""                      Sense & Sensibility          4       0
##  5 "(1811)"                Sense & Sensibility          5       0
##  6 ""                      Sense & Sensibility          6       0
##  7 ""                      Sense & Sensibility          7       0
##  8 ""                      Sense & Sensibility          8       0
##  9 ""                      Sense & Sensibility          9       0
## 10 "CHAPTER 1"             Sense & Sensibility         10       1
## # ... with 73,412 more rows
```

이것을 tidy 데이터셋으로 사용하려면 `unnest_tokens()` 함수를 사용해 **1행당 1토큰(one-token-per-row)** 형식으로 구성해야 한다.


```r
library(tidytext)
tidy_books <- original_books %>%
  unnest_tokens(word, text)

tidy_books
## # A tibble: 725,055 x 4
##    book                linenumber chapter word       
##    <fct>                    <int>   <int> <chr>      
##  1 Sense & Sensibility          1       0 sense      
##  2 Sense & Sensibility          1       0 and        
##  3 Sense & Sensibility          1       0 sensibility
##  4 Sense & Sensibility          3       0 by         
##  5 Sense & Sensibility          3       0 jane       
##  6 Sense & Sensibility          3       0 austen     
##  7 Sense & Sensibility          5       0 1811       
##  8 Sense & Sensibility         10       1 chapter    
##  9 Sense & Sensibility         10       1 1          
## 10 Sense & Sensibility         13       1 the        
## # ... with 725,045 more rows
```

이 함수는 [tokenizers](https://github.com/ropensci/tokenizers)를 사용해 원래 데이터 프레임에 있는 텍스트의 각 행을 토큰으로 분리한다. 기본 토큰화는 단어에 대한 것이지만 다른 옵션을 사용하면 문자, 엔그램, 문장, 줄, 단락 단위로 토큰호하 할 수 있고, 또는 정규 표혀노식 패턴을 사용해서 분리할 수 있다.

불용어(stop words)는 분석에 유용하지 않은 단어들을 말하며, 일반적으로 영어의 'the', 'of', 'to' 등과 같은 매우 전형적인 단어를 말한다. `anti_join()`을 사용해 불용어를 제거할 수 있다. 이 때, 사용되는 불용어는 `stop_words`를 사용한다.


```r
data(stop_words)

tidy_books <- tidy_books %>%
  anti_join(stop_words)
```

tidytext 패키지의 `stop_words` 데이터셋에는 3개의 불용어 용어집(lexicon)이 들어있다. 지금처럼 모두 함께 사용할 수도 있고, 특정 분석에 더 적합한 경우 `filter()`를 사용해 1개 불용어 집합만 사용할 수도 있다.

또한, dplyr의 `count()`를 사용해 모든 도서에서 가장 흔하게 나오는 단어를 찾을 수 있다.


```r
tidy_books %>%
  count(word, sort = TRUE) 
## # A tibble: 13,914 x 2
##    word       n
##    <chr>  <int>
##  1 miss    1855
##  2 time    1337
##  3 fanny    862
##  4 dear     822
##  5 lady     817
##  6 sir      806
##  7 day      797
##  8 emma     787
##  9 sister   727
## 10 house    699
## # ... with 13,904 more rows
```

단어 카운트(word count) 결과는 tidy data frame에 저장되었기 때문에 아래처럼 ggplot2 패키지로 직접 연결(pipe)할 수 있습니다 (Figure \@ref(fig:plotcount)).


```r
library(ggplot2)

tidy_books %>%
  count(word, sort = TRUE) %>%
  filter(n > 600) %>%
  mutate(word = reorder(word, n)) %>%
  ggplot(aes(n, word)) +
  geom_col() +
  labs(y = NULL)
```

<div class="figure" style="text-align: center">
<img src="08-tm-eng_files/figure-html/plotcount-1.png" alt="The most common words in Jane Austen's novels" width="100%" />
<p class="caption">(\#fig:plotcount)The most common words in Jane Austen's novels</p>
</div>

## Ref.

1.  [janeaustenr](https://cran.r-project.org/package=janeaustenr)
2.  [tokenizers](https://github.com/ropensci/tokenizers)
