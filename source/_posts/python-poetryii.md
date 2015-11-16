title: Generating Poetry with Python, pt. II
---
### Links:
[Live Example](http://wordsmith.es/poetry_generator)
[Full Project Code](https://github.com/CameronSima/python_poetry_generator)

In the introduction, we went over some basics of Markov chains, an implementation in Python, and a small app that utilizes it. Here, we'll get into some of the actual code that makes the app I made with it, [Pastiche](http://www.wordsmith.es/poetry_generator/), tick.

The Python Markov chain object, contained in file `markov.py`, is based heavily on an example found on the [Agiliq Blog](http://agiliq.com/blog/). 

``` bash
class Markov(object):
	
	def __init__(self, open_file):
		self.cache = {}
		self.words = open_file.split()
		self.word_size = len(self.words)
		self.database()
	
	def triples(self):
		if len(self.words) < 3:
			return	
		for i in range(len(self.words) - 2):
			yield (self.words[i], self.words[i+1], self.words[i+2])
			
	def database(self):
		for w1, w2, w3 in self.triples():
			key = (w1, w2)
			if key in self.cache:
				self.cache[key].append(w3)
			else:
				self.cache[key] = [w3]
				
	def generate_markov_text(self, size=30):
		seed = random.randint(0, self.word_size-3)
		seed_word, next_word = self.words[seed], self.words[seed+1]
		w1, w2 = seed_word, next_word
		gen_words = []

		for i in xrange(size):
			gen_words.append(w1)
			
			w1, w2 = w2, random.choice(self.cache[(w1, w2)])
		gen_words.append(w2)
		self.text = ' '.join(gen_words)
		return self.text
```

Next, we'll make a small `app.py` file using [Bottle.py](http://bottlepy.org/docs/dev/index.html), a great web micro-framework for Python. To use the framework, you simply need to download [bottle.py](https://raw.githubusercontent.com/bottlepy/bottle/master/bottle.py) into your project directory.

```bash
from bottle import route, run, template, get, post, redirect, request, TEMPLATE_PATH, static_file

import markov

STATIC_ROOT = '/path/to/project/static'
TEMPLATE_PATH.insert(0, os.path.abspath(os.path.join(os.path.dirname(__file__), "views")))

@route('/static/<filename>')
def server_static(filename):
    return static_file(filename, root=STATIC_ROOT)

@get('/')
def page():
    try:
        return template('mainPage', poem=poem)
    except:
        return template('mainPage', poem="Oops...Something went wrong. Please try again!")

@post('/')
def enter_score():
    global poem
    poem = markov.write(request.forms)
    redirect('/')

run(host='localhost', port=8080, debug=True)

```
The important thing to note here is that Bottle will receive POST requests from the client from the `@post('/')` function via `requests.forms`, which will return a list of poets' names selected  by the user.

Finally, we'll pass `requests.forms` to our `write()` function, which we'll now add to `markov.py`:

``` bash
FILENAME_PREFIX = '/path/to/project/'
...
def write(poets):
	if not poets:
		return "Please select at least one poet!"
	combined_text = ''

	for poet in poets:
		fname = FILENAME_PREFIX + poet + '.txt'
		for line in open(fname):
			combined_text += line
	
	markov = Markov(combined_text)
	combined_text = ''
	markov.generate_markov_text()
	return markov.text
	```

If the user hasn't selected any names and `request.forms` is empty, we ask the user to select at least one poet. For each of the names in `poets`, we open the `.txt` file containing their works line by line, and add the lines to string `combined_text`. We instantiate a new `Markov` object, passing in `combined_text`, then call the `generate_markov_text()` method to get our line of poetry.

The [complete project](https://github.com/CameronSima/python_poetry_generator) contains functions for formatting, scraping the web for additional texts, and a basic front-end layout using Bootstrap. Eventually, I will add a database where users can save lines that come out particulary well, as well as a voting system.
