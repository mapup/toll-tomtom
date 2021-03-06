# [](https://developer.tomtom.com/)

### Get API key to access TomTom APIs (if you have an API key skip this)
#### Step 1: Login/Singup
* Create an account to access [TomTom Developer Portal](https://developer.tomtom.com/)
* go to signup/login link https://developer.tomtom.com/user/login
* you will need to agree to TomTom's Terms of Service https://developer.tomtom.com/terms-and-conditions

#### Step 2: Getting you Key
* Login to your TomTom Developer Portal
* You can find you key at https://developer.tomtom.com/user/me/apps
* If you want you can create additional keys as per different
  applications
  
#### Step 3: Getting Geocodes from Tomtom
* Use the following function to get geocodes for an address
```python
import json
import requests

#API key for TomTom
token=os.environ.get("Tomtom_API_Key")

def get_geocode_from_tomtom(address):
    url = f"https://api.tomtom.com/search/2/geocode/{address}.JSON?key={token}&limit=1"
    latitude,longitude=requests.get(url).json()['results'][0]['position'].values()
    return (latitude,longitude)
```
With this in place, make a GET request: https://api.tomtom.com/routing/1/calculateRoute/${source.latitude},${source.longitude}:${destination.latitude},${destination.longitude}/json?avoid=unpavedRoads&key=${key}
#### Step 4: Extracting Polyline from TomTom
### Note:
* TomTom accepts source and destination, as `:` seperated `${longitude,latitude}`.
* TomTom doesn't return us route as a polyline, but as a list of `{ latitude, longitude }`, we need to convert this to a polyline
* Use the following function to extract polyline from TomTom
```python
#Importing modules
import json
import requests
import os
import polyline as poly

#API key for TomTom
token=os.environ.get("Tomtom_API_Key")

def get_polyline_from_tomtom(source_latitude,source_longitude,destination_latitude,destination_longitude):
    #Query Tomtom with Key and Source-Destination coordinates
    url='https://api.tomtom.com/routing/1/calculateRoute/{a},{b}:{c},{d}/json?avoid=unpavedRoads&key={e}'.format(a=source_latitude,b=source_longitude,c=destination_latitude,d=destination_longitude,e=token)
    #converting the response to json
    response_from_tomtom=requests.get(url).json()
    #Tomtom oes not provide polylines directly , it rather embeds Latitude and Longitude of each node in the route as dictionary key-value pair inside a list 
    #We thus are extracting the list of such dictionary
    coordinates_dict_list=response_from_tomtom['routes'][0]['legs'][0]['points']
    #We now must convert this list of dictionary to simple tuple or list iterables for "polyline.encode" to work
    coordinates_list=[(i['latitude'],i['longitude']) for i in coordinates_dict_list]
    #generating polyline from list of lat-lon pairs
    polyline_from_tomtom=poly.encode(coordinates_list)
    return(polyline_from_tomtom)
```

Note:

We extracted the polyline for a route from TomTom Maps API

We need to send this route polyline to TollGuru API to receive toll information

## [TollGuru API](https://tollguru.com/developers/docs/)

### Get key to access TollGuru polyline API

#### Step 5: Calling TollGuru API to get rates
* Create a dev account to receive a free key from TollGuru https://tollguru.com/developers/get-api-key
* Suggest adding `vehicleType` parameter. Tolls for cars are different than trucks and therefore if `vehicleType` is not specified, may not receive accurate tolls. For example, tolls are generally higher for trucks than cars. If `vehicleType` is not specified, by default tolls are returned for 2-axle cars. 
* Similarly, `departure_time` is important for locations where tolls change based on time-of-the-day.
* Use the following function to get rates

```python
#Importing modules
import json
import requests
import os
import polyline as poly

#API key for Tollguru
Tolls_Key = os.environ.get("TollGuru_API_Key")

def get_rates_from_tollguru(polyline):
    #Tollguru querry url
    Tolls_URL = 'https://dev.tollguru.com/v1/calc/route'
    #Tollguru resquest parameters
    headers = {
                'Content-type': 'application/json',
                'x-api-key': Tolls_Key
                }
    params = {
                #Explore https://tollguru.com/developers/docs/ to get best of all the parameter that tollguru has to offer 
                'source': "tomtom",
                'polyline': polyline ,                      # this is the encoded polyline that we made     
                'vehicleType': '2AxlesAuto',                #'''Visit https://tollguru.com/developers/docs/#vehicle-types to know more options'''
                'departure_time' : "2021-01-05T09:46:08Z"   #'''Visit https://en.wikipedia.org/wiki/Unix_time to know the time format'''
                }
    #Requesting Tollguru with parameters
    response_tollguru= requests.post(Tolls_URL, json=params, headers=headers).json()
    #checking for errors or printing rates
    if str(response_tollguru).find('message')==-1:
        return(response_tollguru['route']['costs'])
    else:
        raise Exception(response_tollguru['message'])
```

The working code can be found in TomTom.py file.

## License
ISC License (ISC). Copyright 2020 &copy;TollGuru. https://tollguru.com/

Permission to use, copy, modify, and/or distribute this software for any purpose with or without fee is hereby granted, provided that the above copyright notice and this permission notice appear in all copies.

THE SOFTWARE IS PROVIDED "AS IS" AND THE AUTHOR DISCLAIMS ALL WARRANTIES WITH REGARD TO THIS SOFTWARE INCLUDING ALL IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS. IN NO EVENT SHALL THE AUTHOR BE LIABLE FOR ANY SPECIAL, DIRECT, INDIRECT, OR CONSEQUENTIAL DAMAGES OR ANY DAMAGES WHATSOEVER RESULTING FROM LOSS OF USE, DATA OR PROFITS, WHETHER IN AN ACTION OF CONTRACT, NEGLIGENCE OR OTHER TORTIOUS ACTION, ARISING OUT OF OR IN CONNECTION WITH THE USE OR PERFORMANCE OF THIS SOFTWARE.
