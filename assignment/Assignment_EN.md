# Temporal camera clustering and book recommendation: assignment requirements

## Practical exercise 1: temporal camera clustering

Dr Iman Gholampour's third Big Data Analytics assignment uses traffic.csv. The practical designers are Mehdi Noureldini and Iman Ahmadi; the theory designer is Seyed Hassan Sadeghzadeh. For the camera exercise use only RDD transformations/actions, without DataFrames or high-level equivalents. Count total observations and parse PassDateTime into weekday and hour. Construct a 168-dimensional vector for every DeviceCode, with seven days times 24 hours and index=(DOW-1)*24+hour. DOW is 1 through 7 and hour 0 through 23. The extracted source has truncated/reversed numerals in places; the indexing formula unambiguously defines 168 dimensions. Normalize each nonzero camera vector by its L1 sum and explain why. Perform PCA on the normalized vectors. Choose the smallest k_pca whose cumulative explained variance reaches at least 90 percent, report k_pca and the cumulative value, and plot the variance curve. Cluster in PCA space using random K-Means initialization and the requested K-Means++-style initialization. Compare K from 3 through 12, write the silhouette formula, tabulate scores, choose K, and discuss initialization stability. Identify the actual Spark initialization mode when implementing a scalable variant rather than labeling it exact sequential K-Means++. Produce four figures for each method: cluster sizes; mean 24-hour profiles; PC1/PC2 scatter colored by cluster; and a representative camera nearest the centroid superimposed on its cluster's mean hourly profile. Compare the methods based on those outputs.

## Camera export requirements

Produce camera-cluster CSVs with DeviceCode, ClusterID, TotalPasses, UniquePlates, and PeakHour. Produce cluster-hour-profile CSVs with ClusterID, Hour, and AvgPHour, the average normalized hourly share across cameras in the cluster. Export a workbook with sheets KM_CameraClusters, KM_HourProfile, KMpp_CameraClusters, and KMpp_HourProfile. Deliver the code, report, CSVs, and workbook. Preserve each camera's normalized distribution when averaging so that highly trafficked cameras do not unintentionally dominate.

## Practical exercise 2: book recommendation

Load Books.csv, Users.csv, and Ratings.csv into PySpark. Display each schema and five sample rows. Clean the rating data and inappropriate/missing book fields as justified. Left-join ratings to users on User-ID and then to books on the common ISBN field. The source's reference to goods is treated as the Books dataset. Produce five titled, labeled, interpreted plots: rating histogram; ten most-rated books; ten most popular authors; age distribution of active users; and age versus mean rating. Choose and explain ALS and neural-based recommendation methods as requested. Convert the model input to (user id,isbn,rating), split training/test data, train the selected models, and analyze rank, maxIter, and regParam where applicable. Recommend five products to at least five existing customers, and provide popular-product recommendations for a new customer. Evaluate and discuss results on the supplied book rating.json evaluation data. Introduce/compare suitable metrics, explain recommender challenges and remedies, and give at least two realistic improvement proposals.

## Theory requirements

1. Define BFR's discard, compression, and retained sets and their RAM implications. Give its diagonal Mahalanobis-distance formula and explain axis-aligned elliptical clusters.
2. Prove that the squared Frobenius norm equals the sum of squared singular values, using trace and A=U Sigma V-transpose.
3. For A=2,B=5,C=9,D=15,E=16,F=18, perform centroid-linkage agglomeration until two clusters remain and draw the dendrogram.
4. For singular values 20,12,8,1.5,0.5, compute total energy and the percentage preserved by rank two.
5. Use the utility matrix below for item-item similarity after subtracting each user's mean, then estimate A's missing rating for item 2. Explain the specified similarity convention because one shared user is insufficient for ordinary statistical Pearson correlation.

| User | Item 1 | Item 2 | Item 3 | Item 4 |
|---|---:|---:|---:|---:|
| A | 4 | unknown | 5 | 1 |
| B | 5 | 5 | 4 | unknown |
| C | unknown | 4 | unknown | 2 |

6. Explain cold start for a new user and a new item, why interaction-only collaborative filtering has insufficient information, and remedies. Theory answers use the requested StudentID_LastName_HW3.pdf naming convention.
