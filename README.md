# Deliverando: A Business Intelligence project

BI coding challenge at Spiced Academy. KPIs, competitor analysis. Created in Python.

Final presentation can be found in 'final_presentation.pdf'.


## Questions

What is the state of business in Graz for Deliverando, a fictive food delivery company?

1. What can we say about relevant KPIs?
2. How many restaurants are active on Deliverando or our competitors and how much have the respective platforms grown?
3. Which 10 restaurants have placed the most orders with our competitors and how are they doing on Deliverando?

## Data

Data were provided by Spiced. However, the data sets contain real-world restaurants from Austria. I anonymized them using Python's Faker library, while in the same time attempting to generate Austrian sounding restaurant names.

The anonymized data sets ready for analysis are available in the 'data' folder as CSV files.

## Anonymization

The whole anonymization code is described in the anonymization.ipynb notebook in the 'notebooks' folder.

I started by importing the relevant libraries, importing files and saving them as CSV files with comma used as a separator. In this way, I could use loops to anonymize all the files in the same time. This was necessary to rename all restaurants consistently across all files (so called mapping). I also used dictionaries with Austrian adjectives and nouns to randomly generate the new names so the project would be more authentic.

The main part of the code consists of defining three functions, a function to read and write CSV files with data and calling inner function anonymizing names. A third function was needed to check that all randomly generated names were unique (or lead to new generation otherwise). The example of the function anonymizing individiual rows is below.

```
def anonymize_rows(rows, name_mappings):
    """
    Rows is an iterable of a dictionary that contains name field that needs to be anonymized.
    Name_mappings is the dictionary.
    """
    for row in rows:
        original_name = row['name']   # original name of the restaurant in the 'name' column
        anonymized_name = name_mappings.get(original_name, original_name)   # check if it is present in mapping
        row['name'] = anonymized_name   # replace the name field with the generated or mapped restaurant name
        yield row   # more efficient, returns row one by one during an iteration in a loop
```

I followed the approach by [District Data Labs here](https://medium.com/district-data-labs/a-practical-guide-to-anonymizing-datasets-with-python-faker-ecf15114c9be) and used the help of ChatGPT in this notebook to customize the code to my needs and to generate lists of Austrian words.

## Cleaning and wrangling

The cleaning part is included in the deliverando_business_intelligence.ipynb notebook in the 'notebooks' folder.

It consisted mainly of standardizing columns, filtering for relevant geographical locations, distinguishing between franchises with cummulative count, melting and pivoting. Here is an example of data wrangling, melting two columns with KPI values (month_1, month_2) and subsequently pivoting the KPIs to be in separate columns. Such wrangling is useful for further analyses and visualizations.

```
df_deliverando_long = pd.melt(df_deliverando, 
                               id_vars=['name', 'zip', 'delivery_service', 'franchise_name', 'kpi'],
                                      # variables not to be considered, identificating rows
                               value_vars=['month_1', 'month_2'],   # columns to be melted into one
                               var_name='month',   # the name of the new column with original variable names
                               value_name='value')   # the name of the new column with values

df_deliverando_wide = pd.pivot(df_deliverando_long,
                    columns='kpi',   # column to pivot
                    index=['franchise_name', 'month'],   # column to be kept, unique restaurant franchise for each row
                    values='value')   # column of values to pivot
df_deliverando_wide = df_deliverando_wide.reset_index()   # get rid of multicolumn index
```

## Analysis and visualizations

This time, I found it useful to analyze the numerous questions and visualize at the same place. Hence both codes are part of the deliverando_business_intelligence.ipynb notebook in the 'notebooks' folder.

I went through different KPIs for Deliverando, checking the change between two months, and answered some questions comparing Deliverando and the competition. I created simple easy-to-understand plots for a presentation in front of stakeholders.

An example is the average time to accept an order. First, I created a new data frame with only relevant values and then calculated a product of two columns to be used subsequantly to obtain weighted averages. The result of the code is the average time to accept over all restaurants for first and second month and the change as percentage:

```
df_time = df_deliverando_wide[['franchise_name', 'month', 'orders', 'avg_acc_time']]

df_time['product'] = df_time['orders']*df_time['avg_acc_time']

acc_time_avg_m1 = df_time[df_time['month']=='month_1']['product'].sum()/\
df_time[df_time['month']=='month_1']['orders'].sum()
    # weighted average for June

acc_time_avg_m2 = df_time[df_time['month']=='month_2']['product'].sum()/\
df_time[df_time['month']=='month_2']['orders'].sum()
    # weighted average for July

acc_time_avg_ratio = (acc_time_avg_m2/acc_time_avg_m1-1)*100
print(acc_time_avg_m1, acc_time_avg_m2, acc_time_avg_ratio)
```

I also calculated how much waiting time in total over all customers was saved thanks to the improvement:

```
acc_time_sum_m1 = df_time[df_time['month']=='month_1']['product'].sum()
acc_time_sum_m2 = df_time[df_time['month']=='month_2']['product'].sum()
acc_time_avg_diff = acc_time_sum_m2 - acc_time_sum_m1
print(acc_time_sum_m1, acc_time_sum_m2, acc_time_avg_diff)
```

An example of the code comparing both companies is a snippet, printing the number of restaurants only present at competition and their names. I like to use set operations for similar type of tasks:

```
set_competition = set(df_competition_month1['name'].unique())|set(df_competition_month2['name'].unique())
set_deliverando = set(df_deliverando['name'].unique())
only_competition = sorted(set_competition - set_deliverando, key=lambda x: x.lower()) # sorts regardless upper / lowercase

only_comp_str = ', '.join(only_competition) # makes a string out of list items
print(f'There are {len(only_competition)} restaurants solely on competing platforms.\n\
These are:', only_comp_str + '.')
```
