{
 "cells": [
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "## Advanced Lane Finding\n",
    "#### Self Driving Car Project for Udacity\n",
    "\n",
    "![alt text][image1]\n",
    "\n",
    "The goals of this project on tunnistaa ja merkitä videoon lane lines,\n",
    "auton sijainti suhteessa kaistan keskikohtaan sekä tien kaarevuussäde. \n",
    "\n",
    "\n",
    "Steps of the project are the following:\n",
    "* 1. Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.\n",
    "* Apply a distortion correction to raw images.\n",
    "* 2. Use color transforms, gradients, etc., to create a thresholded binary image.\n",
    "* 3. Apply a perspective transform to rectify binary image (\"birds-eye view\").\n",
    "* 4. Detect lane pixels and fit to find the lane boundary.\n",
    "* 5. Determine the curvature of the lane and vehicle position with respect to center.\n",
    "* 6. Warp the detected lane boundaries back onto the original image.\n",
    "* 7. Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.\n",
    "\n",
    "[//]: # (Image References)\n",
    "\n",
    "[image1]: ./examples/undistort_output.png \"Undistorted\"\n",
    "[image2]: ./test_images/test1.jpg \"Road Transformed\"\n",
    "[image3]: ./examples/binary_combo_example.jpg \"Binary Example\"\n",
    "[image4]: ./examples/warped_straight_lines.jpg \"Warp Example\"\n",
    "[image5]: ./examples/color_fit_lines.jpg \"Fit Visual\"\n",
    "[image6]: ./examples/example_output.jpg \"Output\"\n",
    "[video1]: ./project_video.mp4 \"Video\"\n",
    "\n",
    "#### [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points\n",
    "\n",
    "#### Code\n",
    "The code for this project is contained in the IPython notebook located in \"/Advanced-Lane-Finding.ipynb\" \n",
    "\n",
    "Samples are in `pictures` foulder\n",
    "\n",
    "Project requires python 3.5 and the following ohjelmistot:\n",
    "* NumPy\n",
    "* matplotlib\n",
    "* OpenCV\n",
    "* MoviePy\n",
    "\n",
    "\n",
    "### Camera Calibration\n",
    "\n",
    "#### 1. Camera matrix and distortion coefficients and an example of a distortion corrected calibration image.\n",
    "\n",
    "Kameran kalibrointi \n",
    "\n",
    "Vääristymän korjaaminen toteutettiin shakkilautakuvalla etsimällä kulmat  \n",
    "OpenCV functions `findChessboardCorners` and `drawChessboardCorners`.\n",
    "\n",
    "I start by preparing \"object points\", which are (x, y, z) coordinates of the chessboard corners in the world (assuming coordinates such that z=0). Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image. imgpoints will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.\n",
    "\n",
    "`objpoints` and `imgpoints` are then used to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function. I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: \n",
    "\n",
    "Kuva vaaristyneestä sakkilaudasta\n",
    "![alt text][image2]\n",
    "\n",
    "Kuva suoristetusta sakkilaudasta\n",
    "![alt text][image2]\n",
    "\n",
    "\n",
    "### Pipeline (single images)\n",
    "\n",
    "#### 1. An example of a distortion-corrected image.\n",
    "\n",
    "Applying the undistortion transformation to a test image\n",
    "\n",
    "Kuvan vääristymän korjaus tapahtui kalibroimalla kamera OpenCV function `calibrateCamera` ja käyttämällä OpenCV function `undistort` to remove distortion from kuvista.\n",
    "\n",
    "\n",
    "\n",
    "Original Image (distorted)\n",
    "![alt text][image1]\n",
    "\n",
    "Undistorted Image (corrected)\n",
    "![alt text][image1]\n",
    "\n",
    "\n",
    "#### 2. Description of used color transforms, gradients or other methods to create a thresholded binary image and example of a binary image result.\n",
    "\n",
    "Tässä vaiheessa muutin alkuperäisen kuvan eri kuva spaceihin joista valitsin sen missä kaistaviivat parjhaiten erottuivat tiestä.\n",
    "\n",
    "\n",
    "\n",
    "\n",
    "* The S Channel from the HLS color space, with a min threshold of 180 and a max threshold of 255, did a fairly good job of identifying both the white and yellow lane lines, but did not pick up 100% of the pixels in either one, and had a tendency to get distracted by shadows on the road.\n",
    "* The L Channel from the LUV color space, with a min threshold of 225 and a max threshold of 255, did an almost perfect job of picking up the white lane lines, but completely ignored the yellow lines.\n",
    "* The B channel from the Lab color space, with a min threshold of 155 and an upper threshold of 200, did a better job than the S channel in identifying the yellow lines, but completely ignored the white lines.\n",
    "\n",
    "I chose to create a combined binary threshold based on the three above mentioned binary thresholds, to create one combination thresholded image which does a great job of highlighting almost all of the white and yellow lane lines.\n",
    "\n",
    "Original Image\n",
    "![alt text][image3]\n",
    "\n",
    "Warped Image\n",
    "\n",
    "S binary treshold\n",
    "B binary treshold\n",
    "L binary treshold\n",
    "Combined color binary treshold\n",
    "\n",
    "\n",
    "\n",
    "\n",
    "\n",
    "#### 3. Describtion of perspective transform and an example of a transformed image.\n",
    "\n",
    "Tavoitteena oli muuttaa kuva \"birds eye view\" of the road. Kekittyen ainoastaan siihen osaan tiestä missänäkyy kaistaviivat.\n",
    "\n",
    "Kuvan kääntäminen tapahtui testikuvien pohjalta käyttäen seuraavia \n",
    "OpenCV functions `getPerspectiveTransform` and `warpPerspective` \n",
    "\n",
    "\n",
    "\n",
    "----\n",
    "\n",
    "I chose the hardcode the source and destination points in the following manner:\n",
    "\n",
    "```python\n",
    "src = np.float32(\n",
    "    [[(img_size[0] / 2) - 55, img_size[1] / 2 + 100],\n",
    "    [((img_size[0] / 6) - 10), img_size[1]],\n",
    "    [(img_size[0] * 5 / 6) + 60, img_size[1]],\n",
    "    [(img_size[0] / 2 + 55), img_size[1] / 2 + 100]])\n",
    "dst = np.float32(\n",
    "    [[(img_size[0] / 4), 0],\n",
    "    [(img_size[0] / 4), img_size[1]],\n",
    "    [(img_size[0] * 3 / 4), img_size[1]],\n",
    "    [(img_size[0] * 3 / 4), 0]])\n",
    "```\n",
    "\n",
    "This resulted in the following source and destination points:\n",
    "\n",
    "| Source        | Destination   | \n",
    "|:-------------:|:-------------:| \n",
    "| 585, 460      | 320, 0        | \n",
    "| 203, 720      | 320, 720      |\n",
    "| 1127, 720     | 960, 720      |\n",
    "| 695, 460      | 960, 0        |\n",
    "\n",
    "-----\n",
    "```python\n",
    "corners = np.float32([[190,720],[589,457],[698,457],[1145,720]])\n",
    "    new_top_left=np.array([corners[0,0],0])\n",
    "    new_top_right=np.array([corners[3,0],0])\n",
    "    offset=[150,0]\n",
    "    \n",
    "    img_size = (img.shape[1], img.shape[0])\n",
    "    src = np.float32([corners[0],corners[1],corners[2],corners[3]])\n",
    "    dst = np.float32([corners[0]+offset,new_top_left+offset,new_top_right-offset ,corners[3]-offset])    \n",
    "```\n",
    "\n",
    "| Source        | Destination   | \n",
    "|:-------------:|:-------------:| \n",
    "| 190, 720      | 340, 720        | \n",
    "| 589, 457      | 340, 0      |\n",
    "| 698, 457     | 995, 0      |\n",
    "| 1145, 720      | 995, 720        |\n",
    "\n",
    "\n",
    "\n",
    "I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.\n",
    "\n",
    "Original Image\n",
    "![alt text][image4]\n",
    "\n",
    "Undistorted Image\n",
    "![alt text][image4]\n",
    "\n",
    "\n",
    "#### 4. Identifying lane-line pixels and fit their positions with a polynomial\n",
    "\n",
    "* Identifying peaks in a histogram of the image to determine location of lane lines.\n",
    "* Identifying all non zero pixels around histogram peaks using the numpy function numpy.nonzero().\n",
    "* Fitting a polynomial to each lane using the numpy function numpy.polyfit().\n",
    "\n",
    "\n",
    "\n",
    "\n",
    "\n",
    "\n",
    "![alt text][image5]\n",
    "\n",
    "#### 5. Calculating the radius of curvature of the lane and the position of the vehicle with respect to center.\n",
    "\n",
    "\n",
    "The radius of curvature is computed upon calling the Line.update() method of a line. The method that does the computation is called Line.get_radius_of_curvature(). The mathematics involved is summarized in this tutorial here.\n",
    "For a second order polynomial f(y)=A y^2 +B y + C the radius of curvature is given by R = [(1+(2 Ay +B)^2 )^3/2]/|2A|.\n",
    "\n",
    "The distance from the center of the lane is computed in the Line.set_line_base_pos() method, which essentially measures the distance to each lane and computes the position assuming the lane has a given fixed width of 3.7m.\n",
    "-> https://github.com/ksakmann/CarND-Advanced-Lane-Lines\n",
    "----\n",
    "* Calculated the average of the x intercepts from each of the two polynomials `position = (rightx_int+leftx_int)/2`\n",
    "* Calculated the distance from center by taking the absolute value of the vehicle position minus the halfway point along the horizontal axis `distance_from_center = abs(image_width/2 - position)`\n",
    "* If the horizontal position of the car was greater than `image_width/2` than the car was considered to be left of center, otherwise right of center.\n",
    "* Finally, the distance from center was converted from pixels to meters by multiplying the number of pixels by `3.7/700`.\n",
    "\n",
    "\n",
    "Seuraavaksi Laskin tien kaareutumisen laskeminen reunaviivoista seuraavalla koodilla: \n",
    "`\n",
    "ym_per_pix = 30./720 # meters per pixel in y dimension\n",
    "xm_per_pix = 3.7/700 # meteres per pixel in x dimension\n",
    "left_fit_cr = np.polyfit(lefty*ym_per_pix, leftx*xm_per_pix, 2)\n",
    "right_fit_cr = np.polyfit(righty*ym_per_pix, rightx*xm_per_pix, 2)\n",
    "left_curverad = ((1 + (2*left_fit_cr[0]*np.max(lefty) + left_fit_cr[1])**2)**1.5) \\\n",
    "                             /np.absolute(2*left_fit_cr[0])\n",
    "right_curverad = ((1 + (2*right_fit_cr[0]*np.max(lefty) + right_fit_cr[1])**2)**1.5) \\\n",
    "                                /np.absolute(2*right_fit_cr[0])            \n",
    "`\n",
    "Huomaa että lopullinen radius on keskiarvo oikeasta ja vasemmasta viivasta\n",
    "\n",
    "Source: http://www.intmath.com/applications-differentiation/8-radius-curvature.php\n",
    "\n",
    "\n",
    "#### 6. Example image of my result plotted back down onto the road such that the lane area is identified clearly.\n",
    "\n",
    "Kaistaalueen merkintä tapahtui piirtämällä viivat kaistaviivojen päälle sekä täyttämällä niiden välinen alue puoliläpäisevällä värityksellä.\n",
    "\n",
    "Here is an example of my result on a test image:\n",
    "\n",
    "Kaistaviivojen merkintä (birds eye)\n",
    "![alt text][image6]\n",
    "\n",
    "Kaista-alueen, kohdistuksen sekä säteen merkintä \n",
    "![alt text][image6]\n",
    "\n",
    "---\n",
    "\n",
    "### Pipeline (video)\n",
    "\n",
    "Lopuksi tein pysäytyskuville tehdyt tunnistukset ja merkinnät esimerkki videoihin jotka vastaavat reaaliaikaista kuvaa autosta.\n",
    "\n",
    "Tavoitteena oli saada tunnistuksesta mahdollisimman pehmeä ja nykimätön. \n",
    "- keskiarvon käyttö framien välillä\n",
    "\n",
    "Ohjelma tarkistaa onko edellisesta kuvasta löytynyt reunaviiva. Jos viiva on löytynyt niin silloin ohjelma tarkistaa seuraavasta kuvasta vain vastaavan kohdan je sen lähialueen. Näin koko kuvaa ei tarvitse tutkia.\n",
    "\n",
    "Jos ohjelma ei löydä viivaa oletetulta alueelta, niin silloin ohjelma skannaa läpi koko kuvan tyhjältä pöydältä.\n",
    "\n",
    "Jotta lopputuloksesta tulisi pehmeä käytetään merkinnässä usean kuvan keskiarvoa.\n",
    "\n",
    "Project Video\n",
    "[link to my video result](./project_video.mp4)\n",
    "\n",
    "Challence Video\n",
    "[link to my video result](./project_video.mp4)\n",
    "---\n",
    "\n",
    "### Discussion\n",
    "\n",
    "Tehtävässä olevassa videossa on vähän sää ja valaistus vaihteluita ja se helpottaa viivojen löytymistä. \n",
    "\n",
    "Jatkossa voisi kokella soveltaa ohjelmaa itsekuvattuihin näkymiin erilaisista olosuhteista jolloin näkisi kuinka kaistaviivat löytyy vielä haastavimmissa olosuhteissa.\n",
    "\n",
    "Erilaiset tienpinnat, vesi- tai lumisade, voimakaat valoisuuden vaihtelut.\n"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "metadata": {
    "collapsed": true
   },
   "outputs": [],
   "source": []
  }
 ],
 "metadata": {
  "kernelspec": {
   "display_name": "Python 3",
   "language": "python",
   "name": "python3"
  },
  "language_info": {
   "codemirror_mode": {
    "name": "ipython",
    "version": 3
   },
   "file_extension": ".py",
   "mimetype": "text/x-python",
   "name": "python",
   "nbconvert_exporter": "python",
   "pygments_lexer": "ipython3",
   "version": "3.5.2"
  },
  "widgets": {
   "state": {},
   "version": "1.1.2"
  }
 },
 "nbformat": 4,
 "nbformat_minor": 2
}