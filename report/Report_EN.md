## Feature engineering using RDDs

The first practical exercise represents each observed camera by its weekly temporal distribution. Raw traffic strings are loaded into an RDD; DeviceCode is extracted from index 0 and PassDateTime from index 4. datetime parses YYYY-MM-DD HH:MM:SS and supplies weekday/hour. With Monday numbered 1, the feature index is (DOW-1)*24+hour. Each observation maps to ((camera,index),1); reduceByKey counts observations for each slot. The grouped slots then populate a zero-initialized vector of length 168. Missing time slots remain zero. The vector describes accumulated observations over the dataset, not necessarily one particular calendar week. The source text's reversed 861/42/32 numerals correspond to 168 dimensions, 24 hours, and an hour range of 0 to 23. 

## L1 normalization and interpretation

mapValues divides each nonzero vector by its total. The components then sum to one. This compares temporal shape rather than absolute traffic volume: a camera observing 100 passages and another observing 10000 passages can have similar normalized profiles if their relative hourly distribution is similar. Keep the raw volume as a separate descriptive feature. A zero-total vector needs an explicit guard rather than division by zero. Low-count cameras also have noisier normalized profiles; equal normalization does not remove uncertainty from sparse sampling.

## Centering, SVD, and PCA selection

Statistics.colStats computes the per-feature mean. Broadcasting the mean permits every camera vector to be centered consistently. SVD of the centered matrix yields singular values and right singular vectors. PCA covariance eigenvalues are s_i^2/(n-1), for n cameras and sample covariance; the denominator cancels in explained-variance ratios. Singular-value squares are proportional to the sample-covariance eigenvalues, with a common factor of 1/(n-1). Cumulative explained variance is calculated from squared singular values, and the first component count reaching 0.90 is selected. The right singular vectors supply the projection matrix; centered vectors are multiplied by the first k_pca columns. The submission prints the selected dimension and displays the variance plot. The narrative does not contain a reliably extractable numeric k_pca, so this edition retains the exact output image instead of inserting a guessed value. PCA preserves a stated fraction of variance, not every kind of task-relevant information.

## Initialization comparison and silhouette

K is varied from 3 to 12 for random and the requested spread-based initialization. The silhouette for point i is s(i)=(b(i)-a(i))/max(a(i),b(i)), where a is average within-cluster distance and b is the minimum mean distance to another cluster. Scores near one indicate good cohesion/separation under that distance. Singleton conventions and distance choices matter. The report describes evaluating a random sample capped at 10000 to avoid excessive memory use. Sampling can make evaluation practical, but the central limit theorem does not guarantee that rare clusters are represented or that the chosen K is correct. Report sample size, seed, and cluster coverage; repeated samples are useful for sensitivity analysis. Spread-based initialization can improve the chance of a favorable start, but cannot guarantee faster convergence, repeatability without a fixed seed, or a global optimum. Spark's k-means|| is a scalable initializer and should be distinguished from exact sequential K-Means++ when that mode is used. The source contains the full score/output tables for both approaches. 

## Four visualization families and interpretation

Cluster-size bar charts reveal how many cameras receive each label. Unequal sizes may reflect actual class imbalance; they are not by themselves proof of a poor clustering. Mean 24-hour line profiles are obtained by aggregating the 168 slots across weekdays, exposing possible morning/evening peaks or flatter profiles. A PC1/PC2 scatter plot is a visual projection of the fitted data; apparent overlap in two dimensions does not automatically invalidate separation in the full fitted PCA space. For each cluster, the closest camera to its centroid is selected as a representative. Comparing that camera's hourly profile with the mean makes the average more interpretable. A good match is useful descriptive evidence, but does not prove that every member has the same behavior or that the mean is a unique physical traffic archetype. The experiment report favors the spread-based initialization based on its plots and scores; that conclusion remains restricted to this submitted run.

## Enriched camera statistics and exports

Raw records supply TotalPasses through counting by camera, UniquePlates through distinct camera/plate pairs, and PeakHour through hourly count comparisons. Cluster labels are joined to these fields. Each camera's 24-hour distribution is normalized and the cluster profile averages those distributions, avoiding unintended weighting by high-volume cameras. The report exports an Analysis_Results.xlsx workbook with four camera/profile sheets corresponding to the two initializations. Available CSV inputs are in the code directory under their required relative paths. The workbook/screenshots are submitted outputs, not newly regenerated results. The pipeline demonstrates low-level Spark feature construction, normalization, PCA, clustering, and interpretable export; it does not establish a causal classification of neighborhoods or independently validated road semantics.

## BFR summaries and Mahalanobis distance

The discard set contains points assigned confidently to a main cluster; keep N, SUM, and SUMSQ rather than each point. The compression set summarizes groups close to one another but not yet confidently assigned to a main cluster. The retained set stores unresolved isolated points explicitly. DS/CS reduce RAM, while RS may grow if the clustering assumptions poorly fit the data. BFR's diagonal model uses axis-aligned Gaussian clusters, not arbitrary rotated covariance ellipses. The distance to centroid c is the square root of the sum over dimensions of ((x_i-c_i)/sigma_i)^2. Scaling by standard deviations accounts for unequal spread along different axes. Zero or very small variance requires a defined numerical policy. Euclidean K-Means tends to favor comparable spherical distances; Mahalanobis scaling makes an axis with a large natural spread contribute less for the same absolute offset.

## Frobenius/SVD proof

For a reduced SVD A=U Sigma V-transpose with orthonormal columns, ||A||_F^2=trace(A-transpose A). Substitute the decomposition to obtain trace(V Sigma-transpose U-transpose U Sigma V-transpose). Since U-transpose U=I, cyclic trace rearrangement gives trace(Sigma-transpose Sigma V-transpose V)=trace(Sigma-transpose Sigma)=sum_i sigma_i^2. This argument works for rectangular matrices; Sigma need not be treated as a square matrix unless using the compact square diagonal factor.

## Centroid-linkage agglomeration

Start with six singleton points 2,5,9,15,16,18. Merge D/E at distance 1, producing centroid 15.5. Merge DE/F at distance 2.5, producing centroid 49/3. Merge A/B at distance 3, producing centroid 3.5. Merge AB/C at distance 5.5, producing centroid 16/3. The two final clusters are {A,B,C} and {D,E,F}. The merge table defines the requested dendrogram:

| Step | Merge | Height | Resulting centroid |
|---|---|---:|---:|
| 1 | D with E | 1 | 15.5 |
| 2 | DE with F | 2.5 | 16.333333 |
| 3 | A with B | 3 | 3.5 |
| 4 | AB with C | 5.5 | 5.333333 |

## Rank-two energy

For singular values 20,12,8,1.5,0.5, total squared energy is 400+144+64+2.25+0.25=610.5. The leading two retain 544, or 544/610.5*100=89.1072891 percent. The residual squared Frobenius error of the truncated rank-two SVD is 66.5. Energy fraction describes squared matrix norm; it should not be mistaken for an arbitrary task accuracy percentage.

## Item-item similarity and missing rating

User means over observed scores are A=10/3, B=14/3, C=3. Centered rows are A:(2/3,unknown,5/3,-7/3), B:(1/3,1/3,-2/3,unknown), C:(unknown,1,unknown,-1). Under the assignment's user-centered cosine convention on co-rated users, item 2 has similarity +1 with item 1, -1 with item 3, and -1 with item 4. Each pair has only one common user. Ordinary Pearson correlation on a single paired observation is undefined; the calculation here is adjusted cosine under the prescribed centering, not a statistically reliable Pearson estimate. The weighted deviation estimate is mean_A + [1*(2/3)-1*(5/3)-1*(-7/3)]/(1+1+1)=10/3+4/9=34/9=3.7777778. The prediction is approximately 3.77; using exact fractions avoids accumulated rounding. Such single-overlap similarities would need significance weighting or a minimum co-rating count in a real system.

## Cold start

An unseen user's preferences and an unseen item's appeal cannot be learned from an empty interaction history alone. Similarity denominators may be zero or latent factors absent. Popularity recommendations provide a general fallback. Content metadata, onboarding preferences, and hybrid models provide alternative information. Their effectiveness requires evaluation; they do not guarantee personalized accuracy for every new entity.
