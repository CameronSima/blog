title: Generating Poetry with Python, pt. I
---
### Links:
[Live Example](http://wordsmith.es/poetry_generator)
[Full Project Code](https://github.com/CameronSima/python_poetry_generator)

Writing isn't fun. It's tedious and time-consuming. One of the neat things about computer programming is it allows you to make difficult, slow, and boring things much less so. Want to write a blog/novel/book of poetry without having to worry about all that...writing? Computers are smart -- why can't they do all the work for us?

Such were the sort of thoughts that led me to the topic of the [Markov Chain](https://en.wikipedia.org/wiki/Markov_chain), a model which outlines a sequence of possible events where the probability of each event depends on the state of the previous event. Basically, it's an algorithm which detects patterns statelessly. From Wikipedia: 

>"A Markov chain, named after Andrey Markov, is a random process that undergoes transitions from one state to another on a state space. It must possess a property that is usually characterized as "memorylessness": the probability distribution of the next state depends only on the current state and not on the sequence of events that preceded it. This specific kind of "memorylessness" is called the Markov property. Markov chains have many applications as statistical models of real-world processes."


After [studying an actual implementation](http://agiliq.com/blog/2009/06/generating-pseudo-random-text-with-markov-chains-u/) in Python, I was inspired to create [Pastiche](http://www.wordsmith.es/poetry_generator/) -- a web app that composes pseudo-poetry based on user-selected poetry styles.

While still a work in progress, what Pastiche does is this:

- Creates a large corpus of data based on user selection. For example, if a user selects 'Emily Dickinson', the app will pull from a file containing the complete works of Emily Dickinson (the larger the corpus, the better this works). If a user selects 'Emily Dickinson' and 'Charles Bukowski', it will combine the two into a single file. (Part of the novelty, at least to me, is to see what comes of strange combinations of styles.)

- Feeds the data through the markov algorithm which creates a dictionary in which keys are a tuple containing every two-word sequence in the text, and which values are a list of all the words which follow them. For example:

Given the text:

>"I do not like them, Sam I am. I do not like Green Eggs and Ham",

we would get:

``` bash 
{('I', 'do'): ['not', 'not'], 
('do, not'): ['like', 'like'], 
('not', 'like'): ['them', 'Green'], 
('like', 'them'): ['Sam'], 
('them', 'Sam'): ['I'], 
('Sam', 'I'): ['am'], 
('I', 'am'), ['I'], 
('am', 'I'): ['do'], 
('like', 'Green'): ['Eggs'],
('Eggs', 'and'): ['Ham']}
```

- The markov chain will select a key from the dictionary, then an element from the list in its value randomly. We can see that the algorithm weighs the probability of each 'following word' simply by virtue of its frequency in the list. It goes on, taking the last two words in the now three-word long 'composition', finds that in the dictionary...and so on.

This works surprisingly well to create only sort-of nonsensical gibberish, and can sometimes impress with flourishes of electronic elegance. Some keepers:

> Armies, emblem of the day; a blackbird bates his jargoning for the passing calvary. Auto-da-fe and judgment are nothing to me before the stake. Respite is no more.

> O soul, we have positively gain'd a hearing, to far more than the gown of orchids in the wind, a few light kisses, a few well held hands to hasten holding.

> Cut the hawsers — haul out — shake out your sails, Steer straight toward Boston Bay. Now call for the eternal march of firemen in their cradles.


