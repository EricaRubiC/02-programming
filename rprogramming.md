Programming in R
================
Austin Hart

- [Workflow and projects](#workflow-and-projects)
- [Packages](#packages)
- [Function basics](#function-basics)
- [Data I/O](#data-io)
- [Explore](#explore)
- [Understanding R objects](#understanding-r-objects)
- [Pipes](#pipes)

Dive into the world of data analysis projects using R. For those with a
background in R, use this as an opportunity to get new perspective on
familiar functions, object structures, and more. For those without the
background, try to think about the operations below in terms of syntax
and function rather than as a fixed set of routines to memorize.

Note that I use generic object and element names wherever possible. For
example, `summary(mydata$var1)` refers to a generic object, `mydata`
that contains a column named `var1`.

## Workflow and projects

Data analytic work has a life cycle that goes well beyond how we think
about class notes or a single problem set. I really like Hadley et al.’s
depiction of the data science project:

<figure>
<img
src="https://github.com/SIS-data-analysis/02-programming/blob/main/base.png"
alt="Hadley et al, R for Data Science" />
<figcaption aria-hidden="true">Hadley et al, R for Data
Science</figcaption>
</figure>

 

You need a dedicated project folder to manage the many inputs and
outputs that come with this cycle. The project (`.Rproj`) and associated
folder create a home where you can pull and push formatted data, code,
and figures. The project folder is also the place for non-data
documentation and resources. This might include codebooks,
presentations, or notes. A generic project folder for a problem set in
our course might include:

- `PS1.Rproj`: A configuration file for managing an RStudio project.
  This file sets the working directory automatically and stores project
  settings (e.g., git integration).
- `PS1data.csv`: the data  
- `PS1code.R`: raw R code
- `HartPS1.rmd`: Markdown document with your final code and answers  
- `HartPS1.pdf`: Final, typeset product

To that end, I recommend that you create a new project
(`File -> New Project`) or fork and clone the repository for each class
session and assignment.

## Packages

The world of R grows and evolves through user-written packages (similar
to how “apps” add functionality to your phone). After you install a
package, you can use it by attaching its library of commands for the
whole session or calling specific operations on the fly using the `::`
operator.[^1]

``` r
# Installation: a one-time deal
  install.packages('new_package') # downloads from R server and installs

# attaching a library: for a new R session
  library(new_package)
  
# accessing commands on the fly: new_package::whizbang
  mydf = haven::read_dta('stataData.dta')
  
# get help
  ?package_name
  ?mystery_function
```

## Function basics

R packages contain a library of functions. A function is a pre-defined
routine to perform some task. For example, the command `count` calls a
`dplyr` function that tallies the unique values that appear in one or
more variables of a data frame or vector.

Functions have three main parts:

- Name: how to call the function into action (e.g., `count` or
  `summary`).  
- Arguments: the inputs required for the function to work. For example,
  `kable` requires an object that you want to format as a table:
  `knitr::kable(x)` where `x` is your data frame.  
- Body: the code that defines what the function does.

``` r
# Example of a custom function
  add_nums <- function(a, b) {
    sumtot <- a + b
    return(sumtot)
  }

# Use the command to call the function
  add_nums(3, 5) # Output: 8
  mysum <- add_nums(3, 5) # stores output as an object
```

## Data I/O

How do you input data in R? R can read in an incredible array of data
types. Simply identify the type of data you have (look at the file
extension!) and choose the right command. *Take special note to wrap all
file names in quotes*

``` r
# Import an R object (data, list, etc)
  load('file.RData')
  
# Other formats (must input as a new object)
  df1 <- read_csv(file = 'file.csv', na = c("",".")) # note the na option
  df2 <- readxl::read_excel(file = 'file.xlsx')
  df3 <- haven::read_dta(file = 'file.dta')
```

Don’t forget about outputting, or saving, the data too. Write it to your
preferred format. I recommend saving in `.RData` format for yourself and
`.csv` format if you intend to distribute the data for public use.[^2]

``` r
# Output
  save(df1, 'newdata.RData')
  write_csv(df1, 'newdata.csv')
```

## Explore

Get those hands dirty! Check the data structure. Take a peak at the raw
data. Get a quick summary of each variable in the data.

``` r
# Structure and head
  str(mydata)
  head(mydata)
  
# Explore variables
  summary(mydata)
```

### Character and factor variables

What to do with non-numeric variables? Start with a relative frequency
table and bar chart.

``` r
# Initial Tabulation
  mytab <- 
    mydata %>%
    count(var_name) %>%
    mutate(percent = 100 * n / sum(n))

# Beautify the table for printing
  mytab %>%
    knitr::kable(digits = 1) # optional last step

# Graphing
  mytab %>% # uses the table you created, not raw data
    ggplot(mytab, aes(x = var.name, y = percent)) +
    geom_col()
```

### Numeric and integer variables

Get those summary statistics and make a picture. Notice the flexibility
of `summarise` versus a quicker but inflexible call to `summary`.

``` r
# Summary stats
  summary(data['var']) # or summary(data$var)
  
  data %>%
    summarise(
      stat1 <- some_function(var1),
      mean2 <- mean(var2),
      sd2 <- sd(var2),
      meanDiff <- mean(var1) - mean(var1),
    )

# Plot the distribution    
  mydata %>%
    ggplot(aes(x = var.name)) +
    geom_histogram()
```

## Understanding R objects

R is an Object Oriented Language (OOL), and objects hold primary
importance in the way we write and execute code. If you want to store
information/data in memory (rather than just print it out in the
console), you have to store it in a named (or assigned) object. Note
that `<-` is called the assignment operator. If you want to call on or
use stored information, you have to call the object by name. As a matter
of style: keep names short, intuitive, and free of spaces.[^3]

``` r
# Compare:
  1:5
  obj1 <- 1:5
  obj1
```

Objects come in many flavors: vectors, matrices, data frames, lists, and
more. Object names are case sensitive. We’ll typically work with data
frames (and a special kind of frame called a `tibble`), but there are
moments where knowing the difference between a data frame and a vector
is really important.

``` r
# Vectors/values:
  a <- 1:10
  b <- rnorm(10)
  c <- letters[1:10]
  d <- a >= 4

# Data frames/tibbles
  df1 <- data.frame(a, b, c, d)
  df2 <- tibble(d, a, b) # requires tidyverse

# Lists
  l1 <- list(
    item1 = df1, 
    item2 = df2
  )
```

Run the code in the blob above and then check the properties of each
object you created using `is()`. For example, note that `is(l1)` returns
`list` and `vector`.

How do you index or extract information stored in an object? For
example, you want to calculate the mean of `var1` within `mydata`. R is
very rectangular in its thinking about objects: they are `n` rows long
and/or `k` columns wide.[^4] A `tibble` is an `n x k` rectangular
structure. A `tibble` recording grades for fifteen students on four
problem sets is a 15x4 rectangular object. A `list` of 2 `tibbles` and a
`vector` is a 3x1 list. If you know this, you can access information
with reference to it’s location in the rectangle. There are three main
indexing operators:

- `[` subsets an object and returns an object of the same class.  
- `[[` *extracts* elements of list or object
- `$` *extracts* elements by literal name

Notice the difference between extraction and subsetting. Pay special
attention to the different use of quotations for variable names as well
as the type of object each operation creates.

``` r
# Extract an item from list or frame
  object$element
  object[['element']]
  
# Subset an element (maintains orig structure)
  object[1:30, 4:5] # object[rows, columns]; defaults to object[column]
  object['var_name'] # use quotes to call by name
  object %>%
    select(var1, var2, var3) %>%
    filter(criterion)
```

## Pipes

Pipes are awesome. The base R pipe, `|>`, and `magrittr` pipe, `%>%`,
push the output from one code segment forward as the first input for the
next function. It saves you from creating and tracking loads of
intermediate objects. Compare:

``` r
# with pipes
  mydata %>% # start with my data
    filter(group == 'A') %>%
    count(var1) %>% 
    mutate(percent = 100 * n / sum(n)) %>% # add a percent column
    ggplot(aes(x = var1, y = percent)) +
    geom_col()

# Without a pipe
  d1 <- filter(.data = mydata, group == 'A')
  d2 <- count(x = d1, var1)
  d3 <- mutate(.data = d2, percent = 100 * n / sum(n))
  ggplot(data = d3, aes(x = var1, y = percent)) + 
    geom_col()
    
  rm(d1,d2,d3)
```

Notice where the “piped” output from the first or left-hand side of the
chunk fits into the next (righ-hand side) code segment. For example,
`count(x = mydata, var1)` becomes `mydata %>% count(x = ., var1)` where
`mydata` flows directly into `.`, an explicit placeholder for the input
from a prior operation. You can simplify to `mydata %>% count(var1)`.

[^1]: Attaching is great for packages you want to use repeatedly (e.g.,
    `tidyverse`). The `::` operator is excellent for something you may
    only use once (e.g., `haven::read_dta()`).

[^2]: One important caveat is that writing to `.csv` will lose any
    factor ordering and/or `haven` labeling that you created.

[^3]: Strictly speaking, you can assign any name to an object. However,
    if it has spaces or other special/illegal characters, you’ll have to
    wrap it in backquotes, `` ` ``, every time you call it:
    `` `$hi++y Name` ``.

[^4]: Object rows are identifiable by number. Columns can be identified
    with reference to name, `var1`, or number.
