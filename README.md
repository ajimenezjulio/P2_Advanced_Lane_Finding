## Advanced Lane Finding
[![Python 3.6](https://img.shields.io/badge/python-3.6-blue.svg)](https://www.python.org/downloads/release/python-360/)
[![Udacity - Self-Driving Car NanoDegree](https://s3.amazonaws.com/udacity-sdc/github/shield-carnd.svg)](http://www.udacity.com/drive)


Lane line detection is a critical technique for the design of systems that allows a car to drive itself. The detection of these lines allows our car to stay on right path and follow it, in the same way that we use our eyes to stay on the lane. This project uses an approach based on image processing, it uses Python as the main language and OpenCV as complementary framework for image analysis and processing.

![](https://github.com/ajimenezjulio/P2_Advanced_Lane_Finding/blob/master/Results/intro.gif)

---

### 1. Approach

The approach consisted of 6 steps:

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
<br>

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
<br>

3. **Thresholding**: Thresholding is a method of segmenting image. In this project we use a combined threshold (sobel and colorspaces) to create a binary image where the lane line pixels are activated.
<ul>
<ul>
<li>Convolute a sobel of kernel size of 5 in the x direction with the image:
<p align="center" style="text-align: center;"><img align="center" src="https://tex.s2cms.ru/svg/%0ASob%20%26%3D%20s_x%20%5Ccircledast%20img%20%0A" alt="
Sob &amp;= s_x \circledast img 
" /></p>
  
<li>Get S channel from HLS colorspace transform:
<p align="center" style="text-align: center;"><img align="center" src="https://tex.s2cms.ru/svg/%0AV_%7Bmax%7D%20%5Cleftarrow%20%7Bmax%7D(R%2CG%2CB)%20%5C%3B%5C%3B%5C%3B%5C%3B%0AV_%7Bmin%7D%20%5Cleftarrow%20%7Bmax%7D(R%2CG%2CB)%0A" alt="
V_{max} \leftarrow {max}(R,G,B) \;\;\;\;
V_{min} \leftarrow {max}(R,G,B)
" /></p>
<p align="center" style="text-align: center;"><img align="center" src="https://tex.s2cms.ru/svg/%0AL%20%5Cleftarrow%20%5Cfrac%7BV_%7Bmax%7D%20%2B%20V_%7Bmin%7D%7D%7B2%7D%20%5C%3B%5C%3B%5C%3B%5C%3B%5C%3B%5C%3B%5C%3B%5C%3B%0AS%20%5Cleftarrow%20%5Cbegin%7Bcases%7D%20%0A%5Cdfrac%7BV_%7Bmax%7D%20-%20V_%7Bmin%7D%7D%7BV_%7Bmax%7D%20%2B%20V_%7Bmin%7D%7D%20%26%20if%20%5C(L%20%3C%200.5%5C)%20%5C%5C%20%5C%5C%0A%5Cfrac%7BV_%7Bmax%7D%20-%20V_%7Bmin%7D%7D%7B2%20-%20(V_%7Bmax%7D%20%2B%20V_%7Bmin%7D)%7D%20%26%20if%20%5C(L%20%5Cge%200.5%5C)%0A%5Cend%7Bcases%7D%0A" alt="
L \leftarrow \frac{V_{max} + V_{min}}{2} \;\;\;\;\;\;\;\;
S \leftarrow \begin{cases} 
\dfrac{V_{max} - V_{min}}{V_{max} + V_{min}} &amp; if \(L &lt; 0.5\) \\ \\
\frac{V_{max} - V_{min}}{2 - (V_{max} + V_{min})} &amp; if \(L \ge 0.5\)
\end{cases}
" /></p>
  
<li>Get B channel from LAB colorspace transform:
<p align="center" style="text-align: center;"><img align="center" src="https://tex.s2cms.ru/svg/%0A%5Cbegin%7Bbmatrix%7D%7BX%20%5C%5C%20Y%20%5C%5C%20Z%7D%20%5Cend%7Bbmatrix%7D%0A%5Cleftarrow%0A%5Cbegin%7Bbmatrix%7D%7B0.412453%20%26%200.357580%20%26%200.180423%20%5C%5C%200.212671%20%26%200.715160%20%26%200.072169%20%5C%5C%200.019334%20%26%200.119193%20%26%200.950227%7D%5Cend%7Bbmatrix%7D%0A%5Ccdot%20%5Cbegin%7Bbmatrix%7D%7BR%20%5C%5CG%20%5C%5C%20B%7D%5Cend%7Bbmatrix%7D%0A" alt="
\begin{bmatrix}{X \\ Y \\ Z} \end{bmatrix}
\leftarrow
\begin{bmatrix}{0.412453 &amp; 0.357580 &amp; 0.180423 \\ 0.212671 &amp; 0.715160 &amp; 0.072169 \\ 0.019334 &amp; 0.119193 &amp; 0.950227}\end{bmatrix}
\cdot \begin{bmatrix}{R \\G \\ B}\end{bmatrix}
" /></p>
<p align="center" style="text-align: center;"><img align="center" src="https://tex.s2cms.ru/svg/%0AX%20%5Cleftarrow%20X%2FX_n%2C%20%5Ctext%7Bwhere%7D%20X_n%20%3D%200.950456%20%5C%3B%5C%3B%5C%3B%5C%3B%5C%3B%5C%3B%5C%3B%5C%3B%5C%3B%0AZ%20%5Cleftarrow%20Z%2FZ_n%2C%20%5Ctext%7Bwhere%7D%20Z_n%20%3D%201.088754%0A" alt="
X \leftarrow X/X_n, \text{where} X_n = 0.950456 \;\;\;\;\;\;\;\;\;
Z \leftarrow Z/Z_n, \text{where} Z_n = 1.088754
" /></p>
<p align="center" style="text-align: center;"><img align="center" src="https://tex.s2cms.ru/svg/%0AL%20%5Cleftarrow%20%5Cbegin%7Bcases%7D%20116*Y%5E%7B1%2F3%7D-16%20%26%20for%20%5C%20Y%3E0.008856%5C%20%5C%5C%20%0A903.3*Y%20%26%20for%20%5C%20Y%20%5Cle%200.008856%5C%7D%20%5Cend%7Bcases%7D%0A" alt="
L \leftarrow \begin{cases} 116*Y^{1/3}-16 &amp; for \ Y&gt;0.008856\ \\ 
903.3*Y &amp; for \ Y \le 0.008856\} \end{cases}
" /></p>
<p align="center" style="text-align: center;"><img align="center" src="https://tex.s2cms.ru/svg/%0AA%20%5Cleftarrow%20500%20(f(X)-f(Y))%20%2B%20delta%20%5C%3B%5C%3B%5C%3B%5C%3B%5C%3B%5C%3B%5C%3B%5C%3B%5C%3B%0AB%20%5Cleftarrow%20200%20(f(Y)-f(Z))%20%2B%20delta%0A" alt="
A \leftarrow 500 (f(X)-f(Y)) + delta \;\;\;\;\;\;\;\;\;
B \leftarrow 200 (f(Y)-f(Z)) + delta
" /></p>
<p align="center" style="text-align: center;"><img align="center" src="https://tex.s2cms.ru/svg/%0Af(t)%3D%20%5Cbegin%7Bcases%7D%20t%5E%7B1%2F3%7D%20%26%20for%20%5C%20t%3E0.008856%5C)%20%5C%5C%0A7.787%20t%2B16%2F116%20%26%20for%20%5C%20t%5Cleq%200.008856%5C)%20%5Cend%7Bcases%7D%0A" alt="
f(t)= \begin{cases} t^{1/3} &amp; for \ t&gt;0.008856\) \\
7.787 t+16/116 &amp; for \ t\leq 0.008856\) \end{cases}
" /></p>
<p align="center" style="text-align: center;"><img align="center" src="https://tex.s2cms.ru/svg/%0Adelta%20%3D%20%5Cbegin%7Bcases%7D%20128%20%26%20%5Ctexttt%7Bfor%208-bit%20images%7D%20%5C%5C%0A0%20%26%20%5Ctexttt%7Bfor%20floating-point%20images%7D%20%5Cend%7Bcases%7D%0A" alt="
delta = \begin{cases} 128 &amp; \texttt{for 8-bit images} \\
0 &amp; \texttt{for floating-point images} \end{cases}
" /></p>
  
  <li> Every channel is then binarized using the next criteria:
 <p align="center" style="text-align: center;"><img align="center" src="https://tex.s2cms.ru/svg/%0ASob%20%5Cleftarrow%20%5Cbegin%7Bcases%7D%20%0A1%20%26%20if%20%5C%20%2040%20%3C%3D%20Sob%20%3C%3D%20255%20%5C%5C%20%0A0%20%26%20otherwise%20%5Cend%7Bcases%7D%0A%5C%20%5C%20%5C%20%0AS%20%5Cleftarrow%20%5Cbegin%7Bcases%7D%20%20%0A1%20%26%20if%20%5C%20%2070%20%3C%3D%20S%20%3C%3D%20255%20%5C%5C%20%0A0%20%26%20otherwise%20%5Cend%7Bcases%7D%0A%5C%20%5C%20%5C%0AB%20%5Cleftarrow%20%5Cbegin%7Bcases%7D%20%0A1%20%26%20if%20%5C%20%20140%20%3C%3D%20B%20%3C%3D%20255%20%5C%5C%20%0A0%20%26%20otherwise%20%5Cend%7Bcases%7D%0A" alt="
Sob \leftarrow \begin{cases} 
1 &amp; if \  40 &lt;= Sob &lt;= 255 \\ 
0 &amp; otherwise \end{cases}
\ \ \ 
S \leftarrow \begin{cases}  
1 &amp; if \  70 &lt;= S &lt;= 255 \\ 
0 &amp; otherwise \end{cases}
\ \ \
B \leftarrow \begin{cases} 
1 &amp; if \  140 &lt;= B &lt;= 255 \\ 
0 &amp; otherwise \end{cases}
" /></p>
<li> Finally, the binary image is made by the union of the 3 channels:
<p align="center" style="text-align: center;"><img align="center" src="https://tex.s2cms.ru/svg/%0Abinary%20%3D%20Sob%20%5Ccup%20S%20%5Ccup%20B%0A" alt="
binary = Sob \cup S \cup B
" /></p>
</ul>
</ul>
<br>

4. **Polynomial fit**: After the threshold, we fit a quadratic polynomial to each of the lane lines, this step will allow us to detect curves in the road.
<ul>
<ul>
<li> First we need to calculate a histogram in the bottom half of the image by summing all the pixels in the y axis, two peaks will be detected and will be the base for the left and right lines. To avoid noise, margins (<img src="https://render.githubusercontent.com/render/math?math=\alpha">) in the left and right edges were added.
<p align="center" style="text-align: center;"><img align="center" src="https://tex.s2cms.ru/svg/%0A%5Cbegin%7Btikzpicture%7D%5Bisosceleles%20trapezium%2F.style%20args%3D%7Bof%20width%20%231%20and%20height%20%232%0Aand%20name%20%233%7D%7Binsert%20path%3D%7B%0A(45%3A%7B%231%2Fsqrt(2)%7D)%20coordinate(%233-TR)%20--%20(-45%3A%7Bsqrt(%232*%232-%231*%231%2F2)%7D)%20coordinate(%233-BR)%20%0A--%20(-135%3A%7Bsqrt(%232*%232-%231*%231%2F2)%7D)%20coordinate(%233-BL)%20--%20(135%3A%7B%231%2Fsqrt(2)%7D)%20coordinate(%233-TL)%20--%20cycle%7D%7D%5D%0A%5Cdraw%5Bisosceleles%20trapezium%3Dof%20width%201%20and%20height%203%20and%20name%20my%20trap%5D%3B%0A%5Cdraw%5Blatex-latex%5D%20(%5Bxshift%3D-10mm%2C%20yshift%3D12mm%5Dmy%20trap-TL%20-%7C%20my%20trap-BL)%20--%20%0A(%5Bxshift%3D-10mm%5Dmy%20trap-BL)%20node%5Bmidway%2Cfill%3Dwhite%5D%20%7B%24h%24%7D%3B%0A%5Cdraw%5Bdraw%3Dblack%20and%20name%20myRect%5D%20(-2.8%2C%201.7)%20rectangle%20%2B%2B(5.5%2C-3.75)%3B%0A%5Cdraw%5Blatex-latex%5D%20(%5Bxshift%3D-8mm%2C%20yshift%3D-3mm%5D%20my%20trap-BL)%20--%20(%5Byshift%3D-3mm%5D%20my%20trap-BL)%0Anode%5Bmidway%2Cfill%3Dwhite%5D%20%7B%24%5Calpha%24%7D%3B%0A%5Cdraw%5Blatex-latex%5D%20(%5Byshift%3D-3mm%5D%20my%20trap-BL)%20--%20(%5Byshift%3D-3mm%5D%20my%20trap-BR)%0Anode%5Bmidway%2Cfill%3Dwhite%5D%20%7B%24b_i%24%7D%3B%0A%5Cend%7Btikzpicture%7D%0A" alt="
\begin{tikzpicture}[isosceleles trapezium/.style args={of width #1 and height #2
and name #3}{insert path={
(45:{#1/sqrt(2)}) coordinate(#3-TR) -- (-45:{sqrt(#2*#2-#1*#1/2)}) coordinate(#3-BR) 
-- (-135:{sqrt(#2*#2-#1*#1/2)}) coordinate(#3-BL) -- (135:{#1/sqrt(2)}) coordinate(#3-TL) -- cycle}}]
\draw[isosceleles trapezium=of width 1 and height 3 and name my trap];
\draw[latex-latex] ([xshift=-10mm, yshift=12mm]my trap-TL -| my trap-BL) -- 
([xshift=-10mm]my trap-BL) node[midway,fill=white] {$h$};
\draw[draw=black and name myRect] (-2.8, 1.7) rectangle ++(5.5,-3.75);
\draw[latex-latex] ([xshift=-8mm, yshift=-3mm] my trap-BL) -- ([yshift=-3mm] my trap-BL)
node[midway,fill=white] {$\alpha$};
\draw[latex-latex] ([yshift=-3mm] my trap-BL) -- ([yshift=-3mm] my trap-BR)
node[midway,fill=white] {$b_i$};
\end{tikzpicture}
" /></p>
<p align="center" style="text-align: center;"><img align="center" src="https://tex.s2cms.ru/svg/%0A%5Ctexttt%7Bwhere%20%7D%20%5Calpha%20%3D%20%5Cfrac%7Bimg_w%7D%7B4%7D%0A" alt="
\texttt{where } \alpha = \frac{img_w}{4}
" /></p>

<li> Next step we use the sliding window algoritm to fit 15 windows in each line starting from the base lines until the top of the image in order to look for all the pixels related to the lines. Once all pixels for each line are detected we fit a second degree polynomial in each set of the lines by minimizing the squared error.
<p align="center" style="text-align: center;"><img align="center" src="https://tex.s2cms.ru/svg/%0Ay_k%20%3D%20x_k%5Enp_0%20%2B%20...%20%2B%20x_kp_%7Bn-1%7D%20%2B%20p_n%0A" alt="
y_k = x_k^np_0 + ... + x_kp_{n-1} + p_n
" /></p>
<p align="center" style="text-align: center;"><img align="center" src="https://tex.s2cms.ru/svg/%0AE%20%3D%20%5Csum%5Ek%20%7Cp(x_j)-y_j%7C%5E2%0A" alt="
E = \sum^k |p(x_j)-y_j|^2
" /></p>

<li> For optimization, once on the next frame we don't need to blindly look again for the lines, but use previous information. Adding a margin in the polynomial equation we can create a region of interest customized for every frame that follows the line in the previous frame.
<p align="center" style="text-align: center;"><img align="center" src="https://tex.s2cms.ru/svg/%0AROI%20%3D%20%5Cbegin%7Bcases%7D%0Aay%5E2%20%2B%20bx%20%2B%20c%20-%20margin%20%5C%5C%0Aay%5E2%20%2B%20bx%20%2B%20c%20%2B%20margin%0A%5Cend%7Bcases%7D%0A" alt="
ROI = \begin{cases}
ay^2 + bx + c - margin \\
ay^2 + bx + c + margin
\end{cases}
" /></p>
</ul>
</ul>
<br>

5. **Polynomial fit validation**: In order to determine if the fit found is valid we need to met some criteria.
<ul>
<ul>
<li> In case that a previous fit exists we need to compare the difference between coefficients, if the difference is too big, then it's not a valid one (differences in line coefficients shouldn't vary that much from frame to frame).
<p align="center" style="text-align: center;"><img align="center" src="https://tex.s2cms.ru/svg/%0Afit%20%5Cleftarrow%0A%5Cbegin%7Bcases%7D%0Ainvalid%20%26%20if%20%5C%20%5C%20a%20%3E%200.001%20%5C%20%5Coplus%20%5C%20b%20%3E%201%20%5C%20%5Coplus%20%5C%20c%20%3E%20100%0A%5C%5C%0Avalid%20%26%20otherwise%0A%5Cend%7Bcases%7D%0A" alt="
fit \leftarrow
\begin{cases}
invalid &amp; if \ \ a &gt; 0.001 \ \oplus \ b &gt; 1 \ \oplus \ c &gt; 100
\\
valid &amp; otherwise
\end{cases}
" /></p>
  
<li> In order to make a smooth transition the fit in the current frame will be equal to an average of the previous 5 fits.
<p align="center" style="text-align: center;"><img align="center" src="https://tex.s2cms.ru/svg/%0Acurrent%5C_fit%20%3D%20%5Cfrac%7B%5Csum_i%5E4%20fit_i%7D%7B5%7D%0A" alt="
current\_fit = \frac{\sum_i^4 fit_i}{5}
" /></p>
</ul>
</ul>
<br>

6. **Measuring curvature**: Having the coefficients we can calculate the radius of curvature at any point <img src="https://render.githubusercontent.com/render/math?math=x"> of the function <img src="https://render.githubusercontent.com/render/math?math=x=f(y)">.
<p align="center" style="text-align: center;"><img align="center" src="https://tex.s2cms.ru/svg/%0AR_%7Bcurve%7D%20%3D%20%5Cfrac%7B%5Cleft(%201%20%2B%20%5Cleft(%20%5Cfrac%7Bdx%7D%7Bdy%7D%20%5Cright)%5E2%20%5Cright)%5E%5Cfrac%7B3%7D%7B2%7D%20%7D%0A%7B%5Cleft%7C%20%5Cfrac%7Bd%5E2x%7D%7Bd%5E2y%7D%20%5Cright%7C%7D%0A" alt="
R_{curve} = \frac{\left( 1 + \left( \frac{dx}{dy} \right)^2 \right)^\frac{3}{2} }
{\left| \frac{d^2x}{d^2y} \right|}
" /></p>
<p align="center" style="text-align: center;"><img align="center" src="https://tex.s2cms.ru/svg/%0Af'(y)%3D%20%5Cfrac%7Bdx%7D%7Bdy%7D%20%3D%202ay%20%2B%20b%20%5C%20%5C%20%5C%20%5C%20%5C%20%5C%20%5C%20%5C%20%5C%0Af''(y)%3D%20%5Cfrac%7Bd%5E2x%7D%7Bd%5E2y%7D%20%3D%202a%0A" alt="
f'(y)= \frac{dx}{dy} = 2ay + b \ \ \ \ \ \ \ \ \
f''(y)= \frac{d^2x}{d^2y} = 2a
" /></p>
<p align="center" style="text-align: center;"><img align="center" src="https://tex.s2cms.ru/svg/%0AR_%7Bcurve%7D%20%3D%20%5Cfrac%7B%5Cleft(%201%20%2B%20%5Cleft(%202ay%20%2B%20b%20%5Cright)%5E2%20%5Cright)%5E%5Cfrac%7B3%7D%7B2%7D%20%7D%0A%7B%5Cleft%7C%202a%20%5Cright%7C%7D%0A" alt="
R_{curve} = \frac{\left( 1 + \left( 2ay + b \right)^2 \right)^\frac{3}{2} }
{\left| 2a \right|}
" /></p>
<br>


### 2. Shortcomings

There are some flaws in this approach:

* It is not illumination invariant, so the lack of contrast and light would result in a malfunction of the program.

* Thresholding is without a doubt the core of the problem, playing with multiple thresholdings make the model more robust but also can detect undesired patterns and objects that will lead to a wrong lane detection.

* A fixed camera position and consistent image size are assumed, so positioning the camera at a different angle and handling different image sizes would have undesirable results.

<br>


### 3. Improvements

Some points of the pipeline can be improved.

1. **Perspective Transform**: The images and videos have a fixed size and perspective, right now the 4 selected points for the transform are fixed, so a better way to select them automatically based on features to handle multiple sizes and camera angles will be necessary for a more robust approach

2. **Thresholding**: Without a doubt trhesholding is the core of the algorithm, a more robust method for illumination, rotation and translation invariance should be developed for a real application. This approach showed a good performance for the *project_video* and a fairly average for the *challenge_video*, but its completely useless for the *harder_challenge_video*.

3. **Polynomal fit**: Sliding windows algorithm and its optimization for looking around previous polynomial are pretty robust methods for lane line detection, minor improvements could be done here. However, these methods are strongly dependant on the thresholding for finding a good fit, i.e. we are as strong as the weakest link.
