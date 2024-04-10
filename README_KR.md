
<div align="center">
  <img src="assets/logo.svg" width="15%"/>
</div>

# Harmony-Image ⚡️

<div align="center">
  <h3> <a href="https://github.com/Solrikk/Harmony-Image/blob/main/README.md"> English | <a href="https://github.com/Solrikk/Harmony-Image/blob/main/README_GE.md"> Deutsch </a> | <a href="https://github.com/Solrikk/Harmony-Image/blob/main/README_JP.md"> 日本語 </a> | <a href="README_KR.md">한국어</a> | <a href="https://github.com/Solrikk/Harmony-Image/blob/main/README_RU.md">Русский</a> | <a href="README_CN.md">中文</a> </h3>
</div>

 **_Harmony-Image:_** 시각적 검색을 한 단계 높이세요. Harmony-Image의 핵심에는 유사한 이미지의 정확한 발견을 위해 설계된 고급 플랫폼이 있습니다. 구조적 유사성 및 키포인트 매칭 알고리즘의 힘을 활용함으로써, 이 도구는 이미지 비교에 대해 신속하고 정확한 접근을 제공합니다.

이미지를 직접 업로드하거나 URL을 사용하든, Harmony-Image는 방대한 이미지 데이터베이스를 효율적으로 탐색하여 최적의 매치를 찾습니다. 비동기 기술의 능숙한 사용은 빠른 처리를 보장하여, 검색을 원활하고 성공적인 경험으로 전환합니다.

## Features
- **Supports Multiple-Technologies** ☄️

  - `Python` with libraries:
  - `FastAPI` for the web framework.
  - `aiohttp` for asynchronous HTTP requests.
  - `cv2` (OpenCV) for image processing.
  - `numpy` for numerical operations.
  - `skimage` for additional image processing techniques.
  - **Can also be used as a service**

- **Supports Multiple-Indexes** 🚀

  - `Structural Similarity Index (SSIM)` ([details](https://en.wikipedia.org/wiki/Structural_similarity_index_measure))
  - `Feature Matching with ORB (Oriented FAST and Rotated BRIEF) Descriptor` ([details](https://en.wikipedia.org/wiki/Oriented_FAST_and_rotated_BRIEF))
  - `Resizing and Grayscale Conversion` ([details](https://en.wikipedia.org/wiki/Grayscale))
  - `Hashing for Image Identification` 

#

![image](https://wikimedia.org/api/rest_v1/media/math/render/svg/4203f29f732e5cdc9d8a95907ef6d8e12f08ca09)

SSIM compares patterns of pixel intensity changes which are important attributes for human vision. The SSIM score ranges from -1 to +1, where a value of 1 indicates identical images. The process can be broken down into three components:
1) **Luminance Comparison:** This allows for the assessment of the overall luminance of the images. Luminance in SSIM is measured as the average of all pixel values.
2) **Contrast Comparison:** Similarity in contrast is measured through the variance of pixel intensities (variations from the average value), understanding how similar the patterns of light and shadow distribution are between two images.
3) **Structure Comparison:** Compares patterns of spatial pixel distribution, ignoring variations in luminance and contrast. It is done by calculating the covariance between the images relative to their local average values.

![image](https://wikimedia.org/api/rest_v1/media/math/render/svg/96b4f1c3840c3707a93197798dcbfbfff24fa92b)
![image](https://wikimedia.org/api/rest_v1/media/math/render/svg/fcda97086476fa420b3b06568a0d202980a600d0)
![image](https://wikimedia.org/api/rest_v1/media/math/render/svg/1aebd62ba5b7e6ae47780ccfa659333f078d6eac)

To compare images, SSIM (Structural Similarity Index) is used to assess the similarity of images, as well as the ORB (Oriented FAST and Rotated BRIEF) algorithm to detect key points and their descriptors.

ORB (Oriented FAST and Rotated BRIEF) is a method used in computer vision, especially popular for tasks related to object recognition, image matching, and tracking. It's aimed at quickly finding key points in images and describing them in a way that allows for efficient comparison. Let's break down what ORB does into simpler terms:

_Using the ORB algorithm, key points and descriptors are determined for both the current and target images._
![image](https://i.stack.imgur.com/spSvt.png)
The found key points are compared with each other to determine matches. These matches allow assessing the similarity of images from a perspective other than SSIM. The final similarity score is calculated as the average between the SSIM score and the relative number of matching key points (using the ORB algorithm), providing a comprehensive approach to analyzing the similarity of images.

Harmony-Image application, both the SSIM and ORB methods are utilized to find images that are similar to an uploaded image. Here's a simplified explanation of how each method works in the context of your application and contributes to finding similar images:

## How SSIM Works in Harmony-Image:
1) **_Resizing Images:_** When comparing the uploaded image to each image in the database, both images are resized to the same dimensions (256x256 pixels). This standardizes the comparison, making it fair and more efficient since we're working with images of the same size.

2) **_Converting to Grayscale:_** Both images are converted to grayscale. This simplifies the comparison by focusing on the structure and intensity of light rather than being distracted by color differences.

3) **_Structural Similarity Comparison:_** The SSIM method then compares these grayscale images to assess their structural similarity. This involves analyzing how similar the patterns of light and shadow are between the two images, giving a score that reflects their similarity. A high score means the images are structurally similar.

## How ORB Works in Harmony-Image:
1) **_Detecting Key Points:_** ORB first identifies key points in both the uploaded image and each database image. These key points are distinctive spots that can be easily recognized and compared between images, such as corners and interesting textures.

2) **_Describing Key Points:_** For each key point detected, ORB generates a unique descriptor that summarizes the key point's characteristics. This descriptor is made rotation-invariant, meaning it describes the key point in a way that's consistent even if the image is rotated.

3) **_Matching Key Points:_** The application then matches key points between the uploaded image and each database image using these descriptors. The matching process involves finding key points in the database image that have descriptors similar to those in the uploaded image.

4) **_Scoring Matches:_** The more key points that match between two images, the higher the score of similarity based on ORB. This score reflects how many distinctive features are shared between the images.

## Combining SSIM and ORB:
 After calculating similarity scores using both SSIM and ORB for each image comparison, Harmony-Image averages these scores to get a final measure of similarity.
Images from the database are then ranked based on their final similarity scores, and the top 5 most similar images are selected.

 Final Selection of Similar Images:
The application filters out duplicate URLs to ensure a diverse set of similar images.
 It returns URLs of the top similar images, which can then be presented to the user.
 In essence, your application uses a combination of structural analysis (SSIM) and feature matching (ORB) to find and rank images in your database that are most similar to an image uploaded by the user. This dual approach leverages the strengths of both methods, ensuring a robust and nuanced comparison that goes beyond simple pixel-by-pixel analysis.