# Data Munging
## [Translate SQL query to Pandas](https://medium.com/jbennetcodes/how-to-rewrite-your-sql-queries-in-pandas-and-more-149d341fc53e)

## [Kaggle Microcourse: Pandas](https://www.kaggle.com/learn/pandas)
### Creating, Reading and Writing
- A **DataFrame** is a table. It contains an array of individual entries, each of which has a certain value. Each entry corresponds with a *row* (or record) and a *column*.
- The dictionary-list constructor assigns values to the column labels, but just uses an ascending count from *0 (0, 1, 2, 3, ...)* for the *row labels*. Sometimes this is OK, but oftentimes we will want to assign these labels ourselves. The list of row labels used in a DataFrame is known as an **Index**. We can assign values to it by using an `index` parameter in our constructor: 
  ```python
  pd.DataFrame({"Bob": ["I liked it.", "It was awful."], "Sue": ["Pretty good.", "Bland."]}, index=["Product A", "Product B"])
  pd.DataFrame([["I liked it.", "Pretty good."], ["It was awful.", "Bland."]], columns=["Bob", "Sue"], index=["Product A", "Product B"])
  ```
- A **Series**, by contrast, is a sequence of data values. If a DataFrame is a table, a Series is a list. And in fact you can create one with nothing more than a list. A Series is, in essence, a single column of a DataFrame. So you can assign *row labels* to the Series the same way as before, using an `index` parameter. However, a Series do not have a column name, it only has one overall `name`.
- Reading data:
  ```python
  # Read from a CSV file
  data_csv = pd.read_csv(filepath, index_col=0)
  
  # Read from an Excel file
  data_excel = pd.read_excel(filepath, sheet_name="...")
  
  # Read from a SQL database
  # Kaggle only supports SQLite
  import sqlite3
  conn = sqlite3.connect("file_name.sqlite")
  data_sql = pd.read_sql_query("SELECT * from table_name", conn)
  ```
- Writing data:
  ```python
  # Write a CSV file
  data_csv.to_csv("file_name.csv")
  
  # Write an Excel file
  data_excel("file_name.xlsx", sheet_name="sheet_name")
  
  # Output to a SQL database
  conn = sqlite3.connect("file_name.sqlite")
  data_sql.to_sql("table_name", conn)
  ```

### Indexing, Selecting and Assigning
- To obtain a specific entry (corresponding to column `column` and row `i`) in a DataFrame table, we can call `table.column.iloc[i]`. Remember that Python indexing starts at `0`.
- To obtain a specific row of a DataFrame, we can use the `iloc` operator. 
- Select the records with index labels `1`, `2`, `3`, `5`, and `8`, assigning the result to the variable `sample_data`.
  ```python
  indices = [1, 2, 3, 5, 8]
  sample_data = df.loc[indices]
  sample_data = df.iloc[indices]
  ```
- Create a variable `df` containing the `country`, `province`, `region_1`, and `region_2` columns of the records with the index labels `0`, `1`, `10`, and `100`. `iloc` uses the Python stdlib indexing scheme, where the first element of the range is included and the last one excluded. So `0:10` will select entries `0,...,9`. `loc`, meanwhile, indexes inclusively. So `0:10` will select entries `0,...,10`
  ```python
  cols = ['country', 'province', 'region_1', 'region_2']
  indices = [0, 1, 10, 100]
  df = reviews.loc[indices, cols]
  ```
- Create a variable `df` containing the `country` and `variety` columns of the first 100 records.
  ```python
  indices = range(100)
  df = reviews.loc[indices, ["country", "variety"]]
  
  cols = ['country', 'variety']
  df = reviews.loc[:99, cols]
  
  cols_idx = [0, 11]
  df = reviews.iloc[:100, cols_idx]
  ```
- Create a DataFrame `italian_wines` containing reviews of wines made in `Italy`.
  ```python
  italian_wines = reviews[reviews["country"] == "Italy"]
  ```
- Create a DataFrame `top_oceania_wines` containing all reviews with at least 95 points (out of 100) for wines from Australia or New Zealand.
  ```python
  top_oceania_wines = reviews.loc[(reviews.country.isin(["Australia", "New Zealand"])) & (reviews.points >= 95)]
  ```

### Summary Function and Map
- Dataset: `winemag-data-130k-v2.csv`
- What is the median of the `points` column in the `reviews` DataFrame?
  ```python
  median_points = reviews.points.median()
  ```
- What countries are represented in the dataset?
  ```python
  countries = reviews.country.unique()
  ```
- How often does each country appear in the dataset?
  ```python
  reviews_per_country = reviews.country.value_counts()
  ```
- Create variable `centered_price` containing a version of the `price` column with the mean price subtracted. This **centering transformation** is a common preprocessing step before applying various machine learning algorithms.
  ```python
  centered_price = reviews.price - reviews.price.mean()
  ```
- Create a variable `bargain_wine` with the title of the wine with the highest points-to-price ratio in the dataset.
  ```python
  # Method 1
  reviews.points_to_price = reviews.points / reviews.price
  best_bargain = reviews.points_to_price.max()
  bargain_wine = reviews.loc[reviews.points_to_price == best_bargain].title.iloc[0]
  
  # Method 2
  bargain_idx = (reviews.points / reviews.price).idmax()
  bargain_wine = reviews.loc[bargain.idx, "title"]
  ```
- There are only so many words you can use when describing a bottle of wine. Is a wine more likely to be "tropical" or "fruity"? Create a Series `descriptor_counts` counting how many times each of these two words appears in the `description` column in the dataset.
  ```python
  n_trop = reviews.description.map(lambda desc: "tropical" in desc).sum()
  n_fruity = reviews.description.map(lambda desc:: "fruity" in desc).sum()
  descriptor_counts = pd.Series([n_trop, n_fruity], index=["tropical", "fruity"])
  ```
- We'd like to host these wine reviews on our website, but a rating system ranging from 80 to 100 points is too hard to understand - we'd like to translate them into simple star ratings. A score of 95 or higher counts as 3 stars, a score of at least 85 but less than 95 is 2 stars. Any other score is 1 star. Also, the Canadian Vintners Association bought a lot of ads on the site, so any wines from Canada should automatically get 3 stars, regardless of points. Create a series `star_ratings` with the number of stars corresponding to each review in the dataset.
  ```python
  def star_ratings_of_row(row):
    stars = 3
    if row.country != "Canada": 
        if (85 <= row.points < 95): stars = 2
        else: stars = 3
    
    return stars
  
  star_ratings = reviews.apply(star_ratings_of_row, axis=1)
  ```

### Grouping and Sorting
- Who are the most common wine reviewers in the dataset? Create a `Series` whose index is the `taster_twitter_handle` category from the dataset, and whose values count how many reviews each person wrote.
  ```python
  # Method 1
  reviews_written = reviews.groupby(["taster_twitter_handle"]).size()

  # Method 2
  reviews_written = reviews.groupby(["taster_twitter_handle"]).taster_twitter_handle.count()
  ```
- What is the best wine I can buy for a given amount of money? Create a `Series` whose index is wine prices and whose values is the maximum number of points a wine costing that much was given in a review. Sort the values by price, ascending (so that `4.0` dollars is at the top and `3300.0` dollars is at the bottom).
  ```python
  best_rating_per_price = reviews.groupby(["price"]).points.max().sort_index(ascending=True)
  ```
- What are the minimum and maximum prices for each `variety` of wine? Create a `DataFrame` whose index is the `variety` category from the dataset and whose values are the `min` and `max` values thereof.
  ```python
  price_extremes = reviews.groupby("variety").price.agg(["min", "max"])
  ```
- What are the most expensive wine varieties? Create a variable `sorted_varieties` containing a copy of the dataframe from the previous question where varieties are sorted in descending order based on minimum price, then on maximum price (to break ties).
  ```python
  sorted_varieties = price_extremes.sort_values(by=["min", "max"], ascending=False)
  ```
- Create a `Series` whose index is reviewers and whose values is the average review score given out by that reviewer. Hint: you will need the `taster_name` and `points` columns.
  ```python
  reviewer_mean_ratings = reviews.groupby(["taster_name"]).points.mean()
  ```
- What combination of countries and varieties are most common? Create a `Series` whose index is a `MultiIndex`of `{country, variety}` pairs. For example, a pinot noir produced in the US should map to `{"US", "Pinot Noir"}`. Sort the values in the `Series` in descending order based on wine count.
  ```python
  country_variety_counts = reviews.groupby(["country", "variety"]).size().sort_values(ascending=False)
  ```

### Data Types and Missing Data
- The data type for a column in a `DataFrame` or a `Series` is known as the `dtype`. `dtype` property can be used to grab the type of a specific column.
  ```python
  reviews.price.dtype
  ```
- The `dtypes` property returns the `dtype` of every column in the dataset.
  ```python
  reviews.dtypes
  ```
- Note that there isn't a `dtype` specific for strings; they instead have the `object` type. It's possible to convert a column type with the `astype` function. For example, we may transform the points column from its existing `int64` data type into a `float64` data type.
  ```python
  reviews.points.astype("float64")
  ```
- A `DataFrame` or `Series` index has its own `dtype`.
  ```python
  reviews.index.dtype
  ```
- Missing values are shown as `NaN`, short for "Not a Number". These `NaN` values are always of the `float64` dtype. `pandas` provides some methods specific to missing data. To select `NaN` entreis you can use `pd.isnull` (or its companion `pd.notnull`).
  ```python
  reviews[reviews.country.isnull()]
  ```
- Sometimes the price column is `null`. How many reviews in the dataset are missing a price?
  ```python
  missing_price_reviews = reviews[reviews.price.isnull()]
  n_missing_prices = len(missing_price_reviews)
  # Cute alternative solution: if we sum a boolean series, True is treated as 1 and False as 0
  n_missing_prices = reviews.price.isnull().sum()
  # or equivalently:
  n_missing_prices = pd.isnull(reviews.price).sum()
  ```
- Replacing missing values is a common operation.  `pandas` provides a really handy method for this problem: `fillna`. `fillna` provides a few different strategies for mitigating such data. For example, we can simply replace each `NaN` with an `"Unknown"`. `fillna` supports a few strategies for imputing missing values.
  ```python
  reviews.region_2.fillna("Unknown")
  ```
- The `replace` method is worth mentioning here because it's handy for replacing missing data which is given some kind of sentinel value in the dataset: things like `"Unknown"`, `"Undisclosed"`, `"Invalid"`, and so on.
  ```python
  reviews.taster_twitter_handle.replace("@kerinokeefe", "@kerino")
  ```

### Renaming and Combining
- Data can come with crazy column names or conventions. You'll use `pandas` renaming utility functions to change the names of the offending entries to something better. You can do this with the `rename` method. For example, you can change the `points` column to `score` like this.
  ```python
  reviews.rename(columns={"points": "score"})
  ```
- `rename` lets you rename index or column values by specifying a `index` or `column` keyword parameter, respectively. Python dict appears to be the most convenient one.
  ```python
  reviews.rename(index=dict(0: "first", 1: "second"))
  ```
- Renaming columns is very common, but renaming index values is very rarely. `set_index` is usually more convenient. Both the row index and the column index can have their own `name` attribute. The complimentary `rename_axis` method may be used to change these names.
  ```python
  reviews.rename_axis("wines", axis='rows').rename_axis("fields", axis='columns')
  ```
- When performing operations on a dataset we will sometimes need to combine different `DataFrame` and/or `Series` in non-trivial ways. There are **three core methods** for doing this. In order of increasing complexity, these are `concat`, `join`, and `merge`.'
- The simplest combining method is `concat`. Given a list of elements, it will smashes those elements together along an axis.
  ```python
  canadian_youtube = pd.read_csv("../input/youtube-new/CAvideos.csv")
  british_youtube = pd.read_csv("../input/youtube-new/GBvideos.csv")
  pd.concat([canadian_youtube, british_youtube])
  ```
- The `join` function combines DataFrame objects with a common index. For example, to pull down videos that happened to be trending on the same day in both Canada and the UK, you would write. The `lsuffix` and `rsuffix` parameters are necessary here because the data has the same column names in both British and Canadian datasets. If this wasn't true (because, say, we'd renamed them beforehand) we wouldn't need them.
  ```python
  left = canadian_youtube.set_index(['title', 'trending_date'])
  right = british_youtube.set_index(['title', 'trending_date'])

  left.join(right, lsuffix='_CAN', rsuffix='_UK')
  ```

### [Additional Stuffs](https://www.kaggle.com/sohier/tutorial-accessing-data-with-pandas/)
- It's a good practice to clean column names. By convention, the names should be converted to lower case. `pandas` is case sensitive, so future calls to all of the columns will need to be updated.
  ```python
  df.columns = [col.replace(" ", "_").lower() for col in df.columns]
  ```
- Select a subset of the data. Logical operators: `~` replaces `not`, `|` replaces `or`, and `&` replaces `and`. Multiple arguments should be wrapped in parentheses.
  ```python
  df[df.state == "UT"] # 1 condition
  df[(df.latitude > 50) | (df.acres > 10**6)].head() # Multiple conditions
  df[df.park_name.str.split().apply(lambda x: len(x)==3)].head() # Complicated expressions
  ```
- Key companion methods: `isin` and `isnull`. These methods make it much easier and faster to perform some very common tasks. Suppose we wanted to find all parks on the West coast. `isin` makes that simple.
  ```python
  df[df.state.isin(["WA", "OR", "CA"])].head()
  ```
- Less common methods: `pandas` offers many more indexing methods. You should probably stick to a few of them for the sake of keeping your code readable, but it's worth knowing they exist in case you need to read other people's code or have an unusual use case.
  - There are other ways to slice data with brackets. For the sake of readability, please don't use of them.
  - `.at` and `.iat`: like `.loc` and `.iloc` but much faster in exchange for only working on a single column and only returning a single result.
  - `.eval`: fast evaluation of a limited set of simple operators. `.query` works by calling this.
  - `.ix`: deprecated method that tried to determine if an index should be evaluated with `.loc` or `.iloc`. This led to a lot of subtle bugs! If you see this, you're looking at old code that won't work any more.
  - `.get`: like `.loc`, but will return a default value if the key doesn't exist in the index. Only works on a single column/series.
  - `.lookup`: Not recommended. It's in the documentation, but it's unclear if this is actually still supported.
  - `.mask`: like boolean indexing, but returns a dataframe/series of the same size as the original and anywhere that the boolean evaluates to `True` is set to `nan`.
  - `.query`: similar to boolean indexing. Faster for large dataframes. Only supports a restricted set of operations; don't use if you need `isnull()` or other dataframe methods.
  - `.take`: equivalent to `.iloc`, but can operate on either rows or columns.
  - `.where`: like boolean indexing, but returns a dataframe/series of the same size as the original and anywhere that the boolean evaluates to `False` is set to `nan`.
  - [Multi-indexing](http://pandas.pydata.org/pandas-docs/stable/user_guide/advanced.html): potentially useful for small to mid sized heirarchical datasets. Slow on larger datasets.

# Data Visualization: 
## [Kaggle Microcourse: From Non-Coder to Coder Micro-Course](https://www.kaggle.com/learn/data-visualization-from-non-coder-to-coder)
- **Trends** - A trend is defined as a pattern of change.
  - `sns.lineplot` - **Line charts** are best to show trends over a period of time, and multiple lines can be used to show trends in more than one group.
- **Relationship** - There are many different chart types that you can use to understand relationships between variables in your data.
  - `sns.barplot` - **Bar charts** are useful for comparing quantities corresponding to different groups.
  - `sns.heatmap` - **Heatmaps** can be used to find colour-coded patterns in tables of numbers.
  - `sns.scatterplot` - **Scatter plots** show the relationship between 2 continuous variables; if colour-coded, we can also show the relationship with a third categorical variable.
  - `sns.regplot` - Including a **regression line** in the scatter plot makes it easier to see any linear relationship between 2 variables.
  - `sns.lmplot` - This command is useful for drawing multiple regression lines, if the scatter plot contains multiple colour-coded groups.
  - `sns.swarmplot` - **Categorical scatter plot** show the relationship between a continuous variable and a categorical variable. Data points are separated in the chart.
  - `sns.stripplot` - Similar to `sns.swarmplot`. Data points can overlap in the chart.
- **Distribution** - We visualise distributions to show the possible values that we can expect to see in a varible, along with how likely they are.
  - `sns.distplot` - **Histograms** show the distribution of a single numerical variable.
  - `sns.kdeplot` - **KDE plots (Kernel Density Estimation)** (or **2D KDE plots**) show an estimated, smooth distribution of a single numerical variable (or two numerical variables).
  - `sns.jointplot` - This command is useful for simultaneously displaying a 2D KDE plot with the corresponding KDE plots for each individual variable.

## [Kaggle Microcourse: Data Visualisation](https://www.kaggle.com/learn/data-visualization)
### Univariate plotting with `pandas`
- Basic chart types:
  ```python
  df.plot.bar() # Bar Chart
  df.plot.line() # Line Chart
  df.plot.area() # Area Chart
  df.plot.hist() # Histogram
  df.plot.pie() # Pie Chart
  ```
- Wine-producing provinces of the world (category) to the number of labels of wines they produce (number).
  ```python
  reviews.province.value_counts().head(10).plot.bar() # Top 10 wine-producing provinces (absolute numbers)
  (reviews.province.value_counts().head(10) / len(reviews)).plot.bar() # Top 10 wine-produing provinces (%)
  (reviews.province.value_counts().head(10) / len(reviews)).plot.pie() # Top 10 wine-produing provinces (%)
  ```
- The number of reviews of a certain score allotted by Wine Magazine.
  ```python
  reviews.points.value_counts().sort_index().plot.bar()
  reviews.points.value_counts().sort_index().plot.line()
  reviews.points.value_counts().sort_index().plot.area()
  ```

### Bivariate plotting with `pandas`
- **Scatter plot** and **Hex plot**: Good for interval and some nominal categorical data.
- **Stacked bar chart**: Good for nominal and ordinal categorical data.
- **Bivariate line chart**: Good for ordinal categorical and interval data.
  ```python
  df.plot.scatter() # Scatter plot
  df.plot.hexbin() # Hex plot
  df.plot.bar(stacked=True) # Stacked bar chart
  df.plot.line() # Bivariate line chart
  ```
- A simple scatter plot simply maps each variable of interest to a point in two-dimensional space. Note that in order to make effective use of this plot, we had to **downsample** our data, taking just 100 points from the full set. This is because naive scatter plots do not effectively treat points which map to the same place. For example, if two wines, both costing 100 dollars, get a rating of 90, then the second one is overplotted onto the first one, and we add just one point to the plot.
  ```python
  reviews[reviews.price < 100].sample(100).plot.scatter(x="price", y="points")
  reviews[reviews.price < 100].plot.scatter(x="price", y="points") # shapeless blob
  ```
- Due to its weakness to overplotting, scatter plot works best with relatively small datasets, and with variables which have a large number of unique values.
- Ways to deal with overplotting.
  - Sampling the points.
  - Hexplot.
- A **hex plot** aggregates points in space into hexagons, and then colors those hexagons based on the values within them.
  ```python
  reviews[reviews["price"] < 100].plot.hexbin(x="price", y="points", gridsize=15)
  ```

### Styling your plots
- Under the hood, `pandas` data visualization tools are built on top of another, lower-level graphics library called `matplotlib`. Anything that you build in `pandas` can be built using `matplotlib` directly. `pandas` merely make it easier to get that work done. `matplotlib` *does* provide a way of adjusting the title size. Let's go ahead and do it that way, and see what's different.
  ```python
  fig = reviews.points.value_counts().sort_index().plot.bar(
    figsize=(12, 6), # Change figure size
    color='mediumvioletred', # Change color
    fontsize=16 # Change font size
  )
  fig.set_title("Rankings Given by Wine Magazine", fontsize=20) # Add chart title
  sns.despine(bottom=True, left=True) # Remove chart borders
  ```

### Subplotting
- Subplotting is a technique for creating multiple plots that live side-by-side in one overall figure. We can use the `subplots` method to create a figure with multiple subplots. `subplots` takes two arguments. The first one controls the number of *rows*, the second one the number of *columns*.
  ```python
  import matplotlib.pyplot as plt
  fig, axarr = plt.subplots(2, 1, figsize=(12, 8))
  ```
- When `pandas` generates a bar chart, behind the scenes here is what it actually does:
  - Generate a new `matplotlib` `Figure` object.
  - Create a new `matplotlib` `AxesSubplot` object, and assign it to the `Figure`.
  - Use `AxesSubplot` methods to draw the information on the screen.
  - Return the result to the user.
- In a similar way, our `subplots` operation above created one overall `Figure` with two `AxesSubplots` vertically nested inside of it. `subplots` returns two things, a figure (which we assigned to `fig`) and an array of the axes contained therein (which we assigned to `axarr`).
- To tell `pandas` which subplot we want a new plot to go in \- the first one or the second one \- we need to grab the proper axis out of the list and pass it into `pandas` via the `ax` parameter.
  ```python
  fig, axarr = plt.subplots(2, 1, figsize=(12,8))
  # Drawing with pandas
  reviews.points.value_counts().sort_index().plot.bar(ax=axarr[0])
  reviews.province.value_counts().head(20).plot.bar(ax=axarr[1])
  
  # Drawing with seaborn
  sns.countplot(reviews.points, ax=axarr[0])
  df = pd.DataFrame(reviews.groupby(["province"]).size().sort_values(ascending=False).head(20), columns=["count_province"])
  sns.barplot(y=df.index, x=df.count_province, data=df, ax=axarr[1])
  ```
- Why are subplots useful?
  - Create a large number of smaller charts probing one or a few specific aspects of the data.
  - Make attractive and informative panel displays.
  - **Subplots** are critically useful because they enable **faceting**, which is the act of breaking data variables up across multiple subplots, and combining those subplots into a single figure.

### Plotting with `seaborn`
- [`seaborn` Example Gallery](https://seaborn.pydata.org/examples/index.html)
- `seaborn` is a standalone data visualization package that provides many extremely valuable data visualizations in a single package. It is generally a much more powerful tool than pandas.
- The `pandas` bar chart becomes a `seaborn` `countplot`.
  ```python
  sns.countplot(reviews.points)
  ```
- **KDE**, short for "kernel density estimate", is a statistical technique for smoothing out data noise. It addresses an important fundamental weakness of a line chart: it will buff out outlier or "in-betweener" values which would cause a line chart to suddenly dip.
- KDE plots can also be used in 2 dimensions.
  ```python
  sns.kdeplot(reviews[reviews.price < 200].loc[:, ["price", "points"]].dropna().sample(5000))
  ```
- Bivariate KDE plots like this one are a great alternative to scatter plots and hex plots. They solve the same data overplotting issue that scatter plots suffer from and hex plots address, in a different but similarly visually appealing. However, note that bivariate KDE plots are very computationally intensive.
- The `seaborn` equivalent to a `pandas` histogram is the `distplot`.
  ```python
  sns.distplot(reviews['points'], bins=10, kde=False)
  ```
- To plot two variables against one another in `seaborn`, we use `jointplot`.
  ```python
  sns.jointplot(x="price", y="points", data=reviews[reviews.price < 100]) # Scatter plot
  sns.jointplot(x="price", y="points", data=reviews[reviews.price < 100], kind="hex", gridsize = 20) # Hex plot
  ```
- `seaborn` provides a `boxplot` and `violinplot` function. The center of the distributions shown above is the "box" in boxplot. The top of the box is the 75th percentile, while the bottom is the 25th percentile. The other part of the plot, the "whiskers", shows the extent of the points beyond the center of the distribution. Individual circles beyond that are outliers.
  ```python
  df = reviews[reviews.variety.isin(reviews.variety.value_counts().head().index)]
  sns.boxplot(x="variety", y="points", data=df)
  ```
- Boxplots are great for summarizing the shape of many datasets. They also don't have a limit in terms of numeracy: you can place as many boxes in the plot as you feel comfortable squeezing onto the page. Nonetheless, they only work for interval and nominal variables with a large number of possible values; they assume your data is roughly normally distributed (otherwise, their design doesn't make much sense); and they don't carry any information about individual values, only treating the distribution as a whole.
  ```python
  df = reviews[reviews.variety.isin(reviews.variety.value_counts().head().index)]
  sns.violinplot(x="variety", y="points", data=df)
  ```
- A `violinplot` cleverly replaces the box in the boxplot with a kernel density estimate for the data. It shows basically the same data, but is harder to misinterpret and much prettier than the utilitarian boxplot.

### Faceting with `seaborn`
- **Facet Grid**: Good for data with at least 2 categorical variables.
- **Pair Plot**: Good for exploring most kinds of data.
- **Faceting** is the act of breaking data variables up across multiple subplots, and combining those subplots into a single figure.
- The core `seaborn` utility for faceting is the `FacetGrid`. A `FacetGrid` is an object which stores some information on how you want to break up your data visualization.
- For example, suppose that we're interested in (as in the previous notebook) comparing strikers and goalkeepers in some way. To do this, we can create a `FacetGrid` with our data, telling it that we want to break the `Position` variable down by `col` (column). From there, we use the `map` object method to plot the data into the laid-out grid.
  ```python
  df = footballers[footballers.Position.isin(["ST", "GK"])]
  g = sns.FacetGrid(df, col="Position")
  g.map(sns.kdeplot, "Overall")
  ```
- `FacetGrid` comes equipped with a `col_wrap` parameter to control the maximum number of graphs appearing in any particular row.
  ```python
  g = sns.FacetGrid(df, col="Position", col_wrap=6)
  g.map(sns.kdeplot, "Overall)
  ```
- So far we've been dealing exclusively with one `col` (column) of data. The "grid" in `FacetGrid`, however, refers to the ability to lay data out by row *and* column. For example, suppose we're interested in comparing the talent distribution for (goalkeepers and strikers specifically, to keep things succinct) across rival clubs Real Madrid, Atlético Madrid, and FC Barcelona. As the plot below demonstrates, we can achieve this by passing `row=Position` and `col=Club` parameters into the plot.
  ```python
  df = footballers[footballers.Position.isin(["ST", "GK"])]
  df = df[df.Club.isin(["Real Madrid CF", "FC Barcelona", "Atlético Madrid"])]
  
  g = sns.FacetGrid(df, row="Position", col="Club")
  g.map(sns.violinplot, "Overall")
  ```
- `FacetGrid` orders the subplots effectively arbitrarily by default. To specify your own ordering explicitly, pass the appropriate argument to the `row_order` and `col_order` parameters.
  ```python
  g = sns.FacetGrid(df, row="Position", col="Club",
                    row_order=["GK", "SG"],
                    col_order=["Atlético Madrid", "FC Barcelona", "Real Madrid CF"])
  g.map(sns.violinplot, "Overall")
  ```
- In a nutshell, faceting is the easiest way to make your data visualization multivariate. Nonetheless, faceting does have some important limitations however. It can only be used to break data out across singular or paired categorical variables with very low numeracy \- any more than five or so dimensions in the grid, and the plots become too small (or involve a lot of scrolling). Additionally it involves choosing (or letting Python) an order to plot in, but with nominal categorical variables that choice is distractingly arbitrary.
- `pairplot` is a very useful and widely used `seaborn` method for faceting *variables* (as opposed to *variable values*). You pass it a `pandas` `DataFrame` in the right shape, and it returns you a gridded result of your variable values. By default `pairplot` will return scatter plots in the main entries and a histogram in the diagonal.
  ```python
  sns.pairplot(footballers[["Overall, "Potential", "Value"]])
  ```
- **Pair Plots** are most useful when just starting out with a dataset, because they help contextualise relationships within it.
