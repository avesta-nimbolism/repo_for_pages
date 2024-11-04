<details>
<summary>Documentations for Neshan API subgroup</summary>

## Table of Contents
1. [Directions API](#1-directions-api)
2. [Address API](#2-address-api)
3. [Point API](#3-point-api)
4. [Permissions and Authentication](#4-permissions-and-authentication)
5. [Response Structure](#5-response-structure)

## 1. Directions API
### Get directions with accords to traffic.[^11] (POST Method)
### Request sample for *Neshan Direction API*  
```json
{
    "origin":{
        "lat":35.71526,
        "long":51.33544
    },
    "destination":{
        "lat":35.74564,
        "long":51.49611
    },
    "transportType":"car",
    "bearing":0
}
```


### Structure of *Neshan Direction API* request should follow these guidelines:
```go
type DirectionRequest struct {
	Origin        NeshanLocation `json:"origin" validate:"required"`
	Destination   NeshanLocation `json:"destination" validate:"required"`
	TransportType string         `json:"transportType" validate:"required"`
	Bearing       int            `json:"bearing"`
}

type NeshanLocation struct {
	Lat  float64 `json:"lat" validate:"required"`
	Long float64 `json:"long" validate:"required"`
}
```
for more info visit this *Neshan Documentations*.[^12]

## 2. Address API
### Get the location of an address.[^13] (POST Method)
### Request sample for Neshan Address API
```json
{
    "point":{
        "lat":35.71526,
        "long":51.33544
    }
}
```

### Structure of Neshan Address API request should follow these guidelines:
```go
type GetAddressRequest struct {
	Point NeshanLocation `json:"point" validate:"required"`
}
```
for more info visit this *Neshan Documentations*.[^14]

## 3. Point API
### Get address of a location.[^15] (POST Method)
### Request sample for Neshan Point API
```json
{
   "address":"آذربایجان شرقی تبریز خیابان ارتش شمالی کوچه بازارچه رنگی"
}
```

### Structure of Neshan Point API request should follow these guidelines:
```go
type GetPointRequest struct {
	Address string `json:"address" validate:"required"`
}
```  
for more info visit this *Neshan Documentations*.[^16]

## 4. Permissions and Authentication
In the subgroup of every Neshan API on the server, there is a middleware which checks for JWT tokens in the cookies. This implies that every API going through /v1/neshan will have to include a token.  
This authentication happens on another server so there is not a whole lot of info shown to the user.  
All you got to remember is to include this header in every request to /v1/neshan APIs!  
You can get this Token via the .NET server!  
```txt
Key: Cookie
Value: token=<YOUR_BARKOOSH_TOKEN>
```

## 5. Response Structure
The 'data' field in this structure, represents the data that neshan api sends.  
```go
type Response struct {
	IsSuccess bool        `json:"isSuccess"`
	Error     string      `json:"error,omitempty"`
	Message   string      `json:"message,omitempty"`
	Data      interface{} `json:"data,omitempty"`
}
```

[^11]: API endpoint for this request: https://192.168.1.34:3030/v1/neshan/directions (POST Method)
[^12]: Documentations for this request: [direction docs](https://platform.neshan.org/api/direction/)
[^13]: API endpoint for this request: https://192.168.1.34:3030/v1/neshan/address (POST Method)
[^14]: Documentations for this request: [address docs](https://platform.neshan.org/api/geocoding/)
[^15]: API endpoint for this request: https://192.168.1.34:3030/v1/neshan/point (POST Method)
[^16]: Documentations for this request: [point docs](https://platform.neshan.org/api/reverse-geocoding/)
</details>  
<details>
<summary>Documentations for Location API subgroup</summary>

## Table of Contents
1. [Location API](#1-location-api)
2. [Location WebSocket](#2-location-websocket)

## 1. Location API
### Sending truck location via API Endpoint.[^21] (POST Method)
### Request sample for *Location API*
```json
{
    "truckServiceId": "TRUCKSERVICE67890",
    "name": "test",
    "mobile": "09432346642",
    "plate": "12ع12312",
    "userId":"USERID16513",
    "truckId":"TRUCKID63686",
    "type":"", // is it pwa or a native app
    "userAgent":"", // the client phone/browser
    "satteliteCount":"", // number of sattelites at the time of this recorded location
    "signal":"", // how many bars does the signal have
    "location": {
        "xLocation": 35.71526,
        "yLocation": 51.33544,
        "date": "1730291966919", // UNIX Timestamp in ms
        "bearing": 0, // degrees from north (clockwise)
        "speed": 10, // meter per second (m/s)
    }
}
```


### Structure of *Location API* request should follow these guidelines:
```go
type SendLocationRequest struct {
	TruckServiceID string   `json:"truckServiceId" validate:"required"`
	Name           string   `json:"name" validate:"required"`
	Mobile         string   `json:"mobile" validate:"required"`
	Plate          string   `json:"plate" validate:"required"`
	Location       Location `json:"location" validate:"required"`
	UserId         string   `json:"userId" validate:"required"`
	TruckId        string   `json:"truckId" validate:"required"`
	Type           string   `json:"type"`
	UserAgent      string   `json:"userAgent"`
	SatteliteCount string   `json:"satteliteCount"`
	Signal         string   `json:"signal"`
}

type Location struct {
	XLocation float64 `json:"xLocation" validate:"required"`
	YLocation float64 `json:"yLocation" validate:"required"`
	Bearing   int     `json:"bearing"`
	Speed     int     `json:"speed" bson:"speed"`
	Date      string  `bson:"date" json:"date" validate:"required"`
}
```

## 1-1. Helper API
### Get more info about this trip.[^22] (POST Method)
You can use another API on .NET server to get these data needed for *Location API*.  
This API will give you a lot of information about the truck and its destinations. (such as locations of transfer company, destination store, source store, truck owner name, truck owner mobile number, turck ID, plate (in 4 parts), and user ID (via token))  
You should have a valid JWT Token to call this API.  
### Sample request for this endpoint:
```javascript
const request = {
	truckServiceId: <YOUR_TRUCK_SERVICE_ID>,
};
```
### The response from this API follow the same guidelines and therefore return an object in 'data' field:
```javascript
{
  "data": {},
  "isSuccess": true,
  "message": "string",
  "errors": [
    "string"
  ]
}
```

## 2. Location WebSocket
### Send truck location on any event using this API.[^23] (GET Method)
### Connection guide written in javascript for *Location WebSocket*
```javascript
// connecting to https websocket -> wss
let socket = new WebSocket("wss://192.168.1.34:3030/v1/location/ws");
// you could log messages comming from server like this:
socket.onmessage = (event) => {console.log("from server: ", event.data)}
// messages recieved from server are the same as messages recieved in 'message' field in *Location API*
// this is a sample request for websocket:
const locationUpdate = {
	truckServiceId: "TRUCKSERVICE67890",
        userId: "12112312",
        truckId: "12112312",
        name: "test",
        mobile: "09027009737",
        plate: "12ع12312",
        location: {
            xLocation: 35.748642,
            yLocation: 51.301685,
	    bearing: 32,
	    speed: 20,
            date: "1730291966919" // UNIX
	}
};
// but before sending it, keep in mind it should be converted to string!
const message = JSON.stringify(locationUpdate);
// now you can send this message through the websocket channel:
socket.send(message);
```

[^21]: API endpoint for this request: https://192.168.1.34:3030/v1/location/location (POST Method)
[^22]: API endpoint for this request: https://192.168.1.34:3002/api/Truck/GetTruckUserInfoByTruckService (POST Method)
[^23]: API endpoint for this request: https://192.168.1.34:3030/v1/location/ws (GET Method)
</details>  
