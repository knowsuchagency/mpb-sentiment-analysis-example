# mpb-sentiment-analysis-example
Toying around with reddit and textblob to get data and perform sentiment analysis.
[![Binder](http://mybinder.org/badge.svg)](http://mybinder.org:/repo/knowsuchagency/mpb-sentiment-analysis-example)


## What does Reddit think about the new MBP?
### Is it da bomb, or did it just bomb?

My friends and I have been talking today about Apple's announcement of their new Macbook Pro line.
I personally own an Apple TV, Ipad mini, Macbook Pro, and an iPhone. I was definitely looking forward
to see what Apple was going to come out with the new Macbook Pro line. My thoughts on the announcement aside,
it seemed to me like the overwhelming majority of users on Reddit [didn't come away very impressed](https://www.reddit.com/r/apple/comments/59plnp/lets_talk_about_those_prices/) with the announcements Apple made in regards to the new Macbook Pro. I thought this would be a good opportunity to play with Reddit's API and to try out some rudimentary sentiment analysis.

## Tech

To talk to Reddit, I'm using the aptly named, [**PRAW**](https://praw.readthedocs.io/en/stable/index.html) library. PRAW stands for the Python Reddit Api Wrapper. Nice.

Now for the sentiment analysis, I'm going to use the [**TextBlob**](https://textblob.readthedocs.io/en/dev/) library. TextBlob is a library that provides an easy to use interface for a lot of common natural language processing tasks.

## Get the data

To begin, we instantiate praw's Reddit class with an appropriate user agent. Be careful how you define your user agent. Doing so incorrectly can get you banned according to Reddit. Fortunately, writing a user agent isn't hard at all. You can find the official documentation on how to do so on the [Reddit API wiki page](https://github.com/reddit/reddit/wiki/API). 

As of this writing, the format should look something like this (taken from the aformentioned wiki):

    <platform>:<app ID>:<version string> (by /u/<reddit username>)

Example: 

    User-Agent: android:com.example.myredditapp:v1.2.3 (by /u/kemitche)



```python
import praw

# I saved my user agent string in a local file
# since one should use their own
with open('./.user_agent') as f:
    user_agent = f.read()

# instantiate Reddit connection class
r = praw.Reddit(user_agent=user_agent)

# let's get the current top 10 submissions
# since praw interacts with reddit lazily, we wrap the method
# call in a list
submissions = list(r.get_subreddit('apple').get_hot(limit=10))
[str(s) for s in submissions]
```




    ['375 :: Apple Keynote, October 2016 | Post-Event Megathread',
     '44 :: Pre-order Shipping Megathread | Mac Lineup, 2016',
     "4942 :: Let's talk about those prices...",
     "3027 :: I dont mind the lack of standard USB, but you're telling me I can't u...",
     '1759 :: Tim Cook: "We think technology should be available to everyone" and t...',
     '1147 :: (UK) 13" MacBook Pro has gone from £999 to £1449',
     "741 :: What is Apple's definition of a pro?",
     "725 :: Let's talk about the touch bar.",
     '617 :: Gone is lit Apple logo on the back of Macbook Pro',
     '487 :: What are you thoughts on the new MacBook Pros? Being honest I am not i...']



Cool, so we have some data. From looking at the actual subreddit in a browser, we can see the two top submissions are official threads so we'll just skip over them.

<a href="http://imgur.com/8MJXgdg"><img src="http://i.imgur.com/8MJXgdg.png" title="source: imgur.com" /></a>


```python
submissions = submissions[2:]
[str(s) for s in submissions]
```




    ["4942 :: Let's talk about those prices...",
     "3027 :: I dont mind the lack of standard USB, but you're telling me I can't u...",
     '1759 :: Tim Cook: "We think technology should be available to everyone" and t...',
     '1147 :: (UK) 13" MacBook Pro has gone from £999 to £1449',
     "741 :: What is Apple's definition of a pro?",
     "725 :: Let's talk about the touch bar.",
     '617 :: Gone is lit Apple logo on the back of Macbook Pro',
     '487 :: What are you thoughts on the new MacBook Pros? Being honest I am not i...']



### Submission 1

We'll start by looking at the comments in the first submission about Apple's new pricing. Can you guess what people think?!

In praw, every Submission has a comments attribute which is iterable. This attribute isn't homogeneous. That is, some of the items will be Comment objects, and there may be a MoreComments class in there as well, so we'll need to handle that.


```python
# grab the first submission
submission = submissions[0]

# the actual text is in the body
# attribute of a Comment
def get_comments(submission, n=20):
    """
    Return a list of comments from a submission.
    
    We can't just use submission.comments, because some
    of those comments may be MoreComments classes and those don't
    have bodies.
    
    n is the number of comments we want to limit ourselves to
    from the submission.
    """
    count = 0
    def barf_comments(iterable=submission.comments):
        """
        This generator barfs out comments.
        
        Some comments seem not to have bodies, so we skip those.
        """
        nonlocal count
        for c in iterable:
            if hasattr(c, 'body') and count < n:
                count += 1
                yield c.body
            else:
                # Sometimes c here will be a Comment
                # class which is not iterable.
                if hasattr(c, '__iter__'):
                    yield from barf_comments(c)
                else:
                    # c was a Comment and did not have a body
                    continue
    return barf_comments()
                
comments = list(get_comments(submission))
list(comments)
```




    ['I\'m still trying to figure out what justifies a $200 increase on the 13" ***without*** the Touch Bar...',
     "Probably shouldn't have been surprised, but it hit me like a slippery fish. My jaw dropped.\n\nThey've priced me out, sadly.",
     '$4300 to max out the 15 pro...lol',
     "I was waiting for the new Macbooks to replace my water damaged 2015 Macbook Pro.\n\nNow, I'm off to Apple Store to get my Macbook fixed. These prices are insane.",
     'As an Australian, I am expecting to be even less thrilled when regional pricing is revealed. ',
     '2000 euros here in Germany. I am shocked. Really',
     "Ain't paying that much. May have to stick with my base model 2012 air :(",
     "Even without the TouchBar, it is still $1500. \n$200 over the last year's MacBook Pro. \nThe price increase is just insane. ",
     "Unfortunately Apple deserves a stiff reality check from consumers after this.  They'll keep inflating prices with minimal improvements unless we make a statement by NOT BUYING their stuff.  They need to get grounded in the sense that they're just not far ahead of the competition as far as aesthetics go anymore.  There's no reason to pay thoese prices except for that logo, and they hope that's reason enough that you'll buy it.  They're so wrong.\n\nRemember when Microsoft announced Xbox One with the DRM...remember the outcry and the lousy sales actually prompted a positive change from Microsoft and since then, the Xbox and MS has completely recovered.  We need to give Apple the same reality check, because it'll help them find their way again.\n\nDo not buy this crap.",
     'Well the Macbook that I am typing on will be the last one that I use...',
     '$2399USD for a Quad Core i7, 16GB of 2133Mhz RAM, 256GB SSD, and an AMD 400 mobile series, but not even the 480RX equivalent?\n\nFuck right off. ',
     "As expected, this has made Microsoft's Surface line extremely attractive.",
     "I was getting hyped for my new Macbook Pro for the entire keynote only to find out that it's $500 out of my budget at the end. It's like a mouse being lured in by cheese only to get whacked by a mousetrap.",
     'Rip students...',
     "What. The. Fuck.\n\n+ $200 for better processor? okay I guess, I can live with that\n\n+ $400 for 1TB of space? seems a bit much but okay\n\n+ **$1200 for 2TB?!** Are they fucking joking?\n\n+ $100 for better graphics? seems pointless, they could just include the 460 version by default, they are just milking money at this point\n\nSad, really sad. It's been two years and if we want a better model than the 2015, we need to pay about $1000 extra...\n\nApple products are already the most expensive products on the market and it seems like they are heading for a direction where everything is just going to get more expensive. Wouldn't be surprised if the next best model of iPhone would be $1500 or something like that.",
     'The base price with 512GB I would have accepted, but 256 is way too low. \n\nAlso a Canadian\n\nedit: GB, sorry typo',
     'This is the mac mini all over again.',
     'By far the most disappointing Apple event ever.',
     'That price is STUPID. I was really excited, especially with the space gray, but the price makes me not want this at all. ',
     "let's talk about releasing two flagship products within months of each other that require a dongle to work together."]



## Sentiment analysis!

So now we have the first twenty comments of the first submission.

We'll combine them into one piece of text and determine the overall sentiment from them.

According to the TextBlob docs, this is how to use their sentiment analysis api and how to interpret it


### The sentiment property returns a namedtuple of the form Sentiment(polarity, subjectivity). The polarity score is a float within the range [-1.0, 1.0]. The subjectivity is a float within the range [0.0, 1.0] where 0.0 is very objective and 1.0 is very subjective.

```
>>> testimonial = TextBlob("Textblob is amazingly simple to use. What great fun!")
>>> testimonial.sentiment
Sentiment(polarity=0.39166666666666666, subjectivity=0.4357142857142857)
>>> testimonial.sentiment.polarity
0.39166666666666666
```


```python
from textblob import TextBlob

comment_blob = TextBlob(''.join(comments))
```

    /Users/stephanfitzpatrick/.pyenv/versions/3.5.1/envs/py35/lib/python3.5/site-packages/nltk/decorators.py:59: DeprecationWarning: inspect.getargspec() is deprecated, use inspect.signature() instead
      regargs, varargs, varkwargs, defaults = inspect.getargspec(func)





    Sentiment(polarity=-0.06283927712499143, subjectivity=0.5978612657184086)




```python
print(comment_blob.sentiment)
```

    Sentiment(polarity=-0.06283927712499143, subjectivity=0.5978612657184086)


Huh, things aren't looking good so far. Well, let's look at more data.


```python
more_comments = []
```


```python
for submission in submissions:
    more_comments.extend(get_comments(submission, 200))
```


```python
len(more_comments)
```




    404



I figure there are many more than 404 comments for those submissions. I suspect we only get that many because praw [tries to do the right thing](https://praw.readthedocs.io/en/stable/pages/faq.html) and follow Reddit's guidelines for the amount of requests one can make within a given time limit. We're not going to worry about that now. 404 comments is good enough since we're only tinkering :)


```python
bigger_blob = TextBlob(''.join(more_comments))

# The first time I ran this method, it failed
# because I hadn't read TextBlob's docs closely
# and downloaded the corpus of text in needed.
# python -m textblob.download_corpora

print(len(bigger_blob.words))
```

    17046


Let's see what some of the most common words are


```python
from collections import Counter

counter = Counter(bigger_blob.words)

# the most common words are pretty mundane common parts of speech, so we'll skip the first few
counter.most_common()[60:100]
```




    [('no', 43),
     ('about', 42),
     ('has', 42),
     ('can', 39),
     ('there', 39),
     ('when', 38),
     ('from', 37),
     ('screen', 37),
     ('iPhone', 37),
     ('going', 37),
     ("'re", 36),
     ('Touch', 36),
     ('out', 35),
     ('even', 35),
     ('only', 34),
     ('15', 32),
     ('been', 32),
     ('too', 31),
     ('buy', 30),
     ('know', 30),
     ('we', 30),
     ('than', 30),
     ('up', 29),
     ('13', 29),
     ('people', 28),
     ('because', 28),
     ('time', 28),
     ('same', 27),
     ('They', 27),
     ('model', 27),
     ('which', 27),
     ('way', 26),
     ('other', 26),
     ('using', 26),
     ('prices', 26),
     ('still', 25),
     ('see', 25),
     ('by', 25),
     ('down', 25),
     ('Bar', 25)]



### Finally, let's see what the over all sentiment analysis is


```python
bigger_blob.sentiment
```




    Sentiment(polarity=0.07565960941292733, subjectivity=0.52062214346886)



## In conclusion

We've hopefully learned a little more about communicating with Reddit using Python and doing some simple sentiment analysis on the content there. This wasn't meant to be a very scientific excercise, but I thought it was a fun way to play around with [PRAW](https://praw.readthedocs.io/en/stable/index.html) and [TextBlob](https://textblob.readthedocs.io/en/dev/index.html). Both libraries are really powerful and simple to use and I can definitely see myself taking advantage of them a lot more in the future. 


```python

```
