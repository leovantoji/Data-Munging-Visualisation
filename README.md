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
- 
