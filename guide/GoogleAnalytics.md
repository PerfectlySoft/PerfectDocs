# Google Analytics Measurement Protocol

The Google Analytics Measurement Protocol is the server-side equivalent of embedding Google Analytics into a web page.

It means that you can log any sort of activity - Raw TCP or UDP events, or specific interactions that are triggered by events like AJAX.

## API Documentation

For full API documentation, visit [https://www.perfect.org/docs/api-Perfect-GoogleAnalytics-MeasurementProtocol.html](https://www.perfect.org/docs/api-Perfect-GoogleAnalytics-MeasurementProtocol.html).

The API documentation explains every property that can be set within the system.

## Configuration

The `PerfectGAMeasurementProtocol` struct enables the setting of application-wide defaults for Property ID and Hit Type.

``` swift
PerfectGAMeasurementProtocol.propertyid = "UA-XXXXXXXX-X"
PerfectGAMeasurementProtocol.hitType = "pageview"
```

## Building

Add this project as a dependency in your Package.swift file.

``` swift
.Package(url: "https://github.com/PerfectlySoft/Perfect-GoogleAnalytics-MeasurementProtocol.git", majorVersion: 3)
```

## Example Usage

To set up and execute the logging of an event:

```swift
PerfectGAMeasurementProtocol.propertyid = "UA-XXXXXXXX-X"
let gaex = PerfectGAEvent()
gaex.user.uid = "donkey"
gaex.user.cid = "kong"
gaex.session.ua = "aua"
gaex.traffic.ci = "ci"
gaex.system.fl = "x"
gaex.hit.ni = 2


do {
	let str = try gaex.generate()
	print(str)
	let resp = gaex.makeRequest(useragent: "TestingAPI1.0", body: str)
	print(resp)
} catch {
	print("\(error)")
}

```


A series of common hit types and configurations can be found in Google's documentation, [https://developers.google.com/analytics/devguides/collection/protocol/v1/devguide#commonhits](https://developers.google.com/analytics/devguides/collection/protocol/v1/devguide#commonhits) 

