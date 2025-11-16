---
jupytext:
  custom_cell_magics: kql
  text_representation:
    extension: .md
    format_name: myst
kernelspec:
  display_name: Python 3 (ipykernel)
  language: python
  name: python3
language_info:
  name: python
  pygments_lexer: ipython3
  nbconvert_exporter: python
---

# grouping by period and category

+++

before starting this TP:
- put the `2-14-pandas-TP-event-nb.md` file in your github repository `p25-numerique-groupe8`  
- put the file `events.csv` and the file `country.csv` in its `data` folder 
- put the files `result-color-w.png`, `result-color-m.png`, `result-color-y.png`,  
   `result-bw-w.png`, `result-bw-m.png` and `result-bw-y.png` in its `media` folder

+++

in this TP we work on 

- data that represents *periods* and not just one timestamp
- checking for overlaps
- grouping by period (week, month, year..)
- then later on, grouping by period *and* category
- and some simple visualization tools

+++

here's an example of the outputs we will obtain

````{grid} 3 3 3 3
```{image} media/result-color-w.png
```
```{image} media/result-color-m.png
```
```{image} media/result-color-y.png
```
````

+++

## imports

```{code-cell} ipython3
:lines_to_next_cell: 2

import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
```

## the data

we have a table of events, each with a begin (`beg`) and `end` time; in addition each is attached to a `country`  
(we do not yet know what these events are)

```{code-cell} ipython3
events = pd.read_csv("data/events.csv")
events.head(10)
```

### adapt the type of each columns

surely the columns dtypes need some care
hints:
1. use the the `datetime` formats described here https://docs.python.org/3/library/datetime.html#strftime-and-strptime-behavior
2. or find and use the international format described https://en.wikipedia.org/wiki/ISO_8601

```{code-cell} ipython3
events["beg"] =pd.to_datetime(events["beg"])
events["end"] =pd.to_datetime(events["end"])
```

```{code-cell} ipython3
# check it

events.dtypes
```

### raincheck

check that the data is well-formed, i.e. **the `end`** timestamp **happens after `beg`**

```{code-cell} ipython3
if (events["end"]<= events['beg']).any():
    print ("error")
else:
    print ("valid")

#on precise any car sinon message d'erreur impossible de comparer 
```

sort the dataframe by the `'beg'` column

+++

### are there any overlapping events ?

+++

check if there are overlapping events  

hints:
1. you can use the `pandas.Series.shift` method  
  (i.e. the method `shift` applied to a `pandas` `Series`)
2. there is a `pandas.Timedelta` function

```{code-cell} ipython3
#on trie d'abord les evenements dans l'ordre du debut, ensuite on cree une colonne qui nous donne la date de fin de l'evenement de la ligne precedente 
#et on compare si celle ci est avant la fin de l'evenement de cette ligne

events = events.sort_values("beg")
end_prev = events["end"].shift(1)
```

```{code-cell} ipython3
if (events["beg"] < end_prev).any():
    print("overlap")
    print(events[overlap_mask])
else:
    print("no overlap")
```

### timespan

What is the timespan covered by the dataset (**earliest** and **latest** events, and **duration** in-between) ?

```{code-cell} ipython3
earliest=events['beg'].min() 
latest= events['end'].max() 
timespan=latest-earliest
```

### aggregated duration

so, given that there is no overlap, we can assume this corresponds to "reservations" attached to a unique resource  
write a code that computes the **overall reservation time**, as well as the **average usage ratio** over the overall timespan

keep a column with the duration of each event

```{code-cell} ipython3
events["duration"] = events["end"] - events["beg"]
overall_reservation_time = events["duration"].sum()

av_usage_ratio = overall_reservation_time / timespan

overall_reservation_time, av_usage_ratio
```

## visualization - grouping by period

### usage by period

grouping by periods: by week, by month or by year, plot the `bar` of the **total duration in that period**

There are at least 2 options to do this grouping, based on `resample()` and `to_period()` : **write them both**

(you can access the `dt` (datetime) attibuts and methods of a column if needed)


`````{admonition} for now, **just get the grouping right (do not improve your plots yet)**
:class: dropdown

you should produce something like e.g.

````{grid} 3 3 3 3
```{image} media/result-bw-w.png
```
```{image} media/result-bw-m.png
```
```{image} media/result-bw-y.png
```
````
we'll make cosmetic improvements below, and [the final results look like this](#label-events-output), but let's not get ahead of ourselves
`````

```{code-cell} ipython3
# methode 1 avec 

events = events.reset_index()      # remet 'beg' comme colonne car si je run le programme deux fois ca bug

events = events.sort_values("beg").set_index("beg")
# la colonne beg devient l'index de events  et on a trié la colonne beg du pus ancien au plus recent evenement
month = events["duration"].resample("M").sum() #on resample l'utilisation par mois et par semaine
week= events["duration"].resample("w").sum()
year=events["duration"].resample("Y").sum()
month.plot(kind="bar")
```

```{code-cell} ipython3
week.plot(kind="bar")
```

### improve the title and bottom ticks

add a title to your visualisations

also, and particularly relevant in the case of the per-week visu, we don't get to read **the labels on the horizontal axis**, because there are **too many of them**  
to improve this, you can use matplotlib's `set_xticks()` function; you can either figure out by yourself, or read the few tips below

````{admonition} a few tips
:class: dropdown tip

- the object that receives the `set_xticks()` method is an instance of `Axes` (one X&Y axes system),  
  which is not the figure itself (a figure may contain several Axes)  
  ask google or chatgpt to find the way you can spot the `Axes` instance in your figure
- it is not that clear in the docs, but all you need to do is to pass `set_xticks` a list of *indices* (integers)  
  i.e. if you have, say, a hundred bars, you could pass `[0, 10, 20, ..., 100]` and you will end up with one tick every 10 bars.
- there are also means to use smaller fonts, which may help see more relevant info
````

```{code-cell} ipython3
# let's say as arule of thumb
LEGEND = {
    'W': "week",
    'M': "month",
    'Y': "year",
}

SPACES = {
    'W': 12,   # in the per-week visu, show one tick every 12 - so about one every 3 months
    'M': 3,    # one every 3 months
    'Y': 1,    # on all years
}
```

```{code-cell} ipython3
plots = {
    'W': week,
    'M': month,
    'Y': year
}

for plot in plots:
    fig, ax = plt.subplots()
    plots[plot].plot(kind = "bar", label = LEGEND[plot])
    ax.set_xticks(range(0, len(plots[plot]), SPACES[plot]))
    ax.legend()
    plt.show()
```

### a function to convert to hours

you are to write a function that converts a `pd.Timedelta` into a number of hours  
1. read and understand the test code for the details of what is expected
2. use it to test your own implementation

note that if an hour has started even by one second, **it is counted**

```{code-cell} ipython3
:lines_to_next_cell: 1

# your code
import numpy as np
# the type of timedelta is pd.Timedelta
# the function returns an int
def convert_timedelta_to_hours(timedelta: pd.Timedelta) -> int:
    secondes=timedelta.total_seconds()
    return np.ceil(secondes/3600)
```

```{code-cell} ipython3
# test it

# if an hour has started even by one second, it is counted
test_cases = ( 
    # input in seconds, expected result in hours
    (0, 0), 
    (1, 1),     (3599, 1),     (3600, 1), 
    (3601, 2),  (7199, 2),     (7200, 2), 
    # 2 hours + 1s -> 3 hours
    (7201, 3),  
    # 3 hours + 2 minutes -> 4 hours
    (pd.Timedelta(3, 'h') + pd.Timedelta(2, 'm'), 4),
    # 2 days -> 48 hours
    (pd.Timedelta(2, 'D'), 48),
)

def test_convert_timedelta_to_hours():
    for seconds, exp in test_cases:
        # convert into pd.Timedelta if not already one
        if not isinstance(seconds, pd.Timedelta):
            timedelta = pd.Timedelta(seconds=seconds)
        else:
            timedelta = seconds
        # compute and compare
        got = convert_timedelta_to_hours(timedelta)
        print(f"with {timedelta=} we get {got} and expected {exp} -> {got == exp}")

test_convert_timedelta_to_hours()


#ca marche !
```

```{code-cell} ipython3
:lines_to_next_cell: 2

# for debugging; this should return 48

convert_timedelta_to_hours(pd.Timedelta(2, 'D'))
```

### use it to display totals in hours

keep the same visu, but display **the Y axis in hours**  
btw, what was the unit in the graphs above ?

hint:  
you can use `map` to apply a function (for example `convert_timedelta_to_hours`) to a `pandas.Series`

```{code-cell} ipython3
plots_hours = {p: plots[p].map(convert_timedelta_to_hours) for p in plots}

for plot in plots_hours:
    fig, ax = plt.subplots()
    plots_hours[plot].plot(kind="bar", label=LEGEND[plot], ax=ax)
    ax.set_ylabel("Hours")
    ax.set_xticks(range(0, len(plots_hours[plot]), SPACES[plot]))
    ax.legend()
    plt.show()
```

## grouping by period and region

the following table allows you to map each country into a region

```{code-cell} ipython3
# load it

countries = pd.read_csv("data/countries.csv")
countries.head(3)
```

### a glimpse on regions

what's the most effective way to see how many regions and how many countries per region we have ?

```{code-cell} ipython3
print(f"There are {countries['region'].nunique()} unique regions")
print(countries['region'].value_counts())
```

### attach a region to each event

your mission is to now show the same graphs, but we want to reflect the relative usage of each region, so we want to [split each bar into several colors, one per region see expected result below](#label-events-output)

+++

most likely your first move is to tag all events with a `region` column

remember that you can `pandas.merge` two data frames

```{code-cell} ipython3
events = events.reset_index()
corresponding = pd.merge( events, countries, left_on = "country", right_on = "name").drop(columns = ["name"])  #on utilise merge
```

### visu by period by region

you can now produce [the target figures, again they look like this](#label-events-output)

remember that missing values can be filled with the `fillna` method and that timedelta can be computed with `pandas.Timedelta`

```{code-cell} ipython3
times = ["W", "M", "Y"]

for time in times:

    corresponding["time"] = corresponding["beg"].dt.to_period(time)

    corresponding["duration_hours"] = corresponding["duration"].map(convert_timedelta_to_hours)

    events_tag = corresponding.pivot_table(
        index="time",
        values="duration_hours",
        columns="region",
        aggfunc="sum"
    ).fillna(0)

    ax = events_tag.plot.bar(
        title=f" Duration in hours{LEGEND[time]}",
        stacked=True,
    )

    ax.set_ylabel("Hours")
    ax.set_xticks(range(0, len(events_tag), SPACES[time]))

    plt.show()
```

***
