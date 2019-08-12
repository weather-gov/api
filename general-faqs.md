## General FAQs

### What's the API?
api.weather.gov represents the public face of the next generation of data services from the
[National Weather Service](https://weather.gov). It offers public access to a wide range of essential weather data, in
a way that modern web developers expect: a REST-style, JSON-based web service.

The API is also being built to provide a powerful and modern platform for data dissemination for both internal and
external customers. We're doing this to help eliminate some internal redundancies, giving you and our own forecasters
and developers a one-stop shop for vital data.

---

### I'm new to this. How do I use the API?
Our current documentation is located at <https://www.weather.gov/documentation/services-web-api>. Over time we will
also be offering this information here as well.

We also provide a service definition in [OpenAPI v3.0](https://swagger.io/specification/) (previously known as Swagger)
format. You can access this file at <https://api.weather.gov/openapi.json> (in JSON format)
or <https://api.weather.gov/openapi.yaml> (in YAML format).

Much of the data returned from the API is in [GeoJSON](http://geojson.org/) (RFC 7946) format. Certain endpoints also
offer useful alternative formats, such as
[OASIS Common Alerting Protocol (CAP) XML](http://docs.oasis-open.org/emergency/cap/v1.2/CAP-v1.2-os.html) for alerts
data.

---

<a name="how-to-get-forecast"></a>
### How do I get a forecast for a location from the API?
You'll need to know the latitude and longitude of the location in decimal degrees. (If you want to get really
geospatially technical, your location should be a WGS 84 or EPSG 4326 coordinate.)

* For our example here, we'll use the Washington Monument in Washington, D.C. It's located at 38.8894 latitude,
  -77.0352 longitude.
* Please note that, for efficiency purposes, the API doesn't support more than four decimal places of precision in
  coordinates. If you send a more precise coordinate, you'll receive an error giving you the closest proper coordinate.
  Four decimal places is about 30 feet (10 meters) over most of the United States, so that should still be close
  enough!

Once you know the latitude and longitude, it's an easy three-step process from there. You can follow along in your
browser with the links below:

1. Retrieve the metadata for your location from `https://api.weather.gov/points/{lat},{lon}`.
   * For our example the URL will be <https://api.weather.gov/points/38.8894,-77.0352>
2. You'll get back a JSON document. Inside the document, find the `properties` object, and inside that, find the
   `forecast` property. You'll find another URL there.
   * For our example this gives us <https://api.weather.gov/gridpoints/LWX/96,70/forecast>
   * You can also get the hour-by-hour forecast from the `forecastHourly` property. For our example it's
     <https://api.weather.gov/gridpoints/LWX/96,70/forecast/hourly>
3. Retrieve that URL. You'll get a JSON document containing the forecast for that location. There you go!

We're still working on documentation for the JSON that API returns, but we think it's pretty easy to understand if you
just look at it. If you need more reference, the forecast JSON very closely aligns with the information you'd see on a
web page on forecast.weather.gov. If you still have questions, please [see our Reporting Issues page](reporting-issues)
for how to ask.

---

<a name="geocoding"></a>
### I have an address, city name, or zip code location. Can I request data for this location via the API?
Not directly. You'll need to turn that location into a latitude/longitude pair as described earlier. This is called
_geocoding_.

Our API does not offer a geocoding service. There are many free and paid API services available for this. Here are a
few:

* [Bing Maps Locations](https://docs.microsoft.com/en-us/bingmaps/rest-services/locations/)
* [Esri ArcGIS World Geocoding Service](https://developers.arcgis.com/rest/geocode/api-reference/overview-world-geocoding-service.htm)
* [Geocode.xyz](https://geocode.xyz/api)
* [Google Maps Geocoding](https://developers.google.com/maps/documentation/geocoding/start)
* [MapBox Geocoding](https://docs.mapbox.com/api/search/#geocoding)
* [MapQuest Geocoding](https://developer.mapquest.com/documentation/geocoding-api/)
* [US Census Bureau Geocoding Service](https://geocoding.geo.census.gov/geocoder/Geocoding_Services_API.html)

---

### Do I really have to do two requests?
Yes. Part of the API's design is to improve efficiency and reduce data redundancy. National Weather Service forecasts
are issued on a 2.5km grid. The /points request is to translate your position to the grid square that it's in. A lot of
different points will resolve to the same grid, and we can share that same data with many different users in the same
area.

The point mappings don't change very often, so you can cache the result of the /points request to avoid doing it
repeatedly. (Also see the next question.)

---

### How do I know I'm getting the latest data? Do I need to use "cache busting" methods?
The API is designed from the ground up to always provide current data as well as properly support HTTP caching. We send
back Cache-Control headers to advise how long to hold on to a response, and Last-Modified headers so you can validate
if the data has changed later. Many clients will automatically make use of these headers with a bit of configuration.

Please don't use cache busting techniques like random numbers in the query string. Requests like this with query
parameters that aren't recognized by the API will return a 400 (Bad Request) response.

If you do run into a case where you believe the API is not giving you the latest data, or advising you to cache it
longer than it should, please [let us know](reporting-issues).

---

### I'm getting a 403 (Forbidden/Access Denied) error from the API.
Make sure your program is including a `User-Agent` header in your request. We recommend setting the value to something
that identifies your application and includes a contact email. This helps us contact you if we notice unusual behavior,
such as your program consuming a high amount of resources.

In the future we will replace the User-Agent requirement with a more typical API key system.

If you're including the User-Agent header and are still having problems, please
[let us know](reporting-issues).

---

### I already use some of the NWS web services on forecast.weather.gov. How can I switch my application to use the API?
Depending on what service you're using, the data may already be available via the API.

##### I use the Digital Weather Markup Language service on forecast.weather.gov.
You can follow [the directions from earlier](#how-to-get-forecast) on how to get a forecast. On the third
step, you can request the DWML instead of the JSON forecast by adding an `Accept` header with the value
`application/vnd.noaa.dwml+xml` to your request. (You'll need to consult the documentation for your program's HTTP
library on how to add a request header.)

We would encourage you to change your application to use the new JSON forecast format, as this will be our main focus
for support going forward.

##### I use the JSON service on forecast.weather.gov.
The API won't support the existing JSON format, but take a look at the new JSON format
[as described earlier](#how-to-get-forecast). We think you'll agree that it's much simpler to use in your
application.

##### I'm not able to switch to using the API just yet.
You can continue to use the existing legacy services on `forecast.weather.gov`, but we encourage you to move to the API
when you can. We have more resources dedicated to supporting the API and much of our future development will be focused
here.

##### I'm using a service that isn't described here.
Please [give us some feedback](reporting-issues) on what you're using, so we can evaluate it for future
inclusion into the API, or point you to the best place to get the information.
