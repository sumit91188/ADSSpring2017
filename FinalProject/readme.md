[![N|Solid](http://i.quoteaddicts.com/media/quotes/3/102740-allstate-home-insurance-quote.jpg)](https://www.allstate.com/)
# Overview
Allstate provides customized insurance policies to its customers with personal attention and service. Marketing in insurance terms would mean to provide the perfect quote to your customer and reduce the quote processing time.
When a car owner inquires about the car-insurance plan, insurance agents propose several quotes with different coverage options. After considering all the possible options, car owner settles with a plan and purchases it.

# About the data
The data contains transaction history for customers that ended up purchasing a policy. For each customer_ID, we have given their quote history. In the dataset we have the entire quote history, the last row of which contains the coverage options they purchased.
### What is a customer?
Each customer has many shopping points, where a shopping point is defined by a customer with certain characteristics viewing a product and its associated cost at a particular time.
  - Some customer characteristics may change over time (e.g. as the customer changes or provides new information), and the cost depends on both the product and the customer characteristics.
  - A customer may represent a collection of people, as policies can cover more than one person.
  - A customer may purchase a product that was not viewed!

### Coverage options
Each product has 7 customizable options selected by customers, each with 2, 3, or 4 ordinal values possible. A policy is simply a vector with length 7 whose values are chosen from each of the options listed below. The cost of a product is a function of both the product options and customer characteristics:

[![N|Solid](https://kaggle2.blob.core.windows.net/competitions/kaggle/3788/media/product_options.png)](https://www.allstate.com/)

### Variable Descriptions
  - **customer_ID** - A unique identifier for the customer
  - **shopping_pt** - Unique identifier for the shopping point of a given customer
  - **record_type** - 0=shopping point, 1=purchase point
  - **day** - Day of the week (0-6, 0=Monday)
  - **time** - Time of day (HH:MM)
  - **state** - State where shopping point occurred
  - **location** - Location ID where shopping point occurred
  - **group_size** - How many people will be covered under the policy (1, 2, 3 or 4)
  - **homeowner** - Whether the customer owns a home or not (0=no, 1=yes)
  - **car_age** - Age of the customer’s car
  - **car_value** - How valuable was the customer’s car when new
  - **risk_factor** - An ordinal assessment of how risky the customer is (1, 2, 3, 4)
  - **age_oldest** - Age of the oldest person in customer's group
  - **age_youngest** - Age of the youngest person in customer’s group
  - **married_couple** - Does the customer group contain a married couple (0=no, 1=yes)
  - **C_previous** - What the customer formerly had or currently has for product option C (0=nothing, 1, 2, 3,4)
  - **duration_previous** -  how long (in years) the customer was covered by their previous issuer
  - **A,B,C,D,E,F,G** - the coverage options
  - **cost** - cost of the quoted coverage options

---
# Goal of the project
If the eventual purchase can be predicted sooner in the shopping window, the quoting process is shortened and the issuer is less likely to lose the customer's business.This application is likely be applied to any business in which customers are deciding between multiple products (or multiple options for the same product). If the business can gain an insight into which product or option a customer is likely to end up choosing, they could nudge the customer toward that product (in order to increase their conversion rate), or instead nudge the customer toward a slightly more expensive product (in order to maximize their profit from that sale).
* **Dockerizing an application** - [Docker File](https://github.com/sumit91188/deo_sumit_spring2017/blob/master/Final/Data%20Download.ipynb) describes the step by step implementation of creating an docker image.
* **Pipelining tasks** - [Start-Pipeline](https://github.com/sumit91188/deo_sumit_spring2017/blob/master/Final/Data%20Download.ipynb) uses Luigi to pipelining the tasks.
* **Data Download** - [Data-download](https://github.com/sumit91188/deo_sumit_spring2017/blob/master/Final/Data%20Download.ipynb) describes the step by step implementation of downloading the dataset provided on the kaggle.com.
* **Data Wrangling & Preprocessing** - [Clean Data Script](https://github.com/sumit91188/deo_sumit_spring2017/blob/master/Final/Data%20Download.ipynb) describe the step by step process to clean and seperate the data into 3 different files for purchases, quotes & last quotes.
* **Exploratory data analysis** - [EDA Notebook](https://github.com/sumit91188/deo_sumit_spring2017/blob/master/Final/analysis/ana_1.ipynb) performs Exploratory data analysis on the purchases and quotes.
Data scientist view of [Power BI dashboard](https://github.com/sumit91188/deo_sumit_spring2017/blob/master/Final/analysis/ana_1.ipynb) to illustrate your key insights.
* **Classification Of Coverages** - Classify all the 7 coverages based on the customer's profile.
* **Predicting Cost** - Based on the possible coverages and customer's profile, predict the cost, customer is likely to pay.
* **Create a WebApp for user-interaction** - [Allstate Insurance](https://github.com/sumit91188/deo_sumit_spring2017/blob/master/Final/analysis/ana_1.ipynb) webapp takes the inputes from a customer and provides the best possible quote, with the option of customizing it.

---
## Docker File
[Docker File](https://github.com/sumit91188/deo_sumit_spring2017/blob/master/Final/Data%20Download.ipynb) has following dependency.
```sh
FROM python
RUN apt-get update && \
    apt-get clean && \
            rm -rf /var/lib/apt/lists/*
```
### Installation of dependencies
```sh
RUN pip install 'pandas' \
		'numexpr' \
        'numpy' \
		'matplotlib' \
		'scipy' \
		'scikit-learn' \
		'beautifulsoup4' \
		'lxml' \
		'html5lib' \
		'boto' \
		'luigi' \
		'mechanicalsoup'
```
### Copying scripts from local to docker machine
```sh
ADD clean_data.py clean_data.py
ADD download_data.py download_data.py
ADD start_pipeline.py start_pipeline.py
```
## Luigi Pipeline
[Start Pipeline Script](https://github.com/sumit91188/deo_sumit_spring2017/blob/master/Final/Data%20Download.ipynb) description.
### Start Task
**Task Requirement**
```
def requires(self):
    return Clean_data()
```
**Task Output**
```
def output(self):
    return luigi.LocalTarget('logs/start_pipeline.log')
```
**Run Task**
```
#take AWS keys from user

#create connection to AWS
conn = boto.connect_s3(AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY)
bucket = conn.lookup(bucket_name)

#check if files are already present
if not (bucket == None):
    for key in bucket.list(delimiter='/'):
        for f in bucket.list(key.name):
            if(f.name == filePath):
                #files are already on AWS

#upload files on AWS
k = Key(bucket)
    k.key = file
    k.set_contents_from_filename(file)
```

### Clean_data Task
**Task Requirement**
```
def requires(self):
    return Download_data()
```
**Task Output**
```
def output(self):
    return{
    'output1' : luigi.LocalTarget('data/processed_data/purchase_data.csv'),\
    'output2' : luigi.LocalTarget('data/processed_data/quotes_data.csv'),\
    'output3' : luigi.LocalTarget('data/processed_data/last_quote_data.csv')
    }
```
**Run Task**
```
#run clean data script
```

### Download_data Task
**Task Requirement**
```
def requires(self):
    return None
```
**Task Output**
```
def output(self):
    return luigi.LocalTarget('luigi/download_data_completed.txt')
```
**Run Task**
```
#run download data script
```

## Data Download
[Data-download Script](https://github.com/sumit91188/deo_sumit_spring2017/blob/master/Final/Data%20Download.ipynb) description.
### Get login credentials
```
username = input("Please provide username : ")
password = input("Please provide password : ")
```
<img src ="extras/screenshots/login.PNG" />

### Loging to the site
```
login_page = browser.get(url)
login_form = login_page.soup.find('form', {"id":"signin"})
login_form.find("input", {"id" : "UserName"})["value"] = username
login_form.find("input", {"id" : "Password"})["value"] = password
login_form.find("input", {"id" : "RememberMe"})["value"] = True
response = browser.submit(login_form, login_page.url)
```

### Check if username & password are correct
```
if (response.url == 'https://www.kaggle.com/account/login.action'):
    print(username + " logged in successfully!")
    --download files--
else:
    print("Either username or password or both are wrong..")
```
### Download & unzip files
```
with urlopen(baseURL + extn) as zipresp:
    with ZipFile(BytesIO(zipresp.read())) as zfile:
        zfile.extractall(path)
```
---

## Data Wrangling & Preprocessing
[Clean Data Script](https://github.com/sumit91188/deo_sumit_spring2017/blob/master/Final/Data%20Download.ipynb) description.
### Fill missing data
Below function will fill all the NaNs with some default values.
```
def funFillMissingData(df):
    df['car_value'].fillna('z', inplace=True)
    df['C_previous'].fillna(df['C'], inplace=True)
    df['duration_previous'].fillna(0, inplace=True)
    return df
```
### Derive new columns
Below function will derive new features from the existing features.
```
def funDeriveColumns(df):
    df['plan'] = df['A'].map(str) + df['B'].map(str) + df['C'].map(str) + df['D'].map(str) + df['E'].map(str) + df['F'].map(str) + df['G'].map(str)
    df['hour'] = df['time'].dt.hour
    df['is_weekend'] = np.where(df['day'].isin(list(range(0, 4))) , 0, 1)
    df['is_family'] = np.where(((df['group_size'] > 2) & (df['age_youngest'] < 25) & (df['married_couple'] == 1)), 0, 1)
    df['agediff'] = df['age_oldest'] - df['age_youngest']
    df['is_individual'] = np.where(((df['agediff'] == 0) & (df['group_size'] == 1)), 1, 0)
    return df
```
### Change data types
Below function will change the data type of columns to category.
```
def funChangeDataTypes(df):
    df['state_code'] = df['state'].astype('category')
    df['state_code'] = df['state_code'].cat.codes
    df['car_value_code'] = df['car_value'].astype('category')
    df['car_value_code'] = df['car_value_code'].cat.codes

    columns = #columns to be channged

    for col in columns:
        df[col] = df[col].astype('category')
    return df
```
### Predict Risk Factor
In the raw data,we have found that almost in 33% of the rows, 'risk_factor' is missing. After carefully looking into the data, we came to the connclusion that,  instaed of filling these NaN by mean/max/min, we can predict it with the help of other available features.
We have used Random Forest Multiclass Classifier to classify the Risk Factor.
**Confusion Metric**
<img src ="extras/screenshots/CF_Risk_Factor.PNG" />

**Accuracy Score**
<img src ="extras/screenshots/accuracy_Risk_Factor.PNG" />
```
def funPredictNFillRiskFactor(df):
    dfrisk = df[~ pd.isnull(df['risk_factor'])]
    dfnorisk = df[pd.isnull(df['risk_factor'])]
    columns = ['location', 'car_age', 'age_oldest', 'age_youngest', 'state_code']
    train_X = dfrisk[columns]
    train_y = dfrisk['risk_factor']
    model = RandomForestClassifier(n_jobs=2)
    model.fit(train_X, train_y)
    test_X = dfnorisk[columns]
    dfnorisk['risk_factor'] = model.predict(test_X)
    test_X = dfnorisk[columns]
    dfnorisk['risk_factor'] = model.predict(test_X)
    df = pd.concat([dfrisk, dfnorisk])
    return df
```

### Write data to CSVs in chunks
```
def chunker(seq, size):
    return (seq[pos:pos + size] for pos in range(0, len(seq), size))
```
### Snapshot of cleaned data
<img src ="extras/screenshots/purchase_data.PNG" />


---

## Exploratory data analysis
### Installation
[EDA Notebook](https://github.com/sumit91188/deo_sumit_spring2017/blob/master/Final/analysis/ana_1.ipynb) has following dependency.

```sh
$ pip install pandas
$ pip install numpy
$ pip install seaborn
$ pip install plotly
```
### Number of shopping points
While working on exploratory data analysis, we found that more than 80% of purchases are finalized after considering more than 5 quotes. If the eventual purchase can be predicted sooner in the shopping window, the quoting process is shortened and the issuer is less likely to lose the customer's business.
```
#by shopping_pt and counting the total number of records
seriesCount = dfLastQuote['customer_ID'].groupby(dfLastQuote['shopping_pt']).count()
columns=['Number of Quotes Reviewed', 'Count of Customers']
dfSummary = pd.DataFrame({'Number of Quotes Reviewed':seriesCount.index,'Count of Customers': seriesCount})
```
<img src ="extras/screenshots/EDA1.PNG" />

### Predictive power of final quote before purchase
As seen in the plot below, the final quote a customer requests before the "purchase point" is hugely predictive of which options they will actually purchase. The final quote correctly predicted the purchased options 50% to 75% of the time, with that percentage steadily increasing as customers review more quotes. This proves that customer continues to take quotes unless and untill, he is 100% satisfied with the coverages & cost.
```
#by shopping_pt and counting the total number of records
# df = dfLastQuote[dfLastQuote['is_changed'] == 0]
seriesCount = dfLastQuote[dfLastQuote['is_changed'] == 0]['customer_ID'].groupby(dfLastQuote['shopping_pt']).count()
columns=['Number of Quotes Reviewed', 'Prediction Accuracy']
dfSummary1 = pd.DataFrame({'Number of Quotes Reviewed':seriesCount.index,'Prediction Accuracy': seriesCount})
```
<img src ="extras/screenshots/EDA2.png" />

### Summarizing data by States
  - Summary of purchases are displayed on the plot using plotly in offline mode.
  - Group the data by State and take average/mode of the required columns.
  - Create a dictionary of the summary and display on the plotly plot.
  - Hover over each state to get the summary.

<img src ="extras/screenshots/EDA3.png" />

### Power BI Dashboard
> Loan Data Summary


<img src ="extras/screenshots/NumberOfLoansByYear.PNG" />

---

## Azure Machine Learning
>We have created total 3 API from the Azure ML.

- To classify the risk factor.
- To classify all the 7 coverages.
- To predict the cost of the policy.

### Classify Risk Factor
**Staging**

[Stage_Classify_Risk_Factor](https://studio.azureml.net/Home/ViewWorkspaceCached/ae308c93a0db4b72bf3db66d7dd8ec34#Workspaces/Experiments/Experiment/ae308c93a0db4b72bf3db66d7dd8ec34.f-id.b58dc05493ac4aa59038079aa4c2f32d/ViewExperiment) is a staginng experiment to find out the best model for the classification.
We have compared the Multiclass Neural Network, Multiclass Descision Jungle & Multiclass Descision Forest for the risk factor classification.

<img src ="extras/screenshots/Azure1.PNG" />

Based on the below results we have decided to deploy the Random Forest model and create an API.

<img src ="extras/screenshots/Azure2.PNG" />

<img src ="extras/screenshots/Azure3.PNG" />

**Deployment**

[Classify_Risk_Factor](https://studio.azureml.net/Home/ViewWorkspaceCached/ae308c93a0db4b72bf3db66d7dd8ec34#Workspaces/Experiments/Experiment/ae308c93a0db4b72bf3db66d7dd8ec34.f-id.b58dc05493ac4aa59038079aa4c2f32d/ViewExperiment) is a Random Forest model to classify the Risk Factor.

<img src ="extras/screenshots/Azure4.PNG" />

Above experiment we have deployed as an [Classify_Risk_Factor](https://studio.azureml.net/Home/ViewWorkspaceCached/ae308c93a0db4b72bf3db66d7dd8ec34#Workspaces/WebServiceGroups/WebServiceGroup/e2ed8cc7f7514f00acadf06c57aba7a8/dashboard) API.

### Classify Coverages
**Staging**

[Stage_Classify_A](https://studio.azureml.net/Home/ViewWorkspaceCached/ae308c93a0db4b72bf3db66d7dd8ec34#Workspaces/Experiments/Experiment/ae308c93a0db4b72bf3db66d7dd8ec34.f-id.54c9eb809f2345beb9abb0009250f326/ViewExperiment) is a staginng experiment to find out the best model for the classification of coverage A.
We have compared the Multiclass Neural Network, Multiclass Descision Jungle & Multiclass Descision Forest for the risk factor classification.

<img src ="extras/screenshots/Azure5.PNG" />

Based on the below results we have decided to deploy the Multiclass descision jungle and create an API.

<img src ="extras/screenshots/Azure6.PNG" />

<img src ="extras/screenshots/Azure7.PNG" />

**Deployment**

[Classify_A](https://studio.azureml.net/Home/ViewWorkspaceCached/ae308c93a0db4b72bf3db66d7dd8ec34#Workspaces/Experiments/Experiment/ae308c93a0db4b72bf3db66d7dd8ec34.f-id.c362c8a5926e4bcbb5779a255264545d/ViewExperiment) is a Multiclass Descision Jungle model to classify the Risk Factor.

<img src ="extras/screenshots/Azure8.PNG" />

> Similarly we have deployed all the models for the coverages B-G.

All the above experiments for coverages A to G are deployed in only one [first_quote](https://studio.azureml.net/Home/ViewWorkspaceCached/ae308c93a0db4b72bf3db66d7dd8ec34#Workspaces/Experiments/Experiment/ae308c93a0db4b72bf3db66d7dd8ec34.f-id.29416ef2bf3c44a8a411d39404283f12/ViewExperiment) API. We have done this to predict the coverages based on other coverages and customer's profile.

### Predict Cost
**Staging**

[Stage_Predict_Cost](https://studio.azureml.net/Home/ViewWorkspaceCached/ae308c93a0db4b72bf3db66d7dd8ec34#Workspaces/Experiments/Experiment/ae308c93a0db4b72bf3db66d7dd8ec34.f-id.23bc533b01f9444ea9e41b2069457583/ViewExperiment) is a staginng experiment to find out the best model for the prediction of cost.
We have compared the Neural Network, Linear Regression & Descision Forest for the cost regression.

<img src ="extras/screenshots/Azure9.PNG" />

Based on the below results we have decided to deploy the descision forest regression and create an API.

<img src ="extras/screenshots/Azure10.PNG" />

**Deployment**

[predict_cost](https://studio.azureml.net/Home/ViewWorkspaceCached/ae308c93a0db4b72bf3db66d7dd8ec34#Workspaces/Experiments/Experiment/ae308c93a0db4b72bf3db66d7dd8ec34.f-id.03b71eda4fd1416490aa7146d71e6a41/ViewExperiment) is a Descision Forest Regression model to predict the cost.

<img src ="extras/screenshots/Azure11.PNG" />

Above experiment for the prediction of cost is deployed as [predict_cost](https://studio.azureml.net/Home/ViewWorkspaceCached/ae308c93a0db4b72bf3db66d7dd8ec34#Workspaces/WebServiceGroups/WebServiceGroup/610c3af7eb98496fa19563b62c296efb/dashboard) API.
