# Oracle APEX Live Location Tracking

This project demonstrates how to create a **Live Location Tracking** application in **Oracle APEX** using **HTML5 Geolocation API** to capture the user's real-time latitude and longitude and display it on an interactive map or Address.

## Overview

The application uses:
1. **HTML5 Geolocation API** to get the user's current geographic location (latitude and longitude).
2. **Oracle APEX** to store the captured location data (latitude and longitude) in the database.
3. **Interactive Map** (using Google Maps or Leaflet) to visualize the live location on the frontend.

## Features

- Capture the user's real-time **latitude** and **longitude**.
- Store the captured location in the Oracle APEX database.
- Display the live location on an interactive Address / map.
- Update location in real-time (optional).
  
## Prerequisites

1. **Oracle APEX** environment set up and running.
2. Access to a **Google Maps API key** (if using Google Maps for map display).
3. A modern web browser that supports the **HTML5 Geolocation API**.

- Here i User Free API from https://geocode.maps.co/

## Create Table Schema

Create a table in your Oracle database to store the captured location data.

```sql
CREATE TABLE user_location_data (
    id NUMBER PRIMARY KEY,
    user_id NUMBER,
    latitude NUMBER(9, 6),
    longitude NUMBER(9, 6),
    login_address varchar(1000),
    capture_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

```

## APEX Page Setup
1. Create APEX Items for Latitude,  Longitude and Address

Create two hidden and One Textarea  items in your APEX page to store the latitude (P6_LATITUDE), longitude (P6_LONGITUDE) and Textarea (P6_ADDRESS)

Optionally, create a Text Area or Display Only Items to show the Address againest captured latitude and longitude to the user.


## Add below javascript code to page Function and Global Variable Declaration

```JavaScript code

if (navigator.geolocation) {
    navigator.geolocation.getCurrentPosition(function(position) {
        // Get Latitude and Longitude from the position object
        var latitude = position.coords.latitude;
        var longitude = position.coords.longitude;

        // Set the values to hidden items in APEX
        $s("P6_LATITUDE", latitude);
        $s("P6_LONGITUDE", longitude);
    }, function(error) {
        alert("Error occurred. Error code: " + error.code);
    });
} else {
    alert("Geolocation is not supported by this browser.");
}


```
## Create a dynamic action to call the REST API, which collects data from the Google Maps API (if using Google Maps), triggered by a change event on the fields P6_LATITUDE and P6_LONGITUDE.

- Here i User Free API from https://geocode.maps.co/

1. Below is the PL/SQL Code:

2. Sends a GET request to the REST API.

3. Parses the JSON response using JSON_TABLE.

Inserts the parsed data into the api_data table.

## PL/SQL Code to show Login User Address and Insert into table

```pl/sql code to consume rest api

DECLARE
  l_response_clob  CLOB;
  l_request_body   CLOB;
  l_access_token VARCHAR2(10000);
  l_json_values    apex_json.t_values;
  BEGIN
l_response_clob := apex_web_service.make_rest_request(
  p_url         => 'https://geocode.maps.co/reverse?lat='||:P6_LATITUDE||'&lon='||:P6_LONGITUDE||'&api_key=66209bf0268a5150577189pqr43d58e',
  p_http_method => 'GET',
  p_wallet_path => 'file:/home/oracle/wallet',
  p_wallet_pwd  => ''
  );
  -- Check HTTP status code 
    --APEX_JSON.PARSE(l_response_clob);
     apex_json.parse(
    p_values => l_json_values,
    p_source => l_response_clob
  );

  
  apex_json.parse (l_response_clob);
  
  --dbms_output.put_line('Response2:'||apex_json.get_varchar2 ('display_name'));

  --dbms_output.put_line('Response3:'||apex_json.get_varchar2 ('address.country'));

:P6_ADDRESS := apex_json.get_varchar2 ('display_name');
  

  end;
```


## Insert User Location Data into Oracle Table

```sql insert statement
BEGIN
    INSERT INTO user_location_data (user_id, latitude, longitude, login_address)
    VALUES (:APP_USER, :P1_LATITUDE, :P1_LONGITUDE, :P6_ADDRESS);
    COMMIT;
END;

```

## Steps to Run the Application

1. Set up the Database:

- Create the location_data table in your Oracle database.

- Set up the necessary APEX items to capture and store the latitude and longitude.
  
1. Create the APEX Page:

- Add two hidden items for latitude and longitude.

- Add JavaScript to capture location using the HTML5 Geolocation API.

- Add a button to submit the location to the database.
3. Display the Location on a Map:

- Use Google Maps or Leaflet to visualize the captured location on the map.

- Make sure to update the map in real-time if necessary.

4. Deploy:

- Test the live location tracking on different devices (desktop, mobile) to ensure that the Geolocation API works correctly.

- Deploy the APEX page to production.



 # Thank you
 ## Sanjay Sikder

