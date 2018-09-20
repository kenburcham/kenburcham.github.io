Tinkering with Code Blocks
==========================

After googling a bit for Tinkerer and reStructuredText documentation, it would seem that it should be easy to include some nice looking, syntax highlighted code blocks in your blog posts. Buuut it still doesn't seem completely clear and I probably won't immediately remember so let's consider this a cheatsheet / sandbox for Tinkerer code block experiments.

Of course, we might consider trying or editing our stylesheet, but I'm interested in what the default Tinkerer makes possible.

Single backticks: I can italicise using backticks `around some text` and that's that.

Double backticks: I can use double backticks ``and get a grey background`` which is nice for inline key words.

For a code snippet, just add a double colon to the end of a paragraph, give a new line and then indent the next paraphgraph and you get this::

    And then it is a literal block with python syntax highlighting.

Note: if you don't indent the paragraph that follows the double colon it leaves it all literal:: 
and mashes the line right in with the double colon.

Now, this paragraph ends with a double colon with some python code::

    import requests

    payload = { 
	    'Username' : 'kenb',
	    'Password' : "password"
    }

And then just un-indent and carry on with your text copy to break back out. That's nice, in that you don't have to wrap or anything! 

That was fun, so here is more code::

    login_url = "http://10.10.10.63:8082/services/api/v1/account/login"

    with requests.Session() as s:
	    p = s.post(login_url, data=payload)
	
	    r = s.get("http://10.10.10.63:8082/services/api/v1/account/whoami")
	
	    user = r.json()
	    print(user["Fullname"]) #prints "Ken Burcham"

And now for a final code snippet::

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



.. author:: default
.. categories:: none
.. tags:: none
.. comments::
