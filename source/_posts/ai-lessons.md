---
title: AI Lessons
date: 2018-02-27 15:27:54
categories:
  - tech
  - AI
tags:
  - study
  - AI
---

# Synopsis

感谢我的导师@徐一智提供的学习AI（主要是自动驾驶）的课程。
下面是其wiki原文，我还没有完全看完，后续可能会在此基础上进行修改

---

我报名参加了 udacity 的 Self-Driving Car Nanodegree Program 课程，在这里记录下课程内容供大家参考。
课程大纲整个学位有三个核心课程，如下:

* Computer Vision and Deep Learning
In this term, you'll become an expert in applying Computer Vision and Deep Learning on automotive problems. You will teach the car to detect lane lines, predict steering angle, and more all based on just camera data!

* Sensor Fusion, Localization, and Control
In this term, you'll learn how to use an array of sensor data to perceive the environment and control the vehicle. You'll evaluate sensor data from camera, radar, lidar, and GPS, and use these in closed-loop controllers that actuate the vehicle.

* Path Planning, Concentrations, and Systems
In this term, you'll learn how to plan where the vehicle should go, how the vehicle systems work together to get it there, and you'll perform a deep-dive into a concentration of your choice.

 

 

# Lesson 1 welcome

这节介绍了什么是无人驾驶，课程将会讲哪些内容，以及 DARPA 无人车挑战赛的情况. 所有课程视频都免费放到youtube上面了。课程列表如下：

 1. Why Self-Driving Cars?
    https://www.youtube.com/watch?v=jHA__A61nqc

 2. Meet Your Instructors
    https://www.youtube.com/watch?v=QiflJFVOt18

 3. Overview of ND Program
    https://www.youtube.com/watch?v=RZ5iolr4RGs

 4. What Projects Will You Build?
    https://www.youtube.com/watch?v=JGpXenoW0dk

 5. Career Support
    https://www.youtube.com/watch?v=4MGOyNXh4EQ

 6. Nanodegree Support
    一些 mentorship 的文字说明

 7. Deadline Policy
    关于作业等的说明

 8. Self-Driving Car History
    简单提了下 自动驾驶的历史，发源于 DARPA Grand Challenge.

 9. The Great Robot Race
    介绍 DARPA Grand Challenge 的纪录片.
    https://www.youtube.com/watch?v=saVZ_X9GfIM

 10. Self-Driving Car Quiz
    关于以上内容的小测验


# Project 1 Finding Lane Lines

In this first lesson, you'll get taste of some basic computer vision techniques to find lane markings on the road. We will be diving much deeper into computer vision in later lessons, so just relax and have some fun in this first week!

本章节主要讲了怎么写代码完成车道线识别的任务

1. Setting up the Problem
  https://www.youtube.com/watch?v=aIkAcXVxf2w

2. Color Selection
  https://www.youtube.com/watch?v=bNOWJ9wdmhk

3. Color Selection Code Example
  具体的颜色选择代码实例，在最后github链接里能看到相关内容

4. Quiz: Color Selection
  对以上内容的一个小检测

5. Region Masking
  https://www.youtube.com/watch?v=ngN9Cr-QfiI

6. Color and Region Combined
  怎么将color 和 region combined 结合起来的代码实例，在最后project  的github 里能看到相关内容

7. Quiz: Color Region
  对以上问题的一个段落小测验

8. Finding Lines of Any Color
  文字描述什么是finding lines of any color，并提出cv才是解决这个问题的smart 的方法。下一章就讲什么是cv

9. What is Computer Vision?
  https://www.youtube.com/watch?v=wxQhfSdxjKU

10. Canny Edge Detection
  https://www.youtube.com/watch?v=Av2GsgQWX8I
  https://www.youtube.com/watch?v=LQM--KPJjD0

11. Canny to Detect Lane Lines
  讲了怎么用canny算法去检测Lane Lines，在最后project  的github 里能看到相关内容

12. Quiz: Canny Edges
  对Canny掌握程度的测试

13. Hough Transform
  https://www.youtube.com/watch?v=JFwj5UtKmPY

14. Hough Transform to Find Lane Lines
  怎么用 Hough Transform 去寻找直线，在最后project  的github 里能看到相关内容

15. Quiz: Hough Transform
  对Hough Transform掌握程度的测试

16. Project Intro
  https://www.youtube.com/watch?v=LatP7XUPgIE

17. Starter Kit Installation
  完成作业需要的环境，有Anaconda  和 Docker两种方式，在最后project  的github 里能看到相关内容

18. Run Some Code!
  如果环境安装成功会是什么样子

19. Project Expectations
  作业要求

20. Project: Finding Lane Lines on the Road
  作业地址 https://github.com/udacity/CarND-LaneLines-P1

# Lesson 3: Career Services Available to You

介绍了他的找工作服务

# Lesson 4: Introduction to Neural Networks

Learn to build and train neural networks, starting with the foundations in linear and logistic regression, and culminating in backpropagation and multilayer perceptron networks.

1. Introduction to Deep Learning
  https://www.youtube.com/watch?v=uyLRFMI4HkA
 
2. Starting Machine Learning
  https://www.youtube.com/watch?v=UIycORUrPww
 
3. Linear Regression Quiz
  https://www.youtube.com/watch?v=sf51L0RN6zc

4. Linear Regression Answer
  https://s3.cn-north-1.amazonaws.com.cn/u-vid-hd/L5QBqYDNJn0.mp4
  待续...
 

# Project 2 Traffic Sign Classifier

1. Intro to Traffic Sign Classifier
  https://www.youtube.com/watch?v=7pULs5sC_7A

2. LeNet Architecture
  https://www.youtube.com/watch?v=FQYrzmnbsMs

3. AWS GPU Instances
  一篇介绍AWS GPU 主机怎么用的文章。内容和网上其他类似文章差不多

4. LeNet Data
  https://www.youtube.com/watch?v=eCP8o1BnhRA

5. LeNet Implementation
  https://www.youtube.com/watch?v=EA3f6zTbbo0

6. LeNet Training Pipeline
  https://www.youtube.com/watch?v=yof-J8Xktlk

7. LeNet Evaluation Pipeline
  https://www.youtube.com/watch?v=cibEsIOTE60

8. LeNet Training the Model
  https://www.youtube.com/watch?v=9580p7ZRQVY

9. LeNet Testing
  https://www.youtube.com/watch?v=NHoyQSZNWA4

10. LeNet on AWS
  https://www.youtube.com/watch?v=1shr08mozc0

11. LeNet for Traffic Signs
  https://www.youtube.com/watch?v=VfJvSF087SI

12. Project: Traffic Sign Classifier


大作业，要求用LeNet 完成交通标志识别。作业说明，框架代码和数据集都在github上有
https://github.com/udacity/CarND-Traffic-Sign-Classifier-Project

---

夹带的私货:

我在油管里看到了这个视频：https://www.youtube.com/watch?v=aircAruvnKk。
