1. What is Climate REST API
========================
The Climate Wizard tool (available at http://climatewizard.ciat.cgiar.org/) is a climate data analysis, visualization and dissemination application both for current and future gridded climate data. It provides a wide range of users with synthesized, summarized and packaged climate change information tailored to specific places of interest globally. It is currently developed to provide future projected climate information from downscaled General Circulation Models (GCMs) using data from the Coupled Model Intercomparison Project phase 3 and 5 (CMIP3 and CMIP5). The downscaled approach is based on scientifically accepted bias-corrected statistical downscaling (BCSD) method (Wood et al., 2003; Girvetz et al, 2009). 

The REST API developed here provides a programmatic interface to the Climate Wizard portal. This is designed for querying real-time data to provide stakeholders with climate change information as well as some agro-climatic indices. At the grid-cell (i.e. 0.05 deg ~ 25 km resolution) scale, we calculated a set of indices of crop responses to climate including indicators of drought and heat stress, as well as complementary indicators such as flash-floods (extreme precipitation events) and reductions in crop cycle duration, and overall growing season indicators –i.e. total seasonal precipitation and seasonal mean temperatures. A total of 34 climate indices are provided (see Table 1). 

The input data is currently stored at CIAT (in total 80 GB of GeoTiff raster files) but is available upon request and soon will be available through the CCAFS-Climate portal (http://ccafs-climate.org/). The global database used as input in the API comes from GCMs from the CMIP5, for a historical period and two Representative Concentration Pathways (RCPs): RCP4.5 and RCP8.5. Projections of a total of 16 individual GCMs as well as for the multi-model ensemble mean are provided (see Table 2). 

The API is developed in the Python language, and has been configured to work through Apache in both Linux and Windows operating systems. This has been tested in Ubuntu and CentOS 7 Linux distributions, though it has been designed to work in any Linux distribution. Table 3 summarizes the parameters required for the API. A total of 7 parameters can be provided to specify a particular query, but only 4 out of 7 are required

2. Description of service
-----------
The service returns the specific index value using the latitude and longitude values (in decimal degrees) as inputs, along with other parameters (see below and Table 3). The service is hosted online at:

http://maprooms.ciat.cgiar.org/climatewizard/service

2.1 Parameters
----------
**lat** (float)  (Required) 
    Latitude from the geocoordinate

**lon** (float)  (Required) 
    Longitude from the geocoordinate

**geojson** (json)
    Polygon

**stats** (str)(optional)
    Statistical values that are desired computed separated by commas (,)
    **Permitted values:** min, max, mean, count, sum, median, majority, minority, unique, range, nodata, percentile
    **Default:** min, max, median, mean, percentile_5, percentile_25, percentile_75, percentile_95

**index** (str)  (Required) 
    Climate index
    **Permitted values:** see “Acronym” column of Table 2

**scenario** (str) (Required) 
	Scenario applied
	**Permitted values:** historical, rcp45, rcp85

**gcm** (str) (optional) 
	Model applied
	**Permitted values:** see “Acronym” column of Table 1
	**Default:** ensemble_lowest




**range** (str) (optional)
	Rage of years will be returned, the range must be separated by dash (-). 
	Ex: “2010-2020”
	Default: 
        If scenario = “historical”; range="1976-2005"
        If scenario = “rcp45” or scenario = “rcp85”; range="2040-2069"
        If range= ”false” then all years are extracted

**avg** (boolean) (optional)
	Whether if the result is the average of the range selected or a list of the yearly values
    **Default:** true

**baseline** (str) (optional)
    The baseline period used as reference to compute future projected change, for example “1981–2000”. This parameter can only be specified when scenario = “rcp45” or scenario = “rcp85”. If specified, the arithmetic difference between the future and baseline period is provided.
    **Permitted values:** any 20-year period within the period 1950–2005.
    **Default:** when not specified, change is not computed. That is, the actual future projected values are provided.

**climatology** (boolean) (optional)
    Get monthly data.
    **Note:** parameter "avg" no work with this option, set avg in false.

2.2 Return Values
--------------
The service returns the values in json format. If the data is not found, an error message will be returned.


2.3 Examples
--------
Example #1
----------
Querying the average future projected Cooling Degree Days for the period 2040–2069 (the default period for RCP4.5) for the climate model ACCESS1-0 under RCP4.5.

http://maprooms.ciat.cgiar.org/climatewizard/service?lat=9.58&lon=-74.41&index=CD18&scenario=rcp45&gcm=ACCESS1-0

Output:

.. code-block::

	{
	acronym: "CD18",
	model: "ACCESS1-0",
	-values: (1)[
	-{
	date: "avg_2040-2069",
	value: "464847.6"
	}
	],
	name: "cooling degree days",
	scenario: "rcp45"
	}

Example #2
----------
Querying the average Cooling Degree Days for the period 2006–2099 for the climate model ACCESS1-0 and RCP4.5.

http://maprooms.ciat.cgiar.org/climatewizard/service?lat=9.58&lon=-74.41&index=CD18&scenario=rcp45&gcm=ACCESS1-0&range=false


Output:

.. code-block::

    {
    acronym: "CD18",
    model: "ACCESS1-0",
    -values: (94)[
    -{
    date: 2006,
    value: 411926
    },
    -{
    date: 2007,
    value: 433230
    },

    ETC …

    -{
    date: 2098,
    value: 516298
    },
    -{
    date: 2099,
    value: 500290
    }
    ],
    name: "cooling degree days",
    scenario: "rcp45"
    }


Example #3
----------
Querying the yearly values of Cooling Degree Days for the period 1960–1970 (11 years) for the climate model ACCESS1-0.

http://maprooms.ciat.cgiar.org/climatewizard/service?lat=9.58&lon=-74.41&index=CD18&scenario=historical&gcm=ACCESS1-0&range=1960-1970&avg=false


Output:

.. code-block::

    {
    acronym: "CD18",
    model: "ACCESS1-0",
    -values: (6)[
    -{
    date: 1960,
    value: 3937.16
    },
    -{
    date: 1961,
    value: 3869.43
    },
    -{
    date: 1962,
    value: 3792.88
    },
    -{
    date: 1963,
    value: 3761.62
    },
    -{
    date: 1964,
    value: 3633.05
    },
    -{
    date: 1965,
    value: 3933.23
    }
    ],
    name: "cooling degree days",
    scenario: "historical"
    }

Example #4
----------
Querying the average change in consecutive dry days projected for the period 2041–2060 with respect to the average of a baseline period (1980–2000), for the climate model ACCESS1-0.

http://maprooms.ciat.cgiar.org/climatewizard/service?lat=9.58&lon=-74.41&index=CDD&scenario=rcp45&gcm=ACCESS1-0&range=2041-2060&baseline=1980-2000&avg=true

Output:

.. code-block::

    {
    acronym: "CDD",
    model: "ACCESS1-0",
    -values: (1)[
    -{
    date: avg_2041-2060,
    value: -9.98333333333
    }
    ],
    name: "Consecutive dry days",
    scenario: "rcp45"
    }


Example #5
----------
Querying the zonal statistic (std,percentile_25 and percentile_50) using a polygon in consecutive dry days projected for the period 2007-2017 with respect to the average of a baseline period (1980–2000), for the climate model ACCESS1-0

`http://maprooms.ciat.cgiar.org/climatewizard/service?index=CDD&scenario=rcp45&gcm=ACCESS1-0&range=2007-2017&baseline=1980-2000&geojson={"type":"FeatureCollection","features":[{"type":"Feature","properties":{},"geometry":{"type":"Polygon","coordinates":[[[-75.7177734375,4.061535597066106],[-75.7177734375,5.7690358661221355],[-73.8720703125,5.7690358661221355],[-73.8720703125,4.061535597066106],[-75.7177734375,4.061535597066106]]]}}]}&stats=percentile_25,percentile_50,mean <http://maprooms.ciat.cgiar.org/climatewizard/service?index=CDD&scenario=rcp45&gcm=ACCESS1-0&range=2007-2017&baseline=1980-2000&geojson={"type":"FeatureCollection","features":[{"type":"Feature","properties":{},"geometry":{"type":"Polygon","coordinates":[[[-75.7177734375,4.061535597066106],[-75.7177734375,5.7690358661221355],[-73.8720703125,5.7690358661221355],[-73.8720703125,4.061535597066106],[-75.7177734375,4.061535597066106]]]}}]}&stats=percentile_25,percentile_50,mean>`_

Output:

.. code-block::

    {
    acronym: "CDD",
    model: "access1-0",
    -values: (11)[
    -{
    date: 2007,
    -value: {
    percentile_25: 1100,
    percentile_50: 1100,
    mean: 1203.5714285714287
    }
    },
    -{
    date: 2008,
    -value: {
    percentile_25: 800,
    percentile_50: 1100,
    mean: 1048.2142857142858
    }
    },
    -{
    date: 2009,
    -value: {
    percentile_25: 800,
    percentile_50: 900,
    mean: 855.3571428571429
    }
    },
    -{
    date: 2010,
    -value: {
    percentile_25: 1600,
    percentile_50: 1600,
    mean: 1730.357142857143
    }
    },
    -{
    date: 2011,
    -value: {
    percentile_25: 700,
    percentile_50: 800,
    mean: 921.4285714285714
    }
    },
    -{
    date: 2012,
    -value: {
    percentile_25: 1000,
    percentile_50: 1100,
    mean: 1112.5
    }
    },
    -{
    date: 2013,
    -value: {
    percentile_25: 1475,
    percentile_50: 1700,
    mean: 1607.142857142857
    }
    },
    -{
    date: 2014,
    -value: {
    percentile_25: 700,
    percentile_50: 1100,
    mean: 980.3571428571429
    }
    },
    -{
    date: 2015,
    -value: {
    percentile_25: 800,
    percentile_50: 1400,
    mean: 1157.142857142857
    }
    },
    -{
    date: 2016,
    -value: {
    percentile_25: 700,
    percentile_50: 1300,
    mean: 1135.7142857142858
    }
    },
    -{
    date: 2017,
    -value: {
    percentile_25: 1400,
    percentile_50: 1500,
    mean: 1457.142857142857
    }
    }
    ],
    name: "Consecutive dry days",
    scenario: "rcp45"
    }

Example #6
----------
Querying the monthly maximum temperature for all months of the decade 2030-2040, for the ensemble model (multimodel-mean).

http://maprooms.ciat.cgiar.org/climatewizard/service?lat=3.1&lon=-76.3&index=txx&scenario=rcp85&gcm=ensemble&range=2030-2040&avg=false&climatology=true

Output:

.. code-block::

    {
    acronym: "txx",
    model: "ensemble",
    -values: [
    -{
    date: "2030-1",
    value: 31.920000000000016
    },
    -{
    date: "2030-2",
    value: 32.69
    },
    -{
    date: "2030-3",
    value: 32.650000000000034
    },
    -{
    date: "2030-4",
    value: 32.02000000000004
    },
    {
    date: "2030-5",
    value: 32.150000000000034
    },
    -{
    date: "2030-6",
    value: 32.09000000000003
    },
    -{
    date: "2030-7",
    value: 33.06
    },
    -{
    date: "2030-8",
    value: 33.97000000000003
    },
    -{
    date: "2030-9",
    value: 33.73000000000002
    }
    ],
    name: "Monthly maximum temperatures",
    scenario: "rcp85"    
    }

3. Installing the REST API
=======================
The REST API is deployed as a standard webapp for your servlet container / Apache. The technology used is Python, specifically the libraries GDAL, Bottle and rasterstats.


3.1 APACHE MOD_WSGI

Instead of running your own HTTP server from within Bottle, you can attach Bottle applications to an Apache server using mod_wsgi.
All you need is an app.wsgi file that provides an application object. This object is used by mod_wsgi to start your application and should be a WSGI-compatible Python callable.
File /var/www/html/yourapp/app.wsgi:

.. code-block:: python

	import os
	# Change working directory so relative paths (and template lookup) work again
	sys.path.insert(0, "/var/www/html/yourapp")

	import bottle
	import service
	# ... build or import your bottle application here ...
	# Do NOT use bottle.run() with mod_wsgi
	application = bottle.default_app()

The Apache configuration may look like this:

.. code-block::

    WSGIDaemonProcess yourapp user=ubuntu group=ubuntu processes=1 threads=5
    application-group=%{GLOBAL}
    WSGIScriptAlias /climate /var/www/html/yourapp/app.wsgi
    <Directory /var/www/html/yourapp/app.wsgi>
      WSGIProcessGroup %{GLOBAL}
      WSGIApplicationGroup %{GLOBAL}
      Order deny,allow
      Allow from all
    </Directory>


**Table 1** Indices, acronyms and units used in the REST API

+----------+------------------------------------------------------------------------------------------------------+---------------------------+
|Acronyms  | Description                                                                                          |     Units                 |
+==========+======================================================================================================+===========================+
| TXX      | Annual maximum temperatures                                                                          | Celsius degrees           |
+----------+------------------------------------------------------------------------------------------------------+---------------------------+
| txxi     | Monthly maximum temperatures for month i (e.g. txx1, txx2, txx3, … txx12)                            | Celsius degrees           |
+----------+------------------------------------------------------------------------------------------------------+---------------------------+
| TNN      | Annual minimum temperatures                                                                          | Celsius degrees           |
+----------+------------------------------------------------------------------------------------------------------+---------------------------+
| tnni     | Monthly minimum temperatures for month i (e.g. tnn1, tnn2, tnn3, … tnn12)                            | Celsius degrees           |
+----------+------------------------------------------------------------------------------------------------------+---------------------------+
| tasmax   | Annual mean maximum temperatures                                                                     | Celsius degrees           |
+----------+------------------------------------------------------------------------------------------------------+---------------------------+
| tasmaxi  | Monthly mean maximum temperatures for month i (e.g. tasmax1, tasmax2, tasmax3, … tasmax12)           | Celsius degrees           |
+----------+------------------------------------------------------------------------------------------------------+---------------------------+
| tasmin   | Annual mean minimum temperatures                                                                     | Celsius degrees           |
+----------+------------------------------------------------------------------------------------------------------+---------------------------+
| tasmini  | Monthly mean minimum temperatures for month i (e.g. tasmin1, tasmin2, tasmin3, … tasmin12)           | Celsius degrees           |
+----------+------------------------------------------------------------------------------------------------------+---------------------------+
| tas      | Annual mean average temperatures                                                                     | Celsius degrees           |
+----------+------------------------------------------------------------------------------------------------------+---------------------------+
| tasi     | Monthly mean average temperatures for month i (e.g. tas1, tas2, tas3, … tas12)                       | Celsius degrees           |
+----------+------------------------------------------------------------------------------------------------------+---------------------------+
| TDJF     | Mean temperatures for DJF season                                                                     | Celsius degrees           |
+----------+------------------------------------------------------------------------------------------------------+---------------------------+
| TMAM     | Mean temperatures for MAM season                                                                     | Celsius degrees           |
+----------+------------------------------------------------------------------------------------------------------+---------------------------+
| TJJA     | Mean temperatures for JJA season                                                                     | Celsius degrees           |
+----------+------------------------------------------------------------------------------------------------------+---------------------------+
| TSON     | Mean temperatures for SON season                                                                     | Celsius degrees           |
+----------+------------------------------------------------------------------------------------------------------+---------------------------+
| TWET     | Mean wet / rainy season temperatures                                                                 | Celsius degrees           |
+----------+------------------------------------------------------------------------------------------------------+---------------------------+
| TDRY     | Dry season temperatures                                                                              | Celsius degrees           |
+----------+------------------------------------------------------------------------------------------------------+---------------------------+
| PTOT     | Total precipitation                                                                                  | Millimeters/year          |
+----------+------------------------------------------------------------------------------------------------------+---------------------------+
| pri      | Monthly accumulated precipitation for month i (e.g. pr1, pr2, pr3, … pr12)                           | Millimeters/month         |
+----------+------------------------------------------------------------------------------------------------------+---------------------------+
| PDJF     | Accumulated precipitation for DJF season                                                             | Millimeters/season        |
+----------+------------------------------------------------------------------------------------------------------+---------------------------+
| PMAM     | Accumulated precipitation for MAM season                                                             | Millimeters/season        |
+----------+------------------------------------------------------------------------------------------------------+---------------------------+
| PJJA     | Accumulated precipitation for JJA season                                                             | Millimeters/season        |
+----------+------------------------------------------------------------------------------------------------------+---------------------------+
| PSON     | Accumulated precipitation for SON season                                                             | Millimeters/season        |
+----------+------------------------------------------------------------------------------------------------------+---------------------------+
| PWET     | Wet/rainy season accumulated precipitation                                                           | Millimeters/season        |
+----------+------------------------------------------------------------------------------------------------------+---------------------------+
| PDRY     | Dry season accumulated precipitation                                                                 | Millimeters/season        |
+----------+------------------------------------------------------------------------------------------------------+---------------------------+
| SDII     | Annual simple daily precipitation intensity index                                                    | Millimeters/day           |
+----------+------------------------------------------------------------------------------------------------------+---------------------------+
| sdiii    | Monthly simple daily precipitation intensity index for month i (e.g. sdii1, sdii2, sdii3, … sdii12)  | Millimeters/day           |
+----------+------------------------------------------------------------------------------------------------------+---------------------------+
| SDIIDJF  | Simple daily precipitation intensity index for DJF season                                            | Millimeters/day           |
+----------+------------------------------------------------------------------------------------------------------+---------------------------+
| SDIIMAM  | Simple daily precipitation intensity index for MAM season                                            | Millimeters/day           |
+----------+------------------------------------------------------------------------------------------------------+---------------------------+
| SDIIJJA  | Simple daily precipitation intensity index for JJA season                                            | Millimeters/day           |
+----------+------------------------------------------------------------------------------------------------------+---------------------------+
| SDIISON  | Simple daily precipitation intensity index for SON season                                            | Millimeters/day           |
+----------+------------------------------------------------------------------------------------------------------+---------------------------+
| SDIIDRY  | Dry season simple daily precipitation intensity index                                                | Millimeters/day           |
+----------+------------------------------------------------------------------------------------------------------+---------------------------+
| SDIIWET  | Wet/rainy season simple daily precipitation intensity index                                          | Millimeters/day           |
+----------+------------------------------------------------------------------------------------------------------+---------------------------+
| R02      | Annual number of wet days > 0.2 mm/day                                                               | Number of days            |
+----------+------------------------------------------------------------------------------------------------------+---------------------------+
| R02i     | Monthly number of wet days > 0.2 mm/day for month i (e.g. r021, r022, r023, … r0212)                 | Number of days            |
+----------+------------------------------------------------------------------------------------------------------+---------------------------+
| R02DJF   | Number of wet days > 0.2 mm/day for DJF season                                                       | Number of days            |
+----------+------------------------------------------------------------------------------------------------------+---------------------------+
| R02MAM   | Number of wet days > 0.2 mm/day for MAM season                                                       | Number of days            |
+----------+------------------------------------------------------------------------------------------------------+---------------------------+
| R02JJA   | Number of wet days > 0.2 mm/day for JJA season                                                       | Number of days            |
+----------+------------------------------------------------------------------------------------------------------+---------------------------+
| R02SON   | Number of wet days > 0.2 mm/day for SON season                                                       | Number of days            |
+----------+------------------------------------------------------------------------------------------------------+---------------------------+
| R02WET   | Wet/rainy season number of wet days > 0.2 mm/day                                                     | Number of days            |
+----------+------------------------------------------------------------------------------------------------------+---------------------------+
| R02DRY   | Dry season number of wet days > 0.2 mm/day                                                           | Number of days            |
+----------+------------------------------------------------------------------------------------------------------+---------------------------+
| R5D      | Maximum consecutive 5-day of precipitation                                                           | Integer                   |
+----------+------------------------------------------------------------------------------------------------------+---------------------------+
| CD18     | Cooling degree days                                                                                  | Number of days            |
+----------+------------------------------------------------------------------------------------------------------+---------------------------+
| CDD      | Consecutive dry days                                                                                 | Integer                   |
+----------+------------------------------------------------------------------------------------------------------+---------------------------+
| DROI     | Standardized precipitation index (SPI)                                                               | Float                     |
+----------+------------------------------------------------------------------------------------------------------+---------------------------+
| DROF     | Monthly frequency of SPI < -1.5 (Severe dryness) by year                                             | Probability of occurrence |
+----------+------------------------------------------------------------------------------------------------------+---------------------------+
| GSL      | Growing period length                                                                                | Number of days            |
+----------+------------------------------------------------------------------------------------------------------+---------------------------+
| GD10     | Growing degree days                                                                                  | Number of days            |
+----------+------------------------------------------------------------------------------------------------------+---------------------------+
| HD18     | Heating degree days                                                                                  | Number of days            |
+----------+------------------------------------------------------------------------------------------------------+---------------------------+
| HWDI     | Heat wave duration index                                                                             | Number of days            |
+----------+------------------------------------------------------------------------------------------------------+---------------------------+


**Table 2** Description of the CMIP5 Global Climate Models available in the REST API. Note that the case for the acronyms should be kept when using the API. Spelling errors (including lower/upper case use) will result no data being returned.

+-------------+-----------------+-------------------------------------------------------------------------------------------------------------------------------+
|Acronym      | Full Name Model | Institute                                                                                                                     |
+=============+=================+===============================================================================================================================+
|bcc-csm1-1   | BCC-CSM1.1      | Beijing Climate Center, China Meteorological Administration                                                                   |
+-------------+-----------------+-------------------------------------------------------------------------------------------------------------------------------+
|BNU-ESM      | BNU-ESM         | Beijing Normal University                                                                                                     |
+-------------+-----------------+-------------------------------------------------------------------------------------------------------------------------------+
|CanESM2      | CCCMA-CanESM2   | Canadian Centre for Climate Modelling and analysis                                                                            |
+-------------+-----------------+-------------------------------------------------------------------------------------------------------------------------------+
|CESM1-BGC    | CESM1-BGC       | National Science Foundation, Department of Energy, National Center for Atmospheric Research                                   |
+-------------+-----------------+-------------------------------------------------------------------------------------------------------------------------------+
|MIROC-ESM    | MIROC-ESM       | University of Tokyo, National Institute for Environmental Studies and Japan Agency for Marine-Earth Science and technology    |
+-------------+-----------------+-------------------------------------------------------------------------------------------------------------------------------+
|CNRM-CM5     | CNRM-CM5        | Centre National de Recherches Meteorologiques and Centre Europeen de Recherche et Formation Avancees en Calcul Scientifique   |
+-------------+-----------------+-------------------------------------------------------------------------------------------------------------------------------+
|ACCESS1-0    | CSIRO-ACCESS1.0 | Commonwealth Scientific and Industrial Research Organization (CSIRO) and Bureau of Meteorology (BOM), Australia               |
+-------------+-----------------+-------------------------------------------------------------------------------------------------------------------------------+
|CSIRO-Mk3-6-0| CSIRO-Mk3.6.0   | Queensland Climate Change Centre of Excellence and Commonwealth Scientific and Industrial Research Organization               |
+-------------+-----------------+-------------------------------------------------------------------------------------------------------------------------------+
|GFDL-CM3     | GFDL-CM3        |  NOAA Geophysical Fluid Dynamics Laboratory                                                                                   |
|GFDL-ESM2G   | GFLD-ESM2G      |                                                                                                                               |
|GFDL-ESM2M   | GFLD-ESM2M      |                                                                                                                               |
+-------------+-----------------+-------------------------------------------------------------------------------------------------------------------------------+
|inmcm4       | INM-CM4         | Institute of Numerical Mathematics of the Russian Academy of Sciences                                                         |
+-------------+-----------------+-------------------------------------------------------------------------------------------------------------------------------+
|IPSL-CM5A-LR | IPSL-CM5A-LR    | Institut Pierre Simon Laplace                                                                                                 |
|IPSL-CM5A-MR | IPSL-CM5A-MR    |                                                                                                                               |
+-------------+-----------------+-------------------------------------------------------------------------------------------------------------------------------+
|CCSM4        | NCAR-CCSM4      | US National Centre for Atmospheric Research                                                                                   |
+-------------+-----------------+-------------------------------------------------------------------------------------------------------------------------------+
|ensemble     | Ensemble mean   | –                                                                                                                             |
+-------------+-----------------+-------------------------------------------------------------------------------------------------------------------------------+


**Table 3** Parameters required for using the REST API

+----------+---------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+----------------------+----------+
| Param.   | Type    | Description                                                                                                                                                                                            |Default               | Required |
+==========+=========+========================================================================================================================================================================================================+======================+==========+
| lat      | float   | Latitude (in decimal degrees) of the site of interest                                                                                                                                                  | –                    | Yes      |
+----------+---------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+----------------------+----------+
| lon      | float   |  Longitude (in decimal degrees) of the site of interest                                                                                                                                                | –                    | Yes      |
+----------+---------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+----------------------+----------+
| index    | str     | Acronym of the index (first column of Table 2)                                                                                                                                                         | –                    | Yes      |
+----------+---------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+----------------------+----------+
| scenario |   str   | Climate scenario (historical, rcp45, rcp85)                                                                                                                                                            | –                    | Yes      |
+----------+---------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+----------------------+----------+
| gcm      |  str    | Global Climate Model (first column of Table 1)                                                                                                                                                         | ensemble             | No       |
+----------+---------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+----------------------+----------+
| range    |  str    | Range of years for which data is to be extracted (false for all years)                                                                                                                                 | 1976-2005 2040-2069  | No       |
+----------+---------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+----------------------+----------+
| avg      | boolean | Whether or not the average of the years requested is to be provided                                                                                                                                    | true                 | No       |
+----------+---------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+----------------------+----------+
| baseline |  str    | Baseline period used as a reference to calculate future projected change. Must be at least 20 years. If this parameter is not specified, the actual value (instead of the change) will be provided.    | –                    | No       |
+----------+---------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+----------------------+----------+

