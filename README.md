# **Geospatial Data Visualisation**

*Over the summer months, I had a part-time job as a Courier at **UberEats**. The idea to do this came suddenly, even though I’ve always loved biking and being outside. The weather was mostly sunny (or at least dry), and the bike lanes are well developed – I am in the Netherlands, after all.*

*During delivering, I had plenty time to think. While paying attention to the traffic and the work, of course. I was thinking about the system behind delivery application. How often or when exactly do I get a new order? In what cases do I get double or sometimes triple orders? In short, **how does their algorithm work**? I wanted to understand it and make observations. How does the time of the day, the day of the week, the current weather or local events influence the number of orders to deliver? Are these related to the distance between the restaurant and the customer? Or take the bonus system, for another example. In a given time interval of a day, and after a certain number of orders delivered, one gets some bonuses. My first thought was, **how can I optimise this system**? When should I take my shifts to maximise the expected earnings? I think that these show very well how my brain is wired – analytical and curious, constantly thinking of real-world algorithms and strategies.*

*As a final step for my summer Courier career, I wanted to visualise my performance. I could have plotted the number of daily completed orders, but I wanted more than a bar chart. Or I could have presented the customer reviews, but for that, there was not enough data. Fortunately, I measured every delivery shift of mine, as a workout (indeed they were a workout) with geolocational data. Now, **I have quite a lot of tracks to visualise**.*

## **Extracting the Data**

Firstly, we need to extract the relevant geospatial data. I personally measured my routes using my Apple Watch, which provides GPS data in a format that we can work with. The data includes information such as the route taken, distance, and elevation changes. To extract the data from Apple's Health app, open the app and navigate to your picture or initials at the top right corner. Tap Export All Health Data, then choose a method for sharing your data. Source: [Share your health and fitness data in XML format](https://support.apple.com/en-gb/guide/iphone/iph5ede58c3d/ios). After some preparation, which can take a while depending on the amount of data, you should have a .zip file containing your health data in XML format. The folder is named "workout-routes". This folder contains the .gpx files that are basically XML files with GPS data.

One data file looks like this:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<gpx version="1.1" creator="Apple Health Export" xmlns="http://www.topografix.com/GPX/1/1" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://www.topografix.com/GPX/1/1 http://www.topografix.com/GPX/1/1/gpx.xsd">
  <metadata>
    <time>2025-08-25T14:57:33Z</time>
  </metadata>
  <trk>
    <name>Route 2025-05-05 5:24pm</name>
    <trkseg>
      <trkpt lon="4.637620" lat="52.382577"><ele>0.357328</ele><time>2025-05-05T12:32:58Z</time><extensions><speed>0.012652</speed><course>39.776441</course><hAcc>5.546754</hAcc><vAcc>1.326746</vAcc></extensions></trkpt>
      <trkpt lon="4.637620" lat="52.382577"><ele>0.348379</ele><time>2025-05-05T12:32:59Z</time><extensions><speed>0.012272</speed><course>39.779809</course><hAcc>4.936129</hAcc><vAcc>1.181078</vAcc></extensions></trkpt>
      <trkpt lon="4.637620" lat="52.382577"><ele>0.339116</ele><time>2025-05-05T12:33:00Z</time><extensions><speed>0.011041</speed><course>39.953316</course><hAcc>4.565568</hAcc><vAcc>1.061498</vAcc></extensions></trkpt>

```

## **Quick visualisation and selection of routes**

The second step can be omitted if you already have the specific routes ready. As for me, this is not the case, because I only want to visualise the routes I took during my shift. Therefore, I will create a quick visualisation of all the routes and then select the ones I need. Most of them are in Haarlem, in the Netherlands, but I also recorded some workouts elsewhere, and I want to filter out those.

Here are some example. The first visualisation shows an average route taken in Haarlem, the second one shows a specific route I took during a weekend for leisure, and the third one shows a route taken around the Tower of London.

<p float="center">
  <img src="https://github.com/user-attachments/assets/8cbf8d34-c00d-4583-902c-1413e95381e3" width="250" />
  <img src="https://github.com/user-attachments/assets/2991b987-6f14-4d4c-914e-059242f2c620" width="250" />
  <img src="https://github.com/user-attachments/assets/3e568949-b7d0-4065-b195-0bb92ee94551" width="250" />
</p>

## **Reading and Transforming Data**

Now, that I have only the relevant routes selected in my folder, I need to read and transform the data into a suitable format for visualisation. 

After reading every .gpx file, only the relevant columns are kept in the DataFrame. These are the *time*, *latitude*, and *longitude* columns. This results in 68 DataFrames being created, meaning that I did 68 workouts. These DataFrames are then concatenated into a single DataFrame called *coordinates*.

coordinates has 717265 entries, i.e. 717265 GPS data points of latitude and longitude. This is a substantial amount of data, which using a lot of memory and therefore making it a bit slower to process. Fortunately, the data is well-structured and does not have any missing values.

```python
coordinates.info()
```
```
RangeIndex: 717265 entries, 0 to 717264
Data columns (total 4 columns):
 #   Column      Non-Null Count   Dtype  
---  ------      --------------   -----  
 0   Unnamed: 0  717265 non-null  int64  
 1   time        717265 non-null  object 
 2   latitude    717265 non-null  float64
 3   longitude   717265 non-null  float64
dtypes: float64(2), int64(1), object(1)
memory usage: 21.9+ MB

```

Right now, this is how the data looks:

```python
coordinates.head(10)
```
```
   time                       latitude    longitude
0  2025-08-01 15:26:25+00:00  52.392703   4.646152
1  2025-08-01 15:26:27+00:00  52.392721   4.646159
2  2025-08-01 15:26:28+00:00  52.392737   4.646165
3  2025-08-01 15:26:30+00:00  52.392758   4.646168
4  2025-08-01 15:26:31+00:00  52.392766   4.646169
5  2025-08-01 15:26:32+00:00  52.392772   4.646168
6  2025-08-01 15:26:33+00:00  52.392775   4.646167
7  2025-08-01 15:26:34+00:00  52.392778   4.646166
8  2025-08-01 15:26:35+00:00  52.392781   4.646165
9  2025-08-01 15:26:36+00:00  52.392783   4.646163
```

As the next step, I will remove the time column. For the visualization, I only need the latitude and longitude columns. If you want to make an animated plot, you should keep the time column as well, but I have something else in mind. I do not want to animate the plot over time, but rather over a specific route from a specific starting point.

As for handling the large number of data points, I will use several techniques to reduce it and aggregate it. First, I will round the latitude and longitude values to a certain number of decimal places. Then, I create a column *occurrence* that counts the number of times each latitude and longitude pair appears in the DataFrame. All coordinates are decimal numbers with 6 decimal places. Counting the occurrences of each pair without rounding would result in an identical plot, as the coordinates would be the same. Rounding to 5 decimal places means that the coordinates are accurate to about 1.1 meters, which would be sufficient for my purposes. Rounding to 4 decimal places would result in an accuracy of about 11 meters, which is too coarse, and I would like to see smooth lines. And rounding to 3 decimal places would result in an accuracy of about 110 meters, which is definitely too scattered. Rounding to 5 decimal places would be a good compromise between accuracy and data reduction, but there are too many unique pairs of coordinates, resulting in almost 400,000 entries. This is still quite large and would make the visualization slow. That is why I scaled the numbers so the 5th decimal became an integer, rounded that integer to the nearest multiple of 5, and then scaled back down. This way, the 5th decimal digit always ends up as 0 or 5.

<p float="center">
  <img src="https://github.com/user-attachments/assets/1302a80a-c7f5-4c8f-9943-d8e533504b17" width="250" />
  <img src="https://github.com/user-attachments/assets/601a73cd-dd33-4150-bdef-cf2b006c3efb" width="250" />
  <img src="https://github.com/user-attachments/assets/c686c3e9-79b6-4eb2-883b-35a7246a536b" width="250" />
</p>

After rounding and aggregating, this is how the data looks:

```python
coordinates.sort_values(by="occurrence", ascending=False).head(10)
```
```
   latitude    longitude       occurrence
0  52.37937    4.63258         704
1  52.37931    4.63339         538
2  52.37930    4.63339         401
3  52.37930    4.63338         400
4  52.37930    4.63340         348
5  52.37932    4.63339         338
6  52.36485    4.64489         337
7  52.38363    4.64178         335
8  52.38677    4.63664         329
9  52.37931    4.63340         324
```

I would like the routes to be a distinct colour. First, I thought of using the official UberEats green colour. However, I wanted to show a gradient of colours, from light green to dark green, to represent the frequency of visits to each location. After some experimentation, I found that it would not be possible, becuase my data is heavily skewed. There are a few locations that I visited very frequently, while most locations were visited only a few times. Therefore, I decided to start the gradient from my starting location, which is the location with the highest occurrence, in the heart of Haarlem (which is a McDonald's). I tried several colour maps consisting of different shades of green, then I settled on the one I liked the most.

I used the Euclidean distance formula $(1)$ for calculating the distance from the starting point for each coordinate pair. This distance is in the column *eucledian_distance*, which is then normalised to a range between 0 and 1, in the column *normalised_distance*. This is then used to map the distances to colours in the chosen colour map. The resulting colours are stored in a new column called *gradient_colour*.

$\text{eucledian\_distance} = \sqrt{(\text{latitude} - \text{starting\_latitude})^2 + (\text{longitude} - \text{starting\_longitude})^2}$

<div align="center">
  <img src="https://github.com/user-attachments/assets/31716618-8e38-45a9-81e0-11c93512d44a" width="250" />
  <img src="https://github.com/user-attachments/assets/3549d1ba-efed-46cd-bf7e-e3baf0100b33" width="250" />
  <img src="https://github.com/user-attachments/assets/6431bb78-064d-4925-a809-a36bb32e235d" width="250" />
</div>

