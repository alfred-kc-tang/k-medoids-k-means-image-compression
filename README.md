# [K-Medoids and K-Means Clustering for Image Compression](https://alfred-kctang.github.io/k-medoids-k-means-image-compression/)

## Table of Contents

* [Goal](#goal)
* [Methodology](#methodology)
* [Findings](#findings)
* [Keywords](#keywords)
* [Coding Style](#coding-style)
* [License](#license)

## Goal

This project aims at utilizing the k-medoids and k-means clustering algorithms to compress images, so that the size of the images can be reduced without reducing their quality significantly. The two algorithms are written using only NumPy and SciPy without importing a machine learning library, such as scikit-learn. Comparison on the quality of the compressed images between the two algorithms will then be drawn.

## Methodology

In both the K-medoids and K-means algorithms, the cluster centers (i.e. centroids) are initialized as the pixel points from the image picked randomly, so the initial location of centroids is randomized. All pixel points are then assigned with their corresponding cluster whose centroid has the lowest distance. For K-medoids algorithm, during the iterative process, the pixel point with the lowest distances with other points from within the same cluster is chosen as the representative of each cluster, i.e. the centroid, in order to minimize the distortion function per cluster; for the total distortion function is the sum of the cluster distortion function across all the clusters. For K-means algorithm, the mean of the pixel points in a given cluster is selected as the representative, so that the distortion function per cluster is minimized. 

The assignment step comes after the above update step: all the pixel points will once again be assigned with their corresponding cluster with the updated centroid. Here the distortion function will be calculated to see if the new centroids can reduce the distortion function: if it does then the new centroids will be used as the updated centroids; otherwise, keep using the one from the last iteration and stop the iteration right away. Thus, iterative process aims at reducing the total distortion function, and stops when the distortion function no longer decreases. Speaking of distance measures, I tried Manhattan, Euclidean and Minkowski distances, and I decided to use Manhattan because it leads to the shorter computational time without significant impact on the quality of the compressed image. One side note, in case of empty cluster errors, where there is no pixel point or only the pixel point used as the centroid in the cluster, I chose the point that is the most far away from this centroid as the new centroid.

## Findings

After running the K-medoids algorithm with my picture, I observed that the compressed picture using lower small K values like 3 has a lower image quality, such as fewer colors and is less sharp. This makes sense since different pixel points are replaced by only the few representative RGB components of their cluster centroids, which results in an image captured in terms of much fewer pixels. Yet the compressed image still preserves the basic shapes of objects depicted by the original image, and it is more artistic in some sense. On the other hand, the compressed picture using higher K values like 32 or even higher has a higher image quality, with more colors and sharpness. With a high value of K such as 512, the compressed image still appears the same as the original one. In general, it takes much longer to converge when the number of K is too small or large, e.g. it takes around 8 minutes when K is 3 and around 6 minutes when K is 8192; for the update step takes longer time to compute the distances between much more points from within the same cluster when K is too small, whereas it also takes a long time to compute the distances between each pixel points and more number of centroids when K is too large. In general, the algorithm runs much faster when we choose K in between, such as 32 and even 512.

When both the K-medoids and K-means algorithms were run with the intentional poor initialization of choosing similar unique RGB components from the image as the centroids, it affects the final result negatively because the pixels of the compressed image are replaced by similar centroid pixels that are not representative of some of the pixels. This results in the compressed image of lower quality, compared with those outputted from the algorithm using random initialization. I do see different results for each trial with different initial assignments, and such a difference is more probable when the K is small. When K is small, the initialized representative pixels are more likely to be different, which produces the compressed image in different colors. Such an experiment shows the initialization problem of K-medoids and K-means, where the algorithm could result in suboptimal clustering when the initial centroids are selected poorly. This also happens with some probability under our random initialization, so there are some techniques to deal with this initialization problem, such as using K-means++ which picks most dissimilar points from the previously selected centroids with higher probability. Yet for simplicity, the random initialization is used in my algorithm, and it mitigates the problem only when a poor initial centroid is found– assigning the most dissimilar point as the new centroid when there is no pixel point or only the centroid pixel point itself in the cluster.

In comparison with the K-medoids, K-means algorithm has a shorter running time than its K-medoids counterpart. The extent to which K-means runs faster than K-medoids depends on the number of K, e.g. it takes K-means just around 84 millisecond while it takes K-medoids around 8 minutes when K is 3, yet it does not make a much difference when K is 512 - it takes K-means 5.3 seconds while it takes K-medoids 6.8 seconds. The reason is obvious: the former algorithm does not need to compute the distances between pixel points from within the same cluster, for the sake of updating the centroids; instead, it just computes the mean values of the RGB components from the pixel points in the same cluster. As a result, it saves a huge amount of computation during the iterative process. However, K-means outputted the compressed image with a narrower range colors and lower sharpness than those from K-medoids, so the output is of worse quality. Sometimes it does not work very well in terms of capturing the basic shapes of the objects depicted by the original image, it is not as robust as its K-medoids counterpart on different images. It indicates the fact that K-medoids algorithm is more robust to noise and outliers as compared to K-means, since the former is set to minimize the sum of pairwise dissimilarities instead of sum of the sum of a given distance metric used by the latter. In conclusion, K-means runs faster than K-medoids at the cost of robustness and output quality.

The effects of varying values of K on the compressed image are shown by the following images:

K-medoids: K = 32          |Original                   |K-means: K = 32
:-------------------------:|:-------------------------:|:-------------------------:
![CUHK](https://github.com/alfred-kctang/k-medoids-k-means-image-compression/blob/master/my_picture_32K_medoids.bmp?raw=true)  |  ![CUHK](https://github.com/alfred-kctang/k-medoids-k-means-image-compression/blob/master/my_picture.bmp?raw=true)  |  ![CUHK](https://github.com/alfred-kctang/k-medoids-k-means-image-compression/blob/master/my_picture_32K_means.bmp?raw=true)

K-medoids: K = 512         |Original                   |K-means: K = 512
:-------------------------:|:-------------------------:|:-------------------------:
![CUHK](https://github.com/alfred-kctang/k-medoids-k-means-image-compression/blob/master/my_picture_512K_medoids.bmp?raw=true)  |  ![CUHK](https://github.com/alfred-kctang/k-medoids-k-means-image-compression/blob/master/my_picture.bmp?raw=true)  |  ![CUHK](https://github.com/alfred-kctang/k-medoids-k-means-image-compression/blob/master/my_picture_512K_means.bmp?raw=true)

K-medoids: K = 1024        |Original                   |K-means: K = 1024
:-------------------------:|:-------------------------:|:-------------------------:
![CUHK](https://github.com/alfred-kctang/k-medoids-k-means-image-compression/blob/master/my_picture_1024K_medoids.bmp?raw=true)  |  ![CUHK](https://github.com/alfred-kctang/k-medoids-k-means-image-compression/blob/master/my_picture.bmp?raw=true)  |  ![CUHK](https://github.com/alfred-kctang/k-medoids-k-means-image-compression/blob/master/my_picture_1024K_means.bmp?raw=true)

K-medoids: K = 2048        |Original                   |K-means: K = 2048
:-------------------------:|:-------------------------:|:-------------------------:
![CUHK](https://github.com/alfred-kctang/k-medoids-k-means-image-compression/blob/master/my_picture_2048K_medoids.bmp?raw=true)  |  ![CUHK](https://github.com/alfred-kctang/k-medoids-k-means-image-compression/blob/master/my_picture.bmp?raw=true)  |  ![CUHK](https://github.com/alfred-kctang/k-medoids-k-means-image-compression/blob/master/my_picture_2048K_means.bmp?raw=true)

K-medoids: K = 4096        |Original                   |K-means: K = 4096
:-------------------------:|:-------------------------:|:-------------------------:
![CUHK](https://github.com/alfred-kctang/k-medoids-k-means-image-compression/blob/master/my_picture_4096K_medoids.bmp?raw=true)  |  ![CUHK](https://github.com/alfred-kctang/k-medoids-k-means-image-compression/blob/master/my_picture.bmp?raw=true)  |  ![CUHK](https://github.com/alfred-kctang/k-medoids-k-means-image-compression/blob/master/my_picture_4096K_means.bmp?raw=true)

## Keywords

clustering, k-means clustering, k-medoids clustering, image compression

## Coding Style

This project adopts the recommended practice of [PEP 8](https://www.python.org/dev/peps/pep-0008/) as well as [PEP 257](https://www.python.org/dev/peps/pep-0257/), and also by reference to [Google's Python Style Guide](https://google.github.io/styleguide/pyguide.html).

## License

This repository is covered under the [MIT License](https://github.com/alfred-kctang/k-medoids-k-means-image-compression/blob/master/LICENSE).
