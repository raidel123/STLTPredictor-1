# Using Data Analysis to Evaluate Companies on a Short and Long Term Scale
Stock market trends are no longer affected solely by the actions of majority shareholders. With the advent of social media, news and sentiment about a company spread quickly and affect daily stock evaluations. To monitor and accurately predict daily stock market changes, we will be examining social media data, classifying the data based on sentimental value. Our hypothesis is that social media trends will have an effect on the daily fluctuation of the stock market and can create a model for accurate daily predictions. While we predict that social media trends will play a big part in accurate forecasts of the stock market, we do not want to overlook the power board members can have on a company’s evaluation. To do this, we will be analyzing video interviews of board members from public companies in order to monitor their emotional status. We believe that by being able to detect the target’s emotional status, we will be able to make great estimates on that company’s earnings, allowing potential users of this software to make a much more informative decision.

# Dependencies
Requires [Docker](https://www.docker.com) to be running on your machine

# Installation
Clone repository with `git clone https://github.com/tckelly38/STLTPredictor.git`

run `docker build -t myapp .` (~7-8 minutes depending on internet connection and CPU)

You will also need to have a valid twitter account set up and available to use for the twitter extraction. The appropriate data should be set up in `/twitterExtraction/data.py`

# Usage
run `docker run -it myapp /bin/bash` to run the docker image in an interactive mode

Then to test if working:
run `python worker.py NKE_2012-08-16.mp4` (~30s depending on hardware) and you should get the following results:
![Results](https://i.imgur.com/uiBt32D.png)

If you want to run on some other video, title the video as SYM_YYYY-MM-DD.mp4 where SYM is the stock ticker symbol for the video of the company, and the YYYY-MM-DD is the air date of the video. Try to shorten video if possible or computation time can be quite lengthy.

# Limitations
In detail description of limitation in conclusion section.

* Emotional classifier was self built, and as a consequence of time limitation, is not robust enough to give proper classifications. We reached out to a company with a commercial database of emotion based images but our request was denied.

* Docker makes it difficult to display images, so no images can be displayed on run through.

* Data source is not large enough to give truly accurate results. Acquiring viable videos is a manual process.

# Components
There are three main modules to the application which synchronize their results together

## twitterExtraction
Using Twitter’s official API, extract a series of tweets containing certain keywords (company name, company stock symbol) and analyze said tweet. The process of finding sentiment for a single tweet is quite straightforward, clean the tweet of extraneous information which includes stop words (words with little significance in determining sentiment), and other superfluous aspects of a modern sentence structure including irrelevant punctuation. Once the tweet has been cleaned, we can pass this off to a natural language processing library which will send back a sentiment value for the tweet. In particular, we used a python library called TextBlob, which is a general text based library that has a lot of useful components. We just use the NLP part of this library, which allows us to judge the sentiment for a specific tweet. The basics of sentiment analysis are this: TextBlob has a database of most words in the english dictionary, each one assigned a polarity and objectivity score. Polarity is based on how polar a certain word is in the range of [-1, 1] and objectivity is how objective or subjective a word is on the range [-1, 1]. For the purposes of this project, we exclusively look at the polarity, as we found it to be the best measure for determining the general sentiment about a company. TextBlob than utilizes a popular tool called PatternAnalyzer which takes into account the order and structure of the tweet, and uses that information to better inform the polarity score. This makes it so a tweet’s overall sentiment is not just the sum of its parts, but instead takes into account the structure as well.

Once we receive the score of an individual tweet, we can then look at the next tweet in the series, and obtain its sentiment score. This process is continued for N number of tweets, where N’s optimal value as yet to be determined. As it turns out, there a large number of tweets on the internet (6,000 new tweets created every second) so finding a large amount of tweets, while also taking processing time into consideration is a non-trivial task. We ended up sticking with a value 1000 for this N.

One of the larger problems associated with the twitter sentiment is a restriction put upon on accessers of the official Twitter API. Namely, this restriction is a max seven day limit history of tweets. This means that using the official API would make it impossible to go back in the past to access tweets, making data extraction for our model nearly impossible. Luckily for us, we were able to find a open source repository that can access tweets older than seven days. They are not using the official Twitter API, and instead access the data through a series of JSON requests to Twitter’s site. This means that access times are noticeably more time consuming than using the official API, and our approach is also susceptible to not working in the future if Twitter changes their code base in a major way.

## emotion_tensorflow
Emotion recognition is another component of the stock prediction model, for this process a convolutional neural network (CNN)  powered by tensorflow is used to apply mathematical models on out network and produce an output. To train a CNN we had to obtain a number of images from each class we wished to classify on, we were able to obtain an open-source dataset of images from the Cohn Kanade, CK and CK+, database. Once we obtained the images, we used them to train the model and train on 5 of the 7 emotions provided by the database. We used the 5 emotions with the largest set of images, which are anger, disgust, neutral, happy, and surprised. We then used 80% of the images for training and the other 20% for validation. After training a model through the steps discussed in the methodology section, the training model has a validation accuracy of 73%. We predict the model to be overfitting due to the small dataset we are training on, even though the model is overfitting it still does a pretty good job at predicting on images outside of the training sample.

## priceAnalysis
Any prediction model needs to be compared to real results to determine its accuracy. When analyzing a stock’s price data, it’s important to be able to extract trend information from a stock’s daily price fluctuations. In order to do this, one needs to compare the daily opening and closing prices. However, the daily opening and closing prices aren’t sufficient enough to make a determination.The highs and lows of each day must also be factored in to account for intraday trends. Only then will a determination on whether a day’s price data indicates an upward or downward trend be accurate.

Price data for each stock is retrieved from Yahoo Finance using the Python “pandas” library. The data is retrieved for a timeframe indicated by a start and end date. The days’ price information is read into a “pandas” data panel where individual features like the opening, closing, high, and low prices for each day can then be isolated. For a given time period, an assessment is made on whether the day’s data is indicative of a positive trend (bullish), or a negative trend (bearish).  

Analysis of price data begins by calculating the delta for the day. The delta is the change in price as a percentage of the stock’s overall value. Then, a decision tree, shown in figure 1.a, is used to classify the daily data as bullish or bearish. If the delta is greater than a threshold, the program classifies the day as bullish or bearish according to the delta. The confidence for each classification is made based on the delta value after the retracement or rally values are considered. For a bearish classification, the rally is calculated as the amount a stock recovers after hitting its low for the day. For a bullish classification, the retracement is factored in as the change in a stock after hitting its daily high. These values are weighted against and combined with the delta to produce the confidence.

For data where the delta is low, classification is done differently. The trade day fluctuation is the main feature used to classify the day’s data. A fluctuation that is greater than a certain threshold is used to determine whether a reversal occurred. A bullish or bearish reversal occurs when a stock’s price changes a great deal before it hits a high or low, respectively, only for it to return to a value close to its open price. The delta is weighed against and combined with the reversal change to make a classification for each of the patterns. When these patterns aren’t present, the delta is used as the sole factor in classifying the data.

![DT](https://i.imgur.com/3moJOyG.png)

A design challenge with building the decision tree was how much to weigh each feature and how to determine the threshold at each juncture of the tree. The final thresholds were determined after testing different values against actual price data. Research into what constitutes a positive or negative trend to traders was also needed to help hone in on the appropriate values.

When doing the price analysis, the biggest problem encountered was fetching the price data for the stocks. There are a few APIs available, such as Google Finance, Quandl, and Yahoo Finance. However, many of these APIs were either deprecated or nonfunctional when the project was being developed. The pandas library proved to be a dependable method for getting the price data but this method doesn’t work every time. Sometimes a response isn’t returned and another query needs to be made to get the necessary results.

## Putting it all together
The emotion prediction model is called by the worker app, once it has downloaded a video and extracted the frames from the video. It will call the predict function which takes in 2 parameters, the stock name it wishes to predict on, as well as the option to extract faces from the video or not, by default it is of value 0 which means that faces will not be extracted from the images in the dataset provided. The CNN will predict on the raw images located at the company path supplied to the function parameter. To predict correctly, the model rebuilds the weights of each neuron and connection from a graph that was created during the training process. Our project is able to analyze many images from a stock symbol and produce a confidence value of the emotion being more negative or more positive, hence a value between [-1, 1]. To achieve this the application obtains the emotion of every image in the directory being classified and after obtaining a result for each image it places the image into a counter bucket, which keeps track of how many times each emotion was predicted for every frame of the video. The final output of the prediction model is a normalized value between [-1, 1], this is achieved by applying weights to each emotion and then adding or subtracting a bias for each object read in, the bias increases or decreases the value based on the overall predicted emotion of the images.

Finally, we needed to merge all three components together to streamline the entire process. To do this we have a single python script called worker.py that takes in a single parameter, a video file (which presumably contains contents of a CEO). The file is assumed to be in the form SYM_YYYY-MM-DD.mp4. We first parse through this video file, and split it into frames and place the frames into directory. We then import all other previously mentioned components into this script, which will call the necessary functions from each one. Specifically, we obtain the sentiment score for SYM from the period from YYYY-MM-DD to YYYY-MM-DD + 7. We then get the emotional score for the video that was submitted, and then attempt to classify the submission using a separate module. We then get the actual bullishness, and compare that value with the classification. Once we have all the values from the different classification algorithms implemented, we are able to use a K-nearest neighbors classifier to predict which class the new value belong to. To classify using KNN, we graph sentiment and emotion on the x and y coordinates of a 2-D graph, respectively. We then use a KNN with k=3 (because out dataset is small), which always gives us a prediction with one of the following probabilities (1, 0.6, 0.3, 0). Therefore, we use that value as the confidence that a stock will perform good or bad in the future.

# Results
Our results were positive given our smaller dataset. After training on 18 videos, we then tested five videos and got the correct prediction for all cases. Unfortunately, due to the nature of data, most of the objects are primarily classified to be a “not buy”. This could be the nature in which we have defined our data. Meaning, since our data is primarily based on the videos (the date of the video is then used to analyze stock price changes and sentiment scores), there can be an argument made that the only time a CEO or CFO would go on air would be to clear the air about some issues in the company. Effectively trying to help out their stock’s chance at survival.

As you can see in Fig2 below, a good percentage of our data points were deemed to be “bad buys”. Based on this graphical representation of the data, we were able to determine a value for k that would have a meaningful impact. For a k larger than 3, we would never be able to get a “good buy” decision as no 4 “good buy” points are close enough to win a majority vote against the “bad buys”’.

![Fig 2](https://i.imgur.com/lfwmS7r.png)

# Future work / Conclusion
Future work would revolve around a number of aspects that would improve the model. One of the key issues we have, is that the twitter sentiment score and the emotional score carry the same weight, independent from the bullish, or bearishness determined for that week. We would like to incorporate the following: Determine the significance of the scores by observing the bullishness, and comparing that to the two dimensions. Meaning, it is already clear that the two attributes don’t have the same amount of weight through intuition alone, so applying a educated weight to each dimension would significantly increase the capabilities of our model. If we were to find, for example, that even the twitter sentiment was negative, and the emotional score was positive, but the stock was bullish over this period, then emotional score would have a higher weight. Perhaps each model performs better under varying temporal fields or bullishness. In this case, a different weight would be applied in each case.

One of the most challenging parts in a time restricted projected for us was the collection of the data. Twitter sentiment and stock evaluations were quite simple to retrieve as their data sources are well documented and easy to obtain (though we did encounter a few challenges here). However, obtaining the video data was a very time-consuming task as it involved looking up video interviews on youtube with members of the company, finding relevant information in the video (close-ups of said member), noting the date, and downloading the portion of the video that was relevant to us. From here we would need to split the video into its frames, and then analyze these frames and score the video based on this subset frames. Being able to automate even a portion of this process would be hugely beneficial, and finding a database of videos with these CEO’s and CFO’s would be even more so.

Another direction we would like to go in is obtaining the context of the video interviews in an empirical way. As many of these interviews are hours long, they don’t necessarily talk about the company the entire time. At which point it becomes clear that if they are talking about their struggling childhood and show an emotion of “sad”, this shouldn’t have any effect on our predictions of the company’s value. However, due to time constraints, we were unable to obtain the context of many of the portions of video used by our model.

We are also hindered in our emotional data recognition. Right now, the model we have trained for this part is using a very small subset of faces. We are currently awaiting approval for a much larger database of faces, which would enable our emotional recognition model to be far better at predicting the correct emotion. This type of improvement would minimize errors, and provide us with a more accurate emotional score.
