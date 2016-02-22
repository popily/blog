---
layout: narrative
title: Getting to Know the Popily API
author: Jonathon Morgan
---

Let's walk through how to get started with our new "analytics as a service" platform, so you can go from raw data to cool interactive visualizations in just a few lines of code.

First step, grab the `popily-api` package from pypi. 

{% highlight python %}
pip install popily-api
{% endhighlight %}

---

## First, the Super Fast Intro

{% highlight python %}
import popily_api

your_data_url = 'http://your-data.csv'
columns_in_your_data = [
    {
        'column_header': 'Column A',
        'data_type': 'numeric'
    },
    {
        'column_header': 'Column B',
        'data_type': 'category'
    }
]
popily = popily_api.Popily('YOUR API TOKEN')
source = popily.add_source('http://your-csv-file.csv', 
                    columns=columns_in_your_data)

insights = popily.get_insights(source['id'],
                        columns=['Column A', 'Column B'])

for insight in insights['results']:
    embed_url = insight['embed_url']
    print embed_url

# now go put the embed_url in an iframe
{% endhighlight %}

---

## Now More Details

First you'll need some data. Popily works with most machine-readable data like CSV files, JSON APIs, and databases like MySQL, PostgreSQL or ElasticSearch. For demonstration purposes I'll use [this CSV file of DC Comics characters](https://github.com/fivethirtyeight/data/blob/master/comic-characters/dc-wikia-data.csv) that the website 538 used in a [great story about women in comic books](http://fivethirtyeight.com/features/women-in-comic-books/).

Here's a few rows from the file so you know what we're working with.

| page_id | name                                 | urlslug                                      | ID              | ALIGN              | EYE                | HAIR                  | SEX               
|---------|--------------------------------------|----------------------------------------------|-----------------|--------------------|--------------------|-----------------------|-------------------
| 1422    | Batman (Bruce Wayne)                 | \/wiki\/Batman_(Bruce_Wayne)                 | Secret Identity | Good Characters    | Blue Eyes          | Black Hair            | Male Characters   
| 23387   | Superman (Clark Kent)                | \/wiki\/Superman_(Clark_Kent)                | Secret Identity | Good Characters    | Blue Eyes          | Black Hair            | Male Characters   
| 1448    | Wonder Woman (Diana Prince)          | \/wiki\/Wonder_Woman_(Diana_Prince)          | Public Identity | Good Characters    | Blue Eyes          | Black Hair            | Female Characters 

---

## Adding Data

We can post this file to Popily with a description of the type of data in each column, and Popily should do the rest. Popily understands more than just integers and strings, it can also work with higher-level concepts like categories, US states or currency. We're always adding to this list, but these are the available types as of today. 

- category
- numeric (integers or floats)
- currency 
- datetime (any kind of date)
- country 
- state (only US states for now)
- coordinate (formatted like latitude,longitude)
- zipcode 
- rowlabel (a unique id)

With all that out of the way, here's the code that adds this data source to Popily.

{% highlight python %}
import popily_api

popily = popily_api.Popily('YOUR API TOKEN')

# Grab the file on your machine
file_data = {'data': open('/path/to/the/file.csv')}

# Give it a title (optional)
title = 'DC Comics Data'

# Define the column types
columns = [
    {
        'column_header': 'name',
        'data_type': 'rowlabel'
    },
    {
        'column_header': 'ALIGN',
        'data_type': 'category'
    },
    {
        'column_header': 'EYE',
        'data_type': 'category'
    },
    {
        'column_header': 'HAIR',
        'data_type': 'category'
    },
    {
        'column_header': 'SEX',
        'data_type': 'category'
    }
]

source = popily.add_source(file_obj=file_data, 
                                columns=columns, title=title)
{% endhighlight %}

A couple things to point out:

1) In the example I uploaded a file. You can just as easily give Popily the URL of a file, or connect to a database. 

{% highlight python %}
# Get the data from a URL
url = 'https://raw.githubusercontent.com/fivethirtyeight/data/master/comic-characters/dc-wikia-data.csv'
source = popily.add_source(url=url, 
                                columns=columns, title=title)

# Or if your data is in a database in a 'characters' table
connection_string = 'mysql://username:password@host:port/database'
query = 'SELECT * FROM characters'
source = popily.add_source(
                        connection_string=connection_string, 
                        query=query, columns=columns, 
                        title=title)
{% endhighlight %}

2) You only need to define the columns you care about. Popily will ignore everything else.

3) We're exploring this data through the API, but once a data source has been added, you can also explore it through the Popily user interface. In the API response after you create a new data source, you'll see an `explore_path` property, which is where you'll find your new source via the interface.

{% highlight json %}
{
    "id": 432,
    "explore_path": "/explore/source/dc-wikia-data-csv-3/discoveries/"
}
{% endhighlight %}

So this source is available at `https://popily.com/explore/source/dc-wikia-data-csv-3/discoveries/`.

---

## Getting Charts

Now you can start retrieving cool interactive charts. For example let's say that we were interested in the relationship between gender and "alignment" (whether a character is good or bad). Just ask Popily for the relationship you're curious about based on the column names.

{% highlight python %}
insights = popily.get_insights(source['id'],
                                    columns=['ALIGN','SEX'])
{% endhighlight %}

This gives us a response that looks like this:

{% highlight json %}
{
    "count": 1,
    "previous": null,
    "results": [{
        "title": "Number of ALIGN grouped by SEX",
        "x_label": "ALIGN",
        "y_label": "Number of Records",
        "filters_key": "",
        "source": 429,
        "insight_type_category": "count",
        "id": 249985,
        "z_label": "",
        "insight_type": "count_by_category_by_category"
    }],
    "next": null
}
{% endhighlight %}

Don't worry about the `insight_type`, `insight_type_category` and `filters_key` properties, we'll get to those in a second. Now if we want to embed this insight into our application, we can ask for an embeddable URL based on the insight's id. 

{% highlight python %}
insight = popily.get_insight(249985)
{% endhighlight %}

This returns the following:

{% highlight json %}
{
    "embed_url": "https://popily.com/widget/chart/align-hair/-/?style=compact-500-500&app_id=18&sig=8b7c8e557a9ab1bcf4d5d762dda773d2&timestamp=1456113107",
    "x_label": "ALIGN",
    "y_label": "Number of Records",
    "filters_key": "",
    "source": 429,
    "title": "Number of ALIGN grouped by SEX",
    "insight_type_category": "count",
    "id": 249985,
    "z_label": "",
    "insight_type": "count_by_category_by_category"
}
{% endhighlight %}

---

## Displaying Data

If you want to visualize the data yourself you can get the x, y and z values (see the insight documentation for more details), or you can use the charts Popily generates for you via the insight's `embed_url` property. Just drop it in an iframe:

{% highlight html %}
<iframe frameborder="0" height="500" width="500" src="https://popily.com/widget/chart/hair-align-2/-/?style=compact-500-500&app_id=18&sig=63bb87883e20548076aefac271911c13&timestamp=1456114892"></iframe>
{% endhighlight %}

You'll see a chart that looks something like this:

<img style="display:inline" src="{{ site.baseurl }}/public/images/example-chart.png">

You can manually change the height and width in the returned URL by modifying the width and height values assigned to the `style` parameter (`style=compact-WIDTH-HEIGHT`), or you can pass these as parameters when you retrieve the insight. 

{% highlight python %}
popily.get_insight(249985,height=500,width=500)
{% endhighlight %}

Now you're ready to further customize the chart display, like changing the title, axis labels, category ordering, and pretty much whatever else you want.

{% highlight python %}
popily.customize_insight(249985,
                            title='This Chart is Awesome',
                            x_label='Goodness',
                            y_label='Genders',
                            category_order='z-a')
{% endhighlight %}

This changes will now immediately be reflected in our embedded visualizations.

<img style="display:inline" src="{{ site.baseurl }}/public/images/example-chart-2.png">

Check out the full documentation for all the available customization options. 

---

## Filtering Your Data

Sometimes you want to tell a more specific story. With Popily you can apply filters to any insight or list or insights, and then assign customizations that are specific to the filters you've applied. This is a little more clear in practice. Let's take the insight object we were working with before, with id `249985`. First we'll apply a filter to exclude male and female characters.

{% highlight python %}
filters = [
    {
        'column': 'SEX',
        'values': ['Genderless Characters','Transgender Characters']
    }
]
insight = popily.get_insight(249985, filters=filters)
{% endhighlight %}

<img style="display:inline" src="{{ site.baseurl }}/public/images/example-chart-3.png">

Now we can apply customizations to this insight that will be displayed _only when these filters are applied_. Cool!

{% highlight python %}
filters = [
    {
        'column': 'SEX',
        'values': ['Genderless Characters','Transgender Characters']
    }
]
insight = popily.customize_insight(249985, 
                title='Breakdown Without Male or Female Characters',
                filters=filters)
{% endhighlight %}

<img style="display:inline" src="{{ site.baseurl }}/public/images/example-chart-4.png">

---

## Next Steps

Using these basic ingredients, you can quickly add a few charts to a dashboard, or even build complex analytics applications -- giving your users the power to query, filter and share their data. Here are a couple tips to help you brainstorm:

**First**: You can set a schedule for Popily charts to update automatically, so your users will see updated visualizations whenever the data source changes -- whether you're updating a file on a server, creating new database records, or polling an API. Checkout the insight documentation for more details.

**Second**: Each `insight_type` represents a specific calculation that's only performed once for each combination of columns. So if you already know the structure of your data, you can retrieve a specific type of chart (like `count_by_category_category`), even if you don't know the insight id in advance.

{% highlight python %}
# This will only return 1 insight
insights = popily.get_insights(source['id'],
                columns=['SEX','ALIGN'],
                insight_type='count_by_category_by_category')

# So we can go right ahead and get the embed url
embed_url = popily.get_insight(insights['results'][0]['id'])['embed_url']
{% endhighlight %}

**Third**: We're here to help. Come hang out with us on Slack, or send an email to awesome@popily.com. 
