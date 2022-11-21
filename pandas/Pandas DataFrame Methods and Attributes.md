- `df.shape` gives no. of rows and no. of columns in the dataframe in a tuple
- `df.info()` gives no. of rows and no. of columns along with the datatypes of each column
  ![[Pasted image 20221101124157.png]]
  - to change no. of columns/rows to display in jupyter notebook output
```python
  pd.set_option('display.max_columns', 85) # will display 85 columns in the output
  pd.set_option('display.max_rows', 85) # will display 85 rows in the output
```