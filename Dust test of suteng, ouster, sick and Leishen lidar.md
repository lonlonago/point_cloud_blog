Dust test of suteng, ouster, sick and Leishen lidar


video is here:

https://zhuanlan.zhihu.com/p/344305097


There will be a lot of dust in recent usage scenarios, so it is necessary to do a selection test for the dust resistance of lidar. We have successively used flour and milk powder as dust simulations to see the dust resistance of various types of radars. However, this test process is relatively rough, only as a preliminary conclusion of selection.

suteng and ouster, tested by tossing flour:
suteng is the last echo mode, 32-line radar, and the middle harness is dense;

ouster only has a single echo mode, which is a 128-line radar.


Lidar Dust Testing





On the left is Sagitar, on the right is ouster. It can be seen that Sagitar will always detect the fine flour that continues to float in the air. Although ouster has a much denser wire bundle, it only detects dust at the first moment of falling, and will not follow up. detected; however, its immunity to floating dust may also be related to its relatively low lateral resolution.

Radium God:
32-wire uniform harness type;




Flour is not used here, but dust in the actual scene. It can be found that the fine dust that is continuously floating in the air is always detected.

Sick LMS511 single beam radar, multiple echo type:





During the process, milk powder was used to simulate dust, and the video can see that there were obstacles triggered twice, but it did not last. Its 5th echo mode hardware and filtering algorithms still have some merit.

Finally, after comprehensive consideration, we decided to use ouster, which has strong anti-dust ability, but there are too few wiring harnesses.



PS: In the actual use scenario, we use the Lei Shen Lidar, and a simple software algorithm can also filter out more serious dust. . Therefore, this selection has little significance for our follow-up. .


