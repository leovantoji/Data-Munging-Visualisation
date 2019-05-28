# Kaggle Data Munging:
## [Pandas](https://www.kaggle.com/learn/pandas)
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

# Kaggle Data Visualization: 
## [From Non-Coder to Coder Micro-Course](https://www.kaggle.com/learn/data-visualization-from-non-coder-to-coder)
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
