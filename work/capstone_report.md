# Content Opportunity Scoring for Search Performance

**Author: Shefira Marizcha**  
**Lane:** Refresh / Content Opportunity Scoring  
**Repo: https://github.com/shefiramarizcha62-sudo/flyrank-ml-portfolio.git**  
**Date: 13-October-2026**

## 0. Abstract

This study investigates how observable search and content performance signals can be used to prioritize client-content pairs for human review. The study uses March 2026 search and engagement data from the FlyRank internship warehouse and defines a future opportunity proxy based on April 2026 impression and click-through rate thresholds. A Random Forest classifier using six performance features was evaluated through a client-grouped train-test split and compared with a rule-based baseline on the same test set. The Random Forest achieved a ROC-AUC of 0.921432 and an Average Precision of 0.680600, outperforming the baseline scores of 0.738707 and 0.318966, respectively. The resulting scoring approach is intended to support the prioritization of content and SEO reviews through ranked recommendations, rather than to establish causal improvements in search performance.

## 1. Problem framing

Organic search performance provides measurable signals for understanding how content appears in search results and attracts user clicks. Metrics such as impressions, clicks, click-through rate (CTR), and average search position describe different aspects of search visibility and user interaction. Impressions indicate how frequently content appears in search results, while clicks represent user interactions with those results. CTR expresses the proportion of impressions that result in clicks, and average position provides an indication of a content page's visibility in search results. These metrics can be examined together to identify content that receives search exposure but generates relatively limited click engagement.

For content teams managing a large portfolio of pages, determining which content should be reviewed first can be challenging. Reviewing every page individually may require substantial time and resources, while prioritization based on a single performance metric may overlook differences in visibility, search position, and engagement. A systematic approach that considers multiple performance signals can therefore help content teams identify candidates for further investigation.

Previous research has examined the relationship between organic search visibility and user click behavior. Baye et al. [1] investigated factors associated with organic traffic to retail websites, providing evidence of the relevance of search engine optimization to organic traffic performance. Strzelecki and Miklosik [2] examined click-through rate estimation in Google organic search results using clicks and impressions data, highlighting the relevance of these metrics in analyzing search result interactions. These studies provide a research context for examining search visibility and click engagement as signals for content performance analysis.

Machine learning offers a potential approach to identifying patterns across multiple performance signals. Random Forest, introduced by Breiman [3], is an ensemble learning method that combines multiple decision trees and can be applied to classification tasks. For imbalanced classification problems, Saito and Rehmsmeier [4] explain why precision-recall analysis can provide more informative evaluation than relying solely on ROC analysis. Furthermore, validation strategies should account for grouped or hierarchical data structures to reduce the risk of misleading performance estimates. Roberts et al. [5] discuss the importance of selecting validation strategies that reflect the dependence structure of the data.

Despite the availability of search and content performance metrics, identifying which content should receive review priority remains a practical challenge. This study addresses that challenge by developing a machine learning-based scoring approach to prioritize client-content pairs using historical search and content performance signals. Specifically, March 2026 performance data are used as model inputs, while an April 2026 performance condition is used to define a proxy outcome.

The unit of analysis in this study is a client-content pair. The proposed system generates a model score, ranking, reason code, and recommended action to support content and SEO reviewers in selecting candidates for further investigation. The intended decision is to prioritize which content should be examined first, particularly content associated with sufficient search impressions but relatively low click-through rates. A wrong recommendation may cause reviewers to spend time investigating content that does not require immediate attention. Therefore, the model is designed as a human-in-the-loop decision-support tool rather than an automated content optimization system.

The contribution of this study is an evaluated content opportunity scoring approach that compares a Random Forest classifier with a transparent rule-based baseline and translates the resulting scores into ranked review recommendations. The study does not claim to predict Google's ranking algorithm or establish that refreshing content will cause improvements in search performance.

## 2. Data safety

The study uses the `FlyRank/internship-warehouse` dataset, specifically the `fact_content_daily_performance` table. The observation period covers March 1–31, 2026, while April 1–30, 2026 is used to construct the proxy outcome.

The March dataset contains 9,841,378 raw records, which were aggregated into 331,437 client-content pairs. The April dataset contains 10,424,730 raw records. After joining the aggregated March and April data using the client and content identifiers, 331,436 pairs remained for modeling.

March features were constructed by aggregating impressions, clicks, pageviews, and engaged sessions using summation, while average search position was calculated using the mean. March CTR was calculated as clicks divided by impressions, with zero assigned when impressions were zero. The April outcome was constructed from aggregated impressions and clicks.

The modeling dataset contains 331,436 unique client-content pairs and six input features:

- `gsc_impressions`
- `gsc_clicks`
- `march_ctr`
- `gsc_avg_position`
- `ga4_pageviews`
- `ga4_engaged_sessions`

The `gsc_avg_position` feature contains 154,699 missing values, representing 46.68% of the modeling dataset. Missing values were handled using median imputation within a machine learning pipeline fitted on the training data.

Client and content identifiers were retained only for joining and grouping observations. They were not used as model input features. April outcome fields, target labels, baseline scores, and reason codes were excluded from the model features to prevent target leakage. Only March performance features were used as model inputs.

The proxy target was derived from April performance and was not included among the March input features. The public report excludes client-identifying information, private queries, raw exports, and sensitive identifiers.

## 3. Baseline

A rule-based scoring approach was developed as the baseline for comparison with the Random Forest model. The baseline combines three percentile-based components:

1. Visibility score, calculated from the percentile rank of impressions.
2. CTR opportunity, calculated as one minus the percentile rank of CTR.
3. Position opportunity, calculated as one minus the percentile rank of positive average search position. Invalid or non-positive position values receive zero.

The baseline score is calculated as the average of these three components. This approach provides a transparent comparison because it uses observable March performance signals and does not require model training.

The baseline was evaluated on the same client-grouped test set used for the Random Forest model. It achieved a ROC-AUC of 0.738707 and an Average Precision of 0.318966.

## 4. Model / analysis

This study uses a Random Forest classifier to estimate the likelihood of a predefined future opportunity condition. Random Forest was selected as an ensemble learning method capable of modeling relationships among multiple numerical performance features.

The model uses six March 2026 input features: `gsc_impressions`, `gsc_clicks`, `march_ctr`, `gsc_avg_position`, `ga4_pageviews`, and `ga4_engaged_sessions`.

The proxy target, `future_opportunity`, is defined as 1 when a client-content pair has more than 100 April impressions and an April CTR below 0.001; otherwise, it is defined as 0.

The model uses median imputation followed by a Random Forest classifier. The classifier configuration consists of 200 trees, a maximum depth of 12, a minimum of 20 samples per leaf, balanced class weights, and a random state of 42. The imputation step is fitted using training data only.

The final model is trained using all available labeled observations to generate the retrospective ranked action queue. This full-data model is used for recommendation generation, while the held-out test evaluation is reported separately.

## 5. Evaluation

The model was evaluated using a client-grouped train-test split to prevent observations from the same client from appearing in both training and testing sets. The split used 80% of the data for training and 20% for testing, with a random state of 42.

The training set contained 300,879 observations from 44 clients, while the test set contained 30,557 observations from 11 clients. No client overlap was found between the two sets.

The target distribution consisted of 280,002 negative observations (84.48%) and 51,434 positive observations (15.52%). This class imbalance was considered when interpreting the evaluation results.

The Random Forest achieved a ROC-AUC of 0.921432 and an Average Precision of 0.680600. The rule-based baseline achieved a ROC-AUC of 0.738707 and an Average Precision of 0.318966 on the same test set.

At a classification threshold of 0.5, the Random Forest achieved an accuracy of 0.832379, precision of 0.509681, recall of 0.932015, and F1-score of 0.658988.

The confusion matrix contained 20,486 true negatives, 4,761 false positives, 361 false negatives, and 4,949 true positives. The results indicate that the model identified most positive proxy cases, while also producing false-positive predictions that require human verification.

## 6. Interpretation

The feature importance results indicate that `gsc_impressions` contributed the largest share of the Random Forest model's feature importance at 61.89%. This was followed by `gsc_avg_position` at 17.92%, `march_ctr` at 10.27%, `gsc_clicks` at 6.99%, `ga4_pageviews` at 2.68%, and `ga4_engaged_sessions` at 0.25%.

These results show that search visibility and search position were the most influential features in the fitted model. However, feature importance represents relative contribution within the model and does not establish causal relationships or the direction of an individual feature's effect.

The model's Average Precision was higher than that of the rule-based baseline, indicating improved positive-class ranking performance under the evaluated test conditions. Nevertheless, the positive class represented only 15.52% of the dataset, and the classification results included 4,761 false positives and 361 false negatives.

Overall, the findings suggest that combining search and content performance signals can support the prioritization of content candidates associated with the defined future opportunity condition. The findings should be interpreted within the specific March-to-April observation window and proxy target definition.

## 7. Recommendation

The final Random Forest model was trained on all 331,436 available client-content pairs to generate a retrospective ranked action queue. Each pair received a model score and was sorted in descending order of that score, with March impressions used as a secondary sorting criterion.

The queue contains three recommended action categories: `NO_ACTION`, `REVIEW_CTR`, and `REVIEW`. The `NO_ACTION` category contains 191,301 pairs, `REVIEW_CTR` contains 86,158 pairs, and `REVIEW` contains 53,977 pairs.

Reason codes provide additional context for human reviewers:

- `HIGH_IMPRESSIONS_LOW_CTR`: content with more than 100 impressions and CTR below 0.001, recommended for CTR-related review.
- `GOOD_POSITION_LOW_CTR`: content with a positive average position of 10 or better and CTR below 0.001, recommended for title and meta description review.
- `HIGH_VISIBILITY`: content with more than 100 impressions, recommended for general content performance review.
- `REVIEW`: no specific rule-based opportunity signal was identified.

A FlyRank editor can use the queue to select candidates for further investigation. For example, candidates with high impressions and low CTR may be examined for search snippet relevance, title and meta description alignment, and consistency with search intent.

The ranked queue is a retrospective decision-support artifact generated using the March-April labeled dataset. Its model scores represent estimated likelihoods of the defined proxy outcome, not probabilities that content refresh actions will improve search performance. Human review is required before any content changes are considered.

## 8. Reproducibility

The experiments were implemented in a Python notebook using a repeatable data-processing and machine learning workflow. The client-grouped split and Random Forest configuration used a random state of 42.

The reproducibility package should include the capstone notebook, the weekly assignment notebooks, the exported aggregate tables, and the privacy-safe figures used in this report.

The final repository should provide the notebook and supporting files required to reproduce the data preparation, baseline scoring, model training, evaluation, and ranked recommendation workflow.

Exact execution commands, dependency versions, and repository paths will be documented after the final repository structure and environment have been verified.

## 9. Acknowledgments & data credit

This work was developed as part of the FlyRank Machine Learning Internship Capstone. The analysis uses the **[Built on the FlyRank ML Internship dataset](https://flyrank.ai)** as its data source.


## References

[1] M. R. Baye, B. de los Santos, and M. R. Wildenbeest, “Search Engine Optimization: What Drives Organic Traffic to Retail Sites?,” *Journal of Economics & Management Strategy*, vol. 25, no. 1, pp. 6–31, 2016. doi: 10.1111/jems.12141.

[2] A. Strzelecki and A. Miklosik, “Device-dependent click-through rate estimation in Google organic search results based on clicks and impressions data,” *Aslib Journal of Information Management*, vol. 77, no. 3, pp. 513–529, 2025. doi: 10.1108/AJIM-04-2023-0107.

[3] L. Breiman, “Random Forests,” *Machine Learning*, vol. 45, no. 1, pp. 5–32, 2001. doi: 10.1023/A:1010933404324.

[4] T. Saito and M. Rehmsmeier, “The Precision-Recall Plot Is More Informative than the ROC Plot When Evaluating Binary Classifiers on Imbalanced Datasets,” *PLOS ONE*, vol. 10, no. 3, Art. no. e0118432, 2015. doi: 10.1371/journal.pone.0118432.

[5] D. R. Roberts et al., “Cross-validation strategies for data with temporal, spatial, hierarchical, or phylogenetic structure,” *Ecography*, vol. 40, no. 8, pp. 913–929, 2017. doi: 10.1111/ecog.02881.
