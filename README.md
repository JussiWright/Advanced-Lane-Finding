
## Advanced Lane Finding
#### Self Driving Car Project for Udacity
- Kesken -

![alt text][image6]

The goals of this project on tunnistaa ja merkitä videoon lane lines,
auton sijainti suhteessa kaistan keskikohtaan sekä tien kaarevuussäde. 


Steps of the project are the following:
* 1. Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* 2. Use color transforms, gradients, etc., to create a thresholded binary image.
* 3. Apply a perspective transform to rectify binary image ("birds-eye view").
* 4. Detect lane pixels and fit to find the lane boundary.
* 5. Determine the curvature of the lane and vehicle position with respect to center.
* 6. Warp the detected lane boundaries back onto the original image.
* 7. Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

[//]: # (Image References)

[image1]: ./examples/undistort_output.png "Undistorted"
[image2]: ./test_images/test1.jpg "Road Transformed"
[image3]: ./examples/binary_combo_example.jpg "Binary Example"
[image4]: ./examples/warped_straight_lines.jpg "Warp Example"
[image5]: ./examples/color_fit_lines.jpg "Fit Visual"
[image6]: ./examples/example_output.jpg "Output"
[video1]: ./project_video.mp4 "Video"

#### [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

#### Code
The code for this project is contained in the IPython notebook located in "/Advanced-Lane-Finding.ipynb" 

Samples are in `pictures` foulder

Project requires python 3.5 and the following ohjelmistot:
* NumPy
* matplotlib
* OpenCV
* MoviePy


### Camera Calibration

#### 1. Camera matrix and distortion coefficients and an example of a distortion corrected calibration image.

Kameran kalibrointi 

Vääristymän korjaaminen toteutettiin shakkilautakuvalla etsimällä kulmat  
OpenCV functions `findChessboardCorners` and `drawChessboardCorners`.

I start by preparing "object points", which are (x, y, z) coordinates of the chessboard corners in the world (assuming coordinates such that z=0). Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image. imgpoints will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.

`objpoints` and `imgpoints` are then used to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function. I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

Kuva vaaristyneestä ja suoristetusta sakkilaudasta
![alt text][image1]


### Pipeline (single images)

#### 1. An example of a distortion-corrected image.

Applying the undistortion transformation to a test image

Kuvan vääristymän korjaus tapahtui kalibroimalla kamera OpenCV function `calibrateCamera` ja käyttämällä OpenCV function `undistort` to remove distortion from kuvista.



Original Image (distorted)





#### 2. Description of used color transforms, gradients or other methods to create a thresholded binary image and example of a binary image result.

Tässä vaiheessa muutin alkuperäisen kuvan eri kuva spaceihin joista valitsin sen missä kaistaviivat parjhaiten erottuivat tiestä.




* The S Channel from the HLS color space, with a min threshold of 180 and a max threshold of 255, did a fairly good job of identifying both the white and yellow lane lines, but did not pick up 100% of the pixels in either one, and had a tendency to get distracted by shadows on the road.
* The L Channel from the LUV color space, with a min threshold of 225 and a max threshold of 255, did an almost perfect job of picking up the white lane lines, but completely ignored the yellow lines.
* The B channel from the Lab color space, with a min threshold of 155 and an upper threshold of 200, did a better job than the S channel in identifying the yellow lines, but completely ignored the white lines.

I chose to create a combined binary threshold based on the three above mentioned binary thresholds, to create one combination thresholded image which does a great job of highlighting almost all of the white and yellow lane lines.

Original Image
![alt text][image3]

Warped Image

S binary treshold
B binary treshold
L binary treshold
Combined color binary treshold





#### 3. Describtion of perspective transform and an example of a transformed image.

Tavoitteena oli muuttaa kuva "birds eye view" of the road. Kekittyen ainoastaan siihen osaan tiestä missänäkyy kaistaviivat.

Kuvan kääntäminen tapahtui testikuvien pohjalta käyttäen seuraavia 
OpenCV functions `getPerspectiveTransform` and `warpPerspective` 



----

I chose the hardcode the source and destination points in the following manner:

```python
src = np.float32(
    [[(img_size[0] / 2) - 55, img_size[1] / 2 + 100],
    [((img_size[0] / 6) - 10), img_size[1]],
    [(img_size[0] * 5 / 6) + 60, img_size[1]],
    [(img_size[0] / 2 + 55), img_size[1] / 2 + 100]])
dst = np.float32(
    [[(img_size[0] / 4), 0],
    [(img_size[0] / 4), img_size[1]],
    [(img_size[0] * 3 / 4), img_size[1]],
    [(img_size[0] * 3 / 4), 0]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 585, 460      | 320, 0        | 
| 203, 720      | 320, 720      |
| 1127, 720     | 960, 720      |
| 695, 460      | 960, 0        |

-----
```python
corners = np.float32([[190,720],[589,457],[698,457],[1145,720]])
    new_top_left=np.array([corners[0,0],0])
    new_top_right=np.array([corners[3,0],0])
    offset=[150,0]
    
    img_size = (img.shape[1], img.shape[0])
    src = np.float32([corners[0],corners[1],corners[2],corners[3]])
    dst = np.float32([corners[0]+offset,new_top_left+offset,new_top_right-offset ,corners[3]-offset])    
```

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 190, 720      | 340, 720        | 
| 589, 457      | 340, 0      |
| 698, 457     | 995, 0      |
| 1145, 720      | 995, 720        |



I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

Original Image
![alt text][image4]

Undistorted Image
![alt text][image4]


#### 4. Identifying lane-line pixels and fit their positions with a polynomial

* Identifying peaks in a histogram of the image to determine location of lane lines.
* Identifying all non zero pixels around histogram peaks using the numpy function numpy.nonzero().
* Fitting a polynomial to each lane using the numpy function numpy.polyfit().






![alt text][image5]

#### 5. Calculating the radius of curvature of the lane and the position of the vehicle with respect to center.


The radius of curvature is computed upon calling the Line.update() method of a line. The method that does the computation is called Line.get_radius_of_curvature(). The mathematics involved is summarized in this tutorial here.
For a second order polynomial f(y)=A y^2 +B y + C the radius of curvature is given by R = [(1+(2 Ay +B)^2 )^3/2]/|2A|.

The distance from the center of the lane is computed in the Line.set_line_base_pos() method, which essentially measures the distance to each lane and computes the position assuming the lane has a given fixed width of 3.7m.
-> https://github.com/ksakmann/CarND-Advanced-Lane-Lines
----
* Calculated the average of the x intercepts from each of the two polynomials `position = (rightx_int+leftx_int)/2`
* Calculated the distance from center by taking the absolute value of the vehicle position minus the halfway point along the horizontal axis `distance_from_center = abs(image_width/2 - position)`
* If the horizontal position of the car was greater than `image_width/2` than the car was considered to be left of center, otherwise right of center.
* Finally, the distance from center was converted from pixels to meters by multiplying the number of pixels by `3.7/700`.


Seuraavaksi Laskin tien kaareutumisen laskeminen reunaviivoista seuraavalla koodilla: 
`
ym_per_pix = 30./720 # meters per pixel in y dimension
xm_per_pix = 3.7/700 # meteres per pixel in x dimension
left_fit_cr = np.polyfit(lefty*ym_per_pix, leftx*xm_per_pix, 2)
right_fit_cr = np.polyfit(righty*ym_per_pix, rightx*xm_per_pix, 2)
left_curverad = ((1 + (2*left_fit_cr[0]*np.max(lefty) + left_fit_cr[1])**2)**1.5) \
                             /np.absolute(2*left_fit_cr[0])
right_curverad = ((1 + (2*right_fit_cr[0]*np.max(lefty) + right_fit_cr[1])**2)**1.5) \
                                /np.absolute(2*right_fit_cr[0])            
`
Huomaa että lopullinen radius on keskiarvo oikeasta ja vasemmasta viivasta

Source: http://www.intmath.com/applications-differentiation/8-radius-curvature.php


#### 6. Example image of my result plotted back down onto the road such that the lane area is identified clearly.

Kaistaalueen merkintä tapahtui piirtämällä viivat kaistaviivojen päälle sekä täyttämällä niiden välinen alue puoliläpäisevällä värityksellä.

Here is an example of my result on a test image:

Kaistaviivojen merkintä (birds eye)
![alt text][image6]

Kaista-alueen, kohdistuksen sekä säteen merkintä 
![alt text][image6]

---

### Pipeline (video)

Lopuksi tein pysäytyskuville tehdyt tunnistukset ja merkinnät esimerkki videoihin jotka vastaavat reaaliaikaista kuvaa autosta.

Tavoitteena oli saada tunnistuksesta mahdollisimman pehmeä ja nykimätön. 
- keskiarvon käyttö framien välillä

Ohjelma tarkistaa onko edellisesta kuvasta löytynyt reunaviiva. Jos viiva on löytynyt niin silloin ohjelma tarkistaa seuraavasta kuvasta vain vastaavan kohdan je sen lähialueen. Näin koko kuvaa ei tarvitse tutkia.

Jos ohjelma ei löydä viivaa oletetulta alueelta, niin silloin ohjelma skannaa läpi koko kuvan tyhjältä pöydältä.

Jotta lopputuloksesta tulisi pehmeä käytetään merkinnässä usean kuvan keskiarvoa.

Project Video
[link to my video result](./project_video.mp4)


### Discussion

Tehtävässä olevassa videossa on vähän sää ja valaistus vaihteluita ja se helpottaa viivojen löytymistä. 

Jatkossa voisi kokella soveltaa ohjelmaa itsekuvattuihin näkymiin erilaisista olosuhteista jolloin näkisi kuinka kaistaviivat löytyy vielä haastavimmissa olosuhteissa.

Erilaiset tienpinnat, vesi- tai lumisade, voimakaat valoisuuden vaihtelut.



```python

```
