ETL, Python + REST Style
========================

One of the main projects I work on we call "CDMS" for Centralized Database Management System. Very creative name, but somehow it stuck. CDMS (`github <https://github.com/ctuir/cdms>`_) is a web application we use to store all of the natural resource data (as well as other data) for the CTUIR organization. 

Last week I was given the task to create a map that would show the 3-day maximum water temperature at a variety of locations as well as the 3-day average discharge (water flow in cfs) at a set of locations tracked by the `Walla Walla Basin Watershed Council <https://www.wwbwc.org/>`_. 

The WWBWC were very accommodating to provide us with links to 7-day data exports from their system and so we want to pull that data in, parse it and then push it into the CDMS.

The first task was to login to our own system with a session cookie so that we would be able to POST the data to our API data import endpoint::

    import requests

    login_url = "http://cdms-test:8082/services/api/v1/account/login"
    login_payload = { 
	    'Username' : 'ken',
	    'Password' : 'secret'
    }

    with requests.Session() as s:
	    p = s.post(login_url, data=login_payload)
	
	    r = s.get("http://cdms-test:8082/services/api/v1/account/whoami")
	
        #do a little test
	    user = r.json()
	    print("logged in as " + user["Fullname"])

The ``requests`` module is quite slick with a delightfully terse syntax.

From there, I can fetch and parse the datafile from the WWBWC to prepare for loading into CDMS. Since I have a few sites to process data for, I'll configure them in an list::

    files_to_load = [
	    {
		    "LocationId": "15925",
		    "InstrumentId": "265",
		    "URL" : "http://wwbwc.org/images/StreamFlowSites/S108_7_Day_Discharge.txt",
		    "MeasureColumn": 'WaterFlow'
	    },
	    {
		    "LocationId": "15926",
		    "InstrumentId": "266",
		    "URL" : "http://wwbwc.org/images/StreamFlowSites/S105_7_Day_Discharge.txt",
		    "MeasureColumn": "WaterFlow"
	    },
        ...
        {
		    "LocationId": "15925",
		    "InstrumentId": "278",
		    "URL" : "http://wwbwc.org/images/StreamFlowSites/S108_7_Day_WaterTemp.txt",
		    "MeasureColumn": 'WaterTemperature'
	    },
	    {
		    "LocationId": "15926",
		    "InstrumentId": "279",
		    "URL" : "http://wwbwc.org/images/StreamFlowSites/S105_7_Day_WaterTemp.txt",
		    "MeasureColumn": "WaterTemperature"
	    },
        ...
    ]

In our system, I've defined the locations and the instruments that this data is going to be represented from and put in the ID's. Since for some files the measurement is water temperature and others the measurement is flow, I also have an parameter for which field is the target field in our system.

Then we can iterate the list of files to load, build up our payload, then make a POST to our own endpoint to save the data::

    for file in files_to_load:

		payload = {
			'LocationId' : file['LocationId'],
			'InstrumentId' : file['InstrumentId'],
			'Header': {
				"FieldActivityType": "Data File Upload",
				"CollectionType" : "Surface water",
				"SamplePeriod" : "15min",
			},
			'Details': []
		}

		print("-------------------------------")
		print("fetching " + file["URL"])
		r = requests.get(file["URL"])

		on_line = 1

		for line in r.iter_lines():
			if on_line < lines_to_skip:
				on_line += 1
			else:
				on_line += 1
				words = line.decode("UTF-8").split()

				if(len(words)==0):
					continue

				try:
					measure_date = datetime.strptime(words[0]+" "+words[1],"%Y-%m-%d %H:%M")
				except: 
					continue
				
				#verify the measure is a valid number (float)
				if(yesterday.strftime('%m%d%y') == measure_date.strftime('%m%d%y') and isfloat(words[2])):
					payload["Details"].append({
						'ReadingDateTime' : measure_date.strftime("%Y-%m-%d %H:%M"),
						file["MeasureColumn"] : words[2] 
					});
				

		print("Total lines of data in file: " + str(on_line))
		print("Total details from yesterday to push: " + str(len(payload["Details"])))

		if(len(payload["Details"]) > 0):
			print("pushing to " + service_url)
			payload_wrapper = {"DatasetId": dataset_id, "UserId": user["Id"], "activities" : {"1": payload}}
			p = s.post(service_url, json=payload_wrapper)

			print (p.text)
		else:
			print ("nothing to do.")

Since our endpoint requires the login, this must all be under the ``with requests.Session() as s:`` block.

Conclusion
==========

Running the script logs into our API service, fetches and parses each file, builds up a JSON payload object from the data records that match yesterday's date... then it posts the payload to our server.

.. author:: default
.. categories:: none
.. tags:: none
.. comments::
