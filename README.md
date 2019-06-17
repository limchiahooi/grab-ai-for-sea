# Grab AI for S.E.A.

<br>

**Note to Moderator:** GitHub does not allow pushing of file over 100MB, so the raw data has to be removed from the repo. Kindly download the raw data from https://s3-ap-southeast-1.amazonaws.com/grab-aiforsea-dataset/traffic-management.zip to proceed with my submission. Thank you.
---

<br>

This repo is the submission for Grab AI for S.E.A. - Traffic Management challenge. 

1. [Problem Statement](#ps)
2. [Solution Approach](#sa)
3. [Hypotheses](#h)
4. [Features](#f)
5. [Target](#t)
6. [Model Algorithm](#ma)
7. [Train-Test-Split](#tts)
8. [Model Evaluation](#me)
9. [For Moderator](#fm)
10. [Credits](#c)

---
## <a name="ps">Problem Statement</a>
### Traffic Management
Economies in Southeast Asia are turning to AI to solve traffic congestion, which hinders mobility and economic growth. The first step in the push towards alleviating traffic congestion is to understand travel demand and travel patterns within the city.

Can we accurately forecast travel demand based on historical Grab bookings to predict areas and times with high travel demand?


## <a name="sa">Solution Approach</a>
Participants are given 2 months (61 days) of data in 15-minute intervals (buckets) and are required to forecast ahead for T+1 to T+5 given all data up to time T. There are 1,329 geographic location (in geohash6) to forecast. This is a multi-step time seris forecasting problem. A common approach is to train a model for each location. There are "missing data points" or gaps in the time series and we are required to treat them as zero aggregated demand. While it is possible to create a model for each location using ARIMA, it will consume a lot of time and resources to train over one thousand models for this challenge. Therefore, my approach is to train a single model to predict for multiple locations at the same time. This way, the model will have access to more data and able to learn the subtle patterns that repeat across locations. In effect, I am approaching the problem as a multivariate multi-step time series forecasting framed as a traditional supervised ML problem so that I can use traditional ML algorithms to solve it. 

## <a name="h">Hypotheses</a>
By creating a single model for all locations, I am hypothesizing that there are certain features that will influence the travel demand across locations. For example, the time of the day (demand will rise during rush hours), and the clusters based on proximity (locations with close proximity to each other will have similar demand).

## <a name="f">Features</a>
The "day" and "timestamp" given can be used to create "day", "hour", "minute" features. Clustering using k-means on the latitude and longitude decoded from geohash6 can be used to create clusters of location with close proximity. Lastly, as time-series problem, the lagged time series will be created as features for the model to learn about past information by bringing them forward to the present.

## <a name="t">Target</a>
However, it should be pointed out that traditional ML models can only predict one value but in this challenge we are required to forecast 5 values. To do so, we will have to use 5 data points from the past. For example, we can train a model to predict 5 steps in the future. This means that given a time series of data up to T,  the data point in T can be used to forecast T+5. In order to forecast T+1, T+2, T+3 and T+4 we will use the data points on T-4, T-3, T-2 and T-1. In effect, we are using T-4, T-3, T-2, T-1 and T to forecast ahead for T+1, T+2, T+3, T+4 and T+5, respectively.

### <a name="ma">Model Algorithm</a>
The model will be trained using [LightGBM](https://lightgbm.readthedocs.io/en/latest/). It works well on large dataset and is faster to train than XGBoost. It can work with integer-encoded categorical features without the need for one-hot encoding. It is also able to generate prediction intervals for the forecast (by means of quantile regression). Ideally, we can also train more  models using XGBoost and RandomForest and then average the results (ensemble modeling), but maybe for next iteration.


## <a name="tts">Train-Test-Split</a>
There are 61 days or 5856 buckets (after resampled), I will split 47 days (4512 buckets) as Training set and 14 days (1344 buckets) as Validation set, or roughly 77:23 ratio. Ideally, we should have a backtesting strategy using sliding window approach and the expanding window approach to test the model as suggested by [Uber](https://eng.uber.com/forecasting-introduction/), but maybe in the next iteration. 

## <a name="me">Model Evaluation</a>
As mentioned in the challenge, submissions will be evaluated by RMSE (root mean squared error) averaged over all geohash6, 15-minute-bucket pairs. I will use both MAE and RMSE to evaluate the model on the validation set.


## <a name="fm">For Moderator</a>
- I have tried my best to write comment for every cell to explain what each cell does, my apology for frequent "# take a look" comments
- Cells which require longer time to run (on my laptop) will have timer (%%time)
- I am not sure about the hold-out test dataset details, my apology for not providing step-by-step guide for making prediction on the test set, but in general:
 - pre-process the test data into time series
 - create features from datetime ("day", "dow", "hour", "minute")
 - create features for lagged series (5 periods)
 - use the same geo_cluster labels since the set of geohashes are the same in training dataset and test dataset
 - make sure the shape of test set is (x, 12)
 - load the pre-trained model ("lgb.pkl")
 - make predictions
 - evaluate results

## <a name="c">Credits</a>
- Since the challenge was announced, there has been several starter-kits made available online to help participants to overcome the cold-start problem. Particularly, I have benefited from the How-to guide from [Husein Zolkepli](https://github.com/huseinzol05/Machine-Learning-Data-Science-Reuse/tree/master/grab-aiforsea/traffic-management) and the Kaggle kernel of [Mahadir Ahmad](https://www.kaggle.com/mahadir/grab-traffic-demand-forecasting) especially on decoding geohash6 that I would like to record my appreciation here. Thank you.
---
