Merging Excel Sheets
================
Pedro Serio
2023-03-10

This will be a fast one. I made this tutorial because some people comes
to me with excel files with sheets divided by samples/subjects/etc. The
problem is that now they have 30 sheets with a lot of lines and columns
and they want to merge it.  
Although doing it manually is possible, we all know how hands-on excel
is error-prone (and in this case, tedious and time consuming). That
being said, here is a simple tutorial in how to merge excel sheets in a
single data frame.

Set your working directory (where your excel file is) and load the
needed packages

``` r
library(dplyr)
library(xlsx)
library(readxl)
library(rio)
```

# Example Data

The code below produces an excel file with 5 sheets, 2000 lines and 5
columns, each.  
Obs: I am pretty sure that there is an easier way to do it, but I could
not figure out how to make for loops and the `paste()` function work
without the `xlsx` package complaining. If you have the solution,
please, reach out to me.

Make the dataframe.

``` r
#Make a dataframe.
df <- as.data.frame(matrix(runif(n=50000, min=1, max=20), nrow=10000))

#Make a group column and bind it to the previous df.
df2<-data.frame(group=LETTERS[1:5])
df2<-bind_rows(replicate(2000,df2,simplify=FALSE))

#Bind both
full_df<-cbind(df2,df)
full_df$group<-as.factor(full_df$group)

#Create the example excel file with 5 sheets, 2000 lines and 5 columns in each sheet.
write.xlsx(subset(full_df, subset=group=="A"),
           file="example.xlsx",sheetName = "sheet1",append = TRUE,row.names = FALSE)

write.xlsx(subset(full_df, subset=group=="B"),
           file="example.xlsx",sheetName = "sheet2",append = TRUE,row.names = FALSE)

write.xlsx(subset(full_df, subset=group=="C"),
           file="example.xlsx",sheetName = "sheet3",append = TRUE,row.names = FALSE)

write.xlsx(subset(full_df, subset=group=="D"),
           file="example.xlsx",sheetName = "sheet4",append = TRUE,row.names = FALSE)

write.xlsx(subset(full_df, subset=group=="E"),
           file="example.xlsx",sheetName = "sheet5",append = TRUE,row.names = FALSE)
```

Use the `import_list()` command from the `rio` package to merge the
sheets and its done!

``` r
data_list <- import_list("example.xlsx", setclass = "tbl", rbind = TRUE)

#If you want, you can create and additional column to keep trace of the sheet of origin.

#Make a sheet list
excel_file<-"example.xlsx"
excel_sheets(excel_file)
```

    ## [1] "sheet1" "sheet2" "sheet3" "sheet4" "sheet5"

``` r
sheets_list<-as.vector(excel_sheets(excel_file))

#Count lines and convert each file line to the name of the original sheet
tags_freqs<-as.data.frame(table(data_list$`_file`))
tags_freqs<-as.vector(tags_freqs$Freq)
SHEET_ID<-rep(sheets_list,tags_freqs)
SHEET_ID<-as.data.frame(SHEET_ID)
data_list<-cbind(data_list,SHEET_ID)
```
