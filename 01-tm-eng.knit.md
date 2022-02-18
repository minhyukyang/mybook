# Text Mining (Eng) {#tm-eng}

## ���� ����ƾ(Jane Austen) ��ǰ �м� {#janeaustenr}

> ��ó : R�� ���� �ؽ�Ʈ ���̴�

���� ����ƾ(Jane Austen)�� Ż���� ������ �Ҽ� ���� ���� [janeaustenr](https://cran.r-project.org/package=janeaustenr) ��Ű�� ���� ������ ���� tidy �������� ������ ����.

-   janeaustenr ��Ű���� �ؽ�Ʈ�� 1�ٴ� 1��(one-row-per-line) �������� ����
-   `mutate()`�� ����� `linenumber` ���� �ش��ϴ� ��ŭ�� �ּ����� ó�������ν� ���� �� ������ �����ϴµ� ���
-   `chapter`�� ����� ��� ���� ������ �������� ã�Ƴ���


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
#> # A tibble: 73,422 x 4
#>    text                    book                linenumber chapter
#>    <chr>                   <fct>                    <int>   <int>
#>  1 "SENSE AND SENSIBILITY" Sense & Sensibility          1       0
#>  2 ""                      Sense & Sensibility          2       0
#>  3 "by Jane Austen"        Sense & Sensibility          3       0
#>  4 ""                      Sense & Sensibility          4       0
#>  5 "(1811)"                Sense & Sensibility          5       0
#>  6 ""                      Sense & Sensibility          6       0
#>  7 ""                      Sense & Sensibility          7       0
#>  8 ""                      Sense & Sensibility          8       0
#>  9 ""                      Sense & Sensibility          9       0
#> 10 "CHAPTER 1"             Sense & Sensibility         10       1
#> # ... with 73,412 more rows
```

�̰��� tidy �����ͼ����� ����Ϸ��� `unnest_tokens()` �Լ��� ����� **1��� 1��ū(one-token-per-row)** �������� �����ؾ� �Ѵ�.


```r
library(tidytext)
tidy_books <- original_books %>%
  unnest_tokens(word, text)

tidy_books
#> # A tibble: 725,055 x 4
#>    book                linenumber chapter word       
#>    <fct>                    <int>   <int> <chr>      
#>  1 Sense & Sensibility          1       0 sense      
#>  2 Sense & Sensibility          1       0 and        
#>  3 Sense & Sensibility          1       0 sensibility
#>  4 Sense & Sensibility          3       0 by         
#>  5 Sense & Sensibility          3       0 jane       
#>  6 Sense & Sensibility          3       0 austen     
#>  7 Sense & Sensibility          5       0 1811       
#>  8 Sense & Sensibility         10       1 chapter    
#>  9 Sense & Sensibility         10       1 1          
#> 10 Sense & Sensibility         13       1 the        
#> # ... with 725,045 more rows
```

�� �Լ��� [tokenizers](https://github.com/ropensci/tokenizers)�� ����� ���� ������ �����ӿ� �ִ� �ؽ�Ʈ�� �� ���� ��ū���� �и��Ѵ�. �⺻ ��ūȭ�� �ܾ ���� �������� �ٸ� �ɼ��� ����ϸ� ����, ���׷�, ����, ��, �ܶ� ������ ��ūȣ�� �� �� �ְ�, �Ǵ� ���� ǥ����� ������ ����ؼ� �и��� �� �ִ�.

�ҿ��(stop words)�� �м��� �������� ���� �ܾ���� ���ϸ�, �Ϲ������� ������ 'the', 'of', 'to' ��� ���� �ſ� �������� �ܾ ���Ѵ�. `anti_join()`�� ����� �ҿ� ������ �� �ִ�. �� ��, ���Ǵ� �ҿ��� `stop_words`�� ����Ѵ�.


```r
data(stop_words)

tidy_books <- tidy_books %>%
  anti_join(stop_words)
```

tidytext ��Ű���� `stop_words` �����ͼ¿��� 3���� �ҿ�� �����(lexicon)�� ����ִ�. ����ó�� ��� �Բ� ����� ���� �ְ�, Ư�� �м��� �� ������ ��� `filter()`�� ����� 1�� �ҿ�� ���ո� ����� ���� �ִ�.

����, dplyr�� `count()`�� ����� ��� �������� ���� ���ϰ� ������ �ܾ ã�� �� �ִ�.


```r
tidy_books %>%
  count(word, sort = TRUE) 
#> # A tibble: 13,914 x 2
#>    word       n
#>    <chr>  <int>
#>  1 miss    1855
#>  2 time    1337
#>  3 fanny    862
#>  4 dear     822
#>  5 lady     817
#>  6 sir      806
#>  7 day      797
#>  8 emma     787
#>  9 sister   727
#> 10 house    699
#> # ... with 13,904 more rows
```

�ܾ� ī��Ʈ(word count) ����� tidy data frame�� ����Ǿ��� ������ �Ʒ�ó�� ggplot2 ��Ű���� ���� ����(pipe)�� �� �ֽ��ϴ� (Figure \@ref(fig:plotcount)).


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
<img src="01-tm-eng_files/figure-html/plotcount-1.png" alt="The most common words in Jane Austen's novels" width="100%" />
<p class="caption">(\#fig:plotcount)The most common words in Jane Austen's novels</p>
</div>

## gutenbergr ��Ű�� {#gutenbergr}

[gutenbergr](https://github.com/ropensci/gutenbergr) ��Ű���� ���ٺ���ũ ������Ʈ ������ �� ���� ���۹��� �ش��ϴ� �ؽ�Ʈ�� ������ �� �ְ� �Ѵ�. �� ��Ű������ ������ �����ޱ� ���� ������ �����ִ� ��ǰ�� ã�µ� ����� �� �ִ� ���ٺ���ũ ������Ʈ ��Ÿ�������� ��ü �����ͼ��� ���ԵǾ� �ִ�. 

> gutenbergr�� ���� �ڼ��� ������ rOpenSci�� ��Ű�� Ʃ�丮��(https://ropensci.org/tutorials/gutenbergr_tutorial/)�� ��������. 

### �ܾ� ��

���� 19���� ������ 20���� �ʹݿ� ���� ��Ҵ� ������ ���� ���� �Ҽ��� ��Ÿ�� �Ҽ��� ���캸��.

- [*The Time Machine*](https://www.gutenberg.org/ebooks/35)
- [*The War of the Worlds*](https://www.gutenberg.org/ebooks/36)
- [*The Invisible Man*](https://www.gutenberg.org/ebooks/5230)
- [*The Island of Doctor Moreau*](https://www.gutenberg.org/ebooks/159)

�츮�� `gutenberg_download()`�� �� �Ҽ��� ���� ���ٺ���ũ ������Ʈ �ĺ� ��ȣ�� ����� �̷��� ��ǰ�� �׼����� �� �ִ�.


```r
library(gutenbergr)

hgwells <- gutenberg_download(c(35, 36, 5230, 159))

tidy_hgwells <- hgwells %>%
  unnest_tokens(word, text) %>%
  anti_join(stop_words)
```

��� ��Ƽ� ������ �Ҽ��� ���� ���������� ������ �ܾ�� �������� �˾ƺ���.













