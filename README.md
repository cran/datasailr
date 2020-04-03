# About DataSailr

DataSailr is an R package which enables row by row data manipulation. 

The data manipulation process is written in Sailr, a language designed specifically for data manipulation. The internal data manipulation engine is a library called libsailr, which is written in C/C++ and the processing speed is relatively fast. If you are interested in DataSailr's internal implementation, please also have a look at libsailr library. 

This package is still in development. Please let me know if you find bugs or inconsistent behaviors. The documentation is not enough, so I am willing to explain it if you find something difficult.


## Description & Motivation

DataSailr package brings intuitive and fast row by row data manipulation to R. The data manipulation instruction for each row is writtein in Sailr. Sailr is an easy script language designed specially for data manipulation. In vanilla R, dataframe is manipulated using column vector and vector operations. When summarizing dataframe, column wise calculation is intuitive and ideal, but when manipulating dataframe you need to see each record in the row direction. For example, when calculating body mass index (BMI) from body weight and height, calculation needs to be done for each row. Categorizing each person based on his/her BMI is also done for each row.

A famous R package, dplyr, has been improving the same kind of points. It enables data manipulation without thinking much about column vectors. Pipe operator, %>% in magrittr package, and dplyr functions realize intuitive data manipulation flow. The DataSailr package enables the same kind of thing with a single Sailr code. The two packages do not compete, and I intend to implement DataSailr as it also can work with dplyr.


## How to install

### Binaries

Binaries are provided at "http://datasailr.io/download"

Especially for windows users, it is recommended to install from binary package.


### Install from source

* Linux(ubuntu)

For compilation, autotools, bison, flex and some other UNIX programs are required. 

```
git clone <datasailr repository>
cd datasailr/src
./update_3rd_party_library.sh  # This downloads Onigmo library.
cd ..
autoconf  # Generate configure script.
cd ..

R CMD INSTALL datasailr --preclean --no-multiarch --build
```



## How to run Sailr script

datasailr::sail() is the main function of this package. This function takes data.frame as the 1st argument and Sailr script as the 2nd argument. (Note that the 1st argument is data.frame, so you can use this package with %>% operator (in magrittr package), and combine this with functions of dplyr.)

```
library(datasailr)

datasailr::sail( df, '
<Sailr script>
'
)
```


## Quick introduction

In this quick introduction, built-in R dataframe called mtcars is going to be used. The mtcars datafram has automobile information in 1970's. The detail of this dataframe is described here. (https://www.rdocumentation.org/packages/datasets/versions/3.6.1/topics/mtcars)

* Built-in dataframe, mtcars.

```
data(mtcars)
head(mtcars,2)
```

(OUTPUT)

|              | mpg | cyl | disp | hp  | drat |   wt  | qsec  |vs  | am |gear |carb
---------------|-----|-----|------|-----|------|-------|-------|----|----|-----|-----
|Mazda RX4     | 21  | 6   | 160  | 110 | 3.9  | 2.620 | 16.46 | 0  | 1  | 4   | 4
|Mazda RX4 Wag | 21  | 6   | 160  | 110 | 3.9  | 2.875 | 17.02 | 0  | 1  | 4   | 4


### Code example

Before executing the following code, install DataSailr package (and magrittr package.) Installation instruction is mentioned later in this README (Binary package is the easiest way to try, until this package becomes available on CRAN).

* (Example)

```
library(datasailr)
library(magrittr)

mtcars %>% datasailr::sail('
if( hp > 145 ){  power = "high" }
else if( 145 >= hp && hp > 0){ power = "low" }
else{  print("hp variable has missing value")}

germany = re/(^Merc|^Porsche|^Volvo)/
usa = re/(^Hornet|^Cadillac|^Lincoln|^Chrysler|^Dodge|^AMC|^Camaro|^Chevrolet|^Pontiac|^Ford)/
japan = re/(^Mazda|^Datsun|^Honda|^Toyota)/

if ( _rowname_ =~ germany ) { country = "Germany" ; type = rexp_matched(1); }
else if( _rowname_ =~ usa ) { country = "USA"  ; type = rexp_matched(1);  }
else if( _rowname_ =~ japan ) { country = "Japan"  ; type = rexp_matched(1); }
else { country = "Other" }
') %>% head(10)
```


* (OUTPUT)

	
|   | power | country | type
----|-------|---------|-----
| 1 | low   | Japan   | Mazda	
| 2 | low   | Japan   | Mazda	
| 3 | low   | Japan   | Datsun	
| 4 | low   | USA     | Hornet	
| 5 | high  | USA     | Hornet	
| 6 | low   | Other   |
| 7 | high  | Other   |
| 8 | low   | Germany | Merc	
| 9 | low   | Germany | Merc	
| 10 | low  | Germany | Merc


### Example code explanation

* library(datasailr)
    + Make DataSailr package available.
* library(magrittr)
    + This is not required by DataSailr. In the above code, %>% , pipe operator, is used, and it's defined in magrittr package.
* mtcars %>% datasailr::sail(' .... 
    + The 1st argument of datasailr::sail is dataframe. In this case, %>% passes dataframe to the 1st argument.
    + The 2nd argument of datasailr::sail is Sailr code.
* if( hp > 145 ){ ....
    + if-else statement. 
* power = "high" 
    + assing "high" to power column.
* germany = re/(^Merc|^Porsche|^Volvo)/
    + Regular expression literal is re/pattern/ .
* if ( _rowname_ =~ germany ){ ....
    + string =~ regular expression executes regular expression matching.
    + _rowname_ column is one of automatic varaibles. This holds the row names of the original R dataframe.
* rexp_matched(1)
    + Calling function of rexp_matched()
    + Extract the 1st matching group from the last regular expression matching.



## Grammar of Sailr

Sailr is specially intended for use of data manipulation. From the programming language view, the functionality is not enough and is not a general-purpose programming language, but you can write instructions and manipulate data in an intuitive way. For SAS(*) software users, the grammer of Sailr is relatively easy to understand. Differences are mentioned later in this README.

(*) SAS is the world's largest privately held software company. SAS provides SAS softwares which are used by professional data analysis users.


### [IMPORTANT] Variables & Assignment operation [MUST READ]

* Variables in Sailr and variables in statistics

In general-purpose programming language, variables usually mean identifiers to point to some memory or some objects. However in statistics, dataset column names are usually called variables. Sailr follows the statistics way because it is designed not for general-purpose programming language, but is designed for processing dataset. Variables in codes correspond to the column with the same name. 

(e.g.) The following code print the value of column X.

```
print(X)
```

(e.g.) The following code assign the value of column X to column Y.

```
Y = X
```


### Types

Sailr can deal with the following types, and so does DataSailr.

1. Int
2. Double
    + Int and Double are interchangeable when needed.
3. String
    + String literal 
4. Boolean 
    + Only available within if-else conditions
5. Regular expression
    + Currently only regular expression literal in Sailr script is accepted.
    + At now, there is no such way to dynamically generate regular expression object.


R can handle vectors, list, S4 class objects, and so on. Among those R objects, dataframe usually consists of only vectors. The following list shows which data types are supported by DataSailr. Supporting vectors of numbers and strings is enough in most of the cases of data analysis. 

* logical vector : DataSailr imports TRUE as 1, and FALSE as 0.
* integer vector : DataSailr deals with it as integer.
* numeric vector : DataSailr deals with it as double.
* complex vector : Not supported
* character vector : DataSailr deals with it as string.
* factor : DataSailr deals with it as string.
    + Precisely, factor is a type called S3 class. S3 class is an R type with "class" attribute and some additional attributes (such as levels) required by the class. Internally factor is just an integer vector with these attributes. Attribute of "levels" is assumed to be CharacterVector in the current implementaion.
* list : not supported.
* S3 class objects : As mentioned in factor, the base type of S3 is a basic R type. If the basic R type is supported by DataSailr, the S3 type can be dealt from DataSailr. However, DataSailr's interpretation is not as the user's intention, but follows the internal data type. (i.e.) Factor is recognized as an integer vector.
* S4 class objects : Not supported
    + S4 objects are not supported. Those objects need to be converted to DataSailr types beforehand. Moreover, S4 objects are only stored in list, and list will not be supported in any case.


| Column of R data.frame |  DataSailr    | 
-------------------------|-----------------
| Integer Vector         | Int           |
| Double (Real) Vector   | Double        | 
| Character Vector       | String        |
| Boolean Vector         | Int (0/1)     |
| Factor                 | String        |
| List (No plan)         | No plan to be supported |


### Arithmetic operators


* Supported

The following operators can be applied to numbers (int + double).

Only plus (+) operator can deal with strings. It can concatenate strings. 

```
+    add
-    subtract
*    multiply
/    divide
**   power
^    power
=~   regular expression matching (  )
```

* Not supported

The following operators are not supported in DataSailr

```
++  // Increment operator
--  // Decrement operator
+=
-=
*=
/=
```


### if-else statement

The following syntax is accepted. The former one is more familiar to programmers, though the latter is easier to read and may be better for data scientists.

```
if(bmi >= 40){
    weight_level = .
}else if( bmi >= 35 ){ 
    weight_level = .
}else if( bmi >= 30 ){
    weight_level = 3 
}else if( bmi >= 25 ){
    weight_level = 2
}else if( bmi >= 20 ){
    weight_level = 1
}else{
    weight_level = .
}
```

```
if(bmi >= 40){ weight_level = . } 
else if( bmi >= 35 ){ weight_level = . } 
else if( bmi >= 30 ){ weight_level = 3 } 
else if( bmi >= 25 ){ weight_level = 2 }
else if( bmi >= 20 ){ weight_level = 1 }
else { weight_level = . }
```


### Built-in functions

The following is the list of built-in functions.

```
print( str1, str2 ... )
num_to_str( num ) 
str_strip( str )
str_lstrip( str ) 
str_rstrip( str )
str_concat( str1, str2 ... ) // numbers are also accepted as input.
str_repeat( str , num )
str_subset( str, num, num )  // index starts from one. UTF8 compatible.
str_to_num( str )
rexp_matched( num )  // back reference
date_ymd( year, month, day )  // convert a combination of year, month and day into numbers that is a day from Jan 1st 1970.
date_ym_weekday_nth( year, month, weekday, nth )  // weekday should be "Sun", "Mon" ... 
date_add_n_years( unix_date, years )
date_add_n_months( unix_date, months )
date_add_n_days( unix_date, days )
date_format( unix_date, str_format )  // This format should follow C++'s std::chrono::format. "%m/%d/%y" or "%Y-%m-%d" is popular.
```

Currently, users cannot implement functions in Sailr by themselves. 
If you want to implement your own function, you need to modify libsailr and write code in C.
The libsailr C API is not mature. The interface for users is not organized. Please feel free to contact the author if you want your own function.



### Support for special numbers

#### Missing value support

For numbers (integer or double), single dot (.) represents missing value in code. Internally missing value is defined as -nan in C (which is generated by sqrt(-1)). See libsailr's node.c file and new_node_nan_double () function. ) Any calculation with this value always results in -nan.

For string, empty string ("") represents missing string.



#### Missing value (-nan) for if-condition

Note that in vanilla R, calculation with missing value returns missing value, and missing values act like TRUE in logical vector. In vanilla R, filtering of vectors and dataframes is done using logical vectors. If missing values are treated like TRUE, unintended elements or rows that have missing values are extracted. For example, you are calculating BMI (body mass index) from body weight and height. One person has a missing for body weight. His/her BMI will be calculated as missing value. You would like to categorize this person as "unknown BMI" category, then missingness should be checked first and these people should be categorized first. Otherwise, missing values in logical vector works like TRUE, so these people are categorized into some BMI categories, such as "high BMI" or "low BMI".

In DataSailr, on the other hand, comparison operators with missing values return false except that missing is compared with missing.

```
library(datasailr)

col1 = c(1.7, 1.5 , 1.5 , 1.65 ,  1.5)
col2 = c(50 , 60  , 80  , NA   ,  100)
df = data.frame(height = col1, bw = col2)

code ='
bmi = bw / (height ^ 2)

if(bmi >= 40){ overweight = 5 } 
else if( bmi >= 35 ){ overweight = 4 } 
else if( bmi >= 30 ){ overweight = 3 } 
else if( bmi >= 25 ){ overweight = 2 }
else if( bmi >= 20 ){ overweight = 1 }
else if( bmi >= 0  ){ overweight = 0 }
else { overweight = . }
'

new_df = datasailr::sail( df, code , fullData = T);
new_df
```

* (OUTPUT)

```
  height  bw      bmi overweight
1   1.70  50 17.30104          0
2   1.50  60 26.66667          2
3   1.50  80 35.55556          4
4   1.65  NA       NA        NaN
5   1.50 100 44.44444          5
```


#### Infinite numbers

Infinite numbers are also supported. Internally this is implemented as Inf in C. The behavior is same as C.

(ref.) NAN and infinity in C Standard ( https://www.gnu.org/software/libc/manual/html_node/Infinity-and-NaN.html )
(ref.) This support seems to require at least C99. ( https://stackoverflow.com/questions/6235847/how-to-generate-nan-infinity-and-infinity-in-ansi-c )

```
x = 10
y = 0
z = -10

a = x / y /* INF */
b = z / y /* -INF */
c = (-1) ** (-1/2) /* sqrt(-1) => -nan */

print("a is ", a , "\n")
print("b is ", b , "\n")
print("c is ", c , "\n")

if( a >= 0 ){
  print("a is zero or more than zero. \n")
}else if( a < 0){
  print("a is less than zero. \n")
}else{
  print("a is not a number. \n")
}

if( b >= 0 ){
  print("b is zero or more than zero. \n")
}else if( b < 0){
  print("b is less than zero. \n")
}else{
  print("b is not a number. \n")
}

if( c >= 0 ){
  print("c is zero or more than zero. \n")
}else if(c < 0){
  print("c is less than zero. \n")
}else if(c == .){
  print("c is not a number. \n")
}
```

* OUTPUT

```
a is inf
b is -inf
c is -nan
a is zero or more than zero. 
b is less than zero. 
c is not a number. 
```


### UTF8 support and encodings

UTF8 encoding is strongly encouraged. The recommended way is to have dataframe as UTF8, and also have your code as UTF8. ASCII or latin1 is compatible with UTF8, so they must also work properly. (Your terminal encoding also need to be UTF8 for properly printing out UTF8 strings.)
 
In DataSailr (libsailr), strings are stored just as a sequence of bytes. So even if you use other encodings, just copying and concatenating strings must work fine. Problems must happen when your source code and dataframe do not have the same encoding. Encoding of source code affects the string objects that are generated during parsing phase (=string objects that are generated before running code), dataframe encoding affects the string that are accessed via variables. 

For functions that process strings, UFF8-CPP library is used internally. ( UTF8-CPP:UTF-8 with C++ in a Portable Way https://github.com/nemtrif/utfcpp )

```
greeting = "Hello World"  // Only ascii string
str_len(greeting)  // 11
str_subset(1,5)  // Hello


greeting_jp = "こんにちは! 世界"  // Mixture of single byte string and multi byte strings.
str_len(greeting_jp)     // 8
str_subset(1,5)    // こんにちは
str_subset(8,9)    // 世界

```

### Regular expression support

Currently, the regular expression objects are not generated during run time but only in parsing phase. In other words, regular expressions can be generated only in regular expression literal. Regular expression syntax is re/pattern/ , which start with re/ and end with /. 

The regular expression engine used internally is Onigmo ( https://github.com/k-takata/Onigmo ). The source code must be written in UTF8, because regular expression object is programmed to assume UTF8 for its input (in libsailr).

As the following code shows, backreference (outside of the pattern) can be used via regexp_matched() function.

```
germany = re/(^Merc|^Porsche|^Volvo)/

if ( _rowname_ =~ germany ) {
    country = "Germany" ;
    type = rexp_matched(1);
}
```


### Date support

In DataSailr(libsair), there are no specific types for dates, and dates are stored as just integers in DataSailr. The number represents the day from Jan 1st 1970, which is the same as UNIX date. (Be careful that SAS software also store date information as numbers, but their starting date is 1/1/1960. https://support.sas.com/resources/papers/proceedings12/244-2012.pdf )

Internally for date calculation, date library is used ( "date.h" is a header-only library which builds upon <chrono>. https://github.com/HowardHinnant/date ).

```
date_ymd( year, month, day )
date_ym_weekday_nth( year, month, weekday, nth )  // weekday should be "Sun", "Mon" ... 
date_add_n_years( unix_date, years )
date_add_n_months( unix_date, months )
date_add_n_days( unix_date, days )
date_format( unix_date, str_format )  // This format should follow C++'s std::chrono::format. "%m/%d/%y" or "%Y-%m-%d" is popular.
```


## For SAS software users

List of major differences between DataSailr and DATA Step in SAS software.

1. DataSailr if-else statements use curly brackets { }. SAS software uses if-then-else style.
2. DataSailr if-else curly brackets can hold multiple statements. SAS software needs to use do statement for multiple statements. i.e. if-then-do-end style. 
3. OUTPUT statement is not supported in DataSailr yet.
4. RETAIN statement is not supported and there is no plan to support it in the future.
5. The default character encoding of DataSailr is UTF8. 
6. Not only semicolon but also new line means the end of each statement in DataSailr. In SAS software, semicolons are always required.
7. (CAUTION) Date starts from Jan. 1st 1970, which is same as UNIX date. SAS software also stores date as number that is a day from 1/1/1960. Meaning NEVER DIRECTLY USE SAS NUMBER FOR DATES in DataSailr(libsailr).
8. Column name does not have label (In SAS software, dataset column can have a description called label.)



## For dplyr users

DataSailr does not compete with dplyr. DataSailr focuses on row by row manipulation, and dplyr has more functionalities about the whole dataframe manipulation (e.g. arrange(), select(), group_by() & summarize()).

DataSailr's special feature is that you can write multiple row level manipulations in one code, though dplyr enables it with a chain of functions.


|                        |     DataSailr                  |     dplyr            |
-------------------------|--------------------------------|------------------------ 
| How to manipulate data | Apply a single Sailr code      | Apply multiple functions using (%>%) |
| Create new column      | assign value to new variable   | mutate()             |
| Keep some columns      | (Not implemented yet)          | select()             |
| Keep some rows         | (Not implemented yet)          | filter()             |
| Summarize columns      | No use for this purpose        | summarize()          |
| Sort rows              | No use for this purpose        | arrange()            |
| Regular expression     | Built-in                       | Partially available with another R package |
| Available functions    | Can call only Sailr functions  | Can call R functions |



## For developers & Inside DataSailr

### Architecture summary

The core engine of this package is libsailr. Libsailr takes Sailr script and does arithmetic calculations and string/character manipulations. This R package wraps libsair, and passes values of each row to the libsailr library. The results of each row will be returned as each row of new dataframe.



## Wish list

Writing a long string in R is not easy b/c I need to escape quotations. If R allowed HERE document or triple single quotation literal for strings, it would relieve this pain. 


## Contact

Your feedback is welcome. 

Toshi Umehara, toshi@niceume.com.

Maintainer: Toshi Umehara <toshi@niceume.com>


