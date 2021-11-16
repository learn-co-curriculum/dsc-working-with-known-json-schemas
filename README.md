# Working with Known JSON Schemas

## Introduction
You've started taking a look at JSON files and you'll continue to explore how to navigate and traverse these files. One common use case of JSON files will be when you are connecting to various websites through their established APIs to retrieve data from them. With these, you are typically given a schema for how the data is structured and then will use this knowledge to retrieve pertinent information.

## Objectives

You will be able to:

* Use the JSON module to load and parse JSON documents
* Extract data using predefined JSON schemas
* Convert JSON to a pandas dataframe

## Reading a JSON Schema

In this lesson, you'll take a look at the response from the New York Times API. (We cover APIs in more depth in other lessons, but the general idea is that the New York Times makes some of its data available over the web, and it uses the JSON format to do so.)

Here's the JSON schema provided for a section of the NY Times API:
<img src="images/schema_overview.png" width="400">

or a more detailed view (truncated):

<img src="images/schema_detailed.png" width="500">

You can see that the master structure is a dictionary and has a key named `'response'`. The value associated with the `'response'` key is also a dictionary and has two keys: `'docs'` and `'meta'`. As you continue to examine the schema hierarchy, you'll notice the vast majority of the elements comprising this data structure, in this case, are dictionaries. 

## Loading the Data File

As we have done in previous lessons, let's start by importing this data from the file. The code below uses the `json` module ([documentation here](https://docs.python.org/3/library/json.html)) and built-in `open` function to load the data from a JSON file into a Python object called `data`.


```python
import json
```


```python
with open('ny_times_response.json', 'r') as f:
    data = json.load(f)
```


```python
print(type(data))
print(data.keys())
```

    <class 'dict'>
    dict_keys(['status', 'copyright', 'response'])


You should see that there are two additional keys `'status'` and `'copyright'` which were not shown in the schema documentation. As with most forms of documentation, it's important to be aware that published schemas may differ somewhat from the actual data, and your code should be able to handle these unexpected differences, within reason.

## Loading Specific Data

Looking at the schema, you might be interested in retrieving a specific piece of data, such as the articles' headlines. Notice that this is a key under `'docs'`, which is under `'response'`. So the schema is roughly: data --> 'response' --> 'docs' --> 'headline', something like `data['response']['docs']['headline']`.

Let's see what happens if we try that:


```python
data['response']['docs']['headline']
```


    ---------------------------------------------------------------------------

    TypeError                                 Traceback (most recent call last)

    <ipython-input-4-a26674dfb096> in <module>
    ----> 1 data['response']['docs']['headline']
    

    TypeError: list indices must be integers or slices, not str


Ok, this error message is saying that somewhere along the way, **we treated something like a dictionary when it was actually a list**. Let's break down that chain of commands to figure out what went wrong.

We are pretty sure that `data['response']` will not cause an error, since we already checked that `data` is type `dict`, and that `'response'` is one of the keys. But what is the type of `data['response']`?


```python
type(data['response'])
```




    dict



Ok, that's a dictionary, too. How about `data['response']['docs']`?


```python
type(data['response']['docs'])
```




    list



So, that is the source of the error. We tried to treat this as a dictionary (accessing the value associated with the key `'headline'`) but it's a list!

If you scroll back up to the schema pictured above, this makes sense. The value associated with the `'docs'` key is shown surrounded by `[` and `]`, right before the `{` and `}`, indicating that this is a *list* of dictionaries, not just a dictionary.

You'll run into this kind of distinction repeatedly when working with JSON data. Sometimes values will be nested in unexpected ways, or you'll miss a key detail when you're skimming the schema. What's most important is that you're able to keep going and figure out what went wrong, not that you get it right on the first try!

Now that we know that this is a list, let's extract it and print out some more information about it:


```python
docs = data['response']['docs']

print("`docs` is a data structure of type", type(docs))
print("It contains", len(docs), "elements")
print("The first element is type", type(docs[0]))
```

    `docs` is a data structure of type <class 'list'>
    It contains 9 elements
    The first element is type <class 'dict'>


This confirms what we expected. Now we can loop over that list of dictionaries and print the values associated with the `'headline'` keys:


```python
for doc in docs:
    print(doc['headline'])
```

    {'main': "HIGGINS, SPENT $22,189.53.; Governor-Elect's Election Expenses -- Harrison $9,220.28.", 'kicker': None, 'content_kicker': None, 'print_headline': None, 'name': None, 'seo': None, 'sub': None}
    {'main': 'GARDEN BOUTS CANCELED; Mauriello Says He Could Not Be Ready on Nov. 3', 'kicker': '1', 'content_kicker': None, 'print_headline': None, 'name': None, 'seo': None, 'sub': None}
    {'main': 'Stock Drop Is Biggest in 2 Months--Margin Rise Held Factor in Lightest Trading of 1955', 'kicker': '1', 'content_kicker': None, 'print_headline': None, 'name': None, 'seo': None, 'sub': None}
    {'main': 'MUSIC OF THE WEEK', 'kicker': None, 'content_kicker': None, 'print_headline': None, 'name': None, 'seo': None, 'sub': None}
    {'main': 'Anacomp Inc. reports earnings for Qtr to March 31', 'kicker': None, 'content_kicker': None, 'print_headline': None, 'name': None, 'seo': None, 'sub': None}
    {'main': 'Brooklyn Routs Yeshiva', 'kicker': '1', 'content_kicker': None, 'print_headline': None, 'name': None, 'seo': None, 'sub': None}
    {'main': 'Albuquerque Program Gives Drinkers a Lift', 'kicker': '1', 'content_kicker': None, 'print_headline': None, 'name': None, 'seo': None, 'sub': None}
    {'main': 'Front Page 7 -- No Title', 'kicker': '1', 'content_kicker': None, 'print_headline': None, 'name': None, 'seo': None, 'sub': None}
    {'main': 'UNIONS AND BUILDERS READY FOR LONG FIGHT; None of the Strikers Back - Lock-Out Soon in Effect. 23,000 ALREADY INVOLVED Orders Sent to Every Building Employer Within Twenty-five Miles -- House-smiths Vote Not to Strike.', 'kicker': None, 'content_kicker': None, 'print_headline': None, 'name': None, 'seo': None, 'sub': None}


Or if you want to just print the main headlines themselves:


```python
for doc in docs:
    print(doc['headline']['main'])
```

    HIGGINS, SPENT $22,189.53.; Governor-Elect's Election Expenses -- Harrison $9,220.28.
    GARDEN BOUTS CANCELED; Mauriello Says He Could Not Be Ready on Nov. 3
    Stock Drop Is Biggest in 2 Months--Margin Rise Held Factor in Lightest Trading of 1955
    MUSIC OF THE WEEK
    Anacomp Inc. reports earnings for Qtr to March 31
    Brooklyn Routs Yeshiva
    Albuquerque Program Gives Drinkers a Lift
    Front Page 7 -- No Title
    UNIONS AND BUILDERS READY FOR LONG FIGHT; None of the Strikers Back - Lock-Out Soon in Effect. 23,000 ALREADY INVOLVED Orders Sent to Every Building Employer Within Twenty-five Miles -- House-smiths Vote Not to Strike.


## Flattening Data (i.e. Breaking Out Nested Data)

Let's say we want to create a list of dictionaries containing information about the documents contained in this JSON. It should contain the publication date (value associated with `pub_date` key), word count (value associated with `word_count` key), and both the `'main'` and `'kicker'` associated with the `headline` key. This list should be called `doc_info_list` and should look something like this:

```python
[
    {
        'headline_main': "HIGGINS, SPENT $22,189.53.; Governor-Elect's Election Expenses -- Harrison $9,220.28.",
        'headline_kicker': None,
        'pub_date': '1904-11-17T00:00:00Z',
        'word_count': 213
    },
    {
        'headline_main': 'GARDEN BOUTS CANCELED; Mauriello Says He Could Not Be Ready on Nov. 3',
        'headline_kicker': '1',
        'pub_date': '1944-10-23T00:00:00Z',
        'word_count': 149
    },
    ...
]
```

The tricky part is, each dictionary needs to be "flat", meaning that each key is associated with a single string or number value, not a deeper data structure. So we need to flatten the nested `headline` dictionary.

It's also conventional when flattening data to make a compound name for the newly-created keys. So, let's call the new keys `headline_main` and `headline_kicker`.

Recall the structure of a `headline` dictionary:


```python
docs[2]['headline']
```




    {'main': 'Stock Drop Is Biggest in 2 Months--Margin Rise Held Factor in Lightest Trading of 1955',
     'kicker': '1',
     'content_kicker': None,
     'print_headline': None,
     'name': None,
     'seo': None,
     'sub': None}



So, first let's write a function that takes in that complete dictionary, and returns a copy with only the `'main'` and `'kicker'` keys and values, now labeled `'headline_main'` and `'headline_kicker'`:


```python
def extract_headline_info(headline_dict):
    result = {}
    result['headline_main'] = headline_dict['main']
    result['headline_kicker'] = headline_dict['kicker']
    return result
```

Then we test it out:


```python
extract_headline_info(docs[2]['headline'])
```




    {'headline_main': 'Stock Drop Is Biggest in 2 Months--Margin Rise Held Factor in Lightest Trading of 1955',
     'headline_kicker': '1'}




```python
extract_headline_info(docs[0]['headline'])
```




    {'headline_main': "HIGGINS, SPENT $22,189.53.; Governor-Elect's Election Expenses -- Harrison $9,220.28.",
     'headline_kicker': None}



Now let's write another function that calls that function, then adds the `pub_date` and `word_count` keys and values:


```python
def extract_doc_info(doc):
    info = extract_headline_info(doc['headline'])
    info['pub_date'] = doc['pub_date']
    info['word_count'] = doc['word_count']
    return info
```

Again, testing it out on a couple examples:


```python
extract_doc_info(docs[2])
```




    {'headline_main': 'Stock Drop Is Biggest in 2 Months--Margin Rise Held Factor in Lightest Trading of 1955',
     'headline_kicker': '1',
     'pub_date': '1955-05-15T00:00:00Z',
     'word_count': 823}




```python
extract_doc_info(docs[0])
```




    {'headline_main': "HIGGINS, SPENT $22,189.53.; Governor-Elect's Election Expenses -- Harrison $9,220.28.",
     'headline_kicker': None,
     'pub_date': '1904-11-17T00:00:00Z',
     'word_count': 213}



Now we can loop over the full list and create `doc_info_list`:


```python
doc_info_list = [extract_doc_info(doc) for doc in docs]
doc_info_list
```




    [{'headline_main': "HIGGINS, SPENT $22,189.53.; Governor-Elect's Election Expenses -- Harrison $9,220.28.",
      'headline_kicker': None,
      'pub_date': '1904-11-17T00:00:00Z',
      'word_count': 213},
     {'headline_main': 'GARDEN BOUTS CANCELED; Mauriello Says He Could Not Be Ready on Nov. 3',
      'headline_kicker': '1',
      'pub_date': '1944-10-23T00:00:00Z',
      'word_count': 149},
     {'headline_main': 'Stock Drop Is Biggest in 2 Months--Margin Rise Held Factor in Lightest Trading of 1955',
      'headline_kicker': '1',
      'pub_date': '1955-05-15T00:00:00Z',
      'word_count': 823},
     {'headline_main': 'MUSIC OF THE WEEK',
      'headline_kicker': None,
      'pub_date': '1904-11-06T00:00:00Z',
      'word_count': 2609},
     {'headline_main': 'Anacomp Inc. reports earnings for Qtr to March 31',
      'headline_kicker': None,
      'pub_date': '1992-05-06T00:00:00Z',
      'word_count': 129},
     {'headline_main': 'Brooklyn Routs Yeshiva',
      'headline_kicker': '1',
      'pub_date': '1972-12-24T00:00:00Z',
      'word_count': 144},
     {'headline_main': 'Albuquerque Program Gives Drinkers a Lift',
      'headline_kicker': '1',
      'pub_date': '1972-12-25T00:00:00Z',
      'word_count': 151},
     {'headline_main': 'Front Page 7 -- No Title',
      'headline_kicker': '1',
      'pub_date': '1944-10-24T00:00:00Z',
      'word_count': 29},
     {'headline_main': 'UNIONS AND BUILDERS READY FOR LONG FIGHT; None of the Strikers Back - Lock-Out Soon in Effect. 23,000 ALREADY INVOLVED Orders Sent to Every Building Employer Within Twenty-five Miles -- House-smiths Vote Not to Strike.',
      'headline_kicker': None,
      'pub_date': '1904-08-06T00:00:00Z',
      'word_count': 883}]



Thus we have successfully flattened the required data!

## Transforming JSON to Alternative Formats

### Viewing the Raw Dataset in Pandas

You've also previously started to take a look at how to transform JSON to DataFrames. Investigating the schema, a good option for this could again be the value associated with the `'docs'` key. While this still has nested data itself, it's often easier to load the entire contents as a DataFrame for viewing and then use additional functions to break apart the internally nested data from there.

So, first we will display the full information associated with the `'docs'` key:


```python
import pandas as pd
pd.DataFrame(data['response']['docs'])
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>web_url</th>
      <th>snippet</th>
      <th>abstract</th>
      <th>print_page</th>
      <th>blog</th>
      <th>source</th>
      <th>multimedia</th>
      <th>headline</th>
      <th>keywords</th>
      <th>pub_date</th>
      <th>document_type</th>
      <th>type_of_material</th>
      <th>_id</th>
      <th>word_count</th>
      <th>score</th>
      <th>byline</th>
      <th>news_desk</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>https://query.nytimes.com/gst/abstract.html?re...</td>
      <td>Spent $22,200</td>
      <td>Spent $22,200</td>
      <td>2</td>
      <td>{}</td>
      <td>The New York Times</td>
      <td>[]</td>
      <td>{'main': 'HIGGINS, SPENT $22,189.53.; Governor...</td>
      <td>[{'name': 'persons', 'value': 'HIGGINS, LT. GO...</td>
      <td>1904-11-17T00:00:00Z</td>
      <td>article</td>
      <td>Article</td>
      <td>4fc04eb745c1498b0d23da00</td>
      <td>213</td>
      <td>1</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>https://query.nytimes.com/gst/abstract.html?re...</td>
      <td></td>
      <td>NaN</td>
      <td>15</td>
      <td>{}</td>
      <td>The New York Times</td>
      <td>[]</td>
      <td>{'main': 'GARDEN BOUTS CANCELED; Mauriello Say...</td>
      <td>[]</td>
      <td>1944-10-23T00:00:00Z</td>
      <td>article</td>
      <td>Article</td>
      <td>4fc21ebf45c1498b0d612b22</td>
      <td>149</td>
      <td>1</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>https://query.nytimes.com/gst/abstract.html?re...</td>
      <td>Stock prices last week, on the lightest volume...</td>
      <td>NaN</td>
      <td>F1</td>
      <td>{}</td>
      <td>The New York Times</td>
      <td>[]</td>
      <td>{'main': 'Stock Drop Is Biggest in 2 Months--M...</td>
      <td>[]</td>
      <td>1955-05-15T00:00:00Z</td>
      <td>article</td>
      <td>Article</td>
      <td>4fc3b41d45c1498b0d7fd41e</td>
      <td>823</td>
      <td>1</td>
      <td>{'original': 'By JOHN G. FORREST', 'person': [...</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>https://query.nytimes.com/gst/abstract.html?re...</td>
      <td>The first public rehearsal and concert of the ...</td>
      <td>Healy, Michael, will suit</td>
      <td>20</td>
      <td>{}</td>
      <td>The New York Times</td>
      <td>[]</td>
      <td>{'main': 'MUSIC OF THE WEEK', 'kicker': None, ...</td>
      <td>[{'name': 'persons', 'value': 'HEALY, MICHAEL'...</td>
      <td>1904-11-06T00:00:00Z</td>
      <td>article</td>
      <td>Article</td>
      <td>4fc04eb745c1498b0d23da12</td>
      <td>2609</td>
      <td>1</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>https://www.nytimes.com/1992/05/06/business/an...</td>
      <td></td>
      <td>NaN</td>
      <td>20</td>
      <td>{}</td>
      <td>The New York Times</td>
      <td>[]</td>
      <td>{'main': 'Anacomp Inc. reports earnings for Qt...</td>
      <td>[{'name': 'subject', 'value': 'COMPANY EARNING...</td>
      <td>1992-05-06T00:00:00Z</td>
      <td>article</td>
      <td>Statistics</td>
      <td>4fd1b3018eb7c8105d6d690a</td>
      <td>129</td>
      <td>1</td>
      <td>NaN</td>
      <td>Financial Desk</td>
    </tr>
    <tr>
      <th>5</th>
      <td>https://query.nytimes.com/gst/abstract.html?re...</td>
      <td></td>
      <td>NaN</td>
      <td>S9</td>
      <td>{}</td>
      <td>The New York Times</td>
      <td>[]</td>
      <td>{'main': 'Brooklyn Routs Yeshiva', 'kicker': '...</td>
      <td>[]</td>
      <td>1972-12-24T00:00:00Z</td>
      <td>article</td>
      <td>Article</td>
      <td>4fc47bb045c1498b0da03363</td>
      <td>144</td>
      <td>1</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>6</th>
      <td>https://query.nytimes.com/gst/abstract.html?re...</td>
      <td>ALBUQUERQUE, N. M., Dec. 24 -- Holiday drinker...</td>
      <td>NaN</td>
      <td>11</td>
      <td>{}</td>
      <td>The New York Times</td>
      <td>[]</td>
      <td>{'main': 'Albuquerque Program Gives Drinkers a...</td>
      <td>[]</td>
      <td>1972-12-25T00:00:00Z</td>
      <td>article</td>
      <td>Article</td>
      <td>4fc47bb045c1498b0da03367</td>
      <td>151</td>
      <td>1</td>
      <td>{'original': 'Special to The New York Times', ...</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>7</th>
      <td>https://query.nytimes.com/gst/abstract.html?re...</td>
      <td></td>
      <td>NaN</td>
      <td>1</td>
      <td>{}</td>
      <td>The New York Times</td>
      <td>[]</td>
      <td>{'main': 'Front Page 7 -- No Title', 'kicker':...</td>
      <td>[]</td>
      <td>1944-10-24T00:00:00Z</td>
      <td>article</td>
      <td>Front Page</td>
      <td>4fc21ebf45c1498b0d612b3c</td>
      <td>29</td>
      <td>1</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>8</th>
      <td>https://query.nytimes.com/gst/abstract.html?re...</td>
      <td>The employers and the unions have lined up in ...</td>
      <td>housesmiths won't strike</td>
      <td>1</td>
      <td>{}</td>
      <td>The New York Times</td>
      <td>[]</td>
      <td>{'main': 'UNIONS AND BUILDERS READY FOR LONG F...</td>
      <td>[{'name': 'glocations', 'value': 'NEW YORK CIT...</td>
      <td>1904-08-06T00:00:00Z</td>
      <td>article</td>
      <td>Front Page</td>
      <td>4fc04eb745c1498b0d23da17</td>
      <td>883</td>
      <td>1</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>



Note that because the value associated with the `'headline'` key is a dictionary, it is displayed in this crowded, messy way within the DataFrame, including `{` and `'` characters.

### Viewing the Flattened Info List

Because `doc_info_list` is already flattened so the value associated with each key is just a number or string, it looks much neater when loaded into pandas:


```python
pd.DataFrame(doc_info_list)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>headline_main</th>
      <th>headline_kicker</th>
      <th>pub_date</th>
      <th>word_count</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>HIGGINS, SPENT $22,189.53.; Governor-Elect's E...</td>
      <td>None</td>
      <td>1904-11-17T00:00:00Z</td>
      <td>213</td>
    </tr>
    <tr>
      <th>1</th>
      <td>GARDEN BOUTS CANCELED; Mauriello Says He Could...</td>
      <td>1</td>
      <td>1944-10-23T00:00:00Z</td>
      <td>149</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Stock Drop Is Biggest in 2 Months--Margin Rise...</td>
      <td>1</td>
      <td>1955-05-15T00:00:00Z</td>
      <td>823</td>
    </tr>
    <tr>
      <th>3</th>
      <td>MUSIC OF THE WEEK</td>
      <td>None</td>
      <td>1904-11-06T00:00:00Z</td>
      <td>2609</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Anacomp Inc. reports earnings for Qtr to March 31</td>
      <td>None</td>
      <td>1992-05-06T00:00:00Z</td>
      <td>129</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Brooklyn Routs Yeshiva</td>
      <td>1</td>
      <td>1972-12-24T00:00:00Z</td>
      <td>144</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Albuquerque Program Gives Drinkers a Lift</td>
      <td>1</td>
      <td>1972-12-25T00:00:00Z</td>
      <td>151</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Front Page 7 -- No Title</td>
      <td>1</td>
      <td>1944-10-24T00:00:00Z</td>
      <td>29</td>
    </tr>
    <tr>
      <th>8</th>
      <td>UNIONS AND BUILDERS READY FOR LONG FIGHT; None...</td>
      <td>None</td>
      <td>1904-08-06T00:00:00Z</td>
      <td>883</td>
    </tr>
  </tbody>
</table>
</div>



We could also re-create this from the raw data using pandas rather than base Python:


```python
# Create dataframe of raw docs info
df = pd.DataFrame(data['response']['docs'])

# Make new headline_main and headline_kicker columns
df['headline_main'] = df['headline'].apply(lambda headline_dict: headline_dict['main'])
df['headline_kicker'] = df['headline'].apply(lambda headline_dict: headline_dict['kicker'])

# Subset to only the relevant columns
df = df[['headline_main', 'headline_kicker', 'pub_date', 'word_count']]
df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>headline_main</th>
      <th>headline_kicker</th>
      <th>pub_date</th>
      <th>word_count</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>HIGGINS, SPENT $22,189.53.; Governor-Elect's E...</td>
      <td>None</td>
      <td>1904-11-17T00:00:00Z</td>
      <td>213</td>
    </tr>
    <tr>
      <th>1</th>
      <td>GARDEN BOUTS CANCELED; Mauriello Says He Could...</td>
      <td>1</td>
      <td>1944-10-23T00:00:00Z</td>
      <td>149</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Stock Drop Is Biggest in 2 Months--Margin Rise...</td>
      <td>1</td>
      <td>1955-05-15T00:00:00Z</td>
      <td>823</td>
    </tr>
    <tr>
      <th>3</th>
      <td>MUSIC OF THE WEEK</td>
      <td>None</td>
      <td>1904-11-06T00:00:00Z</td>
      <td>2609</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Anacomp Inc. reports earnings for Qtr to March 31</td>
      <td>None</td>
      <td>1992-05-06T00:00:00Z</td>
      <td>129</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Brooklyn Routs Yeshiva</td>
      <td>1</td>
      <td>1972-12-24T00:00:00Z</td>
      <td>144</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Albuquerque Program Gives Drinkers a Lift</td>
      <td>1</td>
      <td>1972-12-25T00:00:00Z</td>
      <td>151</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Front Page 7 -- No Title</td>
      <td>1</td>
      <td>1944-10-24T00:00:00Z</td>
      <td>29</td>
    </tr>
    <tr>
      <th>8</th>
      <td>UNIONS AND BUILDERS READY FOR LONG FIGHT; None...</td>
      <td>None</td>
      <td>1904-08-06T00:00:00Z</td>
      <td>883</td>
    </tr>
  </tbody>
</table>
</div>



Wahoo! This is a good general strategy for transforming nested JSON: create a DataFrame and then break out nested features into their own column features.

## Outputting to JSON

Finally, take a look at how you can write data back to JSON. Like loading, you first open a file (this time in write mode) and use the json package to interact with that file object. Only instead of `json.load` to load the contents of the file into a Python object, you call `json.dump` to write the contents of the Python object into the file.


```python
with open('doc_info_list.json', 'w') as f:
    json.dump(doc_info_list, f)
```

Then if we want to load that cleaned dataset for future use, we can open that new file:


```python
with open('doc_info_list.json') as f:
    doc_info_list_from_disk = json.load(f)
```

The new file should contain identical information to the original Python variable:


```python
doc_info_list_from_disk == doc_info_list
```




    True



## Summary
There you have it! In this, you practiced using JSON some more, this time interpreting an example schema diagram in order to retrieve information. You also looked at a general procedure for transforming nested data to pandas DataFrames (create a DataFrame, and then break apart nested data using lambda functions to create additional columns). Finally, you also took a brief look at saving data to JSON files. 
