# Installation and Loading data

## Pandas Installation
- create python virtualenv
```bash
  python3 -m venv .env
  source .env/bin/active
```
- pip install pandas
- we are going to use **jupyter notebook** so, install jupyterlab
```bash
pip install pandas jupyterlab
```


## Downloading data
- download stackoverflow survey data 2019 [stack-overflow-developer-survey-2022.zip](https://info.stackoverflowsolutions.com/rs/719-EMH-566/images/stack-overflow-developer-survey-2019.zip)
```bash
wget https://info.stackoverflowsolutions.com/rs/719-EMH-566/images/stack-overflow-developer-survey-2019.zip
```
- extract the zip file to *data* folder
```bash
unzip stack-overflow-developer-servey-2019.zip -d data
```
- to start jupyter notebook run
```bash
jupyter notebook --no-browser
```
- open the URL shown in the output, after the running the command


## Loading data
- first, create a new **python3** file from the browser
- in the python file write
```python
import pandas as pd

pd.read_csv("<path to CSV file")
```

