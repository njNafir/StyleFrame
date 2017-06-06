# StyleFrame v1.0
_Exporting DataFrame to designed excel file has never been so easy_


A library that wraps pandas and openpyxl and allows easy styling of dataframes in excel
```
$ pip install styleframe
```
You can read the documentation at http://styleframe.readthedocs.org/en/latest/

<img src=https://s.faketrumptweet.com/j3kkdbjj_1vtght_m1uf0.png width=295px height=120px/>    

---

## Contents
1. [Rationale](#rationale)
2. [Basics](#basics)
3. [Usage Examples](#usage-examples)    
&nbsp;&nbsp;&nbsp;&nbsp;- [Simple Example](#simple-example)    
&nbsp;&nbsp;&nbsp;&nbsp;- [Advance Example](#advance-example)   
4. [Commandline Interface](#commandline-interface)


## Rationale

Pandas's DataFrame is great.   
Dealing with a lot of data is not easy and DataFrame helps us to manage it in the besy way possible.   
 
There are many ways to present the output and one of them is excel files.   
Excel files are easy to understand, can be viewed offline, can be sent over the email
and a large percentage of the population familiar with it.   
That is why many times we would choose excel files as our output.   

StyleFrame package allows us to design the excel file on the data in a similar way to DataFrame api.   
It saves us the trouble of working with excel workbook and the suffering of trying to match it with the data stored in our DataFrame.


## Basics

* ***Styler***:
```python
__init__(self, bg_color=None, bold=False, font="Arial", font_size=12, font_color=None,
         number_format=utils.number_formats.general, protection=False, underline=None,
         border_type=utils.borders.thin)
```
Object that represents the style of a cell in our excel file.
Styler is responsible of storing the style of single cell.
Once the style is ready, ```.create_style()``` method is called.

* ***utils***:
```python
from StyleFrame import utils
```
Before you start to style your StyleFrame, take a look in the utils module.
You may find there very useful things such as number formats, colors, borders and more!


* ***Container***: 
```python
__init__(self, value, styler=None)
```
Object that represents cell in our excel file.
 it contains two variables:          
&nbsp;&nbsp;&nbsp;&nbsp;- value which may be anything you wish to put in the cell as long as excel file support its format.   
&nbsp;&nbsp;&nbsp;&nbsp;- style which is the style of the cell- created by ```Styler(...).create_style()```

And finally:

* ***StyleFrame***:
```python
__init__(self, obj, styler_obj=None):
```
StyleFrame is the main object we will be dealing with.
It contains self DataFrame which is based on the given obj.
Each item of the self DataFrame is wrapped by a Container object to store the given data and its` style.
StyleFrame (usually referred as sf) reveals a very easy api for styling.

## Usage Examples

### Simple Example

```python
import pandas as pd
import time

from StyleFrame import StyleFrame, Styler, utils

expected = 'Hey how are you today?'.split()
actual = 'Hello how are u today?'.split()
pass_or_failed = ['Passed' if e == a else 'Failed' for e, a in zip(expected, actual)]

df = pd.DataFrame({
    'Time': [time.time() for i in xrange(5)],
    'Expect': expected,
    'Actual': actual,
    'Pass/Fail': pass_or_failed
    },
    columns=['Time', 'Expect', 'Actual', 'Pass/Fail'])
"""Our DataFrame looks like this:

           Time  Expect  Actual Pass/Fail
0  1.496728e+09     Hey   Hello    Failed
1  1.496728e+09     how     how    Passed
2  1.496728e+09     are     are    Passed
3  1.496728e+09     you       u    Failed
4  1.496728e+09  today?  today?    Passed

"""

# Create StyleFrame object that wrap our DataFrame and assign default style.
defaults = {'font': 'Aharoni', 'font_size': 14}
sf = StyleFrame(df, styler_obj=Styler(**defaults))

# Style the headers of the table
header_style = Styler(bold=True, font_size=18)
sf.apply_headers_style(styler_obj=header_style)

# Set the background color to green where the test marked as 'passed'
passed_style = Styler(bg_color=utils.colors.green, font_color=utils.colors.white, **defaults)
sf.apply_style_by_indexes(indexes_to_style=sf[sf['Pass/Fail'] == 'Passed'],
                          cols_to_style='Pass/Fail',
                          styler_obj=passed_style)

# Set the background color to red where the test marked as 'failed'
failed_style = Styler(bg_color=utils.colors.red, font_color=utils.colors.white, **defaults)
sf.apply_style_by_indexes(indexes_to_style=sf[sf['Pass/Fail'] == 'Failed'],
                          cols_to_style='Pass/Fail',
                          styler_obj=failed_style)


sf.set_column_width(columns=list(sf.columns), width=20)

# excel rows starts from 1
# row number 1 is the headers
# len of StyleFrame (same as DataFrame) does not count the headers row
all_rows = tuple(i for i in range(1, len(sf) + 2))
sf.set_row_height(rows=all_rows, height=25)

sf.to_excel('output.xlsx',
            # Add filters in row 0 to each column.
            row_to_add_filters=0, 
            # Freeze the columns before column 'A' (=None) and rows above '2' (=1).
            columns_and_rows_to_freeze='A2').save()
```

### Advance Example

First, let us create a DataFrame that contains data we would like to export to an .xlsx file 
```python
import pandas as pd


columns = ['Date', 'Col A', 'Col B', 'Col C', 'Percentage']
df = pd.DataFrame(data={'Date': [date(1995, 9, 5), date(1947, 11, 29), date(2000, 1, 15)],
                        'Col A': [1, 2004, -3],
                        'Col B': [15, 3, 116],
                        'Col C': [33, -6, 9],
                        'Percentage': [0.113, 0.504, 0.005]},
                  columns=columns)

only_values_df = df[columns[1:-1]]

rows_max_value = only_values_df.idxmax(axis=1)

df['Sum'] = only_values_df.sum(axis=1)
df['Mean'] = only_values_df.mean(axis=1)
```

Now, once we have the DataFrame ready, lets create a StyleFrame object
```python
from StyleFrame import StyleFrame

sf = StyleFrame(df)
# it is also possible to directly initiate StyleFrame
sf = StyleFrame({'Date': [date(1995, 9, 5), date(1947, 11, 29), date(2000, 1, 15)],
                 'Col A': [1, 2004, -3],
                 'Col B': [15, 3, 116],
                 'Col C': [33, -6, 9],
                 'Percentage': [0.113, 0.504, 0.005]})
```

The StyleFrame object will auto-adjust the columns width and the rows height
but they can be changed manually
```python
sf.set_column_width_dict(col_width_dict={
    ('Col A', 'Col B', 'Col C'): 15.3,
    ('Sum', 'Mean'): 30,
    ('Percentage', ): 12
})

# excel rows starts from 1
# row number 1 is the headers
# len of StyleFrame (same as DataFrame) does not count the headers row
all_rows = tuple(i for i in range(1, len(sf) + 2))
sf.set_row_height_dict(row_height_dict={
    all_rows[0]: 45,
    all_rows[1:]: 25
})
```

Applying number formats
```python
from StyleFrame import Styler, utils


sf.apply_column_style(cols_to_style='Date',
                      styler_obj=Styler(number_format=utils.number_formats.date, font='Calibri', bold=True))

sf.apply_column_style(cols_to_style='Percentage',
                      styler_obj=Styler(number_format=utils.number_formats.percent))

sf.apply_column_style(cols_to_style=['Col A', 'Col B', 'Col C'],
                      styler_obj=Styler(number_format=utils.number_formats.thousands_comma_sep))                     
```

Next, let's change the background color of the maximum values to red and the font to white  
we will also protect those cells and prevent the ability to change their value
```python
style = Styler(bg_color=utils.colors.red, bold=True, font_color=utils.colors.white, protection=True,
               underline=utils.underline.double, number_format=utils.number_formats.thousands_comma_sep).create_style()
for row_index, col_name in rows_max_value.iteritems():
    sf[col_name][row_index].style = style
```

And change the font and the font size of Sum and Mean columns
```python
sf.apply_column_style(cols_to_style=['Sum', 'Mean'],
                      styler_obj=Styler(font_color='#40B5BF',
                                        font_size=18,
                                        bold=True),
                      style_header=True)
```

Change the background of all rows where the date is after 14/1/2000 to green
```python                 
sf.apply_style_by_indexes(indexes_to_style=sf[sf['Date'] > date(2000, 1, 14)],
                          cols_to_style='Date',
                          styler_obj=Styler(bg_color='green', number_format=utils.number_formats.date, bold=True))
```

Finally, let's export to Excel but not before we use more of StyleFrame's features:
- Change the page writing side
- Freeze rows and columns
- Add filters to headers

```python
ew = StyleFrame.ExcelWriter('sf tutorial.xlsx')
sf.to_excel(excel_writer=ew,
            sheet_name='1',
            right_to_left=False,
            columns_and_rows_to_freeze='B2', # will freeze the rows above 2 (=row 1 only) and columns that before column 'B' (=col A only)
            row_to_add_filters=0,
            allow_protection=True)
```

Adding another excel sheet
```python
other_sheet_sf = StyleFrame({'Dates': [date(2016, 10, 20), date(2016, 10, 21), date(2016, 10, 22)]},
                            styler_obj=Styler(number_format=utils.number_formats.date))
```

Don't forget to save
```python
ew.save()
```

**_the result:_**
<img src="https://s10.postimg.org/ppt8gt5m1/Untitled.png">


## Commandline Interface
