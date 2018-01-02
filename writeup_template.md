# **Finding Lane Lines on the Road**

---

## Overview

The goals / steps of this project are the following:

* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report
* 프로젝트 코드 link : [project code](https://github.com/kdsian/CarND-P1-LaneLines)
---

[image1]: ./describe/2_grayscale_before.jpg "Gray-before"
[image2]: ./describe/1_grayscale_after.jpg "Gray-after"
[image3]: ./describe/4_canny.jpg "canny"
[image4]: ./describe/4_1canny_ex.png "canny_example"
[image5]: ./describe/3_ROI.png "ROI"
[image6]: ./describe/6_hough.PNG "6_hough"
[image6-1]: ./describe/6_1_hough.PNG "6_1_hough"
[image6-2]: ./describe/6_2_hough.PNG "6_2_hough"
[image6-3]: ./describe/6_3_hough.PNG "6_3_hough"
[image6-4]: ./describe/6_4_hough.PNG "6_4_hough"
[image6-5]: ./describe/6_5_hough.PNG "6_5_hough"



### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

pipeline은 총 9개로 구성하였습니다.

* garyscale
* canny
* gaussian_blur
* region_of_interest
* draw_lines
* hough_line
* weight_img
* find_lane_formula
* seperate_line

#### 1) Grayscale
첫 번째로 영상을 grayscale 로 변환하는 작업을 `grayscale` 함수를 이용하여 수행합니다.

이는 영상에서 line 부분만 뽑아내기 위해 수행하는 과정으로 gray scale로 영상을 변환한 후 일정 기준값 이상의 '흰' 부분만 뽑아낸다면 line을 도드라 지게 얻을 수 있습니다.

original 영상(위), 출력 영상 (아래)
![alt text][image1]
 
![alt text][image2]

#### 2) gaussian blur
그 이후`gaussian_bluer` 함수를 이용하여 블러 처리를 수행 합니다.
이는 이후에 수행할 canny 알고리즘에서 edge 성분을 축출할 때, error를 줄이기 위함이빈다.

#### 3) Canny
앞서 잠시 언급하였듯이 line을 찾기 위해서 edge detection 방식을 이용합니다.

아래 예시 그림과 같이 도로와 line은 gray scale로 보았을 때 dark와 bright 영역으로 구분되는 것을 볼 수 있습니다.  
![alt text][image4]

따라서 영상에 대해 df/dx=delta(picel value)
로 미분 방식으로 edge detection을 수행하게 되면 아래 그림과 같이 edge 성분만 얻을 수 있스빈다.

![alt text][image3]


#### 4) Region of interest (Region masking)

일반적으로 고속도로에서 차 앞에서 line 존재하는 영역은 아래 그림과 같이 일정하게 유지되어 있습니다.

![alt text][image5]

따라서 해당 영역에서의 정보만을 이용하여 line을 tracking하는 것이 높은 성능을 갖고올 수 있습니다.

따라서 `vertices` 변수에 보고자 하는 영역의 값(위 예시엣 사다리꼴 영역)을 삽입한 후에 이를 `region_of_interest` 함수에 넣어 원하는 영역의 edge 성분만 받아옵니다.

#### 5) hough line

`hough_line`함수에서 line 을 찾는 작업을 수행하게 됩니다.

hough 알고리즘에 대해 간단하게 설명 드리면,

![alt text][image6]

위 그림 과 같이 x,y 평면에서 직선의 기울기(m)과 상수(b) 측면의 평면으로 분석한 것을 의미합니다.

만약 다수의 점들을 hough domian 으로 transform 하게 되면 아래 그림과 같이 hough domain 에서 일정 부분에서 겹치는 지점이 나타나게 됩니다.

![alt text][image6-1]

이를 통해 하나의 line을 근사화 할 수 있습니다.

여기서 `cv2.HoughLinesP` 함수를 이용하여 자동차 line을 받아오는데 이 경우 점선으로 표시된 line의 경우 일직선으로 나타나지 않는 문제가 있습니다.

아래 그림의 경우 우측의 line이 실선으로 나타나게 됩니다.
![alt text][image6-2]


이를 해결하기 위해 좌측과 우측 두 가지의 line을 나누어 구분하고

그 영역을 regression 을 이용하여 한 줄로 생성하는 작업을 수행했습니다.

우선 좌우측 영역을 나누기 위해 `separate_lines` 함수를 이용하여 중앙을 기준으로 좌측과 우측의 line 을 나누었습니다.

line의 좌측 영역
![alt text][image6-3]

line의 우측 영역
![alt text][image6-4]

그 이후 각 line 을 한 선으로 구성하기 위해 line 에 대한 formula 를 구하는 함수인 `find_lane_lines_formula`함수를 통해 실제로 그릴 새로운 line 을 구성하였습니다.

이를 최종적으로 `weight_img`함수를 통해 그림 위에 overlay 하였습니다.
![alt text][image6-5]

### 2. Identify potential shortcomings with your current pipeline

일단 기본틀을 그대로 유지하면서 코드를 구성하려다 보니 `hough_line` 함수에 많은 작업들이 들어가있어서 이를 분리하여 pipline을 수행한다면 좀 더 좋은 결과를 가져올 수 있을 것으로 예상합니다.


### 3. Suggest possible improvements to your pipeline

#### 1) lane을 한 line으로 근사화 
lane은 도로 사정에 따라 곡선형태를 유지할 경우도 있는데 이것을 하나의 line을 구성한 점이 개선이 필요할 것 같습니다.

#### 2) region of interest 영역
역시 곡선 도로나 조건에 따라 ROI의 영역이 고정된 점은 수정될 필요가 있어 보입니다.

#### 3) lane 의 color 고려
lane 이 흰색으로만 구성되지 않았을 경우를 고려해야합니다.
이는 특히 마지막 *optional Challenge* 과정을 수행했을 때 현재 알고리즘으로는 좋은 성과를 보이지 않는 원인이 됩니다.