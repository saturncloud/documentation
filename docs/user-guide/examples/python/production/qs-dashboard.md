# Create a Dashboard with Panel


This Jupyter notebook uses _Panel_ to create a dashboard with Python. While the dashboard can be viewed at the bottom of this notebook by running all the cells, it can also be continuously hosted on Saturn using a _deployment_, allowing people who don't have access to the notebook to be able to see the dashboard. See the [Saturn Cloud docs](https://saturncloud.io/docs/user-guide/examples/dashboards/dashboard/) for instructions on how to deploy it.

## Dashboard code
First, we import the packages, load the data, and do minor cleaning of it:


```python
import numpy as np
import pandas as pd
import hvplot.pandas  # noqa
import panel as pn
import urllib.request

pickup_by_zone = pd.read_csv(
    "https://saturn-public-data.s3.us-east-2.amazonaws.com/examples/dashboard/pickup_grouped_by_zone.csv"
)
pickup_by_time = pd.read_csv(
    "https://saturn-public-data.s3.us-east-2.amazonaws.com/examples/dashboard/pickup_grouped_by_time.csv"
)
tip_timeseries = pd.read_csv(
    "https://saturn-public-data.s3.us-east-2.amazonaws.com/examples/dashboard/pickup_average_percent_tip_timeseries.csv"
)

tip_timeseries = tip_timeseries.set_index(tip_timeseries.pickup_datetime.astype(np.datetime64))

total_rides = pickup_by_zone.total_rides.sum()
total_fare = pickup_by_zone.total_fare.sum()
```

Next, we use define all the materials to go in the different parts of the dashboard:


```python
def kpi_box(title, color, value, unit=""):
    if value > 1e9:
        value /= 1e9
        increment = "B"
    elif value > 1e6:
        value /= 1e6
        increment = "M"
    elif value > 1e3:
        value /= 1e3
        increment = "K"
    else:
        increment = ""

    return pn.pane.Markdown(
        f"""
        ### {title}
        # {unit}{value :.02f} {increment}
        """,
        style={
            "background-color": "#F6F6F6",
            "border": "2px solid black",
            "border-radius": "5px",
            "padding": "10px",
            "color": color,
        },
    )


fares = kpi_box("Total Fares", "#10874a", total_fare, "$")
rides = kpi_box("Total Rides", "#7a41ba", total_rides)
average = kpi_box("Average Fare", "coral", (total_fare / total_rides), "$")


def heatmap(C, data, **kwargs):
    return data.hvplot.heatmap(
        y="pickup_weekday",
        x="pickup_hour",
        C=C,
        hover_cols=["total_rides"] if C == "average_fare" else ["average_fare"],
        yticks=[
            (0, "Mon"),
            (1, "Tues"),
            (2, "Wed"),
            (3, "Thur"),
            (4, "Fri"),
            (5, "Sat"),
            (6, "Sun"),
        ],
        responsive=True,
        min_height=200,
        colorbar=False,
        **kwargs,
    ).opts(toolbar=None, padding=0)


tip_heatmap = heatmap(
    data=pickup_by_time,
    C="average_percent_tip",
    cmap="coolwarm",
    clim=(12, 18),
    title="Average Tip %",
)

date_range_slider = pn.widgets.DateRangeSlider(
    name="Show between",
    start=tip_timeseries.index[0],
    end=tip_timeseries.index[-1],
    value=(tip_timeseries.index.min(), tip_timeseries.index.max()),
)

discrete_slider = pn.widgets.DiscreteSlider(
    name="Rolling window",
    options=["1H", "2H", "4H", "6H", "12H", "1D", "2D", "7D", "14D", "1M"],
    value="1D",
)


def tip_plot(xlim, window):
    data = tip_timeseries.rolling(window).mean()
    return data.hvplot(
        y="percent_tip", xlim=xlim, ylim=(10, 18), responsive=True, min_height=200
    ).opts(toolbar="above")


tip_timeseries_plot = pn.pane.HoloViews(tip_plot(date_range_slider.value, discrete_slider.value))


def trim(target, event):
    target.object = tip_plot(event.new, discrete_slider.value)


def roll(target, event):
    target.object = tip_plot(date_range_slider.value, event.new)


discrete_slider.link(tip_timeseries_plot, callbacks={"value": roll})
date_range_slider.link(tip_timeseries_plot, callbacks={"value": trim})

with urllib.request.urlopen(
    "https://saturn-public-data.s3.us-east-2.amazonaws.com/examples/dashboard/pickup_map.html"
) as f:
    pickup_map = pn.pane.HTML(f.read().decode("utf-8"), min_height=500, min_width=500)

dashboard_intro = """
# NYC Taxi Data

This dashboard demonstrates one mechanism for displaying summary statistics of
the [NYC Taxi Dataset](https://www1.nyc.gov/site/tlc/about/tlc-trip-record-data.page).
This particular page uses data from 2017 to 2020.

## Why Use Dashboards?

Dashboards provide a simple alternative to notebooks that can be more easily digested
by less technical audiences. A mixture of visualizations, text, and tables lets the reader
explore the data in a guided manner without having to write code.
"""

about_saturn = """
## Deploying in Saturn Cloud

This example uses [Panel](https://panel.holoviz.org) to create a deployable interactive dashboard.
In Saturn it is equally easy to create a dashboard using any of the other popular dashboarding
libraries such as: voila, dash, and bokeh. Learn more about how to deploy models and dashboards
in our [documentation](https://saturncloud.io/docs/concepts/projects/deployments).
"""

with urllib.request.urlopen(
    "https://saturn-public-data.s3.us-east-2.amazonaws.com/examples/dashboard/logo.svg"
) as f:
    with open("logo.svg", "wb") as g:
        g.write(f.read())
logo = pn.pane.SVG("logo.svg", style={"float": "right"})
```

Finally, we create the actual dashboard and load the newly created components into different parts of it. The last step is to serve up the dashboard as well:


```python
dashboard = pn.GridSpec(
    name="dashboard", sizing_mode="stretch_both", min_width=800, min_height=600, max_height=850
)

dashboard[0:5, :3] = pn.Column(dashboard_intro, tip_heatmap, about_saturn)
dashboard[0, 3] = fares
dashboard[0, 4] = rides
dashboard[0, 5] = average
dashboard[0, 6] = logo
dashboard[1:5, 3:6] = pickup_map
dashboard[5:8, 0:2] = pn.Column(
    date_range_slider,
    discrete_slider,
    "*Use widgets to control rolling window average on the timeseries plot or and to restrict to between certain dates*",
)
dashboard[5:7, 2:6] = tip_timeseries_plot

dashboard.servable(title="Saturn Taxi")
```

Now you've created a dashboard! To deploy it, follow the steps in the [Saturn Cloud docs](https://saturncloud.io/docs/user-guide/examples/dashboards/dashboard/).
