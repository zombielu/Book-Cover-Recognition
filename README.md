# Book Cover Recognition
## Project Goal: 
Localize and recognize book covers from various oblique angles and occlusion patterns in an input image given a database of a large set of book covers.
## Data Preparation:
### 1. Query images/Test images:
- Query image with one book:
  
<img width="320" height="150" src="https://github.com/zombielu/Book-Cover-Recognition/blob/main/images/query_images_1.png?raw=true">

- Query image with three books:
  
<img width="350" height="150" src="https://github.com/zombielu/Book-Cover-Recognition/blob/main/images/query_images_3.png?raw=true">

- Query image with five books:
  
<img width="450" height="150" src="https://github.com/zombielu/Book-Cover-Recognition/blob/main/images/query_images_5.png?raw=true">

### 2. Book cover database with 104 original book cover images:
<img width="650" height="300" src="https://github.com/zombielu/Book-Cover-Recognition/blob/main/images/database.png?raw=true">

### 3. Take the first image as an example, generate images of different viewpoints:
<img width="600" height="300" src="https://github.com/zombielu/Book-Cover-Recognition/blob/main/images/data_prep.png?raw=true">

## Algorithm Explanation:
1. Create Vocabulary Tree<br>
   - Extract the descriptors for each image in our database by SIFT.
   - Create vocabulary tree using K-means with n levels (n = 4 in our case) and each level has K branches (K = 12 in our case), so we end up with K^n words (leaves) for the vocabulary tree.
   - Create the Inverted File Index so that we can trace back the images that contains certain words.

2. Candidate Selection<br>
   Find out the top 30 image candidates based on descriptor similarity using the vocabulary tree:
   - For each descriptor of the query image:
     + Find the leaf node it belongs to;
     + Find the top 20 closest descriptors in the same leaf node;
   - Gather all the descriptor pairs from the previous step and keep the top 20000 closest ones.
   - Based on the inverted file index, find the corresponding images that contains the descriptors occurred in the 20000 descriptor pairs.
   - The images with the most amount of descriptors are our candidates, keep the top 30 ones.

3. Candidate Filtering<br>
   For each image candidate:
   - Calculate the homography matrix;
   - Rule out the image candidates if there are fewer than 10 inliers;

4. Candidate Verification<br>
   For each leftover image candidate:
   - Find the book cover segment using the homography matrix in the previous step;
   - Re-calculate the homography matrix based on the segment;
   - Rule out the image candidate if the inlier ratio is smaller than 0.6;

5. Non-maximum Suppression<br>
   - Put a box around the book cover found in the test image;
   - If there is any boxes overlap and the overlap area takes up over 70% of the area of the smaller box, keep only the one with corresponding image that has the higher inlier ratio;

## Result Display
- One Book Queries:
  
  <img width="160" height="300" src="https://github.com/zombielu/Book-Cover-Recognition/blob/main/images/results/1_query_1.png?raw=true">
  <img width="160" height="300" src="https://github.com/zombielu/Book-Cover-Recognition/blob/main/images/results/1_query_2.png?raw=true">
  <img width="160" height="300" src="https://github.com/zombielu/Book-Cover-Recognition/blob/main/images/results/1_query_3.png?raw=true">

- Three Book Queries:

  <img width="300" height="300" src="https://github.com/zombielu/Book-Cover-Recognition/blob/main/images/results/3_query_1.png?raw=true">
  <img width="300" height="300" src="https://github.com/zombielu/Book-Cover-Recognition/blob/main/images/results/3_query_2.png?raw=true">
  <img width="300" height="300" src="https://github.com/zombielu/Book-Cover-Recognition/blob/main/images/results/3_query_3.png?raw=true">

- Five Book Queries:

  <img width="350" height="300" src="https://github.com/zombielu/Book-Cover-Recognition/blob/main/images/results/5_query_1.png?raw=true">
  <img width="300" height="300" src="https://github.com/zombielu/Book-Cover-Recognition/blob/main/images/results/5_query_2.png?raw=true">

  <img width="380" height="300" src="https://github.com/zombielu/Book-Cover-Recognition/blob/main/images/results/5_query_3.png?raw=true">
  <img width="380" height="300" src="https://github.com/zombielu/Book-Cover-Recognition/blob/main/images/results/5_query_4.png?raw=true">



## Key Findings
- The performance of the efficient retrieval approach by Nister and Stewenius can be greatly affected by the noise in the image. 

- Hough Voting cannot directly be used to detect projectively transformed objects. To get a good performance, we need to combine Hough Voting with CNN.

- When the data in the database changes, we need to adjust the level number and the branch number of the vocabulary tree.

- Some book covers do not have as many descriptors as others, so the accuracy of them are lower. That’s why we may have a box that is in trapezoid shape.

- The RANSAC algorithm is used to find the homography matrix, it's not guaranteed the query results are the same every time.

## Challenges
- Pre-preparation are needed such as convert the image into appropriate resolution level. 

- There are multiple hyper-parameters to tune:
  + number of levels for the vocabulary tree
  + number of branches for the vocabulary tree
  + amount of descriptor candidates
  + amount of image candidates
  + maximum RANSAC iterations
  + threshold of inlier number during candidate filtering
  + threshold of inlier ratio during candidate verification
  + Intersection over Union (IoU) threshold for non-maximum suppression

- During image matching, the more candidates to examine, also the more RANSAC iterations applied, the higher the accuracy while the more time it takes. It's a time-accuracy tradeoff.

- The book covers in the test images must be clean and flat. Otherwise, the descriptors may differ significantly from the descriptors in the database, and therefore result in failure retrieval.

## Future Work
- A preprocessing pipeline can be added to adjust different resolutions, lighting conditions, waved problems so that the retrieval system can handle different image qualities.

- Improve the scalability of the retrieval process by adding parallel processing or distributed computing so that the retrieval system can handle large volume of images. 

- Train a CNN model for segment detection in order to achieve better book cover recognition.

- In addition to cover detection, incorporate natural language processing so that the retrieval system can also provide textural information for the users.

- Create user interface with features such as image uploading and book information display so that the retrieval system is more user-friendly.




