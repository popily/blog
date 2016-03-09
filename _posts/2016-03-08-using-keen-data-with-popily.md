---
layout: post
title: Using Keen.io Data in Popily
author: Jonathon Morgan
---

[Keen.io](http://keen.io) is a fantastic platform for capturing and analyzing event data. Let's see how we can use Popily to automatically turn that data into explorable visualizations.<!--more-->

## Getting the Data

If you'd like to follow along, [here's a dataset of _actual UFO sighting events_](https://www.dropbox.com/s/su2zq2hm8b192ga/56d46faf59949a742e00aa52-My_First_Project-ufo_sightings-1457521846-3YGHCO?dl=0) that we'll be working with for this tutorial. It was created by [@planetsig](https://github.com/planetsig/ufo-reports), and I did a little processing to make the data more analytics-friendly before bulk uploading it to Keen. 

If you'd rather extract your own collection from Keen, here's the curl command I used to get my data back (as [outlined in the Keen docs](https://keen.io/docs/api/?shell#extract-to-csv-file)). 

{% highlight curl %}
curl https://api.keen.io/3.0/projects/YOUR-PROJECT-ID/queries/extraction \
  -H "Authorization: YOUR READ KEY" \
  -H 'Content-Type: application/json' \
  -d "{
    \"event_collection\": \"COLLECTION NAME\",
    \"timeframe\": \"previous_100_years\",
    \"email\": \"YOU@YOUREMAIL.COM\"
  }"
{% endhighlight %}

## Using the Popily UI

Easy enough. Now that you have the data, let's go to Popily. 

**Step 1**: From your profile, click "Add a Data Source"

<img style="margin:0px auto;border:1px solid #eee;" src="{{ site.baseurl }}/public/images/keen/step1.png">

**Step 2**: Choose the option to upload data from a file (because we're working with a CSV)

<img style="margin:0px auto;border:1px solid #eee;" src="{{ site.baseurl }}/public/images/keen/step-2.png">

**Step 3**: Correct any mistakes Popily made in guessing data types (no one's perfect)

<img style="margin:0px auto;border:1px solid #eee;" src="{{ site.baseurl }}/public/images/keen/step-3.png">

**Step 4**: Choose a column to start exploring!

<img style="margin:0px auto;border:1px solid #eee;" src="{{ site.baseurl }}/public/images/keen/step4.png">

You can keep choosing additional columns to view more specific visualizations. In this screenshot we've already selected the "keen.timestamp" column, and we're adding "shape" so we can see how reports of UFO shapes have changed over time. 

<img style="margin:0px auto;border:1px solid #eee;" src="{{ site.baseurl }}/public/images/keen/exploration.png">

In the list view we show a stripped down version of every visualiation so it's easier to understand at a glance. If you want more detail, just click on the visualization's title to learn more about it. For example, when we click on the first visualization returned for the "keen.timestamp" column, we discover that reported UFO sighting events started to increase dramatically in the 1990s. 

<img style="margin:0px auto;border:1px solid #eee;" src="{{ site.baseurl }}/public/images/keen/num-records.png">

The obvious conclusion is that [we are not alone](http://www.washingtontimes.com/news/2009/apr/21/astronaut-says-were-not-alone/?page=all), or maybe the [X-Files](https://en.wikipedia.org/wiki/Colonist_(The_X-Files)) was just a very popular TV show. (It's cool Mulder, we [want to believe](https://images.newrepublic.com/82a6d0770aeaafbae8f26bf40a822b9b79a5c412.png?w=800) too.)

## Using the Popily API

You can also add data and get embeddable visualizations directly from [Popily's API](http://developers.popily.com). (Email [awesome@popily.com](mailto:awesome@popily.com) if you're a developer and would like an API key.) Let's quickly walk through the above example using [our Python client](https://github.com/popily/popily-api) (we also have a [JavaScript client](https://github.com/popily/popily-js), and more languages on the way).

**Step 1**: Add a data source

{% highlight python %}
from popily_api import Popily

popily = Popily('YOUR API KEY')

columns = [
    {
        'column_header': 'city',
        'data_type': 'category'
    },
    {
        'column_header': 'state',
        'data_type': 'state'
    },
    {
        'column_header': 'country',
        'data_type': 'country'
    },
    {
        'column_header': 'shape',
        'data_type': 'category'
    },
    {
        'column_header': 'duration',
        'data_type': 'number'
    },
    {
        'column_header': 'latlng',
        'data_type': 'coords'
    },
    {
        'column_header': 'keen.timestamp',
        'data_type': 'datetime'
    }
]

source = popily.add_source(columns=columns)
{% endhighlight %}

For more information on adding sources, see our [API docs](http://developers.popily.com/#sources).

Now we can get a visualization showing the number of sightings over time, just like we did through the user interface. 

{% highlight python %}
insight = popily.get_insights(columns=['keen.timestamp'], 
                                insight_actions=['count'],
                                single=True)
print insight['embed_url']
{% endhighlight %}

You can use the `embed_url` property of the returned insight in an iframe, and coming soon you'll be able to append fully responsive, interactive visualizations directly to DOM elements with our new charting library. For more information about retrieving visualizations (that we call `insights`), check out the [insight section](http://developers.popily.com/#insights) of our API docs.

Of course that's just the beginning. With Popily you can combine data sources, automatically update your visualizations when the underlying data changes, connect directly to databases and API endpoints (like Keen's REST API!), and customize visualizations to your heart's content. Give us a shout at [awesome@popily.com](mailto:awesome@popily.com) or [join us on Slack](https://gentle-shore-82359.herokuapp.com/) if you have any questions, feature requests, or ideas for our [nerdy podcast about data and drinking](http://partiallyderivative.com).

