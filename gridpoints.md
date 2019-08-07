## Gridpoint Frequently Asked Questions

### What's a gridpoint?

Each National Weather Service forecast office issues numerical forecasts on a 2.5-kilometer grid across their entire
forecast area. Each gridpoint is one of these 2.5km squares.

The `/gridpoints` endpoint in the API gives you access to this raw numerical data.

---

### How do I determine the gridpoint for my location?

You can retrieve the metadata for a given latitude/longitude coordinate with the `/points` endpoint
(`https://api.weather.gov/points/{lat},{lon}`).

The `forecastGridData` property will provide a link to the correct gridpoint for that location.

---

### Can I search by city or airport instead?

Please see [our FAQ about geocoding](general-faqs#geocoding).

---

### How is the gridpoint data formatted?

* The data is presented as a JSON document.
* The document will contain metadata about the forecast grid, along with multiple layers of data (such as temperature).
* The layer data is presented as a time series.
  * Much of the data is calculated for each hour of the forecast period, but to conserve bandwidth the API will merge
    consecutive values that are equal.
  * Each data point will have the following properties:
    * validTime: This is the time interval that the value applies to. The interval is represented in
      [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) format.
      * For example, `2019-07-04T18:00:00+00:00/PT3H` represents an interval starting on July 4, 2019 at 1800 UTC and
        continuing for 3 hours (from 1800 to 2100).
    * value: This is the data that applies to that validTime interval.
      * For values that have a unit of measure, such as temperature, the layer will contain a `uom` property to
        indicate the unit of all the values in that time series.
      * The value may also be `null` to indicate there is no applicable data for that interval.

---

### What layer data is available?

Below is a list of layers that the API exposes.

##### Commonly used layers

* temperature: Air temperature
* dewpoint: Dewpoint temperature
* maxTemperature: High temperature
* minTemperature: Low temperature
* relativeHumidity: Relative humidity
* apparentTemperature: Apparent temperature (heat index or wind chill)
* heatIndex: Heat index. This is the apparentTemperature data with values filtered out that are below the threshold of
    the heat index calculation.
* windChill: Wind chill. This is the apparentTemperature data with values filtered out that are above the threshold of
    the wind chill calculation.
* skyCover: Percentage of sky with cloud cover
* windDirection: Direction of prevailing wind
* windSpeed: Maximum prevailing wind speed
* windGust: Maximum wind gust
* weather: Weather conditions
  * This data is not numeric, but an array.
  * Each array element is a JSON object representing a decoded
    [NDFD GRIB weather string](https://www.weather.gov/mdl/degrib_ndfdwx).
* hazards: Long-duration area watches, warnings, and advisories in effect.
  * This data is not numeric, but an array.
  * Each array element is a JSON object with `phenomenon` and `significance` properties. These values correspond to
    [P-VTEC phenomenon and significance codes](https://www.weather.gov/media/vtec/VTEC_explanation4-18.pdf). For
    example, `FF` and `A` would indicate a Flash Flood Watch.
  * Optionally, the object may include an `event_number` property. This number will correspond to an issued watch,
    warning, or advisory product. For example, `TO`, `A`, and `267` would indicate Tornado Watch #267.
  * Note that not all forecast offices utilize this layer for all watch/warning/advisory products. It is most typically
    used for tropical storm and hurricane watches and warnings.
* probabilityOfPrecipitation: Chance of precipitation
* quantitativePrecipitation: Amount of liquid precipitation

##### Other layers

* iceAccumulation:
* snowfallAmount:
* snowLevel:
* ceilingHeight:
* visibility:
* transportWindSpeed:
* transportWindDirection:
* mixingHeight:
* hainesIndex:
* lightningActivityLevel:
* twentyFootWindSpeed:
* twentyFootWindDirection:
* waveHeight:
* wavePeriod:
* waveDirection:
* primarySwellHeight:
* primarySwellDirection:
* secondarySwellHeight:
* secondarySwellDirection:
* wavePeriod2:
* windWaveHeight:
* dispersionIndex:
* pressure:
* probabilityOfTropicalStormWinds:
* probabilityOfHurricaneWinds:
* potentialOf15mphWinds:
* potentialOf25mphWinds:
* potentialOf35mphWinds:
* potentialOf45mphWinds:
* potentialOf20mphWindGusts:
* potentialOf30mphWindGusts:
* potentialOf40mphWindGusts:
* potentialOf50mphWindGusts:
* potentialOf60mphWindGusts:
* grasslandFireDangerIndex:
* probabilityOfThunder:
* davisStabilityIndex:
* atmosphericDispersionIndex:
* lowVisibilityOccurrenceRiskIndex:
* stability:
* redFlagThreatIndex:

---

### How can I tell if the grid data has been updated?

The API supports [HTTP conditional requests](https://developer.mozilla.org/en-US/docs/Web/HTTP/Conditional_requests).
All gridpoint endpoints will include a `Last-Modified` header showing the creation time of the grid data we received
from the forecast office. You may include this time in the `If-Modified-Since` header on subsequent requests to
determine if the data has been updated. If it has not, you will receive a 304 (Not Modified) response.

If your HTTP client cannot support conditional requests for some reason, this information is also in the `updateTime`
property of the gridpoint data.

---

### I see some really strange values (like temperatures in the thousands).

[Please report this as an operational issue.](reporting-issues)
