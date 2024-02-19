DIY 3D Scanning Instrument with DJI livox lidar


Recently, I have been addicted to DIY a set of cheap 3D scanning instruments after get off work. Currently, there are architectural 3D scanning instruments on the market, such as Faru’s. One instrument is more than 100,000 yuan:


Then I reviewed the multiple radars I have used and found that only DJI's most suitable for the needs, because its scanning is in the shape of a chrysanthemum, which can cover a wider range, and at the same time, its price is also the cheapest, as long as more than 6,000.

Then in theory, I only need to add a rotating motor to the DJI radar, and I can realize a 360-degree 3D scanner. However, in practice, I found that there are many problems.

for example:

How to get the angle of rotation in real time?
How to stitch the data of each angle?
After splicing the data of each angle, it is found that the overlap is not very accurate. How to improve the accuracy?
How is the motor connected? How to drive? How to send serial protocol?
How to obtain and save the data of DJI radar?
The above are the contents that have not been touched before, so it is very slow to do, especially the part of the CAN port to serial port protocol, I am not familiar with it at all, and it took a long time to do it.



The final DIY hardware is as follows, and the structure is relatively rough. In order to ensure the synchronization of the radar and the motor, it is even fixed with tape. .



Finally, I scanned it once at home, and scanned the outside community once. The overall effect is ok. The picture effect:


Video effect:


https://zhuanlan.zhihu.com/p/448397248



Improvements in the program:

How to design a more reasonable structure so that the radar can rotate better​?
At present, it is a single-station scan, how to stitch the results of multi-station scans​?
How to support RTK single-station measurement results?
How can I continue to improve the accuracy?


questions to contract me : lonlonago@foxmail.com


Mainly focus on lidar data process \ PCL \ VR \ AR \ image.

questions to contract me : lonlonago@foxmail.com

