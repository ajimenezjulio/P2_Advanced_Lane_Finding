## Advanced Lane Finding
[![Python 3.6](https://img.shields.io/badge/python-3.6-blue.svg)](https://www.python.org/downloads/release/python-360/)
[![Udacity - Self-Driving Car NanoDegree](https://s3.amazonaws.com/udacity-sdc/github/shield-carnd.svg)](http://www.udacity.com/drive)


Lane line detection is a critical technique for the design of systems that allows a car to drive itself. The detection of these lines allows our car to stay on right path and follow it, in the same way that we use our eyes to stay on the lane. This project uses an approach based on image processing, it uses Python as the main language and OpenCV as complementary framework for image analysis and processing.


The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

---

### 1. Approach

The approach consisted of X steps:

1. **Distortion correction**: Light rays often bend a little too much or too little at the edges of the curved lenses in real cameras, this creates a *radial distortion* and if the camera’s lens is not aligned perfectly parallel to the imaging plane a *tangential distortion* also occurs. In this project the distortion coefficients were calculated to correct radial distortion using a 9 x 6 corners chessboard images.
<ul>
<ul>
<li>Radial distortion:
<p align="center" style="text-align: center;"><img align="center" src="https://tex.s2cms.ru/svg/x_%7Bdistorted%7D%20%3D%20x_%7Bideal%7D%20(1%20%2B%20k_1r%5E2%20%2B%20k_2r%5E4%20%2B%20k_3r%5E6)" alt="x_{distorted} = x_{ideal} (1 + k_1r^2 + k_2r^4 + k_3r^6)" /></p>
<p align="center" style="text-align: center;"><img align="center" src="https://tex.s2cms.ru/svg/y_%7Bdistorted%7D%20%3D%20y_%7Bideal%7D%20(1%20%2B%20k_1r%5E2%20%2B%20k_2r%5E4%20%2B%20k_3r%5E6)" alt="y_{distorted} = y_{ideal} (1 + k_1r^2 + k_2r^4 + k_3r^6)" /></p>

<li>Tangential distortion:
<p align="center" style="text-align: center;"><img align="center" src="https://tex.s2cms.ru/svg/x_%7Bcorrected%7D%20%3D%20x%20%2B%20%5B2p_1xy%20%2B%20p_2(r%5E2%20%2B%202x%5E2)%5D" alt="x_{corrected} = x + [2p_1xy + p_2(r^2 + 2x^2)]" /></p>
<p align="center" style="text-align: center;"><img align="center" src="https://tex.s2cms.ru/svg/y_%7Bcorrected%7D%20%3D%20y%20%2B%20%5Bp_1(r%5E2%20%2B%202y%5E2)%20%2B%202p_2xy%5D" alt="y_{corrected} = y + [p_1(r^2 + 2y^2) + 2p_2xy]" /></p>
</ul>
</ul>

2. **Perspective transform**: A perspective transform maps the points in a given image to different, desired, image points with a new perspective. In this project a bird’s-eye view transform that let’s us view a lane from above was used (it is useful for calculating curvature in the next steps).
<ul>
<ul>
<li>Calculate transformation and inverse matrices from four pair of points:
<p align="center" style="text-align: center;"><img align="center" src="https://tex.s2cms.ru/svg/%5Cbegin%7Bbmatrix%7D%20t_i%20x'_i%20%5C%5C%20t_i%20y'_i%20%5C%5C%20t_i%20%5Cend%7Bbmatrix%7D%20%3D%20%5Ctexttt%7Bmap%5C_matrix%7D%20%5Ccdot%20%5Cbegin%7Bbmatrix%7D%20x_i%20%5C%5C%20y_i%20%5C%5C%201%20%5Cend%7Bbmatrix%7D" alt="\begin{bmatrix} t_i x'_i \\ t_i y'_i \\ t_i \end{bmatrix} = \texttt{map\_matrix} \cdot \begin{bmatrix} x_i \\ y_i \\ 1 \end{bmatrix}" /></p>
<p align="center" style="text-align: center;"><img align="center" src="https://tex.s2cms.ru/svg/%20%5Ctexttt%7Bwhere%3A%7D%20%5C%2C%5C%2C%20dst(i)%3D(x'_i%2Cy'_i)%2C%20src(i)%3D(x_i%2C%20y_i)%2C%20i%3D0%2C1%2C2%2C3%20" alt=" \texttt{where:} \,\, dst(i)=(x'_i,y'_i), src(i)=(x_i, y_i), i=0,1,2,3 " /></p>

<li> Apply perspective transform using the matrix:
<p align="center" style="text-align: center;"><img align="center" src="https://tex.s2cms.ru/svg/%5Ctexttt%7Bdst%7D%20(x%2Cy)%20%3D%20%5Ctexttt%7Bsrc%7D%20%5Cleft%20(%20%5Cfrac%7BM_%7B11%7D%20x%20%2B%20M_%7B12%7D%20y%20%2B%20M_%7B13%7D%7D%7BM_%7B31%7D%20x%20%2B%20M_%7B32%7D%20y%20%2B%20M_%7B33%7D%7D%20%2C%20%5Cfrac%7BM_%7B21%7D%20x%20%2B%20M_%7B22%7D%20y%20%2B%20M_%7B23%7D%7D%7BM_%7B31%7D%20x%20%2B%20M_%7B32%7D%20y%20%2B%20M_%7B33%7D%7D%20%5Cright%20)" alt="\texttt{dst} (x,y) = \texttt{src} \left ( \frac{M_{11} x + M_{12} y + M_{13}}{M_{31} x + M_{32} y + M_{33}} , \frac{M_{21} x + M_{22} y + M_{23}}{M_{31} x + M_{32} y + M_{33}} \right )" /></p>

<li> Points used in this project:
<p align="center" style="text-align: center;"><img align="center" src="https://tex.s2cms.ru/svg/%5Ctexttt%7Bsrc%7D%20%3D%20%5Cbegin%7Bbmatrix%7D%20258%20%26%20682%5C%5C%20575%20%20%26%20464%20%5C%5C%20707%20%26%20464%20%5C%5C%201049%20%26%20682%20%5Cend%7Bbmatrix%7D%20%5C%3B%5C%3B%5C%3B%5C%3B%20%0A%5Ctexttt%7Bdst%7D%20%3D%20%5Cbegin%7Bbmatrix%7D%20450%20%26%20img_h%20%5C%5C%20450%20%26%200%20%5C%5C%20img_w%20-%20450%20%26%200%20%5C%5C%20img_w%20-%20450%20%26%20img_h%5Cend%7Bbmatrix%7D" alt="\texttt{src} = \begin{bmatrix} 258 &amp; 682\\ 575  &amp; 464 \\ 707 &amp; 464 \\ 1049 &amp; 682 \end{bmatrix} \;\;\;\; 
\texttt{dst} = \begin{bmatrix} 450 &amp; img_h \\ 450 &amp; 0 \\ img_w - 450 &amp; 0 \\ img_w - 450 &amp; img_h\end{bmatrix}" /></p>
</ul>
</ul>

3. **Threshold**
Thresholding is a method of segmenting image. In this project we use a combined threshold (sobel and colorspaces) to create a binary image where the lane lines pixels are activated.
