GSoC has been invaluable to OpenCV. It both introduces our org to a new set of people we may not have had a chance to interact with otherwise, and gives our mentors a chance to work on contributions to the library that they wanted to see done but had not found the resources to do. But the slots GSoC offers to orgs are limited. There are always projects that cannot be done during GSoC period, and not all the applicants can be accepted into GSoC. Here we list the ideas and projects that we wanted to do below. Any students who are interested in them, who want to contribute to OpenCV can contact us any time. You will be working under the supervision of our mentors to complete the projects, just as in GSoC.

# Ideas

1. #### _IDEA:_ Improve the lightweight object tracking model NanoTrack
   * ***Description:*** [NanoTrack](https://github.com/HonglinChu/SiamTrackers/tree/master/NanoTrack) is a lightweight and fast object tracking deep learning model. It can run at >120FPS on the Apple M1 CPU. It is suitable to be deployed on embedded systems or mobile devices. Even though NanoTrack works fairly good compared to some other object tracking models, there's still room to improve the accuracy.
   * ***Expected Outcomes:*** 
      * The improved NanoTrack model which is of higher accuracy within 10% model size increase. Submit it to [OpenCV Model Zoo](https://github.com/opencv/opencv_zoo).
      * Necessary patches, if any, for OpenCV DNN to support the model.
      * Examples in C++ and Python that demonstrate the use of the model.
   * ***Resources:***
      * [NanoTrack](https://github.com/HonglinChu/SiamTrackers/tree/master/NanoTrack)
   * ***Skills Required:*** Training and using deep learning, basic to good knowledge of computer vision, good C++/Python coding skills.
   * ***Possible Mentors:*** [MU Zihao](mailto:zihao.mu@opencv.org.cn)

1. #### _IDEA:_ Evaluation scripts for models in the OpenCV Model Zoo 
   * ***Description:*** [OpenCV Model Zoo](https://github.com/opencv/opencv_zoo) contains different deep learning models, such as face detection and recognition, human segmentation, QR code recognition etc. Int8 quantized models of most of these models are provided as well. They can run efficiently on various [hardwares](https://github.com/opencv/opencv_zoo#models--benchmark-results). Besides the efficiency, accuracy of the models has to be considered as well. Inaccurate models are less useful even though it's fast. So there's the need to evaluate the accuracy of all the models in the zoo, automatically. 
   * ***Expected Outcomes:*** 
      * Accuracy evaluation scripts for the models.
   * ***Resources:***
      * [OpenCV Model Zoo](https://github.com/opencv/opencv_zoo)
   * ***Skills Required:*** Using deep learning, basic knowledge of computer vision, good C++/Python coding skills.
   * ***Possible Mentors:*** [FENG Yuantao](mailto:yuantao.feng@opencv.org.cn)

1. #### _IDEA:_ Interesting applications implemented using OpenCV 
   * ***Description:*** OpenCV has been widely used since its release back in 2000. Actually many interesting applications can be implemented using OpenCV. Here are some that came to our mind.
      * **Group Photos**<br/> Nowadays we use our phones to take selfies or pitures of friends and families anywhere anytime. This generates huge amount of files in the devices. Grouping the photos in terms of different faces helps organise the files and one also can quickly find the photo he wants to show . This can be done using the face recognition technique. [OpenCV Model Zoo](https://github.com/opencv/opencv_zoo) provides efficient deep learning models for [face detection](https://github.com/opencv/opencv_zoo/blob/master/models/face_detection_yunet) and [face recognition](https://github.com/opencv/opencv_zoo/tree/master/models/face_recognition_sface) with good accuracy.
      * **Virtual Backgrounds**<br/> In the past two or three years, COVID-19 made video conferencing apps popular. Even now we still use these apps quite a lot to have meetings, discussions, give classes etc. Sometimes users don't want their surroundings to be shown in the video, and they use a feature of the apps which is called 'virtual backgrounds'. Virtual backgrounds display an image or video as the users' background instead of the actual background. Terminologically this is called green screen matting or chroma keying. One can easily implement the virtual backgrounds with the help of OpenCV.
      * **Hand Guesture Recognition**      
      * **Palm Tracking**
      * **People Counting**<br/> Nucleic acid test became part of the daily life of the Chinese people. Sometimes the queue is just incredibly long. Nobody wants to stand there in the sun or in the rain waiting for half an hour, an hour or more. If there is one or more cameras filming the testing site, then the number of people can be automatically and constantly analysed by computer using computer vision algorithms. Once the queue shortens, a message is sent to mobile phones telling people that they can go for the test now. Such a complete system requires more than computer vision. But people counting can be solved by using OpenCV.
   * ***Skills Required:*** Basic to good knowledge of computer vision, good C++/Python coding skills.
   * ***Possible Mentors:*** TBD