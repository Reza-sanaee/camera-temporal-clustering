# Weekly Camera-Traffic Profiles with RDD PCA and K-Means

RDD processing constructs 168-dimensional weekly camera profiles, normalizes them by traffic volume, reduces dimensions with SVD/PCA, and compares K-means initialization strategies. The notebook selects 47 components to retain 90.24 percent of variance and exports camera/cluster profiles for inspection.

## Technical highlights

- Built 168-dimensional weekly camera-traffic profiles using Spark RDDs and L1 normalization.
- Applied SVD/PCA and compared K-means initialization strategies; exported camera statistics and cluster-hour profiles.

## Explore the project

- [Source and execution guide](code/RUNNING.md)
- [Technical report (PDF)](report/Report_EN.pdf)
- [Technical report (Markdown)](report/Report_EN.md)
- [Assignment/specification (PDF)](assignment/Assignment_EN.pdf)

## Reproduction and attribution

Read the execution guide and report before running the code. Dependencies, input data, licensed tools, and hardware requirements vary by project. Recorded results are distinguished from rerun results in the accompanying documentation. Team and third-party source attribution in the reports and source headers is retained. This portfolio does not claim independent authorship of supplied frameworks or dependencies.
