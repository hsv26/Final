<p style="text-align:center">
    <a href="https://skills.network/?utm_medium=Exinfluencer&utm_source=Exinfluencer&utm_content=000026UJ&utm_term=10006555&utm_id=NA-SkillsNetwork-Channel-SkillsNetworkCoursesIBMDS0321ENSkillsNetwork865-2022-01-01" target="_blank">
    <img src="https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/assets/logos/SN_web_lightmode.png" width="200" alt="Skills Network Logo"  />
    </a>
</p>


# **SpaceX  Falcon 9 first stage Landing Prediction**


# Lab 1: Collecting the data


Estimated time needed: **45** minutes


In this capstone, we will predict if the Falcon 9 first stage will land successfully. SpaceX advertises Falcon 9 rocket launches on its website with a cost of 62 million dollars; other providers cost upward of 165 million dollars each, much of the savings is because SpaceX can reuse the first stage. Therefore if we can determine if the first stage will land, we can determine the cost of a launch. This information can be used if an alternate company wants to bid against SpaceX for a rocket launch. In this lab, you will collect and make sure the data is in the correct format from an API. The following is an example of a successful and launch.


![](https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBMDeveloperSkillsNetwork-DS0701EN-SkillsNetwork/lab_v2/images/landing_1.gif)


Several examples of an unsuccessful landing are shown here:


![](https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBMDeveloperSkillsNetwork-DS0701EN-SkillsNetwork/lab_v2/images/crash.gif)


Most unsuccessful landings are planned. Space X performs a controlled landing in the oceans. 


## Objectives


In this lab, you will make a get request to the SpaceX API. You will also do some basic data wrangling and formating. 

- Request to the SpaceX API
- Clean the requested data


----


## Import Libraries and Define Auxiliary Functions


We will import the following libraries into the lab



```python
# Requests allows us to make HTTP requests which we will use to get data from an API
import requests
# Pandas is a software library written for the Python programming language for data manipulation and analysis.
import pandas as pd
# NumPy is a library for the Python programming language, adding support for large, multi-dimensional arrays and matrices, along with a large collection of high-level mathematical functions to operate on these arrays
import numpy as np
# Datetime is a library that allows us to represent dates
import datetime

# Setting this option will print all collumns of a dataframe
pd.set_option('display.max_columns', None)
# Setting this option will print all of the data in a feature
pd.set_option('display.max_colwidth', None)
```

Below we will define a series of helper functions that will help us use the API to extract information using identification numbers in the launch data.

From the <code>rocket</code> column we would like to learn the booster name.



```python
# Takes the dataset and uses the rocket column to call the API and append the data to the list
def getBoosterVersion(data):
    for x in data['rocket']:
       if x:
        response = requests.get("https://api.spacexdata.com/v4/rockets/"+str(x)).json()
        BoosterVersion.append(response['name'])
```

From the <code>launchpad</code> we would like to know the name of the launch site being used, the logitude, and the latitude.



```python
# Takes the dataset and uses the launchpad column to call the API and append the data to the list
def getLaunchSite(data):
    for x in data['launchpad']:
       if x:
         response = requests.get("https://api.spacexdata.com/v4/launchpads/"+str(x)).json()
         Longitude.append(response['longitude'])
         Latitude.append(response['latitude'])
         LaunchSite.append(response['name'])
```

From the <code>payload</code> we would like to learn the mass of the payload and the orbit that it is going to.



```python
# Takes the dataset and uses the payloads column to call the API and append the data to the lists
def getPayloadData(data):
    for load in data['payloads']:
       if load:
        response = requests.get("https://api.spacexdata.com/v4/payloads/"+load).json()
        PayloadMass.append(response['mass_kg'])
        Orbit.append(response['orbit'])
```

From <code>cores</code> we would like to learn the outcome of the landing, the type of the landing, number of flights with that core, whether gridfins were used, wheter the core is reused, wheter legs were used, the landing pad used, the block of the core which is a number used to seperate version of cores, the number of times this specific core has been reused, and the serial of the core.



```python
# Takes the dataset and uses the cores column to call the API and append the data to the lists
def getCoreData(data):
    for core in data['cores']:
            if core['core'] != None:
                response = requests.get("https://api.spacexdata.com/v4/cores/"+core['core']).json()
                Block.append(response['block'])
                ReusedCount.append(response['reuse_count'])
                Serial.append(response['serial'])
            else:
                Block.append(None)
                ReusedCount.append(None)
                Serial.append(None)
            Outcome.append(str(core['landing_success'])+' '+str(core['landing_type']))
            Flights.append(core['flight'])
            GridFins.append(core['gridfins'])
            Reused.append(core['reused'])
            Legs.append(core['legs'])
            LandingPad.append(core['landpad'])
```

Now let's start requesting rocket launch data from SpaceX API with the following URL:



```python
spacex_url="https://api.spacexdata.com/v4/launches/past"
```


```python
response = requests.get(spacex_url)
```

Check the content of the response



```python
#print(response.content)
```

You should see the response contains massive information about SpaceX launches. Next, let's try to discover some more relevant information for this project.


### Task 1: Request and parse the SpaceX launch data using the GET request


To make the requested JSON results more consistent, we will use the following static response object for this project:



```python
static_json_url='https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBM-DS0321EN-SkillsNetwork/datasets/API_call_spacex_api.json'
```

We should see that the request was successfull with the 200 status response code



```python
response.status_code
```




    200



Now we decode the response content as a Json using <code>.json()</code> and turn it into a Pandas dataframe using <code>.json_normalize()</code>



```python
# Use json_normalize meethod to convert the json result into a dataframe
data = pd.json_normalize(response.json())
#data = data.json_normalize()
```

Using the dataframe <code>data</code> print the first 5 rows



```python
# Get the head of the dataframe
data.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>static_fire_date_utc</th>
      <th>static_fire_date_unix</th>
      <th>net</th>
      <th>window</th>
      <th>rocket</th>
      <th>success</th>
      <th>failures</th>
      <th>details</th>
      <th>crew</th>
      <th>ships</th>
      <th>capsules</th>
      <th>payloads</th>
      <th>launchpad</th>
      <th>flight_number</th>
      <th>name</th>
      <th>date_utc</th>
      <th>date_unix</th>
      <th>date_local</th>
      <th>date_precision</th>
      <th>upcoming</th>
      <th>cores</th>
      <th>auto_update</th>
      <th>tbd</th>
      <th>launch_library_id</th>
      <th>id</th>
      <th>fairings.reused</th>
      <th>fairings.recovery_attempt</th>
      <th>fairings.recovered</th>
      <th>fairings.ships</th>
      <th>links.patch.small</th>
      <th>links.patch.large</th>
      <th>links.reddit.campaign</th>
      <th>links.reddit.launch</th>
      <th>links.reddit.media</th>
      <th>links.reddit.recovery</th>
      <th>links.flickr.small</th>
      <th>links.flickr.original</th>
      <th>links.presskit</th>
      <th>links.webcast</th>
      <th>links.youtube_id</th>
      <th>links.article</th>
      <th>links.wikipedia</th>
      <th>fairings</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2006-03-17T00:00:00.000Z</td>
      <td>1.142554e+09</td>
      <td>False</td>
      <td>0.0</td>
      <td>5e9d0d95eda69955f709d1eb</td>
      <td>False</td>
      <td>[{'time': 33, 'altitude': None, 'reason': 'merlin engine failure'}]</td>
      <td>Engine failure at 33 seconds and loss of vehicle</td>
      <td>[]</td>
      <td>[]</td>
      <td>[]</td>
      <td>[5eb0e4b5b6c3bb0006eeb1e1]</td>
      <td>5e9e4502f5090995de566f86</td>
      <td>1</td>
      <td>FalconSat</td>
      <td>2006-03-24T22:30:00.000Z</td>
      <td>1143239400</td>
      <td>2006-03-25T10:30:00+12:00</td>
      <td>hour</td>
      <td>False</td>
      <td>[{'core': '5e9e289df35918033d3b2623', 'flight': 1, 'gridfins': False, 'legs': False, 'reused': False, 'landing_attempt': False, 'landing_success': None, 'landing_type': None, 'landpad': None}]</td>
      <td>True</td>
      <td>False</td>
      <td>None</td>
      <td>5eb87cd9ffd86e000604b32a</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>[]</td>
      <td>https://images2.imgbox.com/94/f2/NN6Ph45r_o.png</td>
      <td>https://images2.imgbox.com/5b/02/QcxHUb5V_o.png</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>[]</td>
      <td>[]</td>
      <td>None</td>
      <td>https://www.youtube.com/watch?v=0a_00nJ_Y88</td>
      <td>0a_00nJ_Y88</td>
      <td>https://www.space.com/2196-spacex-inaugural-falcon-1-rocket-lost-launch.html</td>
      <td>https://en.wikipedia.org/wiki/DemoSat</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>None</td>
      <td>NaN</td>
      <td>False</td>
      <td>0.0</td>
      <td>5e9d0d95eda69955f709d1eb</td>
      <td>False</td>
      <td>[{'time': 301, 'altitude': 289, 'reason': 'harmonic oscillation leading to premature engine shutdown'}]</td>
      <td>Successful first stage burn and transition to second stage, maximum altitude 289 km, Premature engine shutdown at T+7 min 30 s, Failed to reach orbit, Failed to recover first stage</td>
      <td>[]</td>
      <td>[]</td>
      <td>[]</td>
      <td>[5eb0e4b6b6c3bb0006eeb1e2]</td>
      <td>5e9e4502f5090995de566f86</td>
      <td>2</td>
      <td>DemoSat</td>
      <td>2007-03-21T01:10:00.000Z</td>
      <td>1174439400</td>
      <td>2007-03-21T13:10:00+12:00</td>
      <td>hour</td>
      <td>False</td>
      <td>[{'core': '5e9e289ef35918416a3b2624', 'flight': 1, 'gridfins': False, 'legs': False, 'reused': False, 'landing_attempt': False, 'landing_success': None, 'landing_type': None, 'landpad': None}]</td>
      <td>True</td>
      <td>False</td>
      <td>None</td>
      <td>5eb87cdaffd86e000604b32b</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>[]</td>
      <td>https://images2.imgbox.com/f9/4a/ZboXReNb_o.png</td>
      <td>https://images2.imgbox.com/80/a2/bkWotCIS_o.png</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>[]</td>
      <td>[]</td>
      <td>None</td>
      <td>https://www.youtube.com/watch?v=Lk4zQ2wP-Nc</td>
      <td>Lk4zQ2wP-Nc</td>
      <td>https://www.space.com/3590-spacex-falcon-1-rocket-fails-reach-orbit.html</td>
      <td>https://en.wikipedia.org/wiki/DemoSat</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>None</td>
      <td>NaN</td>
      <td>False</td>
      <td>0.0</td>
      <td>5e9d0d95eda69955f709d1eb</td>
      <td>False</td>
      <td>[{'time': 140, 'altitude': 35, 'reason': 'residual stage-1 thrust led to collision between stage 1 and stage 2'}]</td>
      <td>Residual stage 1 thrust led to collision between stage 1 and stage 2</td>
      <td>[]</td>
      <td>[]</td>
      <td>[]</td>
      <td>[5eb0e4b6b6c3bb0006eeb1e3, 5eb0e4b6b6c3bb0006eeb1e4]</td>
      <td>5e9e4502f5090995de566f86</td>
      <td>3</td>
      <td>Trailblazer</td>
      <td>2008-08-03T03:34:00.000Z</td>
      <td>1217734440</td>
      <td>2008-08-03T15:34:00+12:00</td>
      <td>hour</td>
      <td>False</td>
      <td>[{'core': '5e9e289ef3591814873b2625', 'flight': 1, 'gridfins': False, 'legs': False, 'reused': False, 'landing_attempt': False, 'landing_success': None, 'landing_type': None, 'landpad': None}]</td>
      <td>True</td>
      <td>False</td>
      <td>None</td>
      <td>5eb87cdbffd86e000604b32c</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>[]</td>
      <td>https://images2.imgbox.com/6c/cb/na1tzhHs_o.png</td>
      <td>https://images2.imgbox.com/4a/80/k1oAkY0k_o.png</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>[]</td>
      <td>[]</td>
      <td>None</td>
      <td>https://www.youtube.com/watch?v=v0w9p3U8860</td>
      <td>v0w9p3U8860</td>
      <td>http://www.spacex.com/news/2013/02/11/falcon-1-flight-3-mission-summary</td>
      <td>https://en.wikipedia.org/wiki/Trailblazer_(satellite)</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2008-09-20T00:00:00.000Z</td>
      <td>1.221869e+09</td>
      <td>False</td>
      <td>0.0</td>
      <td>5e9d0d95eda69955f709d1eb</td>
      <td>True</td>
      <td>[]</td>
      <td>Ratsat was carried to orbit on the first successful orbital launch of any privately funded and developed, liquid-propelled carrier rocket, the SpaceX Falcon 1</td>
      <td>[]</td>
      <td>[]</td>
      <td>[]</td>
      <td>[5eb0e4b7b6c3bb0006eeb1e5]</td>
      <td>5e9e4502f5090995de566f86</td>
      <td>4</td>
      <td>RatSat</td>
      <td>2008-09-28T23:15:00.000Z</td>
      <td>1222643700</td>
      <td>2008-09-28T11:15:00+12:00</td>
      <td>hour</td>
      <td>False</td>
      <td>[{'core': '5e9e289ef3591855dc3b2626', 'flight': 1, 'gridfins': False, 'legs': False, 'reused': False, 'landing_attempt': False, 'landing_success': None, 'landing_type': None, 'landpad': None}]</td>
      <td>True</td>
      <td>False</td>
      <td>None</td>
      <td>5eb87cdbffd86e000604b32d</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>[]</td>
      <td>https://images2.imgbox.com/95/39/sRqN7rsv_o.png</td>
      <td>https://images2.imgbox.com/a3/99/qswRYzE8_o.png</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>[]</td>
      <td>[]</td>
      <td>None</td>
      <td>https://www.youtube.com/watch?v=dLQ2tZEH6G0</td>
      <td>dLQ2tZEH6G0</td>
      <td>https://en.wikipedia.org/wiki/Ratsat</td>
      <td>https://en.wikipedia.org/wiki/Ratsat</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>None</td>
      <td>NaN</td>
      <td>False</td>
      <td>0.0</td>
      <td>5e9d0d95eda69955f709d1eb</td>
      <td>True</td>
      <td>[]</td>
      <td>None</td>
      <td>[]</td>
      <td>[]</td>
      <td>[]</td>
      <td>[5eb0e4b7b6c3bb0006eeb1e6]</td>
      <td>5e9e4502f5090995de566f86</td>
      <td>5</td>
      <td>RazakSat</td>
      <td>2009-07-13T03:35:00.000Z</td>
      <td>1247456100</td>
      <td>2009-07-13T15:35:00+12:00</td>
      <td>hour</td>
      <td>False</td>
      <td>[{'core': '5e9e289ef359184f103b2627', 'flight': 1, 'gridfins': False, 'legs': False, 'reused': False, 'landing_attempt': False, 'landing_success': None, 'landing_type': None, 'landpad': None}]</td>
      <td>True</td>
      <td>False</td>
      <td>None</td>
      <td>5eb87cdcffd86e000604b32e</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>[]</td>
      <td>https://images2.imgbox.com/ab/5a/Pequxd5d_o.png</td>
      <td>https://images2.imgbox.com/92/e4/7Cf6MLY0_o.png</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>[]</td>
      <td>[]</td>
      <td>http://www.spacex.com/press/2012/12/19/spacexs-falcon-1-successfully-delivers-razaksat-satellite-orbit</td>
      <td>https://www.youtube.com/watch?v=yTaIDooc8Og</td>
      <td>yTaIDooc8Og</td>
      <td>http://www.spacex.com/news/2013/02/12/falcon-1-flight-5</td>
      <td>https://en.wikipedia.org/wiki/RazakSAT</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>



You will notice that a lot of the data are IDs. For example the rocket column has no information about the rocket just an identification number.

We will now use the API again to get information about the launches using the IDs given for each launch. Specifically we will be using columns <code>rocket</code>, <code>payloads</code>, <code>launchpad</code>, and <code>cores</code>.



```python
# Lets take a subset of our dataframe keeping only the features we want and the flight number, and date_utc.
data = data[['rocket', 'payloads', 'launchpad', 'cores', 'flight_number', 'date_utc']]

# We will remove rows with multiple cores because those are falcon rockets with 2 extra rocket boosters and rows that have multiple payloads in a single rocket.
data = data[data['cores'].map(len)==1]
data = data[data['payloads'].map(len)==1]

# Since payloads and cores are lists of size 1 we will also extract the single value in the list and replace the feature.
data['cores'] = data['cores'].map(lambda x : x[0])
data['payloads'] = data['payloads'].map(lambda x : x[0])

# We also want to convert the date_utc to a datetime datatype and then extracting the date leaving the time
data['date'] = pd.to_datetime(data['date_utc']).dt.date

# Using the date we will restrict the dates of the launches
data = data[data['date'] <= datetime.date(2020, 11, 13)]
```

* From the <code>rocket</code> we would like to learn the booster name

* From the <code>payload</code> we would like to learn the mass of the payload and the orbit that it is going to

* From the <code>launchpad</code> we would like to know the name of the launch site being used, the longitude, and the latitude.

* From <code>cores</code> we would like to learn the outcome of the landing, the type of the landing, number of flights with that core, whether gridfins were used, whether the core is reused, whether legs were used, the landing pad used, the block of the core which is a number used to seperate version of cores, the number of times this specific core has been reused, and the serial of the core.

The data from these requests will be stored in lists and will be used to create a new dataframe.



```python
#Global variables 
BoosterVersion = []
PayloadMass = []
Orbit = []
LaunchSite = []
Outcome = []
Flights = []
GridFins = []
Reused = []
Legs = []
LandingPad = []
Block = []
ReusedCount = []
Serial = []
Longitude = []
Latitude = []
```

These functions will apply the outputs globally to the above variables. Let's take a looks at <code>BoosterVersion</code> variable. Before we apply  <code>getBoosterVersion</code> the list is empty:



```python
BoosterVersion
```




    []



Now, let's apply <code> getBoosterVersion</code> function method to get the booster version



```python
# Call getBoosterVersion
getBoosterVersion(data)
```

the list has now been update 



```python
BoosterVersion[0:5]
```




    ['Falcon 1', 'Falcon 1', 'Falcon 1', 'Falcon 1', 'Falcon 9']



we can apply the rest of the  functions here:



```python
# Call getLaunchSite
getLaunchSite(data)
```


```python
# Call getPayloadData
getPayloadData(data)
```


```python
# Call getCoreData
getCoreData(data)
```

Finally lets construct our dataset using the data we have obtained. We we combine the columns into a dictionary.



```python
launch_dict = {'FlightNumber': list(data['flight_number']),
'Date': list(data['date']),
'BoosterVersion':BoosterVersion,
'PayloadMass':PayloadMass,
'Orbit':Orbit,
'LaunchSite':LaunchSite,
'Outcome':Outcome,
'Flights':Flights,
'GridFins':GridFins,
'Reused':Reused,
'Legs':Legs,
'LandingPad':LandingPad,
'Block':Block,
'ReusedCount':ReusedCount,
'Serial':Serial,
'Longitude': Longitude,
'Latitude': Latitude}

```

Then, we need to create a Pandas data frame from the dictionary launch_dict.



```python
# Create a data from launch_dict
data2 = pd.DataFrame(launch_dict)
```

Show the summary of the dataframe



```python
# Show the head of the dataframe
data2.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>FlightNumber</th>
      <th>Date</th>
      <th>BoosterVersion</th>
      <th>PayloadMass</th>
      <th>Orbit</th>
      <th>LaunchSite</th>
      <th>Outcome</th>
      <th>Flights</th>
      <th>GridFins</th>
      <th>Reused</th>
      <th>Legs</th>
      <th>LandingPad</th>
      <th>Block</th>
      <th>ReusedCount</th>
      <th>Serial</th>
      <th>Longitude</th>
      <th>Latitude</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>2006-03-24</td>
      <td>Falcon 1</td>
      <td>20.0</td>
      <td>LEO</td>
      <td>Kwajalein Atoll</td>
      <td>None None</td>
      <td>1</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>None</td>
      <td>NaN</td>
      <td>0</td>
      <td>Merlin1A</td>
      <td>167.743129</td>
      <td>9.047721</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>2007-03-21</td>
      <td>Falcon 1</td>
      <td>NaN</td>
      <td>LEO</td>
      <td>Kwajalein Atoll</td>
      <td>None None</td>
      <td>1</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>None</td>
      <td>NaN</td>
      <td>0</td>
      <td>Merlin2A</td>
      <td>167.743129</td>
      <td>9.047721</td>
    </tr>
    <tr>
      <th>2</th>
      <td>4</td>
      <td>2008-09-28</td>
      <td>Falcon 1</td>
      <td>165.0</td>
      <td>LEO</td>
      <td>Kwajalein Atoll</td>
      <td>None None</td>
      <td>1</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>None</td>
      <td>NaN</td>
      <td>0</td>
      <td>Merlin2C</td>
      <td>167.743129</td>
      <td>9.047721</td>
    </tr>
    <tr>
      <th>3</th>
      <td>5</td>
      <td>2009-07-13</td>
      <td>Falcon 1</td>
      <td>200.0</td>
      <td>LEO</td>
      <td>Kwajalein Atoll</td>
      <td>None None</td>
      <td>1</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>None</td>
      <td>NaN</td>
      <td>0</td>
      <td>Merlin3C</td>
      <td>167.743129</td>
      <td>9.047721</td>
    </tr>
    <tr>
      <th>4</th>
      <td>6</td>
      <td>2010-06-04</td>
      <td>Falcon 9</td>
      <td>NaN</td>
      <td>LEO</td>
      <td>CCSFS SLC 40</td>
      <td>None None</td>
      <td>1</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>None</td>
      <td>1.0</td>
      <td>0</td>
      <td>B0003</td>
      <td>-80.577366</td>
      <td>28.561857</td>
    </tr>
  </tbody>
</table>
</div>



### Task 2: Filter the dataframe to only include `Falcon 9` launches

Finally we will remove the Falcon 1 launches keeping only the Falcon 9 launches. Filter the data dataframe using the <code>BoosterVersion</code> column to only keep the Falcon 9 launches. Save the filtered data to a new dataframe called <code>data_falcon9</code>.



```python
# Hint data['BoosterVersion']!='Falcon 1'
data_falcon9 = data2[data2['BoosterVersion'] == 'Falcon 9']
len(data_falcon9['FlightNumber'])
```




    90



Now that we have removed some values we should reset the FlgihtNumber column



```python
data_falcon9.loc[:,'FlightNumber'] = list(range(1, data_falcon9.shape[0]+1))
data_falcon9
```

    /home/jupyterlab/conda/envs/python/lib/python3.7/site-packages/pandas/core/indexing.py:1773: SettingWithCopyWarning: 
    A value is trying to be set on a copy of a slice from a DataFrame.
    Try using .loc[row_indexer,col_indexer] = value instead
    
    See the caveats in the documentation: https://pandas.pydata.org/pandas-docs/stable/user_guide/indexing.html#returning-a-view-versus-a-copy
      self._setitem_single_column(ilocs[0], value, pi)





<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>FlightNumber</th>
      <th>Date</th>
      <th>BoosterVersion</th>
      <th>PayloadMass</th>
      <th>Orbit</th>
      <th>LaunchSite</th>
      <th>Outcome</th>
      <th>Flights</th>
      <th>GridFins</th>
      <th>Reused</th>
      <th>Legs</th>
      <th>LandingPad</th>
      <th>Block</th>
      <th>ReusedCount</th>
      <th>Serial</th>
      <th>Longitude</th>
      <th>Latitude</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>4</th>
      <td>1</td>
      <td>2010-06-04</td>
      <td>Falcon 9</td>
      <td>NaN</td>
      <td>LEO</td>
      <td>CCSFS SLC 40</td>
      <td>None None</td>
      <td>1</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>None</td>
      <td>1.0</td>
      <td>0</td>
      <td>B0003</td>
      <td>-80.577366</td>
      <td>28.561857</td>
    </tr>
    <tr>
      <th>5</th>
      <td>2</td>
      <td>2012-05-22</td>
      <td>Falcon 9</td>
      <td>525.0</td>
      <td>LEO</td>
      <td>CCSFS SLC 40</td>
      <td>None None</td>
      <td>1</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>None</td>
      <td>1.0</td>
      <td>0</td>
      <td>B0005</td>
      <td>-80.577366</td>
      <td>28.561857</td>
    </tr>
    <tr>
      <th>6</th>
      <td>3</td>
      <td>2013-03-01</td>
      <td>Falcon 9</td>
      <td>677.0</td>
      <td>ISS</td>
      <td>CCSFS SLC 40</td>
      <td>None None</td>
      <td>1</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>None</td>
      <td>1.0</td>
      <td>0</td>
      <td>B0007</td>
      <td>-80.577366</td>
      <td>28.561857</td>
    </tr>
    <tr>
      <th>7</th>
      <td>4</td>
      <td>2013-09-29</td>
      <td>Falcon 9</td>
      <td>500.0</td>
      <td>PO</td>
      <td>VAFB SLC 4E</td>
      <td>False Ocean</td>
      <td>1</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>None</td>
      <td>1.0</td>
      <td>0</td>
      <td>B1003</td>
      <td>-120.610829</td>
      <td>34.632093</td>
    </tr>
    <tr>
      <th>8</th>
      <td>5</td>
      <td>2013-12-03</td>
      <td>Falcon 9</td>
      <td>3170.0</td>
      <td>GTO</td>
      <td>CCSFS SLC 40</td>
      <td>None None</td>
      <td>1</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>None</td>
      <td>1.0</td>
      <td>0</td>
      <td>B1004</td>
      <td>-80.577366</td>
      <td>28.561857</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>89</th>
      <td>86</td>
      <td>2020-09-03</td>
      <td>Falcon 9</td>
      <td>15600.0</td>
      <td>VLEO</td>
      <td>KSC LC 39A</td>
      <td>True ASDS</td>
      <td>2</td>
      <td>True</td>
      <td>True</td>
      <td>True</td>
      <td>5e9e3032383ecb6bb234e7ca</td>
      <td>5.0</td>
      <td>12</td>
      <td>B1060</td>
      <td>-80.603956</td>
      <td>28.608058</td>
    </tr>
    <tr>
      <th>90</th>
      <td>87</td>
      <td>2020-10-06</td>
      <td>Falcon 9</td>
      <td>15600.0</td>
      <td>VLEO</td>
      <td>KSC LC 39A</td>
      <td>True ASDS</td>
      <td>3</td>
      <td>True</td>
      <td>True</td>
      <td>True</td>
      <td>5e9e3032383ecb6bb234e7ca</td>
      <td>5.0</td>
      <td>13</td>
      <td>B1058</td>
      <td>-80.603956</td>
      <td>28.608058</td>
    </tr>
    <tr>
      <th>91</th>
      <td>88</td>
      <td>2020-10-18</td>
      <td>Falcon 9</td>
      <td>15600.0</td>
      <td>VLEO</td>
      <td>KSC LC 39A</td>
      <td>True ASDS</td>
      <td>6</td>
      <td>True</td>
      <td>True</td>
      <td>True</td>
      <td>5e9e3032383ecb6bb234e7ca</td>
      <td>5.0</td>
      <td>12</td>
      <td>B1051</td>
      <td>-80.603956</td>
      <td>28.608058</td>
    </tr>
    <tr>
      <th>92</th>
      <td>89</td>
      <td>2020-10-24</td>
      <td>Falcon 9</td>
      <td>15600.0</td>
      <td>VLEO</td>
      <td>CCSFS SLC 40</td>
      <td>True ASDS</td>
      <td>3</td>
      <td>True</td>
      <td>True</td>
      <td>True</td>
      <td>5e9e3033383ecbb9e534e7cc</td>
      <td>5.0</td>
      <td>12</td>
      <td>B1060</td>
      <td>-80.577366</td>
      <td>28.561857</td>
    </tr>
    <tr>
      <th>93</th>
      <td>90</td>
      <td>2020-11-05</td>
      <td>Falcon 9</td>
      <td>3681.0</td>
      <td>MEO</td>
      <td>CCSFS SLC 40</td>
      <td>True ASDS</td>
      <td>1</td>
      <td>True</td>
      <td>False</td>
      <td>True</td>
      <td>5e9e3032383ecb6bb234e7ca</td>
      <td>5.0</td>
      <td>8</td>
      <td>B1062</td>
      <td>-80.577366</td>
      <td>28.561857</td>
    </tr>
  </tbody>
</table>
<p>90 rows × 17 columns</p>
</div>



## Data Wrangling


We can see below that some of the rows are missing values in our dataset.



```python
data_falcon9.isnull().sum()
```




    FlightNumber       0
    Date               0
    BoosterVersion     0
    PayloadMass        5
    Orbit              0
    LaunchSite         0
    Outcome            0
    Flights            0
    GridFins           0
    Reused             0
    Legs               0
    LandingPad        26
    Block              0
    ReusedCount        0
    Serial             0
    Longitude          0
    Latitude           0
    dtype: int64



Before we can continue we must deal with these missing values. The <code>LandingPad</code> column will retain None values to represent when landing pads were not used.


### Task 3: Dealing with Missing Values


Calculate below the mean for the <code>PayloadMass</code> using the <code>.mean()</code>. Then use the mean and the <code>.replace()</code> function to replace `np.nan` values in the data with the mean you calculated.



```python
# Calculate the mean value of PayloadMass column
m = data_falcon9['PayloadMass'].mean()
# Replace the np.nan values with its mean value
data_falcon9['PayloadMass'].replace(np.nan, m)
data_falcon9.isnull().sum()
```




    FlightNumber       0
    Date               0
    BoosterVersion     0
    PayloadMass        5
    Orbit              0
    LaunchSite         0
    Outcome            0
    Flights            0
    GridFins           0
    Reused             0
    Legs               0
    LandingPad        26
    Block              0
    ReusedCount        0
    Serial             0
    Longitude          0
    Latitude           0
    dtype: int64



You should see the number of missing values of the <code>PayLoadMass</code> change to zero.


Now we should have no missing values in our dataset except for in <code>LandingPad</code>.


We can now export it to a <b>CSV</b> for the next section,but to make the answers consistent, in the next lab we will provide data in a pre-selected date range. 


<code>data_falcon9.to_csv('dataset_part_1.csv', index=False)</code>


## Authors


<a href="https://www.linkedin.com/in/joseph-s-50398b136/?utm_medium=Exinfluencer&utm_source=Exinfluencer&utm_content=000026UJ&utm_term=10006555&utm_id=NA-SkillsNetwork-Channel-SkillsNetworkCoursesIBMDS0321ENSkillsNetwork865-2022-01-01">Joseph Santarcangelo</a> has a PhD in Electrical Engineering, his research focused on using machine learning, signal processing, and computer vision to determine how videos impact human cognition. Joseph has been working for IBM since he completed his PhD. 


## Change Log


|Date (YYYY-MM-DD)|Version|Changed By|Change Description|
|-|-|-|-|
|2020-09-20|1.1|Joseph|get result each time you run|
|2020-09-20|1.1|Azim |Created Part 1 Lab using SpaceX API|
|2020-09-20|1.0|Joseph |Modified Multiple Areas|


Copyright © 2021 IBM Corporation. All rights reserved.



```python

```


```python

```
