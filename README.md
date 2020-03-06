# FFOOD
**Framework for Feature and Observation Outlier Detection (FFOOD)**

## Note this package doesn't work yet. 

Feature Outlier Detection and Explanation using Gradient Boosting Prediction Residuals

The package is nice and simple and boils down to one command.

```python
from ffood import tables as tb

outliers, features = tb(clean_data)
```

#### Description

```
pip install ffood
```

FFOOD is a unique method to audit potential model inputs. It is designed to help you identify whether you might need additional variables and whether you have mistakes or outliers in your datasets. It is a unique outlier model because it investigates the prediction outlier for every individual feature leading to extremely robust inputs. Note, it is a very slow algorithm, with the number of models being trained equaling 30 times the Number of Features. 

FFOOD addresses whether confounders or outliers drive the prediction error. This method is primarily meant for cross-sectional datasets. All functions only work with machine readible data; do no pass it NaN or Infinite values and make sure that all category variables are one-hot encoded.

The method is at first an analysis of the most overpredicted instances (hence most underpredicted too) for each feature. The method starts by dividing the dataset in two random subsets. The model is trained on subset one to predict subset two, after which it is trained on subset two to predict subset one. The feature to be predicted is one plus log transformed to ensure there is no negative predictions.

The percentage difference between the predicted and the actual target is used to determine the most overpredicted and under predicted instances. This process is repeated N number of times different test-train samples to ensure that a stable overpredicted and underpredicted percentage have been obtained.

The next step is to find the features most associated with the overprediction. At this point one knows which instances are predicted outliers and what features are driving this outliers. While paying close attention to both the outlier instances and outlier features the following three questions have to be asked.


- Going back to the unstructured data, is there any additional features (confounders) that you could have missed?
- If no additional attributes or characteristics coul help to explain the overprediction, is the instance a data entry mistake or an outlier?
- Have a look at the unsupervised feature characteristics like predictability, informativeness, underpredictor, overpredictor and, outlier-driver and repeat the first two steps.
- Run the model again and repeat the steps until no prediction outliers are observed; this requires some judgement.

The best way to dealt with this issue is with an example. This example uses an Airbnb dataset. 

# Example

#### Airbnb Daily Fair Valuation


Welcome to Airbnb Analysis Corp.! Your task is to set the competitive ****daily accomodation rate**** for a client's house in Bondi Beach. The owner currently charges $500. We have been tasked to estimate a ****fair value**** that the owner should be charging. The house has the following characteristics and constraints. While developing this model you came to realise that Airbnb can use your model to estimate the fair value of any property on their database, your are effectively creating a recommendation model for all prospective hosts!

1. The owner has been a host since ****August 2010****
1. The location is ****lon:151.274506, lat:33.889087****
1. The current review score rating ****95.0****
1. Number of reviews ****53****
1. Minimum nights ****4****
1. The house can accomodate ****10**** people.
1. The owner currently charges a cleaning fee of ****370****
1. The house has ****3 bathrooms, 5 bedrooms, 7 beds****.
1. The house is available for ****255 of the next 365 days****
1. The client is ****verified****, and they are a ****superhost****.
1. The cancelation policy is ****strict with a 14 days grace period****.
1. The host requires a security deposit of ****$1,500****

```python
    from dateutil import parser
    dict_client = {}
    dict_client["city"] = "Bondi Beach"
    dict_client["longitude"] = 151.274506
    dict_client["latitude"] = -33.889087
    dict_client["review_scores_rating"] = 95
    dict_client["number_of_reviews"] = 53
    dict_client["minimum_nights"] = 4
    dict_client["accommodates"] = 10
    dict_client["bathrooms"] = 3
    dict_client["bedrooms"] = 5
    dict_client["beds"] = 7
    dict_client["security_deposit"] = 1500
    dict_client["cleaning_fee"] = 370
    dict_client["property_type"] = "House"
    dict_client["room_type"] = "Entire home/apt"
    dict_client["availability_365"] = 255
    dict_client["host_identity_verified"] = 1  ## 1 for yes, 0 for no
    dict_client["host_is_superhost"] = 1
    dict_client["cancellation_policy"] = "strict_14_with_grace_period"
    dict_client["host_since"] = parser.parse("01-08-2010")
```
<br />

**Raw Data**
The algorithm applied to this dataset runs about 30 minutes. I will include some efficiency improvements in the future.

```python
raw_data = pd.read_csv("https://github.com/firmai/random-assets/blob/master/listings.csv?raw=true")
```

<br />

**Cleaned Data**

```python
clean_data = your_cleaning_operations(raw_data)
```

<br />

### Start Here

<br />

**FFOOD Tables**

```python
outliers, features = tables(clean_data)
````

<br />

**Outliers**

This operation finds the prediction outlier for all feature. The first is an anlysis of 'price' as the target. 

```python
outliers[outliers["Predicted Feature"]=="price"]
```

| Overprediction Index | Overpredict Percentage | Underprediction Index | Underpredict Percentage | Predicted Feature | Top Feature          | ABS SHAP Value | Larger Feature Leads to Overprediction (FLO) | FLO Value  | Larger Feature Leads to Underprediction (FLU) | FLU Value  |
| -------------------- | ---------------------- | --------------------- | ----------------------- | ----------------- | -------------------- | -------------- | -------------------------------------------- | ---------- | --------------------------------------------- | ---------- |
| 18039                | 900                    | 33057                 | -96                     | price             | Entire home/apt      | 204941.78      | Private room                                 | 5678.24536 | bathrooms                                     | 2219.96877 |
| 32657                | 629                    | 30218                 | -95                     | price             | accommodates         | 184487.462     | security_deposit                             | 903.426214 | Entire home/apt                               | 846.470856 |
| 18416                | 441                    | 24287                 | -94                     | price             | bathrooms            | 164815.563     | longitude                                    | 237.58082  | bathrooms_per_person                          | 503.535783 |
| 32731                | 431                    | 27492                 | -93                     | price             | bedrooms             | 122461.56      | cleaning_fee                                 | 191.342111 | accommodates                                  | 445.722047 |
| 27078                | 351                    | 30957                 | -93                     | price             | bathrooms_per_person | 96394.2702     | beds                                         | 105.192666 | Shared room                                   | 394.079955 |

<br />

The next is the same table but for the average reviewer rating. All feature are are contained within the *outliers* data frame.

```python
outliers[outliers["Predicted Feature"]=="review_scores_rating"]
```

| Overprediction Index | Overpredict Percentage | Underprediction Index | Underpredict Percentage | Predicted Feature    | Top Feature                | ABS SHAP Value | Larger Feature Leads to Overprediction (FLO) | FLO Value  | Larger Feature Leads to Underprediction (FLU) | FLU Value  |
| -------------------- | ---------------------- | --------------------- | ----------------------- | -------------------- | -------------------------- | -------------- | -------------------------------------------- | ---------- | --------------------------------------------- | ---------- |
| 15082                | 393                    | 32944                 | -23                     | review_scores_rating | number_of_reviews          | 17494.1253     | bedrooms                                     | 205.592071 | Private room                                  | 304.302714 |
| 24905                | 384                    | 32943                 | -20                     | review_scores_rating | past_and_future_popularity | 13891.5491     | security_deposit                             | 194.578755 | accommodates                                  | 124.632332 |
| 29090                | 380                    | 31644                 | -20                     | review_scores_rating | latitude                   | 2382.14212     | host_identity_verified                       | 95.738997  | number_of_reviews                             | 68.089677  |
| 5375                 | 377                    | 16751                 | -20                     | review_scores_rating | accommodates               | 2242.57177     | latitude                                     | 62.409664  | longitude                                     | 62.943133  |
| 3210                 | 374                    | 31149                 | -19                     | review_scores_rating | host_is_superhost          | 1933.74017     | host_is_superhost                            | 13.874651  | bathrooms_per_person                          | 39.594302  |



*From here forward, I will focus on the price feature as target.*

<br />

**Raw Data**


| Overprediction Instances                                                       | Archive                                                  | Underprediction Instances                                                      | Archive                                                  |
| ------------------------------------------------------------------------------ | -------------------------------------------------------- | ------------------------------------------------------------------------------ | -------------------------------------------------------- |
| [https://www.airbnb.com/rooms/21743681](https://www.airbnb.com/rooms/21743681) | [http://archive.today/BR1ss](http://archive.today/BR1ss) | [https://www.airbnb.com/rooms/26932284](https://www.airbnb.com/rooms/26932284) | [http://archive.today/i2AjM](http://archive.today/i2AjM) |
| [https://www.airbnb.com/rooms/21884828](https://www.airbnb.com/rooms/21884828) | [http://archive.today/dIyVM](http://archive.today/dIyVM) | [https://www.airbnb.com/rooms/30043604](https://www.airbnb.com/rooms/30043604) | [http://archive.today/ttcqI](http://archive.today/ttcqI) |
| [https://www.airbnb.com/rooms/29807040](https://www.airbnb.com/rooms/29807040) | [http://archive.today/3I9GP](http://archive.today/3I9GP) | [https://www.airbnb.com/rooms/31601306](https://www.airbnb.com/rooms/31601306) | [http://archive.today/uc8m3](http://archive.today/uc8m3) |
| [https://www.airbnb.com/rooms/33861409](https://www.airbnb.com/rooms/33861409) | [http://archive.today/EPLdO](http://archive.today/EPLdO) | [https://www.airbnb.com/rooms/32384612](https://www.airbnb.com/rooms/32384612) | [http://archive.today/EDKtZ](http://archive.today/EDKtZ) |
| [https://www.airbnb.com/rooms/33912597](https://www.airbnb.com/rooms/33912597) | [http://archive.today/IeHQ9](http://archive.today/IeHQ9) | [https://www.airbnb.com/rooms/34231022](https://www.airbnb.com/rooms/34231022) | [http://archive.today/SclC0](http://archive.today/SclC0) |





There is a lot of other raw data that can be found here for the [overpredicted](https://github.com/firmai/FFOOD/blob/master/raw/Over.csv) and [underpredicted](https://github.com/firmai/FFOOD/blob/master/raw/Under.csv) instances. 

</br >

**Features**

The figures to this table rely on the entire data set and are not specific to any one feature.

| predictability Feature     | predictability Value | informativeness Feature    | informativeness Value | overpredictor Feature | overpredictor Value | underpredictor Feature     | underpredictor Value | outlier_driver Feature | outlier_driver Value |
| -------------------------- | -------------------- | -------------------------- | --------------------- | --------------------- | ------------------- | -------------------------- | -------------------- | ---------------------- | -------------------- |
| number_of_reviews          | 236616.671           | availability_365           | 249729.3              | Private room          | 1742.98441          | accommodates               | 3961.16197           | availability_365       | 16876.2              |
| price                      | 154620.127           | past_and_future_popularity | 149925.182            | bedrooms              | 1359.6978           | past_and_future_popularity | 1847.40775           | number_of_reviews      | 1414.2               |
| security_deposit           | 116134.893           | Entire home/apt            | 82007.025             | bathrooms_per_person  | 976.245801          | host_is_superhost          | 922.621391           | minimum_nights         | 842.2                |
| past_and_future_popularity | 43272.0684           | cleaning_fee               | 64619.9799            | bedrooms_per_person   | 500.608543          | bathrooms_per_person       | 904.433052           | price                  | 644.6                |
| cleaning_fee               | 30418.8392           | bathrooms                  | 55519.1308            | Hotel room            | 338.967136          | bathrooms                  | 824.314968           | beds                   | 475.6                |


</br >

## FFOOD Analysis Starts

While performing the analysis remember that we want to establish a fair value for our clients, and for that reason don't want to include data on units or hosts that are not acting within the spirit of Airbnb. 

- Start with the first feature:
- (1) Confounders that you could have missed?
    - (a) Overprediction
    - (b) Underprediction
- (2) Data entry mistakes and outliers?
- (3) Any strange unsupervised feature characteristics?
- (4) Go to the next feature and follow (1) - (3)
- (5) Run the model again and follow (1) - (4) untill all outliers are removed. 


### Price

#### (1) Confounders that you could have missed?

* See the archive links if the original links stop working.

##### (a) Overprediction:

[18039](https://www.airbnb.com/rooms/21743681) ([Archive](http://archive.today/BR1ss))

This instance shows immediate red flags, there is no dates available into the future and the price is really low. It is like offering a product at a very low price but not having stock. These type of locations should not be included in the data set and should be removed. I would remove all homes at a hard limit of say $15 and less with no availbility for the next two months and less than two reviews.

[32657](https://www.airbnb.com/rooms/21884828) ([Archive](http://archive.today/dIyVM))

This host realised her error and she has respectively adjusted her price from [$20](https://github.com/firmai/FFOOD/blob/master/raw/Over.csv) to $50 within two months. But even still, $50 is rediculously low for a prediction of (20 x 600% = $120). She has a cleaning fee of $70, but the model should have picked this up. The cleaning fee is a FLO (an overpredicting feature) so we might want to give it some attention in the future. I would create one additional feature which is the price + cleaning fee, but other than that, this is just a good deal. This could set the baseline for identifying a deal vs an error. Overprrediction of 50/20 ~ we will conservatively say 3, should be removed.

[18416](https://www.airbnb.com/rooms/21743681) ([Archive](http://archive.today/3I9GP))

This again just looks like an amazing deal. Although the reviewer are fairly highly rated at 3.75, there seems to be an issue with cancellations as per the reviews. This might be a contributing factor to him having to offer a low price to attract clients as this is a great cause of uncertainty to holiday plans. To fix the miss prediction issue, I would count the number of reviews with 'cancel' occuring in the text.

[32731](https://www.airbnb.com/rooms/33861409) ([Archive](http://archive.today/EPLdO))

This price changed from $15 to almost $150, as a result, it is likely to be a mistake. Howeverm this change of price is not somethign you would have known at the time of creating the model. The 3 times overvaluation rule established in __32657__ would take care of this mistake of 4.3 times overvaluation. 

[27078](https://www.airbnb.com/rooms/33912597) ([Archive](http://archive.today/IeHQ9))

Again a mistake, was listed as $20, now listed as $200. The 3 times overvaluation rule established in __32657__ would take care of this mistake

##### (a) Underprediction:

[33057](https://www.airbnb.com/rooms/26932284)

The price of this room is extremely high at $937. The price has dropped a bit from the $1,251 it was originally. Prices seem to be quite dynamic first of all. It seems like season has to be taken into account. When presenting our client with a valuation, it would be necessary to have a database of the prices scraped in every month of the year. 

You would expect to see this price luxury hotels. People here seem to be paying for the 'penthouse experience'. It is worth considering that a large amount of availble days and no reviews could be indicative of both bad lodging and a high price. Because hotels are so different to traditional Airbnbs all hotels can be thrown out. If that is too extreme, then one can instead scan the text for 'Penthouse' and add that as a feature.


[30218](https://www.airbnb.com/rooms/26932284)

This entry has dissapeared, clearly a mistake or testament to a too overvalued price leading to no lodgers, we can have a look at the [raw data](https://github.com/firmai/ffood/blob/master/raw/Under.csv) to see why. At some point we therefore have to establish a cut off. 


[30218](https://www.airbnb.com/rooms/26932284)

Also dissapeared.

[30218](https://www.airbnb.com/rooms/26932284)

Although this price is high, it is justified nased on the fact that it is see facing. There is very few homes in this region that are on airbnb so the comparable prices could not be established. A new feature distance to ocean is recommended. This room was extremely overpriced at $1200 for a room in a house. The host has since dropped the price to $78. For these outliers, a threshold at 3 times underprediction will also be established. 



#### Number of Reviews

(1) Confounders that you could have missed?

(a) Overprediction (

[15082__ 
https://www.airbnb.com/rooms/21743681

This instance shows immediate red flags, there is no dates available into the future and the price is really low. It is like offering a product at a very low price but not having stock. These type of locations should not be included in the data set and should be removed. I would remove all homes at a hard limit of say $15 and less with no availbility for the next two months and less than two reviews.














