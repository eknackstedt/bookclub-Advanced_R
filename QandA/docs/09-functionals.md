# Functionals



## 9.2.2 Anonymous functions and shortcuts {-}

:::question
We saw you'll get an error if you try to map over elements that don't exist, and can use `.default` to override that. Is this related to `tryCatch` in some way? Can we look at the `map` source code for `.default?` And how would we overcome this error if we were to use base R's `lapply(x, 'two')`


```r
x <- list(
  list(one = "a"),
  list(two = "b"),
  list(three = "c")
)

map_chr(x, 'two', .default = NA)
```

```
## [1] NA  "b" NA
```


```r
lapply(x, 'two')
```

```
Error in get(as.character(FUN), mode = "function", envir = envir) : object 'two' of mode 'function' was not found
```
:::


Within `purrr` [there's the function](https://github.com/tidyverse/purrr/blob/7104367bb6599f13e56c554bd07488b508a8a02b/R/as_mapper.R#L98)


```r
find_extract_default <- function(.null, .default) {
  if (!missing(.null)) {
    .null
  } else {
    .default
  }
}
```

So it doesn't seem to be a conditional, but rather a way to deal with missing errors


```r
as.character(lapply(x,`[[`,"two"))
```

```
[1] "NULL" "b" "NULL"
```

## 9.2.6.4 Exercise {-}

:::question
In order to extract p-values the solution manual suggests using `map_dbl` but could we use `pluck` to get these values? If so how?
:::


```r
trials <- map(1:100, ~ t.test(rpois(10, 10), rpois(10, 7)))
# tibble(p_value = map_dbl(trials, "p.value"))
map_dbl(trials, pluck, "p.value")
```

```
##   [1] 0.5303040009 0.0089505597 0.0417395883 0.0002164856 0.0265181485
##   [6] 0.0003766042 0.1557962844 0.0004438139 0.0140118774 0.0756930919
##  [11] 0.2803615641 0.0296724624 0.1164730449 0.0599248621 0.0067572903
##  [16] 0.0067204669 0.1884308249 0.0204817791 0.1858304087 0.0007341464
##  [21] 0.0059336018 0.0030878384 0.0868280220 0.0692107871 0.0100640807
##  [26] 0.0013719757 0.0346302420 0.0616181804 0.1076703503 0.0798481448
##  [31] 0.1270752080 0.0470122520 0.1428425682 0.0183566478 0.0109865454
##  [36] 0.0033693138 0.0856473557 0.0018947902 0.0004124945 0.0591705668
##  [41] 0.1010933339 0.0615770981 0.5123538257 0.0128426323 0.1100174145
##  [46] 0.2303570453 0.0716510442 0.0265059965 0.0036540066 0.2526815206
##  [51] 0.0245142249 0.0568365010 0.0024965722 0.1831153692 0.2109461881
##  [56] 0.0696605202 0.0157259104 0.1325773677 0.2410838727 0.0107111942
##  [61] 0.0822798720 0.0007155678 0.0284080266 0.0066456976 0.2725240109
##  [66] 0.8774008687 0.0070823949 0.0010285184 0.0180156155 0.1346942246
##  [71] 0.4217825111 0.0089601656 0.2966475579 0.5316716956 0.1389844265
##  [76] 0.0026130004 0.0218038956 0.0087392086 0.0107435678 0.0293522064
##  [81] 0.0148177702 0.0135627500 0.0081010222 0.0014830284 0.1621635457
##  [86] 0.0177739525 0.0007174240 0.0092903172 0.0090750124 0.0017805213
##  [91] 0.0646776173 0.0701538398 0.0066414190 0.0832948516 0.2201989567
##  [96] 0.2204768059 0.0010181683 0.0050194865 0.1041031924 0.0284673301
```

## 9.2.6.5 Exercise {-}

:::question
Can we make this work?


```r
x <- list(
  list(1, c(3, 9)),
  list(c(3, 6), 7, c(4, 7, 6))
)

triple <- function(x) x * 3
# map(x, map, .f = triple)
```
:::

Specifying `.f` there replaces the second argument of the top level map function, so you'd be doing `map(.x = x, .f = triple, map)` which is not what you mean. What you want here is:


```r
map_depth(x, 2, triple)
```

```
## [[1]]
## [[1]][[1]]
## [1] 3
## 
## [[1]][[2]]
## [1]  9 27
## 
## 
## [[2]]
## [[2]][[1]]
## [1]  9 18
## 
## [[2]][[2]]
## [1] 21
## 
## [[2]][[3]]
## [1] 12 21 18
```

## 9.4.1 Same type of output as input: `modify()` {-}

When using modify we now get the warning:`Warning message: `modify()` is deprecated as of rlang 0.4.0. Vector tools are now out of scope for rlang to make it a more focused package.`

How should we rewrite this example?


```r
data.frame(
  x = 1:3,
  y = 6:4
) %>% 
  modify( ~ .x * 2)
```
:::

`rlang::modify` != `purrr::modify()`

## 9.4.5 Any number of inputs: pmap() and friends {-}

:::question
I want to use pmap to map over a vector, but the metadata for the functions other arguments is elsewhere - how would I combine these using `pmap`? The book says to name the metadata columns the same name as your function which I did but this still doesn't work?
:::


```r
my_data <- 1:3

metadata <- tribble(
  ~id, ~mult, ~adder,
  "one",   2,      5,
  "two",   3,      6,
  "three", 4,      7
)

the_function <- function(vec, id, mult, adder) {
  glue::glue("{id} is now {vec * mult + adder}")
}

# my_data doesn't change but we want to map over the metadata
# x = string
# y = multiplier
# z = adder
# pmap(list(metadata), ~the_function(vec = my_data))
```


```r
pmap(c(list(my_data), metadata), the_function)
```

```
## [[1]]
## one is now 7
## 
## [[2]]
## two is now 12
## 
## [[3]]
## three is now 19
```

## 9.4.6.2 Exercise {-}

:::question
I see how we can use `iwalk` and `walk2` for writing to multiple files, but the question asks about disadvantages to this - what would they be?
:::


```r
cyls <- split(mtcars, mtcars$cyl)
paths <- file.path(temp, paste0("cyl-", names(cyls), ".csv"))
walk2(cyls, paths, write.csv)

mtcars %>% 
  split(mtcars$cyl) %>% 
  set_names(~ file.path(temp, paste0("cyl-", .x, ".csv"))) %>% 
  iwalk(~ write.csv(.x, .y))
```

A readability problem mostly, as well as implicitly using `names(cyls)` as opposed to explicitly like it's used in paths


## 9.5.4 Multiple inputs {-}

:::question
Can we think of a simple example for `reduce()`?
:::

A Fibonacci function! 


```r
n <- 10
purrr::accumulate( .init = c(0L,1L),            # Starting with (0,1)
                   rep(0,n),                    # Accumulate n times
                   ~c(.x,sum(.x))[2:3]          # (x,y) -> (x, y, x+y)[2:3]
                 ) %>% 
    purrr::map_int( `[`, 1 ) 
```

```
##  [1]  0  1  1  2  3  5  8 13 21 34 55
```


## 9.6.3.2 Exercise {-}

:::question
I understand that `if (length(x) == 1L) return(x[[1L]])` covers the case like `simple_reduce(1, +)` but what's the deal with `if (length(x) == 0L) return(default)` and what exactly is `default`?
:::


```r
simple_reduce <- function(x, f, default) {
  # when would you use reduce on something length 0?
  if (length(x) == 0L) return(default)
  if (length(x) == 1L) return(x[[1L]])

  out <- x[[1]]
  for (i in seq(2, length(x))) {
    out <- f(out, x[[i]])
  }
  out
}
```

Default is the user supplied second number to add by, and we can use `integer(0)` to work with a `length(0)` vector.


```r
simple_reduce(integer(0), `+`, default = 0L)
```

```
## [1] 0
```

You would want to make sure your reduce is covered in the case of length zero so that your program doesn't crash when a user passes in an empty vector.

## 9.7.1 Matrices and arrays {-}

:::question
How would you tidyverse the `rowSums` function?


```r
x <- tribble(
  ~x, ~y,
  1, 1,
  2, 2,
  3, 3
)

apply(x, 1, function(x) sum(x>2))
```

```
## [1] 0 0 2
```
:::



```r
pmap_dbl(x, ~sum(c(...) > 2))
```

```
## [1] 0 0 2
```


:::question
I'm not quite sure what idempotent means but Hadley warns that `a2d` and `a1` aren't the same, but isn't that just because we're using `1` which is row wise, and not `2`? What is the warning he is heeding us against here?
:::


```r
a2d <- matrix(1:20, nrow = 5)
a1 <- apply(a2d, 1, identity)
identical(a2d, a1)
```

```
## [1] FALSE
```


I think he is just trying to drive home the idea that you might expect no change by apply-ing an operator like identity which leaves its input unchanged, however what it instead returns is the transpose.  So perhaps it would have been better to more specifically say that `apply(..., MARGIN = 1)` isn't idempotent.

## 9.7.3.2 Exercise {-}

:::question
What's an example of using `eapply` (iterates over the (named) elements of an environment)?
:::


```r
baseenv() %>% eapply(is.primitive) %>% unlist %>% which %>% names
```

:::question
Can we come up with an example for `rapply` allowing us to apply a function to only a specified class? Does something like this exist within `purrr`?
:::

Answer from [this community post](https://community.rstudio.com/t/recursive-purrr-mutate-if/16638/7):

#### rapply


```r
x <- list(list(a = as.character(1), b = as.double(2)), 
          c = as.character(3), 
          d = as.double(4))

as_integer_rapply <- function(x) {
  rapply(
    x,
    as.integer,
    "character",
    how = "replace"
  )
}

x %>% as_integer_rapply() %>% str()
```

```
## List of 3
##  $  :List of 2
##   ..$ a: int 1
##   ..$ b: num 2
##  $ c: int 3
##  $ d: num 4
```

#### map


```r
as_integer_recursive_map <- function(x) {
  x %>%
    map_if(is.character, as.integer) %>%
    map_if(is.list, as_integer_recursive_map)
}
x %>% as_integer_recursive_map() %>% str()
```

```
## List of 3
##  $  :List of 2
##   ..$ a: int 1
##   ..$ b: num 2
##  $ c: int 3
##  $ d: num 4
```

#### Comparing the two:


```r
microbenchmark::microbenchmark(
  as_integer_rapply(x),
  as_integer_recursive_map(x),
  times = 100
)
```

```
## Unit: microseconds
##                         expr     min       lq       mean    median       uq
##         as_integer_rapply(x)   9.018   12.672   32.60612   18.8695   20.824
##  as_integer_recursive_map(x) 982.118 1008.416 1092.26688 1060.1575 1114.135
##       max neval
##  1474.005   100
##  1678.852   100
```



```r
as_integer_safe <- function(y) {
  if (is.character(y) && all(grepl("^\\d+$", y))) {
    as.integer(y)
  } else {
    y
  }
}

x[[1]][[3]] <- "dog"

rapply(x, f = as_integer_safe, how = "replace")
```

```
## [[1]]
## [[1]]$a
## [1] 1
## 
## [[1]]$b
## [1] 2
## 
## [[1]][[3]]
## [1] "dog"
## 
## 
## $c
## [1] 3
## 
## $d
## [1] 4
```


