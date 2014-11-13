---
author:
- 'EhrScape <info@ehrscape.com>'
date: April 2014
title: REST API Examples
...

Getting Started
===============

This guide is designed to help you explore and learn about the API.

Audience
--------

The easiest way to start learning about the API is to study sample code.
This tutorial is designed for people familiar with the Javascript
programming concepts. There are already many good [Javascript
tutorials](http://www.google.com/search?q=javascript+tutorials)
available on the Web. The REST API is designed to be easily accessible
from any programming language – once you understand the basic concepts
of the API, you will be able to apply them in most programming
languages.

Pre-requisites
--------------

### Supporting libraries

To interact with the REST API, a client application needs to construct
HTTP requests. For making JSON based API calls and deal with the
returned data we are using jQuery javascript library. For more info see
[jQuery home page](http://jquery.com).

### Context variables

Throughout this guide we are using the following predefined variables
and functions to improve the readability of the code:

-   variable `baseUrl`: holds the base url of the REST service. To get
    to a specific REST service we’ll append it to the base url, i.e.:

<!-- -->

    var baseUrl = 'https://rest.ehrscape.com/rest/v1';
    var queryUrl = baseUrl + '/query';

-   variable `ehrId`: holds our sample patient’s ehrId.

-   function `getSessionId()`: is used to obtain a session (ehr session)
    used for authentication. In most applications this session is
    injected as part of the application context and would not be
    directly handled by javascript.

### Base HTML page

All examples bellow are displaying the data retrieved from the server in
a very simple HTML page. Here is the source of a skeleton page:

``` {.html}
<!DOCTYPE html>
<html>
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8"/>
    <title>Think!EHR Rest Example</title>
    <script src="//ajax.googleapis.com/ajax/libs/jquery/2.1.0/jquery.min.js"></script>
    <script src="js/example.js"></script>
</head>
<body>

<h3 id="header">
    <!-- header will be rendered here -->
</h3>
<div id="result">
    <!-- result will be rendered here -->
</div>
</body>
</html>
```

### Function `getSessionId()`

For clarity here’s a basic example of the function:

``` {.js}
function getSessionId() {
    var response = $.ajax({
        type: "POST",
        url: baseUrl + "/session?username=" + encodeURIComponent(username) +
                "&password=" + encodeURIComponent(password),
        async: false
    });
    return response.responseJSON.sessionId;
}
```

Note: Refer to Recommendations and Best Practices in Managing HTTP-Based
Client Sessions. There are better implementations of session handling
function, but this is not the focus of this guide.

The "Hello World" example
-------------------------

Let’s use REST API to fetch some data! The following example shows how
to get information about body temperatures measured for a sample patient
with a predefined `ehrId`.

In this example, the following API call is used:

-   `GET /view/{ehrId}/body_temperature`: returns body temperatures
    measured for the specified EHR ([see
    details](https://dev.ehrscape.com/api-explorer.html?api=thinkehr&service=/view&operation=/view/%7BehrId%7D/body_temperature&method=GET&inline=false)).

``` {.js}
$.ajax({
    url: baseUrl + "/view/" + ehrId + "/body_temperature",
    type: 'GET',
    headers: {
        "Ehr-Session": sessionId
    },
    success: function (res) {
        $("#header").append("Body Temperatures");
        for (var i in res) {
            $("#result").append(res[i].time + ': ' + res[i].temperature  + res[i].unit + "<br>");
        }
    }
});
```

Which returns the following result:

    Body Temperatures

    2014-03-05T04:01:37.000Z: 37°C
    2014-02-14T00:57:00.000Z: 37.4°C
    2014-02-07T20:44:28.000Z: 37.1°C
    2014-01-23T01:26:05.000Z: 39.7°C
    2014-01-11T16:48:31.000Z: 37°C

In addition to retrieving the last set of measured temperatures, it’s
also possible to filter results by date. The API description for the
body\_temperature view lists a parameter `from` which requires an ISO
formatted date (and possibly time) as input. For ISO date and time
formats see [ISO8601](http://en.wikipedia.org/wiki/ISO_8601).

Here’s an example with filtering:

``` {.js}
$.ajax({
    url: baseUrl + "/view/" + ehrId + "/body_temperature",
    type: 'GET',
    data: {
        from: '2014-3-1'
    },
    headers: {
        "Ehr-Session": sessionId
    },
    success: function (res) {
        $("#header").append("Body Temperatures");
        for (var i in res) {
            $("#result").append(res[i].time + ': ' + res[i].temperature + res[i].unit + "<br>");
        }
    }
});
```

which, as expected, returns the following result:

    Body Temperatures

    2014-03-05T04:01:37.000Z: 37°C

Display a patient’s name along with measurement data
----------------------------------------------------

The purpose of this example is to introduce the usage of a demographic
API to also print patient name along with the result of weight
measurements.

The following API calls are used:

-   `GET /demographics/ehr/{ehrId}/party`: retrieves patient’s
    demographic data ([see
    details](https://dev.ehrscape.com/api-explorer.html?api=thinkehr&service=/demographics&operation=/demographics/ehr/%7BehrId%7D/party&method=GET&inline=false)).

-   `GET /view/{ehrId}/weight`: returns weight measurements for the
    specified EHR ([see
    details](https://dev.ehrscape.com/api-explorer.html?api=thinkehr&service=/view&operation=/view/%7BehrId%7D/weight&method=GET&inline=false)).

``` {.js}
$.ajax({
    url: baseUrl + "/demographics/ehr/" + ehrId + "/party",
    type: 'GET',
    headers: {
        "Ehr-Session": sessionId
    },
    success: function (data) {
        var party = data.party;
        $("#header").append("Weight measurements for " + party.firstNames + ' ' +
                                                         party.lastNames);
    }
});
$.ajax({
    url: baseUrl + "/view/" + ehrId + "/weight",
    type: 'GET',
    headers: {
        "Ehr-Session": sessionId
    },
    success: function (res) {
        for (var i in res) {
            $("#result").append(res[i].time + ': ' + res[i].weight + res[i].unit + "<br>");
        }
    }
});
```

which returns the following result:

    Weight measurements for Desiree Hanson

    2014-03-09T19:25:20.000Z: 66.7kg
    2014-02-27T00:49:07.000Z: 66.7kg
    2014-02-07T00:29:51.000Z: 65.3kg
    2014-02-03T14:03:21.000Z: 66.7kg
    2014-02-01T15:00:40.000Z: 68kg
    2014-01-02T17:48:55.000Z: 66.8kg

Create a new patient
--------------------

The purpose of this example is to show how to create a new patient via
demographics API and create his or her EHR.

The following API calls are used:

-   `POST /ehr`: creates a new EHR ([see
    details](https://dev.ehrscape.com/api-explorer.html?api=thinkehr&service=/ehr&operation=/ehr&method=POST&inline=false)).

-   `POST /demographics/party`: creates a new party in the demographics
    server and stores an ehrId ([see
    details](https://dev.ehrscape.com/api-explorer.html?api=thinkehr&service=/demographics&operation=/demographics/party&method=POST&inline=false)).

``` {.js}
$.ajaxSetup({
    headers: {
        "Ehr-Session": sessionId
    }
});
$.ajax({
    url: baseUrl + "/ehr",
    type: 'POST',
    success: function (data) {
        var ehrId = data.ehrId;
        $("#header").html("EHR: " + ehrId);

        // build party data
        var partyData = {
            firstNames: "Mary",
            lastNames: "Wilkinson",
            dateOfBirth: "1982-7-18T19:30",
            partyAdditionalInfo: [
                {
                    key: "ehrId",
                    value: ehrId
                }
            ]
        };
        $.ajax({
            url: baseUrl + "/demographics/party",
            type: 'POST',
            contentType: 'application/json',
            data: JSON.stringify(partyData),
            success: function (party) {
                if (party.action == 'CREATE') {
                    $("#result").html("Created: " + party.meta.href);
                }
            }
        });
    }
});
```

which returns the following result:

    EHR: 8521e620-d38e-4fd6-9071-f785c2ece9b3

    Created: https://rest.ehrscape.com/rest/v1/demographics/party/104

Search for a patient by name
----------------------------

The purpose of this example is to show how to find a patient by name.

The following API call is used:

-   `POST /demographics/party/query`: searches for a party using the
    party’s name ([see
    details](https://dev.ehrscape.com/api-explorer.html?api=thinkehr&service=/demographics&operation=/demographics/party/query&method=POST&inline=false)).

``` {.js}
$.ajaxSetup({
    headers: {
        "Ehr-Session": sessionId
    }
});
var searchData = [
    {key: "firstNames", value: "Mary"},
    {key: "lastNames", value: "Wilkinson"}
];
$.ajax({
    url: baseUrl + "/demographics/party/query",
    type: 'POST',
    contentType: 'application/json',
    data: JSON.stringify(searchData),
    success: function (res) {
        $("#header").html("Search for Mary Wilkinson");
        for (i in res.parties) {
            var party = res.parties[i];
            var ehrId;
            for (j in party.partyAdditionalInfo) {
                if (party.partyAdditionalInfo[j].key === 'ehrId') {
                    ehrId = party.partyAdditionalInfo[j].value;
                    break;
                }
            }
            $("#result").append(party.firstNames + ' ' + party.lastNames +
                ' (ehrId = ' + ehrId + ')<br>');
        }
    }
});
```

which returns the following result:

    Search for Mary Wilkinson

    Mary Wilkinson (ehrId = 8521e620-d38e-4fd6-9071-f785c2ece9b3)

Search for a patient by ehrId
-----------------------------

The purpose of this example is to show how to find a patient by his or
her EHR ID.

The following API call is used:

-   `POST /demographics/party/query`: searches for a party using a
    patient’s ehrId ([see
    details](https://dev.ehrscape.com/api-explorer.html?api=thinkehr&service=/demographics&operation=/demographics/party/query&method=GET&inline=false)).

``` {.js}
$.ajaxSetup({
    headers: {
        "Ehr-Session": sessionId
    }
});
var searchData = [
    {key: "ehrId", value: "8521e620-d38e-4fd6-9071-f785c2ece9b3"}
];
$.ajax({
    url: baseUrl + "/demographics/party/query",
    type: 'POST',
    contentType: 'application/json',
    data: JSON.stringify(searchData),
    success: function (res) {
        $("#header").html("Search by ehrId 8521e620-d38e-4fd6-9071-f785c2ece9b3");
        for (i in res.parties) {
            var party = res.parties[i];
            $("#result").append(party.firstNames + ' ' + party.lastNames + '<br>');
        }
    }
});
```

which returns the following result:

    Search by ehrId 8521e620-d38e-4fd6-9071-f785c2ece9b3

    Mary Wilkinson

Save a new set of measurements of a patient’s body temperature, blood pressure, height and weight
-------------------------------------------------------------------------------------------------

The purpose of this example is to show how to store a new set of
measurements.

The following API call is used:

-   `POST /composition`: stores a new composition (document) with
    measurements ([see
    details](https://dev.ehrscape.com/api-explorer.html?api=thinkehr&service=/composition&operation=/composition&method=POST&inline=false))

``` {.js}
$.ajaxSetup({
    headers: {
        "Ehr-Session": sessionId
    }
});
var compositionData = {
    "ctx/time": "2014-3-19T13:10Z",
    "ctx/language": "en",
    "ctx/territory": "CA",
    "vital_signs/body_temperature/any_event/temperature|magnitude": 37.1,
    "vital_signs/body_temperature/any_event/temperature|unit": "°C",
    "vital_signs/blood_pressure/any_event/systolic": 120,
    "vital_signs/blood_pressure/any_event/diastolic": 90,
    "vital_signs/height_length/any_event/body_height_length": 171,
    "vital_signs/body_weight/any_event/body_weight": 57.2
};
var queryParams = {
    "ehrId": ehrId,
    templateId: 'Vital Signs',
    format: 'FLAT',
    committer: 'Belinda Nurse'
};
$.ajax({
    url: baseUrl + "/composition?" + $.param(queryParams),
    type: 'POST',
    contentType: 'application/json',
    data: JSON.stringify(compositionData),
    success: function (res) {
        $("#header").html("Store composition");
        $("#result").html(res.meta.href);
    }
});
```

which returns the following result:

    Store composition

    https://rest.ehrscape.com/rest/v1/composition/75c7bb72-0e7f-4a06-983e-73833a5c9615::ehrscape.com::1

Let’s retrieve one of these measurements by using the view call
`GET /view/{ehrId}/blood_pressure`:

``` {.js}
$.ajaxSetup({
    headers: {
        "Ehr-Session": sessionId
    }
});
$.ajax({
    url: baseUrl + "/view/" + ehrId + "/blood_pressure",
    type: 'GET',
    success: function (res) {
        $("#header").html("Blood pressures");
        for (var i in res) {
            $("#result").append(res[i].time + ': ' + res[i].systolic + '/' + res[i].diastolic + res[i].unit + "<br>");
        }
    }
});
```

which returns the following result:

    Blood pressures

    2014-03-19T13:10:00.000Z: 120/90mm[Hg]

Query data using AQL
--------------------

The purpose of this example is to show how to use a very simple
[AQL](http://www.openehr.org/wiki/display/spec/Archetype+Query+Language+Description)
for querying.

The following API call is used:

-   `GET /query`: queries data using AQL ([see
    details](https://dev.ehrscape.com/api-explorer.html?api=thinkehr&service=/query&operation=/query&method=GET&inline=false))

``` {.js}
$.ajaxSetup({
    headers: {
        "Ehr-Session": sessionId
    }
});
var aql = "SELECT c/uid/value as uid, " +
    "c/context/start_time as time, " +
    "c/name/value as name " +
    "FROM EHR[ehr_id/value = '" + ehrId + "'] CONTAINS COMPOSITION c " +
    "ORDER BY c/context/start_time DESC";
$.ajax({
    url: baseUrl + "/query?" + $.param({"aql": aql}),
    type: 'GET',
    success: function (res) {
        $("#header").html("Compositions");
        var rows = res.resultSet;
        for (var i in rows) {
            $("#result").append(rows[i].uid + ': ' + rows[i].name + ' (on ' +
                                rows[i].time.value + ")<br>");
        }
    }
});
```

which returns the following result:

    Compositions

    4f3e47ab-e350-4994-a15e-e819533aba14::ehrscape.com::1: Vital Signs (on 2014-03-09T20:25:20.000+01:00)
    fbc6d182-1720-4e99-aec9-b0f925639c66::ehrscape.com::1: Vital Signs (on 2014-03-07T18:31:54.000+01:00)
    a6568a7e-0cf9-483b-8691-adb46a9fdb6b::ehrscape.com::1: Medications (on 2014-02-09T01:08:27.000+01:00)

Clinical decision support
-------------------------

The purpose of this example is to show how to execute a simple guide
using the CDS API.

The following API call is used:

-   `GET /guide/execute/{guideId}/{ehrIds}`: executes CDS guide ([see
    details](https://dev.ehrscape.com/api-explorer.html?api=thinkcds&service=/guide&operation=/guide/execute/%7BguideId%7D/%7BehrIds%7D&method=GET&inline=false))

``` {.js}
function bmi() {
    return $.ajax({
        url: cdsUrl + "/guide/execute/BMI.Calculation.v.1/" + ehrId,
        type: 'GET',
        headers: {
            "Ehr-Session": sessionId
        },
        success: function (data) {
            if (data instanceof Array) {
                if (data[0].hasOwnProperty('results')) {
                    data[0].results.forEach(function (v, k) {
                        if (v.archetypeId === 'openEHR-EHR-OBSERVATION.body_mass_index.v1') {
                            var rounded = Math.round(v.value.magnitude * 100.0) / 100.0;
                            $("#result").append('BMI: ' + rounded);
                        }
                    })
                }
            }
        }
    });
}

function login() {
    return $.ajax({
        type: "POST",
        url: baseUrl + "/session?" + $.param({username: username, password: password}),
        success: function (res) {
            sessionId = res.sessionId;
        }
    });
}

function logout() {
    return $.ajax({
        type: "DELETE",
        url: baseUrl + "/session",
        headers: {
            "Ehr-Session": sessionId
        }
    });
}

login().done(function() {
    $.when(bmi()).then(logout);
});
```

which returns the following result:

    BMI: 26.08
